# Factory Pattern & Abstract Factory Pattern - Comprehensive Notes

## Overview

### Pattern Category
- **Type:** Creational Design Pattern
- **Purpose:** Controls and encapsulates object creation logic in one place

---

## The Problem

### Scenario
```
┌─────────────────────────────────────────────────────────────────┐
│                    PROBLEM: SCATTERED OBJECT CREATION            │
└─────────────────────────────────────────────────────────────────┘

                    ┌──────────────┐
                    │    Shape     │ (Interface/Product)
                    │  (Product)   │
                    └──────┬───────┘
                           │
                 ┌─────────┴─────────┐
                 │                   │
                 ▼                   ▼
          ┌──────────┐        ┌──────────┐
          │  Circle  │        │  Square  │
          └──────────┘        └──────────┘
                 ▲                   ▲
                 │                   │
    ┌────────────┼───────────────────┼────────────┐
    │            │                   │            │
┌───────┐   ┌───────┐           ┌───────┐   ┌───────┐
│Class A│   │Class B│           │Class C│   │Class D│
│       │   │       │           │       │   │       │
│new    │   │new    │           │new    │   │new    │
│Circle │   │Circle │           │Square │   │Square │
│()     │   │()     │           │()     │   │()     │
└───────┘   └───────┘           └───────┘   └───────┘

Object creation spread across entire codebase!
```

### Problems

```java
// Class A
Shape shape = new Circle();

// Class B  
Shape shape = new Circle();

// Class C
Shape shape = new Square();

// ... spread across 100+ classes in repository
```

**What if Circle constructor changes?**
```java
// OLD: new Circle()
// NEW: new Circle(radius)  ← Need to update ALL classes!
```

| Problem | Impact |
|---------|--------|
| **Scattered Creation** | Object creation logic spread everywhere |
| **Hard to Maintain** | Changes require touching multiple classes |
| **No Encapsulation** | Creation logic not centralized |
| **Difficult to Modify** | Adding new products is complex |

---

## Factory Pattern Variants

```
┌─────────────────────────────────────────────────────────────────┐
│                    FACTORY PATTERN VARIANTS                     │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  1. SIMPLE FACTORY PATTERN                                      │
│     └── Common in industry, not in books                        │
│     └── Single factory class with if-else logic                 │
│                                                                 │
│  2. FACTORY METHOD PATTERN                                      │
│     └── From GoF book (official)                                │
│     └── Separate factory for each product                       │
│                                                                 │
│  3. ABSTRACT FACTORY PATTERN                                    │
│     └── Factory of factories                                    │
│     └── Creates families of related products                    │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

> **Note:** Both Simple Factory and Factory Method are correct! Simple Factory is widely used in industry, while Factory Method is the "official" pattern from design pattern books.

---

## 1. Simple Factory Pattern

### Concept
> A single factory class with **static method** that uses **if-else conditions** to create appropriate objects.

### UML Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    SIMPLE FACTORY PATTERN                        │
└─────────────────────────────────────────────────────────────────┘

                         ┌──────────────────┐
                         │   ShapeFactory   │
                         ├──────────────────┤
                         │ +createShape()   │ ◄── Static method
                         │   [static]       │     with if-else
                         └────────┬─────────┘
                                  │
                                  │ creates
                                  ▼
                    ┌─────────────────────────┐
                    │    <<interface>>        │
                    │        Shape            │
                    ├─────────────────────────┤
                    │ +draw()                 │
                    └─────────────────────────┘
                               △
                               │ implements
                    ┌──────────┴──────────┐
                    │                     │
             ┌──────────┐          ┌──────────┐
             │  Circle  │          │  Square  │
             ├──────────┤          ├──────────┤
             │ +draw()  │          │ +draw()  │
             └──────────┘          └──────────┘
```

### Implementation

#### Step 1: Product Interface & Concrete Products

```java
// Product Interface
interface Shape {
    void draw();
}

// Concrete Product 1
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }
}

// Concrete Product 2
class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Square");
    }
}
```

#### Step 2: Simple Factory Class

```java
// Shape Type Enum
enum ShapeType {
    CIRCLE, SQUARE
}

// Simple Factory
class ShapeFactory {
    
    // Static method - can be invoked without creating factory object
    public static Shape createShape(ShapeType shapeType) {
        
        if (shapeType == ShapeType.CIRCLE) {
            return new Circle();
        } 
        else if (shapeType == ShapeType.SQUARE) {
            return new Square();
        }
        
        throw new IllegalArgumentException("Unknown shape type");
    }
}
```

#### Step 3: Client Code

```java
public class Main {
    public static void main(String[] args) {
        // Instead of: Shape shape = new Circle();
        // Use factory:
        Shape circle = ShapeFactory.createShape(ShapeType.CIRCLE);
        Shape square = ShapeFactory.createShape(ShapeType.SQUARE);
        
        circle.draw();  // Output: Drawing Circle
        square.draw();  // Output: Drawing Square
    }
}
```

### Advantages & Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| Encapsulates object creation at one place | Violates **Open/Closed Principle** |
| Easy to use and understand | Factory class becomes **bloated** |
| Changes only affect factory class | Violates **Single Responsibility Principle** |

### Why It Violates SOLID?

#### Open/Closed Principle Violation
```java
// Adding new shape requires modifying factory!
public static Shape createShape(ShapeType shapeType) {
    if (shapeType == ShapeType.CIRCLE) {
        return new Circle();
    } 
    else if (shapeType == ShapeType.SQUARE) {
        return new Square();
    }
    // ❌ Must add new else-if for Rectangle!
    else if (shapeType == ShapeType.RECTANGLE) {
        return new Rectangle();
    }
}
```

#### Single Responsibility Violation
```java
// Factory does TWO things:
public static Shape createShape(ShapeType shapeType) {
    // 1️⃣ SELECTION: Decides which object to create
    if (shapeType == ShapeType.CIRCLE) {
        
        // 2️⃣ CONSTRUCTION: Complex creation logic
        Config config = loadConfiguration();
        validateConfig(config);
        initializeResources();
        return new Circle(config);  // Complex creation!
    }
}
```

#### Bloating Problem
```java
class ShapeFactory {
    public static Shape createShape(ShapeType type) {
        if (type == CIRCLE) {
            // 50 lines of complex circle creation logic
            Config config = loadCircleConfig();
            validateCircleConfig(config);
            CircleResources resources = initCircleResources();
            return new Circle(config, resources);
        }
        else if (type == SQUARE) {
            // 50 lines of complex square creation logic
            Config config = loadSquareConfig();
            validateSquareConfig(config);
            SquareResources resources = initSquareResources();
            return new Square(config, resources);
        }
        // ... factory becomes 500+ lines! 😱
    }
}
```

---

## 2. Factory Method Pattern

### Concept
> Create a **separate factory for each product**. Separates **selection logic** from **creation logic**.

### UML Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                   FACTORY METHOD PATTERN                        │
└─────────────────────────────────────────────────────────────────┘

┌──────────────────────┐              ┌──────────────────────┐
│  <<interface>>       │              │   <<interface>>      │
│   ShapeFactory       │              │       Shape          │
├──────────────────────┤              ├──────────────────────┤
│ +createShape():Shape │              │ +draw()              │
└──────────────────────┘              └──────────────────────┘
          △                                     △
          │ implements                          │ implements
    ┌─────┴─────┐                        ┌──────┴──────┐
    │           │                        │             │
┌───────────┐ ┌───────────┐      ┌───────────┐ ┌───────────┐
│  Circle   │ │  Square   │      │  Circle   │ │  Square   │
│  Factory  │ │  Factory  │      │           │ │           │
├───────────┤ ├───────────┤      ├───────────┤ ├───────────┤
│+create    │ │+create    │      │ +draw()   │ │ +draw()   │
│ Shape()   │ │ Shape()   │      └───────────┘ └───────────┘
└─────┬─────┘ └─────┬─────┘            ▲             ▲
      │             │                  │             │
      │   creates   │                  │   creates   │
      └─────────────┴──────────────────┴─────────────┘


                    ┌──────────────────────┐
                    │  ShapeFactoryMethod  │  ◄── Selection Logic
                    ├──────────────────────┤
                    │ +getShapeFactory()   │
                    │   [static]           │
                    └──────────────────────┘
```

### Key Difference from Simple Factory
```
┌─────────────────────────────────────────────────────────────────┐
│           SIMPLE FACTORY vs FACTORY METHOD                      │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIMPLE FACTORY:                                                │
│  ┌─────────────────────────────────────────┐                    │
│  │ ShapeFactory                            │                    │
│  │  • Selection (if-else)    } Combined    │                    │
│  │  • Creation logic         }             │                    │
│  └─────────────────────────────────────────┘                    │
│                                                                 │
│  FACTORY METHOD:                                                │
│  ┌─────────────────────────┐  ┌─────────────────────────┐       │
│  │ ShapeFactoryMethod      │  │ CircleFactory           │       │
│  │  • Selection only       │  │  • Circle creation only │       │
│  └─────────────────────────┘  └─────────────────────────┘       │
│                               ┌─────────────────────────┐       │
│           SEPARATED!          │ SquareFactory           │       │
│                               │  • Square creation only │       │
│                               └─────────────────────────┘       │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Implementation

#### Step 1: Products (Same as before)

```java
interface Shape {
    void draw();
}

class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Circle");
    }
}

class Square implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Square");
    }
}
```

#### Step 2: Factory Interface & Concrete Factories

```java
// Factory Interface
interface ShapeFactory {
    Shape createShape();
}

// Concrete Factory for Circle
class CircleFactory implements ShapeFactory {
    @Override
    public Shape createShape() {
        // All complex circle creation logic here
        System.out.println("Creating Circle with complex logic...");
        return new Circle();
    }
}

// Concrete Factory for Square
class SquareFactory implements ShapeFactory {
    @Override
    public Shape createShape() {
        // All complex square creation logic here
        System.out.println("Creating Square with complex logic...");
        return new Square();
    }
}
```

#### Step 3: Factory Provider (Selection Logic)

```java
class ShapeFactoryProvider {
    
    // Selection logic only - decides which factory to return
    public static ShapeFactory getShapeFactory(ShapeType shapeType) {
        
        if (shapeType == ShapeType.CIRCLE) {
            return new CircleFactory();
        }
        else if (shapeType == ShapeType.SQUARE) {
            return new SquareFactory();
        }
        
        throw new IllegalArgumentException("Unknown shape type");
    }
}
```

#### Step 4: Client Code

```java
public class Main {
    public static void main(String[] args) {
        // Step 1: Get the appropriate factory
        ShapeFactory circleFactory = ShapeFactoryProvider.getShapeFactory(ShapeType.CIRCLE);
        
        // Step 2: Use factory to create shape
        Shape circle = circleFactory.createShape();
        circle.draw();
        
        // For Square
        ShapeFactory squareFactory = ShapeFactoryProvider.getShapeFactory(ShapeType.SQUARE);
        Shape square = squareFactory.createShape();
        square.draw();
    }
}
```

### Advantages & Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|--------------|-----------------|
| **Solves bloating** - each factory handles its own creation | Still violates **OCP in selection** |
| **Solves SRP** - selection & creation separated | More classes to manage |
| Creation logic follows **OCP** - add new factory without changing existing | Slightly more complex |
| Each factory is independently testable | - |

### How It Solves Problems

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROBLEMS SOLVED                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ✅ BLOATING SOLVED:                                            │
│     Complex creation logic for Circle → CircleFactory           │
│     Complex creation logic for Square → SquareFactory           │
│     Each factory stays small and focused                        │
│                                                                  │
│  ✅ SRP SOLVED:                                                 │
│     ShapeFactoryProvider → Only selection                       │
│     CircleFactory → Only circle creation                        │
│     SquareFactory → Only square creation                        │
│                                                                  │
│  ✅ OCP PARTIALLY SOLVED:                                       │
│     Creation: Add RectangleFactory (no change to existing)      │
│     Selection: Still need to add else-if ❌                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Adding New Shape (Rectangle)

```java
// 1. Add new product - ✅ OCP followed
class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("Drawing Rectangle");
    }
}

// 2. Add new factory - ✅ OCP followed (no change to existing factories)
class RectangleFactory implements ShapeFactory {
    @Override
    public Shape createShape() {
        return new Rectangle();
    }
}

// 3. Update selection - ❌ OCP violated (must modify existing code)
class ShapeFactoryProvider {
    public static ShapeFactory getShapeFactory(ShapeType shapeType) {
        if (shapeType == ShapeType.CIRCLE) {
            return new CircleFactory();
        }
        else if (shapeType == ShapeType.SQUARE) {
            return new SquareFactory();
        }
        // ❌ Must add this!
        else if (shapeType == ShapeType.RECTANGLE) {
            return new RectangleFactory();
        }
    }
}
```

---

## Simple Factory vs Factory Method Comparison

| Aspect | Simple Factory | Factory Method |
|--------|----------------|----------------|
| **Structure** | One factory class | One factory per product |
| **Selection & Creation** | Combined | Separated |
| **Bloating** | Gets bloated | Stays clean |
| **SRP** | Violated | Followed |
| **OCP** | Violated | Partially followed |
| **Complexity** | Simple | More classes |
| **Use When** | Simple use cases | Complex creation logic |
| **Industry Usage** | Very common | Official pattern |

---

## 3. Abstract Factory Pattern

### Concept
> **Factory of Factories** - Creates **families of related products** that work together.

### When to Use
- When you have **multiple related products**
- Products need to **work together**
- You want to create **complete families** of objects

### Two Variations

```
┌─────────────────────────────────────────────────────────────────┐
│              ABSTRACT FACTORY VARIATIONS                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  1. Built on SIMPLE FACTORY                                      │
│     └── Common in industry                                       │
│     └── Each sub-factory is a simple factory                     │
│                                                                  │
│  2. Built on FACTORY METHOD                                      │
│     └── From design pattern books                                │
│     └── Groups related products into one factory                 │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Abstract Factory (Built on Simple Factory)

### Scenario
```
CAR = Interior + Exterior (Related products that work together)

Economy Car = Economy Interior + Economy Exterior
Luxury Car = Luxury Interior + Luxury Exterior
```

### UML Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│        ABSTRACT FACTORY (Built on Simple Factory)                │
└─────────────────────────────────────────────────────────────────┘

                    ┌─────────────────────┐
                    │    CarProducer      │  ◄── Abstract Factory
                    ├─────────────────────┤      (Factory of Factories)
                    │ +getFactory(choice) │
                    │   [static]          │
                    └──────────┬──────────┘
                               │
                               │ returns
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────────┐          ┌─────────────────────┐
    │  InteriorFactory    │          │  ExteriorFactory    │
    ├─────────────────────┤          ├─────────────────────┤
    │ +getInterior(type)  │          │ +getExterior(type)  │
    │   [static]          │          │   [static]          │
    └──────────┬──────────┘          └──────────┬──────────┘
               │                                │
               │ creates                        │ creates
               ▼                                ▼
    ┌─────────────────────┐          ┌─────────────────────┐
    │    CarInterior      │          │    CarExterior      │
    └─────────────────────┘          └─────────────────────┘
            △                                 △
            │                                 │
    ┌───────┴───────┐                ┌───────┴───────┐
    │               │                │               │
┌─────────┐   ┌─────────┐      ┌─────────┐   ┌─────────┐
│Economy  │   │Luxury   │      │Economy  │   │Luxury   │
│Interior │   │Interior │      │Exterior │   │Exterior │
└─────────┘   └─────────┘      └─────────┘   └─────────┘
```

### Implementation

#### Step 1: Product Families

```java
// Product Family 1: Car Interior
interface CarInterior {
    void assemble();
}

class EconomyInterior implements CarInterior {
    @Override
    public void assemble() {
        System.out.println("Assembling Economy Interior - Basic fabric seats");
    }
}

class LuxuryInterior implements CarInterior {
    @Override
    public void assemble() {
        System.out.println("Assembling Luxury Interior - Leather seats");
    }
}

// Product Family 2: Car Exterior
interface CarExterior {
    void assemble();
}

class EconomyExterior implements CarExterior {
    @Override
    public void assemble() {
        System.out.println("Assembling Economy Exterior - Standard paint");
    }
}

class LuxuryExterior implements CarExterior {
    @Override
    public void assemble() {
        System.out.println("Assembling Luxury Exterior - Metallic paint");
    }
}
```

#### Step 2: Simple Factories for Each Product

```java
// Simple Factory for Interior
class InteriorFactory {
    public static CarInterior getInterior(String type) {
        if (type.equals("ECONOMY")) {
            return new EconomyInterior();
        } else if (type.equals("LUXURY")) {
            return new LuxuryInterior();
        }
        throw new IllegalArgumentException("Unknown type");
    }
}

// Simple Factory for Exterior
class ExteriorFactory {
    public static CarExterior getExterior(String type) {
        if (type.equals("ECONOMY")) {
            return new EconomyExterior();
        } else if (type.equals("LUXURY")) {
            return new LuxuryExterior();
        }
        throw new IllegalArgumentException("Unknown type");
    }
}
```

#### Step 3: Abstract Factory (Factory of Factories)

```java
class CarProducer {
    
    // Returns the appropriate factory based on choice
    public static Object getFactory(String choice) {
        if (choice.equals("INTERIOR")) {
            return new InteriorFactory();
        } else if (choice.equals("EXTERIOR")) {
            return new ExteriorFactory();
        }
        throw new IllegalArgumentException("Unknown choice");
    }
}
```

#### Step 4: Client Code

```java
public class Main {
    public static void main(String[] args) {
        // Get Interior Factory
        CarInterior interior = InteriorFactory.getInterior("LUXURY");
        interior.assemble();
        // Output: Assembling Luxury Interior - Leather seats
        
        // Get Exterior Factory
        CarExterior exterior = ExteriorFactory.getExterior("LUXURY");
        exterior.assemble();
        // Output: Assembling Luxury Exterior - Metallic paint
    }
}
```

---

## Abstract Factory (Built on Factory Method) - Official Pattern

### Key Concept
> **Each factory produces a COMPLETE FAMILY of products** that work together.

### Difference from Simple Factory Version

```
┌─────────────────────────────────────────────────────────────────┐
│                      KEY DIFFERENCE                              │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  FACTORY METHOD PATTERN:                                         │
│  • CircleFactory → creates Circle only                           │
│  • SquareFactory → creates Square only                           │
│  • ONE factory = ONE product                                     │
│                                                                  │
│  ABSTRACT FACTORY (Factory Method based):                        │
│  • EconomyCarFactory → creates Economy Interior + Exterior       │
│  • LuxuryCarFactory → creates Luxury Interior + Exterior         │
│  • ONE factory = COMPLETE FAMILY of products                     │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### UML Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│        ABSTRACT FACTORY (Built on Factory Method)                │
└─────────────────────────────────────────────────────────────────┘

                    ┌──────────────────────┐
                    │  FactoryProvider     │
                    ├──────────────────────┤
                    │ +getFactory(carType) │
                    └──────────┬───────────┘
                               │
                               │ returns
                               ▼
                    ┌──────────────────────┐
                    │   <<interface>>      │
                    │    CarFactory        │  ◄── Abstract Factory
                    ├──────────────────────┤
                    │ +createInterior()    │      Creates FAMILY
                    │ +createExterior()    │      of products
                    └──────────────────────┘
                               △
                               │ implements
              ┌────────────────┴────────────────┐
              │                                 │
    ┌─────────────────────┐          ┌─────────────────────┐
    │  EconomyCarFactory  │          │  LuxuryCarFactory   │
    ├─────────────────────┤          ├─────────────────────┤
    │ +createInterior()   │          │ +createInterior()   │
    │   → EconomyInterior │          │   → LuxuryInterior  │
    │ +createExterior()   │          │ +createExterior()   │
    │   → EconomyExterior │          │   → LuxuryExterior  │
    └─────────────────────┘          └─────────────────────┘
              │                                 │
              │ creates                         │ creates
              ▼                                 ▼
    ┌─────────────────────┐          ┌─────────────────────┐
    │ Economy Interior    │          │ Luxury Interior     │
    │ Economy Exterior    │          │ Luxury Exterior     │
    │ (work together!)    │          │ (work together!)    │
    └─────────────────────┘          └─────────────────────┘
```

### Implementation

#### Step 1: Product Interfaces & Implementations (Same as before)

```java
// Interior products
interface CarInterior {
    void assemble();
}

class EconomyInterior implements CarInterior {
    public void assemble() {
        System.out.println("Economy Interior - Basic");
    }
}

class LuxuryInterior implements CarInterior {
    public void assemble() {
        System.out.println("Luxury Interior - Premium");
    }
}

// Exterior products
interface CarExterior {
    void assemble();
}

class EconomyExterior implements CarExterior {
    public void assemble() {
        System.out.println("Economy Exterior - Standard");
    }
}

class LuxuryExterior implements CarExterior {
    public void assemble() {
        System.out.println("Luxury Exterior - Premium");
    }
}
```

#### Step 2: Abstract Factory Interface

```java
// Abstract Factory - creates FAMILY of products
interface CarFactory {
    CarInterior createInterior();
    CarExterior createExterior();
    
    // Default method to produce complete vehicle
    default void produceCompleteVehicle() {
        CarInterior interior = createInterior();
        CarExterior exterior = createExterior();
        interior.assemble();
        exterior.assemble();
        System.out.println("Complete vehicle assembled!");
    }
}
```

#### Step 3: Concrete Factories (Each Creates Complete Family)

```java
// Economy Factory - creates all Economy products
class EconomyCarFactory implements CarFactory {
    @Override
    public CarInterior createInterior() {
        return new EconomyInterior();  // Economy Interior
    }
    
    @Override
    public CarExterior createExterior() {
        return new EconomyExterior();  // Economy Exterior
    }
}

// Luxury Factory - creates all Luxury products
class LuxuryCarFactory implements CarFactory {
    @Override
    public CarInterior createInterior() {
        return new LuxuryInterior();   // Luxury Interior
    }
    
    @Override
    public CarExterior createExterior() {
        return new LuxuryExterior();   // Luxury Exterior
    }
}
```

#### Step 4: Factory Provider (Selection)

```java
class FactoryProvider {
    public static CarFactory getFactory(String carType) {
        if (carType.equals("ECONOMY")) {
            return new EconomyCarFactory();
        } else if (carType.equals("LUXURY")) {
            return new LuxuryCarFactory();
        }
        throw new IllegalArgumentException("Unknown car type");
    }
}
```

#### Step 5: Client Code

```java
public class Main {
    public static void main(String[] args) {
        // Get Economy Car Factory
        CarFactory economyFactory = FactoryProvider.getFactory("ECONOMY");
        economyFactory.produceCompleteVehicle();
        // Output:
        // Economy Interior - Basic
        // Economy Exterior - Standard
        // Complete vehicle assembled!
        
        System.out.println("---");
        
        // Get Luxury Car Factory
        CarFactory luxuryFactory = FactoryProvider.getFactory("LUXURY");
        luxuryFactory.produceCompleteVehicle();
        // Output:
        // Luxury Interior - Premium
        // Luxury Exterior - Premium
        // Complete vehicle assembled!
    }
}
```

---

## Complete Comparison

| Aspect | Simple Factory | Factory Method | Abstract Factory |
|--------|----------------|----------------|------------------|
| **Purpose** | Basic object creation | One factory per product | Family of related products |
| **Structure** | Single class with if-else | Interface + multiple factories | Interface creating multiple products |
| **Products** | Single type | Single type | Multiple related types |
| **OCP** | Violated | Partially followed | Partially followed |
| **SRP** | Violated | Followed | Followed |
| **Complexity** | Low | Medium | High |
| **When to Use** | Simple cases | Complex creation logic | Related product families |

---

## Decision Flowchart

```
                        ┌─────────────────────┐
                        │ Need to encapsulate │
                        │ object creation?    │
                        └──────────┬──────────┘
                                   │
                                  YES
                                   │
                        ┌──────────▼──────────┐
                        │ Is creation logic   │
                        │ complex?            │
                        └──────────┬──────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                   NO                            YES
                    │                             │
                    ▼                             ▼
          ┌─────────────────┐          ┌──────────────────┐
          │ SIMPLE FACTORY  │          │ Do you have      │
          │                 │          │ related products │
          └─────────────────┘          │ working together?│
                                       └────────┬─────────┘
                                                │
                                 ┌──────────────┴──────────────┐
                                 │                             │
                                NO                            YES
                                 │                             │
                                 ▼                             ▼
                      ┌─────────────────┐          ┌─────────────────┐
                      │ FACTORY METHOD  │          │ ABSTRACT        │
                      │                 │          │ FACTORY         │
                      └─────────────────┘          └─────────────────┘
```

---

## Quick Summary

```
┌────────────────────────────────────────────────────────────────┐
│                   FACTORY PATTERNS SUMMARY                      │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  SIMPLE FACTORY:                                                │
│  • Single factory class                                         │
│  • If-else to select product                                    │
│  • Quick and easy, common in industry                           │
│                                                                 │
│  FACTORY METHOD:                                                │
│  • One factory per product                                      │
│  • Separates selection from creation                            │
│  • Official pattern from books                                  │
│                                                                 │
│  ABSTRACT FACTORY:                                              │
│  • Factory of factories                                         │
│  • Creates families of related products                         │
│  • Products work together                                       │
│                                                                 │
│  REMEMBER: All are valid! Choose based on your use case.        │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```