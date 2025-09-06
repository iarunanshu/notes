# Java Multithreading - Part 6: Thread Pools & ThreadPoolExecutor

## Overview: Interview Question
**Q: In a thread pool, why did you choose core pool size as 2? Why not 10 or 15?**
- This requires understanding of thread pools
- Must consider multiple factors (CPU cores, memory, task nature)
- We'll answer this at the end after understanding thread pools

## What is a Thread Pool?

### Definition
1. **Collection of threads** (worker threads) available to perform submitted tasks
2. **Thread reusability** - Once task completes, thread returns to pool for new tasks
3. **Avoids thread creation overhead** - Reuses existing threads instead of creating new ones

### Visual Representation
```
Thread Pool: [Thread-1, Thread-2, ..., Thread-n]
                ↓          ↓              ↓
Tasks Queue: [Task-1, Task-2, Task-3, Task-4, ...]
```

### Thread Creation Cost
- **Memory allocation**: Stack space for each thread
- **Program counter** initialization
- **Other thread-specific resources**
- Creating threads takes time - pool eliminates this overhead

## Advantages of Thread Pools

### 1. Save Thread Creation Time
```java
// Without Thread Pool - Creates new thread each time
Thread t1 = new Thread(task1); // Time to create
Thread t2 = new Thread(task2); // Time to create

// With Thread Pool - Reuses existing threads
executor.submit(task1); // No creation time
executor.submit(task2); // No creation time
```

### 2. Lifecycle Management Abstraction
- **Without pool**: Manually manage NEW → RUNNABLE → WAITING → TERMINATED
- **With pool**: Framework handles lifecycle automatically
- Reduces complexity

### 3. Performance Improvement
- **Controls number of threads** - Prevents unlimited thread creation
- **Reduces context switching** - Limited threads = less CPU overhead
- **Better resource utilization**

#### Context Switching Problem
```
Without Control:
- 100 tasks → 100 threads created
- Only 2 CPU cores available
- Excessive context switching
- CPU spends more time switching than processing

With Thread Pool:
- 100 tasks → Fixed number of threads (e.g., 4)
- Controlled context switching
- Better CPU utilization
```

## Java Executor Framework

### Framework Hierarchy
```
Executor (Interface)
    ↓ execute(Runnable)
ExecutorService (Interface)
    ↓ submit(), shutdown(), etc.
ThreadPoolExecutor (Class)
    ↓ Customizable thread pool
ScheduledExecutorService (Interface)
    ↓ Scheduled execution
ForkJoinPool (Class)
    ↓ Work-stealing algorithm
```

## ThreadPoolExecutor - Deep Dive

### Constructor Parameters
```java
ThreadPoolExecutor(
    int corePoolSize,           // Minimum threads
    int maximumPoolSize,        // Maximum threads
    long keepAliveTime,         // Idle thread timeout
    TimeUnit unit,              // Time unit for keepAliveTime
    BlockingQueue<Runnable> workQueue,  // Task queue
    ThreadFactory threadFactory,        // Custom thread creation
    RejectedExecutionHandler handler    // Rejection policy
)
```

### Parameter Details

#### 1. Core Pool Size
- **Minimum number of threads** always in pool
- Created initially and kept even if idle
- Example: `corePoolSize = 3` → Always 3 threads minimum

#### 2. Maximum Pool Size
- **Maximum threads** allowed in pool
- Extra threads created only when needed
- Example: `maximumPoolSize = 5` → Can grow from 3 to 5

#### 3. Keep Alive Time & TimeUnit
- **Idle thread timeout** - How long idle threads survive
- Only applies when `allowCoreThreadTimeout(true)`
- Example: `keepAliveTime = 5, TimeUnit.MINUTES`

#### 4. Work Queue
- **Holds tasks** before execution
- Types:
    - **Bounded**: Fixed capacity (ArrayBlockingQueue)
    - **Unbounded**: No limit (LinkedBlockingQueue)
- Bounded preferred for control

#### 5. Thread Factory
- Customize thread properties:
    - Thread name
    - Priority
    - Daemon flag

#### 6. Rejected Execution Handler
- Handles tasks when pool and queue are full
- Built-in policies:
    - **AbortPolicy**: Throws exception
    - **DiscardPolicy**: Silently discards
    - **CallerRunsPolicy**: Runs in caller thread
    - **DiscardOldestPolicy**: Removes oldest from queue

## How ThreadPoolExecutor Works - The Flow

### Task Submission Flow
```
New Task Arrives
    ↓
1. Any core thread free?
    YES → Assign to thread
    NO ↓
2. Can add to queue?
    YES → Add to queue
    NO ↓
3. Can create new thread? (current < max)
    YES → Create thread & assign
    NO ↓
4. Reject task (use rejection handler)
```

### Detailed Example
```java
// Configuration
corePoolSize = 3
maximumPoolSize = 5
queueCapacity = 2

// Execution Flow:
Task 1 → Thread 1 (core)       ✓
Task 2 → Thread 2 (core)       ✓
Task 3 → Thread 3 (core)       ✓
Task 4 → Queue position 1      ✓
Task 5 → Queue position 2      ✓
Task 6 → Thread 4 (new)        ✓
Task 7 → Thread 5 (new)        ✓
Task 8 → REJECTED              ✗
```

## Implementation Example

### Complete Working Example
```java
import java.util.concurrent.*;

public class ThreadPoolExample {
    
    // Custom Thread Factory
    static class CustomThreadFactory implements ThreadFactory {
        private int counter = 0;
        
        @Override
        public Thread newThread(Runnable r) {
            Thread thread = new Thread(r);
            thread.setName("Worker-" + counter++);
            thread.setPriority(Thread.NORM_PRIORITY);
            thread.setDaemon(false);
            return thread;
        }
    }
    
    // Custom Rejection Handler
    static class CustomRejectHandler implements RejectedExecutionHandler {
        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            System.out.println("Task rejected: " + r.toString());
            // Log for debugging
        }
    }
    
    public static void main(String[] args) {
        // Create ThreadPoolExecutor
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2,                              // Core pool size
            4,                              // Maximum pool size
            10,                             // Keep alive time
            TimeUnit.MINUTES,               // Time unit
            new ArrayBlockingQueue<>(2),    // Queue capacity = 2
            new CustomThreadFactory(),       // Custom factory
            new CustomRejectHandler()        // Rejection handler
        );
        
        // Submit tasks
        for (int i = 1; i <= 7; i++) {
            final int taskId = i;
            executor.submit(() -> {
                System.out.println("Task " + taskId + 
                    " processed by " + Thread.currentThread().getName());
                try {
                    Thread.sleep(5000); // Simulate work
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        
        // Shutdown executor
        executor.shutdown();
    }
}
```

### Expected Output Analysis
```
Tasks 1-7 submitted:
- Tasks 1-2: Handled by core threads
- Tasks 3-4: Added to queue
- Tasks 5-6: New threads created
- Task 7: REJECTED (all threads busy, queue full)
```

## Thread Pool Lifecycle

### States
1. **RUNNING**: Accepts new tasks, processes queued tasks
2. **SHUTDOWN**: No new tasks, completes existing tasks
3. **STOP**: No new tasks, interrupts running tasks
4. **TERMINATED**: All tasks complete, all threads terminated

### Methods
```java
executor.shutdown();     // Graceful shutdown
executor.shutdownNow();  // Force shutdown
executor.isTerminated(); // Check if terminated
```

## Interview Question Answer: Core Pool Size Selection

### Factors to Consider

#### 1. CPU Cores
```java
int cores = Runtime.getRuntime().availableProcessors();
// Example: 64 cores available
```

#### 2. Task Nature
- **CPU-intensive**: Threads ≈ CPU cores
- **I/O-intensive**: Threads > CPU cores (threads wait for I/O)

#### 3. Formula for Maximum Threads
```
Threads = CPU_Cores × (1 + Wait_Time/Processing_Time)

Example:
- CPU cores = 64
- Wait time = 50ms (I/O waiting)
- Processing time = 100ms (CPU work)
- Threads = 64 × (1 + 50/100) = 96
```

#### 4. JVM Memory Constraints
```
JVM Memory Calculation:
- Total JVM: 2GB (2000MB)
- Heap: 1GB (1000MB)
- Code cache: 128MB
- JVM overhead: 256MB
- Available for threads: 500MB

Thread Memory:
- Stack per thread: 2MB
- Other per thread: 3MB
- Total per thread: 5MB
- Max threads = 500MB / 5MB = 100 threads
```

#### 5. Request Memory Requirements
```
Heap Constraint:
- Heap available: 1GB
- Memory per request: 10MB
- Safe usage: 60% of heap = 600MB
- Max concurrent requests = 600MB / 10MB = 60
- Therefore, max threads = 60
```

### Final Answer Approach
```java
// Step 1: Calculate based on CPU and task nature
int cpuBasedThreads = 64; // From formula

// Step 2: Check JVM memory limit
int memoryBasedThreads = 100; // From JVM calculation

// Step 3: Check heap/request memory limit
int heapBasedThreads = 60; // From heap calculation

// Step 4: Take minimum of all constraints
int maxThreads = Math.min(cpuBasedThreads, 
                         Math.min(memoryBasedThreads, heapBasedThreads));
// Result: 60 threads maximum

// Step 5: Set pool sizes
int corePoolSize = maxThreads * 0.5;    // 30 (50% for average load)
int maximumPoolSize = maxThreads;        // 60 (100% for peak load)
```

## Best Practices

### 1. Queue Size Selection
```java
// Bounded queue preferred
int queueSize = corePoolSize * 2; // Rule of thumb
BlockingQueue<Runnable> queue = new ArrayBlockingQueue<>(queueSize);
```

### 2. Naming Threads
```java
ThreadFactory factory = new ThreadFactory() {
    AtomicInteger counter = new AtomicInteger(0);
    public Thread newThread(Runnable r) {
        return new Thread(r, "MyPool-Thread-" + counter.incrementAndGet());
    }
};
```

### 3. Monitoring
- Use profiling tools
- Monitor thread utilization
- Track rejection rates
- Measure task completion times

### 4. Graceful Shutdown
```java
executor.shutdown();
try {
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
} catch (InterruptedException e) {
    executor.shutdownNow();
}
```

## Common Pitfalls

1. **Setting core = max**: No elasticity for load spikes
2. **Unbounded queue**: Can cause memory issues
3. **Too many threads**: Excessive context switching
4. **Too few threads**: Underutilization
5. **Ignoring rejection**: Silent task loss

## Key Interview Points

1. **Why use thread pools?**
    - Reuse threads (save creation cost)
    - Control concurrency
    - Manage lifecycle

2. **Core vs Maximum pool size**
    - Core: Minimum always present
    - Maximum: Can grow to this under load

3. **Queue before creating new threads**
    - Prevents unnecessary thread creation
    - Maintains stable pool size
    - Creates new threads only for sustained load

4. **Choosing pool sizes**
    - Consider CPU cores
    - Analyze task nature (CPU vs I/O)
    - Check memory constraints
    - Perform load testing

5. **Rejection handling**
    - Know all four policies
    - Implement custom if needed
    - Always handle rejections

## Summary Formula
```
Optimal Thread Pool Size = min(
    CPU_Cores × (1 + Wait/Compute_Ratio),
    Available_JVM_Memory / Memory_Per_Thread,
    Available_Heap × Safety_Factor / Memory_Per_Request
)
```

This comprehensive approach ensures:
- Efficient CPU utilization
- Memory safety
- Optimal performance
- Scalability under load