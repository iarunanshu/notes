# Spring Boot Bean Scopes - Complete Notes

## Prerequisites
- Understanding of Bean and its lifecycle
- Knowledge of dependency injection
- Familiarity with IoC container

## Types of Bean Scopes

Spring Boot provides **5 different scopes**:
1. **Singleton** (Default)
2. **Prototype**
3. **Request** (Web-aware)
4. **Session** (Web-aware)
5. **Application** (Web-aware)

## 1. Singleton Scope (Default)

### Characteristics
- **One instance per IoC container**
- **Eagerly initialized** (created at application startup)
- Default scope if nothing specified
- Same instance shared across entire application

### How to Define
```java
// Option 1: Default (no annotation needed)
@Component
public class User {
    // Singleton by default
}

// Option 2: Explicit with enum
@Component
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)
public class User {
    // Explicitly singleton
}

// Option 3: String value
@Component
@Scope("singleton")
public class User {
    // Using string
}
```

### Example Flow
```java
@RestController  // Singleton by default
public class TestController1 {
    @Autowired
    private User user;
    
    @PostConstruct
    public void init() {
        System.out.println("Controller1 User: " + user.hashCode());
    }
}

@RestController  
public class TestController2 {
    @Autowired
    private User user;  // Same User instance as Controller1
    
    @PostConstruct
    public void init() {
        System.out.println("Controller2 User: " + user.hashCode());
        // Same hashcode as Controller1
    }
}
```

**Application Startup Output:**
```
User initialization
User object created: 1140202235
TestController1 initialization
Controller1 User: 1140202235  // Same instance
TestController2 initialization  
Controller2 User: 1140202235  // Same instance
```

## 2. Prototype Scope

### Characteristics
- **New instance created every time** bean is requested
- **Lazily initialized** (created only when needed)
- Not cached by IoC container

### How to Define
```java
@Component
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
// OR
@Scope("prototype")
public class User {
    public User() {
        System.out.println("New User instance created");
    }
}
```

### Example Scenario
```java
@RestController
@Scope("prototype")
public class TestController {
    @Autowired
    private User user;  // Prototype scope
    
    @Autowired
    private Student student;  // Singleton scope
}

@Component
public class Student {  // Singleton
    @Autowired
    private User user;  // Gets its own User instance
}
```

**Behavior:**
- Each time TestController is created → New instance
- Each time User is injected → New User instance
- Student (singleton) created once at startup
- Different User instances in TestController and Student

## 3. Request Scope (Web Applications)

### Characteristics
- **One instance per HTTP request**
- **Lazily initialized**
- Bean lifecycle tied to HTTP request lifecycle
- Requires web environment

### How to Define
```java
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST)
// OR
@Scope("request")
public class User {
    // New instance for each HTTP request
}
```

### Important: Proxy Mode Problem

**Problem Scenario:**
```java
@RestController  // Singleton (eager)
public class TestController {
    @Autowired
    private User user;  // Request scope (lazy)
    // FAILS! No HTTP request during startup
}
```

**Solution: Use Proxy Mode**
```java
@Component
@Scope(value = "request", 
       proxyMode = ScopedProxyMode.TARGET_CLASS)
public class User {
    // Proxy created during eager initialization
    // Real object created during HTTP request
}
```

### Example Flow
```java
@RestController
@Scope("request")
public class TestController {
    @Autowired
    private User user;  // Request scoped
    
    @Autowired  
    private Student student;  // Prototype scoped
}

@Component
@Scope("prototype")
public class Student {
    @Autowired
    private User user;  // Same User instance within request
}
```

**First HTTP Request:**
```
TestController initialization
User initialization (hashCode: 88929)
Student initialization
User injection to Student: 88929  // Same instance
```

**Second HTTP Request:**
```
TestController initialization  // New instance
User initialization (hashCode: 12345)  // New instance
Student initialization  // New instance
User injection to Student: 12345  // Same within request
```

## 4. Session Scope

### Characteristics
- **One instance per HTTP session**
- **Lazily initialized**
- Instance survives multiple HTTP requests in same session
- Destroyed when session expires/invalidates

### How to Define
```java
@Component
@Scope(value = WebApplicationContext.SCOPE_SESSION)
// OR  
@Scope("session")
public class ShoppingCart {
    // One instance per user session
}
```

### Session Configuration
```properties
# application.properties
server.servlet.session.timeout=30m  # Session timeout
```

### Example Flow
```java
@RestController
@Scope("session")
public class TestController {
    @Autowired
    private User user;  // Singleton
    
    @GetMapping("/fetchUser")
    public String fetchUser() {
        return "User: " + this.hashCode();
    }
    
    @GetMapping("/logout")
    public void logout(HttpServletRequest request) {
        request.getSession().invalidate();  // End session
    }
}
```

**Behavior:**
```
First API call → Session created → Controller instance created
Second API call → Same session → Same Controller instance
Logout → Session ended
Third API call → New session → New Controller instance
```

## 5. Application Scope

### Characteristics
- **One instance across multiple IoC containers**
- Similar to Singleton but broader scope
- Rarely used in practice
- For ServletContext-level beans

### How to Define
```java
@Component
@Scope(value = WebApplicationContext.SCOPE_APPLICATION)
public class ApplicationConfig {
    // Shared across all IoC containers
}
```

## Scope Comparison Table

| Scope | Instances | Initialization | Use Case |
|-------|-----------|---------------|----------|
| **Singleton** | One per IoC | Eager | Default, stateless beans |
| **Prototype** | New each time | Lazy | Stateful beans |
| **Request** | One per HTTP request | Lazy | Request-specific data |
| **Session** | One per HTTP session | Lazy | User session data |
| **Application** | One per ServletContext | Eager | Application-wide config |

## Key Interview Points

### Initialization Timing
- **Eager**: Singleton, Application (at startup)
- **Lazy**: Prototype, Request, Session (when needed)

### Proxy Mode Usage
Required when:
- Singleton/eager bean depends on Request/Session bean
- Resolves scope mismatch issues
- Creates proxy placeholder during eager initialization

### Common Patterns
```java
// Pattern 1: Controller (Singleton) + Service (Singleton)
@RestController
public class UserController {
    @Autowired
    private UserService service;  // Works fine
}

// Pattern 2: Controller (Singleton) + Request Data
@RestController
public class UserController {
    @Autowired
    private RequestData data;  // Needs proxy mode!
}

// Pattern 3: Stateful Components
@Component
@Scope("prototype")
public class ShoppingCart {
    private List<Item> items;  // Stateful - needs new instance
}
```

## Best Practices

1. **Use Singleton** for stateless beans (default)
2. **Use Prototype** for stateful components
3. **Use Request scope** for request-specific data
4. **Use Session scope** for user session data
5. **Always use proxy mode** when injecting narrower scope into wider scope
6. **Avoid circular dependencies** between different scopes

## Common Pitfalls

1. **Forgetting proxy mode** → Bean creation failure
2. **Using prototype in singleton** → Only one prototype instance created
3. **Not understanding lazy initialization** → Unexpected behavior
4. **Session scope without web context** → Application failure

## Debugging Tips

```java
@PostConstruct
public void init() {
    System.out.println(this.getClass().getSimpleName() + 
                      " created: " + this.hashCode());
}
```

Use hashcodes to verify:
- Same instance (singleton) → Same hashcode
- Different instances (prototype) → Different hashcodes
- Request boundary → New hashcodes per request