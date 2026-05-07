# Comprehensive Notes: Functional Interface & Lambda Expression (Java 8)

## Overview - Topics Covered

| Topic | Description |
|-------|-------------|
| Functional Interface | Interface with exactly one abstract method |
| Lambda Expression | Concise way to implement functional interface |
| Built-in Functional Interfaces | Consumer, Supplier, Function, Predicate |
| Interface Extension Rules | How functional interfaces can extend others |

---

## 1. What is Functional Interface?

### Definition

A **Functional Interface** is an interface that contains **exactly ONE abstract method**.

Also known as: **SAM Interface** (Single Abstract Method)

### Key Points

| Rule | Description |
|------|-------------|
| **One abstract method** | Must have exactly ONE abstract method |
| **Multiple other methods allowed** | Can have default, static, private methods |
| **Object class methods allowed** | Methods from Object class don't count |
| **@FunctionalInterface optional** | Annotation is optional but recommended |

---

### Basic Example

```java
// Without annotation - still a functional interface
interface Bird {
    void canFly();  // Only ONE abstract method
}

// With annotation - recommended
@FunctionalInterface
interface Bird {
    void canFly();  // Only ONE abstract method
}
```

### Both Are Equivalent

```
┌─────────────────────────────┐      ┌─────────────────────────────┐
│  interface Bird {           │  ══  │  @FunctionalInterface       │
│      void canFly();         │      │  interface Bird {           │
│  }                          │      │      void canFly();         │
│                             │      │  }                          │
└─────────────────────────────┘      └─────────────────────────────┘
         │                                      │
         └──────────────┬───────────────────────┘
                        │
                        ▼
            Both are Functional Interfaces
            (Only ONE abstract method)
```

---

### Purpose of @FunctionalInterface Annotation

| With Annotation | Without Annotation |
|-----------------|-------------------|
| Compiler enforces single abstract method rule | No compile-time enforcement |
| Adding second abstract method → **COMPILE ERROR** | Adding second abstract method → Just becomes normal interface |
| Documents intent clearly | Intent not clear |

### Example: Annotation Prevents Mistakes

```java
@FunctionalInterface
interface Bird {
    void canFly();
    
    void getHeight();  // ❌ COMPILE ERROR!
    // "Invalid '@FunctionalInterface' annotation; 
    // Bird is not a functional interface"
}
```

```java
// Without annotation - no error, but no longer functional interface
interface Bird {
    void canFly();
    void getHeight();  // ✓ No error, but now has 2 abstract methods
}
```

---

### What's Allowed in Functional Interface

```java
@FunctionalInterface
interface Bird {
    
    // ═══════════════════════════════════════════════════════════
    // ONE abstract method (REQUIRED)
    // ═══════════════════════════════════════════════════════════
    void canFly();  // This is the single abstract method
    
    // ═══════════════════════════════════════════════════════════
    // Default methods (ALLOWED - any number)
    // ═══════════════════════════════════════════════════════════
    default void eat() {
        System.out.println("Bird eating");
    }
    
    default void sleep() {
        System.out.println("Bird sleeping");
    }
    
    // ═══════════════════════════════════════════════════════════
    // Static methods (ALLOWED - any number)
    // ═══════════════════════════════════════════════════════════
    static void describe() {
        System.out.println("Birds can fly");
    }
    
    // ═══════════════════════════════════════════════════════════
    // Object class methods (ALLOWED - don't count as abstract)
    // ═══════════════════════════════════════════════════════════
    String toString();     // From Object class - doesn't count
    boolean equals(Object o);  // From Object class - doesn't count
    int hashCode();        // From Object class - doesn't count
}
```

### Why Object Class Methods Don't Count

```java
interface TestInterface {
    String toString();  // Declared but...
}

class TestClass implements TestInterface {
    // No need to implement toString()!
    // Why? Every class extends Object, which already has toString()
}
```

```
┌─────────────────────────────────────────────────────────────────┐
│                         Object                                   │
│  ─────────────────────────────────────────────────────────────  │
│  • toString()                                                   │
│  • equals()                                                     │
│  • hashCode()                                                   │
│  • ... other methods                                            │
└─────────────────────────────────────────────────────────────────┘
                              ▲
                              │ extends (implicit)
                              │
┌─────────────────────────────────────────────────────────────────┐
│                      TestClass                                   │
│  ─────────────────────────────────────────────────────────────  │
│  implements TestInterface                                       │
│  (Already has toString() from Object - no override needed)      │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Three Ways to Implement Functional Interface

### Visual Comparison

```
┌─────────────────────────────────────────────────────────────────┐
│            THREE WAYS TO IMPLEMENT FUNCTIONAL INTERFACE          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Way 1: Traditional (implements keyword)                        │
│  ────────────────────────────────────────                       │
│  • Create a class that implements interface                     │
│  • Most verbose                                                 │
│                                                                 │
│  Way 2: Anonymous Class                                         │
│  ────────────────────────                                       │
│  • Create class without name inline                             │
│  • Moderate verbosity                                           │
│                                                                 │
│  Way 3: Lambda Expression (Java 8)                              │
│  ──────────────────────────────────                             │
│  • Most concise syntax                                          │
│  • Works ONLY with functional interfaces                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### Way 1: Using `implements` Keyword

```java
@FunctionalInterface
interface Bird {
    void canFly(String destination);
}

// Create implementing class
class Eagle implements Bird {
    @Override
    public void canFly(String destination) {
        System.out.println("Eagle flying to " + destination);
    }
}

// Usage
public static void main(String[] args) {
    Bird eagle = new Eagle();
    eagle.canFly("mountain");  // Output: Eagle flying to mountain
}
```

---

### Way 2: Using Anonymous Class

```java
@FunctionalInterface
interface Bird {
    void canFly(String destination);
}

public static void main(String[] args) {
    // Anonymous class implementation
    Bird eagle = new Bird() {
        @Override
        public void canFly(String destination) {
            System.out.println("Eagle flying to " + destination);
        }
    };  // Note the semicolon!
    
    eagle.canFly("mountain");
}
```

### Anonymous Class Breakdown

```
Bird eagle = new Bird() {
─────────────────────────
     │          │    │
     │          │    └─ Start of anonymous class body
     │          └─ Interface being implemented
     └─ Reference variable

    @Override
    public void canFly(String destination) {
        System.out.println("Eagle flying to " + destination);
    }
    ───────────────────────────────────────────────────────
                        │
                        └─ Implementation of abstract method

};
──
 └─ Semicolon required (it's a statement)
```

---

### Way 3: Using Lambda Expression (Java 8)

```java
@FunctionalInterface
interface Bird {
    void canFly(String destination);
}

public static void main(String[] args) {
    // Lambda expression implementation
    Bird eagle = (destination) -> {
        System.out.println("Eagle flying to " + destination);
    };
    
    eagle.canFly("mountain");
}
```

### Comparison: Anonymous vs Lambda

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANONYMOUS CLASS                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Bird eagle = new Bird() {                    ← Boilerplate    │
│       @Override                                ← Boilerplate    │
│       public void canFly(String destination) { ← Boilerplate    │
│           System.out.println("Flying to " + destination);       │
│       }                                                         │
│   };                                                            │
│                                                                 │
│   Lines of code: 6                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘

                              ▼ Simplified to

┌─────────────────────────────────────────────────────────────────┐
│                    LAMBDA EXPRESSION                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Bird eagle = (destination) -> {                               │
│       System.out.println("Flying to " + destination);           │
│   };                                                            │
│                                                                 │
│   Lines of code: 3                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Lambda Expression Syntax

### Why Lambda Works Only with Functional Interface

```
┌─────────────────────────────────────────────────────────────────┐
│                  WHY ONLY FUNCTIONAL INTERFACE?                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Functional Interface → Only ONE abstract method               │
│                                                                 │
│   Lambda Expression → Implements that ONE method                │
│                                                                 │
│   Since there's only ONE method:                                │
│   • No need to specify method name                              │
│   • No need for @Override annotation                            │
│   • No need for access modifier                                 │
│   • Compiler knows exactly which method you're implementing     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Lambda Syntax Breakdown

```
(parameters) -> { body }
─────────────    ──────
      │             │
      │             └─ Implementation (what the method does)
      └─ Input parameters (what the method receives)

           -> 
           ──
            │
            └─ Arrow operator (also called lambda operator)
```

### Syntax Variations

#### 1. No Parameters, Multiple Statements

```java
@FunctionalInterface
interface Greeting {
    void sayHello();
}

// Lambda with no parameters
Greeting greet = () -> {
    System.out.println("Hello");
    System.out.println("Welcome!");
};
```

#### 2. No Parameters, Single Statement (No braces needed)

```java
Greeting greet = () -> System.out.println("Hello");
```

#### 3. One Parameter

```java
@FunctionalInterface
interface Printer {
    void print(String message);
}

// With parentheses
Printer p1 = (message) -> System.out.println(message);

// Without parentheses (allowed for single parameter)
Printer p2 = message -> System.out.println(message);
```

#### 4. Multiple Parameters

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

// Parentheses required for multiple parameters
Calculator calc = (a, b) -> {
    return a + b;
};

// Simplified (single expression, implicit return)
Calculator calc = (a, b) -> a + b;
```

#### 5. With Return Value

```java
@FunctionalInterface
interface StringLength {
    int getLength(String s);
}

// Full syntax
StringLength sl1 = (s) -> {
    return s.length();
};

// Simplified (implicit return for single expression)
StringLength sl2 = s -> s.length();
```

### Lambda Syntax Rules Summary

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAMBDA SYNTAX RULES                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  PARAMETERS:                                                    │
│  ───────────                                                    │
│  • () → No parameters                                           │
│  • (x) or x → One parameter (parentheses optional)              │
│  • (x, y) → Multiple parameters (parentheses required)          │
│  • Type declaration optional: (String s) or (s) both work       │
│                                                                 │
│  BODY:                                                          │
│  ─────                                                          │
│  • Single expression → No braces, no return keyword             │
│  • Multiple statements → Braces required, return if needed      │
│                                                                 │
│  EXAMPLES:                                                      │
│  ─────────                                                      │
│  () -> 42                     // Returns 42                     │
│  x -> x * 2                   // Returns x * 2                  │
│  (x, y) -> x + y              // Returns sum                    │
│  (s) -> { System.out.println(s); }  // No return               │
│  (a, b) -> { return a > b ? a : b; } // With return            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Built-in Functional Interfaces

Java provides four main functional interfaces in `java.util.function` package:

### Overview

| Interface | Input | Output | Abstract Method |
|-----------|-------|--------|-----------------|
| **Consumer<T>** | T | void | `accept(T t)` |
| **Supplier<T>** | None | T | `get()` |
| **Function<T,R>** | T | R | `apply(T t)` |
| **Predicate<T>** | T | boolean | `test(T t)` |

### Visual Summary

```
┌─────────────────────────────────────────────────────────────────┐
│               BUILT-IN FUNCTIONAL INTERFACES                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Consumer<T>         Supplier<T>                               │
│   ┌─────────┐         ┌─────────┐                               │
│   │    T    │───►     │         │───► T                         │
│   │ (input) │  void   │ (none)  │  (output)                     │
│   └─────────┘         └─────────┘                               │
│   "Consumes data"     "Supplies data"                           │
│                                                                 │
│   Function<T,R>       Predicate<T>                              │
│   ┌─────────┐         ┌─────────┐                               │
│   │    T    │───► R   │    T    │───► boolean                   │
│   │ (input) │(output) │ (input) │  (true/false)                 │
│   └─────────┘         └─────────┘                               │
│   "Transforms data"   "Tests condition"                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.1 Consumer<T>

**Definition:** Accepts a single input, returns nothing (void)

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

**Example:**

```java
import java.util.function.Consumer;

public class Main {
    public static void main(String[] args) {
        // Consumer that logs if value > 10
        Consumer<Integer> logger = (value) -> {
            if (value > 10) {
                System.out.println("Value " + value + " is greater than 10");
            } else {
                System.out.println("Value " + value + " is not greater than 10");
            }
        };
        
        // Using the consumer
        logger.accept(11);  // Output: Value 11 is greater than 10
        logger.accept(5);   // Output: Value 5 is not greater than 10
    }
}
```

**Use Cases:**
- Logging
- Printing
- Modifying objects
- Sending notifications

---

### 4.2 Supplier<T>

**Definition:** Accepts nothing, returns a result

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**Example:**

```java
import java.util.function.Supplier;

public class Main {
    public static void main(String[] args) {
        // Method 1: Simplified (single expression)
        Supplier<String> messageSupplier = () -> "Hello from Supplier!";
        
        // Method 2: With block (multiple statements)
        Supplier<String> greetingSupplier = () -> {
            String greeting = "Welcome";
            String name = "User";
            return greeting + " " + name + "!";
        };
        
        // Using the suppliers
        System.out.println(messageSupplier.get());    // Hello from Supplier!
        System.out.println(greetingSupplier.get());   // Welcome User!
    }
}
```

**Use Cases:**
- Lazy initialization
- Factory methods
- Generating random values
- Getting current timestamp

---

### 4.3 Function<T, R>

**Definition:** Accepts one argument, returns a result (different types possible)

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

**Example:**

```java
import java.util.function.Function;

public class Main {
    public static void main(String[] args) {
        // Function: Integer → String
        Function<Integer, String> intToString = (num) -> {
            return num.toString();
        };
        
        // Simplified version
        Function<Integer, String> intToString2 = num -> num.toString();
        
        // Function: String → Integer (length)
        Function<String, Integer> stringLength = str -> str.length();
        
        // Using the functions
        String result = intToString.apply(42);
        System.out.println(result);  // "42" (String)
        
        Integer length = stringLength.apply("Hello");
        System.out.println(length);  // 5
    }
}
```

**Use Cases:**
- Data transformation
- Type conversion
- Mapping operations
- Calculations

---

### 4.4 Predicate<T>

**Definition:** Accepts one argument, returns boolean

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

**Example:**

```java
import java.util.function.Predicate;

public class Main {
    public static void main(String[] args) {
        // Predicate to check if number is even
        Predicate<Integer> isEven = (num) -> {
            return num % 2 == 0;
        };
        
        // Simplified version
        Predicate<Integer> isEven2 = num -> num % 2 == 0;
        
        // Predicate to check if string is empty
        Predicate<String> isEmpty = str -> str.isEmpty();
        
        // Using the predicates
        System.out.println(isEven.test(4));    // true
        System.out.println(isEven.test(5));    // false
        System.out.println(isEmpty.test(""));  // true
        System.out.println(isEmpty.test("Hi")); // false
    }
}
```

**Use Cases:**
- Filtering collections
- Validation
- Conditional logic
- Search criteria

---

### Quick Reference Table

```
┌──────────────┬────────────────┬────────────┬────────────────────┐
│  Interface   │  Method        │  Signature │  Example Usage     │
├──────────────┼────────────────┼────────────┼────────────────────┤
│ Consumer<T>  │ accept(T t)    │ T → void   │ list.forEach()     │
├──────────────┼────────────────┼────────────┼────────────────────┤
│ Supplier<T>  │ get()          │ () → T     │ Optional.orElseGet │
├──────────────┼────────────────┼────────────┼────────────────────┤
│ Function<T,R>│ apply(T t)     │ T → R      │ stream.map()       │
├──────────────┼────────────────┼────────────┼────────────────────┤
│ Predicate<T> │ test(T t)      │ T → boolean│ stream.filter()    │
└──────────────┴────────────────┴────────────┴────────────────────┘
```

---

## 5. Functional Interface Extension Rules

### Use Case 1: Functional Interface Extends Non-Functional Interface

```java
// Non-functional interface (has default method)
interface LivingThing {
    default void canBreathe() {
        System.out.println("Breathing...");
    }
}

// ✓ VALID - Still only one abstract method
@FunctionalInterface
interface Bird extends LivingThing {
    void canFly();  // Only abstract method
}
```

```java
// Non-functional interface (has abstract method)
interface LivingThing {
    void canBreathe();  // Abstract method
}

// ❌ INVALID - Two abstract methods now
@FunctionalInterface
interface Bird extends LivingThing {
    void canFly();  // + canBreathe from parent = 2 abstract methods
}
// COMPILE ERROR: Bird is not a functional interface
```

### Visual Explanation

```
                    ┌───────────────────────┐
                    │    LivingThing        │
                    │  ─────────────────    │
                    │  void canBreathe();   │ ← Abstract method
                    └───────────┬───────────┘
                                │ extends
                                ▼
                    ┌───────────────────────┐
   @FunctionalInterface         │       Bird           │
                    │  ─────────────────    │
                    │  void canFly();       │ ← Abstract method
                    │  + canBreathe()       │ ← Inherited abstract
                    │  = 2 abstract methods │
                    └───────────────────────┘
                                │
                                ▼
                    ❌ COMPILE ERROR!
                    Not a valid functional interface
```

---

### Use Case 2: Normal Interface Extends Functional Interface

```java
@FunctionalInterface
interface LivingThing {
    void canBreathe();
}

// ✓ VALID - No @FunctionalInterface, so can have multiple abstract
interface Bird extends LivingThing {
    void canFly();  // Two abstract methods total - OK for normal interface
}
```

```
                    ┌───────────────────────┐
   @FunctionalInterface         │    LivingThing        │
                    │  ─────────────────    │
                    │  void canBreathe();   │
                    └───────────┬───────────┘
                                │ extends
                                ▼
                    ┌───────────────────────┐
                    │       Bird            │ ← No @FunctionalInterface
                    │  ─────────────────    │
                    │  void canBreathe();   │ ← Inherited
                    │  void canFly();       │ ← Own method
                    └───────────────────────┘
                                │
                                ▼
                    ✓ VALID (but not a functional interface)
```

---

### Use Case 3: Functional Interface Extends Functional Interface

#### Case 3a: Different Methods → ❌ INVALID

```java
@FunctionalInterface
interface LivingThing {
    void canBreathe();
}

// ❌ INVALID - Two different abstract methods
@FunctionalInterface
interface Bird extends LivingThing {
    void canFly();  // Different from canBreathe
}
// COMPILE ERROR!
```

#### Case 3b: Same Method → ✓ VALID

```java
@FunctionalInterface
interface LivingThing {
    boolean canBreathe();
}

// ✓ VALID - Same method, just overriding
@FunctionalInterface
interface Bird extends LivingThing {
    boolean canBreathe();  // Same method - overriding parent
}
```

```
            ┌───────────────────────┐
   @FunctionalInterface         │    LivingThing        │
            │  ─────────────────    │
            │  boolean canBreathe() │
            └───────────┬───────────┘
                        │ extends
                        ▼
            ┌───────────────────────┐
   @FunctionalInterface         │       Bird            │
            │  ─────────────────    │
            │  boolean canBreathe() │ ← Same method (override)
            └───────────────────────┘
                        │
                        ▼
            ✓ VALID - Still only one abstract method
```

### Implementing Case 3b with Lambda

```java
@FunctionalInterface
interface LivingThing {
    boolean canBreathe();
}

@FunctionalInterface
interface Bird extends LivingThing {
    boolean canBreathe();  // Overrides parent
}

public class Main {
    public static void main(String[] args) {
        // Lambda implementation
        Bird eagle = () -> true;
        
        // Equivalent to:
        Bird eagle2 = () -> {
            return true;
        };
        
        // Usage
        System.out.println(eagle.canBreathe());  // true
    }
}
```

---

## 6. Summary Table

### Extension Rules Summary

| Scenario | Parent Abstract Methods | Child Adds | Total | Valid? |
|----------|------------------------|------------|-------|--------|
| FI extends Non-FI (default methods) | 0 | 1 | 1 | ✓ |
| FI extends Non-FI (abstract method) | 1 | 1 | 2 | ❌ |
| Non-FI extends FI | 1 | 1+ | 2+ | ✓ (but not FI) |
| FI extends FI (same method) | 1 | 0 (override) | 1 | ✓ |
| FI extends FI (different method) | 1 | 1 | 2 | ❌ |

*FI = Functional Interface*

---

## 7. Interview Quick Reference

### Common Interview Questions

1. **"What is a functional interface?"**
    - Interface with exactly ONE abstract method
    - Can have default, static, private methods
    - Object class methods don't count

2. **"Why is @FunctionalInterface optional?"**
    - Interface is functional based on content, not annotation
    - Annotation provides compile-time checking
    - Prevents accidental addition of second abstract method

3. **"Why does Lambda work only with functional interfaces?"**
    - Lambda implements ONE method (no name needed)
    - Functional interface has exactly ONE abstract method
    - Compiler knows which method to implement

4. **"Difference between Consumer and Supplier?"**
    - Consumer: Takes input, no output (void)
    - Supplier: No input, produces output

5. **"When to use Function vs Predicate?"**
    - Function: Transform data (any return type)
    - Predicate: Test condition (returns boolean only)

### Key Points to Remember

```
┌─────────────────────────────────────────────────────────────────┐
│              FUNCTIONAL INTERFACE & LAMBDA KEY POINTS           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Functional Interface:                                          │
│  • Exactly ONE abstract method                                  │
│  • @FunctionalInterface is optional but recommended             │
│  • Can have any number of default/static/private methods        │
│  • Object class methods don't count                             │
│                                                                 │
│  Lambda Expression:                                             │
│  • Syntax: (params) -> { body }                                 │
│  • Concise way to implement functional interface                │
│  • Single expression: no braces, no return keyword              │
│  • Multiple statements: braces required, explicit return        │
│                                                                 │
│  Built-in Functional Interfaces:                                │
│  • Consumer<T>: T → void      (accept)                          │
│  • Supplier<T>: () → T        (get)                             │
│  • Function<T,R>: T → R       (apply)                           │
│  • Predicate<T>: T → boolean  (test)                            │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```