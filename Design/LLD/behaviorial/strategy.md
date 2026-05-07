# Strategy Design Pattern - Comprehensive Notes

## Overview

### What is Strategy Pattern?
> A **Behavioral Design Pattern** that defines multiple algorithms, encapsulates their logic in dedicated classes, and enables changing an algorithm's behavior at **runtime**.

### Pattern Category
- **Type:** Behavioral Pattern
- **Focus:** How objects **interact** or **communicate** with each other

---

## The Problem

### Scenario
Consider a `Vehicle` class hierarchy:

```
         Vehicle
      (Normal Drive)
            │
    ┌───────┼───────┬────────────┐
    │       │       │            │
    ▼       ▼       ▼            ▼
 Sports  OffRoad  Passenger   Goods
 Vehicle Vehicle  Vehicle    Vehicle
(Sports  (Sports  (Normal     (Normal
 Drive)   Drive)   Drive)      Drive)
```

### ❌ Problematic Implementation

```java
class Vehicle {
    public void drive() {
        // Normal drive capability (default)
    }
}

class SportsVehicle extends Vehicle {
    @Override
    public void drive() {
        // Sports drive capability
    }
}

class OffRoadVehicle extends Vehicle {
    @Override
    public void drive() {
        // Sports drive capability (DUPLICATE CODE!)
    }
}

class PassengerVehicle extends Vehicle {
    // Uses parent's normal drive
}

class GoodsVehicle extends Vehicle {
    // Uses parent's normal drive
}
```

### Problems Identified

| Problem | Description |
|---------|-------------|
| **Code Duplication** | Same sports drive functionality present in multiple subclasses (SportsVehicle & OffRoadVehicle) |
| **Tight Coupling** | Cannot change drive functionality without modifying the class |
| **Inflexibility** | Once defined, behavior cannot be changed at runtime |
| **Scalability Issues** | With 10-15 subclasses, duplication becomes severe |

---

## The Solution: Strategy Pattern

### Core Concept
**Separate the varying behavior (algorithm) from the main class and encapsulate it in dedicated strategy classes.**

---

## UML Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      STRATEGY PATTERN                       │
└─────────────────────────────────────────────────────────────┘

    ┌──────────────────┐         ┌──────────────────────┐
    │     Vehicle      │         │   <<interface>>      │
    │    (Context)     │ ───────>│   DriveStrategy      │
    ├──────────────────┤ has-a   ├──────────────────────┤
    │ -driveStrategy   │         │ +drive()             │
    ├──────────────────┤         └──────────────────────┘
    │ +drive()         │                    △
    │ +setStrategy()   │                    │ implements
    └──────────────────┘          ┌─────────┴─────────┐
            △                     │                   │
            │ extends    ┌────────────────┐  ┌────────────────┐
    ┌───────┴───────┐    │  NormalDrive   │  │  SportsDrive   │
    │               │    │  Strategy      │  │  Strategy      │
    ▼               ▼    ├────────────────┤  ├────────────────┤
┌─────────┐  ┌──────────┐│ +drive()       │  │ +drive()       │
│ Sports  │  │ OffRoad  │└────────────────┘  └────────────────┘
│ Vehicle │  │ Vehicle  │
└─────────┘  └──────────┘
```

---

## Implementation

### Step 1: Create the Strategy Interface

```java
// Strategy Interface - defines the algorithm contract
interface DriveStrategy {
    void drive();
}
```

### Step 2: Create Concrete Strategy Classes

```java
// Concrete Strategy 1: Normal Drive
class NormalDriveStrategy implements DriveStrategy {
    @Override
    public void drive() {
        System.out.println("Normal drive capability");
        // Complete algorithm for normal driving
    }
}

// Concrete Strategy 2: Sports Drive
class SportsDriveStrategy implements DriveStrategy {
    @Override
    public void drive() {
        System.out.println("Sports drive capability - High performance mode");
        // Complete algorithm for sports driving
    }
}

// Future: Can easily add more strategies
class OffRoadDriveStrategy implements DriveStrategy {
    @Override
    public void drive() {
        System.out.println("Off-road drive capability - Terrain mode");
        // Complete algorithm for off-road driving
    }
}
```

### Step 3: Create the Context Class (Vehicle)

```java
// Context class - uses strategy
class Vehicle {
    private DriveStrategy driveStrategy;  // Composition over inheritance
    
    // Strategy is injected via constructor
    public Vehicle(DriveStrategy driveStrategy) {
        this.driveStrategy = driveStrategy;
    }
    
    // Can also change strategy at runtime
    public void setDriveStrategy(DriveStrategy driveStrategy) {
        this.driveStrategy = driveStrategy;
    }
    
    public void drive() {
        driveStrategy.drive();  // Delegates to strategy
    }
}
```

### Step 4: Create Concrete Vehicle Classes

```java
// Sports Vehicle - uses Sports Drive Strategy
class SportsVehicle extends Vehicle {
    public SportsVehicle() {
        super(new SportsDriveStrategy());  // Default strategy
    }
}

// Off-Road Vehicle - can also use Sports Drive Strategy
class OffRoadVehicle extends Vehicle {
    public OffRoadVehicle() {
        super(new SportsDriveStrategy());  // Same strategy, NO DUPLICATION!
    }
}

// Passenger Vehicle - uses Normal Drive Strategy
class PassengerVehicle extends Vehicle {
    public PassengerVehicle() {
        super(new NormalDriveStrategy());
    }
}

// Goods Vehicle - uses Normal Drive Strategy
class GoodsVehicle extends Vehicle {
    public GoodsVehicle() {
        super(new NormalDriveStrategy());
    }
}
```

### Step 5: Client Code

```java
public class Main {
    public static void main(String[] args) {
        // Create vehicles with their default strategies
        Vehicle sportsVehicle = new SportsVehicle();
        Vehicle offRoadVehicle = new OffRoadVehicle();
        Vehicle passengerVehicle = new PassengerVehicle();
        
        // Drive with default strategy
        sportsVehicle.drive();      // Output: Sports drive capability
        offRoadVehicle.drive();     // Output: Sports drive capability
        passengerVehicle.drive();   // Output: Normal drive capability
        
        // RUNTIME FLEXIBILITY: Change strategy dynamically!
        passengerVehicle.setDriveStrategy(new SportsDriveStrategy());
        passengerVehicle.drive();   // Output: Sports drive capability
        
        // Create vehicle with custom strategy
        Vehicle customVehicle = new Vehicle(new OffRoadDriveStrategy());
        customVehicle.drive();      // Output: Off-road drive capability
    }
}
```

---

## How Problems Are Solved

| Problem | Solution |
|---------|----------|
| **Code Duplication** | Strategy classes are **reused** across multiple vehicle types |
| **Tight Coupling** | Vehicles depend on **abstraction** (DriveStrategy interface), not concrete implementations |
| **Inflexibility** | Strategy can be **changed at runtime** using setter |
| **Scalability** | New strategies and vehicles can be added **independently** |

---

## Key Components

```
┌─────────────────────────────────────────────────────────────────┐
│                    STRATEGY PATTERN COMPONENTS                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. STRATEGY (Interface)                                         │
│     └── Defines the algorithm contract                           │
│     └── Example: DriveStrategy with drive() method               │
│                                                                  │
│  2. CONCRETE STRATEGIES (Classes)                                │
│     └── Implement the actual algorithms                          │
│     └── Example: NormalDriveStrategy, SportsDriveStrategy        │
│                                                                  │
│  3. CONTEXT (Class)                                              │
│     └── Maintains reference to strategy object                   │
│     └── Delegates algorithm execution to strategy                │
│     └── Example: Vehicle class                                   │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Benefits of Strategy Pattern

| Benefit | Description |
|---------|-------------|
| ✅ **Eliminates Code Duplication** | Algorithms written once, reused everywhere |
| ✅ **Loose Coupling** | Context doesn't know concrete strategy implementations |
| ✅ **Runtime Flexibility** | Can switch algorithms dynamically |
| ✅ **Open/Closed Principle** | Add new strategies without modifying existing code |
| ✅ **Single Responsibility** | Each strategy class handles one algorithm |
| ✅ **Independent Scaling** | Strategies and contexts can scale independently |
| ✅ **Easy Testing** | Each strategy can be unit tested in isolation |

---

## When to Use Strategy Pattern

Use Strategy Pattern when:

- ✅ You have multiple algorithms for a specific task
- ✅ You need to switch between algorithms at runtime
- ✅ You have conditional statements (if-else/switch) selecting different behaviors
- ✅ You want to isolate algorithm implementation details from context
- ✅ Multiple classes differ only in their behavior

---

## Real-World Examples

| Domain | Context | Strategies |
|--------|---------|------------|
| **Payment System** | Order | CreditCard, PayPal, UPI, NetBanking |
| **Sorting** | Sorter | BubbleSort, QuickSort, MergeSort |
| **Compression** | FileCompressor | ZIP, RAR, GZIP |
| **Navigation** | Maps | DrivingRoute, WalkingRoute, CyclingRoute |
| **Authentication** | AuthService | OAuth, JWT, BasicAuth |

### Payment Example

```java
interface PaymentStrategy {
    void pay(int amount);
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via Credit Card");
    }
}

class UPIPayment implements PaymentStrategy {
    public void pay(int amount) {
        System.out.println("Paid " + amount + " via UPI");
    }
}

class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment());
cart.checkout(1000);  // Paid 1000 via Credit Card

cart.setPaymentStrategy(new UPIPayment());
cart.checkout(500);   // Paid 500 via UPI
```

---

## Strategy Pattern vs Other Patterns

| Aspect | Strategy | State | Template Method |
|--------|----------|-------|-----------------|
| **Intent** | Define family of algorithms | Change behavior based on state | Define algorithm skeleton |
| **Change** | Client chooses algorithm | State changes internally | Subclasses fill in steps |
| **Composition** | Uses composition | Uses composition | Uses inheritance |

---

## Quick Summary

```
┌────────────────────────────────────────────────────┐
│              STRATEGY PATTERN SUMMARY              │
├────────────────────────────────────────────────────┤
│                                                    │
│  "Define a family of algorithms, put each of      │
│   them into a separate class, and make their      │
│   objects interchangeable"                        │
│                                                    │
│  KEY POINTS:                                       │
│  • Encapsulates algorithms in separate classes    │
│  • Context delegates to strategy object           │
│  • Algorithms are interchangeable at runtime      │
│  • Follows Open/Closed Principle                  │
│  • Uses Composition over Inheritance              │
│                                                    │
└────────────────────────────────────────────────────┘
```

---

## SOLID Principles in Strategy Pattern

| Principle | How Strategy Pattern Follows It |
|-----------|--------------------------------|
| **S** - Single Responsibility | Each strategy has one responsibility (one algorithm) |
| **O** - Open/Closed | New strategies can be added without modifying existing code |
| **L** - Liskov Substitution | All strategies are substitutable through the interface |
| **I** - Interface Segregation | Strategy interface is focused and minimal |
| **D** - Dependency Inversion | Context depends on abstraction (interface), not concrete strategies |