# Dynamic Bean Initialization in Spring Boot

## The Problem: Unsatisfied Dependency

### Scenario
```java
// Interface
public interface Order {
    void createOrder();
}

// Implementation 1
@Component
public class OnlineOrder implements Order {
    public void createOrder() {
        System.out.println("Created online order");
    }
}

// Implementation 2
@Component
public class OfflineOrder implements Order {
    public void createOrder() {
        System.out.println("Created offline order");
    }
}

// Controller with dependency
@RestController
public class UserController {
    @Autowired
    private Order order;  // Which implementation to inject?
    
    // APPLICATION FAILS TO START!
    // Error: expected single matching bean but found 2
}
```

### Issue
- Spring finds multiple implementations of Order
- Doesn't know which one to inject
- Application fails with **Unsatisfied Dependency Exception**

## Basic Solution: @Qualifier (Static Approach)

### Implementation
```java
// Add qualifier names
@Component
@Qualifier("onlineOrderObject")
public class OnlineOrder implements Order {
    // implementation
}

@Component
@Qualifier("offlineOrderObject")
public class OfflineOrder implements Order {
    // implementation
}

// Specify which one to use
@RestController
public class UserController {
    @Autowired
    @Qualifier("onlineOrderObject")  // Hard-coded!
    private Order order;
    
    @GetMapping("/createOrder")
    public void createOrder() {
        order.createOrder();  // Always creates online order
    }
}
```

### Problem with @Qualifier
- **Breaks Dependency Inversion Principle**
- Value is **hard-coded** at compile time
- Cannot change dynamically based on user input or configuration
- Always uses the same implementation

## Solution 1: Dynamic Using Multiple @Qualifiers

### Approach: Inject Both Beans, Choose at Runtime

```java
@RestController
public class UserController {
    
    @Autowired
    @Qualifier("onlineOrderObject")
    private Order onlineOrder;  // Inject online implementation
    
    @Autowired
    @Qualifier("offlineOrderObject")
    private Order offlineOrder;  // Inject offline implementation
    
    @GetMapping("/createOrder")
    public void createOrder(@RequestParam boolean isOnlineOrder) {
        // Dynamic selection based on user input
        if (isOnlineOrder) {
            onlineOrder.createOrder();  // Use online
        } else {
            offlineOrder.createOrder();  // Use offline
        }
    }
}
```

### Advantages
- ✅ **Industry standard approach**
- ✅ Dynamic selection at runtime
- ✅ Both beans are singleton (created once)
- ✅ Client can choose which implementation to use

### How It Works
1. Both beans (OnlineOrder, OfflineOrder) created at startup
2. Both injected into controller with different field names
3. Logic determines which one to use at runtime
4. No dependency inversion violation - decision made at runtime

## Solution 2: Dynamic Using @Configuration and @Value

### Remove @Component, Use @Bean Configuration

```java
// No @Component annotation
public class OnlineOrder implements Order {
    public void createOrder() {
        System.out.println("Created online order");
    }
}

// No @Component annotation
public class OfflineOrder implements Order {
    public void createOrder() {
        System.out.println("Created offline order");
    }
}
```

### Configuration Class with @Value

```java
@Configuration
public class OrderConfig {
    
    @Value("${is.online.order}")  // Read from properties
    private boolean isOnlineOrder;
    
    @Bean
    public Order createOrderBean() {
        // Dynamically decide which bean to create
        if (isOnlineOrder) {
            return new OnlineOrder();
        } else {
            return new OfflineOrder();
        }
    }
}
```

### Application Properties
```properties
# application.properties
is.online.order=false  # Change this to switch implementation
```

### Controller Remains Simple
```java
@RestController
public class UserController {
    
    @Autowired
    private Order order;  // Gets the bean based on configuration
    
    @GetMapping("/createOrder")
    public void createOrder() {
        order.createOrder();  // Uses configured implementation
    }
}
```

## @Value Annotation Deep Dive

### What is @Value?
- Injects values from various sources:
    - **Property files** (application.properties)
    - **Environment variables**
    - **Inline literals**

### Syntax Examples

```java
// From property file
@Value("${property.name}")
private String propertyValue;

// Default value if property not found
@Value("${property.name:defaultValue}")
private String valueWithDefault;

// Inline literal
@Value("hardcodedValue")
private String literalValue;

// Boolean from properties
@Value("${is.enabled}")
private boolean isEnabled;

// With SpEL (Spring Expression Language)
@Value("#{systemProperties['java.home']}")
private String javaHome;
```

### Complete Example: Dynamic Bean with Multiple Sources

```java
@Configuration
public class DynamicConfig {
    
    // From properties file
    @Value("${order.type:online}")  // Default to 'online' if not set
    private String orderType;
    
    // From environment variable
    @Value("${ORDER_PRIORITY:normal}")
    private String priority;
    
    @Bean
    public Order orderBean() {
        if ("online".equals(orderType)) {
            return new OnlineOrder();
        } else if ("offline".equals(orderType)) {
            return new OfflineOrder();
        } else {
            return new DefaultOrder();  // Fallback
        }
    }
}
```

## Comparison of Approaches

| Aspect | @Qualifier (Static) | Multiple @Qualifiers | @Configuration + @Value |
|--------|-------------------|---------------------|------------------------|
| **Flexibility** | ❌ Fixed at compile time | ✅ Runtime decision | ✅ Configuration-based |
| **Dependency Inversion** | ❌ Violates | ✅ Maintains | ✅ Maintains |
| **Memory Usage** | Low (1 bean) | Higher (all beans) | Low (1 bean) |
| **Use Case** | Fixed implementation | User-driven choice | Environment-based |
| **Industry Usage** | Rare | **Most Common** | Configuration-driven |

## Best Practices

### 1. Choose Based on Requirements
- **User Input Driven**: Use multiple @Qualifiers
- **Environment Driven**: Use @Configuration + @Value
- **Never Changing**: Can use single @Qualifier

### 2. Property File Organization
```properties
# Group related properties
order.type=online
order.timeout=30
order.retry.count=3

# Use meaningful names
feature.payment.enabled=true
feature.notification.enabled=false
```

### 3. Provide Defaults
```java
// Always provide defaults for @Value
@Value("${order.type:online}")  // Defaults to 'online'
private String orderType;
```

### 4. Consider @ConditionalOnProperty
```java
@Configuration
@ConditionalOnProperty(
    name = "order.mode",
    havingValue = "online"
)
public class OnlineOrderConfig {
    // Configuration only loaded if property matches
}
```

## Flow Summary

### Solution 1 Flow (Multiple Qualifiers)
```
1. Application starts
2. Both OnlineOrder and OfflineOrder beans created
3. Both injected into UserController
4. API called with parameter
5. Logic determines which bean's method to call
```

### Solution 2 Flow (@Configuration + @Value)
```
1. Application starts
2. @Value reads from properties file
3. @Bean method decides which implementation to create
4. Only one bean created and injected
5. API uses the configured implementation
```

## Key Interview Points

1. **Unsatisfied Dependency** occurs with multiple implementations
2. **@Qualifier alone** breaks dependency inversion (hard-coded)
3. **Industry standard**: Inject all implementations, choose at runtime
4. **@Value** enables configuration-driven bean creation
5. **Dynamic initialization** maintains loose coupling
6. Both solutions maintain **Dependency Inversion Principle**

## Common Pitfalls

1. **Forgetting @Qualifier** with multiple implementations
2. **Not providing defaults** for @Value properties
3. **Creating all beans** when only one is needed (memory waste)
4. **Hard-coding** values instead of using configuration