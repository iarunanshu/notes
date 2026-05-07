# Comprehensive Notes: Types of Classes in Java (Part 1)

## Overview of Class Types in Java

| Class Type | Covered in Part 1 | Covered in Part 2 |
|------------|-------------------|-------------------|
| Concrete Class | ✓ | |
| Abstract Class | ✓ | |
| Super/Subclass | ✓ | |
| Object Class | ✓ | |
| Nested Class | ✓ | |
| Generic Class | | ✓ |
| POJO | | ✓ |
| Enum | | ✓ |
| Final Class | | ✓ |
| Singleton Class | | ✓ |
| Immutable Class | | ✓ |
| Wrapper Class | | ✓ |

---

## 1. Concrete Class

### Definition
A **Concrete Class** is any class from which you can create an instance using the `new` keyword.

### Characteristics

| Feature | Description |
|---------|-------------|
| **Instantiation** | Can create objects using `new` keyword |
| **Method Implementation** | ALL methods must have complete implementation |
| **Inheritance** | Can extend a class or implement an interface |
| **Access Modifiers** | Only `public` or `default` (package-private) |

### Examples

#### Simple Concrete Class:
```java
public class Person {
    public void display() {
        System.out.println("This is a concrete class");
    }
}

// Usage
Person obj = new Person();  // ✓ Allowed
```

#### Concrete Class Implementing Interface:
```java
interface Shape {
    void draw();           // No implementation
    double area();         // No implementation
}

class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing rectangle");  // Implementation provided
    }
    
    @Override
    public double area() {
        return length * width;  // Implementation provided
    }
}

// Usage
Shape shape = new Rectangle();  // ✓ Rectangle is concrete
// Shape shape = new Shape();   // ✗ Cannot instantiate interface
```

### Class Access Modifiers

| Modifier | Scope |
|----------|-------|
| `public` | Accessible from any package |
| `default` (no modifier) | Accessible only within same package (package-private) |

> **Note:** Top-level classes cannot be `private` or `protected` (but nested classes can be!)

---

## 2. Abstract Class

### What is Abstraction?
**Abstraction** means hiding the implementation details and exposing only the features/functionality to the user.

**Real-world Example:** Car brake pedal
- User knows: Press brake → Car stops
- User doesn't know: Internal mechanism of how brakes work

### Ways to Achieve Abstraction

| Method | Abstraction Level |
|--------|-------------------|
| **Interface** | 100% abstraction (all methods are abstract by default) |
| **Abstract Class** | 0% to 100% abstraction (can have both abstract and concrete methods) |

### Abstract Class Characteristics

```java
abstract class Car {
    // Abstract method - NO implementation
    abstract void pressBrake();
    
    // Abstract method - NO implementation  
    abstract void pressClutch();
    
    // Concrete method - HAS implementation
    int getNumberOfWheels() {
        return 4;
    }
}
```

| Feature | Description |
|---------|-------------|
| **Keyword** | `abstract` before class |
| **Object Creation** | ❌ Cannot create instance directly |
| **Abstract Methods** | Can have methods without implementation |
| **Concrete Methods** | Can also have methods with implementation |
| **Child Class** | Must implement all abstract methods (unless child is also abstract) |

### Inheritance Hierarchy with Abstract Classes

```
┌─────────────────────────────────┐
│     abstract class Car          │
│  ─────────────────────────────  │
│  abstract pressBrake()          │
│  abstract pressClutch()         │
│  int getNumberOfWheels() {4}    │
└───────────────┬─────────────────┘
                │ extends
                ▼
┌─────────────────────────────────┐
│  abstract class LuxuryCar       │
│  ─────────────────────────────  │
│  abstract pressDualBrake()      │  ← Added new abstraction
│  void pressBrake() {...}        │  ← Implemented parent's abstract
│  // pressClutch still abstract  │
└───────────────┬─────────────────┘
                │ extends
                ▼
┌─────────────────────────────────┐
│      class Audi (Concrete)      │
│  ─────────────────────────────  │
│  void pressDualBrake() {...}    │  ← Must implement
│  void pressClutch() {...}       │  ← Must implement
└─────────────────────────────────┘
```

### Code Example:

```java
abstract class Car {
    abstract void pressBrake();
    abstract void pressClutch();
    
    int getNumberOfWheels() {
        return 4;
    }
}

abstract class LuxuryCar extends Car {
    abstract void pressDualBrakeSystem();  // New abstraction
    
    @Override
    void pressBrake() {
        System.out.println("Luxury brake applied");  // Implementation
    }
    // pressClutch() remains abstract
}

class Audi extends LuxuryCar {
    @Override
    void pressDualBrakeSystem() {
        System.out.println("Dual brake system activated");
    }
    
    @Override
    void pressClutch() {
        System.out.println("Clutch pressed");
    }
}
```

### Object Creation Rules:

```java
// ❌ NOT Allowed - Cannot instantiate abstract class
Car car = new Car();
LuxuryCar luxuryCar = new LuxuryCar();

// ✓ Allowed - Concrete class
Audi audi = new Audi();

// ✓ Allowed - Store child reference in parent type
Car car = new Audi();
LuxuryCar luxuryCar = new Audi();
```

---

## 3. Superclass and Subclass

### Definitions

| Term | Definition | Also Known As |
|------|------------|---------------|
| **Superclass** | The parent class from which another class inherits | Base class, Parent class |
| **Subclass** | The child class that inherits from another class | Derived class, Child class |

```java
class A {              // Superclass of B
    // ...
}

class B extends A {    // Subclass of A
    // ...
}
```

---

## 4. Object Class - The Root of All Classes

### Key Concept

> **In Java, every class that doesn't explicitly extend another class implicitly extends the `Object` class.**

```
                    ┌─────────────┐
                    │   Object    │  ← Root of all classes
                    │  (java.lang)│
                    └──────┬──────┘
           ┌───────────────┼───────────────┐
           │               │               │
           ▼               ▼               ▼
      ┌─────────┐    ┌─────────┐    ┌─────────┐
      │ Person  │    │  Car    │    │   A     │
      └─────────┘    └────┬────┘    └────┬────┘
                          │              │
                          ▼              ▼
                    ┌─────────┐    ┌─────────┐
                    │  Audi   │    │    B    │
                    └─────────┘    └─────────┘
```

### Common Methods in Object Class

| Method | Purpose |
|--------|---------|
| `toString()` | Returns string representation of object |
| `equals(Object obj)` | Compares two objects for equality |
| `hashCode()` | Returns hash code value for object |
| `clone()` | Creates and returns a copy of object |
| `getClass()` | Returns runtime class of object |
| `wait()` | Causes thread to wait (used in multithreading) |
| `notify()` | Wakes up a single waiting thread |
| `notifyAll()` | Wakes up all waiting threads |
| `finalize()` | Called by GC before object is destroyed (deprecated) |

### Practical Usage:

```java
public class ObjectTest {
    public static void main(String[] args) {
        // Object class can hold reference to ANY object
        Object obj1 = new Person();    // Person has no explicit parent
        Object obj2 = new Audi();      // Audi extends LuxuryCar extends Car
        
        // Get actual class type
        System.out.println(obj1.getClass());  // Output: class Person
        System.out.println(obj2.getClass());  // Output: class Audi
        
        // Check instance type
        if (obj1 instanceof Person) {
            Person p = (Person) obj1;  // Safe cast
        }
    }
}
```

### Why is Object Class Useful?
1. **Universal Reference:** Can store any object type
2. **Common Functionality:** Provides methods all objects need
3. **Polymorphism:** Enables generic programming before generics

---

## 5. Nested Classes

### Overview

A **Nested Class** is a class defined within another class.

```
┌────────────────────────────────────────────────────────────┐
│                      NESTED CLASSES                        │
├────────────────────────┬───────────────────────────────────┤
│   Static Nested Class  │     Non-Static (Inner Class)      │
│                        │                                   │
│   • Keyword: static    ├───────────────┬───────────────────┤
│   • Access: static     │ Member Inner  │ Local Inner │ Anonymous │
│     members only       │    Class      │   Class     │   Class   │
└────────────────────────┴───────────────┴─────────────┴───────────┘
```

### When to Use Nested Classes?

1. **Logical Grouping:** If a class is used by only ONE other class, nest it
2. **Encapsulation:** Hide implementation details
3. **Readability:** Keep related code together in one file

---

## 5.1 Static Nested Class

### Characteristics

| Feature | Description |
|---------|-------------|
| **Keyword** | `static` before class |
| **Access** | Can only access **static** members of outer class |
| **Instantiation** | Does NOT require outer class object |
| **Access Modifiers** | Can be `private`, `protected`, `public`, or default |

### Code Example:

```java
class OuterClass {
    // Instance variable (non-static)
    int instanceVar = 10;
    
    // Class variable (static)
    static int classVar = 20;
    
    // Static Nested Class
    static class NestedClass {
        void display() {
            // ❌ Cannot access: System.out.println(instanceVar);
            System.out.println(classVar);  // ✓ Can access static
        }
    }
}
```

### How to Instantiate:

```java
public class Test {
    public static void main(String[] args) {
        // No outer class object needed!
        OuterClass.NestedClass obj = new OuterClass.NestedClass();
        obj.display();
    }
}
```

### Memory Visualization:

```
┌─────────────────────────────────┐
│         OuterClass              │
│  ┌───────────────────────────┐  │
│  │ static int classVar = 20  │◄─┼─── Accessible
│  └───────────────────────────┘  │
│  ┌───────────────────────────┐  │
│  │ int instanceVar = 10      │  │◄── NOT Accessible
│  └───────────────────────────┘  │
│  ┌───────────────────────────┐  │
│  │ static class NestedClass  │  │
│  │   void display() {...}    │  │
│  └───────────────────────────┘  │
└─────────────────────────────────┘
```

### Private Static Nested Class:

```java
class OuterClass {
    // Private - only accessible within OuterClass
    private static class PrivateNested {
        void print() {
            System.out.println("Private nested class");
        }
    }
    
    // Public method to access private nested class
    public void accessPrivateNested() {
        PrivateNested obj = new PrivateNested();
        obj.print();
    }
}

// Usage
public class Test {
    public static void main(String[] args) {
        // ❌ Cannot do: OuterClass.PrivateNested obj = new OuterClass.PrivateNested();
        
        // ✓ Must go through outer class
        OuterClass outer = new OuterClass();
        outer.accessPrivateNested();
    }
}
```

---

## 5.2 Non-Static Nested Class (Inner Class)

### Types of Inner Classes

| Type | Location | Scope |
|------|----------|-------|
| **Member Inner Class** | Directly inside outer class | Entire outer class |
| **Local Inner Class** | Inside a method/block | Within that block only |
| **Anonymous Inner Class** | Inline, no name | Where defined |

---

### 5.2.1 Member Inner Class

### Characteristics

| Feature | Description |
|---------|-------------|
| **Keyword** | No `static` keyword |
| **Access** | Can access ALL members (static + instance) of outer class |
| **Instantiation** | REQUIRES outer class object |
| **Access Modifiers** | Can be `private`, `protected`, `public`, or default |

### Code Example:

```java
class OuterClass {
    int instanceVar = 10;        // Instance variable
    static int classVar = 20;    // Class variable
    
    // Member Inner Class (no static keyword)
    class InnerClass {
        void display() {
            System.out.println(instanceVar);  // ✓ Can access
            System.out.println(classVar);     // ✓ Can access
        }
    }
}
```

### How to Instantiate:

```java
public class Test {
    public static void main(String[] args) {
        // Step 1: Create outer class object FIRST
        OuterClass outer = new OuterClass();
        
        // Step 2: Use outer object to create inner class object
        OuterClass.InnerClass inner = outer.new InnerClass();
        
        inner.display();
    }
}
```

### Comparison: Static vs Non-Static Nested Class

| Feature | Static Nested | Inner Class (Non-Static) |
|---------|---------------|--------------------------|
| Keyword | `static class` | `class` (no static) |
| Access to outer members | Static only | All (static + instance) |
| Requires outer object | ❌ No | ✓ Yes |
| Instantiation | `new Outer.Nested()` | `outer.new Inner()` |
| Associated with | Class | Object |

```java
// Static Nested - Associated with CLASS
OuterClass.NestedClass obj = new OuterClass.NestedClass();

// Inner Class - Associated with OBJECT
OuterClass outer = new OuterClass();
OuterClass.InnerClass inner = outer.new InnerClass();
```

---

### 5.2.2 Local Inner Class

### Characteristics

| Feature | Description |
|---------|-------------|
| **Location** | Inside a method, loop, or any block `{}` |
| **Scope** | Only within that block |
| **Access Modifiers** | ❌ Cannot be private, protected, or public |
| **Access** | Can access outer class members + local variables of the block |
| **Instantiation** | Only within the same block |

### Code Example:

```java
class OuterClass {
    int instanceVar = 10;
    static int classVar = 20;
    
    void display() {
        int localVar = 30;  // Method local variable
        
        // Local Inner Class - defined inside method
        class LocalInnerClass {
            int innerVar = 40;
            
            void print() {
                System.out.println(instanceVar);  // ✓ Outer instance var
                System.out.println(classVar);     // ✓ Outer static var
                System.out.println(localVar);     // ✓ Method local var
                System.out.println(innerVar);     // ✓ Own variable
            }
        }
        
        // Object can ONLY be created here, inside this block
        LocalInnerClass local = new LocalInnerClass();
        local.print();
        
    }  // LocalInnerClass scope ends here
}
```

### Why No Access Modifiers?

```
Method display() called
        │
        ▼
┌─────────────────────────────┐
│   Stack Frame for display() │
│  ┌────────────────────────┐ │
│  │ localVar = 30          │ │
│  │ LocalInnerClass        │ │  ← Lives only in this frame
│  └────────────────────────┘ │
└─────────────────────────────┘
        │
        ▼ Method ends
     Frame destroyed
     LocalInnerClass gone!
```

- The class exists only within the method's stack frame
- When method ends, stack frame is destroyed
- No point of `public` - can't access from outside anyway
- No point of `private` - already restricted to this block

### How to Use:

```java
public class Test {
    public static void main(String[] args) {
        OuterClass outer = new OuterClass();
        outer.display();  // This internally creates and uses LocalInnerClass
    }
}
```

---

### 5.2.3 Anonymous Inner Class

### Definition
An **Anonymous Inner Class** is an inner class **without a name**, defined and instantiated in a single expression.

### When to Use?
When you want to override behavior of a class/interface **without creating a separate subclass file**.

### Syntax:

```java
ParentType reference = new ParentType() {
    // Override methods here
    @Override
    void method() {
        // Implementation
    }
};  // Note the semicolon!
```

### Code Example:

```java
abstract class Car {
    abstract void pressBrake();
}

public class Test {
    public static void main(String[] args) {
        // Anonymous class - no name, implements abstract method inline
        Car audiCar = new Car() {
            @Override
            void pressBrake() {
                System.out.println("Audi brake pressed");
            }
        };  // Semicolon required!
        
        audiCar.pressBrake();  // Output: Audi brake pressed
    }
}
```

### What Happens Behind the Scenes?

When you compile the above code:

```
Source Files:          Compiled Files:
─────────────          ───────────────
Test.java        →     Test.class
                       Test$1.class  ← Anonymous class!
```

**Compiler generates:**
```java
// This is what compiler creates internally
class Test$1 extends Car {
    @Override
    void pressBrake() {
        System.out.println("Audi brake pressed");
    }
}
```

### Anonymous Class with Interface:

```java
interface Greeting {
    void sayHello();
}

public class Test {
    public static void main(String[] args) {
        // Anonymous class implementing interface
        Greeting greeting = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("Hello!");
            }
        };
        
        greeting.sayHello();
    }
}
```

### Anonymous Class Summary:

| Feature | Description |
|---------|-------------|
| **Name** | None (compiler generates one) |
| **Extends/Implements** | Exactly one class OR one interface |
| **Usage** | One-time use, quick implementation |
| **Syntax** | `new Type() { ... };` |
| **Common Use Cases** | Event handlers, callbacks, threading |

---

## 6. Inheritance in Nested Classes

### 6.1 Inner Class Extending Another Inner Class (Same Outer)

```java
class OuterClass {
    class InnerClass1 {
        int var1 = 10;
        void display() {
            System.out.println("InnerClass1");
        }
    }
    
    class InnerClass2 extends InnerClass1 {
        int var2 = 20;
        void show() {
            display();  // ✓ Inherited from InnerClass1
            System.out.println(var1);  // ✓ Inherited
            System.out.println(var2);  // Own variable
        }
    }
}

// Usage
OuterClass outer = new OuterClass();
OuterClass.InnerClass2 inner = outer.new InnerClass2();
inner.show();
```

### 6.2 External Class Extending Static Nested Class

```java
class OuterClass {
    static class NestedClass {
        void display() {
            System.out.println("Nested class method");
        }
    }
}

// Different class extending static nested class - EASY!
class SomeOtherClass extends OuterClass.NestedClass {
    void show() {
        display();  // Inherited method
    }
}

// Usage - No special constructor needed
SomeOtherClass obj = new SomeOtherClass();
obj.show();
```

### 6.3 External Class Extending Non-Static Inner Class

This is tricky because inner class is associated with an outer class **object**.

```java
class OuterClass {
    class InnerClass {
        void display() {
            System.out.println("Inner class method");
        }
    }
}

// External class extending inner class - REQUIRES special constructor
class SomeOtherClass extends OuterClass.InnerClass {
    
    // MUST have this constructor!
    SomeOtherClass(OuterClass outer) {
        outer.super();  // Call inner class constructor via outer object
    }
    
    void show() {
        display();  // Inherited method
    }
}

// Usage
OuterClass outer = new OuterClass();
SomeOtherClass obj = new SomeOtherClass(outer);  // Pass outer object
obj.show();
```

### Why is the Constructor Required?

```
Normal Inheritance:
class B extends A → B constructor calls super() → A constructor

Inner Class Inheritance:
class C extends Outer.Inner
     │
     ▼
C constructor must:
1. Have access to Outer object (inner is associated with outer object)
2. Call outer.super() to invoke Inner's constructor
```

---

## 7. Quick Reference Summary

### Nested Class Instantiation Cheat Sheet

| Type | Syntax | Outer Object Needed? |
|------|--------|---------------------|
| **Static Nested** | `new Outer.Nested()` | ❌ No |
| **Member Inner** | `outer.new Inner()` | ✓ Yes |
| **Local Inner** | Inside block only | N/A |
| **Anonymous** | `new Type() {...};` | Depends |

### Access Comparison

| Class Type | Instance Vars | Static Vars | Local Vars |
|------------|---------------|-------------|------------|
| Static Nested | ❌ | ✓ | ❌ |
| Member Inner | ✓ | ✓ | ❌ |
| Local Inner | ✓ | ✓ | ✓ |
| Anonymous | ✓ | ✓ | ✓ (effectively final) |

### Access Modifiers Allowed

| Class Type | public | protected | default | private |
|------------|--------|-----------|---------|---------|
| Top-level Class | ✓ | ❌ | ✓ | ❌ |
| Static Nested | ✓ | ✓ | ✓ | ✓ |
| Member Inner | ✓ | ✓ | ✓ | ✓ |
| Local Inner | ❌ | ❌ | ✓ (only) | ❌ |
| Anonymous | N/A | N/A | N/A | N/A |

---

## 8. Interview Tips

1. **Know the difference** between nested class and inner class (static vs non-static)
2. **Remember:** Top-level classes can only be `public` or default, but nested classes can be `private`/`protected`
3. **Object class** is the parent of all classes - common interview question!
4. **Anonymous classes** are frequently asked - know the syntax and what compiler generates
5. **Instantiation syntax** differs for static nested vs inner class - practice both
6. **Abstract classes** can have 0-100% abstraction; interfaces have 100%