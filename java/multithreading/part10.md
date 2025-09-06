# Java Multithreading - Part 10: ThreadLocal & Virtual Threads

## Overview
This session covers two important topics:
- **ThreadLocal**: Thread-specific variable storage
- **Virtual Threads vs Platform Threads**: New threading model in Java 19+

## 1. ThreadLocal

### What is ThreadLocal?
- Each thread has its own copy of ThreadLocal variable
- Provides thread-specific storage
- No sharing between threads (thread-safe by design)
- Generic type - can store any object type

### How ThreadLocal Works

```java
// Each thread internally maintains ThreadLocal storage
Thread {
    ThreadLocalMap threadLocals;  // Thread's local variables
}

// When you call threadLocal.set(value)
// 1. Gets current thread
// 2. Stores value in current thread's ThreadLocalMap
// 3. No need to specify which thread
```

### Basic ThreadLocal Usage

```java
public class ThreadLocalExample {
    // Create ThreadLocal object (shared reference, separate values)
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();
    
    public static void main(String[] args) {
        // Main thread sets value
        threadLocal.set(Thread.currentThread().getName());
        System.out.println("Main thread value: " + threadLocal.get()); // "main"
        
        // Create new thread
        Thread thread1 = new Thread(() -> {
            // Thread-1 sets its own value
            threadLocal.set(Thread.currentThread().getName());
            System.out.println("Thread-1 value: " + threadLocal.get()); // "Thread-0"
        });
        
        Thread thread2 = new Thread(() -> {
            // Thread-2 has no value set
            System.out.println("Thread-2 value: " + threadLocal.get()); // null
            
            // Thread-2 sets its own value
            threadLocal.set("Custom Value for Thread-2");
            System.out.println("Thread-2 new value: " + threadLocal.get());
        });
        
        thread1.start();
        thread2.start();
        
        // Main thread still has its own value
        System.out.println("Main thread value again: " + threadLocal.get()); // "main"
    }
}
```

### Internal Working

```java
// When you call threadLocal.set(value)
public void set(T value) {
    Thread t = Thread.currentThread();  // Get current thread
    ThreadLocalMap map = t.threadLocals;  // Get thread's local map
    map.set(this, value);  // Store value with ThreadLocal as key
}

// When you call threadLocal.get()
public T get() {
    Thread t = Thread.currentThread();  // Get current thread
    ThreadLocalMap map = t.threadLocals;  // Get thread's local map
    return map.get(this);  // Retrieve value using ThreadLocal as key
}
```

### ThreadLocal with Thread Pools - Memory Leak Issue

**Problem**: Thread reuse in pools can cause data leakage

```java
public class ThreadLocalLeakExample {
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();
    
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        
        // Task 1 - Sets ThreadLocal
        executor.submit(() -> {
            threadLocal.set("Task-1 Data");
            System.out.println("Task 1: " + threadLocal.get());
            // PROBLEM: Didn't clean up!
        });
        
        // Submit 5 more tasks
        for (int i = 2; i <= 6; i++) {
            final int taskId = i;
            executor.submit(() -> {
                // Some tasks might see old data!
                String value = threadLocal.get();
                if (value != null) {
                    System.out.println("Task " + taskId + 
                        " found leaked data: " + value);
                }
            });
        }
        
        executor.shutdown();
    }
}
```

### Proper ThreadLocal Cleanup

```java
public class ThreadLocalCleanupExample {
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();
    
    public static void processTask(String taskName) {
        try {
            // Set thread-specific value
            threadLocal.set("Processing: " + taskName);
            
            // Use the value
            System.out.println(Thread.currentThread().getName() + 
                " - " + threadLocal.get());
            
            // Do actual work
            performWork();
            
        } finally {
            // ALWAYS clean up in finally block
            threadLocal.remove();  // Critical for thread pools!
        }
    }
    
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        
        for (int i = 1; i <= 10; i++) {
            final String taskName = "Task-" + i;
            executor.submit(() -> processTask(taskName));
        }
        
        executor.shutdown();
    }
}
```

### ThreadLocal Best Practices

1. **Always clean up in thread pools**
```java
try {
    threadLocal.set(value);
    // Use value
} finally {
    threadLocal.remove();  // Prevent memory leaks
}
```

2. **Use initialValue() for defaults**
```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);
// Or
ThreadLocal<List<String>> threadLocal = ThreadLocal.withInitial(ArrayList::new);
```

3. **Common Use Cases**
- User context in web applications
- Database connections per thread
- SimpleDateFormat (not thread-safe)
- Transaction context

### Real-World ThreadLocal Example

```java
public class UserContext {
    private static final ThreadLocal<User> userContext = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userContext.set(user);
    }
    
    public static User getUser() {
        return userContext.get();
    }
    
    public static void clear() {
        userContext.remove();
    }
}

// Web Filter/Interceptor
public class AuthenticationFilter {
    public void doFilter(Request request, Response response) {
        try {
            // Set user for this request thread
            User user = authenticate(request);
            UserContext.setUser(user);
            
            // Process request - any code can access user via UserContext
            processRequest(request, response);
            
        } finally {
            // Clear after request completes
            UserContext.clear();
        }
    }
}
```

## 2. Virtual Threads vs Platform Threads

### Platform (Normal) Threads

**What are Platform Threads?**
- Traditional Java threads (what we've been using)
- 1:1 mapping with OS threads
- Managed by JVM but backed by OS threads
- Limited by OS resources

```java
// Creating platform thread
Thread t1 = new Thread(() -> {
    System.out.println("Platform thread");
});
t1.start();  // JVM asks OS to create native thread
```

**Platform Thread Architecture**
```
Java Code
    ↓
Platform Thread (JVM wrapper)
    ↓
OS Thread (Native thread)
    ↓
CPU Core
```

### Problems with Platform Threads

1. **Expensive Creation**
```java
// Each thread creation = System call to OS
// System calls are expensive (2-3ms per thread)
for (int i = 0; i < 1000; i++) {
    new Thread(() -> {
        // Creates 1000 OS threads - EXPENSIVE!
    }).start();
}
```

2. **Blocking Wastes Resources**
```java
// Platform thread waiting for I/O
Thread platformThread = new Thread(() -> {
    // This thread is blocked for 4 seconds
    String data = database.query("SELECT * FROM large_table");  // 4 sec
    
    // During this time:
    // - Platform thread is blocked
    // - OS thread is also blocked (1:1 mapping)
    // - OS thread can't do other work
});
```

### Virtual Threads (Java 19+)

**Key Concept**: Many-to-many mapping
- Thousands of virtual threads
- Few OS threads (carrier threads)
- JVM manages mapping dynamically

**Virtual Thread Architecture**
```
Many Virtual Threads (JVM objects)
    ↓ (dynamic mapping)
Few Carrier Threads (Platform threads)
    ↓
OS Threads
    ↓
CPU Cores
```

### How Virtual Threads Work

```java
// Conceptual model
class VirtualThreadScheduler {
    // Few OS threads (carriers)
    OSThread[] carriers = new OSThread[8];  // e.g., 8 cores
    
    // Many virtual threads
    VirtualThread[] virtualThreads = new VirtualThread[10000];
    
    void schedule() {
        for (VirtualThread vt : virtualThreads) {
            if (vt.isRunnable() && !vt.isBlocked()) {
                // Attach to available carrier
                OSThread carrier = findAvailableCarrier();
                carrier.run(vt);
            }
            
            if (vt.isBlocked()) {
                // Detach from carrier, let another VT use it
                detachFromCarrier(vt);
            }
        }
    }
}
```

### Virtual Thread Example

```java
public class VirtualThreadExample {
    public static void main(String[] args) throws InterruptedException {
        
        // Method 1: Using Thread class
        Thread virtualThread = Thread.ofVirtual().start(() -> {
            System.out.println("Virtual thread: " + 
                Thread.currentThread().isVirtual());
        });
        
        // Method 2: Using Executors
        ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
        
        // Submit many tasks - creates virtual threads
        for (int i = 0; i < 10000; i++) {
            final int taskId = i;
            executor.submit(() -> {
                try {
                    // Simulate I/O operation
                    Thread.sleep(1000);
                    System.out.println("Task " + taskId + " completed");
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(5, TimeUnit.SECONDS);
    }
}
```

### Performance Comparison

```java
public class ThroughputComparison {
    
    public static void platformThreadTest() throws Exception {
        long start = System.currentTimeMillis();
        ExecutorService executor = Executors.newFixedThreadPool(100);
        
        for (int i = 0; i < 10000; i++) {
            executor.submit(() -> {
                try {
                    Thread.sleep(100);  // Simulate I/O
                } catch (InterruptedException e) {}
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.MINUTES);
        
        System.out.println("Platform threads time: " + 
            (System.currentTimeMillis() - start) + "ms");
    }
    
    public static void virtualThreadTest() throws Exception {
        long start = System.currentTimeMillis();
        ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
        
        for (int i = 0; i < 10000; i++) {
            executor.submit(() -> {
                try {
                    Thread.sleep(100);  // Simulate I/O
                } catch (InterruptedException e) {}
            });
        }
        
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.MINUTES);
        
        System.out.println("Virtual threads time: " + 
            (System.currentTimeMillis() - start) + "ms");
    }
}
```

## 3. Virtual vs Platform Thread Comparison

| Aspect | Platform Thread | Virtual Thread |
|--------|----------------|----------------|
| OS Thread Mapping | 1:1 (one-to-one) | M:N (many-to-many) |
| Creation Cost | Expensive (system call) | Cheap (JVM object) |
| Memory Usage | ~1MB per thread | ~1KB per thread |
| Blocking Behavior | Blocks OS thread | Only blocks virtual thread |
| Number of Threads | Limited (1000s) | Unlimited (millions) |
| Use Case | CPU-intensive tasks | I/O-intensive tasks |
| Backward Compatible | N/A | Yes, fully compatible |

## 4. When to Use What?

### Use Platform Threads for:
- CPU-intensive operations
- Tasks requiring OS-level features
- Real-time applications
- When you need thread priorities

### Use Virtual Threads for:
- I/O-bound operations
- High concurrency requirements
- Microservices handling many requests
- Replacing thread pools for simple tasks

## 5. Virtual Thread Limitations

```java
// Don't use synchronized blocks - use locks instead
// Bad for virtual threads
synchronized(lock) {
    // This pins the carrier thread
}

// Good for virtual threads
ReentrantLock lock = new ReentrantLock();
lock.lock();
try {
    // Doesn't pin carrier thread
} finally {
    lock.unlock();
}
```

## Interview Key Points

### ThreadLocal
1. **Purpose**: Thread-specific variable storage
2. **Memory Leak Risk**: Must clean up in thread pools
3. **Use Cases**: User context, non-thread-safe utilities
4. **Key Methods**: set(), get(), remove()

### Virtual Threads
1. **Goal**: Higher throughput, not lower latency
2. **Key Difference**: M:N mapping vs 1:1 mapping
3. **Best For**: I/O-bound tasks with high concurrency
4. **Limitations**: Avoid synchronized blocks
5. **Availability**: Java 19+ (preview), Java 21+ (stable)

## Best Practices Summary

### ThreadLocal
```java
// Always use try-finally for cleanup
try {
    threadLocal.set(value);
    // Use value
} finally {
    threadLocal.remove();
}
```

### Virtual Threads
```java
// Prefer virtual threads for I/O operations
ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();

// Use proper synchronization
Lock lock = new ReentrantLock();  // Not synchronized blocks
```

## Summary
- **ThreadLocal** provides thread-isolated storage but requires careful cleanup
- **Virtual Threads** revolutionize Java concurrency for I/O-bound applications
- Virtual threads provide massive scalability improvements for appropriate use cases
- Both features are powerful but require understanding their proper use cases