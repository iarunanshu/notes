# Java Methods - Comprehensive Notes

---

## 1. What is a Method?

**Definition:** A method is a **collection of instructions** that performs a specific task.

```
         ┌─────────────────┐
Input →  │     METHOD      │  → Output
         │  (Instructions) │
         └─────────────────┘
```

### Advantages of Methods:
1. **Code Reusability** - Write once, use multiple times
2. **Code Readability** - Method names describe what they do
3. **Modularity** - Break complex problems into smaller parts

### Example:

```java
public class Calculation {
    
    // Method definition
    public int sum(int var1, int var2) {
        int total = var1 + var2;           // Instruction 1
        System.out.println("Sum: " + total); // Instruction 2
        return total;                       // Instruction 3
    }
    
    // Reusing the sum method
    public int getPriceOfPen() {
        int capPrice = 2;
        int bodyPrice = 5;
        int totalPenPrice = sum(capPrice, bodyPrice);  // REUSED!
        return totalPenPrice;
    }
    
    // Reusing again
    public int getCombinedAge() {
        int youngerSisterAge = 2;
        int olderSisterAge = 5;
        return sum(youngerSisterAge, olderSisterAge);  // REUSED!
    }
}
```

---

## 2. Method Declaration Structure

```java
accessSpecifier returnType methodName(parameterList) throws Exception {
    // Method body
    return value;
}
```

```
┌─────────────┬────────────┬────────────┬──────────────┬─────────────────┐
│   Access    │   Return   │   Method   │  Parameters  │     throws      │
│  Specifier  │    Type    │    Name    │    (args)    │   Exception     │
└─────────────┴────────────┴────────────┴──────────────┴─────────────────┘
     │              │            │              │               │
   public         int          sum        (int a, int b)   throws Exception
   private        void       getInvoice      ()
   protected      boolean     isEmpty      (String s)
   (default)      String     getName        ()
```

---

## 3. Access Specifiers

### Overview:

| Specifier | Same Class | Same Package | Subclass (Different Package) | Different Package |
|-----------|------------|--------------|------------------------------|-------------------|
| `public` | ✓ | ✓ | ✓ | ✓ |
| `protected` | ✓ | ✓ | ✓ | ✗ |
| `default` (no keyword) | ✓ | ✓ | ✗ | ✗ |
| `private` | ✓ | ✗ | ✗ | ✗ |

### What is a Package?

A **package** is a collection of logically similar/related classes.

```
src/
├── salesDepartment/          ← Package 1
│   ├── Invoice.java
│   └── Order.java
│
└── humanResource/            ← Package 2
    ├── JobPortal.java
    └── Application.java
```

---

### 3.1 Public

**Can be accessed from ANY class in ANY package** (Global access)

```java
// In salesDepartment/Invoice.java
package salesDepartment;

public class Invoice {
    public void getInvoice() {    // PUBLIC method
        System.out.println("Inside invoice method");
    }
}

// In humanResource/JobPortal.java - DIFFERENT PACKAGE
package humanResource;
import salesDepartment.Invoice;

public class JobPortal {
    void test() {
        Invoice inv = new Invoice();
        inv.getInvoice();  // ✓ Can access - it's public
    }
}
```

---

### 3.2 Private

**Can ONLY be accessed within the SAME class**

```java
package salesDepartment;

public class Invoice {
    private void getInvoice() {   // PRIVATE method
        System.out.println("Inside invoice");
    }
    
    public void printInvoice() {
        getInvoice();  // ✓ Same class - can access
    }
}

// In Order.java - SAME PACKAGE but different class
public class Order {
    void test() {
        Invoice inv = new Invoice();
        inv.getInvoice();  // ✗ ERROR! Private method
    }
}
```

---

### 3.3 Protected

**Can be accessed by:**
1. Classes in the **same package**
2. **Subclasses** (child classes) in **different packages**

```java
// In salesDepartment/Invoice.java
package salesDepartment;

public class Invoice {
    protected void getInvoice() {   // PROTECTED
        System.out.println("Inside invoice");
    }
}

// In salesDepartment/Order.java - SAME PACKAGE
package salesDepartment;

public class Order {
    void test() {
        Invoice inv = new Invoice();
        inv.getInvoice();  // ✓ Same package - can access
    }
}

// In humanResource/JobPortal.java - DIFFERENT PACKAGE, but SUBCLASS
package humanResource;
import salesDepartment.Invoice;

public class JobPortal extends Invoice {  // Child class
    void test() {
        getInvoice();  // ✓ Subclass - can access
    }
}

// In humanResource/Application.java - DIFFERENT PACKAGE, NOT subclass
package humanResource;
import salesDepartment.Invoice;

public class Application {  // Not a subclass
    void test() {
        Invoice inv = new Invoice();
        inv.getInvoice();  // ✗ ERROR! Not a subclass
    }
}
```

---

### 3.4 Default (No Keyword)

**Can ONLY be accessed by classes in the SAME package**

```java
// In salesDepartment/Invoice.java
package salesDepartment;

public class Invoice {
    void getInvoice() {   // DEFAULT (no keyword)
        System.out.println("Inside invoice");
    }
}

// In salesDepartment/Order.java - SAME PACKAGE
public class Order {
    void test() {
        Invoice inv = new Invoice();
        inv.getInvoice();  // ✓ Same package - can access
    }
}

// In humanResource/JobPortal.java - DIFFERENT PACKAGE
public class JobPortal extends Invoice {  // Even if subclass
    void test() {
        getInvoice();  // ✗ ERROR! Different package
    }
}
```

---

### Access Specifiers Visual Summary:

```
                    VISIBILITY SCOPE
                          │
    ┌─────────────────────┼─────────────────────┐
    │                     │                     │
SAME CLASS          SAME PACKAGE         DIFFERENT PACKAGE
    │                     │                     │
 private ─────┐           │              ┌───── public
              │           │              │
 default ─────┼───────────┤              │
              │           │              │
 protected ───┼───────────┼──────────────┤ (only subclass)
              │           │              │
 public ──────┼───────────┼──────────────┼─────────────────
              │           │              │
```

---

## 4. Return Type

Specifies what type of value the method returns after execution.

| Return Type | Description | Example |
|-------------|-------------|---------|
| `void` | Returns nothing | `public void print()` |
| `int` | Returns integer | `public int getAge()` |
| `boolean` | Returns true/false | `public boolean isEmpty()` |
| `String` | Returns string | `public String getName()` |
| `Object` | Returns any object | `public Employee getEmployee()` |

### Examples:

```java
// void - returns nothing
public void printMessage() {
    System.out.println("Hello");
    return;  // Optional for void
    // OR simply don't write return
}

// int - returns integer
public int add(int a, int b) {
    return a + b;  // Must return int
}

// boolean - returns true/false
public boolean isEven(int num) {
    return num % 2 == 0;
}
```

---

## 5. Method Naming Conventions

### Rules:

| Rule | Example |
|------|---------|
| Should be a **verb** (action) | `get`, `set`, `print`, `calculate` |
| Start with **lowercase** letter | `getAge()` ✓, `GetAge()` ✗ |
| Use **camelCase** for multiple words | `getEmployeeDetails()` |

### Examples:

```java
// ✓ Correct naming
public void printInvoice() { }
public int getEmployeeId() { }
public boolean isValid() { }
public void setName(String name) { }

// ✗ Wrong naming
public void PrintInvoice() { }  // Starts with capital
public int employeeid() { }     // Not descriptive, no camelCase
```

---

## 6. Parameters/Arguments

Parameters are variables passed to the method for computation.

```java
public int sum(int a, int b, String message) {
    //        └──────────┬──────────────┘
    //              Parameter List
    System.out.println(message);
    return a + b;
}

// Calling with arguments
sum(5, 10, "Adding numbers");  // 5, 10, "Adding..." are arguments
```

### Parameter List Can Be Empty:

```java
public void printHello() {  // No parameters
    System.out.println("Hello");
}
```

---

## 7. Types of Methods

```
                    METHODS
                       │
    ┌──────────────────┼──────────────────┐
    │                  │                  │
System-Defined    User-Defined      Special Types
    │                  │                  │
Math.sqrt()       Your methods     ┌──────┼──────┐
Arrays.sort()                      │      │      │
String.length()              Overloaded  Static  Abstract
                             Overridden  Final
```

---

### 7.1 System-Defined Methods

**Pre-built methods provided by Java libraries** (Ready to use)

```java
// Math library methods
double result = Math.sqrt(25);    // Returns 5.0
int absolute = Math.abs(-10);     // Returns 10

// String methods
String s = "Hello";
int length = s.length();          // Returns 5

// Arrays methods
int[] arr = {3, 1, 2};
Arrays.sort(arr);                 // Sorts array
```

> **Question:** Which component provides these libraries - JDK, JRE, or JVM?
> **Answer:** JRE (Java Runtime Environment) contains the class libraries.

---

### 7.2 User-Defined Methods

**Methods created by programmers based on requirements**

```java
public class MyClass {
    // User-defined method
    public int calculateArea(int length, int width) {
        return length * width;
    }
}
```

---

### 7.3 Overloaded Methods

**Multiple methods with SAME NAME but DIFFERENT PARAMETERS in the SAME class**

```java
public class Invoice {
    
    // Method 1: No parameters
    public void getInvoice() {
        System.out.println("No params");
    }
    
    // Method 2: One String parameter
    public void getInvoice(String id) {
        System.out.println("String: " + id);
    }
    
    // Method 3: One int parameter
    public void getInvoice(int id) {
        System.out.println("Int: " + id);
    }
    
    // Method 4: Two int parameters
    public void getInvoice(int id, int price) {
        System.out.println("Two ints: " + id + ", " + price);
    }
}
```

### Overloading Rules:

| Must Be Different | Not Considered |
|-------------------|----------------|
| Number of parameters | Return type |
| Type of parameters | Parameter names |
| Order of parameters | |

```java
// ✗ INVALID - Only return type is different
public void getInvoice(int id) { }
public int getInvoice(int id) { }   // ERROR! Same parameters

// ✓ VALID - Parameter type is different
public void getInvoice(int id) { }
public void getInvoice(String id) { }  // OK!
```

### Calling Overloaded Methods:

```java
Invoice inv = new Invoice();
inv.getInvoice();              // Calls no-param version
inv.getInvoice("INV001");      // Calls String version
inv.getInvoice(101);           // Calls int version
inv.getInvoice(101, 500);      // Calls two-int version
```

---

### 7.4 Overridden Methods

**Child class has the SAME method as parent class (same name, same parameters, same return type)**

```java
// Parent class
public class Person {
    public String profession() {
        return "I am in Person class";
    }
}

// Child class
public class Doctor extends Person {
    
    @Override  // Optional but recommended annotation
    public String profession() {
        return "I am a Doctor";
    }
}
```

### How Overriding Works (Dynamic Binding):

```java
public static void main(String[] args) {
    Person obj = new Doctor();  // Parent reference, Child object
    
    System.out.println(obj.profession());  
    // Output: "I am a Doctor"
    // At RUNTIME, JVM sees object is Doctor, calls Doctor's method
}
```

### Overloading vs Overriding:

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| Where | Same class | Parent-Child classes |
| Method signature | Different parameters | Exactly same |
| Binding | Compile-time (Static) | Runtime (Dynamic) |
| Return type | Not considered | Must be same (or covariant) |

---

### 7.5 Static Methods (Very Important!)

**Methods associated with the CLASS, not with objects**

```java
public class Calculation {
    
    // Static method
    public static int add(int a, int b) {
        return a + b;
    }
    
    // Non-static method
    public int multiply(int a, int b) {
        return a * b;
    }
}
```

### Calling Static vs Non-Static:

```java
// Static method - call using CLASS NAME (no object needed)
int sum = Calculation.add(5, 3);  // ✓

// Non-static method - MUST create object
Calculation obj = new Calculation();
int product = obj.multiply(5, 3);  // ✓
```

### Visual Representation:

```
        Calculation (Class)
    ┌─────────────────────────┐
    │  static add()           │  ← ONE copy, shared by all
    └─────────────────────────┘
              ▲
    ┌─────────┴─────────┐
    │                   │
┌───────────┐     ┌───────────┐
│  obj1     │     │  obj2     │
│           │     │           │
│multiply() │     │multiply() │  ← Each object has own copy
└───────────┘     └───────────┘
```

---

### Static Method Rules:

#### Rule 1: Cannot Access Non-Static Members

```java
public class Calculation {
    int stockPrice = 20;           // Non-static (instance variable)
    static int carPrice = 40;      // Static variable
    
    public static int getPriceOfPen() {
        // ✓ Can access static variable
        System.out.println(carPrice);
        
        // ✗ Cannot access non-static variable
        System.out.println(stockPrice);  // ERROR!
        
        // ✗ Cannot call non-static method
        print();  // ERROR!
    }
    
    public void print() {  // Non-static method
        System.out.println("Printing...");
    }
}
```

**Why?**
- Static methods belong to CLASS
- Non-static members belong to OBJECTS
- Static method doesn't know WHICH object's `stockPrice` to access

```
Static method asks: "Which object's stockPrice?"
                     ┌─────────┐     ┌─────────┐
                     │ obj1    │     │ obj2    │
                     │stock=20 │     │stock=30 │
                     └─────────┘     └─────────┘
                          ?               ?
```

#### Rule 2: Static Methods CANNOT Be Overridden

```java
public class Person {
    public static void profession() {
        System.out.println("Person class");
    }
}

public class Doctor extends Person {
    @Override
    public static void profession() {  // ERROR!
        System.out.println("Doctor class");
    }
}
```

**Why?**
- Overriding uses **dynamic binding** (decided at runtime based on object)
- Static methods use **static binding** (decided at compile time based on class)
- Static methods are called via CLASS NAME, not objects
- No polymorphism possible

**Note:** If you remove `@Override`, it compiles but it's **METHOD HIDING**, not overriding:

```java
Person.profession();  // Calls Person's method
Doctor.profession();  // Calls Doctor's method (hiding, not overriding)
```

---

### When to Declare a Method Static?

| Make Static If... | Don't Make Static If... |
|-------------------|------------------------|
| Only uses parameters for computation | Uses instance variables |
| Doesn't modify object state | Modifies object state |
| Utility/Helper method | Needs object-specific data |

#### ✓ Good Candidate for Static:

```java
public static int sum(int a, int b) {
    return a + b;  // Only uses parameters
}
```

#### ✗ Bad Candidate for Static:

```java
int carPrice = 40;  // Instance variable

public int sum(int a, int b) {
    carPrice = carPrice + a;  // Modifies instance variable
    return a + b;
}
```

**Problem if made static:**
```
Thread 1 calls: sum(2, 1) → carPrice = 42
Thread 2 calls: sum(5, 2) → carPrice = 47 (unexpected!)
// Race condition! Both threads share same static variable
```

---

### 7.6 Final Methods

**Methods that CANNOT be overridden by child classes**

```java
public class Person {
    public final String profession() {  // FINAL method
        return "I am in Person class";
    }
}

public class Doctor extends Person {
    @Override
    public String profession() {  // ERROR! Cannot override final
        return "I am a Doctor";
    }
}
```

**Use Case:** When you want to prevent child classes from changing the method's behavior.

---

### 7.7 Abstract Methods

**Methods with only declaration (no body), must be in abstract class**

```java
// Abstract class
public abstract class Person {
    
    // Abstract method - NO BODY, just declaration
    public abstract void print();
    
    // Regular method - has body
    public void greet() {
        System.out.println("Hello");
    }
}

// Child class MUST implement abstract methods
public class Doctor extends Person {
    
    @Override
    public void print() {  // MUST provide implementation
        System.out.println("I am a doctor");
    }
}
```

### Abstract Method Rules:

| Rule |
|------|
| Can only exist in **abstract class** |
| Has **no body** (just declaration with semicolon) |
| Child class **must implement** it |
| Cannot be `final` (contradicts purpose) |
| Cannot be `private` (must be accessible to children) |

---

## 8. Variable Arguments (Varargs)

**When you don't know how many arguments will be passed**

### Syntax: `dataType... variableName`

```java
public int sum(int... numbers) {  // Variable arguments
    int total = 0;
    for (int num : numbers) {
        total += num;
    }
    return total;
}
```

### Calling Varargs Method:

```java
sum();              // 0 arguments - OK
sum(1);             // 1 argument - OK
sum(1, 2);          // 2 arguments - OK
sum(1, 2, 3, 4, 5); // 5 arguments - OK
```

### Varargs Rules:

| Rule | Example |
|------|---------|
| Only ONE varargs per method | `void m(int... a, int... b)` ✗ |
| Must be LAST parameter | `void m(int... a, int b)` ✗ |
| Can have other params before | `void m(int a, int... b)` ✓ |

### Correct Usage:

```java
// ✓ Correct - varargs is last
public int sum(int multiplier, int... numbers) {
    int total = 0;
    for (int num : numbers) {
        total += num;
    }
    return total * multiplier;
}

// Calling
sum(2, 1, 2, 3);  // multiplier=2, numbers=[1,2,3]
                  // Result: (1+2+3) * 2 = 12
```

### Why Must Be Last?

```java
// If varargs was first, how would Java know where to stop?
sum(1, 2, 3, 4, 5, 10);
//  └──────────┘   └─ Which is multiplier?
//   varargs?          

// With varargs last:
sum(10, 1, 2, 3, 4, 5);
//  └─ multiplier
//      └──────────┘
//         varargs (all remaining)
```

---

## 9. Quick Reference Table

| Method Type | Key Characteristic |
|-------------|-------------------|
| System-defined | Pre-built in Java libraries |
| User-defined | Created by programmer |
| Overloaded | Same name, different parameters |
| Overridden | Same method in child class |
| Static | Associated with class, not object |
| Final | Cannot be overridden |
| Abstract | No body, must be implemented by child |

---

## 10. Interview Key Points

1. **Access Specifiers:** public > protected > default > private
2. **Overloading** = Same class, different parameters (compile-time binding)
3. **Overriding** = Parent-child, same signature (runtime binding)
4. **Static methods:**
    - Called using class name
    - Cannot access non-static members
    - Cannot be overridden (only hidden)
5. **Final methods** cannot be overridden
6. **Abstract methods** have no body, child must implement
7. **Varargs** (`...`) must be last parameter, only one per method
8. **Return type** is NOT considered in overloading

---

## 11. Visual Summary

```
                         METHOD DECLARATION
┌──────────────────────────────────────────────────────────────────┐
│  public    static    int    sum    (int a, int b)    throws Ex  │
│    │         │        │      │           │               │      │
│ Access    Modifier  Return  Name    Parameters      Exception   │
│Specifier            Type                                        │
└──────────────────────────────────────────────────────────────────┘

                         METHOD TYPES
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐         │
│  │ Static   │  │  Final   │  │ Abstract │  │ Overload │         │
│  │          │  │          │  │          │  │ Override │         │
│  │ Class-   │  │ Cannot   │  │ No body  │  │          │         │
│  │ level    │  │ override │  │ Child    │  │ Poly-    │         │
│  │          │  │          │  │ must     │  │ morphism │         │
│  │ ClassName│  │          │  │ implement│  │          │         │
│  │ .method()│  │          │  │          │  │          │         │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```