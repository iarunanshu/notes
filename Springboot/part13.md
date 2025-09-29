# Spring Boot @Transactional Annotation - Part 3: Isolation Levels

## What are Isolation Levels?

**Definition**: Isolation levels define how changes made by one transaction are visible to other transactions running in parallel.

### Using Isolation in Spring Boot
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void updateUser() {
    // Your business logic
}
```

### Default Isolation Level
- **Depends on the database** being used
- Most relational databases default to READ_COMMITTED
- Some might use REPEATABLE_READ
- Always check your specific database documentation (PostgreSQL, MySQL, etc.)

## Concurrency vs Problems Trade-off

| Isolation Level | Concurrency | Problems |
|----------------|-------------|----------|
| READ_UNCOMMITTED | Very High | All 3 problems |
| READ_COMMITTED | High | 2 problems |
| REPEATABLE_READ | Medium | 1 problem |
| SERIALIZABLE | Low | No problems |

## Three Core Problems

### 1. Dirty Read Problem

**Definition**: Transaction A reads uncommitted data from Transaction B. If Transaction B rolls back, Transaction A has read data that never existed.

**Example Timeline**:
```
Time T1: Transaction A and B start
         DB State: ID=123, Status=FREE

Time T2: Transaction B updates ID=123 to Status=BOOKED (not committed)
         DB State: ID=123, Status=BOOKED (uncommitted)

Time T3: Transaction A reads ID=123
         Result: Status=BOOKED (reading uncommitted data)

Time T4: Transaction B rolls back
         DB State: ID=123, Status=FREE

Problem: Transaction A has read BOOKED which never actually existed!
```

### 2. Non-Repeatable Read Problem

**Definition**: Transaction A reads the same row multiple times and gets different values.

**Example Timeline**:
```
Time T1: Transaction A starts
         DB State: ID=1, Status=FREE

Time T2: Transaction A reads ID=1
         Result: Status=FREE

Time T3: Transaction B updates ID=1 to BOOKED and commits
         DB State: ID=1, Status=BOOKED

Time T4: Transaction A reads ID=1 again
         Result: Status=BOOKED (different from T2!)

Problem: Same query, different results within same transaction
```

### 3. Phantom Read Problem

**Definition**: Transaction A executes the same query multiple times but gets different number of rows.

**Example Timeline**:
```
Time T1: Transaction A starts
         DB State: ID=1 (FREE), ID=3 (BOOKED)

Time T2: Transaction A reads WHERE ID > 0 AND ID < 5
         Result: 2 rows (ID=1, ID=3)

Time T3: Transaction B inserts ID=2 (FREE) and commits
         DB State: ID=1, ID=2, ID=3

Time T4: Transaction A reads WHERE ID > 0 AND ID < 5
         Result: 3 rows (ID=1, ID=2, ID=3)

Problem: Same query, different number of rows!
```

## Database Locking Mechanisms

### 1. Shared Lock (S) / Read Lock
- Multiple transactions can acquire shared lock simultaneously
- Only allows reading, no modifications
- Represented as 'S'

**Example**:
```
Transaction T1: Takes shared lock on Row 1 → Can read
Transaction T2: Takes shared lock on Row 1 → Can also read
Both can read simultaneously!
```

### 2. Exclusive Lock (X) / Write Lock
- Only one transaction can acquire exclusive lock
- No other transaction can read or write
- Represented as 'X'

**Example**:
```
Transaction T1: Takes exclusive lock on Row 1 → Can read/write
Transaction T2: Wants to read/write Row 1 → BLOCKED!
```

### Lock Compatibility Matrix

| Current Lock | Can Take Shared? | Can Take Exclusive? |
|--------------|------------------|-------------------|
| None | Yes | Yes |
| Shared | Yes | No |
| Exclusive | No | No |

## Four Isolation Levels Explained

### 1. READ UNCOMMITTED

**Characteristics**:
- No locks acquired (neither read nor write)
- Highest concurrency
- All three problems present

**Problems**:
- ✗ Dirty Read
- ✗ Non-Repeatable Read
- ✗ Phantom Read

**When to use**:
- Read-only applications with static data
- When data accuracy is not critical
- Maximum performance needed

**How it works**:
```
Transaction T1: Reads without lock
Transaction T2: Writes without waiting
Result: No blocking, but all problems possible
```

### 2. READ COMMITTED

**Characteristics**:
- **Read**: Acquires shared lock, releases immediately after reading
- **Write**: Acquires exclusive lock, keeps until transaction ends

**Problems**:
- ✓ Dirty Read (Solved)
- ✗ Non-Repeatable Read
- ✗ Phantom Read

**How it works**:
```
Read Operation:
1. Acquire shared lock
2. Read data
3. Release lock immediately

Write Operation:
1. Acquire exclusive lock
2. Modify data
3. Keep lock until commit/rollback
```

**Example - Why Non-Repeatable Read still occurs**:
```
Time T1: Transaction A reads ID=1 (takes lock, reads FREE, releases lock)
Time T2: Transaction B updates ID=1 to BOOKED (can take exclusive lock now)
Time T3: Transaction A reads ID=1 again (gets BOOKED - different value!)
```

### 3. REPEATABLE READ

**Characteristics**:
- **Read**: Acquires shared lock, keeps until transaction ends
- **Write**: Acquires exclusive lock, keeps until transaction ends

**Problems**:
- ✓ Dirty Read (Solved)
- ✓ Non-Repeatable Read (Solved)
- ✗ Phantom Read

**How it works**:
```
Read Operation:
1. Acquire shared lock
2. Read data
3. Keep lock until commit/rollback

Write Operation:
1. Acquire exclusive lock
2. Modify data
3. Keep lock until commit/rollback
```

**Example - Why Phantom Read still occurs**:
```
Time T1: Transaction A reads ID>0 AND ID<5 (locks rows 1 and 4)
Time T2: Transaction B inserts ID=2 (new row, no lock exists)
Time T3: Transaction A reads ID>0 AND ID<5 (now gets 3 rows!)
```

### 4. SERIALIZABLE

**Characteristics**:
- Same locking as REPEATABLE READ
- **Plus Range Locks** to prevent insertions

**Problems**:
- ✓ Dirty Read (Solved)
- ✓ Non-Repeatable Read (Solved)
- ✓ Phantom Read (Solved)

**How it works**:
```
Range Lock Example:
Query: WHERE ID > 0 AND ID < 5
Locks: All existing rows + prevents new insertions in range

Transaction T1: Reads WHERE ID > 0 AND ID < 5
                Locks rows 1, 3 AND range 0-5
Transaction T2: Tries to insert ID=2 → BLOCKED by range lock!
```

## Isolation Levels Summary Table

| Isolation Level | Dirty Read | Non-Repeatable Read | Phantom Read | Use Case |
|----------------|------------|-------------------|--------------|----------|
| READ_UNCOMMITTED | Possible | Possible | Possible | Read-only apps |
| READ_COMMITTED | Prevented | Possible | Possible | Most common |
| REPEATABLE_READ | Prevented | Prevented | Possible | Financial apps |
| SERIALIZABLE | Prevented | Prevented | Prevented | Critical consistency |

## Practical Implementation in Spring Boot

### Setting Isolation Level
```java
@Service
public class UserService {
    
    // Default isolation (database dependent)
    @Transactional
    public void method1() { }
    
    // Specific isolation level
    @Transactional(isolation = Isolation.READ_COMMITTED)
    public void method2() { }
    
    // For critical operations
    @Transactional(isolation = Isolation.SERIALIZABLE)
    public void criticalFinancialOperation() { }
}
```

### Programmatic Approach
```java
DefaultTransactionDefinition def = new DefaultTransactionDefinition();
def.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
TransactionStatus status = transactionManager.getTransaction(def);
```

## Where Does Locking Happen?

**Important**: Locking is handled by the **database transaction manager**, not your application code.

```java
// Your code - Simple query
SELECT * FROM users WHERE id = 1;

// Database internally (based on isolation level):
// 1. Acquires appropriate lock (shared/exclusive)
// 2. Executes query
// 3. Releases lock (timing depends on isolation level)
```

## Interview Questions & Answers

### Q1: Which isolation level should we use?
**Answer**: Depends on requirements:
- If non-repeatable reads are acceptable → READ_COMMITTED
- If consistency is critical → REPEATABLE_READ
- Ask interviewer about specific use case

### Q2: Why does READ_UNCOMMITTED exist if it has all problems?
**Answer**:
- Useful for read-only applications
- Maximum performance when accuracy isn't critical
- Analytics on historical data

### Q3: Trade-offs between isolation levels?
**Answer**:
- Higher isolation = Better consistency, Lower concurrency
- Lower isolation = Higher concurrency, More anomalies
- Choose based on business requirements

### Q4: Default isolation in Spring Boot?
**Answer**: Depends on database:
- PostgreSQL: READ_COMMITTED
- MySQL: REPEATABLE_READ
- Oracle: READ_COMMITTED

## Best Practices

1. **Use READ_COMMITTED or REPEATABLE_READ** for most applications
2. **Avoid READ_UNCOMMITTED** unless read-only
3. **Use SERIALIZABLE sparingly** - only for critical sections
4. **Test with concurrent load** to identify issues
5. **Monitor deadlocks** with higher isolation levels
6. **Keep transactions short** to minimize lock duration

## Common Pitfalls

1. **Over-using SERIALIZABLE** - Causes performance issues
2. **Ignoring database defaults** - Know your DB's default
3. **Long transactions with high isolation** - Blocks other transactions
4. **Not handling deadlocks** - Higher isolation increases deadlock risk

## Key Takeaways

1. **Isolation levels balance consistency vs concurrency**
2. **Higher isolation = More locks = Less concurrency**
3. **Choose based on business requirements**, not performance alone
4. **Most applications work well with READ_COMMITTED**
5. **Database handles locking**, not application code
6. **Test thoroughly** with concurrent scenarios