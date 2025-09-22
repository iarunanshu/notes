# Spring Boot @ConditionalOnProperty Annotation

## Why This Annotation is Important

In large-scale applications with **thousands of beans**:
- Many beans initialized at startup (singleton scope)
- Application context gets **cluttered with unnecessary beans**
- Wastes memory and increases startup time
- Need way to conditionally create beans

**Interview Question**: "How can we unclutter our application context from unnecessary beans?"
**Answer**: Use `@ConditionalOnProperty` to conditionally create beans based on configuration

## What is @ConditionalOnProperty?

**Definition**: Bean is created **conditionally** based on property configuration
- If condition is **true** → Bean created
- If condition is **false** → Bean NOT created

## Common Use Cases

### Use Case 1: Migration Scenarios
```
Company migrating from MySQL → NoSQL
- During migration: Both beans needed
- After migration: Only NoSQL bean needed
- Control bean creation via configuration
```

### Use Case 2: Shared Codebase
```
Common Library
├── MySQLConnection class
└── NoSQLConnection class

Application 1 → Needs only NoSQL
Application 2 → Needs only MySQL
```

Both applications share common code but need different beans

## How @ConditionalOnProperty Works

### Annotation Parameters

```java
@ConditionalOnProperty(
    prefix = "sql.connection",     // First part of property key
    value = "enabled",              // Second part of property key
    havingValue = "true",          // Expected value to match
    matchIfMissing = false         // What if property not found?
)
```

### Property Key Formation
- **Key** = `prefix` + "." + `value`
- Example: `sql.connection.enabled`

### Matching Logic
1. Spring creates key: `prefix.value`
2. Looks in `application.properties` for this key
3. Compares property value with `havingValue`
4. If match → Create bean
5. If no match → Don't create bean
6. If property missing → Check `matchIfMissing`

## Complete Example

### Step 1: Define Conditional Beans

```java
// MySQL Connection - Only created if enabled
@Component
@ConditionalOnProperty(
    prefix = "sql.connection",
    value = "enabled",
    havingValue = "true",
    matchIfMissing = false
)
public class MySQLConnection {
    
    public MySQLConnection() {
        System.out.println("MySQL Connection bean initialized");
    }
}

// NoSQL Connection - Only created if enabled
@Component
@ConditionalOnProperty(
    prefix = "nosql.connection",
    value = "enabled",
    havingValue = "true",
    matchIfMissing = false
)
public class NoSQLConnection {
    
    public NoSQLConnection() {
        System.out.println("NoSQL Connection bean initialized");
    }
}
```

### Step 2: Use with required=false

```java
@Component
public class DBConnection {
    
    @Autowired(required = false)  // IMPORTANT: Must be false!
    private MySQLConnection mySQLConnection;
    
    @Autowired(required = false)  // Bean might not exist
    private NoSQLConnection noSQLConnection;
    
    @PostConstruct
    public void init() {
        System.out.println("DB Connection bean created");
        System.out.println("MySQL null? " + (mySQLConnection == null));
        System.out.println("NoSQL null? " + (noSQLConnection == null));
    }
}
```

### Step 3: Configure Properties

```properties
# application.properties

# Enable MySQL, disable NoSQL
sql.connection.enabled=true
# nosql.connection.enabled property not present (missing)
```

### Execution Flow

```
1. Spring finds MySQLConnection class
2. Checks: sql.connection.enabled = true
3. Matches havingValue = "true" ✓
4. Creates MySQLConnection bean

5. Spring finds NoSQLConnection class
6. Checks: nosql.connection.enabled = ???
7. Property missing, matchIfMissing = false
8. Does NOT create NoSQLConnection bean

9. Creates DBConnection bean
10. Injects MySQLConnection (exists)
11. Sets noSQLConnection = null (required=false)
```

**Output:**
```
MySQL Connection bean initialized
DB Connection bean created
MySQL null? false
NoSQL null? true
```

## Important: Why required=false?

### Default @Autowired Behavior
```java
@Autowired  // Default: required=true
private MySQLConnection connection;
```
- Spring **MUST** find the bean
- If bean doesn't exist → **Application fails**

### With @ConditionalOnProperty
```java
@Autowired(required = false)
private MySQLConnection connection;
```
- Bean might not exist (conditional)
- If not found → Set to `null`
- Application continues normally

## Advanced Examples

### Using Different String Values

```java
@ConditionalOnProperty(
    prefix = "database",
    value = "type",
    havingValue = "mysql"  // Can be any string
)
public class MySQLConfig {
    // Bean created only if database.type=mysql
}
```

```properties
# Enable with custom string
database.type=mysql
```

### Match If Missing Example

```java
@ConditionalOnProperty(
    prefix = "feature",
    value = "enabled",
    havingValue = "true",
    matchIfMissing = true  // Create bean even if property missing
)
public class OptionalFeature {
    // Created by default if property not specified
}
```

## Feature Toggling Example

```java
@RestController
@ConditionalOnProperty(
    prefix = "feature.payment",
    value = "enabled",
    havingValue = "true"
)
public class PaymentController {
    // Entire controller enabled/disabled via configuration
}
```

```properties
# Toggle features without code changes
feature.payment.enabled=true  # Enable payment feature
feature.payment.enabled=false # Disable payment feature
```

## Advantages

### 1. **Feature Toggling**
- Enable/disable features via configuration
- Support gradual migrations
- A/B testing capabilities

### 2. **Avoid Context Cluttering**
- Don't create unnecessary beans
- Cleaner application context
- Better organized dependencies

### 3. **Save Memory**
- Beans not created = Memory saved
- Important for applications with thousands of beans

### 4. **Reduce Startup Time**
- Fewer beans to initialize
- Faster application startup
- Better for microservices

## Disadvantages

### 1. **Misconfiguration Risk**
```properties
# Typos or wrong values
sql.connection.enabled=ture  # Typo! Should be "true"
```

### 2. **Code Complexity**
- Need to check properties to know if bean exists
- Harder to track dependencies
- More configuration to manage

### 3. **Same Configuration for Multiple Beans**
```java
// Confusing: Both use same property
@ConditionalOnProperty(prefix = "db", value = "enabled")
public class MySQLConnection { }

@ConditionalOnProperty(prefix = "db", value = "enabled")
public class PostgreSQLConnection { }
```

### 4. **Management Complexity**
- Multiple property files
- Environment-specific configurations
- Documentation overhead

## Best Practices

### 1. Use Descriptive Property Names
```properties
# Good
mysql.connection.enabled=true
redis.cache.enabled=false

# Bad
db1.enabled=true
db2.enabled=false
```

### 2. Group Related Properties
```properties
# Feature flags
feature.payment.enabled=true
feature.notification.enabled=true
feature.analytics.enabled=false

# Database configurations
database.mysql.enabled=true
database.postgres.enabled=false
```

### 3. Document Property Usage
```java
/**
 * MySQL connection bean
 * Enable with: mysql.connection.enabled=true
 */
@ConditionalOnProperty(...)
public class MySQLConnection { }
```

### 4. Provide Sensible Defaults
```java
@ConditionalOnProperty(
    prefix = "feature.cache",
    value = "enabled",
    havingValue = "true",
    matchIfMissing = true  // Enabled by default
)
```

### 5. Use with Profiles
```java
@Component
@Profile("production")
@ConditionalOnProperty(
    prefix = "feature.advanced",
    value = "enabled"
)
public class AdvancedFeature {
    // Only in production AND if enabled
}
```

## Common Interview Questions

**Q1: When would you use @ConditionalOnProperty?**
- Feature toggles
- Migration scenarios
- Environment-specific beans
- Reducing memory footprint

**Q2: What happens if property is missing?**
- Check `matchIfMissing` parameter
- true → Create bean anyway
- false → Don't create bean

**Q3: Can you use without @Component?**
```java
@Configuration
public class Config {
    
    @Bean
    @ConditionalOnProperty(
        prefix = "service",
        value = "enabled"
    )
    public MyService myService() {
        return new MyService();
    }
}
```
Yes, works with @Bean methods too!

**Q4: How does it help in microservices?**
- Reduce startup time
- Feature flags for gradual rollouts
- Environment-specific configurations
- Memory optimization

## Summary

`@ConditionalOnProperty` is crucial for:
- **Dynamic bean creation** based on configuration
- **Memory and performance** optimization
- **Feature management** without code changes
- **Clean architecture** with only necessary beans

Remember: Always use `required=false` with `@Autowired` when injecting conditional beans!