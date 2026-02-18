# Decorator Design Pattern - Detailed Notes

## 1. Introduction & Use Case Identification

### Core Concept
The Decorator Pattern is used when you have a **base object** and need to add **features/functionality** on top of it dynamically.

### Key Characteristics
- **Stacking Feature**: Add feature upon feature on a base object
- **Layering Effect**: Each decoration adds a new layer
- **Dynamic Addition**: Features can be added at runtime

### Real-World Example: Pizza
```
Base Pizza (Margherita)
    ↓ + Cheese
    ↓ + Mushroom  
    ↓ + Paneer
    ↓ + Veggies
```

**Any combination is possible:**
- Margherita → Cheese → Mushroom
- Margherita → Mushroom → Cheese → Veggies
- Margherita → Paneer

---

## 2. Problem Without Decorator Pattern

### Class Explosion Problem
If we try to implement without decorator pattern, we need to create classes for **all possible combinations**:

```
Margherita
Margherita + Cheese
Margherita + Paneer
Margherita + Mushroom
Margherita + Cheese + Mushroom
Margherita + Cheese + Paneer
Margherita + Paneer + Mushroom
Margherita + Cheese + Paneer + Mushroom
... and so on
```

### Impact of Adding New Feature
When a new feature (e.g., "Veggie") is added:
- All new combinations must be created
- Number of classes grows exponentially
- Maintenance becomes nightmare

**This is called CLASS EXPLOSION**

---

## 3. Decorator Pattern Solution

### Definition
> Decorator Pattern allows you to **add new functionality** to an object **dynamically** without altering the original structure.

### UML Structure

```
┌─────────────────────┐
│    Base Pizza       │ ←── Interface
│  (Interface)        │
│  + description()    │
│  + cost()           │
└─────────────────────┘
         △
         │ implements
    ┌────┴────────────────────┐
    │                         │
┌───┴───┐  ┌───────┐   ┌──────┴──────────┐
│Plain  │  │Farm   │   │ToppingDecorator │←── Abstract Class
│Pizza  │  │House  │   │ (has BasePizza) │
└───────┘  └───────┘   └─────────────────┘
                              △
                              │ extends
              ┌───────────────┼───────────────┐
              │               │               │
         ┌────┴───┐     ┌─────┴────┐    ┌─────┴────┐
         │ Cheese │     │ Mushroom │    │  Paneer  │
         └────────┘     └──────────┘    └──────────┘
```

### Key Relationships
1. **IS-A Relationship**:
    - Margherita IS-A BasePizza
    - Farmhouse IS-A BasePizza
    - Cheese IS-A Decorator
    - Mushroom IS-A Decorator

2. **Important Point**: ToppingDecorator IS ALSO a BasePizza
    - This allows decorated pizza to be decorated again
    - Result of decoration is still a pizza

3. **HAS-A Relationship**:
    - ToppingDecorator HAS-A BasePizza object

---

## 4. Implementation

### Step 1: Base Pizza Interface

```java
public interface BasePizza {
    String description();
    int cost();
}
```

### Step 2: Concrete Base Pizzas

```java
public class PlainPizza implements BasePizza {
    @Override
    public String description() {
        return "Plain Pizza";
    }
    
    @Override
    public int cost() {
        return 200;
    }
}

public class Farmhouse implements BasePizza {
    @Override
    public String description() {
        return "Farmhouse Pizza";
    }
    
    @Override
    public int cost() {
        return 300;
    }
}
```

### Step 3: Topping Decorator (Abstract Class)

```java
public abstract class ToppingDecorator implements BasePizza {
    
    protected BasePizza pizza;  // HAS-A relationship
    
    public ToppingDecorator(BasePizza pizza) {
        this.pizza = pizza;     // Accepts pizza to decorate
    }
}
```

**Why ToppingDecorator implements BasePizza?**
- After decorating a pizza with cheese, result is still a pizza
- This allows further decoration (stacking/layering)

### Step 4: Concrete Decorators

```java
public class CheeseTopping extends ToppingDecorator {
    
    public CheeseTopping(BasePizza pizza) {
        super(pizza);  // Pass pizza to parent
    }
    
    @Override
    public String description() {
        return pizza.description() + " + Extra Cheese";
    }
    
    @Override
    public int cost() {
        return pizza.cost() + 20;  // Base cost + cheese cost
    }
}

public class MushroomTopping extends ToppingDecorator {
    
    public MushroomTopping(BasePizza pizza) {
        super(pizza);
    }
    
    @Override
    public String description() {
        return pizza.description() + " + Mushroom";
    }
    
    @Override
    public int cost() {
        return pizza.cost() + 40;  // Base cost + mushroom cost
    }
}
```

---

## 5. Usage Examples

### Example 1: Simple Plain Pizza
```java
BasePizza pizza1 = new PlainPizza();
System.out.println(pizza1.cost());  // Output: 200
```

### Example 2: Plain Pizza + Cheese
```java
BasePizza pizza2 = new CheeseTopping(new PlainPizza());
System.out.println(pizza2.cost());  // Output: 220 (200 + 20)
```

### Example 3: Farmhouse + Cheese + Mushroom
```java
BasePizza pizza4 = new MushroomTopping(
                      new CheeseTopping(
                          new Farmhouse()
                      )
                   );
System.out.println(pizza4.cost());  // Output: 360 (300 + 20 + 40)
```

### How Cost Calculation Works (Inside → Outside)

```
pizza4.getCost() is called on MushroomTopping
    ↓
MushroomTopping.getCost() = pizza.getCost() + 40
    ↓
pizza is CheeseTopping, so CheeseTopping.getCost() = pizza.getCost() + 20
    ↓
pizza is Farmhouse, so Farmhouse.getCost() = 300
    ↓
Result: 300 + 20 + 40 = 360
```

---

## 6. Advantages of Decorator Pattern

| Without Decorator | With Decorator |
|------------------|----------------|
| Class explosion | Add new decorator class only |
| All combinations needed upfront | Combine at runtime |
| Hard to maintain | Easy to extend |
| Rigid structure | Flexible structure |

### Adding New Feature
If new topping "Veggie" is needed:
```java
public class VeggieTopping extends ToppingDecorator {
    public VeggieTopping(BasePizza pizza) {
        super(pizza);
    }
    
    @Override
    public String description() {
        return pizza.description() + " + Veggies";
    }
    
    @Override
    public int cost() {
        return pizza.cost() + 30;
    }
}
```
**That's all! No other changes needed.**

---

## 7. When to Use Decorator Pattern

### Identification Hints
✅ Base object with multiple optional features
✅ Stacking/Layering behavior needed
✅ Features can be combined in any order
✅ Need to avoid class explosion
✅ Want to add functionality without modifying existing classes

### Common Use Cases
- Pizza with toppings
- Coffee with add-ons (milk, sugar, cream)
- Text formatting (bold, italic, underline)
- I/O streams in Java (BufferedReader, InputStreamReader)
- UI component decoration

---

## 8. Key Takeaways

1. **Decorator = Wrapper** that adds functionality
2. **Decorator IS-A and HAS-A** the component it decorates
3. **Stacking** is achieved because decorated object is same type as original
4. **Open/Closed Principle**: Open for extension, closed for modification
5. **Single Responsibility**: Each decorator handles one feature