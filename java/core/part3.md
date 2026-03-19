# Java Variables - Notes

---

## 1. What is a Variable?

**Definition:** A variable is a **container** that holds a value.

### Syntax:
```java
dataType variableName = value;

// Example:
int var = 32;
```

| Component | Description |
|-----------|-------------|
| `dataType` | What kind of data it stores (int, float, etc.) |
| `variableName` | Name/identifier of the container |
| `value` | Actual data stored |

---

## 2. Java Type System

### Java is a Statically Typed Language
- Every variable **must have a data type** defined
- Data type tells what kind of data can be stored

```java
int var = 32;    // Must specify 'int'
```

### Java is a Strongly Typed Language
- Each data type has a **range/limit**
- Cannot assign values outside the range
- Restrictions on what values can be assigned

```java
byte b = 127;    // ✓ Valid (byte range: -128 to 127)
byte b = 200;    // ✗ Invalid (out of range)
```

---

## 3. Variable Naming Conventions

### Rules:

| Rule | Example | Valid? |
|------|---------|--------|
| Case sensitive | `int a;` vs `int A;` | Different variables |
| Can contain Unicode letters & digits | `int var1;` | ✓ |
| Can start with: letter, $, _ | `int _var;` `int $var;` `int var;` | ✓ |
| Cannot start with digit | `int 9var;` | ✗ |
| Cannot be Java reserved words | `int class;` `int new;` | ✗ |

### Conventions:

| Scenario | Convention | Example |
|----------|------------|---------|
| Single word | All lowercase | `city`, `name` |
| Multiple words | camelCase | `cityName`, `firstName` |
| Constants | ALL CAPS | `MAX_VALUE`, `PI` |

```java
// Good naming:
int jaipur = 2;           // single word - lowercase
int jaipurCity = 2;       // multiple words - camelCase
static final int JAIPUR = 2;  // constant - UPPERCASE
```

---

## 4. Types of Variables

```
                    Variables
                        │
          ┌─────────────┴─────────────┐
          │                           │
    Primitive Types            Reference/Non-Primitive
    (8 types)                  (covered separately)
          │
    ┌─────┴─────┐
    │           │
 Integral   Floating   Boolean
    │           │          │
char,byte   float     boolean
short,int   double
long
```

---

## 5. Primitive Data Types (8 Types)

### Overview Table:

| Type | Size | Range | Default | Category |
|------|------|-------|---------|----------|
| `char` | 2 bytes (16 bits) | 0 to 65,535 | `'\u0000'` (null) | Character |
| `byte` | 1 byte (8 bits) | -128 to 127 | 0 | Integral |
| `short` | 2 bytes (16 bits) | -32,768 to 32,767 | 0 | Integral |
| `int` | 4 bytes (32 bits) | -2³¹ to 2³¹-1 | 0 | Integral |
| `long` | 8 bytes (64 bits) | -2⁶³ to 2⁶³-1 | 0 | Integral |
| `float` | 4 bytes (32 bits) | IEEE 754 | 0.0f | Floating |
| `double` | 8 bytes (64 bits) | IEEE 754 | 0.0d | Floating |
| `boolean` | 1 bit | true/false | false | Boolean |

---

### 5.1 Character (`char`)

```java
char var = 'a';        // Character literal
char var = 65;         // ASCII value → prints 'A'
char var = 97;         // ASCII value → prints 'a'
```

- 2 bytes (16 bits)
- Range: 0 to 65,535
- Stores **Unicode/ASCII** character representation
- Default: `'\u0000'` (null character)

---

### 5.2 Integral Types (byte, short, int, long)

All integral types are **Signed Two's Complement** (negative numbers allowed)

#### Byte
```java
byte b = 100;
```
- 1 byte (8 bits)
- Range: **-128 to 127**
- Default: 0

#### Short
```java
short s = 1000;
```
- 2 bytes (16 bits)
- Range: **-32,768 to 32,767**
- Default: 0

#### Int
```java
int i = 100000;
```
- 4 bytes (32 bits)
- Range: **-2,147,483,648 to 2,147,483,647**
- Default: 0

#### Long
```java
long l = 100L;    // Add 'L' suffix
long l = 500l;    // lowercase 'l' also works
```
- 8 bytes (64 bits)
- Range: **-2⁶³ to 2⁶³-1**
- Default: 0
- **Must add `L` suffix** when declaring

---

### Understanding Signed Two's Complement

**Why range is -128 to 127 for byte (not 0 to 255)?**

```
8 bits can represent: 0 to 255 (unsigned)

But Java uses SIGNED representation:
- Last bit (MSB) indicates sign:
  - 0 = positive
  - 1 = negative
  
So range becomes: -128 to 127
```

#### How Negative Numbers are Stored (Two's Complement):

```
To represent -3 (using 4 bits for simplicity):

Step 1: Write +3 in binary
        +3 = 0011

Step 2: First complement (flip bits)
        0011 → 1100

Step 3: Add 1 (Second complement)
        1100 + 1 = 1101

So -3 = 1101

Verification: 0011 (+3) + 1101 (-3) = 10000 → 0000 (overflow ignored) = 0 ✓
```

---

### 5.3 Floating Point Types (float, double)

#### Float
```java
float f = 63.20f;    // Must add 'f' suffix
float f = 63.20F;    // 'F' also works
```
- 4 bytes (32 bits)
- IEEE 754 standard
- Default: 0.0f

#### Double
```java
double d = 63.20d;   // Add 'd' suffix
double d = 63.20D;   // 'D' also works
double d = 63.20;    // Default is double
```
- 8 bytes (64 bits)
- IEEE 754 standard
- Default: 0.0d

#### ⚠️ Important Warning: Precision Issues!

```java
float var1 = 0.3f;
float var2 = 0.1f;
float var3 = var1 - var2;

System.out.println(var3);
// Expected: 0.2
// Actual:   0.20000002  ← PRECISION ERROR!
```

**Why?** IEEE 754 floating-point representation cannot exactly represent some decimal numbers.

> **Industry Rule:** For **currency** or precise decimal calculations, **NEVER use float/double**. Use **BigDecimal** instead!

---

### 5.4 Boolean

```java
boolean flag = true;
boolean isValid = false;
```
- Only two values: `true` or `false`
- Default: `false`
- Used for conditional logic

---

## 6. Default Values

### Important Rule:
> **Default values are only assigned to CLASS MEMBER variables, NOT local variables!**

```java
public class Employee {
    byte var;           // Member variable → default = 0
    
    void method() {
        byte localVar;  // Local variable → NO default!
        System.out.println(localVar);  // ✗ ERROR! Must initialize
    }
}
```

```java
// Correct way for local variables:
void method() {
    byte localVar = 7;  // Must initialize manually
    System.out.println(localVar);  // ✓ Works
}
```

---

## 7. Type Conversion/Casting

### 7.1 Widening (Automatic Conversion)

**Lower → Higher data type = Automatic**

```
byte → short → int → long → float → double
  1      2      4      8       4       8   (bytes)
```

```java
byte x = 10;
int intVar = x;    // ✓ Automatic conversion (byte → int)

int a = 10;
long b = a;        // ✓ Automatic conversion (int → long)
```

**Why it works:** Higher data type can easily accommodate lower data type values.

---

### 7.2 Narrowing (Explicit/Downcasting)

**Higher → Lower data type = Manual casting required**

```java
int intVar = 10;
byte byteVar = intVar;           // ✗ ERROR!

byte byteVar = (byte) intVar;    // ✓ Explicit cast
```

#### ⚠️ Data Loss Warning:

```java
int intVar = 128;
byte byteVar = (byte) intVar;

System.out.println(byteVar);  // Output: -128  (NOT 128!)
```

**Why -128?**
- Byte range: -128 to 127
- 128 exceeds max (127)
- Wraps around: 127 + 1 = -128

```
Value: 128
Byte max: 127
Overflow: 128 - 128 = 0 positions after -128
Result: -128

If value was 129:
Overflow: 129 - 128 = 1 position after -128
Result: -127
```

---

### 7.3 Promotion During Expression

#### Rule 1: Automatic promotion when exceeding range

```java
byte a = 127;
byte b = 1;
byte sum = a + b;    // ✗ ERROR!
```

**Why error?**
- `a + b = 128` exceeds byte range (127)
- Java automatically promotes `byte` and `short` to `int` in expressions
- Result becomes `int`, can't store in `byte`

**Solutions:**
```java
// Solution 1: Store in int
int sum = a + b;     // ✓ Output: 128

// Solution 2: Explicit downcast
byte sum = (byte)(a + b);  // ✓ Output: -128 (wraps around)
```

#### Rule 2: Expression promoted to highest data type

```java
int a = 34;
double b = 20.0d;

int sum = a + b;        // ✗ ERROR! Result is double

double sum = a + b;     // ✓ Output: 54.0
int sum = (int)(a + b); // ✓ Output: 54 (explicit cast)
```

> **If any operand is of higher type, the entire expression is promoted to that type.**

---

### Type Conversion Summary:

| Conversion | Direction | Method | Example |
|------------|-----------|--------|---------|
| Widening | Lower → Higher | Automatic | `int i = byteVar;` |
| Narrowing | Higher → Lower | Explicit cast | `byte b = (byte)intVar;` |
| Expression Promotion | Mixed types | Auto to highest | `int + double = double` |

---

## 8. Kinds of Variables

### Overview:

```java
public class Employee {
    // 1. MEMBER/INSTANCE Variable
    int memberVariable = 10;
    
    // 2. STATIC/CLASS Variable
    static int staticVariable = 100;
    
    // 3. CONSTRUCTOR Parameter
    Employee(int a) {
        // 'a' is constructor parameter
    }
    
    void dummyMethod() {
        // 4. LOCAL Variable
        byte localVariable = 100;
    }
    
    int dummyMethod2(int a, int b) {
        // 5. METHOD Parameters
        return a + b;
    }
}
```

---

### 8.1 Member/Instance Variables

- Declared **inside class**, but **outside methods**
- Each object has its **own copy**
- Accessed via object reference

```java
class Employee {
    int memberVar = 10;  // Member variable
}

// Usage:
Employee obj1 = new Employee();
Employee obj2 = new Employee();

obj1.memberVar = 20;  // obj1's copy changed
obj2.memberVar = 30;  // obj2's copy changed (independent)
```

```
obj1 ─────┐     ┌───── obj2
          │     │
    ┌─────▼─────▼─────┐
    │  memberVar=20   │  (obj1's copy)
    └─────────────────┘
    
    ┌─────────────────┐
    │  memberVar=30   │  (obj2's copy)
    └─────────────────┘
```

---

### 8.2 Static/Class Variables

- Declared with `static` keyword
- **Only ONE copy** exists (shared by all objects)
- Accessed via **class name** (not object)

```java
class Employee {
    static int staticVar = 100;  // Static variable
}

// Usage:
Employee obj1 = new Employee();
Employee obj2 = new Employee();

// Access via class name (correct way):
System.out.println(Employee.staticVar);  // 100

// Cannot access via object:
// obj1.staticVar  ← Not recommended / may not work
```

```
        Employee Class
    ┌─────────────────────┐
    │  staticVar = 100    │  ← Only ONE copy
    └─────────────────────┘
              ▲
              │ (shared access)
        ┌─────┴─────┐
        │           │
      obj1        obj2
```

---

### 8.3 Local Variables

- Declared **inside a method**
- Scope: Only within that method
- **Destroyed** after method execution ends
- **Must be initialized** (no default value)

```java
void dummyMethod() {
    byte localVar = 100;  // Local variable
    System.out.println(localVar);
}  // localVar destroyed here
```

---

### 8.4 Method Parameters

- Variables declared in **method signature**
- Values assigned by **caller**

```java
int calculate(int a, int b) {  // a, b are method parameters
    return a + b;
}

// Caller:
obj.calculate(2, 5);  // a=2, b=5
```

---

### 8.5 Constructor Parameters

- Variables declared in **constructor signature**
- Used during **object creation**

```java
class Employee {
    Employee() { }              // Default constructor
    
    Employee(int a) {          // Parameterized constructor
        // 'a' is constructor parameter
    }
}

// Usage:
Employee obj1 = new Employee();      // Calls default
Employee obj2 = new Employee(10);    // Calls parameterized (a=10)
```

---

### Variable Types Comparison:

| Type | Scope | Lifetime | Default Value | Access |
|------|-------|----------|---------------|--------|
| Member/Instance | Class | Object lifetime | Yes | `object.var` |
| Static/Class | Class | Program lifetime | Yes | `ClassName.var` |
| Local | Method | Method execution | **No** | Direct |
| Method Parameter | Method | Method execution | **No** | Direct |
| Constructor Parameter | Constructor | Constructor execution | **No** | Direct |

---

## 9. Quick Reference Diagram

```
                        JAVA VARIABLES
                              │
        ┌─────────────────────┴─────────────────────┐
        │                                           │
   DATA TYPES                                 KINDS/SCOPE
        │                                           │
   ┌────┴────┐                            ┌─────────┼─────────┐
   │         │                            │         │         │
Primitive  Reference               Member    Static    Local
(8 types)  (Objects)               (Instance) (Class)   (Method)
   │                                  │         │         │
┌──┴──┐                           Own copy  Shared   Method
│     │                           per obj    copy    scope
Integral  Floating  Boolean
│         │
char     float
byte     double
short
int
long
```

---

## 10. Interview Quick Points

1. **Java is statically typed** → Must declare data type
2. **Java is strongly typed** → Values must be within range
3. **8 primitive types:** char, byte, short, int, long, float, double, boolean
4. **Default values only for member variables**, not local
5. **Widening = automatic**, Narrowing = explicit cast
6. **Never use float/double for currency** → Use BigDecimal
7. **Static variables** → One copy, accessed via class name
8. **Two's complement** → How negative numbers are stored
9. **Expression promotion** → Auto-promotes to highest data type
10. **Naming conventions:** camelCase for variables, UPPERCASE for constants

---

## 11. Common Mistakes to Avoid

```java
// 1. Forgetting 'L' for long
long l = 10000000000;    // ✗ Error
long l = 10000000000L;   // ✓

// 2. Forgetting 'f' for float
float f = 3.14;          // ✗ Error (defaults to double)
float f = 3.14f;         // ✓

// 3. Using uninitialized local variable
void method() {
    int x;
    System.out.println(x);  // ✗ Error
}

// 4. Overflow during narrowing
int i = 128;
byte b = (byte) i;  // b = -128 (not 128!)

// 5. Float precision in calculations
float f = 0.3f - 0.1f;  // NOT exactly 0.2!
```