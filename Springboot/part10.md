# Spring Boot AOP (Aspect Oriented Programming) - Complete Notes

## What is AOP?

**Definition**: AOP helps intercept method invocations and perform tasks before/after the method execution

**Core Purpose**:
- Focus on **business logic** by separating **cross-cutting concerns**
- Handle **boilerplate and repetitive code** like:
    - Logging
    - Transaction management
    - Security
    - Performance monitoring
    - Exception handling

## Why AOP?

### Without AOP
```java
public void businessMethod() {
    // Logging
    logger.info("Method started");
    
    // Transaction start
    transaction.begin();
    
    // ACTUAL BUSINESS LOGIC
    doBusinessWork();
    
    // Transaction end
    transaction.commit();
    
    // Logging
    logger.info("Method ended");
}
```

### With AOP
```java
public void businessMethod() {
    // ONLY BUSINESS LOGIC
    doBusinessWork();
}
// Logging, transactions handled by AOP separately
```

## Key Terminology

### 1. **Aspect**
- Module that handles repetitive/boilerplate code
- Class annotated with `@Aspect`
- Contains pointcuts and advice

### 2. **Pointcut**
- Expression that tells WHERE advice should be applied
- Identifies which methods need interception

### 3. **Advice**
- Action taken before/after/around method execution
- The actual code that runs during interception

### 4. **Join Point**
- Point where actual method invocation happens
- The method being intercepted

## Setup

### Maven Dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## Basic Example

### Business Class
```java
@RestController
public class EmployeeController {
    
    @GetMapping("/api/v1/fetchEmployee")
    public String fetchEmployee() {
        return "Item Fetched";  // Business logic
    }
}
```

### Aspect Class
```java
@Aspect
@Component
public class LoggingAspect {
    
    @Before("execution(* com.conceptandcoding.EmployeeController.fetchEmployee())")
    public void beforeMethodAspect() {
        System.out.println("Inside before method aspect");
    }
}
```

**Flow**:
1. API called: `/api/v1/fetchEmployee`
2. AOP intercepts before method execution
3. Prints: "Inside before method aspect"
4. Executes actual method
5. Returns: "Item Fetched"

## Types of Pointcuts

### 1. **execution** - Matches specific method in specific class

**Syntax**:
```
execution([access-modifier] return-type package.Class.method(args))
```

**Examples**:

```java
// Specific method, no args
@Before("execution(public String com.example.EmployeeController.fetchEmployee())")

// Any return type (using *)
@Before("execution(* com.example.EmployeeController.fetchEmployee())")

// Any method in class
@Before("execution(* com.example.EmployeeController.*())")

// Method with specific arg
@Before("execution(* com.example.EmployeeController.updateEmployee(String))")

// Any single argument (using *)
@Before("execution(* com.example.EmployeeController.updateEmployee(*))")

// Zero or more arguments (using ..)
@Before("execution(* com.example.EmployeeController.updateEmployee(..))")

// Any method in package and subpackages
@Before("execution(* com.conceptandcoding..*.*(..))")
```

### Wildcards Explained

#### **`*` (Asterisk)** - Matches any single item
- Return type: `*` = any type
- Method name: `*` = any method
- Single argument: `*` = any type but exactly one

#### **`..` (Dot-Dot)** - Matches zero or more items
- In package: `com.example..` = package and all subpackages
- In arguments: `(..)` = zero or more arguments

### 2. **within** - Matches all methods within class/package

```java
// All methods in specific class
@Before("within(com.example.EmployeeController)")

// All methods in package and subpackages
@Before("within(com.example..*)")
```

### 3. **@within** - Matches methods in classes with specific annotation

```java
// All methods in classes annotated with @Service
@Before("@within(org.springframework.stereotype.Service)")
```

**Example**:
```java
@Service  // This annotation matches @within
public class EmployeeUtil {
    public String helperMethod() {  // This method will be intercepted
        return "Helper";
    }
}
```

### 4. **@annotation** - Matches methods with specific annotation

```java
// Any method annotated with @GetMapping
@Before("@annotation(org.springframework.web.bind.annotation.GetMapping)")
```

**Example**:
```java
@GetMapping("/fetch")  // This annotation matches
public String fetchData() {  // This method will be intercepted
    return "Data";
}
```

### 5. **args** - Matches methods with specific arguments

```java
// Method with String and int arguments
@Before("args(String, int)")

// Method with custom object
@Before("args(com.example.Employee)")
```

### 6. **@args** - Matches methods where argument class has annotation

```java
// Method where argument's class is annotated with @Entity
@Before("@args(javax.persistence.Entity)")
```

**Example**:
```java
@Entity  // This annotation on class
public class Employee { }

public void processEmployee(Employee emp) {  // Matches because Employee has @Entity
    // Method intercepted
}
```

### 7. **target** - Matches methods on specific instance

```java
// Any method on EmployeeUtil instance
@Before("target(com.example.EmployeeUtil)")

// Using interface - matches all implementations
@Before("target(com.example.EmployeeInterface)")
```

## Combining Pointcuts

### Using AND (`&&`)
```java
@Before("execution(* com.example.EmployeeController.*(..)) && @within(org.springframework.stereotype.RestController)")
```

### Using OR (`||`)
```java
@Before("@within(org.springframework.stereotype.Service) || @within(org.springframework.stereotype.Component)")
```

## Named Pointcuts

Define reusable pointcut expressions:

```java
@Aspect
@Component
public class LoggingAspect {
    
    // Define named pointcut
    @Pointcut("execution(* com.example.EmployeeController.*(..))")
    public void controllerMethods() {}
    
    // Use named pointcut
    @Before("controllerMethods()")
    public void logBefore() {
        System.out.println("Logging before controller method");
    }
    
    @After("controllerMethods()")
    public void logAfter() {
        System.out.println("Logging after controller method");
    }
}
```

## Types of Advice

### 1. **@Before** - Runs before method
```java
@Before("execution(* com.example.*.*(..))")
public void beforeAdvice() {
    System.out.println("Before method execution");
}
```

### 2. **@After** - Runs after method (regardless of outcome)
```java
@After("execution(* com.example.*.*(..))")
public void afterAdvice() {
    System.out.println("After method execution");
}
```

### 3. **@AfterReturning** - Runs after successful execution
```java
@AfterReturning(
    pointcut = "execution(* com.example.*.*(..))",
    returning = "result"
)
public void afterReturningAdvice(Object result) {
    System.out.println("Method returned: " + result);
}
```

### 4. **@AfterThrowing** - Runs after exception
```java
@AfterThrowing(
    pointcut = "execution(* com.example.*.*(..))",
    throwing = "exception"
)
public void afterThrowingAdvice(Exception exception) {
    System.out.println("Exception thrown: " + exception);
}
```

### 5. **@Around** - Wraps method execution

```java
@Around("execution(* com.example.*.*(..))")
public Object aroundAdvice(ProceedingJoinPoint joinPoint) throws Throwable {
    // Before method
    System.out.println("Before method: " + joinPoint.getSignature());
    
    // Execute actual method
    Object result = joinPoint.proceed();
    
    // After method
    System.out.println("After method: " + joinPoint.getSignature());
    
    return result;
}
```

**Important**: In `@Around`, YOU must call `joinPoint.proceed()` to execute the actual method

## How AOP Works Internally

### Application Startup Process

#### Step 1: Find @Aspect Classes
Spring scans for classes annotated with `@Aspect`

#### Step 2: Parse Pointcut Expressions
- Expressions are parsed and optimized
- Stored in efficient data structures/cache
- Handled by `PointcutParser` class

#### Step 3: Scan for Beans
Look for `@Component`, `@Service`, `@Controller`, etc.

#### Step 4: Check Interception Eligibility
For each bean, check if it matches any pointcut expression

#### Step 5: Create Proxies
If bean needs interception, create proxy using:
- **JDK Dynamic Proxy** - If class implements interface
- **CGLIB Proxy** - If class doesn't implement interface

### Proxy Creation Example

**Original Class**:
```java
@Component
public class EmployeeUtil {
    public void fetchEmployee() {
        System.out.println("Fetching employee");
    }
}
```

**Generated Proxy** (Conceptual):
```java
public class EmployeeUtil$$EnhancerBySpringCGLIB$$54321 extends EmployeeUtil {
    
    @Override
    public void fetchEmployee() {
        // 1. Execute before advice
        // 2. Call super.fetchEmployee()
        // 3. Execute after advice
    }
}
```

### Runtime Execution Flow

1. **Method Called**: Client calls method
2. **Proxy Intercepts**: Proxy class intercepts call
3. **Build Advice Chain**: Create chain of matching advice
4. **Execute Chain**:
    - Run @Before advice
    - Call actual method
    - Run @After advice
5. **Return Result**: Return to client

### Internal Classes Involved

- `PointcutParser` - Parses pointcut expressions
- `AbstractAutoProxyCreator` - Creates proxies
- `JdkDynamicAopProxy` - JDK proxy implementation
- `CglibAopProxy` - CGLIB proxy implementation
- `ReflectiveMethodInvocation` - Manages advice chain

## Complete Working Example

### Controller
```java
@RestController
public class EmployeeController {
    
    @Autowired
    private EmployeeService employeeService;
    
    @GetMapping("/api/employee/{id}")
    public Employee getEmployee(@PathVariable Long id) {
        return employeeService.findById(id);
    }
}
```

### Service
```java
@Service
public class EmployeeService {
    
    public Employee findById(Long id) {
        System.out.println("Finding employee with id: " + id);
        return new Employee(id, "John Doe");
    }
}
```

### Aspect
```java
@Aspect
@Component
public class LoggingAspect {
    
    // Log all service methods
    @Around("@within(org.springframework.stereotype.Service)")
    public Object logServiceMethods(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        
        System.out.println(">>> Entering " + className + "." + methodName);
        long start = System.currentTimeMillis();
        
        try {
            Object result = joinPoint.proceed();
            long duration = System.currentTimeMillis() - start;
            System.out.println("<<< Exiting " + className + "." + methodName + 
                             " (took " + duration + "ms)");
            return result;
        } catch (Exception e) {
            System.out.println("!!! Exception in " + className + "." + methodName + 
                             ": " + e.getMessage());
            throw e;
        }
    }
}
```

**Output**:
```
>>> Entering EmployeeService.findById
Finding employee with id: 1
<<< Exiting EmployeeService.findById (took 15ms)
```

## Best Practices

### 1. Use Specific Pointcuts
```java
// Good - Specific
@Before("execution(* com.example.service.EmployeeService.save(..))")

// Avoid - Too broad
@Before("execution(* *.*(..))")
```

### 2. Named Pointcuts for Reusability
```java
@Pointcut("@within(org.springframework.stereotype.Service)")
public void serviceMethods() {}
```

### 3. Prefer @Around for Complex Logic
- Can control method execution
- Access to arguments and return value
- Can modify behavior

### 4. Order Multiple Aspects
```java
@Aspect
@Order(1)  // Executes first
public class SecurityAspect { }

@Aspect
@Order(2)  // Executes second
public class LoggingAspect { }
```

## Common Use Cases

1. **Logging** - Method entry/exit, parameters, execution time
2. **Security** - Authorization checks
3. **Transactions** - Begin/commit/rollback
4. **Caching** - Check cache before execution
5. **Performance Monitoring** - Track execution time
6. **Exception Handling** - Centralized error handling
7. **Auditing** - Track who did what when

## Performance Considerations

1. **Proxy Creation**: Happens at startup, not runtime
2. **Pointcut Matching**: Optimized and cached
3. **Advice Execution**: Minimal overhead
4. **Memory**: Each proxied bean uses slightly more memory

## Key Interview Points

1. **AOP separates cross-cutting concerns** from business logic
2. **Proxy pattern** is used internally (JDK or CGLIB)
3. **Pointcuts** define WHERE, **Advice** defines WHAT
4. **@Around** requires manual method invocation via `proceed()`
5. **Proxies created at startup**, not runtime
6. **Used heavily** in Spring for @Transactional, @Cacheable, etc.

## Common Pitfalls

1. **Self-invocation doesn't work** - Calling method within same class bypasses proxy
2. **Final methods can't be proxied** with CGLIB
3. **Private methods can't be intercepted**
4. **Forgetting proceed()** in @Around advice
5. **Too broad pointcuts** affecting performance