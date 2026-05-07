# Comprehensive Notes: POJO, Enum, and Final Classes in Java

## Overview - Types of Classes (Part 3)

| Covered in Part 3 | Covered in Part 4 |
|-------------------|-------------------|
| ✓ POJO Classes | Singleton Class |
| ✓ Enum Classes | Immutable Class |
| ✓ Final Class | Wrapper Class |

---

## 1. POJO Classes (Plain Old Java Object)

### Definition

**POJO** stands for **Plain Old Java Object** - a simple Java class with minimal restrictions and no special requirements.

### Characteristics of POJO

| Property | Requirement |
|----------|-------------|
| **Class Modifier** | Must be `public` |
| **Constructor** | Must have `public` default constructor |
| **Variables** | Can have any access modifier |
| **Methods** | Contains getter and setter methods |
| **Annotations** | ❌ No annotations (like `@Entity`, `@Table`) |
| **Inheritance** | ❌ Does NOT extend any class |
| **Interface** | ❌ Does NOT implement any interface |

### POJO Example:

```java
public class Student {
    // Variables - can have any access modifier
    int id;                    // default
    private String name;       // private
    protected String email;    // protected
    public int age;            // public
    
    // Public default constructor (implicitly present)
    public Student() { }
    
    // Getter and Setter methods
    public int getId() {
        return id;
    }
    
    public void setId(int id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    // ... more getters and setters
}
```

### What Makes It NOT a POJO:

```java
// ❌ NOT a POJO - has annotation
@Entity
public class Student { }

// ❌ NOT a POJO - extends a class
public class Student extends Person { }

// ❌ NOT a POJO - implements interface
public class Student implements Serializable { }
```

### Real-World Use Cases for POJO

#### Use Case 1: Request/Response Mapping

```
┌──────────┐     Request      ┌─────────────────────────────────────┐
│  Client  │ ───────────────► │           Component                 │
└──────────┘   { id, name }   │  ┌─────────┐                        │
                              │  │  POJO   │──► Class1              │
                              │  │Mapping  │──► Class2              │
                              │  └─────────┘──► Class3              │
                              └─────────────────────────────────────┘
```

**Why use POJO for mapping?**
- Decouples internal code from external API structure
- If request format changes (e.g., `id` → `customer_id`), only mapping changes
- Internal classes remain unaffected

```java
// Request comes with: { "id": 1, "name": "John" }

// POJO for internal use
public class CustomerPojo {
    private int customerId;      // mapped from "id"
    private String customerName; // mapped from "name"
    
    // getters and setters
}

// Mapping layer
CustomerPojo pojo = new CustomerPojo();
pojo.setCustomerId(request.getId());
pojo.setCustomerName(request.getName());

// Now all internal classes use CustomerPojo, not request object
```

#### Use Case 2: Database Entity Representation

```
Request → Controller → Business Logic → Repository → POJO/Entity → Database
```

---

## 2. Enum Classes

### What is Enum?

**Enum** (Enumeration) is a special class that represents a **collection of constants**.

### Key Properties of Enum

| Property | Description |
|----------|-------------|
| **Constants** | Collection of predefined constant values |
| **Implicitly static final** | All constants are `static final` by default |
| **Cannot extend class** | Implicitly extends `java.lang.Enum` |
| **Can implement interface** | ✓ Allowed |
| **Cannot be instantiated** | Constructor is always `private` |
| **Cannot be extended** | No class can extend an enum |
| **Can have variables, constructors, methods** | ✓ Allowed |
| **Can have abstract methods** | All constants must implement them |

### Why Enum Implicitly Extends java.lang.Enum

```java
public enum DayOfWeek {
    MONDAY, TUESDAY, WEDNESDAY
}

// Compiler converts to something like:
public final class DayOfWeek extends java.lang.Enum<DayOfWeek> {
    public static final DayOfWeek MONDAY;
    public static final DayOfWeek TUESDAY;
    public static final DayOfWeek WEDNESDAY;
    
    private DayOfWeek() { }  // Always private!
}
```

> Since enum already extends `java.lang.Enum`, it **cannot extend any other class** (Java allows only single inheritance)

---

### 2.1 Basic Enum Declaration

```java
public enum EnumSample {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;  // Semicolon optional if no methods/variables follow
}
```

### Ordinal Values (Default Internal Values)

Each constant is automatically assigned an **ordinal** (integer value) starting from 0:

```
MONDAY    → 0
TUESDAY   → 1
WEDNESDAY → 2
THURSDAY  → 3
FRIDAY    → 4
SATURDAY  → 5
SUNDAY    → 6
```

---

### 2.2 Common Enum Methods

These methods come from `java.lang.Enum` class:

| Method | Return Type | Description |
|--------|-------------|-------------|
| `values()` | `EnumType[]` | Returns array of all constants |
| `ordinal()` | `int` | Returns position (0-based index) |
| `valueOf(String)` | `EnumType` | Returns enum constant by name |
| `name()` | `String` | Returns exact name of constant |

### Example Usage:

```java
public class EnumDemo {
    public static void main(String[] args) {
        
        // 1. values() - Get all constants as array
        System.out.println("=== Using values() and ordinal() ===");
        for (EnumSample sample : EnumSample.values()) {
            System.out.println(sample + " -> ordinal: " + sample.ordinal());
        }
        
        // Output:
        // MONDAY -> ordinal: 0
        // TUESDAY -> ordinal: 1
        // WEDNESDAY -> ordinal: 2
        // ... and so on
        
        // 2. valueOf() - Get enum by string name
        System.out.println("\n=== Using valueOf() ===");
        EnumSample friday = EnumSample.valueOf("FRIDAY");
        System.out.println("Got: " + friday);  // Output: Got: FRIDAY
        
        // 3. name() - Get name of constant
        System.out.println("\n=== Using name() ===");
        System.out.println(friday.name());  // Output: FRIDAY
    }
}
```

### Method Flow Diagram:

```
EnumSample.values()
       │
       ▼
┌─────────────────────────────────────────────────┐
│ EnumSample[] array:                             │
│ [MONDAY, TUESDAY, WEDNESDAY, THURSDAY, ...]     │
└─────────────────────────────────────────────────┘

EnumSample.valueOf("FRIDAY")
       │
       ▼
Searches through constants → Finds match → Returns FRIDAY enum

enumVariable.ordinal()
       │
       ▼
Returns integer position (0-based)

enumVariable.name()
       │
       ▼
Returns string name of constant
```

---

### 2.3 Enum with Custom Values

You can assign custom values to enum constants.

### Syntax:

```java
public enum EnumSample {
    // Constants with custom values (calling constructor)
    MONDAY(101, "First day of the week"),
    TUESDAY(102, "Second day of the week"),
    WEDNESDAY(103, "Third day of the week"),
    THURSDAY(104, "Fourth day of the week"),
    FRIDAY(105, "Fifth day of the week"),
    SATURDAY(106, "First weekend day"),
    SUNDAY(107, "Second weekend day");
    
    // Member variables (for each constant)
    private int val;
    private String comment;
    
    // Private constructor (always private, even if not specified)
    EnumSample(int val, String comment) {
        this.val = val;
        this.comment = comment;
    }
    
    // Getter methods
    public int getVal() {
        return val;
    }
    
    public String getComment() {
        return comment;
    }
    
    // Static method - belongs to enum class, not individual constants
    public static EnumSample getEnumFromValue(int value) {
        for (EnumSample sample : EnumSample.values()) {
            if (sample.val == value) {
                return sample;
            }
        }
        return null;  // or throw exception
    }
}
```

### Memory Representation:

```
┌─────────────────────────────────────────────────────┐
│                    EnumSample                        │
├─────────────────────────────────────────────────────┤
│  MONDAY    │ val: 101 │ comment: "First day..."     │
│  TUESDAY   │ val: 102 │ comment: "Second day..."    │
│  WEDNESDAY │ val: 103 │ comment: "Third day..."     │
│  THURSDAY  │ val: 104 │ comment: "Fourth day..."    │
│  FRIDAY    │ val: 105 │ comment: "Fifth day..."     │
│  SATURDAY  │ val: 106 │ comment: "First weekend..." │
│  SUNDAY    │ val: 107 │ comment: "Second weekend..."│
├─────────────────────────────────────────────────────┤
│  Static Methods: getEnumFromValue(int)              │
└─────────────────────────────────────────────────────┘
```

### Usage:

```java
public static void main(String[] args) {
    // Get enum by custom value
    EnumSample sunday = EnumSample.getEnumFromValue(107);
    
    // Access custom values
    System.out.println(sunday.getVal());      // Output: 107
    System.out.println(sunday.getComment());  // Output: Second weekend day
}
```

### Key Points:

| Component | Scope | Description |
|-----------|-------|-------------|
| **Member Variables** | Per constant | Each constant has its own copy |
| **Constructor** | Per constant | Called for each constant definition |
| **Instance Methods** | Per constant | Each constant has its own |
| **Static Methods** | Class level | Shared across all constants |

---

### 2.4 Method Override by Individual Constants

Each constant can override methods defined in the enum.

```java
public enum EnumSample {
    MONDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Monday dummy method");  // Overridden
        }
    },
    TUESDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Tuesday dummy method");  // Overridden
        }
    },
    WEDNESDAY,  // Uses default implementation
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;
    
    // Default implementation
    public void dummyMethod() {
        System.out.println("Default dummy method");
    }
}
```

### Usage:

```java
public static void main(String[] args) {
    EnumSample.MONDAY.dummyMethod();    // Output: Monday dummy method
    EnumSample.TUESDAY.dummyMethod();   // Output: Tuesday dummy method
    EnumSample.FRIDAY.dummyMethod();    // Output: Default dummy method
}
```

### Visual Representation:

```
┌─────────────────────────────────────────────────────────┐
│                      EnumSample                          │
├───────────────┬─────────────────────────────────────────┤
│    MONDAY     │ dummyMethod() → "Monday dummy method"   │
├───────────────┼─────────────────────────────────────────┤
│    TUESDAY    │ dummyMethod() → "Tuesday dummy method"  │
├───────────────┼─────────────────────────────────────────┤
│   WEDNESDAY   │ dummyMethod() → "Default dummy method"  │
├───────────────┼─────────────────────────────────────────┤
│   THURSDAY    │ dummyMethod() → "Default dummy method"  │
├───────────────┼─────────────────────────────────────────┤
│    FRIDAY     │ dummyMethod() → "Default dummy method"  │
├───────────────┼─────────────────────────────────────────┤
│   SATURDAY    │ dummyMethod() → "Default dummy method"  │
├───────────────┼─────────────────────────────────────────┤
│    SUNDAY     │ dummyMethod() → "Default dummy method"  │
└───────────────┴─────────────────────────────────────────┘
```

---

### 2.5 Enum with Abstract Methods

When you declare an abstract method in enum, **every constant MUST implement it**.

```java
public enum EnumSample {
    MONDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Dummy method in Monday");
        }
    },
    TUESDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Dummy method in Tuesday");
        }
    },
    SUNDAY {
        @Override
        public void dummyMethod() {
            System.out.println("Dummy method in Sunday");
        }
    };
    
    // Abstract method - ALL constants must implement
    public abstract void dummyMethod();
}
```

### Why Use Abstract Methods in Enum?

- Forces each constant to provide its own implementation
- Ensures no constant accidentally uses a default that might be incorrect
- Useful when behavior must vary for each constant

### Usage:

```java
EnumSample.MONDAY.dummyMethod();  // Output: Dummy method in Monday
EnumSample.SUNDAY.dummyMethod();  // Output: Dummy method in Sunday
```

---

### 2.6 Enum Implementing Interface

Enums can implement interfaces, useful for defining common behavior.

```java
// Interface definition
interface MyInterface {
    String toLowerCase();
}

// Enum implementing interface
public enum EnumSample implements MyInterface {
    MONDAY,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY,
    SATURDAY,
    SUNDAY;
    
    // Implementation - common for all constants
    @Override
    public String toLowerCase() {
        return this.name().toLowerCase();
    }
}
```

### Usage:

```java
public static void main(String[] args) {
    System.out.println(EnumSample.MONDAY.toLowerCase());   // Output: monday
    System.out.println(EnumSample.SATURDAY.toLowerCase()); // Output: saturday
}
```

### When to Use Interface with Enum:

- When method implementation is **same for all constants**
- When you want to enforce a contract across all constants
- When enum needs to be used polymorphically

---

### 2.7 Enum vs Static Final Constants

### Using Static Final (Traditional Way):

```java
public class WeekConstant {
    public static final int MONDAY = 0;
    public static final int TUESDAY = 1;
    public static final int WEDNESDAY = 2;
    public static final int THURSDAY = 3;
    public static final int FRIDAY = 4;
    public static final int SATURDAY = 5;
    public static final int SUNDAY = 6;
}
```

### Using Enum (Better Way):

```java
public enum EnumSample {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}
```

### Comparison with Example:

```java
// Method using static final constants
public boolean isWeekend(int day) {
    return WeekConstant.SATURDAY == day || WeekConstant.SUNDAY == day;
}

// Method using enum
public boolean isWeekend(EnumSample day) {
    return EnumSample.SATURDAY == day || EnumSample.SUNDAY == day;
}
```

### Usage Comparison:

```java
// Static Final - No type safety
isWeekend(2);     // Returns false (Wednesday)
isWeekend(6);     // Returns true (Sunday)
isWeekend(100);   // Returns false - BUT 100 is INVALID! No compile error!
isWeekend(-1);    // Returns false - INVALID! No compile error!

// Enum - Type safe
isWeekend(EnumSample.WEDNESDAY);  // Returns false
isWeekend(EnumSample.SUNDAY);     // Returns true
isWeekend(100);                   // ❌ COMPILE ERROR! Type mismatch
```

### Advantages of Enum over Static Final:

| Aspect | Static Final | Enum |
|--------|--------------|------|
| **Type Safety** | ❌ Accepts any int | ✓ Only valid constants |
| **Readability** | `isWeekend(6)` - unclear | `isWeekend(SUNDAY)` - clear |
| **Compile-time Check** | ❌ Invalid values compile | ✓ Invalid values caught |
| **Namespace** | Can clash with other ints | Unique type |
| **Can have methods** | ❌ No | ✓ Yes |
| **Can have fields** | ❌ No | ✓ Yes |
| **Iteration** | ❌ Manual | ✓ `values()` method |

### Visual: Type Safety Difference

```
Static Final Approach:
┌──────────────────────────────────────────────┐
│  isWeekend(int day)                          │
│      │                                       │
│      ▼                                       │
│  Accepts: 0, 1, 2, ... 100, -1, 999999       │
│           ↑                                  │
│           No validation! Any int accepted    │
└──────────────────────────────────────────────┘

Enum Approach:
┌──────────────────────────────────────────────┐
│  isWeekend(EnumSample day)                   │
│      │                                       │
│      ▼                                       │
│  Accepts ONLY:                               │
│  [MONDAY, TUESDAY, WEDNESDAY, THURSDAY,      │
│   FRIDAY, SATURDAY, SUNDAY]                  │
│           ↑                                  │
│           Compiler enforces valid values!    │
└──────────────────────────────────────────────┘
```

---

### 2.8 Enum Summary Table

| Feature | Supported | Example |
|---------|-----------|---------|
| **Define constants** | ✓ | `MONDAY, TUESDAY` |
| **Custom values** | ✓ | `MONDAY(101, "First day")` |
| **Variables** | ✓ | `private int val;` |
| **Constructor** | ✓ (always private) | `EnumSample(int val)` |
| **Instance methods** | ✓ | `getVal()` |
| **Static methods** | ✓ | `getEnumFromValue()` |
| **Override methods** | ✓ | Per-constant override |
| **Abstract methods** | ✓ | All constants must implement |
| **Implement interface** | ✓ | `implements MyInterface` |
| **Extend class** | ❌ | Already extends `java.lang.Enum` |
| **Be extended** | ❌ | Enums are implicitly final |
| **Be instantiated** | ❌ | Constructor is private |

---

## 3. Final Class

### Definition

A **final class** is a class that **cannot be inherited** (extended) by any other class.

### Syntax:

```java
public final class TestClass {
    // class body
}
```

### Example:

```java
// Final class - cannot be extended
public final class TestClass {
    public void display() {
        System.out.println("This is a final class");
    }
}

// Attempting to extend final class
public class MyAnotherClass extends TestClass {  // ❌ COMPILE ERROR!
    // Cannot inherit from final 'TestClass'
}
```

### Error Message:
```
Error: Cannot inherit from final 'TestClass'
```

### Purpose of Final Class:

| Purpose | Description |
|---------|-------------|
| **Prevent Inheritance** | No class can extend it |
| **Security** | Prevents malicious subclassing |
| **Immutability** | Often used with immutable classes |
| **Design Intent** | Signals class is complete, not meant for extension |

### Real-World Examples of Final Classes in Java:

```java
// These are all final classes in Java API
public final class String { }
public final class Integer { }
public final class Double { }
public final class Math { }
public final class System { }
```

### Why Make a Class Final?

1. **Security:** Prevent subclasses from altering behavior
2. **Immutability:** Ensure class state cannot be modified by subclass
3. **Performance:** JVM can optimize final classes better
4. **Design:** Class is complete and shouldn't be extended

### Final Class vs Final Method vs Final Variable:

| Final Applied To | Effect |
|-----------------|--------|
| **Class** | Cannot be extended |
| **Method** | Cannot be overridden |
| **Variable** | Cannot be reassigned (constant) |

```java
public final class FinalClass { }        // Cannot extend this class

public class Parent {
    public final void method() { }        // Cannot override this method
}

public class Example {
    public final int VALUE = 100;         // Cannot change this value
}
```

---

## 4. Quick Reference Summary

### POJO Checklist:

```
✓ Public class
✓ Public default constructor
✓ Private/protected/public variables
✓ Getter and setter methods
✗ No annotations
✗ No extending classes
✗ No implementing interfaces
```

### Enum Quick Reference:

```java
public enum DayType {
    // 1. Basic constants
    WEEKDAY,
    WEEKEND;
    
    // 2. With custom values
    // WEEKDAY(1), WEEKEND(2);
    // private int code;
    // DayType(int code) { this.code = code; }
    
    // 3. Common methods
    // values()      - get all constants
    // ordinal()     - get position (0-based)
    // valueOf(str)  - get by name
    // name()        - get constant name
}
```

### Final Class Pattern:

```java
public final class ImmutableClass {
    private final String value;
    
    public ImmutableClass(String value) {
        this.value = value;
    }
    
    public String getValue() {
        return value;
    }
}
```

---

## 5. Interview Tips

1. **POJO:** Know all 6 characteristics and be able to identify what breaks POJO rules

2. **Enum:**
    - Know why enum can't extend classes (already extends `java.lang.Enum`)
    - Understand ordinal vs custom values
    - Be able to explain advantages over `static final` constants
    - Know how to add methods, constructors, and implement interfaces

3. **Final Class:**
    - Know examples from Java API (`String`, `Integer`, etc.)
    - Understand relationship with immutability
    - Know difference between `final` class, method, and variable

4. **Common Question:** "Why use enum instead of static final int constants?"
    - Type safety
    - Compile-time checking
    - Readability
    - Can have methods and fields
    - Namespace isolation