# Comprehensive Notes: Singleton, Immutable, and Wrapper Classes in Java

## Overview - Types of Classes (Part 4 - Final)

| Topic | Coverage |
|-------|----------|
| ✓ Singleton Class | 6 different implementation approaches |
| ✓ Immutable Class | Rules and best practices |
| ✓ Wrapper Class | Brief overview (detailed in Variables video) |

---

## 1. Singleton Class

### Definition & Purpose

A **Singleton Class** ensures that **only ONE instance** of the class exists throughout the application lifecycle.

### Real-World Use Cases:

| Use Case | Why Singleton? |
|----------|---------------|
| **Database Connection** | Single connection shared across application |
| **Logger** | One logging instance for entire application |
| **Configuration Manager** | Single source of configuration |
| **Thread Pool** | Shared pool of threads |
| **Cache** | Single cache instance |

### Core Requirements for Singleton:

```
┌─────────────────────────────────────────────────────────┐
│              SINGLETON REQUIREMENTS                      │
├─────────────────────────────────────────────────────────┤
│  1. Private Constructor - Prevent external instantiation │
│  2. Private Static Instance - Store the single instance  │
│  3. Public Static Method - Provide global access point   │
└─────────────────────────────────────────────────────────┘
```

---

### Six Ways to Create Singleton

| # | Method | Thread Safe | Lazy | Performance | Recommended |
|---|--------|-------------|------|-------------|-------------|
| 1 | Eager Initialization | ✓ | ❌ | Fast | Simple cases |
| 2 | Lazy Initialization | ❌ | ✓ | Fast | Single-threaded only |
| 3 | Synchronized Method | ✓ | ✓ | Slow | ❌ Not recommended |
| 4 | Double-Checked Locking | ✓ | ✓ | Good | ✓ With volatile |
| 5 | Bill Pugh Solution | ✓ | ✓ | Fast | ✓ Best practice |
| 6 | Enum Singleton | ✓ | ✓ | Fast | ✓ Simplest & safest |

---

### 1.1 Eager Initialization

Object is created **at class loading time**, before anyone requests it.

```java
public class DBConnection {
    // 1. Private static instance - created immediately
    private static DBConnection conn = new DBConnection();
    
    // 2. Private constructor - prevents external instantiation
    private DBConnection() {
        // Initialize connection
    }
    
    // 3. Public static method - global access point
    public static DBConnection getInstance() {
        return conn;
    }
}
```

### How It Works:

```
Application Start
       │
       ▼
┌──────────────────────────────┐
│  Class Loading Phase         │
│  ───────────────────────     │
│  static variables loaded     │
│  conn = new DBConnection()   │ ◄── Object created HERE
└──────────────────────────────┘
       │
       ▼
┌──────────────────────────────┐
│  Later... getInstance()      │
│  ───────────────────────     │
│  return conn;                │ ◄── Just returns existing object
└──────────────────────────────┘
```

### Pros & Cons:

| Pros | Cons |
|------|------|
| ✓ Simple implementation | ❌ Object created even if never used |
| ✓ Thread-safe (JVM handles) | ❌ Wastes memory if unused |
| ✓ No synchronization needed | ❌ No exception handling during creation |

---

### 1.2 Lazy Initialization

Object is created **only when first requested**.

```java
public class DBConnection {
    // 1. Private static instance - NOT initialized yet
    private static DBConnection conn;
    
    // 2. Private constructor
    private DBConnection() {
        // Initialize connection
    }
    
    // 3. Public static method - creates instance on first call
    public static DBConnection getInstance() {
        if (conn == null) {           // Check if instance exists
            conn = new DBConnection(); // Create if not
        }
        return conn;
    }
}
```

### How It Works:

```
First Call to getInstance()
       │
       ▼
┌──────────────────────────────┐
│  conn == null ?              │
│       │                      │
│       ▼ YES                  │
│  conn = new DBConnection()   │ ◄── Object created on FIRST call
│  return conn                 │
└──────────────────────────────┘

Subsequent Calls
       │
       ▼
┌──────────────────────────────┐
│  conn == null ?              │
│       │                      │
│       ▼ NO                   │
│  return conn                 │ ◄── Returns existing object
└──────────────────────────────┘
```

### Pros & Cons:

| Pros | Cons |
|------|------|
| ✓ Object created only when needed | ❌ NOT thread-safe |
| ✓ Saves memory if never used | ❌ Multiple instances possible in multi-threaded |

### Thread Safety Problem:

```
Thread 1                          Thread 2
   │                                 │
   ▼                                 ▼
conn == null? YES                conn == null? YES
   │                                 │
   ▼                                 ▼
conn = new DBConnection()        conn = new DBConnection()
   │                                 │
   ▼                                 ▼
  Object 1 created               Object 2 created ❌ PROBLEM!
```

---

### 1.3 Synchronized Method

Entire method is synchronized to ensure thread safety.

```java
public class DBConnection {
    private static DBConnection conn;
    
    private DBConnection() { }
    
    // Synchronized method - only one thread can enter at a time
    public static synchronized DBConnection getInstance() {
        if (conn == null) {
            conn = new DBConnection();
        }
        return conn;
    }
}
```

### How Synchronization Works:

```
Thread 1                          Thread 2
   │                                 │
   ▼                                 │
┌─────────────────────┐              │
│ ACQUIRE LOCK        │              │
│ ─────────────────   │              │ (waiting for lock)
│ conn == null? YES   │              │
│ conn = new DB...    │              │
│ return conn         │              │
│ RELEASE LOCK        │              │
└─────────────────────┘              │
                                     ▼
                          ┌─────────────────────┐
                          │ ACQUIRE LOCK        │
                          │ ─────────────────   │
                          │ conn == null? NO    │
                          │ return conn         │
                          │ RELEASE LOCK        │
                          └─────────────────────┘
```

### Pros & Cons:

| Pros | Cons |
|------|------|
| ✓ Thread-safe | ❌ Very slow - lock acquired EVERY call |
| ✓ Lazy initialization | ❌ Lock/unlock overhead even when not needed |

### Why It's Slow:

```
Call 1: Lock → Check → Create → Unlock
Call 2: Lock → Check → Return → Unlock  ← Unnecessary locking!
Call 3: Lock → Check → Return → Unlock  ← Unnecessary locking!
...
Call 1000: Lock → Check → Return → Unlock  ← Still locking!
```

> After object is created, synchronization is unnecessary but still happens every time.

---

### 1.4 Double-Checked Locking (DCL)

Synchronizes only the critical section, with two null checks.

```java
public class DBConnection {
    // VOLATILE is crucial here!
    private static volatile DBConnection conn;
    
    private DBConnection() { }
    
    public static DBConnection getInstance() {
        if (conn == null) {                    // First check (no lock)
            synchronized (DBConnection.class) { // Lock only if null
                if (conn == null) {            // Second check (with lock)
                    conn = new DBConnection();
                }
            }
        }
        return conn;
    }
}
```

### How Double-Check Works:

```
Thread 1                              Thread 2
   │                                     │
   ▼                                     ▼
conn == null? YES                   conn == null? YES
   │                                     │
   ▼                                     │
┌─────────────────────┐                  │
│ ACQUIRE LOCK        │                  │ (waiting)
│ conn == null? YES   │ ◄─ 2nd check     │
│ conn = new DB...    │                  │
│ RELEASE LOCK        │                  │
└─────────────────────┘                  │
                                         ▼
                              ┌─────────────────────┐
                              │ ACQUIRE LOCK        │
                              │ conn == null? NO    │ ◄─ 2nd check fails
                              │ RELEASE LOCK        │
                              └─────────────────────┘
                                         │
                                         ▼
                                    return conn

Subsequent Calls (Both Threads):
   │
   ▼
conn == null? NO  ◄─ First check fails, no locking needed!
   │
   ▼
return conn
```

### Why TWO Checks?

| Check | Purpose |
|-------|---------|
| **First Check** (outside lock) | Avoid unnecessary locking when object exists |
| **Second Check** (inside lock) | Handle race condition when two threads pass first check |

---

### Why VOLATILE is Critical

### The Memory Visibility Problem:

```
CPU Architecture:
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  ┌─────────┐           ┌─────────┐                      │
│  │ Core 1  │           │ Core 2  │                      │
│  │┌───────┐│           │┌───────┐│                      │
│  ││L1 Cache││           ││L1 Cache││                      │
│  │└───────┘│           │└───────┘│                      │
│  └────┬────┘           └────┬────┘                      │
│       │                     │                           │
│       └──────────┬──────────┘                           │
│                  │                                      │
│           ┌──────┴──────┐                               │
│           │ Main Memory │                               │
│           │ conn = null │                               │
│           └─────────────┘                               │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

### Problem Scenario WITHOUT Volatile:

```
Thread 1 (Core 1)                    Thread 2 (Core 2)
─────────────────                    ─────────────────
1. conn == null? YES
2. Acquire lock
3. conn == null? YES
4. conn = new DBConnection()
   ↓
   Stored in L1 Cache (Core 1)      
   NOT yet synced to Main Memory!   5. conn == null? 
                                       ↓
                                       Checks L1 Cache (Core 2) - no info
                                       Checks Main Memory - still null!
                                       ↓
                                    6. conn == null? YES ❌
                                    7. Creates ANOTHER object!
```

### Solution: Volatile Keyword

```java
private static volatile DBConnection conn;
```

**Volatile guarantees:**
1. **Visibility:** All reads/writes go directly to main memory
2. **No Caching:** Bypasses CPU cache for this variable
3. **Happens-Before:** Ensures proper ordering of operations

```
With Volatile:
─────────────────
Thread 1 writes conn → Immediately visible in Main Memory
Thread 2 reads conn → Always reads from Main Memory
```

### Pros & Cons of DCL:

| Pros | Cons |
|------|------|
| ✓ Thread-safe | Requires volatile (slight overhead) |
| ✓ Lazy initialization | More complex code |
| ✓ Fast after initialization | |
| ✓ Lock only when needed | |

---

### 1.5 Bill Pugh Solution (Initialization-on-Demand Holder)

Uses a **static inner class** to achieve lazy initialization without synchronization.

```java
public class DBConnection {
    
    // Private constructor
    private DBConnection() { }
    
    // Static inner class - not loaded until referenced
    private static class ConnectionHolder {
        private static final DBConnection INSTANCE = new DBConnection();
    }
    
    // Public method to get instance
    public static DBConnection getInstance() {
        return ConnectionHolder.INSTANCE;
    }
}
```

### How It Works:

```
Application Start
       │
       ▼
┌──────────────────────────────────────┐
│ DBConnection class loaded            │
│ BUT ConnectionHolder NOT loaded yet  │ ◄── Inner class loads lazily
└──────────────────────────────────────┘
       │
       ▼
First call to getInstance()
       │
       ▼
┌──────────────────────────────────────┐
│ ConnectionHolder class loaded NOW    │
│ INSTANCE = new DBConnection()        │ ◄── Object created HERE
└──────────────────────────────────────┘
       │
       ▼
Return INSTANCE
```

### Why This Works:

| Property | Explanation |
|----------|-------------|
| **Lazy Loading** | Inner class loaded only when `getInstance()` called |
| **Thread Safety** | JVM guarantees class loading is thread-safe |
| **No Synchronization** | JVM handles thread safety during class initialization |
| **No Volatile** | Static final initialization is inherently safe |

### Pros & Cons:

| Pros | Cons |
|------|------|
| ✓ Thread-safe (JVM guaranteed) | Slightly more complex structure |
| ✓ Lazy initialization | |
| ✓ No synchronization overhead | |
| ✓ No volatile needed | |
| ✓ Best performance | |

> **Bill Pugh Solution is considered the BEST approach for most cases**

---

### 1.6 Enum Singleton

Simplest and safest way to implement singleton.

```java
public enum DBConnection {
    INSTANCE;  // Single instance
    
    // Instance variables
    private Connection connection;
    
    // Constructor (implicitly private in enum)
    DBConnection() {
        // Initialize connection
        connection = createConnection();
    }
    
    // Instance methods
    public Connection getConnection() {
        return connection;
    }
    
    private Connection createConnection() {
        // Create and return connection
        return null;
    }
}

// Usage
DBConnection.INSTANCE.getConnection();
```

### Why Enum Works:

| Property | Explanation |
|----------|-------------|
| **Constructor** | Always private by default |
| **Single Instance** | JVM guarantees only one instance per JVM |
| **Serialization Safe** | Enum serialization handled specially |
| **Reflection Safe** | Cannot create enum instance via reflection |
| **Thread Safe** | JVM handles enum initialization |

### Pros & Cons:

| Pros | Cons |
|------|------|
| ✓ Simplest code (2 lines!) | Cannot extend other classes |
| ✓ Thread-safe | Less flexible |
| ✓ Serialization safe | Cannot do lazy initialization |
| ✓ Reflection safe | |
| ✓ JVM guaranteed singleton | |

---

### Singleton Comparison Summary

```
                    ┌─────────────────────────────────────────────────┐
                    │         SINGLETON IMPLEMENTATIONS               │
                    └─────────────────────────────────────────────────┘
                    
┌─────────────────┬──────────────┬──────────┬─────────┬───────────────┐
│    Method       │ Thread-Safe  │  Lazy    │ Speed   │  Complexity   │
├─────────────────┼──────────────┼──────────┼─────────┼───────────────┤
│ Eager           │     ✓        │    ❌    │  Fast   │    Simple     │
├─────────────────┼──────────────┼──────────┼─────────┼───────────────┤
│ Lazy            │     ❌       │    ✓     │  Fast   │    Simple     │
├─────────────────┼──────────────┼──────────┼─────────┼───────────────┤
│ Synchronized    │     ✓        │    ✓     │  Slow   │    Simple     │
├─────────────────┼──────────────┼──────────┼─────────┼───────────────┤
│ Double-Check    │     ✓*       │    ✓     │  Good   │    Medium     │
├─────────────────┼──────────────┼──────────┼─────────┼───────────────┤
│ Bill Pugh       │     ✓        │    ✓     │  Fast   │    Medium     │
├─────────────────┼──────────────┼──────────┼─────────┼───────────────┤
│ Enum            │     ✓        │    ✓     │  Fast   │    Simplest   │
└─────────────────┴──────────────┴──────────┴─────────┴───────────────┘

* Requires volatile keyword
```

---

## 2. Immutable Class

### Definition

An **Immutable Class** is a class whose instances **cannot be modified** after creation. Once an object is created, its state remains constant.

### Examples in Java:

- `String`
- `Integer`, `Double`, `Long` (all wrapper classes)
- `LocalDate`, `LocalTime`, `LocalDateTime`
- `BigInteger`, `BigDecimal`

### Rules for Creating Immutable Class

| # | Rule | Purpose |
|---|------|---------|
| 1 | Declare class as `final` | Prevent subclasses from overriding methods |
| 2 | Make all fields `private final` | Prevent direct access and reassignment |
| 3 | Initialize via constructor only | Set state once during creation |
| 4 | No setter methods | Prevent state modification |
| 5 | Return copies of mutable fields | Prevent external modification |

---

### Implementation Example

```java
public final class ImmutableClass {
    
    // Private final fields
    private final String name;
    private final List<String> petNames;
    
    // Constructor - only way to set values
    public ImmutableClass(String name, List<String> petNames) {
        this.name = name;
        // Create defensive copy of mutable object
        this.petNames = new ArrayList<>(petNames);
    }
    
    // Getter for immutable field - safe to return directly
    public String getName() {
        return name;  // String is immutable, safe to return
    }
    
    // Getter for mutable field - MUST return copy
    public List<String> getPetNames() {
        return new ArrayList<>(petNames);  // Return defensive copy
    }
    
    // NO SETTER METHODS!
}
```

### Why Each Rule Matters:

#### Rule 1: Class must be `final`

```java
// Without final - PROBLEM:
public class ImmutableClass {
    private final int value;
    // ...
}

class MaliciousSubclass extends ImmutableClass {
    private int mutableValue;
    
    @Override
    public int getValue() {
        return mutableValue++;  // Can change return value!
    }
}
```

#### Rule 2 & 3: Private final fields, constructor initialization

```java
public final class ImmutableClass {
    private final String name;  // Cannot be reassigned after construction
    
    public ImmutableClass(String name) {
        this.name = name;  // Only assignment point
    }
}
```

#### Rule 4: No setters

```java
// WRONG - has setter
public void setName(String name) {
    this.name = name;  // ❌ Allows modification
}

// CORRECT - no setter
// Simply don't include any setName method
```

#### Rule 5: Return copies of mutable fields (CRITICAL!)

### The Mutable Field Problem:

```java
// WRONG Implementation
public final class BadImmutable {
    private final List<String> items;
    
    public BadImmutable(List<String> items) {
        this.items = items;  // ❌ Stores reference to external list
    }
    
    public List<String> getItems() {
        return items;  // ❌ Returns direct reference
    }
}
```

```
┌─────────────────────────────────────────────────────────────────┐
│                      PROBLEM SCENARIO                           │
└─────────────────────────────────────────────────────────────────┘

List<String> originalList = new ArrayList<>();
originalList.add("A");
originalList.add("B");

BadImmutable obj = new BadImmutable(originalList);

                    ┌─────────────────┐
                    │  BadImmutable   │
                    │  ─────────────  │
originalList ──────►│  items ────────┼────► [A, B]
                    └─────────────────┘        ▲
                                               │
originalList.add("C");  // Modifies "immutable" object!
                                               │
                    ┌─────────────────┐        │
                    │  BadImmutable   │        │
                    │  ─────────────  │        │
                    │  items ────────┼────► [A, B, C]  ← MODIFIED!
                    └─────────────────┘

obj.getItems().add("D");  // Also modifies!
```

### Correct Implementation (Defensive Copying):

```java
public final class GoodImmutable {
    private final List<String> items;
    
    public GoodImmutable(List<String> items) {
        // Defensive copy on input
        this.items = new ArrayList<>(items);
    }
    
    public List<String> getItems() {
        // Defensive copy on output
        return new ArrayList<>(items);
    }
}
```

```
┌─────────────────────────────────────────────────────────────────┐
│                      CORRECT SCENARIO                           │
└─────────────────────────────────────────────────────────────────┘

List<String> originalList = new ArrayList<>();
originalList.add("A");
originalList.add("B");

GoodImmutable obj = new GoodImmutable(originalList);

originalList ──────► [A, B]     (original list)
                         
                    ┌─────────────────┐
                    │  GoodImmutable  │
                    │  ─────────────  │
                    │  items ────────┼────► [A, B]  (separate copy)
                    └─────────────────┘

originalList.add("C");  
originalList ──────► [A, B, C]  (only original modified)
                    
                    ┌─────────────────┐
                    │  GoodImmutable  │
                    │  ─────────────  │
                    │  items ────────┼────► [A, B]  (still unchanged!)
                    └─────────────────┘

List<String> retrieved = obj.getItems();  // Returns NEW copy
retrieved ──────► [A, B]  (separate copy)

retrieved.add("D");  
retrieved ──────► [A, B, D]  (only copy modified)

                    ┌─────────────────┐
                    │  GoodImmutable  │
                    │  ─────────────  │
                    │  items ────────┼────► [A, B]  (still unchanged!)
                    └─────────────────┘
```

---

### Complete Immutable Class Example

```java
import java.util.ArrayList;
import java.util.List;

public final class Person {
    
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    public Person(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // Defensive copy for mutable object
        this.hobbies = new ArrayList<>(hobbies);
    }
    
    // Getters only - no setters
    
    public String getName() {
        return name;  // String is immutable, safe
    }
    
    public int getAge() {
        return age;  // Primitive, safe
    }
    
    public List<String> getHobbies() {
        // Return defensive copy
        return new ArrayList<>(hobbies);
    }
    
    // If you need a modified version, return NEW object
    public Person withName(String newName) {
        return new Person(newName, this.age, this.hobbies);
    }
    
    public Person withAge(int newAge) {
        return new Person(this.name, newAge, this.hobbies);
    }
}
```

### Usage:

```java
List<String> hobbies = new ArrayList<>();
hobbies.add("Reading");
hobbies.add("Gaming");

Person person = new Person("John", 25, hobbies);

// Original list modification doesn't affect Person
hobbies.add("Swimming");
System.out.println(person.getHobbies());  // [Reading, Gaming]

// Retrieved list modification doesn't affect Person
List<String> retrieved = person.getHobbies();
retrieved.add("Cooking");
System.out.println(person.getHobbies());  // Still [Reading, Gaming]

// To "modify", create new object
Person olderPerson = person.withAge(26);
System.out.println(person.getAge());      // 25 (original unchanged)
System.out.println(olderPerson.getAge()); // 26 (new object)
```

---

### Immutable Class Checklist

```
┌─────────────────────────────────────────────────────────────┐
│                 IMMUTABLE CLASS CHECKLIST                    │
├─────────────────────────────────────────────────────────────┤
│  □ Class declared as final                                  │
│  □ All fields are private                                   │
│  □ All fields are final                                     │
│  □ No setter methods                                        │
│  □ Defensive copy in constructor for mutable fields         │
│  □ Defensive copy in getters for mutable fields             │
│  □ If class has mutable object fields, they are also        │
│    immutable or defensively copied                          │
└─────────────────────────────────────────────────────────────┘
```

### Benefits of Immutable Classes:

| Benefit | Explanation |
|---------|-------------|
| **Thread Safety** | No synchronization needed - state never changes |
| **Cache Friendly** | Safe to cache and reuse |
| **Hash Key Safe** | Can be used as HashMap keys reliably |
| **Simple** | No need to track state changes |
| **Predictable** | Behavior is consistent |

---

## 3. Wrapper Classes (Brief Overview)

### Definition

**Wrapper Classes** provide object representations for the 8 primitive types.

### Primitive to Wrapper Mapping:

| Primitive | Wrapper Class | Example |
|-----------|---------------|---------|
| `byte` | `Byte` | `Byte b = 10;` |
| `short` | `Short` | `Short s = 100;` |
| `int` | `Integer` | `Integer i = 1000;` |
| `long` | `Long` | `Long l = 10000L;` |
| `float` | `Float` | `Float f = 3.14f;` |
| `double` | `Double` | `Double d = 3.14159;` |
| `char` | `Character` | `Character c = 'A';` |
| `boolean` | `Boolean` | `Boolean bool = true;` |

### Autoboxing and Unboxing:

```java
// Autoboxing: primitive → Wrapper (automatic)
int primitive = 10;
Integer wrapper = primitive;  // Autoboxing

// Unboxing: Wrapper → primitive (automatic)
Integer wrapper2 = 20;
int primitive2 = wrapper2;    // Unboxing
```

### Why Wrapper Classes?

| Reason | Explanation |
|--------|-------------|
| **Collections** | Collections can only store objects, not primitives |
| **Null Values** | Primitives can't be null, wrappers can |
| **Utility Methods** | `Integer.parseInt()`, `Double.valueOf()`, etc. |
| **Generics** | `List<Integer>` works, `List<int>` doesn't |

> **Note:** Detailed coverage of Wrapper Classes is in "Java Variables Part 2" video

---

## 4. Quick Reference Summary

### Singleton Quick Reference:

```java
// Best Practice: Bill Pugh Solution
public class Singleton {
    private Singleton() { }
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

// Alternative: Enum (Simplest)
public enum Singleton {
    INSTANCE;
}
```

### Immutable Quick Reference:

```java
public final class Immutable {
    private final String value;
    private final List<String> list;
    
    public Immutable(String value, List<String> list) {
        this.value = value;
        this.list = new ArrayList<>(list);  // Defensive copy
    }
    
    public String getValue() { return value; }
    public List<String> getList() { return new ArrayList<>(list); }  // Copy
}
```

---

## 5. Interview Tips

### Singleton Questions:

1. **"Implement a thread-safe singleton"** → Use Bill Pugh or Double-Checked Locking with volatile
2. **"Why is volatile needed in DCL?"** → Memory visibility across CPU cores/caches
3. **"Best way to implement singleton?"** → Bill Pugh for flexibility, Enum for simplicity
4. **"How to break singleton?"** → Reflection, Serialization, Cloning (Enum prevents all)

### Immutable Questions:

1. **"How to create immutable class?"** → List the 5 rules
2. **"Why return copy in getter?"** → Prevent external modification of internal state
3. **"Is String immutable? Why?"** → Yes, for security, caching, thread-safety
4. **"What's defensive copying?"** → Creating copies of mutable objects on input/output

### Common Mistakes to Avoid:

| Mistake | Correction |
|---------|------------|
| DCL without volatile | Always use volatile with DCL |
| Returning mutable field directly | Return defensive copy |
| Forgetting to make class final | Subclass can break immutability |
| Using synchronized method singleton | Use Bill Pugh or Enum instead |