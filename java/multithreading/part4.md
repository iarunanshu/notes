# Java Multithreading - Part 4: Different Types of Locks

## Overview: Why Custom Locks?

### Limitation of Synchronized
- **Synchronized** puts monitor lock on **object level**
- Different objects have different locks
- Problem: What if we need to lock a critical section regardless of which object is used?

### Example Problem
```java
class SharedResource {
    synchronized void produce() {
        // Critical section
    }
}

// Different objects - different locks!
SharedResource obj1 = new SharedResource();
SharedResource obj2 = new SharedResource();

Thread t1 = new Thread(() -> obj1.produce()); // Lock on obj1
Thread t2 = new Thread(() -> obj2.produce()); // Lock on obj2
// Both threads can enter simultaneously! ❌
```

### Solution: Custom Locks
- Don't depend on object instance
- Provide more flexibility
- Four main types: ReentrantLock, ReadWriteLock, Semaphore, StampedLock

## 1. ReentrantLock

### Definition
- Custom lock that doesn't depend on object instance
- Same thread can acquire lock multiple times (reentrant)
- More flexible than synchronized

### Implementation
```java
import java.util.concurrent.locks.ReentrantLock;

class SharedResource {
    private ReentrantLock lock = new ReentrantLock();
    
    public void produce() {
        lock.lock(); // Acquire lock
        try {
            System.out.println("Lock acquired by: " + 
                Thread.currentThread().getName());
            Thread.sleep(4000); // Critical section work
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock(); // Always release in finally
            System.out.println("Lock released");
        }
    }
}

// Usage with different objects but same lock
public static void main(String[] args) {
    ReentrantLock sharedLock = new ReentrantLock();
    
    SharedResource obj1 = new SharedResource(sharedLock);
    SharedResource obj2 = new SharedResource(sharedLock);
    
    Thread t1 = new Thread(() -> obj1.produce());
    Thread t2 = new Thread(() -> obj2.produce());
    
    t1.start();
    t2.start();
    // Only one thread can enter at a time ✅
}
```

### Key Points
- Use `lock()` to acquire
- Use `unlock()` to release (always in finally block)
- Works across different object instances
- Provides more control than synchronized

## 2. ReadWriteLock

### Concept: Shared vs Exclusive Locks

| Lock Type | Also Called | Multiple Threads? | Operations Allowed |
|-----------|------------|-------------------|-------------------|
| Shared Lock | Read Lock | Yes | Read only |
| Exclusive Lock | Write Lock | No | Read & Write |

### Lock Compatibility Matrix
| Current Lock | Can Acquire Shared? | Can Acquire Exclusive? |
|-------------|-------------------|----------------------|
| No Lock | ✅ Yes | ✅ Yes |
| Shared Lock | ✅ Yes | ❌ No |
| Exclusive Lock | ❌ No | ❌ No |

### Implementation
```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

class SharedResource {
    private ReadWriteLock lock = new ReentrantReadWriteLock();
    private int data = 0;
    
    // Multiple threads can read simultaneously
    public void read() {
        lock.readLock().lock();
        try {
            System.out.println("Reading data: " + data);
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.readLock().unlock();
        }
    }
    
    // Only one thread can write
    public void write(int value) {
        lock.writeLock().lock();
        try {
            System.out.println("Writing data");
            data = value;
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

### When to Use
- Read operations >> Write operations
- Example: Cache systems, configuration files
- Improves performance when reads are frequent

## 3. StampedLock

### Features
1. **All ReadWriteLock capabilities**
2. **Plus Optimistic Reading** (no lock acquired)

### Understanding Optimistic Locking

#### Database Example
```sql
-- Table with version column
CREATE TABLE users (
    id INT,
    name VARCHAR(100),
    type VARCHAR(50),
    version INT DEFAULT 1
);

-- Optimistic update
UPDATE users 
SET type = 'teacher', version = version + 1
WHERE id = 123 AND version = 1;
-- If version changed, update fails
```

#### How It Works
1. Read data and note version/stamp
2. Perform operations locally
3. Before writing, validate version hasn't changed
4. If changed, rollback and retry
5. If unchanged, commit changes

### Implementation
```java
import java.util.concurrent.locks.StampedLock;

class SharedResource {
    private StampedLock lock = new StampedLock();
    private int data = 10;
    
    // Optimistic Read - No lock acquired
    public void optimisticRead() {
        long stamp = lock.tryOptimisticRead(); // Get stamp
        
        // Read data
        int currentData = data;
        
        // Do some work
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        
        // Validate before update
        if (lock.validate(stamp)) {
            // No write occurred, safe to use data
            System.out.println("Update successful");
            data = currentData + 1;
        } else {
            // Write occurred, need to retry
            System.out.println("Rollback - someone modified data");
            // Retry logic here
        }
    }
    
    // Regular read lock
    public void readWithLock() {
        long stamp = lock.readLock();
        try {
            System.out.println("Reading: " + data);
        } finally {
            lock.unlockRead(stamp);
        }
    }
    
    // Write lock
    public void write(int value) {
        long stamp = lock.writeLock();
        try {
            data = value;
            System.out.println("Written: " + data);
        } finally {
            lock.unlockWrite(stamp);
        }
    }
}
```

### Key Points
- Optimistic read doesn't acquire lock - better performance
- Must validate stamp before committing changes
- Falls back to pessimistic locking if needed
- Good for mostly-read scenarios with rare conflicts

## 4. Semaphore

### Definition
- Controls access to resource by multiple threads
- Allows specified number of threads to enter critical section simultaneously

### Implementation
```java
import java.util.concurrent.Semaphore;

class SharedResource {
    private Semaphore semaphore;
    
    public SharedResource(int permits) {
        semaphore = new Semaphore(permits); // Allow 'permits' threads
    }
    
    public void useResource() {
        try {
            semaphore.acquire(); // Get permit
            System.out.println("Resource acquired by: " + 
                Thread.currentThread().getName());
            Thread.sleep(2000); // Use resource
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            semaphore.release(); // Release permit
            System.out.println("Resource released");
        }
    }
}

// Example: Allow 2 threads at a time
public static void main(String[] args) {
    SharedResource resource = new SharedResource(2); // 2 permits
    
    // Create 4 threads
    for (int i = 0; i < 4; i++) {
        new Thread(() -> resource.useResource()).start();
    }
    // Only 2 threads can access simultaneously
}
```

### Real-World Use Cases
1. **Connection Pool**: Limit database connections
2. **Rate Limiting**: Control API access rate
3. **Resource Pool**: Manage limited resources (printers, etc.)

### Example: Connection Pool
```java
class ConnectionPool {
    private Semaphore connectionSemaphore;
    private Queue<Connection> connections;
    
    public ConnectionPool(int maxConnections) {
        connectionSemaphore = new Semaphore(maxConnections);
        // Initialize connection pool
    }
    
    public Connection getConnection() throws InterruptedException {
        connectionSemaphore.acquire(); // Wait if pool is full
        return connections.poll();
    }
    
    public void releaseConnection(Connection conn) {
        connections.offer(conn);
        connectionSemaphore.release(); // Signal availability
    }
}
```

## 5. Condition - Inter-thread Communication with Custom Locks

### Problem
- `wait()` and `notify()` work with synchronized/monitor locks
- Custom locks need different mechanism

### Solution: Condition Interface

| Synchronized | Condition | Purpose |
|-------------|-----------|---------|
| wait() | await() | Make thread wait |
| notify() | signal() | Wake one thread |
| notifyAll() | signalAll() | Wake all threads |

### Implementation: Producer-Consumer with ReentrantLock
```java
import java.util.concurrent.locks.ReentrantLock;
import java.util.concurrent.locks.Condition;

class SharedBuffer {
    private ReentrantLock lock = new ReentrantLock();
    private Condition condition = lock.newCondition();
    private boolean isAvailable = false;
    
    public void produce() {
        lock.lock();
        try {
            while (isAvailable) {
                condition.await(); // Wait if already available
            }
            isAvailable = true;
            System.out.println("Produced item");
            condition.signal(); // Wake up consumer
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
    
    public void consume() {
        lock.lock();
        try {
            while (!isAvailable) {
                condition.await(); // Wait if nothing to consume
            }
            isAvailable = false;
            System.out.println("Consumed item");
            condition.signal(); // Wake up producer
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
        }
    }
}
```

## Comparison Table

| Lock Type | Multiple Readers | Multiple Writers | Use Case |
|-----------|-----------------|------------------|----------|
| synchronized | No | No | Simple mutual exclusion |
| ReentrantLock | No | No | Advanced locking features |
| ReadWriteLock | Yes | No | Read-heavy workloads |
| StampedLock | Yes + Optimistic | No | Very high read, rare write |
| Semaphore | Yes (limited) | Yes (limited) | Resource pooling |

## Pessimistic vs Optimistic Locking

### Pessimistic Locking
- **Approach**: Acquire lock before accessing
- **Examples**: synchronized, ReentrantLock, ReadWriteLock
- **Pros**: Guarantees exclusive access
- **Cons**: Can cause contention, slower

### Optimistic Locking
- **Approach**: No lock, validate before commit
- **Examples**: StampedLock's tryOptimisticRead()
- **Pros**: Better performance, no blocking
- **Cons**: May need retry logic

## Best Practices

1. **Always release locks in finally block**
```java
lock.lock();
try {
    // Critical section
} finally {
    lock.unlock(); // Guaranteed to execute
}
```

2. **Choose the right lock for your use case**
- Simple cases: synchronized
- Need more control: ReentrantLock
- Read >> Write: ReadWriteLock
- Mostly read, rare conflicts: StampedLock
- Limited resources: Semaphore

3. **Avoid lock ordering issues** (prevent deadlock)
4. **Keep critical sections small**
5. **Use tryLock() for non-blocking attempts**

## Key Interview Points

1. **Why use custom locks over synchronized?**
    - Don't depend on object instance
    - More features (try lock, timed lock, etc.)
    - Better for specific scenarios

2. **When to use each lock type?**
    - Know the use cases for each
    - Understand read vs write patterns

3. **Optimistic vs Pessimistic locking**
    - Trade-offs and when to use each
    - How versioning works

4. **Condition vs wait/notify**
    - Why Condition is needed with custom locks
    - Same functionality, different API

## Common Pitfalls

1. ❌ Forgetting to unlock in finally block
2. ❌ Using wait/notify with custom locks (use Condition)
3. ❌ Not validating stamp in StampedLock optimistic reads
4. ❌ Choosing wrong lock type for use case
5. ❌ Creating deadlocks with multiple locks