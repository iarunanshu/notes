# Comprehensive Notes: Java Interfaces (Part 1)

## Overview - Interface Topics

| Part 1 (This Video) | Part 2 (Next Video) |
|---------------------|---------------------|
| ✓ What is Interface | Java 8 Features (default, static methods) |
| ✓ How to Define Interface | Java 9 Features (private methods) |
| ✓ Why We Need Interface | |
| ✓ Methods & Fields in Interface | |
| ✓ Interface Implementation | |
| ✓ Nested Interfaces | |
| ✓ Interface vs Abstract Class | |

---

## 1. What is an Interface?

### Definition

An **Interface** is a contract that helps two systems interact with each other **without one system knowing the internal details** of the other.

In short: **Interface helps achieve ABSTRACTION**

### Real-World Analogy

```
┌─────────────────┐                    ┌─────────────────┐
│    YOU          │                    │      CAR        │
│   (System 1)    │                    │   (System 2)    │
│                 │    INTERFACE       │                 │
│                 │   ┌─────────┐      │  Complex brake  │
│  Press brake ──────►│ Brake   │─────►│  mechanism,     │
│                 │   │ Pedal   │      │  hydraulics,    │
│  Don't know how │   └─────────┘      │  ABS system...  │
│  braking works  │                    │                 │
└─────────────────┘                    └─────────────────┘
        │                                      │
        │         ABSTRACTION                  │
        │    (Hidden implementation)           │
        └──────────────────────────────────────┘
```

### Programming Example

```
┌─────────────────┐                    ┌─────────────────┐
│   System 1      │    INTERFACE       │    System 2     │
│   (Client)      │   ┌─────────┐      │   (Service)     │
│                 │   │ fly()   │      │                 │
│  bird.fly() ───────►│ eat()   │─────►│  Huge codebase  │
│                 │   │ walk()  │      │  Complex logic  │
│  Doesn't know   │   └─────────┘      │  Implementation │
│  HOW it flies   │                    │  details...     │
└─────────────────┘                    └─────────────────┘
```

---

## 2. How to Define an Interface

### Basic Syntax

```java
[modifier] interface InterfaceName [extends Interface1, Interface2, ...] {
    // constants
    // abstract methods
}
```

### Components Breakdown

```
┌──────────────────────────────────────────────────────────────┐
│                    INTERFACE DEFINITION                       │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   public interface Bird extends LivingThing, Flyable {       │
│   ──────  ─────────  ────  ───────  ─────────────────        │
│     │         │       │       │            │                 │
│     │         │       │       │            └─ Parent interfaces│
│     │         │       │       └─ Interface name              │
│     │         │       └─ extends keyword (for interfaces)    │
│     │         └─ interface keyword                           │
│     └─ Modifier (public or default only)                     │
│                                                              │
│       void fly();   // abstract method                       │
│   }                                                          │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Modifier Rules

| Allowed | Not Allowed |
|---------|-------------|
| `public` | `protected` |
| default (no modifier) | `private` |

### Examples

```java
// Public interface
public interface Bird {
    void fly();
}

// Default (package-private) interface
interface Animal {
    void eat();
}

// Interface extending multiple interfaces
public interface NonFlyingBird extends Bird, LivingThing {
    void walk();
}
```

### Important: Interface Can Only Extend Interfaces

```java
// ✓ Valid - extending interfaces
interface C extends InterfaceA, InterfaceB { }

// ❌ Invalid - cannot extend a class
interface C extends SomeClass { }  // COMPILE ERROR!
```

---

## 3. Why We Need Interfaces

### Three Main Reasons

| # | Reason | Description |
|---|--------|-------------|
| 1 | **Abstraction** | Defines WHAT to do, not HOW to do it |
| 2 | **Polymorphism** | Interface can hold references to implementing classes |
| 3 | **Multiple Inheritance** | A class can implement multiple interfaces |

---

### 3.1 Abstraction

Interface defines **what a class must do** but **not how it will do it**.

```java
// Interface - defines WHAT (signature only)
public interface Bird {
    void fly();  // Only signature, no implementation
}

// Class - defines HOW (provides implementation)
public class Eagle implements Bird {
    @Override
    public void fly() {
        System.out.println("Eagle soars high in the sky");
        // Actual implementation of HOW to fly
    }
}

public class Sparrow implements Bird {
    @Override
    public void fly() {
        System.out.println("Sparrow flaps wings rapidly");
        // Different implementation of HOW to fly
    }
}
```

### Abstraction Visual

```
┌─────────────────────────────────────────────────────────┐
│                  INTERFACE (Bird)                        │
│                  ────────────────                        │
│                    void fly();                           │
│                  (WHAT to do)                            │
└─────────────────────────┬───────────────────────────────┘
                          │
          ┌───────────────┼───────────────┐
          │               │               │
          ▼               ▼               ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│     Eagle       │ │    Sparrow      │ │      Hen        │
│   ───────────   │ │   ───────────   │ │   ───────────   │
│   void fly() {  │ │   void fly() {  │ │   void fly() {  │
│     // soars    │ │     // flaps    │ │     // flutters │
│   }             │ │   }             │ │   }             │
│  (HOW to do)    │ │  (HOW to do)    │ │  (HOW to do)    │
└─────────────────┘ └─────────────────┘ └─────────────────┘
```

> **100% Abstraction:** All methods in interface are abstract (before Java 8)

---

### 3.2 Polymorphism

Interface can be used as a **data type** to hold references of implementing classes.

```java
public interface Bird {
    void fly();
}

public class Eagle implements Bird {
    @Override
    public void fly() {
        System.out.println("Eagle flying high");
    }
}

public class Hen implements Bird {
    @Override
    public void fly() {
        System.out.println("Hen flying low");
    }
}
```

### Usage - Interface as Data Type

```java
public static void main(String[] args) {
    // Interface reference holding Eagle object
    Bird birdObj1 = new Eagle();
    
    // Interface reference holding Hen object
    Bird birdObj2 = new Hen();
    
    // Runtime polymorphism - method determined at runtime
    birdObj1.fly();  // Output: Eagle flying high
    birdObj2.fly();  // Output: Hen flying low
}
```

### How Runtime Polymorphism Works

```
birdObj1.fly()
      │
      ▼
┌─────────────────────────────────────┐
│  JVM Lookup:                        │
│  What object does birdObj1 hold?    │
│         │                           │
│         ▼                           │
│     Eagle object                    │
│         │                           │
│         ▼                           │
│  Call Eagle's fly() method          │
└─────────────────────────────────────┘
      │
      ▼
Output: "Eagle flying high"
```

### Important Points

| Point | Explanation |
|-------|-------------|
| Cannot instantiate interface | `new Bird()` is NOT allowed |
| Can hold reference | Interface variable can hold implementing class objects |
| Runtime decision | JVM decides which method to call at runtime |

---

### 3.3 Multiple Inheritance (Diamond Problem Solution)

### The Diamond Problem with Classes

```java
// If multiple inheritance was allowed with classes...
class WaterAnimal {
    public void canBreathe() {
        System.out.println("Breathes in water");
    }
}

class LandAnimal {
    public void canBreathe() {
        System.out.println("Breathes on land");
    }
}

// ❌ NOT ALLOWED in Java - causes ambiguity
class Crocodile extends WaterAnimal, LandAnimal {
    // Which canBreathe() to use? AMBIGUOUS!
}
```

### Diamond Problem Visual

```
        ┌─────────────────┐
        │   WaterAnimal   │
        │ canBreathe()    │
        └────────┬────────┘
                 │
                 │         ┌─────────────────┐
                 │         │   LandAnimal    │
                 │         │ canBreathe()    │
                 │         └────────┬────────┘
                 │                  │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌─────────────────┐
                 │   Crocodile     │
                 │  ??????????     │ ◄── Which canBreathe()?
                 └─────────────────┘
                 
        This is the DIAMOND PROBLEM!
```

### Solution: Multiple Inheritance Through Interfaces

```java
// Interfaces - only signatures, no implementation
interface WaterAnimal {
    boolean canBreathe();  // Just signature
}

interface LandAnimal {
    boolean canBreathe();  // Just signature
}

// ✓ ALLOWED - Class implements multiple interfaces
class Crocodile implements WaterAnimal, LandAnimal {
    
    // Crocodile provides its OWN implementation
    @Override
    public boolean canBreathe() {
        return true;  // No ambiguity - one implementation
    }
}
```

### Why This Works

```
┌─────────────────┐     ┌─────────────────┐
│  WaterAnimal    │     │   LandAnimal    │
│  (interface)    │     │   (interface)   │
│  canBreathe()   │     │   canBreathe()  │
│  [signature]    │     │   [signature]   │
└────────┬────────┘     └────────┬────────┘
         │                       │
         └───────────┬───────────┘
                     │
                     ▼
          ┌─────────────────────┐
          │     Crocodile       │
          │  canBreathe() {     │
          │    return true;     │ ◄── Crocodile's OWN implementation
          │  }                  │     No ambiguity!
          └─────────────────────┘
```

### Usage

```java
public static void main(String[] args) {
    Crocodile croc = new Crocodile();
    System.out.println(croc.canBreathe());  // Output: true
    
    // Can also use interface references
    WaterAnimal wa = croc;
    LandAnimal la = croc;
    
    wa.canBreathe();  // Calls Crocodile's implementation
    la.canBreathe();  // Calls Crocodile's implementation (same)
}
```

---

## 4. Methods in Interface

### Key Rules for Methods

| Rule | Description |
|------|-------------|
| **Implicitly public** | All methods are `public` by default |
| **Implicitly abstract** | All methods are `abstract` by default (before Java 8) |
| **Cannot be final** | Methods cannot be declared `final` |

### Why Methods Cannot Be Final?

```java
public interface Bird {
    final void fly();  // ❌ COMPILE ERROR!
}
```

**Reason:**
- `final` means "cannot be overridden"
- Interface methods MUST be overridden by implementing classes
- Therefore, `final` is contradictory and not allowed

### Method Declaration Examples

```java
public interface Bird {
    // Both are exactly the same - implicitly public abstract
    void fly();
    public abstract void fly();  // Explicit (redundant)
    
    // ❌ Invalid declarations
    final void walk();       // Cannot be final
    private void eat();      // Cannot be private (before Java 9)
    protected void swim();   // Cannot be protected
}
```

### Implicit Modifiers Visual

```
┌───────────────────────────────────────────────────────────┐
│                    INTERFACE METHODS                       │
├───────────────────────────────────────────────────────────┤
│                                                           │
│   What you write:          What compiler sees:            │
│   ─────────────────        ─────────────────────          │
│                                                           │
│   void fly();       ═══►   public abstract void fly();    │
│                                                           │
│   public void eat(); ═══►  public abstract void eat();    │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

---

## 5. Fields (Variables) in Interface

### Key Rules for Fields

| Rule | Description |
|------|-------------|
| **Implicitly public** | All fields are `public` |
| **Implicitly static** | All fields belong to interface, not instance |
| **Implicitly final** | All fields are constants (cannot be changed) |

### Field Declaration Examples

```java
public interface Bird {
    // Both are exactly the same - implicitly public static final
    int MAX_HEIGHT_IN_FEET = 2000;
    public static final int MAX_HEIGHT_IN_FEET = 2000;  // Explicit (redundant)
}
```

### Implicit Modifiers Visual

```
┌───────────────────────────────────────────────────────────┐
│                    INTERFACE FIELDS                        │
├───────────────────────────────────────────────────────────┤
│                                                           │
│   What you write:              What compiler sees:        │
│   ─────────────────            ─────────────────────      │
│                                                           │
│   int MAX_HEIGHT = 2000;  ═══► public static final        │
│                                int MAX_HEIGHT = 2000;     │
│                                                           │
│   ⚠️ These are CONSTANTS - cannot be changed!             │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### Usage

```java
public interface Bird {
    int MAX_ALTITUDE = 30000;  // Constant
}

// Accessing the constant
System.out.println(Bird.MAX_ALTITUDE);  // 30000

// ❌ Cannot modify
Bird.MAX_ALTITUDE = 40000;  // COMPILE ERROR! It's final
```

---

## 6. Interface Implementation

### Basic Implementation

```java
public interface Bird {
    void fly();
}

public class Eagle implements Bird {
    @Override
    public void fly() {
        System.out.println("Eagle is flying");
    }
}
```

### Implementation Rules

#### Rule 1: Cannot Use More Restrictive Access Modifier

```java
public interface Bird {
    void fly();  // implicitly public
}

public class Eagle implements Bird {
    // ❌ WRONG - cannot be protected (more restrictive than public)
    @Override
    protected void fly() { }
    
    // ✓ CORRECT - must be public
    @Override
    public void fly() { }
}
```

#### Rule 2: Concrete Class Must Override ALL Methods

```java
public interface Bird {
    void fly();
    void eat();
    void walk();
}

// Concrete class MUST implement ALL methods
public class Eagle implements Bird {
    @Override
    public void fly() { /* implementation */ }
    
    @Override
    public void eat() { /* implementation */ }
    
    @Override
    public void walk() { /* implementation */ }
    
    // If ANY method is missing → COMPILE ERROR!
}
```

#### Rule 3: Abstract Class Can Partially Implement

```java
public interface Bird {
    void canFly();
    int numberOfLegs();
}

// Abstract class - not forced to implement all methods
public abstract class Eagle implements Bird {
    
    // Implemented one method
    @Override
    public void canFly() {
        System.out.println("Yes, can fly");
    }
    
    // Did NOT implement numberOfLegs() - that's OK for abstract class
    
    // Can add its own abstract methods
    public abstract int beakLength();
}

// Concrete class extending abstract class
public class WhiteEagle extends Eagle {
    
    // Must implement remaining abstract methods from interface
    @Override
    public int numberOfLegs() {
        return 2;
    }
    
    // Must implement abstract methods from parent abstract class
    @Override
    public int beakLength() {
        return 5;
    }
}
```

### Implementation Hierarchy Visual

```
┌─────────────────────────────────────────────────────────────┐
│                    Bird (Interface)                          │
│              ─────────────────────────                       │
│                   void canFly();                             │
│                   int numberOfLegs();                        │
└───────────────────────────┬─────────────────────────────────┘
                            │ implements
                            ▼
┌─────────────────────────────────────────────────────────────┐
│                Eagle (Abstract Class)                        │
│              ─────────────────────────                       │
│    ✓ canFly() { implemented }                               │
│    ✗ numberOfLegs() - NOT implemented (OK for abstract)     │
│    + abstract beakLength(); - new abstract method           │
└───────────────────────────┬─────────────────────────────────┘
                            │ extends
                            ▼
┌─────────────────────────────────────────────────────────────┐
│              WhiteEagle (Concrete Class)                     │
│              ─────────────────────────                       │
│    ✓ numberOfLegs() { return 2; }  - from interface         │
│    ✓ beakLength() { return 5; }    - from abstract class    │
│                                                              │
│    (Concrete class MUST implement ALL remaining abstract     │
│     methods from both interface and abstract parent)         │
└─────────────────────────────────────────────────────────────┘
```

#### Rule 4: Class Can Implement Multiple Interfaces

```java
interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

interface Walkable {
    void walk();
}

// Multiple interfaces implementation
public class Duck implements Flyable, Swimmable, Walkable {
    @Override
    public void fly() { System.out.println("Duck flying"); }
    
    @Override
    public void swim() { System.out.println("Duck swimming"); }
    
    @Override
    public void walk() { System.out.println("Duck walking"); }
}
```

---

## 7. Nested Interfaces

### Definition

A **Nested Interface** is an interface declared within another interface or class.

### Purpose

Used to **logically group related interfaces**.

---

### 7.1 Interface Within Interface

```java
public interface Bird {
    // Abstract method of outer interface
    boolean canFly();
    
    // Nested interface (MUST be public)
    public interface NonFlyingBird {
        boolean canRun();
    }
}
```

### Rules for Nested Interface in Interface

| Rule | Description |
|------|-------------|
| Must be `public` | Cannot be private, protected, or default |
| Accessed via outer interface | `OuterInterface.NestedInterface` |

### Usage Examples

#### Implementing Only Outer Interface

```java
public class Eagle implements Bird {
    @Override
    public boolean canFly() {
        return true;
    }
    // Don't need to implement NonFlyingBird methods
}
```

#### Implementing Only Nested Interface

```java
public class Penguin implements Bird.NonFlyingBird {
    @Override
    public boolean canRun() {
        return true;
    }
    // Don't need to implement Bird methods
}
```

#### Implementing Both Interfaces

```java
public class Ostrich implements Bird, Bird.NonFlyingBird {
    @Override
    public boolean canFly() {
        return false;
    }
    
    @Override
    public boolean canRun() {
        return true;
    }
}
```

### Using Nested Interface

```java
public static void main(String[] args) {
    // Reference type is nested interface
    Bird.NonFlyingBird penguin = new Penguin();
    penguin.canRun();
}
```

---

### 7.2 Interface Within Class

```java
public class Bird {
    // Nested interface inside a class
    // Can be private, protected, public, or default
    protected interface NonFlyingBird {
        boolean canRun();
    }
}
```

### Rules for Nested Interface in Class

| Rule | Description |
|------|-------------|
| Any access modifier | Can be private, protected, public, or default |
| Accessed via outer class | `OuterClass.NestedInterface` |

### Usage

```java
public class Ostrich implements Bird.NonFlyingBird {
    @Override
    public boolean canRun() {
        return true;
    }
}
```

### Nested Interface Summary

```
┌─────────────────────────────────────────────────────────────┐
│                   NESTED INTERFACES                          │
├───────────────────────────┬─────────────────────────────────┤
│  Inside Interface         │  Inside Class                   │
├───────────────────────────┼─────────────────────────────────┤
│  • Must be public         │  • Any modifier allowed         │
│  • Access: Interface.     │  • Access: Class.               │
│    NestedInterface        │    NestedInterface              │
│  • Not commonly used      │  • Not commonly used            │
└───────────────────────────┴─────────────────────────────────┘
```

> **Note:** Nested interfaces are rarely used in practice but may appear in interviews.

---

## 8. Interface vs Abstract Class

### Comparison Table

| # | Aspect | Abstract Class | Interface |
|---|--------|----------------|-----------|
| 1 | **Keyword** | `abstract` | `interface` |
| 2 | **Inheritance keyword** | `extends` | `implements` |
| 3 | **Method types** | Both abstract and concrete | Only abstract (before Java 8) |
| 4 | **Extends/Implements** | Extends ONE class + Multiple interfaces | Extends multiple interfaces ONLY |
| 5 | **Variables** | Any: static, non-static, final, non-final | Only `public static final` (constants) |
| 6 | **Access modifiers** | private, protected, public, default | Only `public` (Java 9: private allowed) |
| 7 | **Multiple inheritance** | ❌ Not supported | ✓ Supported |
| 8 | **Can implement interface?** | ✓ Yes | ❌ No (can only extend interface) |
| 9 | **Constructor** | ✓ Can have | ❌ Cannot have |
| 10 | **Abstract method declaration** | Requires `abstract` keyword | No keyword needed (implicit) |

### Visual Comparison

```
┌─────────────────────────────────┬─────────────────────────────────┐
│        ABSTRACT CLASS           │           INTERFACE             │
├─────────────────────────────────┼─────────────────────────────────┤
│                                 │                                 │
│  abstract class Animal {        │  interface Animal {             │
│                                 │                                 │
│    // Variables - any type      │    // Variables - constants only│
│    private int age;             │    int MAX_AGE = 100;           │
│    public static String type;   │    // (public static final)    │
│    protected final int legs;    │                                 │
│                                 │                                 │
│    // Constructor allowed       │    // NO constructor            │
│    public Animal() { }          │                                 │
│                                 │                                 │
│    // Concrete method           │    // Only signatures           │
│    public void sleep() {        │    void sleep();                │
│      System.out.println("Zzz"); │                                 │
│    }                            │                                 │
│                                 │                                 │
│    // Abstract method           │    void eat();                  │
│    public abstract void eat();  │    // (implicitly abstract)     │
│                                 │                                 │
│  }                              │  }                              │
│                                 │                                 │
│  // Inheritance                 │  // Inheritance                 │
│  class Dog extends Animal { }   │  class Dog implements Animal { }│
│                                 │                                 │
│  // Multiple inheritance: NO    │  // Multiple inheritance: YES   │
│  class X extends A, B { } ❌    │  class X implements A, B { } ✓  │
│                                 │                                 │
└─────────────────────────────────┴─────────────────────────────────┘
```

### When to Use What?

| Use Abstract Class When... | Use Interface When... |
|---------------------------|----------------------|
| You want to share code among related classes | You want unrelated classes to implement |
| You need non-public members | You want to specify behavior for data types |
| You want to declare non-static/non-final fields | You want multiple inheritance |
| You have a clear "is-a" relationship | You want to define a contract |

---

## 9. Quick Reference Summary

### Interface Syntax Cheat Sheet

```java
// Interface declaration
public interface InterfaceName extends Parent1, Parent2 {
    
    // Constants (implicitly public static final)
    int CONSTANT = 100;
    
    // Abstract methods (implicitly public abstract)
    void method1();
    ReturnType method2(Parameters);
    
    // Nested interface (must be public in interface)
    interface NestedInterface {
        void nestedMethod();
    }
}

// Implementation
public class ClassName implements Interface1, Interface2 {
    // Must implement ALL abstract methods
    @Override
    public void method1() { /* implementation */ }
}
```

### Key Points to Remember

```
┌─────────────────────────────────────────────────────────────┐
│                 INTERFACE KEY POINTS                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ✓ All methods are public abstract (before Java 8)         │
│  ✓ All fields are public static final (constants)          │
│  ✓ Cannot instantiate interface (no new Interface())       │
│  ✓ Can use interface as data type (reference variable)     │
│  ✓ Enables multiple inheritance                            │
│  ✓ Achieves 100% abstraction (before Java 8)               │
│  ✓ Can extend multiple interfaces                          │
│  ✗ Cannot extend classes                                   │
│  ✗ Cannot have constructors                                │
│  ✗ Methods cannot be final                                 │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 10. Interview Tips

1. **"What is an interface?"**
    - Contract defining what to do, not how
    - Achieves abstraction and polymorphism
    - Enables multiple inheritance

2. **"Why can't we have multiple inheritance with classes?"**
    - Diamond problem causes ambiguity
    - Compiler can't decide which method to call

3. **"Why is multiple inheritance allowed with interfaces?"**
    - Interfaces only have signatures (no implementation)
    - Implementing class provides single implementation
    - No ambiguity

4. **"Can interface have variables?"**
    - Yes, but they're constants (`public static final`)
    - Must be initialized at declaration

5. **"Why can't interface methods be final?"**
    - `final` prevents overriding
    - Interface methods MUST be overridden
    - Contradiction!

6. **"What changes came in Java 8 for interfaces?"**
    - `default` methods (with implementation)
    - `static` methods
    - Will be covered in Part 2