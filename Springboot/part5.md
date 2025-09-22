# Spring Boot Dependency Injection - Complete Notes

## Problem Without Dependency Injection

### Tight Coupling Example
```java
public class User {
    private Order order;
    
    public User() {
        order = new Order();  // Creating instance directly - TIGHT COUPLING!
    }
}
```

### Issues with Tight Coupling

1. **High Coupling**: Changes in Order class impact User class
2. **Breaks SOLID Principles**: Violates Dependency Inversion Principle (DIP)
    - DIP says: "Depend on abstractions, not concrete implementations"
3. **Hard to Test**: Cannot mock dependencies
4. **Inflexible**: Cannot switch implementations

### Example Problem Scenario
```java
// Original
class Order { }

// Later changed to interface with implementations
interface Order { }
class OnlineOrder implements Order { }
class OfflineOrder implements Order { }

// Now User class breaks!
public User() {
    order = new Order();  // ERROR: Cannot instantiate interface
}
```

## What is Dependency Injection?

**Definition**: Making classes independent of their dependencies by providing dependencies from external sources

**Benefits**:
- Achieves loose coupling
- Follows Dependency Inversion Principle
- Makes code testable and maintainable

## How Spring Boot Does Dependency Injection

### Basic Setup
```java
@Component
public class User {
    @Autowired
    private Order order;  // Spring injects dependency
    
    // No manual instantiation needed!
}

@Component
public class Order {
    // Order implementation
}
```

### @Autowired Behavior
1. First checks if bean already exists in IoC container
2. If exists → uses existing bean
3. If not exists → creates new bean and injects it

## Three Types of Dependency Injection

### 1. Field Injection

**How it works**: Dependencies injected directly into fields using reflection

```java
@Component
public class User {
    @Autowired
    private Order order;  // Field injection
    
    public User() {
        System.out.println("User initialized");
    }
}
```

**Advantages**:
- ✅ Simple and easy to use
- ✅ Clean and readable code

**Disadvantages**:
- ❌ **Cannot make fields immutable** (cannot use `final`)
- ❌ **Risk of NullPointerException** if object created with `new User()`
- ❌ **Harder to unit test** (needs reflection-based mocking)

**Example of NPE Risk**:
```java
// If someone creates object manually
User user = new User();  // order field is null!
user.process();  // NullPointerException!
```

### 2. Setter Injection

**How it works**: Dependencies injected through setter methods

```java
@Component
public class User {
    private Order order;
    
    @Autowired
    public void setOrder(Order order) {
        this.order = order;
    }
}
```

**Advantages**:
- ✅ Dependencies can be changed after object creation
- ✅ Easy unit testing (can pass mock via setter)

**Disadvantages**:
- ❌ Cannot make fields immutable
- ❌ Difficult to read and maintain
- ❌ Optional dependencies unclear

### 3. Constructor Injection (RECOMMENDED)

**How it works**: Dependencies resolved during object instantiation

```java
@Component
public class User {
    private final Order order;  // Can be final!
    
    @Autowired  // Optional from Spring 4.3 if only one constructor
    public User(Order order) {
        this.order = order;
        System.out.println("User initialized with dependencies");
    }
}
```

**Advantages**:
- ✅ **All dependencies initialized at object creation time**
- ✅ **Can make fields immutable** (`final`)
- ✅ **Fail-fast**: Missing dependencies detected at startup
- ✅ **Easy unit testing**: Pass mocks through constructor
- ✅ **Forces good design**: Many parameters indicate refactoring needed

**Important Note**: From Spring 4.3+, `@Autowired` is optional if only ONE constructor exists

**Multiple Constructors Example**:
```java
@Component
public class User {
    private Order order;
    private Invoice invoice;
    
    // Need @Autowired to specify which constructor
    @Autowired
    public User(Order order) {
        this.order = order;
    }
    
    public User(Invoice invoice) {
        this.invoice = invoice;
    }
}
```

## Common Issues and Solutions

### 1. Circular Dependency

**Problem**: A depends on B, B depends on A

```java
@Component
public class Order {
    @Autowired
    private Invoice invoice;  // Order needs Invoice
}

@Component
public class Invoice {
    @Autowired
    private Order order;  // Invoice needs Order - CIRCULAR!
}
```

**Solutions**:

#### Solution 1: Refactor Code (BEST)
- Extract common code to separate class
- Break the circular dependency

#### Solution 2: Use @Lazy
```java
@Component
public class Order {
    @Autowired
    @Lazy  // Creates proxy, delays initialization
    private Invoice invoice;
}
```

#### Solution 3: Use @PostConstruct
```java
@Component
public class Invoice {
    private Order order;  // No @Autowired
    
    @Autowired
    private Order orderBean;
    
    @PostConstruct
    public void init() {
        orderBean.setInvoice(this);  // Manual injection
    }
}
```

### 2. Unsatisfied Dependency

**Problem**: Multiple implementations of same interface

```java
interface Order { }

@Component
class OnlineOrder implements Order { }

@Component
class OfflineOrder implements Order { }

@Component
public class User {
    @Autowired
    private Order order;  // Which implementation to inject?
}
```

**Solutions**:

#### Solution 1: Use @Primary
```java
@Component
@Primary  // This will be injected by default
class OnlineOrder implements Order { }

@Component
class OfflineOrder implements Order { }
```

#### Solution 2: Use @Qualifier
```java
@Component
@Qualifier("online")
class OnlineOrder implements Order { }

@Component
@Qualifier("offline")
class OfflineOrder implements Order { }

@Component
public class User {
    @Autowired
    @Qualifier("offline")  // Specify which one
    private Order order;
}
```

## Injection Execution Flow

### Field Injection Flow:
1. User object created (constructor called)
2. Spring uses reflection to find @Autowired fields
3. Dependencies resolved and injected
4. @PostConstruct methods called

### Constructor Injection Flow:
1. Dependencies resolved FIRST
2. User object created WITH dependencies
3. @PostConstruct methods called

## Best Practices

### ✅ DO:
1. **Use Constructor Injection** for mandatory dependencies
2. **Make dependencies final** when possible
3. **Use @Primary or @Qualifier** for multiple implementations
4. **Refactor circular dependencies** instead of using workarounds

### ❌ DON'T:
1. **Avoid Field Injection** in production code
2. **Don't create objects with `new`** if they need injection
3. **Don't ignore circular dependency warnings**

## Comparison Table

| Feature | Field Injection | Setter Injection | Constructor Injection |
|---------|----------------|------------------|----------------------|
| **Immutability** | ❌ No | ❌ No | ✅ Yes |
| **NPE Risk** | ❌ High | ❌ Medium | ✅ Low |
| **Testing** | ❌ Hard | ✅ Easy | ✅ Easy |
| **Fail-fast** | ❌ No | ❌ No | ✅ Yes |
| **Readability** | ✅ Good | ❌ Poor | ✅ Good |
| **When Initialized** | After creation | After creation | During creation |
| **Spring Recommendation** | ❌ | ❌ | ✅ |

## Interview Key Points

1. **DI solves tight coupling** and helps achieve Dependency Inversion Principle
2. **Constructor injection is recommended** because:
    - Ensures complete initialization
    - Supports immutability
    - Fails fast with missing dependencies
3. **@Autowired optional** from Spring 4.3 for single constructor
4. **Circular dependencies** indicate design problems - refactor first
5. **@Primary vs @Qualifier**:
    - @Primary: Default choice among multiple beans
    - @Qualifier: Explicit choice by name
6. **Field injection uses reflection** - performance overhead
7. **Constructor with many parameters** = Code smell, needs refactoring

## Common Interview Questions

**Q: Why is constructor injection preferred?**
A: Ensures complete initialization, supports immutability, fails fast, easier testing

**Q: How does Spring resolve dependencies?**
A: Checks IoC container first, creates if not exists, uses reflection for field/setter injection

**Q: How to handle circular dependencies?**
A: Best to refactor, can use @Lazy or @PostConstruct as workarounds

**Q: What happens with multiple implementations?**
A: Application fails to start - use @Primary or @Qualifier to resolve