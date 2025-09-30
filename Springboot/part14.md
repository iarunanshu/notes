# Spring Boot @Async Annotation - Part 1

## Prerequisites
1. **Thread Creation and Lifecycle** (Java)
2. **Thread Pool and ThreadPoolExecutor** (Java)
3. **Future, Callable, and CompletableFuture** (Java)

## Thread Pool Basics

### What is a Thread Pool?
- Collection of pre-created threads available to perform submitted tasks
- Threads are reusable - return to pool after task completion
- Prevents overhead of creating/destroying threads repeatedly

### Thread Pool Parameters
1. **Core Pool Size** (Minimum pool size) - Minimum threads to maintain
2. **Maximum Pool Size** - Maximum threads that can be created
3. **Queue Size** - Number of tasks that can wait in queue

### Thread Pool Execution Flow

```
Initial State: Core Pool Size = 2, Max Pool Size = 4, Queue Size = 3

Task 1 arrives → Thread 1 picks it (Pool: 1 busy, 1 free)
Task 2 arrives → Thread 2 picks it (Pool: 2 busy, 0 free)
Task 3 arrives → Goes to queue (Queue: 1/3 filled)
Task 4 arrives → Goes to queue (Queue: 2/3 filled)
Task 5 arrives → Goes to queue (Queue: 3/3 filled)
Task 6 arrives → Queue full → Create Thread 3 (if max not reached)
Task 7 arrives → Create Thread 4 (Pool: 4/4, max reached)
Task 8 arrives → REJECTED (Pool full, Queue full)
```

### Creating ThreadPoolExecutor in Java
```java
// Plain Java approach
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    2,     // Core pool size
    4,     // Maximum pool size  
    60L,   // Keep alive time
    TimeUnit.SECONDS,
    new LinkedBlockingQueue<>(3)  // Queue size
);
```

## @Async Annotation

### Purpose
- Marks methods to run asynchronously in separate threads
- Main thread continues without blocking
- Improves application responsiveness

### Basic Setup

#### 1. Enable Async Processing
```java
@SpringBootApplication
@EnableAsync  // Required to activate async functionality
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

#### 2. Mark Method as Async
```java
@Service
public class UserService {
    
    @Async
    public void asyncMethod() {
        System.out.println("Inside async method: " + 
            Thread.currentThread().getName());
        // Method runs in separate thread
    }
}
```

#### 3. Controller Example
```java
@RestController
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/api/getUser")
    public String getUser() {
        System.out.println("Inside getUser: " + 
            Thread.currentThread().getName());  // Main thread
        
        userService.asyncMethod();  // Runs in new thread
        
        return "Processing...";
    }
}
```

### Output Example
```
Inside getUser: http-nio-8080-exec-1 (Main thread)
Inside async method: task-1 (New thread)
Inside getUser: http-nio-8080-exec-2
Inside async method: task-2
...
Inside async method: task-8
Inside async method: task-1 (Reuses after 8)
```

## How @Async Creates Threads

### Common Misconception
"Spring Boot uses SimpleAsyncTaskExecutor by default which creates new threads blindly"
- **This is not fully correct!**

### Actual Internal Logic
```java
// Simplified Spring internal code
public class AsyncExecutionInterceptor {
    
    protected void execute() {
        Executor executor = getDefaultExecutor();
        
        if (executor != null) {
            // Use found executor
            executor.execute(task);
        } else {
            // Only if no executor found
            SimpleAsyncTaskExecutor simple = new SimpleAsyncTaskExecutor();
            simple.execute(task);
        }
    }
}
```

## Use Cases and Executor Selection

### Use Case 1: No Custom Configuration

```java
@Configuration
public class AppConfig {
    // Empty - no custom executor defined
}

@Service
public class UserService {
    @Async
    public void asyncMethod() {
        // Implementation
    }
}
```

**What happens:**
1. Spring Boot checks for ThreadPoolTaskExecutor bean → Not found
2. Creates default ThreadPoolTaskExecutor with:
    - Core Pool Size: 8
    - Max Pool Size: Integer.MAX_VALUE
    - Queue Capacity: Integer.MAX_VALUE

**Problems with Default:**
1. **Under-utilization** - Queue too large, max threads rarely created
2. **High Latency** - Tasks wait in huge queue
3. **Thread Exhaustion Risk** - No practical limit
4. **High Memory Usage** - Too many threads possible

### Use Case 2: Custom ThreadPoolTaskExecutor (Recommended)

```java
@Configuration
public class AppConfig {
    
    @Bean(name = "customExecutor")
    public ThreadPoolTaskExecutor threadPoolTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(4);
        executor.setQueueCapacity(3);
        executor.setThreadNamePrefix("MyThread-");
        executor.initialize();
        return executor;
    }
}
```

**Usage Options:**
```java
// Option 1: Let Spring auto-detect (if only one bean)
@Async
public void method1() { }

// Option 2: Specify executor explicitly
@Async("customExecutor")
public void method2() { }
```

**What happens:**
- Spring finds ThreadPoolTaskExecutor bean
- Uses it as default for all @Async methods
- Controlled thread creation based on your settings

### Use Case 3: Plain Java ThreadPoolExecutor

```java
@Configuration
public class AppConfig {
    
    @Bean(name = "javaExecutor")
    public Executor threadPoolExecutor() {
        return new ThreadPoolExecutor(
            2, 4, 60L, TimeUnit.SECONDS,
            new LinkedBlockingQueue<>(3),
            new ThreadFactory() {
                private int counter = 0;
                public Thread newThread(Runnable r) {
                    return new Thread(r, "MyThread-" + counter++);
                }
            }
        );
    }
}
```

**Important:**
```java
// Must specify executor name explicitly!
@Async("javaExecutor")  
public void asyncMethod() {
    // Without name, SimpleAsyncTaskExecutor will be used
}
```

**What happens:**
- Spring doesn't recognize plain Java executor as default
- Falls back to SimpleAsyncTaskExecutor if name not specified
- Must explicitly reference the executor

## Industry Standard Approach

### Implementing AsyncConfigurer

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    private ThreadPoolTaskExecutor executor;
    
    @Override
    public synchronized Executor getAsyncExecutor() {
        if (executor == null) {
            executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(2);
            executor.setMaxPoolSize(4);
            executor.setQueueCapacity(3);
            executor.setThreadNamePrefix("AppAsync-");
            executor.initialize();
        }
        return executor;
    }
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return new SimpleAsyncUncaughtExceptionHandler();
    }
}
```

**Benefits:**
1. Always uses your custom executor
2. No need to specify executor name in @Async
3. Consistent behavior across application
4. Centralized configuration

**Important Note:**
- Method is not a bean, so handle singleton pattern manually
- Use synchronized to prevent multiple executor instances

## ThreadPoolTaskExecutor vs ThreadPoolExecutor

| Aspect | ThreadPoolTaskExecutor | ThreadPoolExecutor |
|--------|------------------------|-------------------|
| Type | Spring wrapper | Plain Java |
| Auto-detection | Yes | No |
| Configuration | Spring-style setters | Constructor parameters |
| @Async compatibility | Works without name | Requires explicit name |
| Bean management | Spring managed | Manual |

## Best Practices

1. **Never rely on defaults** - Always configure your own executor
2. **Size appropriately** - Based on CPU cores and task nature
3. **Monitor queue size** - Prevent unbounded growth
4. **Use ThreadPoolTaskExecutor** - Better Spring integration
5. **Implement AsyncConfigurer** - For consistent behavior
6. **Handle rejection** - Define RejectedExecutionHandler
7. **Name your threads** - Easier debugging and monitoring

## Common Interview Questions

### Q1: What executor does @Async use by default?
**Answer:** It's not SimpleAsyncTaskExecutor! Spring first looks for:
1. ThreadPoolTaskExecutor bean (uses if found)
2. Creates default ThreadPoolTaskExecutor (8 core, unlimited max)
3. Only uses SimpleAsyncTaskExecutor if no executor found

### Q2: Why shouldn't we use default configuration?
**Answer:** Default has issues:
- Under-utilization (queue too large)
- High latency (tasks wait in queue)
- Risk of resource exhaustion
- Poor performance under load

### Q3: ThreadPoolTaskExecutor vs ThreadPoolExecutor?
**Answer:**
- ThreadPoolTaskExecutor is Spring's wrapper
- Auto-detected by Spring as default
- ThreadPoolExecutor requires explicit naming in @Async

## Testing Async Behavior

```java
@Service
public class TestService {
    
    @Async
    public void testAsync() throws InterruptedException {
        System.out.println("Start: " + Thread.currentThread().getName());
        Thread.sleep(5000);  // Simulate work
        System.out.println("End: " + Thread.currentThread().getName());
    }
}

// Test multiple calls to observe thread reuse and queue behavior
```

## What's Coming in Part 2

1. **Exception Handling** in async methods
2. **Return Types** (Future, CompletableFuture)
3. **Transaction Management** with @Async
4. **Testing async methods**
5. **Advanced interview questions**

## Key Takeaways

1. **@EnableAsync is mandatory** for async functionality
2. **Default behavior is complex** - not just SimpleAsyncTaskExecutor
3. **Always configure custom executor** for production
4. **ThreadPoolTaskExecutor preferred** over plain ThreadPoolExecutor
5. **AsyncConfigurer provides cleanest solution**
6. **Monitor thread pool metrics** in production