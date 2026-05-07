# Comprehensive Notes: Generic Classes in Java

## Introduction

**Generics** enable you to write classes, interfaces, and methods that work with different types while providing compile-time type safety. They eliminate the need for explicit type casting and help catch type-related errors at compile time rather than runtime.

---

## 1. The Problem: Why Do We Need Generics?

### Without Generics (Using Object Class)

```java
public class Print {
    private Object value;  // Can hold ANY object
    
    public void setPrintValue(Object value) {
        this.value = value;
    }
    
    public Object getPrintValue() {
        return this.value;
    }
}
```

### Usage Without Generics:

```java
public static void main(String[] args) {
    Print obj = new Print();
    
    // Can set ANY type
    obj.setPrintValue(1);           // Integer
    obj.setPrintValue("Hello");     // String - also valid!
    obj.setPrintValue(new Car());   // Custom object - also valid!
    
    // Problem: Must typecast when retrieving
    Object result = obj.getPrintValue();
    
    // We don't know the actual type!
    if (result instanceof Integer) {
        int num = (Integer) result;  // Explicit casting required
    } else if (result instanceof String) {
        String str = (String) result;  // Explicit casting required
    }
}
```

### Problems with Object Approach:

| Problem | Description |
|---------|-------------|
| **No Type Safety** | Any type can be stored, leading to potential runtime errors |
| **Explicit Casting Required** | Must cast every time you retrieve a value |
| **Runtime Errors** | ClassCastException can occur if wrong type is cast |
| **No Compile-time Checks** | Errors discovered only at runtime |

---

## 2. Solution: Generic Classes

### Basic Syntax

```java
public class ClassName<T> {
    // T can be used as a type throughout the class
}
```

**Diamond Syntax:** `< >` with type parameter(s) inside

### Generic Version of Print Class:

```java
public class Print<T> {
    private T value;  // Type determined at object creation
    
    public void setPrintValue(T value) {
        this.value = value;
    }
    
    public T getPrintValue() {
        return this.value;
    }
}
```

### Usage With Generics:

```java
public static void main(String[] args) {
    // Specify type at object creation
    Print<Integer> obj1 = new Print<>();
    
    obj1.setPrintValue(1);           // ✓ Valid
    // obj1.setPrintValue("Hello");  // ✗ Compile-time error!
    
    // No casting needed!
    Integer result = obj1.getPrintValue();  // Type-safe
    
    // Another object for String
    Print<String> obj2 = new Print<>();
    obj2.setPrintValue("Hello");     // ✓ Valid
    String str = obj2.getPrintValue();  // No casting needed
}
```

### Benefits of Generics:

| Benefit | Description |
|---------|-------------|
| **Type Safety** | Compiler enforces type constraints |
| **No Casting** | Return type is already known |
| **Compile-time Errors** | Type mismatches caught during compilation |
| **Code Reusability** | Same class works with different types |

### Important Note:

> **Type Parameter can only be replaced with NON-PRIMITIVE types**
> - ✓ `Integer`, `String`, `Double`, custom classes
> - ✗ `int`, `float`, `boolean` (primitives not allowed)

```java
Print<int> obj = new Print<>();      // ✗ Compile error
Print<Integer> obj = new Print<>();  // ✓ Valid (use wrapper class)
```

---

## 3. Common Type Parameter Naming Conventions

| Letter | Convention | Example Usage |
|--------|------------|---------------|
| `T` | Type | General purpose |
| `E` | Element | Collections (List<E>) |
| `K` | Key | Maps (Map<K,V>) |
| `V` | Value | Maps (Map<K,V>) |
| `N` | Number | Numeric types |
| `S, U, V` | 2nd, 3rd, 4th types | Multiple type parameters |

---

## 4. Inheritance with Generic Classes

### Case 1: Non-Generic Subclass

When the child class is **NOT** generic, you must specify the type at inheritance time.

```java
// Generic parent class
public class Print<T> {
    private T value;
    // ... methods
}

// Non-generic child - MUST specify type when extending
public class ColorPrint extends Print<String> {
    // This class can only work with String type
}
```

**Usage:**
```java
ColorPrint cp = new ColorPrint();
cp.setPrintValue("Hello");  // Only String allowed
// cp.setPrintValue(123);   // ✗ Compile error
```

### Case 2: Generic Subclass

When the child class is also generic, type is specified at object creation time.

```java
// Generic parent class
public class Print<T> {
    private T value;
    // ... methods
}

// Generic child - type parameter passed through
public class ColorPrint<T> extends Print<T> {
    // Type determined when object is created
}
```

**Usage:**
```java
ColorPrint<String> cp1 = new ColorPrint<>();
cp1.setPrintValue("Hello");

ColorPrint<Integer> cp2 = new ColorPrint<>();
cp2.setPrintValue(123);
```

### Inheritance Diagram:

```
┌─────────────────────────┐
│    Print<T>             │  ← Generic Parent
│  ─────────────────────  │
│  T value                │
│  setPrintValue(T)       │
│  T getPrintValue()      │
└───────────┬─────────────┘
            │
     ┌──────┴──────┐
     │             │
     ▼             ▼
┌────────────┐  ┌────────────────┐
│ ColorPrint │  │ ColorPrint<T>  │
│ extends    │  │ extends        │
│ Print<String>│ │ Print<T>       │
└────────────┘  └────────────────┘
 Non-Generic      Generic
 (Type fixed)     (Type flexible)
```

---

## 5. Multiple Type Parameters

You can define multiple type parameters for a single class.

### Syntax:

```java
public class ClassName<T1, T2, T3, ... Tn> {
    // Use T1, T2, T3, etc. as types
}
```

### Example: Key-Value Pair Class

```java
public class Pair<K, V> {
    private K key;
    private V value;
    
    public void put(K key, V value) {
        this.key = key;
        this.value = value;
    }
    
    public K getKey() {
        return key;
    }
    
    public V getValue() {
        return value;
    }
}
```

### Usage:

```java
// Both syntaxes are valid
Pair<String, Integer> pair1 = new Pair<String, Integer>();
Pair<String, Integer> pair2 = new Pair<>();  // Type inference

pair1.put("Age", 25);

String key = pair1.getKey();      // "Age"
Integer value = pair1.getValue(); // 25
```

---

## 6. Generic Methods

You can make individual methods generic without making the entire class generic.

### Syntax:

```java
public <T> ReturnType methodName(T parameter) {
    // Method body
}
```

> **Important:** Type parameter `<T>` must come **BEFORE** the return type

### Example:

```java
public class GenericMethodClass {
    
    // Non-generic class with generic method
    public <T> void setValue(T value) {
        System.out.println("Value: " + value);
        System.out.println("Type: " + value.getClass().getName());
    }
}
```

### Usage:

```java
GenericMethodClass obj = new GenericMethodClass();

obj.setValue(123);           // T is Integer
obj.setValue("Hello");       // T is String  
obj.setValue(new Bus());     // T is Bus
obj.setValue(3.14);          // T is Double
```

### Properties of Generic Methods:

| Property | Description |
|----------|-------------|
| **Type Parameter Location** | Before return type: `public <T> void method()` |
| **Scope** | Limited to the method only |
| **Class Requirement** | Class doesn't need to be generic |
| **Multiple Parameters** | Can have multiple: `public <T, U> void method(T t, U u)` |

### Generic Method vs Generic Class:

```java
// Generic Class - T available throughout class
public class Print<T> {
    private T value;
    public void set(T val) { this.value = val; }
    public T get() { return value; }
}

// Generic Method - T available only in this method
public class Utility {
    public <T> void printValue(T value) {
        System.out.println(value);
    }
}
```

---

## 7. Raw Types

A **Raw Type** is when you use a generic class without specifying the type parameter.

### Example:

```java
// Parameterized type (recommended)
Print<String> typed = new Print<>();

// Raw type (not recommended)
Print raw = new Print();  // No type parameter specified
```

### What Happens with Raw Types:

```java
Print raw = new Print();  // Internally becomes Print<Object>

raw.setPrintValue(1);        // ✓ Accepts Integer
raw.setPrintValue("Hello");  // ✓ Accepts String
raw.setPrintValue(new Car());// ✓ Accepts anything!

// Back to the original problem - must cast!
Object result = raw.getPrintValue();
```

### Raw Type Behavior:

```
┌───────────────────────────────────────┐
│         Print raw = new Print();      │
│                   │                   │
│                   ▼                   │
│    Compiler treats as Print<Object>  │
│                   │                   │
│                   ▼                   │
│    - Accepts any object              │
│    - Returns Object type             │
│    - Requires casting                │
│    - Loses type safety               │
└───────────────────────────────────────┘
```

> **Warning:** Raw types exist for backward compatibility with pre-generics code. Always use parameterized types in new code.

---

## 8. Bounded Generics

Bounded generics **restrict** which types can be used as type arguments.

### 8.1 Upper Bound

**Syntax:** `<T extends UpperBoundClass>`

Allows the type parameter to be the specified class **or any of its subclasses**.

```
            ┌──────────┐
            │  Object  │
            └────┬─────┘
                 │
            ┌────┴─────┐
            │  Number  │ ◄── Upper Bound
            └────┬─────┘
       ┌─────────┼─────────┐
       ▼         ▼         ▼
  ┌─────────┐ ┌─────────┐ ┌─────────┐
  │ Integer │ │ Double  │ │  Float  │
  └─────────┘ └─────────┘ └─────────┘
  
  T extends Number → Can be Number, Integer, Double, Float, etc.
```

### Example:

```java
public class Print<T extends Number> {
    private T value;
    
    public void setPrintValue(T value) {
        this.value = value;
    }
    
    public T getPrintValue() {
        return this.value;
    }
}
```

### Usage:

```java
// ✓ Valid - Integer is a subclass of Number
Print<Integer> intPrint = new Print<>();
intPrint.setPrintValue(10);

// ✓ Valid - Double is a subclass of Number
Print<Double> doublePrint = new Print<>();
doublePrint.setPrintValue(3.14);

// ✗ Compile Error - String is NOT a subclass of Number
Print<String> stringPrint = new Print<>();  // ERROR!
```

### Upper Bound with Interface:

```java
// T must implement Comparable interface
public class SortedBox<T extends Comparable<T>> {
    // Can call compareTo() on T objects
}
```

> **Note:** Use `extends` keyword for both classes AND interfaces in bounded generics (not `implements`)

---

### 8.2 Multi-Bound (Multiple Bounds)

You can specify multiple bounds - one class and multiple interfaces.

### Syntax:

```java
<T extends Class & Interface1 & Interface2>
```

### Rules:

| Rule | Description |
|------|-------------|
| **Class First** | If there's a class, it must come first |
| **One Class Max** | Can have at most ONE class |
| **Multiple Interfaces** | Can have multiple interfaces |
| **Use `&`** | Separate bounds with `&` |

### Example:

```java
// Class hierarchy
class ParentClass { }
interface Interface1 { }
interface Interface2 { }

// Multi-bounded generic class
public class Print<T extends ParentClass & Interface1 & Interface2> {
    private T value;
    // ...
}
```

### Valid Type Arguments:

```java
// This class meets ALL bounds
class ValidClass extends ParentClass implements Interface1, Interface2 {
    // ...
}

// ✓ Valid - meets all bounds
Print<ValidClass> obj = new Print<>();
```

### Invalid Type Arguments:

```java
// Missing Interface2
class PartialClass extends ParentClass implements Interface1 {
    // ...
}

// ✗ Compile Error - doesn't implement Interface2
Print<PartialClass> obj = new Print<>();  // ERROR!
```

### Visual Representation:

```
Type T must satisfy ALL of these:
┌─────────────────────────────────────────┐
│                                         │
│  ┌─────────────────┐                    │
│  │  ParentClass    │ ← extends          │
│  └─────────────────┘                    │
│           AND                           │
│  ┌─────────────────┐                    │
│  │  Interface1     │ ← implements       │
│  └─────────────────┘                    │
│           AND                           │
│  ┌─────────────────┐                    │
│  │  Interface2     │ ← implements       │
│  └─────────────────┘                    │
│                                         │
└─────────────────────────────────────────┘
```

---

## 9. Wildcards

Wildcards (`?`) provide additional flexibility when working with generic types, especially in method parameters.

### Important Concept: List<Vehicle> is NOT a parent of List<Bus>

```java
class Vehicle { }
class Bus extends Vehicle { }
class Car extends Vehicle { }

// Object assignment - Valid
Vehicle v = new Bus();  // ✓ Parent can hold child reference

// List assignment - INVALID!
List<Vehicle> vehicleList = new ArrayList<>();
List<Bus> busList = new ArrayList<>();

vehicleList = busList;  // ✗ Compile Error!
busList = vehicleList;  // ✗ Compile Error!
```

### Why are List assignments invalid?

```java
List<Vehicle> vehicleList = new ArrayList<>();
vehicleList.add(new Bus());  // ✓ Valid
vehicleList.add(new Car());  // ✓ Valid

// If this were allowed...
List<Bus> busList = vehicleList;  // If allowed (but it's NOT)
Bus bus = busList.get(0);  // Would fail! vehicleList contains Cars too!
```

---

### 9.1 Upper Bounded Wildcard

**Syntax:** `<? extends Type>`

Accepts the specified type **and all its subtypes**.

```java
public void processVehicles(List<? extends Vehicle> vehicles) {
    // Can accept List<Vehicle>, List<Bus>, List<Car>
    for (Vehicle v : vehicles) {
        v.drive();
    }
}
```

### Usage:

```java
List<Vehicle> vehicleList = new ArrayList<>();
List<Bus> busList = new ArrayList<>();
List<Car> carList = new ArrayList<>();

processVehicles(vehicleList);  // ✓ Valid
processVehicles(busList);      // ✓ Valid
processVehicles(carList);      // ✓ Valid
```

### Visual:

```
<? extends Vehicle>

            ┌───────────┐
            │  Vehicle  │ ← Upper Bound
            └─────┬─────┘
           ┌──────┴──────┐
           ▼             ▼
      ┌─────────┐   ┌─────────┐
      │   Bus   │   │   Car   │
      └─────────┘   └─────────┘
      
Accepts: Vehicle, Bus, Car (and any other subclass)
```

---

### 9.2 Lower Bounded Wildcard

**Syntax:** `<? super Type>`

Accepts the specified type **and all its supertypes** (parent classes).

```java
public void addBuses(List<? super Bus> list) {
    // Can accept List<Bus>, List<Vehicle>, List<Object>
    list.add(new Bus());
}
```

### Usage:

```java
List<Bus> busList = new ArrayList<>();
List<Vehicle> vehicleList = new ArrayList<>();
List<Object> objectList = new ArrayList<>();

addBuses(busList);      // ✓ Valid - Bus
addBuses(vehicleList);  // ✓ Valid - Parent of Bus
addBuses(objectList);   // ✓ Valid - Parent of Vehicle

List<Car> carList = new ArrayList<>();
addBuses(carList);      // ✗ Error - Car is not a supertype of Bus
```

### Visual:

```
<? super Bus>

      ┌──────────┐
      │  Object  │ ✓ Accepted
      └────┬─────┘
           │
      ┌────┴─────┐
      │ Vehicle  │ ✓ Accepted
      └────┬─────┘
           │
      ┌────┴─────┐
      │   Bus    │ ✓ Accepted (Lower Bound)
      └──────────┘
           │
           ✗ Below Bus - NOT accepted
```

---

### 9.3 Unbounded Wildcard

**Syntax:** `<?>`

Accepts **any type**. Equivalent to `<? extends Object>`.

```java
public void printList(List<?> list) {
    for (Object item : list) {
        System.out.println(item);
    }
}
```

### Usage:

```java
List<Integer> intList = Arrays.asList(1, 2, 3);
List<String> strList = Arrays.asList("a", "b", "c");
List<Vehicle> vehicleList = new ArrayList<>();

printList(intList);      // ✓ Valid
printList(strList);      // ✓ Valid
printList(vehicleList);  // ✓ Valid
```

### When to Use Unbounded Wildcard:

1. When the method works with methods from `Object` class (like `toString()`, `equals()`)
2. When you don't need to know the specific type
3. When the method doesn't depend on the type parameter

---

## 10. Wildcards vs Generic Type Parameters

### Comparison Table:

| Feature | Wildcard (`?`) | Generic Type (`<T>`) |
|---------|---------------|---------------------|
| **Syntax** | `List<? extends Number>` | `<T extends Number> void method(List<T>)` |
| **Lower Bound** | ✓ Supported (`? super`) | ✗ Not supported |
| **Multiple Types** | ✗ Single `?` only | ✓ Multiple (`<T, K, V>`) |
| **Type Enforcement** | Different types allowed | Same type enforced |
| **Type Access** | Cannot reference type | Can reference type as `T` |

### Key Difference: Type Enforcement

```java
// Wildcard - Different types allowed for each parameter
public void processWildcard(List<? extends Number> src, List<? extends Number> dest) {
    // src can be List<Integer>, dest can be List<Double>
}

// Generic Type - Same type enforced for all parameters
public <T extends Number> void processGeneric(List<T> src, List<T> dest) {
    // Both must be same type: both List<Integer> or both List<Double>
}
```

### Example:

```java
List<Integer> intList = new ArrayList<>();
List<Double> doubleList = new ArrayList<>();

// Wildcard method
processWildcard(intList, doubleList);  // ✓ Valid - different types OK

// Generic method
processGeneric(intList, doubleList);   // ✗ Error - must be same type
processGeneric(intList, intList);      // ✓ Valid - same type
```

### When to Use Which:

| Use Wildcard When... | Use Generic Type When... |
|---------------------|-------------------------|
| Different types are acceptable | Same type must be enforced |
| You need lower bounds (`super`) | You need to reference the type |
| Simple, flexible API needed | Type relationships matter |
| Read-only operations | Need multiple type parameters |

---

## 11. Type Erasure

**Type Erasure** is the process by which the Java compiler removes all generic type information during compilation.

### What Happens During Compilation:

```
Source Code (with Generics)  →  Bytecode (Generics Removed)
```

### Generic Class Erasure:

**Before (Source Code):**
```java
public class Print<T> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}
```

**After (Bytecode - Conceptual):**
```java
public class Print {
    private Object value;  // T replaced with Object
    public void set(Object value) { this.value = value; }
    public Object get() { return value; }
}
```

### Bounded Generic Erasure:

**Before:**
```java
public class Print<T extends Number> {
    private T value;
    public void set(T value) { this.value = value; }
    public T get() { return value; }
}
```

**After:**
```java
public class Print {
    private Number value;  // T replaced with bound (Number)
    public void set(Number value) { this.value = value; }
    public Number get() { return value; }
}
```

### Generic Method Erasure:

**Unbounded:**
```java
// Before
public <T> void print(T value) { }

// After
public void print(Object value) { }
```

**Bounded:**
```java
// Before
public <T extends Vehicle> void process(T vehicle) { }

// After
public void process(Vehicle vehicle) { }
```

### Type Erasure Rules:

| Original | Replaced With |
|----------|---------------|
| `<T>` (unbounded) | `Object` |
| `<T extends Bound>` | `Bound` |
| `<T extends Class & Interface>` | `Class` (first bound) |

### Why Type Erasure?

1. **Backward Compatibility:** Works with pre-generics code
2. **No Runtime Overhead:** No generic type information at runtime
3. **Single Class File:** Same bytecode regardless of type parameters

### Implications of Type Erasure:

```java
// Cannot do at runtime:
if (obj instanceof List<String>) { }  // ✗ Error - type erased

// Cannot create:
T[] array = new T[10];  // ✗ Error - T unknown at runtime

// This is why:
List<String> strList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();
System.out.println(strList.getClass() == intList.getClass());  // true!
// Both are just ArrayList at runtime
```

---

## 12. Quick Reference Summary

### Generic Syntax Cheat Sheet:

```java
// Generic Class
public class Box<T> { }
public class Pair<K, V> { }
public class Bounded<T extends Number> { }

// Generic Method
public <T> void method(T param) { }
public <T extends Comparable<T>> T max(T a, T b) { }

// Wildcards
List<?> unbounded;
List<? extends Number> upperBound;
List<? super Integer> lowerBound;
```

### Bounded Types Summary:

| Type | Syntax | Accepts |
|------|--------|---------|
| **Upper Bound** | `T extends X` | X and subclasses |
| **Multi-Bound** | `T extends A & B & C` | Must satisfy all |
| **Upper Wildcard** | `? extends X` | X and subclasses |
| **Lower Wildcard** | `? super X` | X and superclasses |
| **Unbounded** | `?` | Any type |

### PECS Principle (Producer Extends, Consumer Super):

```java
// PRODUCER - Read from it → use EXTENDS
public void readFrom(List<? extends Vehicle> producer) {
    Vehicle v = producer.get(0);  // Reading - safe
}

// CONSUMER - Write to it → use SUPER
public void writeTo(List<? super Vehicle> consumer) {
    consumer.add(new Bus());  // Writing - safe
}
```

---

## 13. Interview Tips

1. **Know the difference** between `<T>` and `<?>` - type parameter vs wildcard
2. **Understand** why `List<Vehicle>` is not a parent of `List<Bus>`
3. **Remember** PECS: Producer Extends, Consumer Super
4. **Explain** type erasure and its implications
5. **Know** that primitives cannot be used as type parameters
6. **Understand** raw types and why they should be avoided
7. **Be able to explain** when to use bounded generics vs wildcards