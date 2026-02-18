# Chain of Responsibility Pattern - Detailed Notes

## Overview
- **Category:** Behavioral Design Pattern
- **Focus:** How objects communicate with each other
- **Definition:** A request is passed along a chain of handlers. Each handler decides either to process the request or pass it to the next handler in the chain.

---

## Core Concept

### How Chain of Responsibility Works

```
┌─────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
│ Request │───►│ Handler 1 │───►│ Handler 2 │───►│ Handler 3 │───►│ Handler N │
└─────────┘    └───────────┘    └───────────┘    └───────────┘    └───────────┘
                    │                │                │                │
               Can handle?      Can handle?      Can handle?      Can handle?
                    │                │                │                │
                YES → Process    YES → Process    YES → Process    YES → Process
                 & Return         & Return         & Return         & Return
                    │                │                │                │
                NO → Pass to     NO → Pass to     NO → Pass to     NO → End
                   next            next             next
```

**Key Points:**
- Request **always starts from the beginning** of the chain
- Each handler checks if it **can handle** the request
- If YES → Process and return (no further processing)
- If NO → Pass to the next handler in the chain

---

## Real-World Examples

### Example 1: Logging System

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  DEBUG  │───►│  INFO   │───►│  ERROR  │───►│  FATAL  │
│ Level 1 │    │ Level 2 │    │ Level 3 │    │ Level 4 │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
```

| Log Level | Value | Description |
|-----------|-------|-------------|
| DEBUG | 1 | Detailed debugging information |
| INFO | 2 | General information messages |
| ERROR | 3 | Error conditions |
| FATAL | 4 | Critical failures |

### Example 2: ATM Cash Dispenser

```
User wants to withdraw ₹50

┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ ₹20 Handler  │───►│ ₹50 Handler  │───►│ ₹100 Handler │
└──────────────┘    └──────────────┘    └──────────────┘
       │                   │
  Not enough         Can dispense!
  ₹20 notes          ✅ Process
       │
  Pass to next
```

---

## UML Class Diagram

```
                    ┌─────────────────────────┐
                    │   <<abstract>>          │
                    │     LogProcessor        │
                    │─────────────────────────│
                    │ - nextLogProcessor      │◄─────┐
                    │─────────────────────────│      │ has-a
                    │ + logMessage()          │      │ (reference to
                    │ + write() {abstract}    │──────┘  next handler)
                    └───────────┬─────────────┘
                                │
           ┌────────────────────┼────────────────────┐
           │                    │                    │
           ▼                    ▼                    ▼
    ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
    │ DebugLog    │     │  InfoLog    │     │  ErrorLog   │
    │ Processor   │     │  Processor  │     │  Processor  │
    │─────────────│     │─────────────│     │─────────────│
    │ + write()   │     │ + write()   │     │ + write()   │
    └─────────────┘     └─────────────┘     └─────────────┘
```

### Key Relationships

| Relationship | Description |
|--------------|-------------|
| **IS-A** | Each concrete handler (Debug, Info, Error) IS-A LogProcessor |
| **HAS-A** | Each handler HAS-A reference to the next handler in chain |

---

## Code Implementation

### Step 1: Define Log Levels

```java
public class LogProcessor {
    public static final int DEBUG = 1;
    public static final int INFO = 2;
    public static final int ERROR = 3;
    public static final int FATAL = 4;
}
```

### Step 2: Abstract Handler Class

```java
// Abstract Handler
abstract class LogProcessor {
    
    public static final int DEBUG = 1;
    public static final int INFO = 2;
    public static final int ERROR = 3;
    public static final int FATAL = 4;
    
    // HAS-A relationship with itself (reference to next handler)
    private LogProcessor nextLogProcessor;
    private int level;
    
    // Constructor
    public LogProcessor(int level, LogProcessor nextLogProcessor) {
        this.level = level;
        this.nextLogProcessor = nextLogProcessor;
    }
    
    // Template method to process log
    public void logMessage(int level, String message) {
        // Can I handle this request?
        if (this.level == level) {
            // YES - Process the request
            write(message);
        } else {
            // NO - Pass to next handler if exists
            if (nextLogProcessor != null) {
                nextLogProcessor.logMessage(level, message);
            }
        }
    }
    
    // Abstract method - each handler implements its own write logic
    abstract void write(String message);
}
```

### Step 3: Concrete Handlers

```java
// Debug Log Processor
class DebugLogProcessor extends LogProcessor {
    
    public DebugLogProcessor(int level, LogProcessor nextLogProcessor) {
        super(level, nextLogProcessor);
    }
    
    @Override
    void write(String message) {
        System.out.println("DEBUG: " + message);
    }
}

// Info Log Processor
class InfoLogProcessor extends LogProcessor {
    
    public InfoLogProcessor(int level, LogProcessor nextLogProcessor) {
        super(level, nextLogProcessor);
    }
    
    @Override
    void write(String message) {
        System.out.println("INFO: " + message);
    }
}

// Error Log Processor
class ErrorLogProcessor extends LogProcessor {
    
    public ErrorLogProcessor(int level, LogProcessor nextLogProcessor) {
        super(level, nextLogProcessor);
    }
    
    @Override
    void write(String message) {
        System.out.println("ERROR: " + message);
    }
}

// Fatal Log Processor
class FatalLogProcessor extends LogProcessor {
    
    public FatalLogProcessor(int level, LogProcessor nextLogProcessor) {
        super(level, nextLogProcessor);
    }
    
    @Override
    void write(String message) {
        System.out.println("FATAL: " + message);
    }
}
```

### Step 4: Building the Chain

```java
public class Client {
    public static void main(String[] args) {
        
        // Building chain from BACK to FRONT
        
        // Step 1: Create FATAL processor (end of chain - next is null)
        LogProcessor fatalProcessor = new FatalLogProcessor(
            LogProcessor.FATAL,    // level = 4
            null                   // no next processor (end of chain)
        );
        
        // Step 2: Create ERROR processor (next is FATAL)
        LogProcessor errorProcessor = new ErrorLogProcessor(
            LogProcessor.ERROR,    // level = 3
            fatalProcessor         // next processor
        );
        
        // Step 3: Create INFO processor (next is ERROR)
        LogProcessor infoProcessor = new InfoLogProcessor(
            LogProcessor.INFO,     // level = 2
            errorProcessor         // next processor
        );
        
        // Step 4: Create DEBUG processor (next is INFO) - FRONT of chain
        LogProcessor logger = new DebugLogProcessor(
            LogProcessor.DEBUG,    // level = 1
            infoProcessor          // next processor
        );
        
        // Now the chain is ready:
        // DEBUG → INFO → ERROR → FATAL → null
        
        // Using the chain
        logger.logMessage(LogProcessor.DEBUG, "This is a debug message");
        logger.logMessage(LogProcessor.INFO, "This is an info message");
        logger.logMessage(LogProcessor.ERROR, "This is an error message");
        logger.logMessage(LogProcessor.FATAL, "This is a fatal message");
    }
}
```

---

## Chain Building Visualization

```
BUILD ORDER (Back to Front):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 1: FatalProcessor(level=4, next=null)
        └── End of chain

Step 2: ErrorProcessor(level=3, next=FatalProcessor)
        └── Points to Fatal

Step 3: InfoProcessor(level=2, next=ErrorProcessor)  
        └── Points to Error

Step 4: DebugProcessor(level=1, next=InfoProcessor)
        └── Points to Info (FRONT of chain)

FINAL CHAIN:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

┌───────────┐    ┌───────────┐    ┌───────────┐    ┌───────────┐
│   DEBUG   │───►│   INFO    │───►│   ERROR   │───►│   FATAL   │───► null
│  level=1  │    │  level=2  │    │  level=3  │    │  level=4  │
└───────────┘    └───────────┘    └───────────┘    └───────────┘
     ▲
     │
  logger
(start here)
```

---

## Execution Flow Examples

### Example 1: DEBUG Level Request

```
logger.logMessage(DEBUG, "This is debug message")

┌───────────┐
│   DEBUG   │  Level matches (1 == 1)? ✅ YES
│  level=1  │  → Process & Return
└───────────┘
     │
     ▼
Output: "DEBUG: This is debug message"
```

### Example 2: INFO Level Request

```
logger.logMessage(INFO, "This is info message")

┌───────────┐    ┌───────────┐
│   DEBUG   │───►│   INFO    │  Level matches (2 == 2)? ✅ YES
│  level=1  │    │  level=2  │  → Process & Return
└───────────┘    └───────────┘
     │                │
 1 != 2              ▼
 Pass next      Output: "INFO: This is info message"
```

### Example 3: ERROR Level Request

```
logger.logMessage(ERROR, "This is error message")

┌───────────┐    ┌───────────┐    ┌───────────┐
│   DEBUG   │───►│   INFO    │───►│   ERROR   │  ✅ Match!
│  level=1  │    │  level=2  │    │  level=3  │
└───────────┘    └───────────┘    └───────────┘
     │                │                │
 1 != 3          2 != 3              ▼
 Pass next       Pass next      Output: "ERROR: This is error message"
```

---

## Key Logic in logMessage()

```java
public void logMessage(int level, String message) {
    
    if (this.level == level) {
        // ✅ CAN handle → Process request
        write(message);
    } else {
        // ❌ CANNOT handle → Pass to next
        if (nextLogProcessor != null) {
            nextLogProcessor.logMessage(level, message);
        }
    }
}
```

| Condition | Action |
|-----------|--------|
| `this.level == level` | Can handle → Call `write()` method |
| `this.level != level` | Cannot handle → Pass to `nextLogProcessor` |
| `nextLogProcessor == null` | End of chain → Request cannot be handled |

---

## When to Use Chain of Responsibility

### Identification Hints

Ask yourself these questions:
1. Is there a **chain** of processors/handlers?
2. Does request always go from **start** of the chain?
3. Does each handler check **one by one**?
4. Can processing **stop** at any level when handled?

If YES to these questions → **Chain of Responsibility is the best pattern!**

### Common Use Cases

| Use Case | Example |
|----------|---------|
| **Logging** | Debug → Info → Error → Fatal |
| **ATM Dispenser** | ₹100 → ₹50 → ₹20 → ₹10 |
| **Request Validation** | Auth → Permission → Validation |
| **Event Handling** | UI event bubbling |
| **Middleware** | Express.js middleware chain |
| **Approval Workflow** | Manager → Director → VP → CEO |

---

## Benefits and Drawbacks

### Benefits

| Benefit | Description |
|---------|-------------|
| **Decoupling** | Sender doesn't know which handler will process |
| **Flexibility** | Easy to add/remove handlers in chain |
| **Single Responsibility** | Each handler has one specific job |
| **Open/Closed** | Add new handlers without modifying existing code |

### Drawbacks

| Drawback | Description |
|----------|-------------|
| **No Guarantee** | Request might reach end without being handled |
| **Debugging** | Hard to trace which handler processed the request |
| **Performance** | Long chains may have performance impact |

---

## Summary

```
┌──────────────────────────────────────────────────────────────────┐
│              CHAIN OF RESPONSIBILITY PATTERN                      │
├──────────────────────────────────────────────────────────────────┤
│  WHAT: Pass request through chain of handlers                    │
│                                                                   │
│  HOW:  Request → Handler1 → Handler2 → Handler3 → ... → HandlerN │
│                                                                   │
│  EACH HANDLER:                                                    │
│     • Check: Can I handle this?                                  │
│     • YES → Process & Return                                     │
│     • NO  → Pass to next handler                                 │
├──────────────────────────────────────────────────────────────────┤
│  KEY COMPONENTS:                                                  │
│     • Abstract Handler (with next handler reference)             │
│     • Concrete Handlers (implement processing logic)             │
│     • Client (builds chain & sends request)                      │
├──────────────────────────────────────────────────────────────────┤
│  REMEMBER:                                                        │
│     • Request ALWAYS starts from front of chain                  │
│     • Each handler HAS-A reference to next handler               │
│     • Build chain from BACK to FRONT                             │
└──────────────────────────────────────────────────────────────────┘
```