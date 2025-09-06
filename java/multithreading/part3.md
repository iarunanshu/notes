# Java Multithreading - Part 3 Study Notes

## 1. Producer-Consumer Problem Solution

### Problem Statement
- Two threads (Producer & Consumer) share a fixed-size buffer (queue)
- **Producer**: Generates data and adds to buffer
- **Consumer**: Removes and processes data from buffer
- **Constraints**:
    - Producer must wait if buffer is full
    - Consumer must wait if buffer is empty

### Implementation

```java
class SharedBuffer {
    private Queue<Integer> queue;
    private int bufferSize;
    
    public SharedBuffer(int bufferSize) {
        this.queue = new LinkedList<>();
        this.bufferSize = bufferSize;
    }
    
    public synchronized void produce(int item) {
        // Wait if buffer is full
        while (queue.size() == bufferSize) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        queue.add(item);
        System.out.println("Produced: " + item);
        notifyAll(); // Wake up waiting consumers
    }
    
    public synchronized int consume() {
        // Wait if buffer is empty
        while (queue.isEmpty()) {
            try {
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        int item = queue.poll();
        System.out.println("Consumed: " + item);
        notifyAll(); // Wake up waiting producers
        return item;
    }
}

// Usage
public static void main(String[] args) {
    SharedBuffer buffer = new SharedBuffer(3);
    
    // Producer Thread
    Thread producer = new Thread(() -> {
        for (int i = 1; i <= 6; i++) {
            buffer.produce(i);
        }
    });
    
    // Consumer Thread
    Thread consumer = new Thread(() -> {
        for (int i = 1; i <= 6; i++) {
            buffer.consume();
        }
    });
    
    producer.start();
    consumer.start();
}
```

### Key Points
- Both methods are `synchronized` - ensures mutual exclusion
- Use `while` loop (not `if`) for condition checking - prevents spurious wakeups
- `notifyAll()` wakes up all waiting threads
- Monitor lock is on the shared buffer object

## 2. Deprecated Methods: stop(), suspend(), resume()

### Why These Methods Are Deprecated

#### stop() Method
- **Problem**: Terminates thread abruptly
- **Issues**:
    - No lock release
    - No resource cleanup
    - Can cause deadlock situations
    - Thread dies immediately without proper cleanup

#### suspend() Method
- **Problem**: Puts thread on hold WITHOUT releasing locks
- **Issues**:
    - Does NOT release monitor locks (unlike wait())
    - Can lead to deadlock
    - Other threads waiting for lock will be blocked forever

#### resume() Method
- **Purpose**: Resume suspended threads
- **Deprecated because**: Since suspend() is deprecated, resume() must be too

### Example Demonstrating the Problem

```java
class SharedResource {
    public synchronized void produce() {
        System.out.println("Lock acquired");
        try {
            Thread.sleep(8000); // Hold lock for 8 seconds
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Lock released");
    }
}

public static void main(String[] args) {
    SharedResource resource = new SharedResource();
    
    Thread thread1 = new Thread(() -> {
        resource.produce();
    });
    
    Thread thread2 = new Thread(() -> {
        try {
            Thread.sleep(1000); // Wait 1 second
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        resource.produce(); // Will wait for lock
    });
    
    thread1.start();
    thread2.start();
    
    // Suspend thread1 after 3 seconds
    try {
        Thread.sleep(3000);
        thread1.suspend(); // DEPRECATED - Lock NOT released!
        // thread2 will wait forever for the lock
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}
```

### Alternative Approaches
- Use `wait()` and `notify()` instead of suspend/resume
- Use flag-based approach for stopping threads
- Use `interrupt()` for thread interruption

## 3. Thread Joining

### Definition
When `join()` is invoked on a thread object, the current thread blocks and waits for the specified thread to finish.

### Syntax
```java
thread.join();        // Wait indefinitely
thread.join(1000);    // Wait maximum 1000ms
```

### Example
```java
public static void main(String[] args) {
    System.out.println("Main thread started");
    
    Thread thread1 = new Thread(() -> {
        System.out.println("Thread1 starting work");
        try {
            Thread.sleep(5000); // Simulate work
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Thread1 completed");
    });
    
    thread1.start();
    
    try {
        System.out.println("Main waiting for thread1");
        thread1.join(); // Main thread waits for thread1 to complete
        System.out.println("Thread1 finished, main continues");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    
    System.out.println("Main thread finished");
}
```

### Use Cases
- Coordinate between threads
- Ensure certain tasks complete before proceeding
- Wait for dependent threads to finish
- Sequential execution when needed

## 4. Thread Priority

### Priority Range
- **Range**: 1 to 10
- **MIN_PRIORITY**: 1 (lowest)
- **NORM_PRIORITY**: 5 (default)
- **MAX_PRIORITY**: 10 (highest)

### Setting Priority
```java
Thread thread = new Thread(() -> {
    // Thread task
});

thread.setPriority(Thread.MAX_PRIORITY);  // Set to 10
thread.setPriority(7);                    // Custom priority
```

### Important Points
- **NOT GUARANTEED**: JVM doesn't guarantee to follow priority order
- Just a **hint** to thread scheduler, not a strict rule
- **Don't rely on it** in production code
- New threads inherit parent thread's priority
- **Rarely used** in real applications

### Why Not to Use
```java
// Even with priorities set, execution order is NOT guaranteed
Thread t1 = new Thread(() -> System.out.println("T1"));
Thread t2 = new Thread(() -> System.out.println("T2"));
Thread t3 = new Thread(() -> System.out.println("T3"));

t1.setPriority(5);   // Normal
t2.setPriority(10);  // Highest
t3.setPriority(1);   // Lowest

// Expected: T2, T1, T3
// Reality: Could be any order!
```

## 5. Daemon Threads

### Definition
- Background threads that provide services to user threads
- Automatically terminated when all user threads finish

### Types of Threads
1. **User Threads**: Normal threads (default)
2. **Daemon Threads**: Background service threads

### Creating Daemon Thread
```java
Thread daemonThread = new Thread(() -> {
    while (true) {
        System.out.println("Daemon running...");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            break;
        }
    }
});

daemonThread.setDaemon(true); // Must be set before start()
daemonThread.start();
```

### Characteristics
- Lives only while at least one user thread is alive
- JVM exits when only daemon threads remain
- Automatically stops when all user threads complete
- Cannot prevent JVM from exiting

### Example
```java
public static void main(String[] args) {
    System.out.println("Main started");
    
    // Create daemon thread
    Thread daemon = new Thread(() -> {
        System.out.println("Daemon started");
        try {
            Thread.sleep(10000); // Try to sleep for 10 seconds
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Daemon finished"); // Won't print if main exits
    });
    
    daemon.setDaemon(true);
    daemon.start();
    
    System.out.println("Main finished");
    // JVM exits here, daemon thread is terminated
}
```

### Real-World Examples of Daemon Threads
1. **Garbage Collector**: Cleans up unused objects
2. **Auto-save**: Periodically saves work in editors
3. **Logging**: Background logging services
4. **Monitoring**: System monitoring threads
5. **Cache cleanup**: Periodic cache clearing

### Best Practices
- Use daemon threads for background services only
- Don't use for critical tasks that must complete
- Set daemon status before starting thread
- Good for housekeeping tasks

## Summary of Key Concepts

### Thread Lifecycle Control
| Method | Purpose | Issues/Notes |
|--------|---------|-------------|
| `wait()` | Thread waits, releases locks | Use in synchronized context |
| `notify()` | Wake one waiting thread | Must own monitor lock |
| `join()` | Wait for thread to complete | Good for coordination |
| `sleep()` | Pause execution | Keeps locks |
| `stop()` ❌ | Terminate thread | DEPRECATED - unsafe |
| `suspend()` ❌ | Pause thread | DEPRECATED - keeps locks |
| `resume()` ❌ | Resume suspended thread | DEPRECATED |

### Thread Types Comparison
| Aspect | User Thread | Daemon Thread |
|--------|------------|---------------|
| Default | Yes | No (must set explicitly) |
| JVM Exit | Prevents JVM exit | Doesn't prevent JVM exit |
| Lifetime | Independent | Depends on user threads |
| Use Case | Main application logic | Background services |

## Interview Tips

1. **Producer-Consumer**: Classic synchronization problem - know the implementation
2. **Deprecated Methods**: Understand WHY they're deprecated (lock issues)
3. **Join**: Useful for thread coordination and dependencies
4. **Priority**: Explain why it's unreliable and shouldn't be used
5. **Daemon Threads**: Know real-world examples (GC, logging, auto-save)

## Common Pitfalls to Avoid

1. ❌ Using deprecated methods (stop, suspend, resume)
2. ❌ Relying on thread priority for execution order
3. ❌ Forgetting to set daemon status before starting thread
4. ❌ Using `if` instead of `while` with wait()
5. ❌ Not handling InterruptedException properly