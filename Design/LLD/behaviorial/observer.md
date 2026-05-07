# Observer Design Pattern - Comprehensive Notes

## Overview

### What is Observer Pattern?
> A **Behavioral Design Pattern** where an object (known as **Observable/Publisher**) maintains a list of dependents (called **Observers/Subscribers**) and **automatically notifies** them whenever there is a change in its state.

### Pattern Category
- **Type:** Behavioral Pattern
- **Purpose:** Establish a one-to-many dependency between objects

---

## Core Concept

```
┌─────────────────────────────────────────────────────────────────┐
│                     OBSERVER PATTERN FLOW                        │
└─────────────────────────────────────────────────────────────────┘

                    ┌──────────────────┐
                    │   OBSERVABLE     │
                    │   (Publisher)    │
                    │                  │
                    │  State: value=1  │
                    │       ↓          │
                    │  State: value=2  │  ← State Changed!
                    └────────┬─────────┘
                             │
                             │ Notifies All
                             │
          ┌──────────────────┼──────────────────┐
          │                  │                  │
          ▼                  ▼                  ▼
    ┌──────────┐      ┌──────────┐      ┌──────────┐
    │ Observer │      │ Observer │      │ Observer │
    │    1     │      │    2     │      │    3     │
    └──────────┘      └──────────┘      └──────────┘
    
    "Hey! State changed. Do whatever you want with this info!"
```

---

## Key Terminology

| Term | Also Known As | Description |
|------|---------------|-------------|
| **Observable** | Publisher, Subject | The object being watched; maintains state and notifies observers |
| **Observer** | Subscriber, Listener | The object watching; receives notifications when state changes |
| **State** | Data | The information maintained by the observable |
| **Notify** | Update, Publish | The action of informing all observers about state change |

---

## Real-World Examples

### 1. Weather Application 🌤️
```
Weather Station (Publisher)
        │
        │ broadcasts updates
        │
    ┌───┴───┬───────────┐
    │       │           │
    ▼       ▼           ▼
   TV     Radio      Mobile
   App    Station      App
```

### 2. Social Media 📱
```
Celebrity Profile (Observable)
        │
        │ posts new content
        │
    ┌───┴───┬───────────┐
    │       │           │
    ▼       ▼           ▼
Follower Follower  Follower
   1        2          3
```

### 3. Other Examples
- **Stock Market Trackers** - Stock price changes notify all trading apps
- **Subscription Services** - Newsletter updates sent to all subscribers
- **Event Systems** - Button click notifies all registered listeners
- **News Agencies** - News updates broadcast to all news channels

---

## Two Models: Push vs Pull

### Common Confusion
> Many engineers get confused because different tutorials show different implementations. **Both Push and Pull models are correct!**

### Quick Comparison

| Aspect | Push Model | Pull Model |
|--------|------------|------------|
| **Who decides data?** | Publisher decides what to send | Observer decides what to fetch |
| **Data flow** | Publisher → Observer | Observer ← Publisher |
| **Observer knows Publisher?** | Not necessarily | Yes, holds reference |
| **Update method** | Has parameters (data) | No parameters |
| **Use when** | Publisher controls visibility | Observer needs flexibility |

---

## Push Model

### Concept
> **Publisher pushes the data** it wants observers to see.

```
Publisher ──────pushes data──────> Observer
           (temperature, humidity)
```

### UML Diagram - Push Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        PUSH MODEL UML                           │
└─────────────────────────────────────────────────────────────────┘

  ┌─────────────────────────┐          ┌─────────────────────────┐
  │    <<interface>>        │          │     <<interface>>       │
  │   WeatherObservable     │          │    WeatherObserver      │
  ├─────────────────────────┤          ├─────────────────────────┤
  │ +addObserver()          │          │ +update(WeatherData)    │◄─── Has parameter!
  │ +removeObserver()       │          └─────────────────────────┘
  │ +notifyObservers()      │                     △
  │ +setWeatherReading()    │                     │ implements
  └─────────────────────────┘           ┌─────────┴─────────┐
             △                          │                   │
             │ implements      ┌────────────────┐  ┌────────────────┐
             │                 │CurrentCondition│  │ForecastDisplay │
  ┌─────────────────────────┐  │    Display     │  │                │
  │    WeatherStation       │  ├────────────────┤  ├────────────────┤
  ├─────────────────────────┤  │+update(data)   │  │+update(data)   │
  │ -observers: List        │  └────────────────┘  └────────────────┘
  │ -weatherData            │
  ├─────────────────────────┤
  │ +addObserver()          │
  │ +removeObserver()       │
  │ +notifyObservers()      │
  │ +setWeatherReading()    │
  └─────────────────────────┘
```

### Implementation - Push Model

#### Step 1: Create Data Class (Optional but Recommended)

```java
// Data class to hold observable data
class WeatherData {
    private float temperature;
    private float humidity;
    
    public WeatherData(float temperature, float humidity) {
        this.temperature = temperature;
        this.humidity = humidity;
    }
    
    // Getters
    public float getTemperature() { return temperature; }
    public float getHumidity() { return humidity; }
}
```

#### Step 2: Create Observable Interface

```java
interface WeatherObservable {
    void addObserver(WeatherObserver observer);
    void removeObserver(WeatherObserver observer);
    void notifyObservers();
    void setWeatherReading(float temp, float humidity);
}
```

#### Step 3: Create Observer Interface (With Parameters)

```java
// PUSH MODEL: update() has parameters
interface WeatherObserver {
    void update(WeatherData data);  // Publisher sends data
}
```

#### Step 4: Implement Concrete Observable

```java
class WeatherStation implements WeatherObservable {
    private List<WeatherObserver> observers;  // List of subscribers
    private WeatherData weatherData;           // Observable state
    
    public WeatherStation() {
        this.observers = new ArrayList<>();    // Initially empty
    }
    
    @Override
    public void addObserver(WeatherObserver observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(WeatherObserver observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        // Iterate and notify each observer
        for (WeatherObserver observer : observers) {
            observer.update(weatherData);  // PUSH: Send data to observer
        }
    }
    
    @Override
    public void setWeatherReading(float temp, float humidity) {
        // Update state
        this.weatherData = new WeatherData(temp, humidity);
        // Notify all observers about the change
        notifyObservers();
    }
}
```

#### Step 5: Implement Concrete Observers

```java
class CurrentConditionDisplay implements WeatherObserver {
    @Override
    public void update(WeatherData data) {
        // Receives data from publisher
        System.out.println("Current Conditions:");
        System.out.println("Temperature: " + data.getTemperature());
        System.out.println("Humidity: " + data.getHumidity());
    }
}

class ForecastDisplay implements WeatherObserver {
    @Override
    public void update(WeatherData data) {
        // Custom logic for forecast
        System.out.println("Forecast based on current data:");
        if (data.getHumidity() > 80) {
            System.out.println("Expect rain!");
        } else {
            System.out.println("Clear skies ahead!");
        }
    }
}
```

#### Step 6: Client Code - Push Model

```java
public class Main {
    public static void main(String[] args) {
        // 1. Create Publisher (Observable)
        WeatherStation weatherStation = new WeatherStation();
        
        // 2. Create Subscribers (Observers)
        WeatherObserver currentDisplay = new CurrentConditionDisplay();
        WeatherObserver forecastDisplay = new ForecastDisplay();
        
        // 3. Register Observers with Publisher
        weatherStation.addObserver(currentDisplay);
        weatherStation.addObserver(forecastDisplay);
        
        // 4. Change State - All observers get notified automatically!
        weatherStation.setWeatherReading(25.5f, 65.0f);
        
        // Output:
        // Current Conditions:
        // Temperature: 25.5
        // Humidity: 65.0
        // Forecast based on current data:
        // Clear skies ahead!
        
        // 5. Can remove observer anytime
        weatherStation.removeObserver(forecastDisplay);
        
        // 6. Now only currentDisplay gets notified
        weatherStation.setWeatherReading(30.0f, 85.0f);
    }
}
```

---

## Pull Model

### Concept
> **Observer holds reference** to Observable and **pulls data** it needs.

```
Observer ──────pulls data──────> Publisher
         (fetches what it needs)
```

### Key Difference from Push Model
```
┌──────────────────────────────────────────────────────────────────┐
│                    PULL MODEL KEY DIFFERENCE                      │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  1. Observer HOLDS REFERENCE to Observable                        │
│  2. update() method has NO PARAMETERS                             │
│  3. Observer FETCHES data using the reference                     │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### UML Diagram - Pull Model

```
┌─────────────────────────────────────────────────────────────────┐
│                        PULL MODEL UML                            │
└─────────────────────────────────────────────────────────────────┘

  ┌─────────────────────────┐          ┌─────────────────────────┐
  │    <<interface>>        │◄─────────│     <<interface>>       │
  │   WeatherObservable     │  has-a   │    WeatherObserver      │
  ├─────────────────────────┤ reference├─────────────────────────┤
  │ +addObserver()          │          │ +update()               │◄─── NO parameter!
  │ +removeObserver()       │          └─────────────────────────┘
  │ +notifyObservers()      │                     △
  │ +setWeatherReading()    │                     │ implements
  │ +getTemperature()       │◄── Getters          │
  │ +getHumidity()          │    for pull  ┌──────┴──────┐
  └─────────────────────────┘              │             │
             △                    ┌────────────────┐ ┌────────────────┐
             │ implements         │CurrentCondition│ │ForecastDisplay │
             │                    │    Display     │ │                │
  ┌─────────────────────────┐     ├────────────────┤ ├────────────────┤
  │    WeatherStation       │     │-observable: ref│ │-observable: ref│
  ├─────────────────────────┤     │+update()       │ │+update()       │
  │ -observers: List        │     └────────────────┘ └────────────────┘
  │ -temperature            │              │                  │
  │ -humidity               │              │    Pulls data    │
  ├─────────────────────────┤              └────────┬─────────┘
  │ +addObserver()          │                       │
  │ +removeObserver()       │◄──────────────────────┘
  │ +notifyObservers()      │
  │ +getTemperature()       │
  │ +getHumidity()          │
  └─────────────────────────┘
```

### Implementation - Pull Model

#### Step 1: Create Observable Interface (With Getters)

```java
interface WeatherObservable {
    void addObserver(WeatherObserver observer);
    void removeObserver(WeatherObserver observer);
    void notifyObservers();
    void setWeatherReading(float temp, float humidity);
    
    // Getters for observers to pull data
    float getTemperature();
    float getHumidity();
}
```

#### Step 2: Create Observer Interface (No Parameters)

```java
// PULL MODEL: update() has NO parameters
interface WeatherObserver {
    void update();  // Just notification, no data passed
}
```

#### Step 3: Implement Concrete Observable

```java
class WeatherStation implements WeatherObservable {
    private List<WeatherObserver> observers;
    private float temperature;  // State maintained directly
    private float humidity;
    
    public WeatherStation() {
        this.observers = new ArrayList<>();
    }
    
    @Override
    public void addObserver(WeatherObserver observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(WeatherObserver observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (WeatherObserver observer : observers) {
            observer.update();  // PULL: Just notify, don't send data
        }
    }
    
    @Override
    public void setWeatherReading(float temp, float humidity) {
        this.temperature = temp;
        this.humidity = humidity;
        notifyObservers();
    }
    
    // Getters for observers to pull data
    @Override
    public float getTemperature() { return temperature; }
    
    @Override
    public float getHumidity() { return humidity; }
}
```

#### Step 4: Implement Concrete Observers (Holds Reference)

```java
class CurrentConditionDisplay implements WeatherObserver {
    private WeatherObservable weatherObservable;  // HOLDS REFERENCE!
    
    public CurrentConditionDisplay(WeatherObservable observable) {
        this.weatherObservable = observable;
        // Auto-register itself
        this.weatherObservable.addObserver(this);
    }
    
    @Override
    public void update() {
        // PULL: Fetch only what it needs
        float temp = weatherObservable.getTemperature();
        float humidity = weatherObservable.getHumidity();
        
        System.out.println("Current Conditions:");
        System.out.println("Temperature: " + temp);
        System.out.println("Humidity: " + humidity);
    }
}

class ForecastDisplay implements WeatherObserver {
    private WeatherObservable weatherObservable;  // HOLDS REFERENCE!
    
    public ForecastDisplay(WeatherObservable observable) {
        this.weatherObservable = observable;
        this.weatherObservable.addObserver(this);
    }
    
    @Override
    public void update() {
        // PULL: Fetch only humidity for forecast logic
        float humidity = weatherObservable.getHumidity();
        
        System.out.println("Forecast:");
        if (humidity > 80) {
            System.out.println("Expect rain!");
        } else {
            System.out.println("Clear skies!");
        }
    }
}
```

#### Step 5: Client Code - Pull Model

```java
public class Main {
    public static void main(String[] args) {
        // 1. Create Publisher (Observable)
        WeatherStation weatherStation = new WeatherStation();
        
        // 2. Create Subscribers - they auto-register themselves!
        //    Pass reference of observable to each observer
        WeatherObserver currentDisplay = new CurrentConditionDisplay(weatherStation);
        WeatherObserver forecastDisplay = new ForecastDisplay(weatherStation);
        
        // Observers already added themselves to the list!
        
        // 3. Change State - All observers get notified
        weatherStation.setWeatherReading(25.5f, 65.0f);
        
        // Output:
        // Current Conditions:
        // Temperature: 25.5
        // Humidity: 65.0
        // Forecast:
        // Clear skies!
        
        // 4. Change State again
        weatherStation.setWeatherReading(30.0f, 85.0f);
        
        // Output:
        // Current Conditions:
        // Temperature: 30.0
        // Humidity: 85.0
        // Forecast:
        // Expect rain!
    }
}
```

---

## Push vs Pull: Side-by-Side Comparison

### Observer Interface

```java
// PUSH MODEL                          // PULL MODEL
interface WeatherObserver {            interface WeatherObserver {
    void update(WeatherData data);         void update();
}                                      }
     ↑                                      ↑
     │                                      │
  Has parameter                         No parameter
  (data pushed)                         (will pull data)
```

### Notify Method

```java
// PUSH MODEL                          // PULL MODEL
void notifyObservers() {               void notifyObservers() {
    for (Observer o : observers) {         for (Observer o : observers) {
        o.update(weatherData);  // Send        o.update();  // Just notify
    }                                      }
}                                      }
```

### Observer Implementation

```java
// PUSH MODEL                          // PULL MODEL
class Display implements Observer {    class Display implements Observer {
                                          private Observable observable;
                                          
    void update(WeatherData data) {       void update() {
        // Data received directly            // Fetch data using reference
        float t = data.getTemperature();     float t = observable.getTemperature();
    }                                      }
}                                      }
```

---

## When to Use Which Model?

| Use Push Model When | Use Pull Model When |
|---------------------|---------------------|
| Publisher controls what data observers see | Observers need different subsets of data |
| All observers need the same data | Observable has lots of data |
| Simpler implementation needed | Observers want flexibility |
| Data is small and specific | Different observers have different needs |

### Decision Flowchart

```
                    ┌─────────────────────┐
                    │ Who should control  │
                    │ what data is seen?  │
                    └──────────┬──────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                 │
              ▼                                 ▼
    ┌─────────────────┐               ┌─────────────────┐
    │    Publisher    │               │    Observer     │
    │  (Same data to  │               │  (Each needs    │
    │   all observers)│               │  different data)│
    └────────┬────────┘               └────────┬────────┘
             │                                  │
             ▼                                  ▼
      ┌─────────────┐                   ┌─────────────┐
      │  USE PUSH   │                   │  USE PULL   │
      │    MODEL    │                   │    MODEL    │
      └─────────────┘                   └─────────────┘
```

---

## Summary Table

| Component | Push Model | Pull Model |
|-----------|------------|------------|
| **Observable Interface** | Standard methods | Standard + Getters |
| **Observer Interface** | `update(Data data)` | `update()` |
| **Observer holds reference?** | No | Yes |
| **Data flow** | Pushed by publisher | Pulled by observer |
| **Flexibility** | Less (fixed data) | More (fetch as needed) |

---

## Benefits of Observer Pattern

| Benefit | Description |
|---------|-------------|
| ✅ **Loose Coupling** | Publishers don't know concrete observer classes |
| ✅ **Dynamic Relationships** | Add/remove observers at runtime |
| ✅ **Broadcast Communication** | One-to-many notification |
| ✅ **Open/Closed Principle** | New observers without modifying publisher |
| ✅ **Single Responsibility** | Subject manages state, observers handle reactions |

---

## SOLID Principles in Observer Pattern

| Principle | How Observer Pattern Follows It |
|-----------|--------------------------------|
| **S** - Single Responsibility | Observable manages state; each Observer handles its own logic |
| **O** - Open/Closed | New observers can be added without modifying Observable |
| **L** - Liskov Substitution | Any Observer implementation can be substituted |
| **I** - Interface Segregation | Clean, minimal interfaces for Observable and Observer |
| **D** - Dependency Inversion | Observable depends on Observer interface, not concrete classes |

---

## Quick Reference

```
┌────────────────────────────────────────────────────────────────┐
│                  OBSERVER PATTERN SUMMARY                       │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  "Define a one-to-many dependency between objects so that      │
│   when one object changes state, all its dependents are        │
│   notified and updated automatically"                          │
│                                                                 │
│  KEY COMPONENTS:                                                │
│  • Observable (Publisher) - maintains state & list of observers│
│  • Observer (Subscriber) - receives notifications              │
│                                                                 │
│  TWO MODELS:                                                    │
│  • Push - Publisher sends data to observers                    │
│  • Pull - Observers fetch data from publisher                  │
│                                                                 │
│  CORE METHODS:                                                  │
│  • addObserver() - subscribe                                   │
│  • removeObserver() - unsubscribe                              │
│  • notifyObservers() - broadcast changes                       │
│  • update() - handle notification                              │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```