# State Design Pattern - Detailed Notes

## Overview
- **Category:** Behavioral Design Pattern
- **Definition:** State design pattern allows an object to change its behavior dynamically at runtime whenever there is a change in its internal state.

---

## Core Concept

### When to Identify State Pattern

> **Key Question:** Does the object change its state after performing certain operations?

```
┌─────────────┐   Operation   ┌─────────────┐   Operation   ┌─────────────┐
│   State 1   │ ───────────►  │   State 2   │ ───────────►  │   State 3   │
│  (Behavior) │               │  (Behavior) │               │  (Behavior) │
└─────────────┘               └─────────────┘               └─────────────┘
```

### Real-World Examples

| Product | Possible States |
|---------|-----------------|
| **Traffic Signal** | Red → Green → Yellow → Red |
| **Television** | ON ↔ OFF |
| **Vending Machine** | Idle → Accept Coin → Product Selection → Dispense |
| **Document** | Draft → Review → Approved → Published |
| **Order** | Placed → Shipped → Delivered → Completed |

---

## UML Class Diagram

```
┌─────────────────────────────────────┐
│            Product                   │
│─────────────────────────────────────│
│ - state: ProductState               │◄────────┐
│─────────────────────────────────────│         │
│ + setCurrentState(state)            │         │ has-a
│ + getCurrentState()                 │         │
│ + performAction()                   │         │
└─────────────────────────────────────┘         │
                                                │
        ┌───────────────────────────────────────┘
        │
        ▼
┌─────────────────────────────────────┐
│     <<interface/abstract>>          │
│        ProductState                 │
│─────────────────────────────────────│
│ + operation1(product)               │
│ + operation2(product)               │
│ + operation3(product)               │
└───────────────┬─────────────────────┘
                │
    ┌───────────┼───────────┬───────────┐
    │           │           │           │
    ▼           ▼           ▼           ▼
┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐
│ State1 │ │ State2 │ │ State3 │ │ StateN │
└────────┘ └────────┘ └────────┘ └────────┘
```

### Key Components

| Component | Description |
|-----------|-------------|
| **Product** | Main object that has state (HAS-A relationship) |
| **ProductState** | Interface/Abstract class with all possible operations |
| **Concrete States** | Implement only operations relevant to that state |
| **State Transition** | Each state decides the next state after operation |

---

## Example 1: Traffic Signal (Simple)

### State Diagram

```
    ┌─────────────────────────────────────────────────┐
    │                                                 │
    ▼                                                 │
┌───────────┐         ┌───────────┐         ┌────────┴──┐
│    RED    │ ──────► │   GREEN   │ ──────► │  YELLOW   │
│  State 1  │         │  State 2  │         │  State 3  │
└───────────┘         └───────────┘         └───────────┘
     │                      │                     │
     ▼                      ▼                     ▼
   Stop               Go (Move)            Slow Down
  Behavior            Behavior              Behavior
```

### Implementation

#### Step 1: State Interface

```java
// State Interface
interface TrafficLightState {
    void action(TrafficLight trafficLight);
}
```

#### Step 2: Concrete States

```java
// Red State
class RedState implements TrafficLightState {
    
    @Override
    public void action(TrafficLight signal) {
        // Perform RED behavior
        System.out.println("🔴 Making signal RED - STOP");
        
        // Change to next state
        signal.setState(new GreenState());
    }
}

// Green State
class GreenState implements TrafficLightState {
    
    @Override
    public void action(TrafficLight signal) {
        // Perform GREEN behavior
        System.out.println("🟢 Making signal GREEN - GO");
        
        // Change to next state
        signal.setState(new YellowState());
    }
}

// Yellow State
class YellowState implements TrafficLightState {
    
    @Override
    public void action(TrafficLight signal) {
        // Perform YELLOW behavior
        System.out.println("🟡 Making signal YELLOW - SLOW DOWN");
        
        // Change to next state (back to RED)
        signal.setState(new RedState());
    }
}
```

#### Step 3: Product Class (Traffic Light)

```java
// Product - Traffic Light
class TrafficLight {
    
    // HAS-A relationship with state
    private TrafficLightState state;
    
    // Constructor - Set initial state
    public TrafficLight() {
        this.state = new RedState();  // Initial state is RED
    }
    
    // Setter for state
    public void setState(TrafficLightState state) {
        this.state = state;
    }
    
    // Trigger state action
    public void change() {
        state.action(this);  // Pass 'this' so state can change it
    }
}
```

#### Step 4: Client Code

```java
public class Client {
    public static void main(String[] args) {
        
        // Create traffic light (initial state: RED)
        TrafficLight trafficLight = new TrafficLight();
        
        trafficLight.change();  // RED behavior → changes to GREEN
        // Output: 🔴 Making signal RED - STOP
        
        trafficLight.change();  // GREEN behavior → changes to YELLOW
        // Output: 🟢 Making signal GREEN - GO
        
        trafficLight.change();  // YELLOW behavior → changes to RED
        // Output: 🟡 Making signal YELLOW - SLOW DOWN
        
        trafficLight.change();  // RED behavior → changes to GREEN
        // Output: 🔴 Making signal RED - STOP
        
        // Cycle continues...
    }
}
```

### Execution Flow

```
┌──────────────────────────────────────────────────────────────────┐
│                        EXECUTION FLOW                             │
├──────────────────────────────────────────────────────────────────┤
│  1. new TrafficLight()                                           │
│     └── Initial state = RedState                                 │
│                                                                   │
│  2. trafficLight.change()                                        │
│     └── state.action(this)                                       │
│     └── RedState.action() → Print "RED" → setState(GreenState)   │
│                                                                   │
│  3. trafficLight.change()                                        │
│     └── state.action(this)                                       │
│     └── GreenState.action() → Print "GREEN" → setState(Yellow)   │
│                                                                   │
│  4. trafficLight.change()                                        │
│     └── state.action(this)                                       │
│     └── YellowState.action() → Print "YELLOW" → setState(Red)    │
└──────────────────────────────────────────────────────────────────┘
```

---

## Example 2: Vending Machine (Complex)

### Vending Machine Operations

```
┌─────────────────────────────────────────────────────────────────┐
│                    VENDING MACHINE FLOW                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Step 0: Machine is IDLE                                        │
│     │                                                            │
│     ▼                                                            │
│  Step 1: User clicks "Start Insert Coin" button                 │
│     │    └── Machine becomes active to accept coins             │
│     ▼                                                            │
│  Step 2: User inserts coins                                     │
│     │    └── Multiple coins can be inserted                     │
│     ▼                                                            │
│  Step 3: User clicks "Select Product" button                    │
│     │    └── User enters product code (e.g., 101)               │
│     ▼                                                            │
│  Step 4: Product dispensed OR Cancel/Refund                     │
│     │    └── Collect item and change if any                     │
│     ▼                                                            │
│  Step 5: Machine returns to IDLE                                │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### State Diagram

```
                    ┌──────────────────────────────────────────┐
                    │                                          │
                    ▼              Refund                      │
              ┌──────────┐    ◄────────────────┐              │
              │   IDLE   │                     │              │
              │  STATE   │                     │              │
              └────┬─────┘                     │              │
                   │                           │              │
        Press Insert                           │              │
         Coin Button                           │              │
                   │                           │              │
                   ▼                           │              │
            ┌────────────┐    Refund     ┌─────┴──────┐       │
   ┌───────►│  HAS MONEY │ ─────────────►│            │       │
   │        │   STATE    │               │            │       │
   │        └─────┬──────┘               │            │       │
   │              │                      │            │       │
Insert Coin       │ Select Product       │            │       │
(stay in          │ Button               │            │       │
same state)       │                      │            │       │
                  ▼                      │            │       │
            ┌────────────┐    Refund     │            │       │
            │ SELECTION  │ ─────────────►│    IDLE    │       │
            │   STATE    │               │            │       │
            └─────┬──────┘               │            │       │
                  │                      │            │       │
         Choose Product                  │            │       │
         (sufficient $)                  │            │       │
                  │                      │            │       │
                  ▼                      │            │       │
            ┌────────────┐               │            │       │
            │  DISPENSE  │ ─────────────►│            │       │
            │   STATE    │  Dispense     └────────────┘       │
            └────────────┘  Complete                          │
```

### States and Operations Matrix

| State | Available Operations |
|-------|---------------------|
| **Idle State** | Click Insert Coin Button, Update Inventory |
| **Has Money State** | Insert Coin, Select Product Button, Refund |
| **Selection State** | Choose Product, Get Change, Refund |
| **Dispense State** | Dispense Product |

---

### Implementation

#### Step 1: State Abstract Class

```java
// Abstract State with all possible operations
abstract class VendingMachineState {
    
    // Default implementations - do nothing
    public void clickOnInsertCoinButton(VendingMachine machine) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void clickOnStartProductSelectionButton(VendingMachine machine) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void insertCoin(VendingMachine machine, Coin coin) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void chooseProduct(VendingMachine machine, int codeNumber) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void getChange(int returnExtraAmount) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void dispenseProduct(VendingMachine machine, int codeNumber) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void refundFullMoney(VendingMachine machine) {
        throw new RuntimeException("Operation not allowed in current state");
    }
    
    public void updateInventory(VendingMachine machine, Item item, int codeNumber) {
        throw new RuntimeException("Operation not allowed in current state");
    }
}
```

#### Step 2: Idle State

```java
class IdleState extends VendingMachineState {
    
    @Override
    public void clickOnInsertCoinButton(VendingMachine machine) {
        // Change state to HasMoney
        machine.setVendingMachineState(new HasMoneyState());
        System.out.println("Machine ready to accept coins");
    }
    
    @Override
    public void updateInventory(VendingMachine machine, Item item, int codeNumber) {
        // Add item to inventory
        machine.getInventory().addItem(item, codeNumber);
        System.out.println("Inventory updated at code: " + codeNumber);
    }
}
```

#### Step 3: Has Money State

```java
class HasMoneyState extends VendingMachineState {
    
    @Override
    public void insertCoin(VendingMachine machine, Coin coin) {
        // Accept coin - STAY IN SAME STATE
        machine.getCoinList().add(coin);
        System.out.println("Coin accepted: " + coin);
        // Note: State does NOT change here
    }
    
    @Override
    public void clickOnStartProductSelectionButton(VendingMachine machine) {
        // Change state to Selection
        machine.setVendingMachineState(new SelectionState());
        System.out.println("Moved to product selection");
    }
    
    @Override
    public void refundFullMoney(VendingMachine machine) {
        // Refund all coins
        System.out.println("Refunding: " + machine.getCoinList());
        machine.getCoinList().clear();
        // Change state back to Idle
        machine.setVendingMachineState(new IdleState());
    }
}
```

#### Step 4: Selection State

```java
class SelectionState extends VendingMachineState {
    
    @Override
    public void chooseProduct(VendingMachine machine, int codeNumber) {
        // Get item from inventory
        Item item = machine.getInventory().getItem(codeNumber);
        
        // Calculate total coins inserted
        int paidByUser = calculateTotalCoins(machine.getCoinList());
        int itemPrice = item.getPrice();
        
        // Check if sufficient money
        if (paidByUser < itemPrice) {
            System.out.println("Insufficient amount! Need: " + itemPrice + ", Have: " + paidByUser);
            refundFullMoney(machine);
            throw new RuntimeException("Insufficient funds");
        }
        
        // If extra money, return change
        if (paidByUser > itemPrice) {
            int change = paidByUser - itemPrice;
            getChange(change);
        }
        
        // Move to Dispense State
        machine.setVendingMachineState(new DispenseState(machine, codeNumber));
    }
    
    @Override
    public void getChange(int returnExtraAmount) {
        System.out.println("Returning change: $" + returnExtraAmount);
    }
    
    @Override
    public void refundFullMoney(VendingMachine machine) {
        System.out.println("Full refund processed");
        machine.getCoinList().clear();
        machine.setVendingMachineState(new IdleState());
    }
    
    private int calculateTotalCoins(List<Coin> coins) {
        return coins.stream().mapToInt(Coin::getValue).sum();
    }
}
```

#### Step 5: Dispense State

```java
class DispenseState extends VendingMachineState {
    
    // Constructor automatically dispenses
    public DispenseState(VendingMachine machine, int codeNumber) {
        dispenseProduct(machine, codeNumber);
    }
    
    @Override
    public void dispenseProduct(VendingMachine machine, int codeNumber) {
        // Dispense the product
        Item item = machine.getInventory().getItem(codeNumber);
        System.out.println("🎉 Product dispensed: " + item.getType());
        
        // Update inventory (mark as sold)
        machine.getInventory().markSoldOut(codeNumber);
        
        // Clear coins
        machine.getCoinList().clear();
        
        // Return to Idle state
        machine.setVendingMachineState(new IdleState());
        System.out.println("Machine returned to idle state");
    }
}
```

#### Step 6: Vending Machine (Product)

```java
class VendingMachine {
    
    // HAS-A relationship with state
    private VendingMachineState vendingMachineState;
    private Inventory inventory;
    private List<Coin> coinList;
    
    // Constructor - Set initial state
    public VendingMachine(int inventorySize) {
        this.vendingMachineState = new IdleState();  // Initial state
        this.inventory = new Inventory(inventorySize);
        this.coinList = new ArrayList<>();
    }
    
    // State setter
    public void setVendingMachineState(VendingMachineState state) {
        this.vendingMachineState = state;
    }
    
    // Getters
    public VendingMachineState getVendingMachineState() {
        return vendingMachineState;
    }
    
    public Inventory getInventory() {
        return inventory;
    }
    
    public List<Coin> getCoinList() {
        return coinList;
    }
}
```

### Supporting Classes

```java
// Coin Enum
enum Coin {
    PENNY(1), NICKEL(5), DIME(10), QUARTER(25);
    
    private int value;
    
    Coin(int value) {
        this.value = value;
    }
    
    public int getValue() {
        return value;
    }
}

// Item Type Enum
enum ItemType {
    COKE, PEPSI, JUICE, SODA, WATER
}

// Item Class
class Item {
    private ItemType type;
    private int price;
    
    public Item(ItemType type, int price) {
        this.type = type;
        this.price = price;
    }
    
    // Getters
    public ItemType getType() { return type; }
    public int getPrice() { return price; }
}

// Item Shelf Class
class ItemShelf {
    private int code;
    private Item item;
    private boolean soldOut;
    
    // Constructor, Getters, Setters
}

// Inventory Class
class Inventory {
    private List<ItemShelf> itemShelves;
    
    public Inventory(int size) {
        itemShelves = new ArrayList<>(size);
        for (int i = 0; i < size; i++) {
            itemShelves.add(new ItemShelf(100 + i));
        }
    }
    
    public void addItem(Item item, int code) { /* ... */ }
    public Item getItem(int code) { /* ... */ }
    public void markSoldOut(int code) { /* ... */ }
}
```

### Client Usage

```java
public class Client {
    public static void main(String[] args) {
        
        // 1. Create Vending Machine (Initial: IDLE state)
        VendingMachine machine = new VendingMachine(10);
        System.out.println("State: " + machine.getVendingMachineState());
        // Output: State: IdleState
        
        // 2. Fill up inventory
        fillUpInventory(machine);
        
        // 3. Click Insert Coin Button (IDLE → HAS_MONEY)
        machine.getVendingMachineState().clickOnInsertCoinButton(machine);
        System.out.println("State: " + machine.getVendingMachineState());
        // Output: State: HasMoneyState
        
        // 4. Insert Coins (stays in HAS_MONEY)
        machine.getVendingMachineState().insertCoin(machine, Coin.NICKEL);   // 5
        machine.getVendingMachineState().insertCoin(machine, Coin.QUARTER);  // 25
        // Total: 30 cents
        
        // 5. Click Product Selection (HAS_MONEY → SELECTION)
        machine.getVendingMachineState().clickOnStartProductSelectionButton(machine);
        System.out.println("State: " + machine.getVendingMachineState());
        // Output: State: SelectionState
        
        // 6. Choose Product (SELECTION → DISPENSE → IDLE)
        machine.getVendingMachineState().chooseProduct(machine, 102);
        // Product 102 costs 12 cents
        // Paid: 30 cents
        // Change: 18 cents returned
        // Product dispensed
        // State: IdleState
    }
}
```

---

## Execution Output

```
┌─────────────────────────────────────────────────────────────────┐
│                       EXECUTION OUTPUT                           │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Current State: IdleState                                       │
│  Filling up inventory...                                        │
│  ─────────────────────────────────────                          │
│  Code: 100 | Item: COKE  | Available                            │
│  Code: 101 | Item: PEPSI | Available                            │
│  Code: 102 | Item: JUICE | Available                            │
│  ...                                                             │
│  ─────────────────────────────────────                          │
│                                                                  │
│  Clicking Insert Coin Button...                                 │
│  Machine ready to accept coins                                  │
│  Current State: HasMoneyState                                   │
│                                                                  │
│  Inserting coins...                                             │
│  Coin accepted: NICKEL (5)                                      │
│  Coin accepted: QUARTER (25)                                    │
│                                                                  │
│  Clicking Product Selection Button...                           │
│  Moved to product selection                                     │
│  Current State: SelectionState                                  │
│                                                                  │
│  Choosing product 102...                                        │
│  Product price: $12                                             │
│  Amount paid: $30                                               │
│  Returning change: $18                                          │
│  🎉 Product dispensed: JUICE                                    │
│  Machine returned to idle state                                 │
│  Current State: IdleState                                       │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Implementation Points

| Aspect | Description |
|--------|-------------|
| **Initial State** | Set in constructor (Idle for vending machine, Red for traffic) |
| **State Change** | Each state decides next state after operation |
| **Operations** | Each state implements only relevant operations |
| **Product Reference** | States receive product reference to change its state |
| **HAS-A Relationship** | Product HAS-A State interface/abstract class |

---

## When to Use State Pattern

### Identification Checklist

| Question | If YES → Use State Pattern |
|----------|---------------------------|
| Does the object have multiple states? | ✅ |
| Does behavior change based on state? | ✅ |
| Do operations trigger state transitions? | ✅ |
| Are there rules about valid state transitions? | ✅ |

---

## Summary

```
┌──────────────────────────────────────────────────────────────────┐
│                    STATE DESIGN PATTERN                           │
├──────────────────────────────────────────────────────────────────┤
│  WHAT: Allows object to change behavior based on internal state  │
│                                                                   │
│  WHY:  • Behavior depends on state                               │
│        • Clean separation of state-specific logic                │
│        • Easy to add new states                                  │
│                                                                   │
│  HOW:  1. Create State interface with all operations             │
│        2. Each concrete state implements relevant operations     │
│        3. Product HAS-A State reference                          │
│        4. States change product's state after operations         │
├──────────────────────────────────────────────────────────────────┤
│  KEY POINTS:                                                      │
│        • Each state implements only its relevant operations      │
│        • States receive product reference to change state        │
│        • Initial state set in product constructor                │
│        • State transitions happen inside state classes           │
└──────────────────────────────────────────────────────────────────┘
```