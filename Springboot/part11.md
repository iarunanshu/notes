# Spring Boot @Transactional Annotation - Part 1

## Prerequisites
1. **Concurrency Control** (HLD playlist)
2. **Spring AOP** (Aspect Oriented Programming)

## Critical Section

**Definition**: Code segment where shared resources are accessed and modified

### Problem Example - Cab Booking
```
Database: Car (ID: 1001, Status: AVAILABLE)

4 parallel requests:
Request 1: Read car 1001 → Status available → Book
Request 2: Read car 1001 → Status available → Book  
Request 3: Read car 1001 → Status available → Book
Request 4: Read car 1001 → Status available → Book

Result: Data inconsistency - 1 car booked 4 times!
```

**Solution**: Use Transactions

## ACID Properties

Transactions guarantee ACID properties:

### 1. **Atomicity**
- All operations in transaction succeed or all fail
- If any operation fails → entire transaction rolls back

**Example**:
```
Initial State: A = ₹10, B = ₹20

Transaction:
  Operation 1: Debit A ₹5  → A = ₹5  ✓
  Operation 2: Credit B ₹5 → FAILS   ✗
  
Result: Rollback Operation 1 → A = ₹10 (back to original)
```

### 2. **Consistency**
- Database remains consistent before and after transaction
- No partial updates allowed

```
Before: A = ₹10, B = ₹20 (Consistent)
After Success: A = ₹5, B = ₹25 (Consistent)
After Failure: A = ₹10, B = ₹20 (Consistent - rolled back)
```

### 3. **Isolation**
- Multiple transactions run in parallel without interference
- Each transaction feels like it's running alone
- Uses locking mechanisms internally

```
Transaction 1: Debit A ₹5, Credit B ₹5
Transaction 2: Debit A ₹2, Credit B ₹2  
Transaction 3: Credit A ₹3, Debit B ₹3

All run "in parallel" but internally sequenced via locks
```

### 4. **Durability**
- Once committed, data persists even if system crashes
- Data is permanently stored

## Traditional Transaction Management

### Without Spring Boot
```java
public void transferMoney() {
    // Boilerplate start
    beginTransaction();
    
    try {
        // Your actual business logic
        debitFromA(5);
        creditToB(5);
        
        // Boilerplate
        commitTransaction();
    } catch(Exception e) {
        // Boilerplate
        rollbackTransaction();
    }
    
    // Boilerplate
    endTransaction();
}
```

**Problems**:
- Lots of boilerplate code
- Duplicate code in every method
- Business logic mixed with transaction management
- If you have 1000 DB methods → 1000 times this boilerplate!

## @Transactional in Spring Boot

### Solution: @Transactional Annotation
```java
@Transactional
public void transferMoney() {
    // ONLY business logic!
    debitFromA(5);
    creditToB(5);
}
```

Spring Boot handles all transaction management automatically!

## Setup

### 1. Dependencies (pom.xml)

**For Relational DB (JPA)**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Database Driver (example: MySQL) -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

**For NoSQL** (if supports transactions):
- Use appropriate NoSQL dependency

### 2. Configuration

**application.properties**:
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

### 3. Enable Transaction Management (Optional)
```java
@SpringBootApplication
@EnableTransactionManagement  // Usually auto-configured
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

## @Transactional Usage Levels

### 1. Class Level
```java
@Service
@Transactional  // Applies to ALL public methods
public class UserService {
    
    public void updateUser() {  // Transactional
        // DB operations
    }
    
    public void deleteUser() {  // Transactional
        // DB operations
    }
    
    private void helperMethod() {  // NOT transactional (private)
        // Some logic
    }
}
```

### 2. Method Level
```java
@Service
public class UserService {
    
    @Transactional  // Only this method
    public void updateUser() {
        // DB operations
    }
    
    public void getUser() {  // NOT transactional
        // Read operations
    }
}
```

## How @Transactional Works Internally

### Uses Spring AOP (Aspect Oriented Programming)

**Remember from AOP**:
- **Pointcut**: WHERE to apply advice
- **Advice**: WHAT code to run
- **Join Point**: Actual method invocation

### Internal Working

#### 1. Pointcut Expression (Simplified)
```java
@within(org.springframework.transaction.annotation.Transactional)
```
Matches all methods with @Transactional annotation

#### 2. Advice Type: AROUND
- Wraps method execution
- Can control before AND after

#### 3. Advice Implementation
Located in: `TransactionInterceptor` class

```java
// Simplified version of what happens internally
public Object invokeWithinTransaction(Method method, ...) {
    // 1. BEGIN TRANSACTION
    TransactionStatus transaction = beginTransaction();
    
    try {
        // 2. INVOKE YOUR METHOD (Join Point)
        Object result = method.invoke(...);
        
        // 3. COMMIT IF SUCCESS
        commitTransaction(transaction);
        return result;
        
    } catch (Exception e) {
        // 4. ROLLBACK IF FAILURE
        rollbackTransaction(transaction);
        throw e;
    }
}
```

## Complete Working Example

### Controller
```java
@RestController
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping("/api/updateUser")  // Should be @PostMapping
    public String updateUser() {
        return userService.updateUser();
    }
}
```

### Service with @Transactional
```java
@Component
public class UserService {
    
    @Transactional
    public String updateUser() {
        // Your business logic
        // Update DB operations
        
        // If you throw exception:
        // throw new RuntimeException("Something failed!");
        // Transaction will rollback
        
        return "User updated successfully";
    }
}
```

## Execution Flow

### Success Scenario
1. API called: `/api/updateUser`
2. Spring intercepts via AOP
3. `TransactionInterceptor.invokeWithinTransaction()` called
4. Creates transaction
5. Invokes `updateUser()` method
6. Method executes successfully
7. Transaction commits
8. Returns response

### Failure Scenario
1. API called: `/api/updateUser`
2. Spring intercepts via AOP
3. `TransactionInterceptor.invokeWithinTransaction()` called
4. Creates transaction
5. Invokes `updateUser()` method
6. Method throws exception
7. Exception caught
8. Transaction rolls back
9. Exception propagated

## Internal Classes Involved

1. **TransactionInterceptor** - Main interceptor class
2. **TransactionAspectSupport** - Parent class with transaction logic
3. **PlatformTransactionManager** - Interface for transaction management
4. **TransactionStatus** - Represents current transaction

## Key Benefits

1. **Clean Code**: Business logic separated from transaction management
2. **No Duplication**: Write once, use everywhere
3. **Automatic Management**: Spring handles begin/commit/rollback
4. **Declarative**: Just declare with annotation
5. **AOP Power**: Leverages aspect-oriented programming

## Key Points to Remember

1. **@Transactional uses AOP internally**
2. **Only works on public methods** (not private)
3. **Creates proxy classes** for transactional beans
4. **Automatically handles**:
    - Begin transaction
    - Commit on success
    - Rollback on exception
    - End transaction

## What's Coming Next (Part 2)

1. **Transaction Context**
2. **Transaction Managers** (Programmatic vs Declarative)
3. **Propagation Levels** (REQUIRED, REQUIRES_NEW, etc.)
4. **Isolation Levels** (READ_COMMITTED, etc.)
5. **Transaction Timeout**
6. **Read-Only Transactions**
7. **Rollback Rules**
8. **Nested Transactions**

## Interview Key Points

1. **Why use @Transactional?**
    - Ensures ACID properties
    - Removes boilerplate code
    - Declarative transaction management

2. **How does it work?**
    - Uses Spring AOP
    - Creates proxy classes
    - Intercepts method calls
    - Manages transaction lifecycle

3. **When to use?**
    - Any method that modifies database
    - Operations requiring atomicity
    - Financial transactions
    - Critical data updates

4. **Limitations?**
    - Only works on public methods
    - Self-invocation doesn't work (calling method within same class)
    - Requires Spring-managed beans