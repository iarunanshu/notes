# Comprehensive Notes: Java Annotations

## Overview - What We'll Cover

| Topic | Description |
|-------|-------------|
| What is Annotation | Metadata added to Java code |
| Annotation Categories | Predefined vs Custom annotations |
| Annotations on Java Code | @Deprecated, @Override, @SuppressWarnings, @FunctionalInterface, @SafeVarargs |
| Meta-Annotations | @Target, @Retention, @Documented, @Inherited, @Repeatable |
| Custom Annotations | Creating your own annotations |

---

## 1. What is Annotation?

### Definition

**Annotation** is a way of adding **metadata** (data about data) to Java code. It provides additional information to the compiler, JVM, or other tools.

### Key Points

| Point | Description |
|-------|-------------|
| **Optional** | Annotations are optional; code works without them |
| **Metadata** | Just additional information, not actual code logic |
| **Access via Reflection** | Can be read at runtime using Reflection API |
| **Applied Anywhere** | Can be used on classes, methods, fields, parameters, etc. |
| **Denoted by @** | All annotations start with `@` symbol |

### Visual Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                    WHAT IS ANNOTATION?                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   @Override                    ← Annotation (metadata)          │
│   public void fly() {          ← Actual code                    │
│       System.out.println("Flying");                             │
│   }                                                             │
│                                                                 │
│   The @Override annotation:                                     │
│   • Tells compiler: "This method overrides a parent method"     │
│   • Compiler adds validation logic                              │
│   • If no matching parent method exists → Compile Error         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### How Annotations Work

```
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│   Your Code      │     │    Compiler/     │     │    Logic         │
│   + Annotation   │────►│    JVM reads     │────►│    Executed      │
│                  │     │    metadata      │     │                  │
└──────────────────┘     └──────────────────┘     └──────────────────┘

Example:
@Override          →    Compiler checks    →    Error if no parent
                        parent class            method exists
```

---

## 2. Types of Annotations

### Classification

```
┌─────────────────────────────────────────────────────────────────┐
│                    ANNOTATION TYPES                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ANNOTATIONS                                                   │
│       │                                                         │
│       ├── PREDEFINED (Built-in by Java)                         │
│       │       │                                                 │
│       │       ├── Used on Java Code                             │
│       │       │   • @Deprecated                                 │
│       │       │   • @Override                                   │
│       │       │   • @SuppressWarnings                           │
│       │       │   • @FunctionalInterface                        │
│       │       │   • @SafeVarargs                                │
│       │       │                                                 │
│       │       └── Meta-Annotations (used on other annotations)  │
│       │           • @Target                                     │
│       │           • @Retention                                  │
│       │           • @Documented                                 │
│       │           • @Inherited                                  │
│       │           • @Repeatable                                 │
│       │                                                         │
│       └── CUSTOM (User-defined)                                 │
│           • Created using @interface keyword                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 3. Predefined Annotations (Used on Java Code)

### 3.1 @Deprecated

**Purpose:** Marks code as outdated; warns users to use alternatives

**Can be applied to:** Constructor, Field, Local Variable, Method, Package, Parameter, Type (Class/Interface/Enum)

```java
public class Mobile {
    
    @Deprecated
    public void oldMethod() {
        System.out.println("This is old implementation");
    }
    
    public void newMethod() {
        System.out.println("Use this instead!");
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        Mobile mobile = new Mobile();
        
        mobile.oldMethod();  // ⚠️ Warning: 'oldMethod()' is deprecated
        mobile.newMethod();  // ✓ No warning
    }
}
```

### Visual Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                      @Deprecated                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   What it does:                                                 │
│   • Marks method/class/field as outdated                        │
│   • Shows warning when deprecated code is used                  │
│   • Suggests using alternative (if available)                   │
│                                                                 │
│   Usage:                                                        │
│   @Deprecated                                                   │
│   public void oldMethod() { }  // Don't use this anymore        │
│                                                                 │
│   When used:                                                    │
│   mobile.oldMethod();  // ⚠️ Strikethrough + Warning            │
│                                                                 │
│   Meaning:                                                      │
│   "No further development on this code. Use alternative."       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 3.2 @Override

**Purpose:** Indicates that a method overrides a parent class/interface method

**Can be applied to:** Methods only

```java
interface Bird {
    void fly();
}

class Eagle implements Bird {
    
    @Override
    public void fly() {  // ✓ Correct - matches interface method
        System.out.println("Eagle flying");
    }
    
    @Override
    public void flyOne() {  // ❌ Compile Error - no such method in Bird
        System.out.println("Error!");
    }
}
```

### How @Override Helps

```
┌─────────────────────────────────────────────────────────────────┐
│                      @Override                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Without @Override:                                            │
│   ─────────────────                                             │
│   public void flly() { }  // Typo! Creates NEW method           │
│                           // No error, but bug introduced       │
│                                                                 │
│   With @Override:                                               │
│   ────────────────                                              │
│   @Override                                                     │
│   public void flly() { }  // ❌ Compile Error!                  │
│                           // "Method does not override          │
│                           //  method from superclass"           │
│                                                                 │
│   Benefit: Catches typos and mistakes at compile time           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 3.3 @SuppressWarnings

**Purpose:** Tells compiler to ignore specific warnings

**Can be applied to:** Field, Method, Parameter, Constructor, Local Variable, Type

### Common Warning Types

| Warning Type | Description |
|--------------|-------------|
| `"deprecation"` | Using deprecated code |
| `"unused"` | Unused variables/methods |
| `"unchecked"` | Unchecked type operations |
| `"rawtypes"` | Using raw types without generics |
| `"all"` | Suppress ALL warnings |

### Examples

```java
public class Main {
    
    // Suppress deprecation warning for this method
    @SuppressWarnings("deprecation")
    public void useOldCode() {
        Mobile mobile = new Mobile();
        mobile.oldMethod();  // No warning shown
    }
    
    // Suppress multiple warnings
    @SuppressWarnings({"deprecation", "unused"})
    public void multipleWarnings() {
        String unused = "test";  // No "unused" warning
        new Mobile().oldMethod();  // No "deprecation" warning
    }
    
    // Suppress all warnings for entire class
    @SuppressWarnings("all")
    public void suppressEverything() {
        // No warnings at all in this method
    }
}
```

### Warning About @SuppressWarnings

```
┌─────────────────────────────────────────────────────────────────┐
│              ⚠️ USE WITH CAUTION!                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Warnings exist for a reason!                                  │
│                                                                 │
│   Example of dangerous suppression:                             │
│   ─────────────────────────────────                             │
│   @SuppressWarnings("all")                                      │
│   public void calculate() {                                     │
│       int result = 5 / 0;  // No warning, but RUNTIME ERROR!    │
│   }                                                             │
│                                                                 │
│   Best Practice:                                                │
│   • Only suppress warnings you understand                       │
│   • Be specific (don't use "all" unless necessary)              │
│   • Add comment explaining why suppression is safe              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 3.4 @FunctionalInterface

**Purpose:** Ensures interface has exactly ONE abstract method (for lambda expressions)

**Can be applied to:** Type (Interface)

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);  // Only ONE abstract method allowed
    
    // ✓ Default methods are allowed
    default void print() {
        System.out.println("Calculator");
    }
    
    // ✓ Static methods are allowed
    static void info() {
        System.out.println("Info");
    }
}

// ❌ This will cause compile error
@FunctionalInterface
interface InvalidCalculator {
    int add(int a, int b);
    int subtract(int a, int b);  // ERROR: Second abstract method!
}
```

---

### 3.5 @SafeVarargs

**Purpose:** Suppresses heap pollution warnings for methods with varargs

**Can be applied to:** Methods (static or final only) and Constructors

**Java 9+:** Can also be used on private methods

### What is Heap Pollution?

```
┌─────────────────────────────────────────────────────────────────┐
│                    HEAP POLLUTION                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Definition:                                                   │
│   When an object of Type A stores reference to object of Type B │
│                                                                 │
│   Example:                                                      │
│   ─────────                                                     │
│   List<String> stringList = new ArrayList<>();                  │
│   stringList.add("Hello");                                      │
│                                                                 │
│   Object[] objArray = stringList.toArray();                     │
│   objArray[0] = 123;  // Putting Integer in String list!        │
│                                                                 │
│   // stringList now contains Integer (HEAP POLLUTION!)          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Why Varargs Can Cause Heap Pollution

```java
public class VarargsExample {
    
    // This method causes "possible heap pollution" warning
    public static void printLogValues(List<Integer>... logNumberList) {
        Object[] objArray = logNumberList;  // Varargs → Array
        
        List<String> stringList = new ArrayList<>();
        stringList.add("Hello");
        
        objArray[0] = stringList;  // HEAP POLLUTION!
        // logNumberList (List<Integer>[]) now contains List<String>!
    }
}
```

### Using @SafeVarargs

```java
public class SafeExample {
    
    // Suppresses heap pollution warning
    @SafeVarargs
    public static void safeMethod(List<String>... lists) {
        for (List<String> list : lists) {
            System.out.println(list);
        }
        // Safe because we're not doing anything dangerous
    }
    
    // Must be static, final, or private (Java 9+)
    @SafeVarargs
    public final void finalMethod(List<Integer>... numbers) {
        // Implementation
    }
}
```

### @SafeVarargs Requirements

```
┌─────────────────────────────────────────────────────────────────┐
│               @SafeVarargs REQUIREMENTS                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Method must be:                                               │
│   • static       OR                                             │
│   • final        OR                                             │
│   • private (Java 9+)                                           │
│                                                                 │
│   Why?                                                          │
│   These methods CANNOT be overridden, so child class can't      │
│   accidentally remove the safety guarantee.                     │
│                                                                 │
│   ❌ Cannot use on:                                             │
│   public void method(List<String>... lists) { }                 │
│   // Not static, not final, not private                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 4. Meta-Annotations (Annotations on Annotations)

Meta-annotations are annotations that are applied to OTHER annotations.

### 4.1 @Target

**Purpose:** Specifies WHERE an annotation can be applied

### Element Types

| ElementType | Can Apply To |
|-------------|--------------|
| `TYPE` | Class, Interface, Enum |
| `FIELD` | Member variables |
| `METHOD` | Methods |
| `PARAMETER` | Method parameters |
| `CONSTRUCTOR` | Constructors |
| `LOCAL_VARIABLE` | Local variables |
| `ANNOTATION_TYPE` | Other annotations |
| `PACKAGE` | Package declarations |
| `TYPE_PARAMETER` | Generic type parameters (Java 8+) |
| `TYPE_USE` | Any type usage (Java 8+) |

### Example: How @Override is Defined

```java
@Target(ElementType.METHOD)  // Can ONLY be used on methods
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

### Example: Custom Annotation with Multiple Targets

```java
@Target({ElementType.METHOD, ElementType.CONSTRUCTOR})
public @interface MyAnnotation {
}

// Now can be used on both methods and constructors
class Example {
    @MyAnnotation
    public Example() { }  // ✓ Constructor
    
    @MyAnnotation
    public void method() { }  // ✓ Method
    
    @MyAnnotation  // ❌ ERROR - not allowed on fields
    private String field;
}
```

### Visual: @Target Options

```
┌─────────────────────────────────────────────────────────────────┐
│                    @Target ElementTypes                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   @Target(ElementType.TYPE)                                     │
│   ─────────────────────────                                     │
│   @MyAnnotation                                                 │
│   public class MyClass { }     ← Applied to class               │
│                                                                 │
│   @Target(ElementType.METHOD)                                   │
│   ──────────────────────────                                    │
│   @MyAnnotation                                                 │
│   public void myMethod() { }   ← Applied to method              │
│                                                                 │
│   @Target(ElementType.FIELD)                                    │
│   ─────────────────────────                                     │
│   @MyAnnotation                                                 │
│   private String name;         ← Applied to field               │
│                                                                 │
│   @Target(ElementType.PARAMETER)                                │
│   ──────────────────────────────                                │
│   public void method(@MyAnnotation String param) { }            │
│                      ↑ Applied to parameter                     │
│                                                                 │
│   @Target(ElementType.ANNOTATION_TYPE)                          │
│   ────────────────────────────────────                          │
│   @MyAnnotation                                                 │
│   public @interface AnotherAnnotation { }                       │
│   ↑ Applied to another annotation (META-ANNOTATION)             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.2 @Retention

**Purpose:** Specifies HOW LONG the annotation is retained

### Retention Policies

| Policy | Retained In | Available At | Use Case |
|--------|-------------|--------------|----------|
| `SOURCE` | Source code only | Compile time | @Override |
| `CLASS` | .class file | Not at runtime | Default if not specified |
| `RUNTIME` | .class file + JVM | Runtime (Reflection) | Most custom annotations |

### Visual Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                 RETENTION POLICIES                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   .java file          .class file         JVM Runtime           │
│   ───────────         ───────────         ───────────           │
│                                                                 │
│   RetentionPolicy.SOURCE                                        │
│   ──────────────────────                                        │
│   [✓ Present]    →    [✗ Discarded]  →    [✗ Not available]    │
│   Example: @Override                                            │
│                                                                 │
│   RetentionPolicy.CLASS                                         │
│   ─────────────────────                                         │
│   [✓ Present]    →    [✓ Present]    →    [✗ Not available]    │
│   (Cannot use Reflection to access)                             │
│                                                                 │
│   RetentionPolicy.RUNTIME                                       │
│   ───────────────────────                                       │
│   [✓ Present]    →    [✓ Present]    →    [✓ Available]        │
│   (CAN use Reflection to access)                                │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Example: Retention Policy Effect

```java
// SOURCE - Only for compiler
@Retention(RetentionPolicy.SOURCE)
@Target(ElementType.TYPE)
@interface SourceAnnotation { }

// RUNTIME - Available via Reflection
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface RuntimeAnnotation { }
```

```java
@SourceAnnotation
@RuntimeAnnotation
public class TestClass { }
```

```java
public class Main {
    public static void main(String[] args) {
        Class<?> clazz = TestClass.class;
        
        // Try to get SourceAnnotation
        SourceAnnotation source = clazz.getAnnotation(SourceAnnotation.class);
        System.out.println(source);  // null (not available at runtime)
        
        // Try to get RuntimeAnnotation
        RuntimeAnnotation runtime = clazz.getAnnotation(RuntimeAnnotation.class);
        System.out.println(runtime);  // @RuntimeAnnotation() (available!)
    }
}
```

---

### 4.3 @Documented

**Purpose:** Includes annotation in JavaDoc documentation

**Default Behavior:** Annotations are NOT included in JavaDoc

```java
// Without @Documented
@Retention(RetentionPolicy.RUNTIME)
@interface NotDocumented { }

// With @Documented
@Documented
@Retention(RetentionPolicy.RUNTIME)
@interface IsDocumented { }
```

```java
public class Example {
    @NotDocumented
    @IsDocumented
    public void myMethod() { }
}
```

### Generated JavaDoc

```
┌─────────────────────────────────────────────────────────────────┐
│                    JAVADOC OUTPUT                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Without @Documented:                                          │
│   ────────────────────                                          │
│   public void myMethod()                                        │
│   // @NotDocumented is NOT shown                                │
│                                                                 │
│   With @Documented:                                             │
│   ──────────────────                                            │
│   @IsDocumented                                                 │
│   public void myMethod()                                        │
│   // @IsDocumented IS shown in JavaDoc                          │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.4 @Inherited

**Purpose:** Makes annotation automatically inherited by child classes

**Default Behavior:** Annotations are NOT inherited

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface MyInheritedAnnotation { }

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface MyNormalAnnotation { }
```

```java
@MyInheritedAnnotation
@MyNormalAnnotation
class ParentClass { }

class ChildClass extends ParentClass { }
// ChildClass automatically has @MyInheritedAnnotation
// ChildClass does NOT have @MyNormalAnnotation
```

```java
public class Main {
    public static void main(String[] args) {
        Class<?> childClass = ChildClass.class;
        
        // Check for inherited annotation
        MyInheritedAnnotation inherited = 
            childClass.getAnnotation(MyInheritedAnnotation.class);
        System.out.println(inherited);  // @MyInheritedAnnotation (inherited!)
        
        // Check for non-inherited annotation
        MyNormalAnnotation normal = 
            childClass.getAnnotation(MyNormalAnnotation.class);
        System.out.println(normal);  // null (not inherited)
    }
}
```

### Visual Explanation

```
┌─────────────────────────────────────────────────────────────────┐
│                    @Inherited Effect                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Without @Inherited:                                           │
│   ───────────────────                                           │
│                                                                 │
│   @NormalAnnotation                                             │
│   class Parent { }                                              │
│        │                                                        │
│        └──► class Child extends Parent { }                      │
│             // Does NOT have @NormalAnnotation                  │
│                                                                 │
│   ═══════════════════════════════════════════════════════════   │
│                                                                 │
│   With @Inherited:                                              │
│   ────────────────                                              │
│                                                                 │
│   @InheritedAnnotation                                          │
│   class Parent { }                                              │
│        │                                                        │
│        └──► class Child extends Parent { }                      │
│             // AUTOMATICALLY has @InheritedAnnotation           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

### 4.5 @Repeatable (Java 8+)

**Purpose:** Allows same annotation to be used multiple times on same element

**Default Behavior:** Same annotation CANNOT be repeated

### Without @Repeatable

```java
@interface Category {
    String name();
}

@Category(name = "Bird")
@Category(name = "Flying")  // ❌ ERROR: Duplicate annotation
class Eagle { }
```

### With @Repeatable (Two-Step Process)

**Step 1:** Create Container Annotation

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Categories {  // Container annotation
    Category[] value();  // Array of the repeatable annotation
}
```

**Step 2:** Create Repeatable Annotation

```java
@Repeatable(Categories.class)  // Point to container
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Category {
    String name();
}
```

**Step 3:** Use Multiple Times

```java
@Category(name = "Bird")
@Category(name = "Living Thing")
@Category(name = "Carnivorous")
class Eagle { }
```

### Complete Example with Access

```java
// Step 1: Container annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Categories {
    Category[] value();
}

// Step 2: Repeatable annotation
@Repeatable(Categories.class)
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@interface Category {
    String name();
}

// Step 3: Usage
@Category(name = "Bird")
@Category(name = "Living Thing")
@Category(name = "Carnivorous")
class Eagle { }

// Step 4: Access via Reflection
public class Main {
    public static void main(String[] args) {
        Class<Eagle> eagleClass = Eagle.class;
        
        Category[] categories = eagleClass.getAnnotationsByType(Category.class);
        
        for (Category category : categories) {
            System.out.println(category.name());
        }
        // Output:
        // Bird
        // Living Thing
        // Carnivorous
    }
}
```

### Visual: @Repeatable Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                @Repeatable STRUCTURE                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────────────────────────────────────────┐               │
│   │  Container: @Categories                      │               │
│   │  ─────────────────────────────────────────  │               │
│   │  Category[] value = [                        │               │
│   │      @Category(name="Bird"),                 │               │
│   │      @Category(name="Living Thing"),         │               │
│   │      @Category(name="Carnivorous")           │               │
│   │  ]                                           │               │
│   └─────────────────────────────────────────────┘               │
│                         ▲                                       │
│                         │                                       │
│   @Category(name="Bird")                                        │
│   @Category(name="Living Thing")      These get stored          │
│   @Category(name="Carnivorous")  ───► in the container          │
│   class Eagle { }                                               │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## 5. Custom Annotations

### Basic Syntax

```java
@interface AnnotationName {
    // Annotation members (optional)
}
```

### Creating Custom Annotation

**Step 1:** Basic Empty Annotation

```java
@interface MyAnnotation {
    // Empty body
}

// Usage
@MyAnnotation
public class MyClass { }
```

**Step 2:** With Members (Elements)

```java
@interface MyAnnotation {
    String name();      // Required member
    int value();        // Required member
}

// Usage
@MyAnnotation(name = "Test", value = 42)
public class MyClass { }
```

**Step 3:** With Default Values

```java
@interface MyAnnotation {
    String name() default "Unknown";  // Optional (has default)
    int value() default 0;            // Optional (has default)
}

// Usage - can skip members with defaults
@MyAnnotation
public class MyClass { }

// Or override defaults
@MyAnnotation(name = "Custom", value = 100)
public class AnotherClass { }
```

### Member Restrictions

```
┌─────────────────────────────────────────────────────────────────┐
│              ANNOTATION MEMBER RULES                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Allowed Return Types:                                         │
│   • Primitive types (int, boolean, double, etc.)                │
│   • String                                                      │
│   • Class<?>                                                    │
│   • Enum                                                        │
│   • Another Annotation                                          │
│   • Array of above types                                        │
│                                                                 │
│   NOT Allowed:                                                  │
│   • Custom objects (new MyClass())                              │
│   • Collections (List, Map, etc.)                               │
│   • null as default value                                       │
│                                                                 │
│   Syntax Rules:                                                 │
│   • Looks like method: String name();                           │
│   • No parameters allowed: String name(int x);  ❌              │
│   • No body allowed: String name() { return ""; }  ❌           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Complete Custom Annotation Example

```java
// Define the annotation with all meta-annotations
@Retention(RetentionPolicy.RUNTIME)  // Available at runtime
@Target(ElementType.TYPE)            // Can only apply to classes
@Documented                          // Include in JavaDoc
@Inherited                           // Inherited by child classes
@interface ApiInfo {
    String author() default "Unknown";
    String version() default "1.0";
    String[] tags() default {};
    int priority() default 1;
}
```

```java
// Use the annotation
@ApiInfo(
    author = "John Doe",
    version = "2.5",
    tags = {"REST", "Public"},
    priority = 5
)
public class UserController {
    // Class implementation
}
```

```java
// Access annotation via Reflection
public class Main {
    public static void main(String[] args) {
        Class<UserController> clazz = UserController.class;
        
        if (clazz.isAnnotationPresent(ApiInfo.class)) {
            ApiInfo info = clazz.getAnnotation(ApiInfo.class);
            
            System.out.println("Author: " + info.author());
            System.out.println("Version: " + info.version());
            System.out.println("Priority: " + info.priority());
            System.out.println("Tags: " + Arrays.toString(info.tags()));
        }
    }
}
// Output:
// Author: John Doe
// Version: 2.5
// Priority: 5
// Tags: [REST, Public]
```

---

## 6. Quick Reference Summary

### All Annotations at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│              ANNOTATION QUICK REFERENCE                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  JAVA CODE ANNOTATIONS:                                         │
│  ───────────────────────                                        │
│  @Override         → Method overrides parent                    │
│  @Deprecated       → Code is outdated                           │
│  @SuppressWarnings → Ignore compiler warnings                   │
│  @FunctionalInterface → Interface has one abstract method       │
│  @SafeVarargs      → Safe varargs usage                         │
│                                                                 │
│  META-ANNOTATIONS:                                              │
│  ─────────────────                                              │
│  @Target           → WHERE annotation can be applied            │
│  @Retention        → HOW LONG annotation is retained            │
│  @Documented       → Include in JavaDoc                         │
│  @Inherited        → Pass to child classes                      │
│  @Repeatable       → Allow multiple usage                       │
│                                                                 │
│  CUSTOM ANNOTATION SYNTAX:                                      │
│  ─────────────────────────                                      │
│  @Retention(RetentionPolicy.RUNTIME)                            │
│  @Target(ElementType.TYPE)                                      │
│  @interface MyAnnotation {                                      │
│      String value() default "default";                          │
│  }                                                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Interview Key Points

1. **What is an annotation?**
    - Metadata added to code using `@` symbol
    - Read by compiler, tools, or at runtime via reflection

2. **Difference between @Retention policies?**
    - SOURCE: Only in source code, discarded by compiler
    - CLASS: In .class file, not available at runtime
    - RUNTIME: Available at runtime via reflection

3. **What is a meta-annotation?**
    - An annotation applied to another annotation
    - Examples: @Target, @Retention, @Inherited

4. **How to create a custom annotation?**
    - Use `@interface` keyword
    - Add meta-annotations for configuration
    - Define members (look like methods)

5. **What is @Repeatable used for?**
    - Allows same annotation multiple times on same element
    - Requires container annotation