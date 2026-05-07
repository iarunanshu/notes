# Comprehensive Notes: Java Reflection API

## Overview - What We'll Cover

| Topic | Description |
|-------|-------------|
| What is Reflection | Examining classes at runtime |
| The `Class` class | Metadata holder for every class |
| Getting Class object | Three ways to obtain Class instance |
| Reflection of Class | Getting class metadata |
| Reflection of Methods | Accessing and invoking methods |
| Reflection of Fields | Accessing and modifying field values |
| Reflection of Constructors | Accessing constructors |
| Breaking Singleton | How reflection bypasses private constructors |
| Disadvantages | Why reflection should be used sparingly |

---

## 1. What is Reflection?

### Definition

**Reflection** is a feature in Java that allows you to **examine** and **modify** the behavior of classes, methods, fields, and interfaces **at runtime**.

### Capabilities of Reflection

```
┌─────────────────────────────────────────────────────────────────┐
│                  REFLECTION CAPABILITIES                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  EXAMINE (Read):                                                │
│  ───────────────                                                │
│  • What methods does a class have?                              │
│  • What fields does a class have?                               │
│  • What constructors does a class have?                         │
│  • What is the return type of a method?                         │
│  • What are the parameters of a method?                         │
│  • What modifiers (public/private) are used?                    │
│  • What interfaces does a class implement?                      │
│                                                                 │
│  MODIFY (Write):                                                │
│  ────────────────                                               │
│  • Change value of public fields                                │
│  • Change value of PRIVATE fields (breaks encapsulation!)       │
│  • Invoke methods (including private methods)                   │
│  • Create objects using private constructors                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Visual Overview

```
┌──────────────────────┐
│     Your Class       │
│  ──────────────────  │
│  - private field1    │
│  - public field2     │
│  + method1()         │
│  - privateMethod()   │
│  - Constructor()     │
└──────────┬───────────┘
           │
           ▼ Reflection can access ALL of this
┌──────────────────────────────────────────────────────────────┐
│                    REFLECTION API                             │
│  ──────────────────────────────────────────────────────────  │
│                                                              │
│  • Get all field names and types                             │
│  • Get all method signatures                                 │
│  • Get all constructors                                      │
│  • Read/Write field values (even private!)                   │
│  • Invoke methods (even private!)                            │
│  • Create objects (even with private constructor!)           │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

---

## 2. The `Class` Class

### What is the `Class` class?

The `Class` class (yes, a class named "Class") holds **metadata information** about every class loaded by JVM.

### Key Points

| Point | Description |
|-------|-------------|
| **One per class** | JVM creates one `Class` object for each class loaded |
| **Created by JVM** | You don't create it; JVM creates it automatically |
| **Contains metadata** | Stores information about methods, fields, constructors, etc. |
| **Package** | Located in `java.lang.Class` |

### How JVM Creates Class Objects

```
┌─────────────────────────────────────────────────────────────────┐
│                    JVM CLASS LOADING                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Your Code:                                                    │
│   ──────────                                                    │
│   class Bird { ... }                                            │
│   class Animal { ... }                                          │
│   class Eagle { ... }                                           │
│                                                                 │
│   When JVM loads each class:                                    │
│   ──────────────────────────                                    │
│                                                                 │
│   Bird ────────────► JVM creates Class object for Bird          │
│                      (contains Bird's metadata)                 │
│                                                                 │
│   Animal ──────────► JVM creates Class object for Animal        │
│                      (contains Animal's metadata)               │
│                                                                 │
│   Eagle ───────────► JVM creates Class object for Eagle         │
│                      (contains Eagle's metadata)                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### What Metadata Does Class Object Contain?

```java
Class<?> eagleClass = Eagle.class;

// Class object exposes methods to get metadata:
eagleClass.getName();           // Class name
eagleClass.getModifiers();      // public, private, etc.
eagleClass.getMethods();        // All public methods
eagleClass.getDeclaredMethods(); // All methods (public + private)
eagleClass.getFields();         // All public fields
eagleClass.getDeclaredFields(); // All fields (public + private)
eagleClass.getConstructors();   // All public constructors
eagleClass.getDeclaredConstructors(); // All constructors
eagleClass.getInterfaces();     // Implemented interfaces
eagleClass.getSuperclass();     // Parent class
```

---

## 3. Three Ways to Get Class Object

### Overview

```
┌─────────────────────────────────────────────────────────────────┐
│              THREE WAYS TO GET CLASS OBJECT                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Method 1: Class.forName("className")                           │
│  ─────────────────────────────────────                          │
│  • Uses fully qualified class name as String                    │
│  • Throws ClassNotFoundException if class not found             │
│                                                                 │
│  Method 2: ClassName.class                                      │
│  ─────────────────────────                                      │
│  • Uses .class property                                         │
│  • Compile-time safe                                            │
│                                                                 │
│  Method 3: object.getClass()                                    │
│  ───────────────────────────                                    │
│  • Uses existing object instance                                │
│  • Inherited from Object class                                  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Example Class

```java
public class Bird {
    public String breed;
    private boolean canSwim;
    
    public void fly() {
        System.out.println("Flying...");
    }
}
```

### Method 1: Using `Class.forName()`

```java
// Pass fully qualified class name as String
Class<?> birdClass = Class.forName("com.example.Bird");

// Note: Throws ClassNotFoundException if class doesn't exist
```

### Method 2: Using `.class`

```java
// Use .class property on the class name
Class<?> birdClass = Bird.class;

// Compile-time safe - error if class doesn't exist
```

### Method 3: Using `getClass()`

```java
// First create an object
Bird bird = new Bird();

// Then get its Class object
Class<?> birdClass = bird.getClass();
```

### Comparison

| Method | Syntax | When to Use |
|--------|--------|-------------|
| `Class.forName()` | `Class.forName("Bird")` | Class name known only at runtime (String) |
| `.class` | `Bird.class` | Class name known at compile time |
| `getClass()` | `obj.getClass()` | When you have an object instance |

---

## 4. Reflection of Class

### Example Class

```java
public class Eagle {
    public String breed;
    private boolean canSwim;
    
    public void fly() {
        System.out.println("Flying...");
    }
    
    private void eat() {
        System.out.println("Eating...");
    }
}
```

### Getting Class Metadata

```java
public class Main {
    public static void main(String[] args) {
        // Step 1: Get the Class object
        Class<?> eagleClass = Eagle.class;
        
        // Step 2: Get metadata using Class methods
        
        // Get class name
        String name = eagleClass.getName();
        System.out.println("Class Name: " + name);  // Eagle
        
        // Get modifiers (public, private, etc.)
        int modifiers = eagleClass.getModifiers();
        System.out.println("Modifiers: " + Modifier.toString(modifiers));  // public
        
        // Get superclass
        Class<?> superClass = eagleClass.getSuperclass();
        System.out.println("Superclass: " + superClass.getName());  // java.lang.Object
        
        // Get interfaces
        Class<?>[] interfaces = eagleClass.getInterfaces();
        for (Class<?> iface : interfaces) {
            System.out.println("Interface: " + iface.getName());
        }
    }
}
```

### Available Methods in Class Object

```
┌─────────────────────────────────────────────────────────────────┐
│              COMMONLY USED CLASS METHODS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Information Methods:                                           │
│  ────────────────────                                           │
│  getName()           → Full class name                          │
│  getSimpleName()     → Simple class name (without package)      │
│  getModifiers()      → Access modifiers                         │
│  getSuperclass()     → Parent class                             │
│  getInterfaces()     → Implemented interfaces                   │
│  getPackage()        → Package information                      │
│                                                                 │
│  Member Access Methods:                                         │
│  ─────────────────────                                          │
│  getMethods()            → All PUBLIC methods (including inherited) │
│  getDeclaredMethods()    → All methods of THIS class only       │
│  getFields()             → All PUBLIC fields (including inherited)  │
│  getDeclaredFields()     → All fields of THIS class only        │
│  getConstructors()       → All PUBLIC constructors              │
│  getDeclaredConstructors() → All constructors                   │
│                                                                 │
│  Object Creation:                                               │
│  ────────────────                                               │
│  newInstance()       → Create object (calls no-arg constructor) │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Reflection of Methods

### Get Methods vs Get Declared Methods

| Method | Returns | Includes Inherited |
|--------|---------|-------------------|
| `getMethods()` | Only PUBLIC methods | Yes (from parent classes) |
| `getDeclaredMethods()` | ALL methods (public + private) | No (only this class) |

### Example: Getting All Methods

```java
public class Eagle {
    public void fly() {
        System.out.println("Flying...");
    }
    
    private void eat() {
        System.out.println("Eating...");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Class<?> eagleClass = Eagle.class;
        
        // Get only PUBLIC methods (includes inherited from Object)
        System.out.println("=== getMethods() ===");
        Method[] publicMethods = eagleClass.getMethods();
        for (Method method : publicMethods) {
            System.out.println("Method: " + method.getName());
            System.out.println("  Return Type: " + method.getReturnType());
            System.out.println("  Declared in: " + method.getDeclaringClass());
        }
        
        // Output includes: fly, wait, equals, toString, hashCode, etc.
        // (all public methods from Eagle AND Object class)
        
        // Get ALL methods of Eagle class only
        System.out.println("\n=== getDeclaredMethods() ===");
        Method[] allMethods = eagleClass.getDeclaredMethods();
        for (Method method : allMethods) {
            System.out.println("Method: " + method.getName());
        }
        
        // Output: fly, eat (only Eagle's methods, including private)
    }
}
```

### Output Visualization

```
getMethods():
┌────────────────────────────────────────────────────────┐
│  fly()      - from Eagle (public)                      │
│  wait()     - from Object (public)                     │
│  equals()   - from Object (public)                     │
│  toString() - from Object (public)                     │
│  hashCode() - from Object (public)                     │
│  notify()   - from Object (public)                     │
│  ...                                                   │
└────────────────────────────────────────────────────────┘

getDeclaredMethods():
┌────────────────────────────────────────────────────────┐
│  fly()  - public                                       │
│  eat()  - private                                      │
│  (Only Eagle's methods, not inherited ones)            │
└────────────────────────────────────────────────────────┘
```

### Method Metadata

```java
Method method = eagleClass.getDeclaredMethod("fly");

// Get method information
method.getName();              // Method name
method.getReturnType();        // Return type
method.getParameterTypes();    // Parameter types array
method.getParameterCount();    // Number of parameters
method.getModifiers();         // public, private, static, etc.
method.getDeclaringClass();    // Class where method is declared
method.getExceptionTypes();    // Checked exceptions thrown
```

---

## 6. Invoking Methods Using Reflection

### Example Class

```java
public class Eagle {
    public void fly(int height, boolean fast, String destination) {
        System.out.println("Flying to " + destination + 
                           " at height " + height + 
                           ", fast: " + fast);
    }
}
```

### Invoking Method via Reflection

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // Step 1: Get the Class object
        Class<?> eagleClass = Class.forName("Eagle");
        
        // Step 2: Create an instance of the class
        Object eagleObject = eagleClass.newInstance();
        // Or: Object eagleObject = eagleClass.getDeclaredConstructor().newInstance();
        
        // Step 3: Get the method with matching name and parameter types
        Method flyMethod = eagleClass.getMethod(
            "fly",                    // Method name
            int.class,                // First parameter type
            boolean.class,            // Second parameter type
            String.class              // Third parameter type
        );
        
        // Step 4: Invoke the method
        flyMethod.invoke(
            eagleObject,              // Object to invoke method on
            100,                      // int height
            true,                     // boolean fast
            "Mountain"                // String destination
        );
        
        // Output: Flying to Mountain at height 100, fast: true
    }
}
```

### Step-by-Step Visualization

```
┌─────────────────────────────────────────────────────────────────┐
│                  INVOKING METHOD VIA REFLECTION                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Step 1: Get Class object                                       │
│  ────────────────────────                                       │
│  Class<?> eagleClass = Class.forName("Eagle");                  │
│                                                                 │
│  Step 2: Create object instance                                 │
│  ──────────────────────────────                                 │
│  Object obj = eagleClass.newInstance();                         │
│                     │                                           │
│                     └─► Internally calls default constructor    │
│                                                                 │
│  Step 3: Get Method object                                      │
│  ─────────────────────────                                      │
│  Method m = eagleClass.getMethod("fly",                         │
│                                  int.class,                     │
│                                  boolean.class,                 │
│                                  String.class);                 │
│                     │                                           │
│                     └─► Finds method matching name & params     │
│                                                                 │
│  Step 4: Invoke the method                                      │
│  ─────────────────────────                                      │
│  m.invoke(obj, 100, true, "Mountain");                          │
│      │     │    │     │        │                                │
│      │     │    └─────┴────────┴─► Arguments                    │
│      │     └─► Object to call method on                         │
│      └─► The Method object                                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 7. Reflection of Fields

### Get Fields vs Get Declared Fields

| Method | Returns | Includes Inherited |
|--------|---------|-------------------|
| `getFields()` | Only PUBLIC fields | Yes |
| `getDeclaredFields()` | ALL fields (public + private) | No |

### Example: Getting All Fields

```java
public class Eagle {
    public String breed;
    private boolean canSwim;
}
```

```java
public class Main {
    public static void main(String[] args) {
        Class<?> eagleClass = Eagle.class;
        
        // Get only PUBLIC fields
        System.out.println("=== getFields() ===");
        Field[] publicFields = eagleClass.getFields();
        for (Field field : publicFields) {
            System.out.println("Field: " + field.getName());
            System.out.println("  Type: " + field.getType());
            System.out.println("  Modifier: " + Modifier.toString(field.getModifiers()));
        }
        // Output: breed (String, public)
        
        // Get ALL fields
        System.out.println("\n=== getDeclaredFields() ===");
        Field[] allFields = eagleClass.getDeclaredFields();
        for (Field field : allFields) {
            System.out.println("Field: " + field.getName());
            System.out.println("  Type: " + field.getType());
            System.out.println("  Modifier: " + Modifier.toString(field.getModifiers()));
        }
        // Output: breed (String, public), canSwim (boolean, private)
    }
}
```

### Field Metadata Methods

```java
Field field = eagleClass.getDeclaredField("breed");

field.getName();           // Field name
field.getType();           // Field type (String.class, int.class, etc.)
field.getModifiers();      // public, private, static, final, etc.
field.getDeclaringClass(); // Class where field is declared
```

---

## 8. Setting Field Values Using Reflection

### Setting PUBLIC Field Value

```java
public class Eagle {
    public String breed;
}
```

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // Step 1: Get Class object
        Class<?> eagleClass = Eagle.class;
        
        // Step 2: Create an instance
        Eagle eagle = (Eagle) eagleClass.newInstance();
        
        // Step 3: Get the field
        Field breedField = eagleClass.getDeclaredField("breed");
        
        // Step 4: Set the value
        breedField.set(eagle, "Golden Eagle");
        
        // Verify
        System.out.println(eagle.breed);  // Output: Golden Eagle
    }
}
```

### Setting PRIVATE Field Value

```java
public class Eagle {
    private boolean canSwim;
    
    public boolean getCanSwim() {
        return canSwim;
    }
}
```

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Class<?> eagleClass = Eagle.class;
        Eagle eagle = (Eagle) eagleClass.newInstance();
        
        // Get private field
        Field canSwimField = eagleClass.getDeclaredField("canSwim");
        
        // Without setAccessible(true), this throws IllegalAccessException:
        // canSwimField.set(eagle, true);  // ❌ ERROR!
        
        // IMPORTANT: Make private field accessible
        canSwimField.setAccessible(true);  // ✓ Bypass access control
        
        // Now we can set the value
        canSwimField.set(eagle, true);
        
        // Verify
        System.out.println(eagle.getCanSwim());  // Output: true
    }
}
```

### The `setAccessible(true)` Hack

```
┌─────────────────────────────────────────────────────────────────┐
│              ACCESSING PRIVATE MEMBERS                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Normal Access:                                                 │
│  ──────────────                                                 │
│  private boolean canSwim;                                       │
│                                                                 │
│  eagle.canSwim = true;  ❌ Compile error (private!)             │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Reflection Access (without setAccessible):                     │
│  ──────────────────────────────────────────                     │
│  Field f = eagleClass.getDeclaredField("canSwim");              │
│  f.set(eagle, true);  ❌ IllegalAccessException at runtime      │
│                                                                 │
│  ─────────────────────────────────────────────────────────────  │
│                                                                 │
│  Reflection Access (with setAccessible):                        │
│  ─────────────────────────────────────────                      │
│  Field f = eagleClass.getDeclaredField("canSwim");              │
│  f.setAccessible(true);  ← Bypass Java access control!          │
│  f.set(eagle, true);  ✓ Works! (Breaks encapsulation!)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Reflection of Constructors

### Example: Accessing Private Constructor

```java
public class Eagle {
    private Eagle() {  // Private constructor!
        System.out.println("Eagle created!");
    }
    
    public void fly() {
        System.out.println("Flying...");
    }
}
```

### Bypassing Private Constructor

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Class<?> eagleClass = Eagle.class;
        
        // Get all constructors (including private)
        Constructor<?>[] constructors = eagleClass.getDeclaredConstructors();
        
        for (Constructor<?> constructor : constructors) {
            System.out.println("Modifier: " + 
                Modifier.toString(constructor.getModifiers()));  // private
            
            // Make private constructor accessible
            constructor.setAccessible(true);
            
            // Create object using private constructor!
            Object eagle = constructor.newInstance();
            
            // Now we can use the object
            Method flyMethod = eagleClass.getMethod("fly");
            flyMethod.invoke(eagle);  // Output: Flying...
        }
    }
}
```

---

## 10. How Reflection Breaks Singleton

### Singleton Pattern (Normal)

```java
public class Singleton {
    private static Singleton instance;
    
    // Private constructor - prevents external instantiation
    private Singleton() {
        System.out.println("Singleton instance created");
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
    
    public void doSomething() {
        System.out.println("Doing something...");
    }
}
```

### Normal Usage (Singleton Works)

```java
Singleton s1 = Singleton.getInstance();
Singleton s2 = Singleton.getInstance();

System.out.println(s1 == s2);  // true (same instance)
```

### Breaking Singleton with Reflection

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // Get instance normally
        Singleton s1 = Singleton.getInstance();
        
        // Now break singleton using reflection!
        Class<?> singletonClass = Singleton.class;
        
        // Get the private constructor
        Constructor<?> constructor = singletonClass.getDeclaredConstructor();
        
        // Make it accessible (bypass private!)
        constructor.setAccessible(true);
        
        // Create a NEW instance!
        Singleton s2 = (Singleton) constructor.newInstance();
        
        // Check if they're the same
        System.out.println(s1 == s2);  // false! Different instances!
        
        // Singleton is broken!
    }
}
```

### Visual Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                HOW REFLECTION BREAKS SINGLETON                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Normal Singleton:                                              │
│  ─────────────────                                              │
│                                                                 │
│  getInstance() ───► Creates instance (first time)              │
│  getInstance() ───► Returns SAME instance                       │
│  getInstance() ───► Returns SAME instance                       │
│                                                                 │
│  new Singleton() ──► ❌ Compile Error (private constructor)     │
│                                                                 │
│  ═══════════════════════════════════════════════════════════    │
│                                                                 │
│  With Reflection:                                               │
│  ────────────────                                               │
│                                                                 │
│  Constructor<?> c = class.getDeclaredConstructor();             │
│  c.setAccessible(true);  ← Bypass private!                      │
│  c.newInstance();  ───► ✓ Creates NEW instance!                 │
│  c.newInstance();  ───► ✓ Creates ANOTHER new instance!         │
│                                                                 │
│  Result: Multiple instances exist - SINGLETON BROKEN!           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Protecting Singleton from Reflection

```java
public class Singleton {
    private static Singleton instance;
    
    private Singleton() {
        // Protection against reflection!
        if (instance != null) {
            throw new RuntimeException(
                "Use getInstance() method to get singleton instance"
            );
        }
        System.out.println("Singleton instance created");
    }
    
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```

---

## 11. Disadvantages of Reflection

### Why Reflection Should Be Used Sparingly

```
┌─────────────────────────────────────────────────────────────────┐
│                DISADVANTAGES OF REFLECTION                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. BREAKS ENCAPSULATION                                        │
│  ───────────────────────                                        │
│  • Can access private fields and methods                        │
│  • Defeats the purpose of access modifiers                      │
│  • Can modify values that should be protected                   │
│                                                                 │
│  2. PERFORMANCE OVERHEAD                                        │
│  ───────────────────────                                        │
│  • Slower than direct access                                    │
│  • Runtime resolution instead of compile-time                   │
│  • JVM cannot optimize reflective calls                         │
│                                                                 │
│  3. COMPILE-TIME SAFETY LOST                                    │
│  ───────────────────────────                                    │
│  • Errors discovered at runtime, not compile time               │
│  • No IDE autocomplete or validation                            │
│  • Typos in method/field names cause runtime exceptions         │
│                                                                 │
│  4. CODE COMPLEXITY                                             │
│  ─────────────────────                                          │
│  • Harder to read and understand                                │
│  • More difficult to maintain                                   │
│  • Requires exception handling                                  │
│                                                                 │
│  5. SECURITY CONCERNS                                           │
│  ─────────────────────                                          │
│  • Can bypass security mechanisms                               │
│  • May be restricted in secure environments                     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### When to Use Reflection

| Use Case | Example |
|----------|---------|
| Frameworks | Spring, Hibernate create objects dynamically |
| Testing | JUnit accesses private methods for testing |
| Serialization | Converting objects to JSON/XML |
| IDE Features | Code completion, refactoring tools |
| Plugin Systems | Loading classes at runtime |

### When NOT to Use Reflection

| Avoid When | Reason |
|------------|--------|
| Direct access is possible | Reflection is slower |
| Performance is critical | Runtime overhead |
| Code needs to be maintainable | Increases complexity |
| Security is important | Bypasses access control |

---

## 12. Quick Reference Summary

### Package

```java
import java.lang.reflect.Method;
import java.lang.reflect.Field;
import java.lang.reflect.Constructor;
import java.lang.reflect.Modifier;
```

### Common Methods Cheat Sheet

```
┌─────────────────────────────────────────────────────────────────┐
│                    REFLECTION CHEAT SHEET                        │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  GET CLASS OBJECT:                                              │
│  Class<?> c = Class.forName("ClassName");                       │
│  Class<?> c = ClassName.class;                                  │
│  Class<?> c = object.getClass();                                │
│                                                                 │
│  CREATE INSTANCE:                                               │
│  Object obj = c.newInstance();  // deprecated                   │
│  Object obj = c.getDeclaredConstructor().newInstance();         │
│                                                                 │
│  GET METHODS:                                                   │
│  Method[] m = c.getMethods();          // public + inherited    │
│  Method[] m = c.getDeclaredMethods();  // all, this class only  │
│  Method m = c.getMethod("name", paramTypes...);                 │
│  Method m = c.getDeclaredMethod("name", paramTypes...);         │
│                                                                 │
│  GET FIELDS:                                                    │
│  Field[] f = c.getFields();            // public + inherited    │
│  Field[] f = c.getDeclaredFields();    // all, this class only  │
│  Field f = c.getField("name");                                  │
│  Field f = c.getDeclaredField("name");                          │
│                                                                 │
│  GET CONSTRUCTORS:                                              │
│  Constructor[] cons = c.getConstructors();                      │
│  Constructor[] cons = c.getDeclaredConstructors();              │
│                                                                 │
│  ACCESS PRIVATE MEMBERS:                                        │
│  field.setAccessible(true);                                     │
│  method.setAccessible(true);                                    │
│  constructor.setAccessible(true);                               │
│                                                                 │
│  INVOKE/SET:                                                    │
│  method.invoke(object, args...);                                │
│  field.set(object, value);                                      │
│  field.get(object);                                             │
│  constructor.newInstance(args...);                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Key Points

1. **What is Reflection?**
    - Ability to examine and modify classes at runtime

2. **How to get Class object?**
    - `Class.forName("name")`, `ClassName.class`, `obj.getClass()`

3. **getMethods() vs getDeclaredMethods()?**
    - `getMethods()`: public only, includes inherited
    - `getDeclaredMethods()`: all (public + private), this class only

4. **How to access private members?**
    - Use `setAccessible(true)`

5. **How does reflection break Singleton?**
    - Can invoke private constructor to create multiple instances

6. **Why use reflection sparingly?**
    - Breaks encapsulation, slow performance, no compile-time safety