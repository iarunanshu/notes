# Java Multithreading - Part 5: Atomic Variables & Concurrent Collections

## Overview: Two Ways to Achieve Concurrency

### 1. Lock-Based Mechanism
- **Examples**: synchronized, ReentrantLock, ReadWriteLock, Semaphore, StampedLock
- **Approach**: Use locks to ensure only one thread enters critical section
- **Use Case**: Complex business logic, multiple operations

### 2. Lock-Free Mechanism
- **Examples**: Atomic variables (AtomicInteger, AtomicBoolean, etc.)
- **Approach**: Use CAS (Compare-And-Swap) operations
- **Use Case**: Simple read-modify-update operations
- **Performance**: Faster than lock-based (no blocking)

### Important Note
Lock-free is **NOT** an alternative to lock-based mechanisms. It only works for specific use cases.

## CAS (Compare-And-Swap) Operation

### What is CAS?
- **Low-level CPU operation** (hardware support)
- **Atomic operation** guaranteed by CPU
- Works across multiple CPU cores
- Foundation for lock-free concurrency

### How CAS Works

CAS accepts three parameters:
1. **Memory location** (where variable is stored)
2. **Expected value** (what we expect to find)
3. **New value** (what we want to update to)

```
CAS Operation Steps:
1. READ: Load variable from memory
2. COMPARE: Check if memory value == expected value
3. SWAP: If match, update to new value
         If no match, return false
```

### Visual Example
```
Initial: Memory[X] = 10

Thread 1: CAS(Memory[X], expected=10, new=12)
Step 1: Read Memory[X] → 10
Step 2: Compare 10 == 10 → TRUE
Step 3: Update Memory[X] = 12 → SUCCESS

Thread 2: CAS(Memory[X], expected=10, new=15)
Step 1: Read Memory[X] → 12 (changed by Thread 1)
Step 2: Compare 12 == 10 → FALSE
Step 3: No update → FAIL (must retry)
```

### CAS vs Optimistic Locking

| Aspect | CAS | Optimistic Locking |
|--------|-----|-------------------|
| Level | CPU/Hardware | Application/Database |
| Where | Memory operations | Database rows |
| Version | Internal (CPU) | Explicit column |
| Language | All modern CPUs | Database feature |

### ABA Problem in CAS
**Problem**: Value changes from A→B→A between read and CAS
**Solution**: Add version/timestamp with value

```
Timeline:
T1: Memory[X] = 10 (version 1)
T2: Thread 1 reads X = 10
T3: Thread 2 changes X = 12 (version 2)
T4: Thread 3 changes X = 10 (version 3)
T5: Thread 1 CAS(X, 10, 15) → Succeeds but shouldn't!

With version:
T5: Thread 1 CAS(X, {10,v1}, 15) → Fails (current is {10,v3})
```

## Atomic Variables in Java

### Problem: Non-Atomic Operations
```java
// This looks atomic but it's NOT!
counter++; // Actually 3 operations:
           // 1. READ counter value
           // 2. INCREMENT by 1
           // 3. WRITE back to memory

// Problem demonstration
class SharedResource {
    private int counter = 0;
    
    public void increment() {
        counter++; // NOT thread-safe!
    }
}

// With 2 threads incrementing 200 times each
// Expected: 400
// Actual: Could be 371 (race condition!)
```

### Solution 1: Using Locks (Synchronized)
```java
class SharedResource {
    private int counter = 0;
    
    public synchronized void increment() {
        counter++; // Now thread-safe with lock
    }
}
```

### Solution 2: Using Atomic Variables (Lock-Free)
```java
import java.util.concurrent.atomic.AtomicInteger;

class SharedResource {
    private AtomicInteger counter = new AtomicInteger(0);
    
    public void increment() {
        counter.incrementAndGet(); // Thread-safe, lock-free!
    }
    
    public int getCounter() {
        return counter.get();
    }
}
```

### How AtomicInteger Works Internally
```java
// Simplified version of incrementAndGet()
public final int incrementAndGet() {
    int expected, newValue;
    do {
        expected = this.value;        // Read current value
        newValue = expected + 1;      // Calculate new value
    } while (!compareAndSwap(expected, newValue)); // CAS operation
    return newValue;
}
```

### Types of Atomic Classes

| Class | Use Case | Example Methods |
|-------|----------|-----------------|
| AtomicInteger | Integer operations | incrementAndGet(), addAndGet() |
| AtomicLong | Long operations | decrementAndGet(), getAndAdd() |
| AtomicBoolean | Boolean flags | compareAndSet(), getAndSet() |
| AtomicReference | Object references | compareAndSet(), updateAndGet() |

### When to Use Atomic Variables
✅ **Good for:**
- Simple read-modify-update operations
- Counters, flags, single value updates
- High-frequency operations

❌ **Not suitable for:**
- Complex multi-step operations
- Multiple related variables
- Operations requiring rollback

## Volatile Keyword

### CPU Cache Architecture
```
Thread 1 → CPU Core 1 → L1 Cache ↘
                                    → L2 Cache → Main Memory
Thread 2 → CPU Core 2 → L1 Cache ↗
```

### Problem Without Volatile
```java
// Thread 1 (Core 1)
x = 10;  // Updates L1 cache of Core 1
x++;     // x = 11 in Core 1's L1 cache

// Thread 2 (Core 2) - May not see update!
int value = x;  // Might read 10 from memory (stale value)
```

### Solution With Volatile
```java
private volatile int x = 10;

// Thread 1
x++;  // Writes directly to main memory

// Thread 2
int value = x;  // Reads directly from main memory
```

### Volatile Characteristics

| Aspect | Description |
|--------|-------------|
| **Visibility** | Changes by one thread visible to all threads |
| **Memory Access** | Direct read/write to main memory (bypasses cache) |
| **Thread Safety** | ❌ NOT thread-safe by itself |
| **Atomicity** | ❌ Does NOT provide atomicity |
| **Use Case** | Flags, status variables (single write, multiple reads) |

### Volatile vs Atomic

| Feature | Volatile | Atomic |
|---------|----------|--------|
| Visibility | ✅ Yes | ✅ Yes (also volatile) |
| Thread Safety | ❌ No | ✅ Yes |
| Atomicity | ❌ No | ✅ Yes |
| Performance | Fast reads | CAS operations |
| Use Case | Simple flags | Counters, updates |

### Example: Volatile is NOT Thread-Safe
```java
private volatile int counter = 0;

// Still NOT thread-safe!
public void increment() {
    counter++;  // volatile doesn't make ++ atomic
}

// Need AtomicInteger for thread-safe increment
private AtomicInteger counter = new AtomicInteger(0);
```

## Concurrent Collections

### Thread-Safe Collection Alternatives

| Regular Collection | Concurrent Alternative | Implementation |
|-------------------|----------------------|----------------|
| ArrayList | CopyOnWriteArrayList | Copy on modification |
| HashSet | CopyOnWriteArraySet | Copy on modification |
| HashMap | ConcurrentHashMap | Segment locking |
| LinkedList | ConcurrentLinkedQueue | CAS operations |
| PriorityQueue | PriorityBlockingQueue | ReentrantLock |
| ArrayDeque | LinkedBlockingDeque | ReentrantLock |

### Implementation Details

#### Lock-Based Collections
```java
// PriorityBlockingQueue - Uses ReentrantLock
public boolean offer(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        // Add element
        return true;
    } finally {
        lock.unlock();
    }
}
```

#### Lock-Free Collections
```java
// ConcurrentLinkedQueue - Uses CAS
public boolean offer(E e) {
    Node<E> newNode = new Node<E>(e);
    for (;;) {  // Retry loop
        Node<E> t = tail;
        if (casNext(t, null, newNode)) {  // CAS operation
            casTail(t, newNode);
            return true;
        }
    }
}
```

### When to Use Which Collection

| Collection | Best For | Avoid When |
|------------|----------|------------|
| ConcurrentHashMap | Frequent reads/writes | Need consistent iteration |
| CopyOnWriteArrayList | Many reads, few writes | Frequent modifications |
| BlockingQueue | Producer-consumer | Non-blocking needed |
| ConcurrentLinkedQueue | Lock-free queue | Need blocking operations |

## Complete Example: Counter Implementation Comparison

```java
// 1. Non-Thread-Safe
class UnsafeCounter {
    private int count = 0;
    
    public void increment() {
        count++;  // Race condition!
    }
    
    public int getCount() {
        return count;
    }
}

// 2. Synchronized (Lock-Based)
class SynchronizedCounter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}

// 3. Atomic (Lock-Free)
class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // CAS operation
    }
    
    public int getCount() {
        return count.get();
    }
}

// 4. Volatile (NOT sufficient for increment!)
class VolatileCounter {
    private volatile int count = 0;
    
    public void increment() {
        count++;  // Still NOT thread-safe!
    }
    
    public int getCount() {
        return count;  // Visibility guaranteed
    }
}
```

## Performance Comparison

| Approach | Performance | Blocking | Use Case |
|----------|------------|----------|----------|
| No Synchronization | Fastest (unsafe) | No | Single-threaded only |
| Volatile | Fast reads | No | Simple flags |
| Atomic (CAS) | Fast | No | Simple updates |
| Synchronized | Slower | Yes | Complex operations |
| ReentrantLock | Configurable | Yes | Advanced features |

## Key Takeaways

### When to Use What

1. **Atomic Variables**
    - ✅ Simple counters, flags
    - ✅ High-frequency updates
    - ✅ Read-modify-update pattern
    - ❌ Complex multi-step operations

2. **Volatile**
    - ✅ Simple status flags
    - ✅ One writer, multiple readers
    - ❌ Compound operations (like ++)
    - ❌ Thread synchronization

3. **Locks (Synchronized/ReentrantLock)**
    - ✅ Complex critical sections
    - ✅ Multiple related operations
    - ✅ Need for wait/notify
    - ❌ Simple value updates

4. **Concurrent Collections**
    - ✅ Thread-safe data structures needed
    - ✅ Multiple threads accessing collection
    - Choose based on read/write patterns

## Interview Questions & Answers

**Q1: What is CAS and how does it work?**
- Hardware-level atomic operation
- Three steps: Read, Compare, Swap
- Retry on failure

**Q2: Difference between volatile and atomic?**
- Volatile: Only visibility, not thread-safe
- Atomic: Visibility + thread-safety via CAS

**Q3: When would you use AtomicInteger over synchronized?**
- Simple increment/decrement operations
- High-frequency updates
- Avoiding lock overhead

**Q4: Can volatile make i++ thread-safe?**
- No, i++ is three operations
- Volatile only ensures visibility
- Need AtomicInteger for thread-safe increment

**Q5: What is ABA problem in CAS?**
- Value changes A→B→A between operations
- CAS succeeds but shouldn't
- Solution: Version/timestamp tracking