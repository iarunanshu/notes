# Java Multithreading - Part 8: Executors Utility Class & Fork-Join Pool

## Overview
This session covers the Executors utility class that provides factory methods for creating thread pools, and the Fork-Join framework for parallel divide-and-conquer algorithms.

## 1. Executors Utility Class

### What is Executors?
- **Utility class** in `java.util.concurrent` package
- Provides **factory methods** to create thread pools
- Alternative to manually creating ThreadPoolExecutor
- Simplifies common thread pool configurations

### Custom vs Factory Thread Pools
```java
// Custom ThreadPoolExecutor (what we've been doing)
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    corePoolSize,
    maximumPoolSize,
    keepAliveTime,
    unit,
    workQueue,
    threadFactory,
    handler
);

// Factory method (simpler for common cases)
ExecutorService executor = Executors.newFixedThreadPool(5);
```

## 2. Fixed Thread Pool

### Configuration
- **Core Pool Size**: n (fixed number)
- **Maximum Pool Size**: n (same as core)
- **Queue**: Unbounded (LinkedBlockingQueue)
- **Keep Alive**: Threads stay alive even when idle

### Creation & Usage
```java
// Create fixed thread pool with 5 threads
ExecutorService executor = Executors.newFixedThreadPool(5);

// Submit tasks
executor.submit(() -> {
    System.out.println("Task executed by: " + 
        Thread.currentThread().getName());
});

// Equivalent custom configuration
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    5,                              // corePoolSize
    5,                              // maximumPoolSize (same as core)
    0L,                             // keepAliveTime (not used)
    TimeUnit.MILLISECONDS,
    new LinkedBlockingQueue<>()     // Unbounded queue
);
```

### When to Use
✅ **Use when**:
- Exact number of async tasks is known
- Steady workload
- Want to limit concurrency to specific number

❌ **Don't use when**:
- Workload is highly variable
- Need elasticity for load spikes
- Tasks might block for long periods

### Disadvantages
- No elasticity - can't scale up during heavy load
- Limited concurrency even if system has capacity
- Unbounded queue can cause memory issues

## 3. Cached Thread Pool

### Configuration
- **Core Pool Size**: 0
- **Maximum Pool Size**: Integer.MAX_VALUE
- **Queue Size**: 0 (SynchronousQueue)
- **Keep Alive**: 60 seconds

### Creation & Usage
```java
// Create cached thread pool
ExecutorService executor = Executors.newCachedThreadPool();

// Submit tasks - creates new threads as needed
for (int i = 0; i < 100; i++) {
    executor.submit(() -> {
        // Short-lived task
        System.out.println("Quick task");
    });
}

// Equivalent custom configuration
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    0,                              // corePoolSize
    Integer.MAX_VALUE,              // maximumPoolSize
    60L,                            // keepAliveTime
    TimeUnit.SECONDS,
    new SynchronousQueue<>()        // No queuing
);
```

### How It Works
1. Initially zero threads
2. Creates new thread for each task
3. Reuses idle threads if available
4. Terminates idle threads after 60 seconds

### When to Use
✅ **Use for**:
- Burst of short-lived tasks
- Highly variable workload
- Tasks complete quickly

❌ **Avoid for**:
- Long-running tasks
- Predictable steady load
- Memory-constrained environments

### Disadvantages
- Can create unlimited threads → memory issues
- No control over thread creation
- Not suitable for long-running tasks

## 4. Single Thread Executor

### Configuration
- **Core Pool Size**: 1
- **Maximum Pool Size**: 1
- **Queue**: Unbounded
- **Keep Alive**: Thread always alive

### Creation & Usage
```java
// Create single thread executor
ExecutorService executor = Executors.newSingleThreadExecutor();

// Tasks execute sequentially
executor.submit(() -> System.out.println("Task 1"));
executor.submit(() -> System.out.println("Task 2"));
executor.submit(() -> System.out.println("Task 3"));
// Output: Task 1, Task 2, Task 3 (always in order)

// Equivalent custom configuration
ThreadPoolExecutor custom = new ThreadPoolExecutor(
    1,                              // corePoolSize
    1,                              // maximumPoolSize
    0L,                             // keepAliveTime
    TimeUnit.MILLISECONDS,
    new LinkedBlockingQueue<>()     // Unbounded queue
);
```

### When to Use
✅ **Use when**:
- Need sequential task processing
- Tasks must execute in order
- Want thread confinement

❌ **Disadvantages**:
- No concurrency at all
- Single point of failure
- Can become bottleneck

## 5. Work Stealing Pool & Fork-Join Framework

### What is Fork-Join?
- **Divide-and-conquer** parallel execution framework
- Split large task into smaller subtasks (Fork)
- Wait for subtasks to complete and combine results (Join)
- Enables work stealing between threads

### Work Stealing Pool Creation
```java
// Method 1: Using Executors
ExecutorService executor = Executors.newWorkStealingPool();
// Uses available processors count

ExecutorService executor = Executors.newWorkStealingPool(4);
// Specify parallelism level

// Method 2: Direct ForkJoinPool
ForkJoinPool pool = ForkJoinPool.commonPool();
// Or
ForkJoinPool pool = new ForkJoinPool(4);
```

## 6. How Work Stealing Works

### Traditional Thread Pool Model
```
Thread Pool: [Thread-1, Thread-2]
            ↓
Common Queue: [Task-1, Task-2, Task-3]

- All threads share single queue
- Threads take tasks from common queue
```

### Work Stealing Model
```
Thread-1 ← Work-Stealing Queue (Deque)
Thread-2 ← Work-Stealing Queue (Deque)
    ↓
Submission Queue (Common)

- Each thread has own deque
- Threads can steal from other's deques
```

### Work Stealing Flow

#### Step 1: Task Submission
```java
Task-1 → Thread-1 (picks from submission queue)
Task-2 → Thread-2 (picks from submission queue)
Task-3 → Submission Queue (threads busy)
```

#### Step 2: Task Division (Fork)
```java
// Task-2 divides into subtasks
Task-2 → SubTask-1 + SubTask-2

Thread-2 works on SubTask-1
SubTask-2 → Thread-2's work-stealing queue
```

#### Step 3: Work Stealing
```java
// Thread-1 completes Task-1
Thread-1 checks:
1. Own work-stealing queue → Empty
2. Submission queue → Pick Task-3
3. After Task-3 done, checks again:
   - Own queue → Empty
   - Submission queue → Empty
   - Other thread's queue → Steal SubTask-2!
```

### Priority Order for Thread
1. **First**: Check own work-stealing queue
2. **Second**: Check submission queue
3. **Third**: Steal from other thread's queue (from back)

## 7. RecursiveTask & RecursiveAction

### RecursiveTask (Returns Value)
```java
public class SumTask extends RecursiveTask<Integer> {
    private int start, end;
    private static final int THRESHOLD = 4;
    
    public SumTask(int start, int end) {
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected Integer compute() {
        // Base case: small enough to compute directly
        if (end - start <= THRESHOLD) {
            int sum = 0;
            for (int i = start; i <= end; i++) {
                sum += i;
            }
            return sum;
        }
        
        // Recursive case: divide task
        int mid = start + (end - start) / 2;
        
        // Create subtasks
        SumTask leftTask = new SumTask(start, mid);
        SumTask rightTask = new SumTask(mid + 1, end);
        
        // Fork left task (async execution)
        leftTask.fork();  // Goes to work-stealing queue
        
        // Compute right task in current thread
        int rightResult = rightTask.compute();
        
        // Join left task (wait for completion)
        int leftResult = leftTask.join();
        
        // Combine results
        return leftResult + rightResult;
    }
}

// Usage
ForkJoinPool pool = new ForkJoinPool();
SumTask task = new SumTask(1, 100);
Integer result = pool.invoke(task);  // Sum of 1 to 100
System.out.println("Sum: " + result);  // 5050
```

### RecursiveAction (No Return Value)
```java
public class PrintTask extends RecursiveAction {
    private int start, end;
    
    public PrintTask(int start, int end) {
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected void compute() {
        if (end - start <= 10) {
            // Direct computation
            for (int i = start; i <= end; i++) {
                System.out.println(Thread.currentThread().getName() + 
                    ": " + i);
            }
        } else {
            // Divide task
            int mid = start + (end - start) / 2;
            
            PrintTask left = new PrintTask(start, mid);
            PrintTask right = new PrintTask(mid + 1, end);
            
            // Fork both tasks
            invokeAll(left, right);  // Alternative to fork/join
        }
    }
}
```

## 8. Fork-Join Methods

### Key Methods
```java
// fork() - Async execution, adds to work-stealing queue
leftTask.fork();

// join() - Wait for completion and get result
Integer result = leftTask.join();

// invoke() - Execute and wait for result
Integer result = pool.invoke(task);

// invokeAll() - Fork all tasks and join all
invokeAll(task1, task2, task3);

// compute() - Execute in current thread
Integer result = task.compute();
```

## 9. Complete Example: Parallel Array Sum

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class ParallelArraySum {
    
    static class ArraySumTask extends RecursiveTask<Long> {
        private int[] array;
        private int start, end;
        private static final int THRESHOLD = 1000;
        
        public ArraySumTask(int[] array, int start, int end) {
            this.array = array;
            this.start = start;
            this.end = end;
        }
        
        @Override
        protected Long compute() {
            System.out.println(Thread.currentThread().getName() + 
                " processing: " + start + "-" + end);
            
            // Small enough to compute sequentially
            if (end - start <= THRESHOLD) {
                long sum = 0;
                for (int i = start; i < end; i++) {
                    sum += array[i];
                }
                return sum;
            }
            
            // Divide into subtasks
            int mid = start + (end - start) / 2;
            
            ArraySumTask leftTask = new ArraySumTask(array, start, mid);
            ArraySumTask rightTask = new ArraySumTask(array, mid, end);
            
            // Fork left, compute right
            leftTask.fork();
            Long rightSum = rightTask.compute();
            Long leftSum = leftTask.join();
            
            return leftSum + rightSum;
        }
    }
    
    public static void main(String[] args) {
        // Create large array
        int[] array = new int[10000];
        for (int i = 0; i < array.length; i++) {
            array[i] = i + 1;
        }
        
        // Sequential sum
        long startTime = System.currentTimeMillis();
        long seqSum = 0;
        for (int val : array) {
            seqSum += val;
        }
        long seqTime = System.currentTimeMillis() - startTime;
        
        // Parallel sum
        ForkJoinPool pool = new ForkJoinPool();
        ArraySumTask task = new ArraySumTask(array, 0, array.length);
        
        startTime = System.currentTimeMillis();
        Long parallelSum = pool.invoke(task);
        long parallelTime = System.currentTimeMillis() - startTime;
        
        System.out.println("Sequential Sum: " + seqSum + 
            " Time: " + seqTime + "ms");
        System.out.println("Parallel Sum: " + parallelSum + 
            " Time: " + parallelTime + "ms");
        
        pool.shutdown();
    }
}
```

## 10. Comparison Table

| Executor Type | Core | Max | Queue | Keep Alive | Use Case |
|--------------|------|-----|-------|------------|----------|
| Fixed | n | n | Unbounded | Always | Known workload |
| Cached | 0 | MAX_VALUE | 0 | 60s | Short bursts |
| Single | 1 | 1 | Unbounded | Always | Sequential tasks |
| Work Stealing | n | n | Per-thread deque | Always | Divide-conquer |

## Key Interview Points

### 1. When to Use Which Executor?
- **Fixed**: Predictable load, resource constraints
- **Cached**: Variable load, short tasks
- **Single**: Sequential processing requirement
- **ForkJoin**: Recursive divide-and-conquer problems

### 2. Work Stealing Benefits
- Better CPU utilization
- Automatic load balancing
- Reduced contention on single queue
- Efficient for recursive algorithms

### 3. Fork vs Join
- **Fork**: Async execution, adds to queue
- **Join**: Blocks until result available
- Always fork before join for parallelism

### 4. Queue Types
- **Submission Queue**: External tasks
- **Work-Stealing Queue**: Subtasks from fork()
- Stealing happens from back of deque

## Best Practices

1. **Choose Right Pool**
    - Don't use cached for long tasks
    - Use ForkJoin for recursive problems
    - Fixed for bounded resources

2. **Set Appropriate Thresholds**
   ```java
   // Too small = overhead > benefit
   // Too large = less parallelism
   THRESHOLD = array.length / (processors * 4);
   ```

3. **Avoid Blocking in ForkJoin**
    - Don't use I/O operations
    - Avoid synchronization
    - Keep tasks CPU-bound

4. **Handle Pool Shutdown**
   ```java
   pool.shutdown();
   if (!pool.awaitTermination(60, TimeUnit.SECONDS)) {
       pool.shutdownNow();
   }
   ```

## Summary
- **Executors** utility provides convenient factory methods
- Each pool type optimized for different scenarios
- **Fork-Join** excels at recursive divide-and-conquer
- **Work stealing** improves load balancing
- Choose pool based on task characteristics and system constraints