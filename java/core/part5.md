# Java Variables Part 2: Reference Types, Wrappers & Constants - Notes

---

## 1. Types of Data Types in Java

```
                Data Types
                    │
        ┌───────────┴───────────┐
        │                       │
   Primitive              Non-Primitive
   (8 types)              (Reference Types)
        │                       │
   int, char,             ┌─────┼─────┐─────┐
   byte, short,           │     │     │     │
   long, float,         Class String Interface Array
   double, boolean
```

---

## 2. What is a Reference?

### Understanding References with Classes

```java
// Employee class
class Employee {
    int employeeId;
    
    int getEmployeeId() {
        return employeeId;
    }
    
    void setEmployeeId(int id) {
        this.employeeId = id;
    }
}

// Creating an object
Employee empObject = new Employee();
```

### What Happens in Memory:

```
        STACK                          HEAP MEMORY
    ┌──────────┐                  ┌─────────────────┐
    │empObject │ ───reference───► │ Employee Object │
    │ (ref)    │                  │ employeeId = ?  │
    └──────────┘                  │ Address: ABC123 │
                                  └─────────────────┘
```

**Key Points:**
- `new Employee()` creates an object in **HEAP memory**
- `empObject` is a **reference variable** stored in STACK
- `empObject` **holds the address/reference** to the actual object in heap
- That's why they're called **Reference Data Types**!

---

## 3. Java: Pass by Value (No Pointers!)

### Important Concept:

> **In Java, EVERYTHING is passed by VALUE. There is NO pass by reference and NO pointers!**

### But How Do We Get Reference-like Behavior?

```java
class Student {
    void modifyExample() {
        Employee empObject = new Employee();
        empObject.employeeId = 10;
        
        modify(empObject);  // Passing reference VALUE
        
        System.out.println(empObject.employeeId);  // Prints: 20
    }
    
    void modify(Employee emp) {
        emp.employeeId = 20;  // Changes the ACTUAL object in heap
    }
}
```

### How It Works:

```
BEFORE modify():
┌────────────┐                    ┌─────────────────┐
│ empObject  │ ───────────────►   │ employeeId = 10 │
└────────────┘                    └─────────────────┘

DURING modify():
┌────────────┐                    ┌─────────────────┐
│ empObject  │ ───────────────►   │ employeeId = 10 │
└────────────┘                ┌─► │    → 20         │
┌────────────┐                │   └─────────────────┘
│    emp     │ ───────────────┘
└────────────┘
(copy of reference - points to SAME object!)

AFTER modify():
Output: 20 (because both pointed to same memory!)
```

### Multiple References to Same Object:

```java
Employee empObject = new Employee();
Employee obj2 = empObject;  // Both point to SAME memory

obj2.employeeId = 30;  // Changes the shared object
System.out.println(empObject.employeeId);  // 30!
```

---

## 4. Reference Data Type 1: Class

```java
// Definition
class Employee {
    int employeeId;
    // getters, setters, methods...
}

// Creating objects
Employee emp1 = new Employee();  // Creates object in heap
Employee emp2 = new Employee();  // Creates ANOTHER object in heap
Employee emp3 = emp1;            // Points to SAME object as emp1
```

---

## 5. Reference Data Type 2: String (Very Important!)

### String Constant Pool

Inside **Heap Memory**, there's a special area called **String Constant Pool** (SCP).

```
                    HEAP MEMORY
    ┌───────────────────────────────────────┐
    │                                       │
    │   ┌─────────────────────────────┐     │
    │   │   STRING CONSTANT POOL      │     │
    │   │                             │     │
    │   │   ┌─────────┐               │     │
    │   │   │ "Hello" │               │     │
    │   │   └─────────┘               │     │
    │   │                             │     │
    │   └─────────────────────────────┘     │
    │                                       │
    │   ┌─────────────────┐                 │
    │   │ String Object   │  (created with  │
    │   │ "Hello"         │   new keyword)  │
    │   └─────────────────┘                 │
    │                                       │
    └───────────────────────────────────────┘
```

### String Literals vs String Objects

```java
// String Literal - goes to String Constant Pool
String s1 = "Hello";

// String Literal - checks SCP first, reuses existing
String s2 = "Hello";  // Points to SAME "Hello" as s1

// String Object - creates NEW object in heap (outside SCP)
String s3 = new String("Hello");
```

### Key Behavior:

| Declaration | Where Created | Checks SCP? |
|-------------|---------------|-------------|
| `String s = "Hello"` | String Constant Pool | Yes, reuses if exists |
| `String s = new String("Hello")` | Heap (outside SCP) | No, always new object |

---

### Strings are IMMUTABLE

**Immutable = Once created, cannot be changed**

```java
String s1 = "Hello";
s1 = "World";  // Does NOT change "Hello"!
```

**What Actually Happens:**

```
BEFORE:
SCP: ┌─────────┐
     │ "Hello" │ ◄── s1
     └─────────┘

AFTER s1 = "World":
SCP: ┌─────────┐
     │ "Hello" │      (still exists, unchanged!)
     └─────────┘
     ┌─────────┐
     │ "World" │ ◄── s1 (new reference)
     └─────────┘
```

> **"Hello" is NOT modified - only the reference s1 points to a new string!**

---

### Comparing Strings: `==` vs `.equals()`

| Operator | What It Compares |
|----------|------------------|
| `==` | **Memory address** (are they same object?) |
| `.equals()` | **Content/Value** (are values same?) |

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

// == compares memory addresses
System.out.println(s1 == s2);      // true (same SCP literal)
System.out.println(s1 == s3);      // false (different memory)

// .equals() compares content
System.out.println(s1.equals(s2)); // true (same content)
System.out.println(s1.equals(s3)); // true (same content)
```

### Visual Explanation:

```
        STACK                    HEAP (String Constant Pool)
    ┌────────┐                  ┌─────────────────────────┐
    │   s1   │ ────────────────►│       "Hello"           │
    └────────┘                  └─────────────────────────┘
    ┌────────┐                            ▲
    │   s2   │ ───────────────────────────┘
    └────────┘                  
                                HEAP (Regular)
    ┌────────┐                  ┌─────────────────────────┐
    │   s3   │ ────────────────►│  String Object "Hello"  │
    └────────┘                  └─────────────────────────┘

s1 == s2  → true  (same address in SCP)
s1 == s3  → false (different addresses)
s1.equals(s3) → true (same content "Hello")
```

---

## 6. Reference Data Type 3: Interface

```java
// Interface definition
interface Person {
    String profession();  // Abstract method
}

// Implementation 1
class Teacher implements Person {
    @Override
    public String profession() {
        return "Teaching";
    }
}

// Implementation 2
class Engineer implements Person {
    @Override
    public String profession() {
        return "Software Engineer";
    }
}
```

### Creating Objects with Interfaces:

```java
// ✓ Valid ways:
Person engineer = new Engineer();     // Parent reference, child object
Person teacher = new Teacher();       // Parent reference, child object
Teacher t = new Teacher();            // Same class reference
Engineer e = new Engineer();          // Same class reference

// ✗ Invalid:
Person p = new Person();              // ERROR! Cannot instantiate interface!
```

### Why Can't We Instantiate Interface?

- Interface only defines **blueprint** (method signatures)
- No actual **implementation**
- Child classes provide implementation → so create objects of children

```
        Person (interface)
        ┌──────────────┐
        │ profession() │ ←── just signature, no code
        └──────────────┘
              ▲
    ┌─────────┴─────────┐
    │                   │
┌───────────┐     ┌───────────┐
│  Teacher  │     │ Engineer  │
│ return    │     │ return    │
│"Teaching" │     │"Software" │
└───────────┘     └───────────┘
     ↑                  ↑
Can create         Can create
objects here!      objects here!
```

---

## 7. Reference Data Type 4: Array

### 1D Array

```java
// Method 1: Declare size, assign later
int[] arr = new int[5];  // Creates 5 blocks in heap
arr[0] = 10;
arr[3] = 40;

// Method 2: Alternative syntax
int arr[] = new int[5];  // Same as above

// Method 3: Initialize directly
int[] arr = {30, 20, 10, 40, 50};  // Size auto-detected (5)
```

**Memory Representation:**

```
STACK                          HEAP
┌─────┐                  ┌─────┬─────┬─────┬─────┬─────┐
│ arr │ ────────────────►│  10 │  0  │  0  │ 40  │  0  │
└─────┘                  └─────┴─────┴─────┴─────┴─────┘
(reference)              Index: 0     1     2     3     4
```

### 2D Array

```java
// Method 1: Declare size, assign later
int[][] arr = new int[5][4];  // 5 rows, 4 columns
arr[2][2] = 20;  // Row 2, Column 2
arr[1][3] = 30;  // Row 1, Column 3

// Method 2: Alternative syntax
int arr[][] = new int[5][4];  // Same

// Method 3: Initialize directly
int[][] arr = {
    {1, 5, 7},
    {4, 2, 3}
};  // 2 rows, 3 columns
```

**Memory Representation (5x4 array):**

```
           Column:  0    1    2    3
                  ┌────┬────┬────┬────┐
        Row 0     │ 0  │ 0  │ 0  │ 0  │
                  ├────┼────┼────┼────┤
        Row 1     │ 0  │ 0  │ 0  │ 30 │  ← arr[1][3]
                  ├────┼────┼────┼────┤
        Row 2     │ 0  │ 0  │ 20 │ 0  │  ← arr[2][2]
                  ├────┼────┼────┼────┤
        Row 3     │ 0  │ 0  │ 0  │ 0  │
                  ├────┼────┼────┼────┤
        Row 4     │ 0  │ 0  │ 0  │ 0  │
                  └────┴────┴────┴────┘
```

### Accessing 2D Array:

```java
int[][] arr = {{1, 5, 7}, {4, 2, 3}};

// arr[row][column]
arr[0][1] → 5  // Row 0, Column 1
arr[1][2] → 3  // Row 1, Column 2
```

---

## 8. Wrapper Classes

### What Are Wrapper Classes?

For each **primitive type**, Java provides a corresponding **Reference/Object type**:

| Primitive | Wrapper Class |
|-----------|---------------|
| `int` | `Integer` |
| `char` | `Character` |
| `byte` | `Byte` |
| `short` | `Short` |
| `long` | `Long` |
| `float` | `Float` |
| `double` | `Double` |
| `boolean` | `Boolean` |

### Why Do We Need Wrapper Classes?

#### Reason 1: Reference Capability

**Primitive - Changes NOT reflected:**
```java
void example() {
    int a = 10;
    modify(a);
    System.out.println(a);  // Still 10!
}

void modify(int x) {
    x = 20;  // Only changes local copy
}
```

**Wrapper - Changes reflected (if using mutable wrapper or array):**
```java
// Note: Integer is immutable, but concept applies to mutable objects
void example() {
    int[] a = {10};
    modify(a);
    System.out.println(a[0]);  // 20!
}

void modify(int[] x) {
    x[0] = 20;  // Changes actual object
}
```

#### Reason 2: Collections Require Objects

```java
// ✗ Collections don't work with primitives
ArrayList<int> list = new ArrayList<>();  // ERROR!

// ✓ Collections work with wrapper classes
ArrayList<Integer> list = new ArrayList<>();  // Works!
```

---

### Autoboxing and Unboxing

#### Autoboxing: Primitive → Wrapper (Automatic)

```java
int a = 10;
Integer a1 = a;  // Autoboxing: int → Integer (automatic)
```

#### Unboxing: Wrapper → Primitive (Automatic)

```java
Integer x = 20;
int x1 = x;  // Unboxing: Integer → int (automatic)
```

### Summary:

```
        AUTOBOXING
    ┌──────────────────┐
    │                  │
    ▼                  │
Primitive ──────────► Wrapper
    │                  │
    │                  ▼
    └──────────────────┘
         UNBOXING
```

---

## 9. Constant Variables (`static final`)

### Static Variable (Review)

```java
class Employee {
    static int empId = 10;  // Shared by ALL objects
}

Employee obj1 = new Employee();
Employee obj2 = new Employee();

// Both share the SAME empId
Employee.empId = 20;  // obj1 and obj2 both see 20
```

### Static ≠ Constant

```java
static int empId = 10;
empId = 20;  // ✓ Can change! Not a constant.
```

### Making a CONSTANT: `static final`

```java
class Employee {
    public static final int EMPLOYEE_ID = 10;  // CONSTANT!
}

// Cannot change!
Employee.EMPLOYEE_ID = 20;  // ✗ COMPILATION ERROR!
```

### Constant Variable Rules:

| Keyword | Effect |
|---------|--------|
| `static` | Only ONE copy exists (belongs to class) |
| `final` | Value CANNOT be changed after initialization |
| `static final` | **CONSTANT** - one copy, cannot change |

### Naming Convention for Constants:

```java
// ✓ Correct: ALL_CAPS with underscores
public static final int MAX_SIZE = 100;
public static final String DEFAULT_NAME = "Unknown";
public static final double PI = 3.14159;

// ✗ Wrong: camelCase
public static final int maxSize = 100;  // Not standard
```

---

## 10. Reference Types Summary

| Type | Description | Example |
|------|-------------|---------|
| **Class** | User-defined template | `Employee emp = new Employee();` |
| **String** | Immutable text, uses SCP | `String s = "Hello";` |
| **Interface** | Blueprint for classes | `Person p = new Teacher();` |
| **Array** | Fixed-size collection | `int[] arr = new int[5];` |

---

## 11. Quick Reference Diagram

```
                    JAVA VARIABLES PART 2
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   Reference Types     Wrapper Classes    Constants
        │                   │                   │
   ┌────┼────┐         Primitive →        static final
   │    │    │         Wrapper
Class String Array     (Autoboxing)
   │    │    │              │
   │    │    │         Wrapper →
   │    │    │         Primitive
   │  String │         (Unboxing)
   │  Constant
   │  Pool
   │
   └── Immutable
       == vs .equals()
```

---

## 12. Interview Key Points

1. **Reference types store addresses**, not actual values
2. **Java has NO pointers** - everything is pass by value
3. **Pass by value with references** achieves similar effect to pass by reference
4. **String Constant Pool** - literals are reused to save memory
5. **Strings are immutable** - "changing" creates new string
6. **`==` checks address**, `.equals()` checks content
7. **Cannot instantiate interfaces** - only their implementations
8. **Wrapper classes** exist for all 8 primitives
9. **Autoboxing**: primitive → wrapper (automatic)
10. **Unboxing**: wrapper → primitive (automatic)
11. **Collections only work with objects** (wrapper classes)
12. **`static final`** = constant variable (cannot change)

---

## 13. Common Interview Questions

### Q1: What's the output?
```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");

System.out.println(s1 == s2);       // ?
System.out.println(s1 == s3);       // ?
System.out.println(s1.equals(s3));  // ?
```

**Answer:** `true`, `false`, `true`

### Q2: Is Java pass by reference or pass by value?
**Answer:** Always **pass by value**. But for objects, the value passed is the **reference/address**, which allows modifications to affect the original object.

### Q3: Why are Strings immutable?
**Answer:**
- Security (credentials can't be modified)
- Thread safety (safe in multi-threaded env)
- String pool optimization (can safely share)
- Hashcode caching (used in HashMap keys)

### Q4: Difference between `int` and `Integer`?
**Answer:**
| `int` | `Integer` |
|-------|-----------|
| Primitive | Object (Wrapper) |
| Stack memory | Heap memory |
| Cannot be null | Can be null |
| Cannot use in Collections | Can use in Collections |
| Default: 0 | Default: null |