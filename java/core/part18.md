# Comprehensive Notes: Exception Handling in Java

---

## 1. What is an Exception?

An **exception** is an event that occurs during the execution of a program that disrupts the normal flow of instructions.

### Key Points:
- A program is a set of instructions executed one by one
- When a certain event occurs during execution, it breaks the normal flow
- This disruption is called an **exception**

---

## 2. Exception Object

When an exception occurs, the **runtime system creates an exception object** containing:

| Information | Description |
|-------------|-------------|
| **Exception Type** | The category/class of exception (e.g., ArithmeticException) |
| **Message** | Explanation of why the exception occurred |
| **Stack Trace** | Path from where error occurred to the starting point |

---

## 3. How Runtime System Handles Exceptions

```
Program Flow:
main() → method1() → method2() → method3() [Exception occurs here]
```

### Exception Propagation Process:
1. Exception occurs in `method3()`
2. Runtime checks: Can `method3()` handle it? → No
3. Goes to caller `method2()`: Can you handle it? → No
4. Goes to caller `method1()`: Can you handle it? → No
5. Goes to caller `main()`: Can you handle it? → No
6. **If no one handles → Program terminates abruptly and prints stack trace**

### Example Code:
```java
public class Main {
    public static void main(String[] args) {
        methodOne();  // Line 6
    }
    
    static void methodOne() {
        methodTwo();  // Line 10
    }
    
    static void methodTwo() {
        methodThree();  // Line 14
    }
    
    static void methodThree() {
        int result = 5 / 0;  // Line 18 - Exception occurs
    }
}
```

### Output (Stack Trace):
```
Exception in thread "main" java.lang.ArithmeticException: / by zero
    at Main.methodThree(Main.java:18)
    at Main.methodTwo(Main.java:14)
    at Main.methodOne(Main.java:10)
    at Main.main(Main.java:6)
```

---

## 4. Exception Hierarchy

```
                    Object
                       │
                   Throwable
                   /        \
              Error        Exception
               │               │
    ┌──────────┴──────┐    ┌──┴──────────────────────┐
    │                 │    │                          │
OutOfMemory    StackOverflow   RuntimeException    Checked Exceptions
   Error          Error        (Unchecked)         (Compile-time)
```

---

## 5. Error vs Exception

| Aspect | Error | Exception |
|--------|-------|-----------|
| **Control** | Cannot be controlled/handled | Can be handled |
| **Cause** | JVM-related issues | Code-related issues |
| **Type** | Unchecked (Runtime) | Can be Checked or Unchecked |
| **Examples** | OutOfMemoryError, StackOverflowError | ArithmeticException, IOException |
| **Should Handle?** | No | Yes |

### Error Examples:

**OutOfMemoryError:**
```java
public class Main {
    public static void main(String[] args) {
        String[] arr = new String[Integer.MAX_VALUE]; // Depletes heap memory
    }
}
```

**StackOverflowError:**
```java
public class Main {
    public static void main(String[] args) {
        recursiveMethod(); // Infinite recursion
    }
    
    static void recursiveMethod() {
        recursiveMethod(); // No break point - fills up stack
    }
}
```

---

## 6. Types of Exceptions

### Two Categories:

| Type | Also Known As | Compiler Behavior | When Occurs |
|------|---------------|-------------------|-------------|
| **Unchecked Exception** | Runtime Exception | Does NOT force handling | During program execution |
| **Checked Exception** | Compile-time Exception | FORCES handling | Compilation fails if not handled |

---

## 7. Runtime Exceptions (Unchecked)

> **Key Point:** Compiler does NOT force you to handle these exceptions.

### Common Runtime Exceptions:

#### 1. ClassCastException
```java
Object val = 0;  // Integer type
String str = (String) val;  // Cannot cast Integer to String
// Output: ClassCastException: Integer cannot be cast to String
```

#### 2. ArithmeticException
```java
int result = 5 / 0;
// Output: ArithmeticException: / by zero
```

#### 3. ArrayIndexOutOfBoundsException
```java
int[] arr = new int[2];  // indices: 0, 1
System.out.println(arr[3]);  // Index 3 doesn't exist
// Output: ArrayIndexOutOfBoundsException
```

#### 4. StringIndexOutOfBoundsException
```java
String value = "Hello";  // indices: 0,1,2,3,4
char c = value.charAt(5);  // Index 5 doesn't exist
// Output: StringIndexOutOfBoundsException
```

#### 5. NullPointerException
```java
String value = null;
char c = value.charAt(0);  // Accessing method on null
// Output: NullPointerException
```

#### 6. NumberFormatException
```java
int num = Integer.parseInt("ABC");  // Cannot convert "ABC" to integer
// Output: NumberFormatException: For input string "ABC"
```

---

## 8. Compile-Time Exceptions (Checked)

> **Key Point:** Compiler FORCES you to handle these. Code won't compile otherwise.

### Common Checked Exceptions:
- ClassNotFoundException
- InterruptedException
- IOException
- FileNotFoundException
- EOFException

### Example - Without Handling (Won't Compile):
```java
public class Main {
    public static void main(String[] args) {
        methodOne();
    }
    
    static void methodOne() {
        throw new ClassNotFoundException("Class not found");
        // ERROR: Unreported exception; must be caught or declared to be thrown
    }
}
```

---

## 9. Five Keywords for Exception Handling

| Keyword | Purpose |
|---------|---------|
| **try** | Contains code that might throw an exception |
| **catch** | Handles the exception |
| **finally** | Always executes (cleanup code) |
| **throw** | Used to throw an exception |
| **throws** | Declares that a method might throw an exception |

---

## 10. Handling Exceptions with `throws`

The `throws` keyword **delegates** exception handling to the caller.

```java
public class Main {
    public static void main(String[] args) throws ClassNotFoundException {
        methodOne();
    }
    
    static void methodOne() throws ClassNotFoundException {
        throw new ClassNotFoundException("Class not found");
    }
}
```

### How `throws` Works:
- Placed after method signature
- Indicates: "This method MIGHT throw this exception"
- Warns the caller to handle it
- Multiple exceptions: `throws Exception1, Exception2`

---

## 11. Handling Exceptions with `try-catch`

This is the **actual handling** of exceptions.

### Basic Syntax:
```java
try {
    // Code that might throw exception
} catch (ExceptionType e) {
    // Handle the exception
}
```

### Example:
```java
public class Main {
    public static void main(String[] args) {
        methodOne();
    }
    
    static void methodOne() {
        try {
            throw new ClassNotFoundException("Not found");
        } catch (ClassNotFoundException e) {
            System.out.println("Exception handled: " + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

---

## 12. Multiple Catch Blocks

### Rule: Catch block can only catch exceptions that CAN be thrown by try block

```java
static void methodOne() throws ClassNotFoundException, InterruptedException {
    // Can throw both exceptions
}

public static void main(String[] args) {
    try {
        methodOne();
    } catch (ClassNotFoundException e) {
        // Handle ClassNotFoundException
    } catch (InterruptedException e) {
        // Handle InterruptedException
    }
}
```

### Important: Order Matters!
```java
// ✅ CORRECT - Specific first, Generic last
try {
    methodOne();
} catch (ClassNotFoundException e) {
    // Specific
} catch (Exception e) {
    // Generic (parent)
}

// ❌ WRONG - Generic catches everything first
try {
    methodOne();
} catch (Exception e) {
    // This catches everything!
} catch (ClassNotFoundException e) {
    // ERROR: Already caught above
}
```

### Catch Multiple Exceptions in One Block:
```java
try {
    methodOne();
} catch (ClassNotFoundException | InterruptedException e) {
    // Handles both exception types
}
```

---

## 13. The `finally` Block

> **Key Point:** `finally` block **ALWAYS** executes, regardless of whether exception occurs or not.

### Valid Combinations:
- `try-catch`
- `try-finally`
- `try-catch-finally`

### Example:
```java
static void methodOne() {
    try {
        System.out.println("Inside try");
        return;  // Even with return, finally executes!
    } finally {
        System.out.println("Inside finally");  // This WILL execute
    }
}
```

### Use Cases for `finally`:
- Closing file streams
- Closing database connections
- Releasing locks
- Logging
- Cleanup operations

### When `finally` Does NOT Execute:
- JVM crashes (OutOfMemoryError, StackOverflowError)
- System shutdown
- Process forcefully killed
- `System.exit()` called

---

## 14. The `throw` Keyword

Used to **explicitly throw an exception**.

### Usage 1: Throw New Exception
```java
if (name.equals("dummy")) {
    throw new Exception("Invalid name");
}
```

### Usage 2: Rethrow Exception
```java
try {
    methodOne();
} catch (ClassNotFoundException e) {
    // Do some logging
    System.out.println("Logging: " + e.getMessage());
    throw e;  // Rethrow to let caller handle
}
```

---

## 15. Custom/User-Defined Exceptions

You can create your own exception classes by extending `Exception` or `RuntimeException`.

### Creating Custom Exception:
```java
public class MyCustomException extends Exception {
    public MyCustomException(String message) {
        super(message);
    }
}
```

### Using Custom Exception:
```java
public class Main {
    public static void main(String[] args) {
        try {
            methodOne();
        } catch (MyCustomException e) {
            System.out.println("Caught: " + e.getMessage());
        }
    }
    
    static void methodOne() throws MyCustomException {
        throw new MyCustomException("Some issue arose");
    }
}
```

---

## 16. Advantages of Exception Handling

### 1. Cleaner Code - Separates Error Handling from Regular Code

**Without Exception Handling:**
```java
int process(int classNumber) {
    if (classNumber > 0 && classNumber <= 12) {
        int students = getStudentCapacity(classNumber);
        if (students != 0) {
            String[] names = new String[students];
            if (names != null && names.length > 0) {
                names[0] = "New Value";
                return 0;  // Success
            } else {
                return -3;  // Error
            }
        } else {
            return -2;  // Error
        }
    } else {
        return -1;  // Error
    }
}
```

**With Exception Handling:**
```java
void process(int classNumber) {
    try {
        int students = getStudentCapacity(classNumber);
        String[] names = new String[students];
        names[0] = "New Value";
    } catch (IndexOutOfBoundsException e) {
        // Handle array issues
    } catch (Exception e) {
        // Handle other issues
    }
}
```

### 2. Other Advantages:
| Advantage | Description |
|-----------|-------------|
| **Program Recovery** | Allows program to continue after handling error |
| **Better Debugging** | Stack trace shows exact location of error |
| **More Information** | Can add custom messages and logging |
| **Security** | Can hide sensitive information in logs |

---

## 17. Disadvantages of Exception Handling

### Exception Handling Can Be Expensive

When stack trace is huge and exception is not handled locally:

```
main() → method1() → method2() → ... → method100() [Exception here]
```

- JVM must check each method in the call stack
- If no one handles, it traverses back through ALL 100 methods
- This is an overhead

### When to Avoid Exception Handling:

**Expensive Approach:**
```java
int divide(int a, int b) {
    try {
        return a / b;
    } catch (ArithmeticException e) {
        return -1;
    }
}
```

**Better Approach:**
```java
int divide(int a, int b) {
    if (b == 0) {
        return -1;
    }
    return a / b;
}
```

> **Tip:** If a simple condition check can prevent the exception, prefer that over try-catch.

---

## 18. Quick Reference Summary

| Concept | Description |
|---------|-------------|
| **Exception** | Event disrupting normal program flow |
| **Error** | JVM issues - cannot/should not handle |
| **Checked Exception** | Must handle at compile time |
| **Unchecked Exception** | Occurs at runtime, handling optional |
| **try** | Block with code that might throw exception |
| **catch** | Block that handles specific exception |
| **finally** | Block that always executes |
| **throw** | Keyword to throw an exception |
| **throws** | Keyword to declare method may throw exception |

---

## 19. Key Interview Points

1. **Difference between Error and Exception**
2. **Difference between Checked and Unchecked Exception**
3. **Difference between `throw` and `throws`**
4. **Why use `finally` block?**
5. **Can we have `try` without `catch`?** - Yes, with `finally`
6. **Order of catch blocks** - Specific to Generic
7. **When does `finally` not execute?**

---