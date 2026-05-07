# SOLID Principles - Comprehensive Notes

## Introduction

### Why SOLID Principles are Required?
- Design patterns follow these principles, but using patterns is **not mandatory**
- In interviews (e.g., "Design a parking lot"), you can create your own classes and relationships
- **Whatever solution you design MUST follow these principles**
- These principles are the **fundamentals/guidelines** that make designs:
    - Clean
    - Flexible
    - Easy to maintain

### Benefits of Following SOLID Principles
- Write better code
- Avoid duplicate code
- Easy to maintain
- Easy to understand
- Flexible
- Reduce complexity

---

## S - Single Responsibility Principle (SRP)

### Definition
> A class should have only **one reason to change**, meaning it should have only **one responsibility/one job**.

### ❌ Violation Example

```java
class Invoice {
    private Marker marker;
    private int quantity;
    
    // Business logic - computing total price
    public double calculateTotal() {
        return marker.price * quantity;
    }
    
    // Database operation
    public void saveToDB() {
        // SQL logic to save invoice
    }
    
    // Printing operation
    public void printInvoice() {
        // Print the invoice
    }
}
```

**Problems:**
- If tax calculation rules change → modify Invoice class
- If database structure changes → modify Invoice class
- If printing logic changes → modify Invoice class
- Class gets modified too frequently
- Higher chance of human error
- Already tested code needs retesting with every change

### ✅ Correct Implementation

```java
// Class 1: Only handles business computation
class Invoice {
    private Marker marker;
    private int quantity;
    
    public double calculateTotal() {
        return marker.price * quantity;
    }
}

// Class 2: Only handles database operations
class InvoiceDao {
    private Invoice invoice;
    
    public InvoiceDao(Invoice invoice) {
        this.invoice = invoice;
    }
    
    public void saveToDB() {
        // Save to database
    }
}

// Class 3: Only handles printing
class InvoicePrinter {
    private Invoice invoice;
    
    public InvoicePrinter(Invoice invoice) {
        this.invoice = invoice;
    }
    
    public void print() {
        // Print invoice
    }
}
```

### Common Misconception
> **Q: Does SRP mean one class can only have one method?**
>
> **A: NO!** One class should have one **responsibility** (logical grouping). A class can have 10-20 methods if they're all related to the same responsibility.

### Benefits of SRP
| Benefit | Description |
|---------|-------------|
| Better Maintenance | Changes in one area only affect specific class |
| Improved Testability | Each class can be tested independently |
| Enhanced Reusability | Classes can be reused in other parts of code |

---

## O - Open/Closed Principle (OCP)

### Definition
> A class should be **open for extension** but **closed for modification**.

New functionality should be added through **inheritance** rather than modifying existing code.

### Why?
- Existing code is already **tested and deployed** in production
- Modifications introduce **additional risk**
- Requires **additional testing effort**

### ❌ Violation Example

```java
class InvoiceDao {
    private Invoice invoice;
    
    public void saveToDB() {
        // Save to SQL database
    }
    
    // New feature added - class modified!
    public void saveToFile() {
        // Save to file
    }
    
    // Another new feature - class modified again!
    public void saveToNoSQL() {
        // Save to NoSQL database
    }
}
```

**Problem:** Every new feature requires modifying the already-tested class.

### ✅ Correct Implementation

```java
// Interface (Abstraction)
interface InvoiceDao {
    void save(Invoice invoice);
}

// Extension for Database
class DatabaseInvoiceDao implements InvoiceDao {
    @Override
    public void save(Invoice invoice) {
        // Save to SQL database
    }
}

// Extension for File
class FileInvoiceDao implements InvoiceDao {
    @Override
    public void save(Invoice invoice) {
        // Save to file
    }
}

// Extension for NoSQL (Future addition)
class NoSQLInvoiceDao implements InvoiceDao {
    @Override
    public void save(Invoice invoice) {
        // Save to NoSQL database
    }
}
```

### Visual Representation
```
         InvoiceDao (Interface)
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
DatabaseDao FileDao  NoSQLDao
```

### Benefits of OCP
- ✅ **Reduced Risk** - No changes to tested code
- ✅ **Better Maintainability** - Each responsibility in separate class
- ✅ **Flexibility** - Easy to add new behavior without touching existing code

---

## L - Liskov Substitution Principle (LSP)

### Definition
> Objects of a **superclass** should be **replaceable** with objects of a **subclass** without breaking the application.

If Class B is a subtype of Class A, we should be able to replace objects of A with B without breaking the program's behavior.

### Key Concept
> **We should only EXTEND the capability of a parent class, NEVER narrow it down.**

### ❌ Violation Example

```java
interface Bike {
    void turnOnEngine();
    void turnOffEngine();
    void accelerate();
    void applyBrake();
}

class Motorcycle implements Bike {
    @Override
    public void turnOnEngine() {
        // Engine ON
    }
    
    @Override
    public void turnOffEngine() {
        // Engine OFF
    }
    
    @Override
    public void accelerate() {
        // Increase speed
    }
    
    @Override
    public void applyBrake() {
        // Decrease speed
    }
}

class Bicycle implements Bike {
    @Override
    public void turnOnEngine() {
        throw new RuntimeException("Bicycle has no engine!"); // ❌ VIOLATION
    }
    
    @Override
    public void turnOffEngine() {
        throw new RuntimeException("Bicycle has no engine!"); // ❌ VIOLATION
    }
    
    @Override
    public void accelerate() {
        // Increase speed
    }
    
    @Override
    public void applyBrake() {
        // Decrease speed
    }
}
```

**Client Code Problem:**
```java
Bike bike = new Motorcycle();
bike.turnOnEngine();  // Works ✅
bike.accelerate();    // Works ✅

// Now replace with Bicycle
Bike bike = new Bicycle();
bike.turnOnEngine();  // THROWS EXCEPTION! ❌
bike.accelerate();    // Works ✅
```

**The code breaks when we substitute Bicycle for Motorcycle!**

### ✅ Correct Implementation

```java
// Separate interface for engine functionality
interface Engine {
    void turnOnEngine();
    void turnOffEngine();
}

// Base interface/abstract class for all bikes
abstract class Bike {
    abstract void accelerate();
    abstract void applyBrake();
}

// Motorcycle has engine + bike capabilities
class Motorcycle extends Bike implements Engine {
    @Override
    public void turnOnEngine() {
        // Engine ON
    }
    
    @Override
    public void turnOffEngine() {
        // Engine OFF
    }
    
    @Override
    public void accelerate() {
        // Increase speed
    }
    
    @Override
    public void applyBrake() {
        // Decrease speed
    }
}

// Bicycle only has bike capabilities
class Bicycle extends Bike {
    @Override
    public void accelerate() {
        // Increase speed
    }
    
    @Override
    public void applyBrake() {
        // Decrease speed
    }
}
```

### Visual Representation
```
    Engine (Interface)          Bike (Abstract)
         │                           │
         │                     ┌─────┴─────┐
         │                     │           │
         ▼                     ▼           ▼
    Motorcycle ◄────────► Motorcycle   Bicycle
    (implements)            (extends)   (extends)
```

### LSP Key Takeaway
> **Always follow the contract!** If an interface defines certain functionality, ALL child classes should properly support that functionality.

---

## I - Interface Segregation Principle (ISP)

### Definition
> Interfaces should be such that clients should **not implement unnecessary functions** that they do not need.

### ❌ Violation Example

```java
interface RestaurantEmployee {
    void takeOrder();
    void serveFood();
    void serveDrinks();
    void cleanKitchen();
    void prepareFood();
    void decideMenu();
}

class Waiter implements RestaurantEmployee {
    @Override
    public void takeOrder() {
        // Takes order - Waiter's job ✅
    }
    
    @Override
    public void serveFood() {
        // Serves food - Waiter's job ✅
    }
    
    @Override
    public void serveDrinks() {
        // Serves drinks - Waiter's job ✅
    }
    
    @Override
    public void cleanKitchen() {
        throw new RuntimeException("Not my job!"); // ❌
    }
    
    @Override
    public void prepareFood() {
        throw new RuntimeException("Not my job!"); // ❌
    }
    
    @Override
    public void decideMenu() {
        throw new RuntimeException("Not my job!"); // ❌
    }
}
```

**Problem:** Waiter is forced to implement methods it will NEVER need.

### ✅ Correct Implementation

```java
// Segregated interfaces
interface ChefTask {
    void prepareFood();
    void decideMenu();
}

interface WaiterTask {
    void takeOrder();
    void serveFood();
    void serveDrinks();
}

interface MaintenanceTask {
    void cleanKitchen();
    void cleanDiningArea();
}

// Classes implement only what they need
class Chef implements ChefTask {
    @Override
    public void prepareFood() {
        // Prepares food
    }
    
    @Override
    public void decideMenu() {
        // Decides menu
    }
}

class Waiter implements WaiterTask {
    @Override
    public void takeOrder() {
        // Takes order
    }
    
    @Override
    public void serveFood() {
        // Serves food
    }
    
    @Override
    public void serveDrinks() {
        // Serves drinks
    }
}
```

### Benefits of ISP
| Benefit | Description |
|---------|-------------|
| No Bloated Classes | Each class implements only interfaces it uses |
| No Forced Dependencies | No dependency on irrelevant functionality |
| Cleaner Design | More maintainable code |

---

## LSP vs ISP - Key Differences

| Aspect | LSP | ISP |
|--------|-----|-----|
| **Focus** | Client-side code behavior | Interface design |
| **Concern** | Objects should be substitutable | Don't force unnecessary implementations |
| **Goal** | Consistent behavior across implementations | Smaller, segregated interfaces |
| **Problem Solved** | Code shouldn't break when switching objects | Child class shouldn't implement unused methods |

---

## D - Dependency Inversion Principle (DIP)

### Definition
> High-level components should **not depend on low-level components** directly. Instead, they should **depend on abstractions**.

### ❌ Violation Example

```java
class WiredKeyboard {
    // Wired keyboard implementation
}

class BluetoothKeyboard {
    // Bluetooth keyboard implementation
}

class MacBook {
    private WiredKeyboard keyboard;  // ❌ Depends on low-level component directly
    
    public MacBook() {
        this.keyboard = new WiredKeyboard();  // Hardcoded!
    }
}
```

**Problem:**
- MacBook directly depends on `WiredKeyboard`
- Cannot dynamically switch to `BluetoothKeyboard`
- Must modify the class to support different keyboards

### ✅ Correct Implementation

```java
// Abstraction
interface Keyboard {
    void type();
}

// Low-level components
class WiredKeyboard implements Keyboard {
    @Override
    public void type() {
        // Wired keyboard typing
    }
}

class BluetoothKeyboard implements Keyboard {
    @Override
    public void type() {
        // Bluetooth keyboard typing
    }
}

// High-level component depends on abstraction
class MacBook {
    private Keyboard keyboard;  // ✅ Depends on abstraction
    
    public MacBook(Keyboard keyboard) {
        this.keyboard = keyboard;  // Injected at runtime
    }
}

// Usage
MacBook macbook1 = new MacBook(new WiredKeyboard());      // With wired
MacBook macbook2 = new MacBook(new BluetoothKeyboard());  // With bluetooth
```

### Visual Representation
```
        MacBook (High-level)
            │
            │ depends on
            ▼
        Keyboard (Abstraction/Interface)
            │
      ┌─────┴─────┐
      │           │
      ▼           ▼
   Wired      Bluetooth
  Keyboard    Keyboard
(Low-level)  (Low-level)
```

### Benefits of DIP
- ✅ **Flexibility** - Easy to swap implementations
- ✅ **Testability** - Can inject mock objects for testing
- ✅ **Loose Coupling** - High-level modules don't depend on low-level details

---

## SOLID Summary Table

| Principle | Letter | Key Idea | Remember As |
|-----------|--------|----------|-------------|
| **S**ingle Responsibility | S | One class = One job | "Do one thing well" |
| **O**pen/Closed | O | Open for extension, closed for modification | "Extend, don't modify" |
| **L**iskov Substitution | L | Subtypes must be substitutable | "Honor the contract" |
| **I**nterface Segregation | I | Many specific interfaces > One general interface | "Don't force unnecessary methods" |
| **D**ependency Inversion | D | Depend on abstractions, not concretions | "Rely on interfaces" |

---

## Quick Reference Checklist

- [ ] Does each class have only one reason to change? (SRP)
- [ ] Can I add new features without modifying existing code? (OCP)
- [ ] Can I substitute child classes without breaking code? (LSP)
- [ ] Are my interfaces small and focused? (ISP)
- [ ] Do high-level modules depend on abstractions? (DIP)