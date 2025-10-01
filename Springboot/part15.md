# Spring Boot @Async Annotation - Part 2

## Conditions for @Async to Work Properly

### Two Essential Conditions

1. **Method must be in a different class** from where it's being called
2. **Method must be public**

### Why These Conditions?

**Root Cause**: Spring uses **AOP (Aspect-Oriented Programming)** for @Async
- AOP works through interception and proxy creation
- Interception only works on public methods
- Interception requires cross-class invocation

### Example - What NOT to Do

```java
@RestController
public class UserController {
    
    @GetMapping("/api/getUser")
    public String getUser() {
        System.out.println("Thread: " + Thread.currentThread().getName());
        test();  // Same class call - WON'T WORK!
        return "Processing...";
    }
    
    @Async  // This won't work as expected
    public void test() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}

// Output:
// Thread: http-nio-8080-exec-1
// Thread: http-nio-8080-exec-1 (SAME THREAD - Not async!)
```

### Correct Implementation

```java
@RestController
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/api/getUser")
    public String getUser() {
        userService.test();  // Different class - WORKS!
        return "Processing...";
    }
}

@Service
public class UserService {
    
    @Async  // Now this works correctly
    public void test() {
        System.out.println("Thread: " + Thread.currentThread().getName());
    }
}
```

## @Async with @Transactional

### Key Concept
**Transaction context does NOT transfer from caller thread to new thread**

### Use Case 1: @Transactional Calling @Async (AVOID)

```java
@Service
public class UserService {
    @Autowired
    private AccountService accountService;
    
    @Transactional
    public void updateUser() {
        // These run in transaction
        updateUserStatus();  
        updateFirstName();
        
        // This runs WITHOUT transaction context!
        accountService.updateBalance();  // @Async method
        
        // Problem: If exception occurs, balance update won't rollback
    }
}

@Service
public class AccountService {
    
    @Async
    public void updateBalance() {
        // Runs in new thread WITHOUT transaction
        // If fails, no rollback!
    }
}
```

**Problem**: Balance update runs outside transaction scope

### Use Case 2: @Async with @Transactional Together (USE WITH CAUTION)

```java
@Service
public class UserService {
    
    @Async
    @Transactional  // Both annotations
    public void updateUser() {
        updateUserStatus();
        updateFirstName();
        updateBalance();
    }
}
```

**Issues**:
- Creates new thread with its own transaction
- Parent transaction context NOT inherited
- Propagation levels won't work as expected
- Independent transaction, not related to parent

### Use Case 3: Proper Pattern (RECOMMENDED)

```java
@RestController
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/api/update")
    public String updateUser() {
        userService.updateUserAsync();  // Async call
        return "Processing...";  // Main thread free
    }
}

@Service
public class UserService {
    @Autowired
    private UserUtility userUtility;
    
    @Async  // First: Create new thread
    public void updateUserAsync() {
        userUtility.updateUserInTransaction();
    }
}

@Component
public class UserUtility {
    
    @Transactional  // Then: Run in transaction
    public void updateUserInTransaction() {
        // All operations in transaction
        updateUserStatus();
        updateFirstName();
        updateBalance();
        // Rollback works correctly if exception
    }
}
```

**Benefits**:
- Clean separation of concerns
- Main thread freed immediately
- Proper transaction management
- Rollback works as expected

## Async Method Return Types

### 1. Future (Deprecated but still used)

```java
@Service
public class UserService {
    
    @Async
    public Future<String> performAsyncTask() {
        // Simulate long-running task
        Thread.sleep(5000);
        String result = "Task completed";
        return new AsyncResult<>(result);
    }
}

@RestController
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/api/task")
    public String executeTask() throws Exception {
        // Start async task
        Future<String> future = userService.performAsyncTask();
        
        // Main thread can continue other work
        doOtherWork();
        
        // Block and wait for result when needed
        String result = future.get();  // Blocks here
        
        return result;
    }
}
```

### 2. CompletableFuture (Recommended)

```java
@Service
public class UserService {
    
    @Async
    public CompletableFuture<String> performAsyncTask() {
        // Simulate processing
        Thread.sleep(5000);
        String result = "Task completed";
        return CompletableFuture.completedFuture(result);
    }
}

@RestController
public class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/api/task")
    public String executeTask() throws Exception {
        // Start async task
        CompletableFuture<String> future = userService.performAsyncTask();
        
        // Continue other work
        doOtherWork();
        
        // Get result when needed
        String result = future.get();  // Blocks until complete
        
        return result;
    }
}
```

**Key Methods**:
- `get()` - Block and wait for result
- `get(timeout, unit)` - Wait with timeout
- `isDone()` - Check if completed
- `isCancelled()` - Check if cancelled

## Exception Handling in @Async Methods

### For Methods with Return Type

```java
@Service
public class UserService {
    
    @Async
    public CompletableFuture<String> performTask() {
        // If exception occurs here
        int result = 5 / 0;  // ArithmeticException
        return CompletableFuture.completedFuture("Done");
    }
}

@RestController
public class UserController {
    
    @GetMapping("/api/task")
    public String executeTask() {
        CompletableFuture<String> future = userService.performTask();
        
        try {
            // Exception thrown here when calling get()
            String result = future.get();
        } catch (Exception e) {
            // Handle exception
            log.error("Async task failed", e);
            return "Task failed";
        }
    }
}
```

### For Void Methods - Option 1: Try-Catch Inside Method

```java
@Service
public class UserService {
    
    @Async
    public void performTask() {
        try {
            // Your logic
            int result = 5 / 0;
        } catch (Exception e) {
            // Handle locally
            log.error("Error in async task", e);
        }
    }
}
```

### For Void Methods - Option 2: Global Handler (RECOMMENDED)

#### Step 1: Create Custom Exception Handler

```java
@Component
public class CustomAsyncExceptionHandler 
    implements AsyncUncaughtExceptionHandler {
    
    @Override
    public void handleUncaughtException(
            Throwable throwable, 
            Method method, 
            Object... params) {
        
        log.error("Exception in async method: " + method.getName(), throwable);
        
        // Custom handling logic
        // - Send alerts
        // - Store in database
        // - Notify monitoring system
        
        // Log parameters for debugging
        for (Object param : params) {
            log.error("Parameter value: " + param);
        }
    }
}
```

#### Step 2: Configure Exception Handler

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    @Autowired
    private CustomAsyncExceptionHandler exceptionHandler;
    
    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return exceptionHandler;
    }
    
    @Override
    public Executor getAsyncExecutor() {
        // Your custom executor
        return threadPoolTaskExecutor();
    }
}
```

#### Step 3: Use in Async Methods

```java
@Service
public class UserService {
    
    @Async
    public void performTask() {
        // No try-catch needed
        int result = 5 / 0;  // Exception handled globally
    }
}
```

### Default Spring Behavior

If no custom handler configured, Spring uses `SimpleAsyncUncaughtExceptionHandler`:
```java
public class SimpleAsyncUncaughtExceptionHandler 
    implements AsyncUncaughtExceptionHandler {
    
    @Override
    public void handleUncaughtException(Throwable ex, Method method, Object... params) {
        log.error("Unexpected exception occurred invoking async method: " + method, ex);
    }
}
```

## Interview Questions & Answers

### Q1: Why doesn't @Async work when calling method in same class?

**Answer**:
- Spring uses AOP proxies for @Async
- Proxies only intercept external calls (cross-class)
- Internal calls bypass the proxy
- Solution: Move async method to different class

### Q2: Can @Transactional and @Async be used together?

**Answer**:
- Yes, but with caution
- Transaction context doesn't transfer to new thread
- Creates independent transaction
- Better to separate: @Async first, then call @Transactional method

### Q3: How to handle exceptions in void async methods?

**Answer**: Two approaches:
1. Local try-catch in each method
2. Global handler via AsyncUncaughtExceptionHandler (recommended)
    - Centralized error handling
    - Consistent logging
    - Better for monitoring

### Q4: Future vs CompletableFuture?

**Answer**:
- Future is older, limited functionality
- CompletableFuture (Java 8+) recommended:
    - Supports chaining
    - Better exception handling
    - Non-blocking operations
    - More flexible

### Q5: What happens to transaction when calling @Async from @Transactional?

**Answer**:
- Async method runs outside transaction scope
- No automatic rollback for async method
- Parent transaction doesn't wait for async
- Can cause data inconsistency

## Best Practices Summary

1. **Always use @Async on public methods in separate classes**
2. **Avoid @Transactional with @Async on same method**
3. **Use CompletableFuture over Future**
4. **Implement global exception handler for void methods**
5. **Separate async and transactional concerns**
6. **Configure custom thread pool executor**
7. **Monitor async method execution**
8. **Log thread names for debugging**

## Common Pitfalls to Avoid

1. **Self-invocation** - Calling @Async method from same class
2. **Transaction context loss** - Expecting parent transaction in async
3. **Unhandled exceptions** - Not catching exceptions in void methods
4. **Resource exhaustion** - Not configuring thread pool limits
5. **Blocking unnecessarily** - Calling get() too early on Future

## Key Takeaways

1. **@Async requires proxy** - Must be cross-class, public method
2. **Transactions are thread-local** - Don't transfer to async threads
3. **Exception handling is critical** - Especially for void methods
4. **Return types matter** - Use CompletableFuture for flexibility
5. **Separation pattern works best** - @Async → regular method → @Transactional