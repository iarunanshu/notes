# Java Constructors - Comprehensive Notes

---

## 1. What is a Constructor?

A constructor is a special block of code used to:
1. **Create an instance** (object) of a class
2. **Initialize instance variables**

```java
public class Employee {
    int employeeId;
    String name;
    
    // Constructor
    public Employee() {
        employeeId = 0;
        name = "Unknown";
    }
}

// Creating object
Employee emp = new Employee();
```

---

## 2. Constructor vs Method

| Aspect | Constructor | Method |
|--------|-------------|--------|
| Name | **Same as class name** | Any valid name |
| Return type | **No return type** (not even void) | Must have return type |
| Purpose | Create & initialize object | Perform operations |
| Called by | `new` keyword | Object reference |
| Inheritance | **Cannot be inherited** | Can be inherited |
| `static` | **Cannot be static** | Can be static |
| `final` | **Cannot be final** | Can be final |
| `abstract` | **Cannot be abstract** | Can be abstract |

### Visual Difference:

```java
public class Employee {
    
    // CONSTRUCTOR - No return type, same name as class
    public Employee() {
        System.out.println("Constructor called");
    }
    
    // METHOD - Has return type (even void), same name allowed but not recommended
    public void Employee() {  // This is a METHOD, not constructor!
        System.out.println("Method called");
    }
    
    // METHOD - With return type
    public int Employee(int x) {  // This is also a METHOD!
        return x;
    }
}
```

---

## 3. Role of `new` Keyword

**Question:** Does `new` create the object or does the constructor?

**Answer:** Both work together!
- `new` tells JVM to **call the constructor** (not a method with same name)
- Constructor **creates and initializes** the object

```java
Employee emp = new Employee();
//              ↑
//              new tells JVM: "Call CONSTRUCTOR, not method!"
```

**Why is `new` needed?**
- Methods can also have the same name as class
- `new` keyword differentiates: "Call constructor, not method"

---

## 4. Frequently Asked Interview Questions

### Q1: Why is constructor name same as class name?

**Answer:** For **easy identification**. By looking at any class, you can immediately identify which block is the constructor.

```java
public class Employee {
    // Easy to spot - same name as class = Constructor
    public Employee() { }
    
    public void work() { }
    public int calculate() { return 0; }
}
```

---

### Q2: Why doesn't constructor have a return type?

**Answer:**
- Constructor **implicitly returns** the object of the class
- Java automatically adds the class type as return type internally
- If we add return type, it becomes a **method**, not constructor

```java
public class Employee {
    
    // CONSTRUCTOR - implicitly returns Employee object
    public Employee() { }
    
    // METHOD - explicitly has return type
    public void Employee() { }  // This is a method!
}
```

> **Key Point:** No return type = Constructor. Any return type (even void) = Method

---

### Q3: Why can't constructor be `final`?

**Answer:**
- `final` prevents **overriding**
- Constructors **cannot be inherited**, so cannot be overridden
- If cannot override → `final` is meaningless

```
Constructor cannot be inherited
        ↓
Cannot be overridden
        ↓
No point of making it final
```

**Why can't constructors be inherited?**

```java
class Employee {
    public Employee() { }  // Constructor
}

class Manager extends Employee {
    // If Employee() constructor was inherited here...
    // It would violate the rule: "Constructor name = Class name"
    // Because this class is Manager, not Employee!
}
```

---

### Q4: Why can't constructor be `abstract`?

**Answer:**
- `abstract` methods have **no body** - child class provides implementation
- Constructor **cannot be inherited**
- If not inherited → child class can't provide implementation
- Therefore, `abstract` is meaningless for constructors

```java
abstract class Employee {
    // If this was allowed (IT'S NOT!):
    public abstract Employee();  // ERROR!
    
    // Child class can't implement it because
    // constructor is not inherited!
}
```

---

### Q5: Why can't constructor be `static`?

**Answer:**

**Reason 1:** Static methods can only access **static variables**
- Constructor's primary purpose is to **initialize instance variables**
- If static, constructor can't access instance variables!

```java
public class Employee {
    int employeeId;  // Instance variable
    
    // If constructor was static (NOT ALLOWED!):
    public static Employee() {
        this.employeeId = 10;  // ERROR! Can't access instance variable
    }
}
```

**Reason 2:** Static methods can't use `this` or `super`
- Constructor chaining uses `this()` and `super()`
- Static constructor would break chaining mechanism

---

### Q6: Can we define constructor in an interface?

**Answer:** **NO!**

**Reason:**
- You **cannot create object** of an interface
- Constructor's purpose is to **create objects**
- No object creation → No need for constructor

```java
interface Employee {
    // Employee() { }  // NOT ALLOWED!
    
    void work();  // Only method declarations allowed
}

// Cannot do this:
// Employee emp = new Employee();  // ERROR!

// Can only do this:
class Manager implements Employee {
    public Manager() { }  // Constructor for Manager, not interface
    
    public void work() { }
}
```

---

## 5. Types of Constructors

```
                    CONSTRUCTORS
                         │
    ┌────────────────────┼────────────────────┐
    │                    │                    │
 Default          No-Argument          Parameterized
(Auto-added)    (Manual, no params)   (Manual, with params)
    │                    │                    │
    │                    │                    │
    └────────────────────┴────────────────────┘
                         │
                    Can be Overloaded
                         │
                    Can be Private
```

---

### 5.1 Default Constructor

**Automatically added by Java** when you don't define any constructor.

```java
// Your code:
public class Calculation {
    String name;
}

// What Java adds internally (after compilation):
public class Calculation {
    String name;
    
    // Default constructor added by Java
    public Calculation() {
        super();  // Calls parent constructor
        // Sets default values: name = null
    }
}
```

**Verify using decompilation:**
```bash
javac Calculation.java
javap -c Calculation.class  # Shows default constructor
```

**Default Constructor:**
- Has **no parameters**
- Sets **default values** for all instance variables
- Calls **parent constructor** using `super()`

---

### 5.2 No-Argument Constructor

**Manually defined** constructor with **no parameters**.

```java
public class Calculation {
    String name;
    
    // No-argument constructor (manually defined)
    public Calculation() {
        name = "Default";
    }
}
```

**Difference from Default Constructor:**

| Default Constructor | No-Argument Constructor |
|---------------------|------------------------|
| Added **automatically** by Java | Defined **manually** by programmer |
| Added only if NO constructor defined | You explicitly write it |
| Sets default values (null, 0, false) | You control initialization |

---

### 5.3 Parameterized Constructor

Constructor that **accepts parameters** to initialize instance variables.

```java
public class Employee {
    String name;
    int employeeId;
    
    // Parameterized constructor
    public Employee(String name, int employeeId) {
        this.name = name;           // 'this' refers to instance variable
        this.employeeId = employeeId;
    }
}

// Usage:
Employee emp = new Employee("Shreyansh", 101);
```

### Using `this` keyword:

```java
public Employee(String name) {
    this.name = name;
//  ↑         ↑
//  │         └── Parameter 'name'
//  └── Instance variable 'name'
}
```

---

### 5.4 Constructor Overloading

**Multiple constructors** with **different parameters** in the same class.

```java
public class Employee {
    String name;
    int employeeId;
    
    // Constructor 1: No parameters
    public Employee() {
        this.name = "Unknown";
        this.employeeId = 0;
    }
    
    // Constructor 2: One parameter
    public Employee(String name) {
        this.name = name;
        this.employeeId = 0;
    }
    
    // Constructor 3: Two parameters
    public Employee(String name, int employeeId) {
        this.name = name;
        this.employeeId = employeeId;
    }
}

// Usage:
Employee e1 = new Employee();                    // Calls Constructor 1
Employee e2 = new Employee("John");              // Calls Constructor 2
Employee e3 = new Employee("Jane", 101);         // Calls Constructor 3
```

> **Note:** There is NO constructor **overriding** because constructors cannot be inherited!

---

### 5.5 Private Constructor

Constructor declared as `private` - **restricts object creation** from outside the class.

```java
public class Calculation {
    private static Calculation instance;
    
    // Private constructor
    private Calculation() {
        System.out.println("Object created");
    }
    
    // Public method to get instance (Singleton pattern)
    public static Calculation getInstance() {
        if (instance == null) {
            instance = new Calculation();  // Only this class can call constructor
        }
        return instance;
    }
}

// Usage:
// Calculation obj = new Calculation();  // ERROR! Constructor is private

Calculation obj = Calculation.getInstance();  // Correct way
```

**Use Cases:**
1. **Singleton Pattern** - Only one object should exist
2. **Factory Pattern** - Control object creation
3. **Utility Classes** - Prevent instantiation (like `Math` class)

**Why `getInstance()` is static?**
- Cannot create object without calling constructor
- Constructor is private, can't call from outside
- Need a way to call without object → **static method**

---

## 6. Important Rule: Default Constructor Removed

> **When you define ANY constructor manually, Java does NOT add default constructor!**

```java
// Case 1: No constructor defined
public class Employee {
    String name;
}
// Java adds: public Employee() { }

// Case 2: Parameterized constructor defined
public class Employee {
    String name;
    
    public Employee(String name) {
        this.name = name;
    }
}

// Now this FAILS:
Employee emp = new Employee();  // ERROR! No default constructor!

// Must use:
Employee emp = new Employee("John");  // OK
```

**Solution:** Define both constructors if needed:

```java
public class Employee {
    String name;
    
    public Employee() {           // No-arg constructor
        this.name = "Unknown";
    }
    
    public Employee(String name) {  // Parameterized constructor
        this.name = name;
    }
}
```

---

## 7. Constructor Chaining

Calling one constructor from another. Two ways:
1. **`this()`** - Call another constructor in **same class**
2. **`super()`** - Call **parent class** constructor

---

### 7.1 Chaining with `this()`

```java
public class Employee {
    String name;
    int employeeId;
    int age;
    
    // Constructor 1
    public Employee() {
        this("Unknown", 0);  // Calls Constructor 2
    }
    
    // Constructor 2
    public Employee(String name, int employeeId) {
        this(name, employeeId, 0);  // Calls Constructor 3
    }
    
    // Constructor 3 - Main constructor
    public Employee(String name, int employeeId, int age) {
        this.name = name;
        this.employeeId = employeeId;
        this.age = age;
    }
}
```

**Execution Flow:**

```
new Employee()
      │
      ▼
Constructor 1: this("Unknown", 0)
      │
      ▼
Constructor 2: this("Unknown", 0, 0)
      │
      ▼
Constructor 3: Sets all values
      │
      ▼
Object Created!
```

### Rules for `this()`:
| Rule |
|------|
| Must be **first statement** in constructor |
| Can only call **one** constructor |
| Cannot have both `this()` and `super()` |

---

### 7.2 Chaining with `super()`

**Calls parent class constructor** from child class.

```java
// Parent class
class Person {
    String name;
    
    public Person() {
        System.out.println("Person no-arg constructor");
    }
    
    public Person(String name) {
        this.name = name;
        System.out.println("Person parameterized constructor");
    }
}

// Child class
class Manager extends Person {
    int managerId;
    
    public Manager() {
        super();  // Calls Person()
        System.out.println("Manager constructor");
    }
    
    public Manager(String name, int managerId) {
        super(name);  // Calls Person(String name)
        this.managerId = managerId;
        System.out.println("Manager parameterized constructor");
    }
}
```

**Execution:**

```java
Manager m = new Manager();

// Output:
// Person no-arg constructor     ← Parent first
// Manager constructor           ← Then child

Manager m2 = new Manager("John", 101);

// Output:
// Person parameterized constructor
// Manager parameterized constructor
```

---

### 7.3 Implicit `super()`

**Java automatically adds `super()` if you don't write it!**

```java
class Person {
    public Person() {
        System.out.println("Person constructor");
    }
}

class Manager extends Person {
    public Manager() {
        // super();  ← Java adds this automatically!
        System.out.println("Manager constructor");
    }
}
```

**But if parent has only parameterized constructor:**

```java
class Person {
    String name;
    
    public Person(String name) {  // Only parameterized
        this.name = name;
    }
}

class Manager extends Person {
    public Manager() {
        // super();  ← Java tries to add this, but Person() doesn't exist!
        // ERROR: There is no default constructor in Person
    }
    
    // Correct way:
    public Manager(String name) {
        super(name);  // Must explicitly call parameterized constructor
    }
}
```

---

### Constructor Chaining Order

```
                    CONSTRUCTOR CALL ORDER
                    
new Manager("John", 101)
        │
        ▼
┌─────────────────────────────────────────┐
│ Manager(String name, int managerId) {   │
│     super(name);  ──────────────────────┼──┐
│     this.managerId = managerId;         │  │
│ }                                       │  │
└─────────────────────────────────────────┘  │
                                             │
        ┌────────────────────────────────────┘
        ▼
┌─────────────────────────────────────────┐
│ Person(String name) {                   │  ← EXECUTES FIRST
│     this.name = name;                   │
│ }                                       │
└─────────────────────────────────────────┘
        │
        ▼
    Returns to Manager constructor
        │
        ▼
    Object fully created!
```

**Key Points:**
- Parent constructor **always executes first**
- Then child constructor completes
- This ensures parent state is initialized before child uses it

---

## 8. Summary Diagram

```
                        CONSTRUCTOR
                             │
         ┌───────────────────┼───────────────────┐
         │                   │                   │
     PROPERTIES          TYPES              CHAINING
         │                   │                   │
    ┌────┴────┐        ┌─────┴─────┐        ┌────┴────┐
    │         │        │           │        │         │
Same name  No return  Default  Parameterized  this()  super()
as class     type       │           │          │         │
    │         │    No-arg    Overloaded   Same     Parent
    │         │        │           │      class    class
    │         │    Private         │
    │         │                    │
    └─────────┴────────────────────┘
              │
         Cannot be:
    • static  • final  • abstract
    • inherited  • in interface
```

---

## 9. Interview Quick Reference

| Question | Answer |
|----------|--------|
| What is constructor? | Creates object + initializes variables |
| Why same name as class? | Easy identification |
| Why no return type? | Implicitly returns class object |
| Why can't be `static`? | Can't access instance variables, can't use `this`/`super` |
| Why can't be `final`? | Can't be inherited → can't be overridden → `final` meaningless |
| Why can't be `abstract`? | Can't be inherited → can't be implemented |
| Why can't be in interface? | Can't create interface objects |
| When is default constructor added? | Only when NO constructor is defined |
| What is `this()`? | Calls another constructor in same class |
| What is `super()`? | Calls parent class constructor |
| Is `super()` always called? | Yes, explicitly or implicitly (first line) |

---

## 10. Common Mistakes

```java
// Mistake 1: Adding return type
public void Employee() { }  // This is a METHOD, not constructor!

// Mistake 2: Forgetting super() when parent has only parameterized constructor
class Child extends Parent {
    public Child() { }  // ERROR if Parent has no default constructor
}

// Mistake 3: this() or super() not on first line
public Employee() {
    System.out.println("Hi");
    this("John");  // ERROR! Must be first line
}

// Mistake 4: Both this() and super()
public Employee() {
    super();
    this("John");  // ERROR! Can't have both
}

// Mistake 5: Expecting default constructor when parameterized exists
class Employee {
    public Employee(String name) { }
}
Employee e = new Employee();  // ERROR! No default constructor
```