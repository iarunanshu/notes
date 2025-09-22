# Spring Boot Beans & IoC Container - Complete Notes

## What is a Bean?

**Bean Definition**: A Java object that is managed by Spring Container (IoC - Inversion of Control Container)
- Not different from regular Java objects, just managed by Spring
- Spring handles creation, initialization, and lifecycle management

## IoC Container

**IoC (Inversion of Control) Container**:
- Contains all beans created in the application
- Manages complete lifecycle: creation → initialization → destruction
- **Application Context** provides the implementation for IoC logic

## Creating Beans - Two Ways

### 1. @Component Annotation
- Follows **"Convention over Configuration"** approach
- Spring uses auto-configuration (default constructor)
- Spring automatically determines how to create the object

**Example:**
```java
@Component
public class User {
    private String username;
    private String email;
    
    // Getters and setters
}
```

**Related Annotations (all internally use @Component):**
- `@Controller` - For controller classes
- `@Service` - For service layer classes
- `@Repository` - For data access layer
- `@RestController` - Controller + ResponseBody

**Limitation**: Fails when no default constructor exists
```java
@Component
public class User {
    private String username;
    private String email;
    
    // Custom constructor - NO default constructor
    public User(String username, String email) {
        this.username = username;
        this.email = email;
    }
    // This will FAIL - Spring doesn't know what values to pass
}
```

### 2. @Bean Annotation
- Provides **external configuration**
- Tells Spring exactly how to create the bean
- Used when @Component won't work

**Example:**
```java
@Configuration  // Tells Spring this class contains bean definitions
public class AppConfig {
    
    @Bean
    public User createUserBean() {
        // Tell Spring exactly how to create User
        return new User("defaultUsername", "defaultEmail");
    }
}
```

**Key Points:**
- External configuration gets **priority** over auto-configuration
- Multiple @Bean methods creating same type = Multiple beans created
- Each @Bean method creates a separate object

## How Spring Finds Beans

### 1. @ComponentScan
- Tells Spring which packages to scan
- Looks for: @Component, @Service, @Repository, @Controller
- Scans specified package and all sub-packages

```java
@SpringBootApplication  // Already includes @ComponentScan
@ComponentScan(basePackages = "com.example.myapp")
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**Default Behavior**:
- @SpringBootApplication includes @ComponentScan
- Auto-configures to scan from package where main class exists

### 2. @Configuration
- Spring looks for classes annotated with @Configuration
- @Configuration is also a @Component
- Processes all @Bean methods inside configuration classes

## When Beans Get Created

### Eager Initialization
- Beans created at **application startup**
- Default for **Singleton scope** beans
- Created even if not immediately needed

### Lazy Initialization
- Beans created **only when needed**
- Can be enforced with `@Lazy` annotation
- Default for **Prototype scope** beans

```java
@Component
@Lazy  // Force lazy initialization even for singleton
public class OrderService {
    // Will only be created when first requested
}
```

## Bean Lifecycle - Complete Flow

### 1. **Application Startup**
```
Application starts → IoC Container initialized
```

### 2. **Scan for Beans**
```
IoC scans using:
- @ComponentScan (finds @Component, @Service, etc.)
- @Configuration (finds @Bean methods)
```

### 3. **Construct Beans**
- Creates beans marked for eager initialization
- Singleton beans created by default
- Lazy beans skipped until needed

### 4. **Dependency Injection**
- Injects dependencies using `@Autowired`
- Creates lazy beans if required as dependency

**Example with Dependency:**
```java
@Component
public class User {
    @Autowired
    private Order order;  // Dependency
    
    public User() {
        System.out.println("Initializing User");
    }
}

@Component
@Lazy
public class Order {
    public Order() {
        System.out.println("Initializing Order");
    }
}
```

**Output:**
```
Initializing User     // User created (eager)
Initializing Order    // Order created because User needs it
```

### 5. **@PostConstruct - Pre-initialization**
- Method executed after bean construction and dependency injection
- Used for initialization logic

```java
@Component
public class User {
    @Autowired
    private Order order;
    
    @PostConstruct
    public void init() {
        System.out.println("Bean constructed, dependencies injected");
        // Initialization logic here
    }
}
```

### 6. **Bean in Use**
- Bean is ready and being used in application
- Methods can be called on the bean

### 7. **@PreDestroy - Pre-destruction**
- Method executed before bean destruction
- Used for cleanup (closing connections, releasing resources)

```java
@Component
public class DatabaseService {
    
    @PreDestroy
    public void cleanup() {
        System.out.println("Releasing database connections");
        // Cleanup logic here
    }
}
```

### 8. **Bean Destroyed**
- Bean removed from IoC container
- Happens when application shuts down

## Complete Lifecycle Example

```java
@Component
public class UserService {
    
    @Autowired
    private OrderService orderService;
    
    // 1. Constructor called during bean creation
    public UserService() {
        System.out.println("1. UserService Constructor");
    }
    
    // 2. Dependencies injected after construction
    // (@Autowired happens here)
    
    // 3. Post-construction initialization
    @PostConstruct
    public void init() {
        System.out.println("3. PostConstruct - Initialization");
    }
    
    // 4. Bean is used (business methods called)
    public void businessMethod() {
        System.out.println("4. Using the bean");
    }
    
    // 5. Pre-destruction cleanup
    @PreDestroy
    public void cleanup() {
        System.out.println("5. PreDestroy - Cleanup");
    }
    
    // 6. Bean destroyed
}
```

**Output when application starts and stops:**
```
1. UserService Constructor
2. [Dependency injection happens]
3. PostConstruct - Initialization
4. Using the bean (when methods called)
5. PreDestroy - Cleanup (on shutdown)
```

## @Autowired - Dependency Injection

**How it works:**
1. First checks if bean already exists in IoC container
2. If exists → uses existing bean
3. If not exists → creates new bean (even if lazy)

**Example:**
```java
@Component
public class UserController {
    
    @Autowired
    private UserService userService;
    // Spring automatically injects UserService bean
}
```

## Important Interview Points

### Bean vs Regular Object
- **Bean**: Managed by Spring (lifecycle, dependencies)
- **Object**: Created with `new`, managed by developer

### @Component vs @Bean
| @Component | @Bean |
|------------|-------|
| Class-level annotation | Method-level annotation |
| Auto-configuration | Manual configuration |
| Requires default constructor | Works without default constructor |
| Convention over configuration | Explicit configuration |

### Priority Rules
1. **@Bean (external config) > @Component (auto-config)**
2. Multiple beans of same type → Use `@Qualifier` to specify

### Scope Default
- No @Scope annotation = **Singleton** (one instance)
- Singleton = Eager initialization (unless @Lazy)

### Lifecycle Sequence
```
Start → Scan → Construct → Inject → @PostConstruct → Use → @PreDestroy → Destroy
```

## Best Practices

1. **Use @Component** for simple beans with default constructors
2. **Use @Bean** when:
    - No default constructor
    - Need custom initialization
    - Creating third-party class beans
3. **Use @Lazy** carefully - can delay startup issues discovery
4. **Always cleanup resources** in @PreDestroy
5. **Initialize collections/maps** in @PostConstruct, not constructor

## Next Topics to Cover
- Dependency Injection Types (Constructor, Setter, Field)
- Bean Scopes (Singleton, Prototype, Request, Session)
- @Qualifier and bean naming
- Circular dependencies handling