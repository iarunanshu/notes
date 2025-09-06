# Java Multithreading - Part 7: Future, Callable & CompletableFuture

## Overview
This session covers three critical concepts for asynchronous programming in Java:
- **Future**: Track status of async tasks
- **Callable**: Runnable that returns values
- **CompletableFuture**: Advanced async programming with chaining

## 1. Future Interface

### The Problem
```java
// Without Future - No way to track task status
ThreadPoolExecutor executor = new ThreadPoolExecutor(...);
executor.submit(() -> {
    // Some async task
    System.out.println("Doing something");
});
// Main thread continues but can't check task status
```

### What is Future?
- **Interface** representing result of async computation
- Allows checking if computation is complete
- Can retrieve result or handle exceptions
- Provides reference to track async task status

### Future Methods

```java
Future<?> future = executor.submit(task);

// 1. cancel() - Attempt to cancel execution
boolean cancelled = future.cancel(true);  // true = interrupt if running

// 2. isCancelled() - Check if task was cancelled
boolean isCancelled = future.isCancelled();

// 3. isDone() - Check if task completed (normal/exception/cancelled)
boolean isDone = future.isDone();

// 4. get() - Block and wait for result
Object result = future.get();  // Blocks indefinitely

// 5. get(timeout) - Wait with timeout
try {
    Object result = future.get(3, TimeUnit.SECONDS);
} catch (TimeoutException e) {
    // Task didn't complete in 3 seconds
}
```

### Complete Future Example
```java
public class FutureExample {
    public static void main(String[] args) throws Exception {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            1, 1, 0L, TimeUnit.MILLISECONDS,
            new ArrayBlockingQueue<>(10)
        );
        
        // Submit task and get Future
        Future<?> future = executor.submit(() -> {
            try {
                Thread.sleep(7000);  // Simulate work
                System.out.println("Task completed by " + 
                    Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        // Check status immediately
        System.out.println("Is done? " + future.isDone());  // false
        
        // Try to get with timeout
        try {
            future.get(2, TimeUnit.SECONDS);
        } catch (TimeoutException e) {
            System.out.println("Timeout after 2 seconds");
        }
        
        // Block until complete
        future.get();  // Waits ~5 more seconds
        
        // Check final status
        System.out.println("Is done? " + future.isDone());  // true
        System.out.println("Is cancelled? " + future.isCancelled());  // false
        
        executor.shutdown();
    }
}
```

## 2. Three Flavors of submit()

### 1. submit(Runnable) - Returns Future<?>
```java
// No return value, Future<?> with null result
Future<?> future = executor.submit(() -> {
    System.out.println("Do something");
});

Object result = future.get();  // Always null for Runnable
```

### 2. submit(Runnable, T) - Returns Future<T>
```java
// Workaround to get result from Runnable
List<Integer> output = new ArrayList<>();

Future<List<Integer>> future = executor.submit(
    new MyRunnable(output),  // Pass shared object
    output                    // Return same object after completion
);

// Custom Runnable modifying shared object
class MyRunnable implements Runnable {
    private List<Integer> list;
    
    MyRunnable(List<Integer> list) {
        this.list = list;
    }
    
    public void run() {
        list.add(300);  // Modify shared object
    }
}

List<Integer> result = future.get();  // Returns modified list
System.out.println(result.get(0));     // Prints 300
```

### 3. submit(Callable<T>) - Returns Future<T>
```java
// Cleaner approach with Callable
Future<List<Integer>> future = executor.submit(() -> {
    List<Integer> output = new ArrayList<>();
    output.add(300);
    return output;  // Callable can return value
});

List<Integer> result = future.get();
System.out.println(result.get(0));  // Prints 300
```

## 3. Callable vs Runnable

| Aspect | Runnable | Callable |
|--------|----------|----------|
| Return Type | void (no return) | Generic type T |
| Method | run() | call() |
| Exception | Can't throw checked exceptions | Can throw exceptions |
| Use Case | Fire-and-forget tasks | Tasks that compute results |

```java
// Runnable - No return
Runnable task = () -> System.out.println("Hello");

// Callable - Returns value
Callable<String> task = () -> {
    return "Hello World";
};
```

## 4. CompletableFuture (Java 8+)

### What is CompletableFuture?
- **Advanced version of Future**
- Implements Future interface
- Provides **chaining** capabilities
- Better async composition
- Non-blocking operations

### Key Methods

#### 1. supplyAsync() - Initiate Async Operation
```java
// Using default ForkJoinPool
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("Thread: " + Thread.currentThread().getName());
    return "Hello";
});

// Using custom executor
ThreadPoolExecutor executor = new ThreadPoolExecutor(...);
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Hello";
}, executor);

String result = future.get();  // "Hello"
```

#### 2. thenApply() vs thenApplyAsync()
```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        System.out.println("Supply: " + Thread.currentThread().getName());
        return "Concept &";
    }, executor)
    .thenApply(value -> {
        // SAME thread as supplyAsync
        System.out.println("Apply: " + Thread.currentThread().getName());
        return value + " Coding";
    })
    .thenApplyAsync(value -> {
        // DIFFERENT thread (new or from pool)
        System.out.println("ApplyAsync: " + Thread.currentThread().getName());
        return value + " Done";
    }, executor);

String result = future.get();  // "Concept & Coding Done"
```

#### 3. thenCompose() - Chain Dependent Operations
```java
// Maintains ordering for dependent async operations
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenCompose(value -> 
        CompletableFuture.supplyAsync(() -> value + " World")
    )
    .thenCompose(value -> 
        CompletableFuture.supplyAsync(() -> value + " All")
    );

String result = future.get();  // Always "Hello World All" (ordered)
```

#### 4. thenAccept() - Terminal Operation
```java
// Consumes value but returns CompletableFuture<Void>
CompletableFuture<Void> future = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenAccept(value -> {
        System.out.println("Received: " + value);
        // No return, typically end of chain
    });

future.get();  // Returns null (Void)
```

#### 5. thenCombine() - Combine Two Futures
```java
// Combine results from two independent async operations
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(() -> 10);
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "K");

CompletableFuture<String> combined = future1.thenCombine(
    future2,
    (intValue, stringValue) -> intValue + stringValue  // BiFunction
);

String result = combined.get();  // "10K"
```

## Complete Example: Chaining Operations

```java
public class CompletableFutureChaining {
    public static void main(String[] args) throws Exception {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(
            2, 4, 10, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(2)
        );
        
        CompletableFuture<String> future = CompletableFuture
            // Step 1: Initial async operation
            .supplyAsync(() -> {
                System.out.println("Step 1: " + Thread.currentThread().getName());
                try { Thread.sleep(1000); } catch (Exception e) {}
                return "Data";
            }, executor)
            
            // Step 2: Transform result (same thread)
            .thenApply(data -> {
                System.out.println("Step 2: " + Thread.currentThread().getName());
                return data + " Processed";
            })
            
            // Step 3: Chain another async operation (new thread)
            .thenComposeAsync(data -> 
                CompletableFuture.supplyAsync(() -> {
                    System.out.println("Step 3: " + Thread.currentThread().getName());
                    return data + " Further";
                }, executor)
            )
            
            // Step 4: Final transformation
            .thenApplyAsync(data -> {
                System.out.println("Step 4: " + Thread.currentThread().getName());
                return data + " Complete";
            }, executor);
        
        // Get final result
        String result = future.get();
        System.out.println("Final: " + result);  // "Data Processed Further Complete"
        
        executor.shutdown();
    }
}
```

## Method Comparison Table

| Method | Sync/Async | Returns | Purpose |
|--------|-----------|---------|---------|
| supplyAsync | Async | CompletableFuture<T> | Start async operation |
| thenApply | Sync (same thread) | CompletableFuture<T> | Transform result |
| thenApplyAsync | Async (new thread) | CompletableFuture<T> | Transform result async |
| thenCompose | Sync | CompletableFuture<T> | Chain dependent operations |
| thenComposeAsync | Async | CompletableFuture<T> | Chain dependent operations async |
| thenAccept | Sync | CompletableFuture<Void> | Consume result (terminal) |
| thenAcceptAsync | Async | CompletableFuture<Void> | Consume result async |
| thenCombine | Sync | CompletableFuture<T> | Combine two futures |
| thenCombineAsync | Async | CompletableFuture<T> | Combine two futures async |

## Best Practices

### 1. Choose Right Approach
```java
// Simple async task - Use Future
Future<String> simple = executor.submit(callable);

// Complex chaining - Use CompletableFuture
CompletableFuture<String> complex = CompletableFuture
    .supplyAsync(supplier)
    .thenApply(transformer)
    .thenCompose(asyncMapper);
```

### 2. Handle Exceptions
```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        if (Math.random() > 0.5) {
            throw new RuntimeException("Error!");
        }
        return "Success";
    })
    .exceptionally(ex -> {
        System.err.println("Error: " + ex.getMessage());
        return "Default Value";
    });
```

### 3. Use Custom Executors
```java
// Control thread pool size
ThreadPoolExecutor customExecutor = new ThreadPoolExecutor(...);
CompletableFuture.supplyAsync(task, customExecutor);
```

### 4. Avoid Blocking Operations
```java
// Bad - Blocks thread
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return someBlockingOperation();  // Avoid
});

// Good - Use async variants
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> startOperation())
    .thenComposeAsync(result -> continueAsync(result));
```

## Interview Key Points

1. **Future vs CompletableFuture**
    - Future: Basic async result tracking
    - CompletableFuture: Advanced with chaining, composition

2. **Runnable vs Callable**
    - Runnable: No return value
    - Callable: Returns value, can throw exceptions

3. **Sync vs Async Methods**
    - Sync (thenApply): Same thread
    - Async (thenApplyAsync): Different thread

4. **Chaining Benefits**
    - Compose complex async workflows
    - Non-blocking operations
    - Better error handling

5. **When to Use What**
    - Simple async: Future + Callable
    - Complex workflows: CompletableFuture
    - Fire-and-forget: Runnable

## Common Pitfalls

1. **Forgetting get() blocks**
```java
future.get();  // This blocks! Use carefully
```

2. **Not handling timeouts**
```java
// Always use timeout variant in production
future.get(5, TimeUnit.SECONDS);
```

3. **Ignoring exceptions**
```java
// Always handle potential exceptions
future.exceptionally(ex -> defaultValue);
```

4. **Mixing thread pools**
```java
// Be consistent with executors
CompletableFuture.supplyAsync(task, executor1)
    .thenApplyAsync(transform, executor2);  // Different pools!
```

## Summary

- **Future**: Track async task status, get results
- **Callable**: Alternative to Runnable with return values
- **CompletableFuture**: Modern async programming with:
    - Chaining operations
    - Combining results
    - Non-blocking composition
    - Better exception handling

**Important**: Practice these concepts with hands-on coding. The chaining features of CompletableFuture are complex but powerful for building async workflows.