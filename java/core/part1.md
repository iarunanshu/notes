# Object-Oriented Programming (OOP) in Java - Notes

---

## 1. Overview of OOP

### What is OOP?
- **OOP** = Object-Oriented Programming
- Everything revolves around **objects**
- When working on any project, think: "What are the objects present?"

### Procedural vs OOP Programming

| Procedural Programming | Object-Oriented Programming |
|------------------------|----------------------------|
| Focus on **functions** | Focus on **data** |
| Data moves freely between functions | Data is **bound** with functions |
| No proper way to hide data | Provides **data hiding** |
| Tightly coupled code | Loosely coupled code |
| No code reusability | Supports code reusability |
| No inheritance/polymorphism | Supports inheritance, polymorphism |

---

## 2. Objects and Classes

### What is an Object?
A **real-world entity** that has two things:
1. **Properties/State** - Attributes you can observe (data variables)
2. **Behavior/Functionality** - Actions it can perform (methods)

**Example - Dog:**
- Properties: color, breed, age, height
- Behavior: bark(), sleep(), eat()

**Example - Car:**
- Properties: brand, type, color
- Behavior: applyBrake(), drive(), increaseSpeed()

### What is a Class?
- A **blueprint/template** from which objects are created
- Contains two things:
    - **Data variables** (properties)
    - **Data methods** (functions that work on variables)

### Creating a Class
```java
class Student {
    // Data variables (properties)
    int age;
    String address;
    
    // Data methods (behavior)
    void updateAddress(String newAddress) {
        this.address = newAddress;
    }
    
    int getAge() {
        return age;
    }
    
    void setAge(int age) {
        this.age = age;
    }
}
```

### Creating Objects
```java
Student engineeringStudent = new Student();
Student mbaStudent = new Student();
```

- **`new`** keyword creates a new object
- Each object has its **own copy** of data variables
- Each object can have **different values**

---

## 3. Four Pillars of OOP

---

### Pillar 1: Data Abstraction

**Definition:** Hiding internal implementation and showing only essential functionality to the user.

**Real-world Examples:**
1. **Car Brake Pedal** - You press it, car slows down. You don't know the internal mechanics.
2. **Cell Phone Call** - You dial and press green button. You don't know how data transfers through networks.

**How to achieve:** Through **Interfaces** and **Abstract Classes**

```java
// Interface - only exposes essential features
interface Car {
    void applyBrake();
    void pressAccelerator();
    void pressHorn();
}

// Implementation is hidden from user
class CarImplementation implements Car {
    public void applyBrake() {
        // Step 1: reduce fuel supply
        // Step 2: engage brake pads
        // Step 3: ...multiple internal steps
        // Result: Speed reduced
    }
    // Other methods...
}
```

**Advantages:**
1. **Security & Confidentiality** - Hide features you don't want to expose
2. **Simplifies client code** - Client doesn't need to know complex internal steps
3. **Prevents messing with internal implementation**

---

### Pillar 2: Data Encapsulation

**Definition:** Bundling the data and the code that works on the data into a single unit (class). Also known as **Data Hiding**.

**Simple Analogy:** Like a medicine **capsule** - actual medicine (data) is inside, protected by the outer covering (class)

```java
class Dog {
    // Data member (private for hiding)
    private String color;
    
    // Data methods (public for access)
    public String getColor() {
        return color;
    }
    
    public void setColor(String color) {
        this.color = color;
    }
}
```

**Access Specifiers (Control mechanism):**

| Modifier | Same Class | Same Package | Subclass | Other Packages |
|----------|------------|--------------|----------|----------------|
| private | ✓ | ✗ | ✗ | ✗ |
| default | ✓ | ✓ | ✗ | ✗ |
| protected | ✓ | ✓ | ✓ | ✗ |
| public | ✓ | ✓ | ✓ | ✓ |

**Key Points:**
- **Private** data cannot be accessed directly from outside the class
- Access through **public methods** (getters/setters)
- Class has **full control** over who can access/modify data

**Advantages:**
- Brings **control** over data
- Enables **loosely coupled** code
- Each class is **independent**

---

### Pillar 3: Inheritance

**Definition:** Capability of a class to inherit properties and methods from a parent class.

**Keyword:** `extends`

```java
// Parent class
class Vehicle {
    boolean hasEngine;
    
    boolean getEngine() {
        return hasEngine;
    }
}

// Child class
class Car extends Vehicle {
    String carType;  // hatchback, SUV, sedan
    
    String getCarType() {
        return carType;
    }
}
```

**Key Points:**
- Child class has access to **all** parent's variables and methods
- Child can have its **own** additional variables and methods
- **Parent cannot access child's** properties

```java
Car swift = new Car();
swift.getEngine();    // ✓ Valid - inherited from Vehicle
swift.getCarType();   // ✓ Valid - own method

Vehicle myVehicle = new Vehicle();
myVehicle.getCarType();  // ✗ Invalid - Vehicle doesn't have this
```

### Types of Inheritance

```
1. SINGLE           2. MULTILEVEL         3. HIERARCHICAL
   A                     A                      A
   |                     |                     / \
   B                     B                    B   C
                         |
                         C

4. MULTIPLE (NOT ALLOWED in Java with classes)
      A     B
       \   /
         C      ← Not allowed!
```

### Diamond Problem (Why Multiple Inheritance Not Allowed)

```java
class A {
    void getEngine() { /* implementation */ }
}

class B {
    void getEngine() { /* different implementation */ }
}

// NOT ALLOWED:
class C extends A, B {  // Which getEngine() to call?
}
```

**Problem:** If C calls `getEngine()`, JVM can't determine which parent's method to invoke.

**Solution:** Use **Interfaces** (can implement multiple interfaces)

```java
interface A {
    void getEngine();  // No implementation
}

interface B {
    void getEngine();  // No implementation
}

class C implements A, B {
    public void getEngine() {
        // C MUST provide implementation
        // No ambiguity!
    }
}
```

**Advantages of Inheritance:**
- **Code Reusability**
- Enables **Polymorphism**

---

### Pillar 4: Polymorphism

**Definition:** Same method behaves differently in different situations. ("Poly" = many, "morph" = forms)

**Real-world Examples:**
- A **person** can be father, husband, employee
- **Water** can be liquid, solid, gas

### Two Types of Polymorphism

#### A. Method Overloading (Static/Compile-time Polymorphism)

**Definition:** Same method name with **different parameters** in the **same class**

```java
class Sum {
    // Method 1: Two integers
    int doSum(int a, int b) {
        return a + b;
    }
    
    // Method 2: Two strings (different type)
    String doSum(String a, String b) {
        return a + b;
    }
    
    // Method 3: Three integers (different count)
    int doSum(int a, int b, int c) {
        return a + b + c;
    }
}
```

**Rules:**
- Same method name
- **Different parameters** (type or count must differ)
- Can have different return types
- **Return type alone CANNOT differentiate** (compilation error)

```java
// NOT VALID OVERLOADING:
int doSum(int a, int b) { return a + b; }
String doSum(int a, int b) { return ""; }  // ✗ Error! Same parameters
```

**Why return type doesn't work?**
- Compiler binds based on **arguments passed**, not return type
- When calling `doSum(5, 2)`, compiler can't know if you want int or String return

#### B. Method Overriding (Dynamic/Runtime Polymorphism)

**Definition:** Same method in **parent and child** class with **exactly same signature**

```java
class A {
    boolean getEngine() {
        return true;
    }
}

class B extends A {
    @Override
    boolean getEngine() {
        // Different implementation
        return false;
    }
}
```

**Rules:**
- Same method name
- Same return type
- Same parameters
- Only **implementation differs**
- Requires **inheritance** (parent-child relationship)

**Which method gets called?**
```java
B bObj = new B();
bObj.getEngine();  // Calls B's getEngine()

A aObj = new A();
aObj.getEngine();  // Calls A's getEngine()
```
- Determined at **runtime** based on object type
- Child class method is checked **first**; if not found, parent's method is called

### Overloading vs Overriding Summary

| Feature | Overloading | Overriding |
|---------|-------------|------------|
| Location | Same class | Parent-Child classes |
| Method name | Same | Same |
| Parameters | Must differ | Must be same |
| Return type | Can differ | Must be same |
| Binding | Compile-time | Runtime |
| Also called | Static polymorphism | Dynamic polymorphism |

---

## 4. Relationships in OOP

### IS-A Relationship (Inheritance)

- Achieved through **inheritance**
- Child "IS-A" type of Parent

```java
class Animal { }
class Dog extends Animal { }
// Dog IS-A Animal
```

**Examples:**
- Car IS-A Vehicle
- Dog IS-A Animal

---

### HAS-A Relationship (Association)

- One class **has an object** of another class as a member
- It's a data variable that is an object of another class

```java
class Engine { }

class Bike {
    Engine engine;  // Bike HAS-A Engine
}
```

**Types of Cardinality:**
- **1 to 1:** Student has one Course
- **1 to Many:** Student has List of Courses
- **Many to Many:** Student has many Courses; Course has many Students

```java
class Student {
    Course course;           // 1 to 1
    List<Course> courses;    // 1 to Many
}

class Course {
    List<Student> students;  // Many to Many (bidirectional)
}
```

**Examples:**
- Bike HAS-A Engine
- School HAS-A Student
- School HAS-A Class (room)

---

### Two Types of HAS-A Relationship

#### 1. Aggregation (Weak Relationship)

- Both objects can **exist independently**
- Destroying one does NOT destroy the other

**Example:** School and Student
```java
class School {
    List<Student> students;
}
```
- If School is destroyed → Students can enroll elsewhere
- If Students leave → School still exists

#### 2. Composition (Strong Relationship)

- Objects are **tightly coupled**
- Destroying one **destroys** the other

**Example:** School and Rooms
```java
class School {
    List<Room> rooms;
}
```
- If School is destroyed → Rooms are also destroyed
- Rooms cannot exist without School

### Aggregation vs Composition Summary

| Aspect | Aggregation | Composition |
|--------|-------------|-------------|
| Relationship | Weak | Strong |
| Dependency | Objects can exist independently | Child cannot exist without parent |
| Example | School-Student | School-Rooms |
| Destruction | One destroyed, other survives | One destroyed, other also destroyed |

---

## Interview Tips

1. **Explain like teaching a 14-year-old** - Use simple real-world examples
2. Be ready to explain **differences**:
    - Procedural vs OOP
    - Abstraction vs Encapsulation
    - Overloading vs Overriding
    - Aggregation vs Composition
3. Know the **Diamond Problem** and why multiple inheritance isn't allowed
4. Understand **why** (not just what) - e.g., why data hiding is important

---

## Quick Revision Diagram

```
                    OOP CONCEPTS
                         |
    ┌────────────────────┼────────────────────┐
    |                    |                    |
 Objects &           4 Pillars           Relationships
 Classes                 |                    |
    |         ┌──────────┼──────────┐    ┌────┴────┐
    |         |          |          |    |         |
    |    Abstraction  Encapsulation  |  IS-A    HAS-A
    |    (Interface)  (Class+Access) |   |         |
    |         |          |           |   |    ┌────┴────┐
    |    Inheritance  Polymorphism   |   | Aggregation Composition
    |    (extends)       |           |   |   (weak)    (strong)
    |         |     ┌────┴────┐      |   |
    |    Diamond   Overloading Overriding |
    |    Problem   (compile)  (runtime)   |
```