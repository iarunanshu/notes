# Java Multithreading - Part 9: Shutdown Methods & ScheduledThreadPoolExecutor

## Overview
This session covers important thread pool management topics:
- Differences between shutdown(), awaitTermination(), and shutdownNow()
- ScheduledThreadPoolExecutor for task scheduling

## 1. Thread Pool Shutdown Methods

### shutdown()
**Purpose**: Initiates orderly shutdown of the executor service

**Key Behaviors**:
1. **No new tasks accepted** - Rejects any new task submissions
2. **Existing tasks continue** - Already submitted tasks complete normally
3. **Non-blocking** - Calling thread continues immediately

```java
public class ShutdownExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Submit task
        executor.submit(() -> {
            try {
                Thread.sleep(5000);
                System.out.println("Task completed");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        // Shutdown executor
        executor.shutdown();
        System.out.println("Main thread continues after shutdown");
        
        // Try to submit new task - will throw exception
        try {
            executor.submit(() -> System.out.println("New task"));
        } catch (RejectedExecutionException e) {
            System.out.println("Task rejected - pool shut down");
        }
    }
}
```

**Output**:
```
Main thread continues after shutdown
Task rejected - pool shut down
Task completed (after 5 seconds)
```

### awaitTermination()
**Purpose**: Optional method to check if executor has terminated

**Key Behaviors**:
1. **Blocks calling thread** for specified timeout
2. **Returns boolean** - true if terminated, false if timeout
3. **Used after shutdown()** - Must call shutdown first
4. **Optional functionality** - Doesn't force termination

```java
public class AwaitTerminationExample {
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Submit long-running task
        executor.submit(() -> {
            try {
                Thread.sleep(5000);
                System.out.println("Task completed");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        // Shutdown executor
        executor.shutdown();
        
        // Wait for termination (max 2 seconds)
        boolean isTerminated = executor.awaitTermination(2, TimeUnit.SECONDS);
        System.out.println("Is terminated in 2 seconds? " + isTerminated);
        
        // Wait again for complete termination
        isTerminated = executor.awaitTermination(5, TimeUnit.SECONDS);
        System.out.println("Is terminated in 5 seconds? " + isTerminated);
    }
}
```

**Output**:
```
Is terminated in 2 seconds? false
Task completed
Is terminated in 5 seconds? true
```

### shutdownNow()
**Purpose**: Best effort attempt to stop all tasks immediately

**Key Behaviors**:
1. **Interrupts active tasks** - Tries to stop running tasks
2. **Halts waiting tasks** - Cancels tasks waiting for events
3. **Returns list** - Returns tasks awaiting execution
4. **Best effort** - May not successfully stop all tasks

```java
public class ShutdownNowExample {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        
        // Submit multiple tasks
        executor.submit(() -> {
            try {
                Thread.sleep(15000);  // Long-running task
                System.out.println("Task 1 completed");
            } catch (InterruptedException e) {
                System.out.println("Task 1 interrupted!");
            }
        });
        
        // Task in queue
        Future<?> future = executor.submit(() -> {
            System.out.println("Task 2 executed");
        });
        
        // Shutdown now - interrupt all
        List<Runnable> pendingTasks = executor.shutdownNow();
        System.out.println("Pending tasks: " + pendingTasks.size());
        System.out.println("Main completed");
    }
}
```

**Output**:
```
Task 1 interrupted!
Pending tasks: 1
Main completed
```

## 2. Comparison Table

| Method | Blocks Caller | Accepts New Tasks | Interrupts Active Tasks | Returns |
|--------|--------------|-------------------|------------------------|---------|
| shutdown() | No | No | No | void |
| awaitTermination() | Yes (timeout) | N/A | No | boolean |
| shutdownNow() | No | No | Yes | List<Runnable> |

## 3. Proper Shutdown Pattern

```java
public class ProperShutdownPattern {
    public static void shutdownExecutor(ExecutorService executor) {
        executor.shutdown(); // Disable new tasks
        
        try {
            // Wait for existing tasks to terminate
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                // Force shutdown if tasks don't complete
                executor.shutdownNow();
                
                // Wait again
                if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                    System.err.println("Executor did not terminate");
                }
            }
        } catch (InterruptedException e) {
            // Re-interrupt and force shutdown
            executor.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
```

## 4. ScheduledThreadPoolExecutor

### What is it?
- **Extends ThreadPoolExecutor**
- Schedules tasks to run after delay or periodically
- Supports both one-time and recurring executions

### Creation
```java
// Using Executors factory
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(5);

// Direct instantiation
ScheduledThreadPoolExecutor scheduler = new ScheduledThreadPoolExecutor(5);
```

### Key Methods

#### 1. schedule() - One-time Delayed Execution

**With Runnable (no return value)**:
```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(5);

// Schedule task to run after 5 seconds
scheduler.schedule(
    () -> System.out.println("Hello after 5 seconds"),
    5,
    TimeUnit.SECONDS
);
```

**With Callable (returns value)**:
```java
// Schedule callable that returns value
ScheduledFuture<String> future = scheduler.schedule(
    () -> {
        System.out.println("Computing...");
        return "Result after 3 seconds";
    },
    3,
    TimeUnit.SECONDS
);

// Get result (blocks until available)
String result = future.get();
System.out.println(result);
```

#### 2. scheduleAtFixedRate() - Periodic Execution

**Fixed Rate**: Next execution starts at fixed intervals regardless of previous task duration

```java
public class FixedRateExample {
    public static void main(String[] args) throws InterruptedException {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
        
        ScheduledFuture<?> future = scheduler.scheduleAtFixedRate(
            () -> {
                System.out.println("Task at: " + 
                    LocalTime.now().format(DateTimeFormatter.ISO_TIME));
                try {
                    Thread.sleep(2000); // Task takes 2 seconds
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            },
            1,  // Initial delay
            3,  // Period
            TimeUnit.SECONDS
        );
        
        // Cancel after 10 seconds
        Thread.sleep(10000);
        future.cancel(true);
        scheduler.shutdown();
    }
}
```

**Behavior when task takes longer than period**:
```java
// If task takes 6 seconds but period is 3 seconds
scheduler.scheduleAtFixedRate(
    () -> {
        System.out.println("Start: " + System.currentTimeMillis());
        Thread.sleep(6000);  // Takes 6 seconds
        System.out.println("End: " + System.currentTimeMillis());
    },
    1,  // Initial delay
    3,  // Period (shorter than task duration!)
    TimeUnit.SECONDS
);

// Output: Next task queues but waits for previous to complete
// Tasks cannot overlap - they queue up
```

#### 3. scheduleWithFixedDelay() - Fixed Delay Between Executions

**Fixed Delay**: Next execution starts after fixed delay from previous task completion

```java
public class FixedDelayExample {
    public static void main(String[] args) throws InterruptedException {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
        
        ScheduledFuture<?> future = scheduler.scheduleWithFixedDelay(
            () -> {
                System.out.println("Start: " + 
                    LocalTime.now().format(DateTimeFormatter.ISO_TIME));
                try {
                    Thread.sleep(2000); // Task takes 2 seconds
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("End: " + 
                    LocalTime.now().format(DateTimeFormatter.ISO_TIME));
            },
            1,  // Initial delay
            3,  // Delay after completion
            TimeUnit.SECONDS
        );
        
        // Cancel after 15 seconds
        Thread.sleep(15000);
        future.cancel(true);
        scheduler.shutdown();
    }
}
```

## 5. Fixed Rate vs Fixed Delay

### Visual Comparison

**Fixed Rate** (Period = 3s, Task Duration = 1s):
```
Time:  0---1---2---3---4---5---6---7---8---9
Task:  [T1]       [T2]       [T3]       [T4]
       Start at fixed intervals (every 3s)
```

**Fixed Delay** (Delay = 3s, Task Duration = 1s):
```
Time:  0---1---2---3---4---5---6---7---8---9---10
Task:  [T1]           [T2]           [T3]
       Start 3s after previous completion
```

### When Task Duration > Period

**Fixed Rate** (Period = 3s, Task Duration = 6s):
```
Time:  0---1---2---3---4---5---6---7---8---9---10---11---12
Task:  [-------T1-------][-------T2-------][-------T3-------]
       Next task queued, starts immediately after previous
```

**Fixed Delay** (Delay = 3s, Task Duration = 6s):
```
Time:  0---1---2---3---4---5---6---7---8---9---10---11---12---13---14---15
Task:  [-------T1-------]   wait   [-------T2-------]   wait   [-------T3--
       3s delay after each completion
```

## 6. Complete Example: Task Scheduler

```java
import java.time.LocalTime;
import java.time.format.DateTimeFormatter;
import java.util.concurrent.*;

public class TaskSchedulerDemo {
    
    public static void main(String[] args) throws Exception {
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(3);
        
        // 1. One-time delayed task
        System.out.println("Scheduling one-time task...");
        ScheduledFuture<String> oneTime = scheduler.schedule(
            () -> "One-time task completed at " + LocalTime.now(),
            2,
            TimeUnit.SECONDS
        );
        
        // 2. Fixed rate periodic task
        System.out.println("Starting fixed rate task...");
        ScheduledFuture<?> fixedRate = scheduler.scheduleAtFixedRate(
            () -> System.out.println("Fixed rate at: " + LocalTime.now()),
            0,  // No initial delay
            3,
            TimeUnit.SECONDS
        );
        
        // 3. Fixed delay periodic task
        System.out.println("Starting fixed delay task...");
        ScheduledFuture<?> fixedDelay = scheduler.scheduleWithFixedDelay(
            () -> {
                System.out.println("Fixed delay start: " + LocalTime.now());
                try {
                    Thread.sleep(1000); // Simulate work
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
                System.out.println("Fixed delay end: " + LocalTime.now());
            },
            1,
            2,
            TimeUnit.SECONDS
        );
        
        // Get one-time result
        System.out.println(oneTime.get());
        
        // Let tasks run for 10 seconds
        Thread.sleep(10000);
        
        // Cancel periodic tasks
        fixedRate.cancel(false);
        fixedDelay.cancel(false);
        
        // Proper shutdown
        scheduler.shutdown();
        if (!scheduler.awaitTermination(5, TimeUnit.SECONDS)) {
            scheduler.shutdownNow();
        }
        
        System.out.println("Scheduler terminated");
    }
}
```

## 7. Use Cases

### ScheduledThreadPoolExecutor Use Cases
1. **Health checks** - Periodic system monitoring
2. **Cache cleanup** - Regular cache eviction
3. **Report generation** - Daily/hourly reports
4. **Retry mechanisms** - Delayed retry after failure
5. **Rate limiting** - Control task execution rate

### Example: Retry with Exponential Backoff
```java
public class RetryWithBackoff {
    private static final int MAX_RETRIES = 5;
    
    public static void retryTask(Runnable task, 
                                 ScheduledExecutorService scheduler) {
        AtomicInteger attempts = new AtomicInteger(0);
        
        Runnable retryableTask = new Runnable() {
            @Override
            public void run() {
                try {
                    task.run();
                    System.out.println("Task succeeded");
                } catch (Exception e) {
                    int attempt = attempts.incrementAndGet();
                    if (attempt < MAX_RETRIES) {
                        long delay = (long) Math.pow(2, attempt); // Exponential
                        System.out.println("Retry " + attempt + 
                            " after " + delay + "s");
                        scheduler.schedule(this, delay, TimeUnit.SECONDS);
                    } else {
                        System.err.println("Max retries reached");
                    }
                }
            }
        };
        
        scheduler.execute(retryableTask);
    }
}
```

## Interview Key Points

1. **shutdown() vs shutdownNow()**
    - shutdown: Graceful, completes existing tasks
    - shutdownNow: Forceful, interrupts running tasks

2. **awaitTermination() purpose**
    - Optional check for termination
    - Doesn't force shutdown
    - Returns boolean status

3. **Fixed Rate vs Fixed Delay**
    - Rate: Period from start to start
    - Delay: Period from end to start

4. **Task Overlap**
    - Tasks never overlap in scheduled executor
    - If task takes longer, next execution queues

5. **Best Practices**
    - Always shutdown executors
    - Use try-finally for cleanup
    - Handle InterruptedException properly
    - Consider task duration when scheduling

## Summary
- **Shutdown methods** provide different levels of control
- **ScheduledThreadPoolExecutor** enables time-based task execution
- Choose appropriate scheduling method based on requirements
- Always implement proper shutdown patterns for resource cleanup