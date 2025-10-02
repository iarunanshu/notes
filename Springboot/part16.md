# Spring Boot Custom Interceptors

## What are Interceptors?

**Definition**: A mediator that gets invoked before or after your actual code execution.

**Prerequisites**:
1. Java Annotations (In-depth understanding)
2. Spring AOP (Aspect-Oriented Programming)

## Two Types of Custom Interceptors

### 1. Request Interceptor (Before Controller)
Intercepts requests before they reach the controller

### 2. Method Interceptor (After Controller)
Intercepts specific methods after controller is invoked

## Type 1: Request Interceptor (HandlerInterceptor)

### Request Flow with Interceptor

```
Client Request 
    ↓
Servlet Container
    ↓
DispatcherServlet
    ↓
[INTERCEPTOR - PreHandle]  ← Custom Logic Here
    ↓
Controller Method
    ↓
[INTERCEPTOR - PostHandle]  ← After Controller
    ↓
[INTERCEPTOR - AfterCompletion]  ← Always Runs
    ↓
Response
```

### Implementation Steps

#### Step 1: Create Custom Interceptor

```java
@Component
public class MyCustomInterceptor implements HandlerInterceptor {
    
    @Override
    public boolean preHandle(HttpServletRequest request, 
                           HttpServletResponse response, 
                           Object handler) throws Exception {
        System.out.println("Inside PreHandle");
        // Logic before controller
        // Return true to continue, false to stop
        return true;
    }
    
    @Override
    public void postHandle(HttpServletRequest request, 
                         HttpServletResponse response, 
                         Object handler, 
                         ModelAndView modelAndView) throws Exception {
        System.out.println("Inside PostHandle");
        // Logic after controller (success only)
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, 
                              HttpServletResponse response, 
                              Object handler, 
                              Exception ex) throws Exception {
        System.out.println("Inside AfterCompletion");
        // Always runs (like finally block)
        // Even if exception occurs
    }
}
```

#### Step 2: Register Interceptor

```java
@Configuration
public class AppConfig implements WebMvcConfigurer {
    
    @Autowired
    private MyCustomInterceptor myCustomInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(myCustomInterceptor)
                .addPathPatterns("/api/**")  // Apply to these paths
                .excludePathPatterns("/api/updateUser", "/api/deleteUser"); // Exclude these
    }
}
```

#### Step 3: Controller Example

```java
@RestController
public class UserController {
    
    @GetMapping("/api/getUser")
    public String getUser() {
        System.out.println("Controller: Doing something");
        return "User Data";
    }
}
```

### Execution Order

```
Output when hitting /api/getUser:
1. Inside PreHandle
2. Controller: Doing something
3. Inside PostHandle
4. Inside AfterCompletion
```

### Key Differences

| Method | When Executed | Exception Handling |
|--------|--------------|-------------------|
| preHandle | Before controller | Can stop execution |
| postHandle | After controller success | Skipped if exception |
| afterCompletion | Always after controller | Runs even with exception |

## Type 2: Method Interceptor (Using AOP)

### Step 1: Create Custom Annotation

#### Basic Annotation Creation

```java
@Target(ElementType.METHOD)  // Where can be used
@Retention(RetentionPolicy.RUNTIME)  // When available
public @interface MyCustomAnnotation {
    // Empty annotation
}
```

#### Annotation with Fields

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface MyCustomAnnotation {
    
    // Field with default value
    String name() default "";
    
    // Multiple field types
    int key() default 0;
    
    String value() default "defaultValue";
    
    Class<?> classType() default String.class;
    
    String[] tags() default {};
}
```

### Meta-Annotations Explained

#### @Target - Where annotation can be used

```java
@Target(ElementType.METHOD)  // Only on methods
@Target(ElementType.TYPE)    // Only on classes/interfaces
@Target({ElementType.METHOD, ElementType.FIELD})  // Multiple targets
```

**ElementType Options**:
- `TYPE` - Class, interface, enum
- `METHOD` - Methods
- `FIELD` - Fields
- `PARAMETER` - Method parameters
- `CONSTRUCTOR` - Constructors
- `LOCAL_VARIABLE` - Local variables
- `ANNOTATION_TYPE` - Other annotations
- `PACKAGE` - Packages

#### @Retention - When annotation is available

```java
@Retention(RetentionPolicy.SOURCE)   // Compile-time only
@Retention(RetentionPolicy.CLASS)    // In bytecode, not runtime
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
```

**RetentionPolicy Comparison**:

| Policy | In Source | In .class | At Runtime | Example |
|--------|-----------|-----------|------------|---------|
| SOURCE | Yes | No | No | @Override |
| CLASS | Yes | Yes | No | Some tools |
| RUNTIME | Yes | Yes | Yes | @Component |

### Step 2: Use Custom Annotation

```java
@Component
public class UserService {
    
    @MyCustomAnnotation(name = "user", key = 10)
    public String getUserDetails() {
        System.out.println("Getting user details");
        return "User Info";
    }
}
```

### Step 3: Create Method Interceptor

```java
@Component
@Aspect
public class MyCustomInterceptor {
    
    @Around("@annotation(com.example.MyCustomAnnotation)")
    public Object interceptMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        
        // Get method metadata
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Method method = signature.getMethod();
        
        // Check if annotation present
        if (method.isAnnotationPresent(MyCustomAnnotation.class)) {
            
            // Get annotation details
            MyCustomAnnotation annotation = method.getAnnotation(MyCustomAnnotation.class);
            String name = annotation.name();
            int key = annotation.key();
            
            System.out.println("Before method - Name: " + name + ", Key: " + key);
        }
        
        // Execute actual method
        Object result = joinPoint.proceed();
        
        System.out.println("After method execution");
        
        return result;
    }
}
```

## Complete Example: Authentication Interceptor

### Custom Annotation

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiresAuth {
    String[] roles() default {};
    boolean checkToken() default true;
}
```

### Service with Annotation

```java
@Service
public class UserService {
    
    @RequiresAuth(roles = {"ADMIN", "USER"})
    public void updateUser(String userId) {
        // Update logic
    }
    
    @RequiresAuth(roles = {"ADMIN"})
    public void deleteUser(String userId) {
        // Delete logic
    }
}
```

### Authentication Interceptor

```java
@Component
@Aspect
public class AuthenticationInterceptor {
    
    @Around("@annotation(requiresAuth)")
    public Object checkAuthentication(ProceedingJoinPoint joinPoint, 
                                     RequiresAuth requiresAuth) throws Throwable {
        
        // Check authentication
        if (requiresAuth.checkToken()) {
            // Validate token logic
            System.out.println("Validating token...");
        }
        
        // Check roles
        String[] requiredRoles = requiresAuth.roles();
        for (String role : requiredRoles) {
            System.out.println("Checking role: " + role);
            // Role validation logic
        }
        
        // If authorized, proceed
        return joinPoint.proceed();
    }
}
```

## Use Cases for Interceptors

### 1. Logging
```java
@Around("@annotation(Loggable)")
public Object logExecution(ProceedingJoinPoint joinPoint) throws Throwable {
    long start = System.currentTimeMillis();
    
    Object result = joinPoint.proceed();
    
    long executionTime = System.currentTimeMillis() - start;
    logger.info("{} executed in {} ms", joinPoint.getSignature(), executionTime);
    
    return result;
}
```

### 2. Caching
```java
@Around("@annotation(cacheable)")
public Object checkCache(ProceedingJoinPoint joinPoint, Cacheable cacheable) throws Throwable {
    String key = generateKey(joinPoint);
    
    // Check cache
    Object cached = cache.get(key);
    if (cached != null) {
        return cached;
    }
    
    // Execute and cache
    Object result = joinPoint.proceed();
    cache.put(key, result);
    return result;
}
```

### 3. Rate Limiting
```java
@Before("@annotation(rateLimited)")
public void checkRateLimit(RateLimited rateLimited) {
    String userId = getCurrentUserId();
    int limit = rateLimited.limit();
    
    if (exceedsLimit(userId, limit)) {
        throw new RateLimitException("Rate limit exceeded");
    }
}
```

## Best Practices

1. **Use HandlerInterceptor for HTTP-specific logic**
    - Authentication headers
    - Request/Response modification
    - Cross-cutting HTTP concerns

2. **Use AOP interceptors for business logic**
    - Method-level concerns
    - Transaction management
    - Caching

3. **Keep interceptors lightweight**
    - Avoid heavy processing
    - Use async for long operations

4. **Order matters**
   ```java
   @Order(1)  // Executes first
   public class SecurityInterceptor implements HandlerInterceptor {}
   
   @Order(2)  // Executes second
   public class LoggingInterceptor implements HandlerInterceptor {}
   ```

5. **Handle exceptions properly**
   ```java
   try {
       return joinPoint.proceed();
   } catch (Exception e) {
       // Log and handle
       throw e;
   }
   ```

## Interview Questions

### Q1: Difference between Filter and Interceptor?

| Aspect | Filter | Interceptor |
|--------|--------|-------------|
| Level | Servlet level | Spring MVC level |
| Configuration | web.xml or @WebFilter | Spring configuration |
| Access to | Request/Response | Handler, ModelAndView |
| Use case | Low-level filtering | Spring-specific logic |

### Q2: When to use HandlerInterceptor vs AOP?

**HandlerInterceptor**: HTTP request/response specific
**AOP**: Method-level, business logic, reusable across layers

### Q3: Can interceptors modify the response?

**HandlerInterceptor**: Yes, in postHandle and afterCompletion
**AOP**: Yes, can modify return value

## Key Takeaways

1. **Two types of interceptors**: Request-level (HandlerInterceptor) and Method-level (AOP)
2. **Custom annotations** need @Target and @Retention
3. **Execution order matters**: preHandle → Controller → postHandle → afterCompletion
   4.<br>**AOP interceptors** are more flexible for business logic
5. **Common use cases**: Authentication, Logging, Caching, Rate Limiting
6. **Always handle exceptions** in interceptors to prevent breaking the flow