# Spring Boot @Transactional Annotation - Part 2

## Transaction Manager Hierarchy

### 1. **TransactionManager** (Top-level Interface)
- Empty interface
- Marker interface for all transaction managers

### 2. **PlatformTransactionManager** (Main Interface)
```java
public interface PlatformTransactionManager {
    TransactionStatus getTransaction(TransactionDefinition definition);
    void commit(TransactionStatus status);
    void rollback(TransactionStatus status);
}
```

### 3. **AbstractPlatformTransactionManager** (Abstract Class)
- Provides default implementation of the three methods
- Most transaction logic is common across implementations
- Concrete classes can override specific behavior

### 4. **Concrete Transaction Managers**
- **DataSourceTransactionManager** - For JDBC
- **JpaTransactionManager** - For JPA (default in most cases)
- **HibernateTransactionManager** - For Hibernate
- **JtaTransactionManager** - For distributed transactions (2-phase commit)

## Types of Transaction Management

### 1. Declarative Transaction Management

**Using Annotations** - Clean and simple approach

```java
@Service
public class UserService {
    
    @Transactional
    public void updateUser() {
        // Spring handles begin, commit, rollback
        // Your business logic only
    }
}
```

**Specifying Transaction Manager**:

```java
// Configuration class
@Configuration
public class AppConfig {
    
    @Bean
    public PlatformTransactionManager userTransactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}

// Using specific transaction manager
@Service
public class UserService {
    
    @Transactional(transactionManager = "userTransactionManager")
    public void updateUser() {
        // Uses specified transaction manager
    }
}
```

### 2. Programmatic Transaction Management

**When to Use Programmatic?**

Example scenario where declarative won't work well:
```java
@Service
public class UserService {
    
    public void updateUser() {
        // Step 1: Initial DB operations
        updateDB();  // Need transaction
        
        // Step 2: External API call
        callExternalAPI();  // Time consuming - shouldn't hold DB connection
        
        // Step 3: Final DB operations
        finalUpdateDB();  // Need transaction
    }
}
```

Problem with @Transactional here:
- DB connection held during external API call
- Can cause connection pool exhaustion during peak traffic
- External calls are typically slow

#### Approach 1: Manual Transaction Management

```java
@Service
public class UserService {
    
    private final PlatformTransactionManager transactionManager;
    
    // Constructor injection
    public UserService(PlatformTransactionManager transactionManager) {
        this.transactionManager = transactionManager;
    }
    
    public void updateUserProgrammatic() {
        // Create transaction
        TransactionStatus status = transactionManager.getTransaction(null);
        
        try {
            // Your business logic
            performDatabaseOperations();
            
            // Commit if successful
            transactionManager.commit(status);
            
        } catch (Exception e) {
            // Rollback on error
            transactionManager.rollback(status);
            throw e;
        }
    }
}
```

#### Approach 2: Using TransactionTemplate (Cleaner)

**Configuration**:
```java
@Configuration
public class AppConfig {
    
    @Bean
    public TransactionTemplate transactionTemplate(PlatformTransactionManager txManager) {
        return new TransactionTemplate(txManager);
    }
}
```

**Usage**:
```java
@Service
public class UserService {
    
    @Autowired
    private TransactionTemplate transactionTemplate;
    
    public void updateUserWithTemplate() {
        transactionTemplate.execute(status -> {
            // Your business logic here
            performDatabaseOperations();
            return null;  // or return result
        });
    }
}
```

**TransactionTemplate internally handles**:
1. getTransaction()
2. Execute your callback function
3. commit() on success
4. rollback() on exception

## Transaction Propagation

**Definition**: Defines how transactions relate to each other when methods with @Transactional call other @Transactional methods

### Scenario Setup
```java
@Service
public class MyService {
    
    @Transactional
    public void method1() {
        // Some operations
        method2();  // Calling another transactional method
    }
    
    @Transactional(propagation = Propagation.REQUIRED)
    public void method2() {
        // Some operations
    }
}
```

### Propagation Types

#### 1. **REQUIRED** (Default)
```java
@Transactional(propagation = Propagation.REQUIRED)
```
- **If parent transaction exists**: Use the same transaction
- **If no parent transaction**: Create new transaction

**Example Output**:
```
Method1: Transaction Active: true, Name: com.example.MyService.method1
Method2: Transaction Active: true, Name: com.example.MyService.method1 (SAME)
```

#### 2. **REQUIRES_NEW**
```java
@Transactional(propagation = Propagation.REQUIRES_NEW)
```
- **If parent transaction exists**: Suspend parent, create new transaction
- **If no parent transaction**: Create new transaction
- After completion, resume parent transaction

**Example Output**:
```
Method1: Transaction Active: true, Name: com.example.MyService.method1
Method2: Transaction Active: true, Name: com.example.MyService.method2 (DIFFERENT)
```

#### 3. **SUPPORTS**
```java
@Transactional(propagation = Propagation.SUPPORTS)
```
- **If parent transaction exists**: Use it
- **If no parent transaction**: Execute without transaction

#### 4. **NOT_SUPPORTED**
```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
```
- **If parent transaction exists**: Suspend it, execute without transaction
- **If no parent transaction**: Execute without transaction
- Always executes without transaction

#### 5. **MANDATORY**
```java
@Transactional(propagation = Propagation.MANDATORY)
```
- **If parent transaction exists**: Use it
- **If no parent transaction**: Throw exception
- Never creates new transaction

#### 6. **NEVER**
```java
@Transactional(propagation = Propagation.NEVER)
```
- **If parent transaction exists**: Throw exception
- **If no parent transaction**: Execute without transaction

### Propagation Summary Table

| Propagation | Parent Exists | No Parent |
|------------|---------------|-----------|
| REQUIRED | Use parent | Create new |
| REQUIRES_NEW | Suspend parent, create new | Create new |
| SUPPORTS | Use parent | No transaction |
| NOT_SUPPORTED | Suspend parent, no transaction | No transaction |
| MANDATORY | Use parent | Throw exception |
| NEVER | Throw exception | No transaction |

## Setting Propagation in Programmatic Approach

### Approach 1: Manual Transaction
```java
public void updateUser() {
    // Create transaction definition
    DefaultTransactionDefinition def = new DefaultTransactionDefinition();
    def.setName("MyTransaction");
    def.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
    
    // Get transaction with definition
    TransactionStatus status = transactionManager.getTransaction(def);
    
    try {
        // Business logic
        transactionManager.commit(status);
    } catch (Exception e) {
        transactionManager.rollback(status);
    }
}
```

### Approach 2: TransactionTemplate
```java
@Configuration
public class AppConfig {
    
    @Bean
    public TransactionTemplate transactionTemplate(PlatformTransactionManager txManager) {
        TransactionTemplate template = new TransactionTemplate(txManager);
        template.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
        template.setName("MyTransaction");
        return template;
    }
}
```

## Internal Code Flow

### How Propagation is Checked

In `TransactionInterceptor`:
```java
// Simplified version
protected TransactionInfo createTransactionIfNecessary() {
    // Check existing transaction
    if (existingTransaction != null) {
        // Check propagation type
        if (propagation == NEVER) {
            throw new IllegalTransactionStateException("Transaction not allowed");
        }
        if (propagation == NOT_SUPPORTED) {
            suspendCurrentTransaction();
            return null;
        }
        if (propagation == REQUIRES_NEW) {
            suspendCurrentTransaction();
            return startNewTransaction();
        }
        // REQUIRED, SUPPORTS, MANDATORY - use existing
        return existingTransaction;
    } else {
        // No existing transaction
        if (propagation == MANDATORY) {
            throw new IllegalTransactionStateException("Transaction required");
        }
        if (propagation == REQUIRED || propagation == REQUIRES_NEW) {
            return startNewTransaction();
        }
        // SUPPORTS, NOT_SUPPORTED, NEVER - no transaction
        return null;
    }
}
```

## Key Interview Questions

### 1. **When to use Declarative vs Programmatic?**

**Declarative**:
- Most common cases (95%)
- Clean code
- When entire method needs transaction

**Programmatic**:
- Fine-grained control needed
- Partial transaction within method
- Complex scenarios with external calls

### 2. **What happens with nested @Transactional methods?**
Depends on propagation:
- REQUIRED: Same transaction
- REQUIRES_NEW: New nested transaction
- Other propagations have different behaviors

### 3. **Can you mix declarative and programmatic?**
Yes, but be careful:
- Programmatic transaction inside @Transactional method works
- Need to understand propagation behavior

### 4. **Default propagation and why?**
REQUIRED - Most common use case where you want transaction if not present, or join existing

### 5. **Common propagation use cases:**
- **REQUIRED**: Default, most operations
- **REQUIRES_NEW**: Audit logging (should commit even if main transaction fails)
- **MANDATORY**: Service methods that must be called within transaction
- **SUPPORTS**: Read operations that may or may not need transaction

## Best Practices

1. **Use Declarative by default** - Cleaner and easier to maintain
2. **Understand propagation** - Wrong propagation can cause data inconsistency
3. **Be careful with REQUIRES_NEW** - Creates separate transaction, can cause deadlocks
4. **Don't hold transactions during external calls** - Use programmatic approach
5. **Name your transactions** - Helps in debugging and monitoring

## Common Pitfalls

1. **Self-invocation doesn't work** with @Transactional (proxy limitation)
2. **Mixing propagation types** without understanding can cause issues
3. **Long-running transactions** holding DB connections
4. **REQUIRES_NEW deadlocks** when accessing same resources

## What's Next (Part 3)

- Isolation Levels
- Transaction Timeouts
- Read-Only Transactions
- Rollback Rules
- Exception Handling in Transactions
- Distributed Transactions