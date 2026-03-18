# Java Overview - Notes

---

## 1. What is Java?

**Definition:** Java is a **platform-independent language** and a highly popular **Object-Oriented Programming (OOP)** language.

### Key OOP Features of Java:
- Inheritance
- Polymorphism
- Encapsulation
- Abstraction

### Major Advantage: Portability (WORA)

**WORA = Write Once, Run Anywhere**

- Write a Java program on one device (e.g., mobile)
- Run it on any other device (laptop, desktop, Linux, Mac, Windows)
- This makes Java **highly portable**

---

## 2. Three Main Components of Java

```
┌─────────────────────────────────────────────────────────┐
│                         JDK                             │
│  (Java Development Kit)                                 │
│  ┌───────────────────────────────────────────────────┐  │
│  │                      JRE                          │  │
│  │  (Java Runtime Environment)                       │  │
│  │  ┌─────────────────────────────────────────────┐  │  │
│  │  │                  JVM                        │  │  │
│  │  │  (Java Virtual Machine)                     │  │  │
│  │  │  - JIT Compiler                             │  │  │
│  │  │  - Converts bytecode → machine code         │  │  │
│  │  └─────────────────────────────────────────────┘  │  │
│  │  + Class Libraries                                │  │
│  └───────────────────────────────────────────────────┘  │
│  + Programming Language                                 │
│  + Compiler (javac)                                     │
│  + Debugger                                             │
└─────────────────────────────────────────────────────────┘
```

---

### 2.1 JVM (Java Virtual Machine)

**Definition:** An **abstract machine** (doesn't exist physically - it's software)

#### Java Program Execution Flow:

```
Java Program (.java)
        ↓
    Compiler (javac)
        ↓
  Bytecode (.class)
        ↓
       JVM
        ↓
   Machine Code
        ↓
       CPU
        ↓
     Output
```

#### Key Points about JVM:

| Aspect | Details |
|--------|---------|
| Input | Bytecode (.class file) |
| Output | Machine Code |
| Contains | JIT (Just-In-Time) Compiler |
| Platform | **Platform DEPENDENT** |

#### Why JVM Makes Java Platform Independent:

1. Java code compiles to **bytecode** (not directly to machine code)
2. Bytecode is **universal** - any JVM can read it
3. JVM is platform-dependent (different JVM for Windows, Mac, Linux)
4. But bytecode is **platform-independent**

```
Example:
- Write Java program on Mobile → Get bytecode
- Take bytecode to Linux/Mac/Windows
- That system's JVM reads the bytecode
- Converts to that system's machine code
- Program runs!
```

#### Practical Example:

```bash
# Java source file
Student.java

# Compile to bytecode
javac Student.java

# Creates bytecode file
Student.class  ← This can run on ANY JVM

# Run the bytecode
java Student
```

---

### 2.2 JRE (Java Runtime Environment)

**JRE = JVM + Class Libraries**

#### Components:

1. **JVM** - Executes bytecode
2. **Class Libraries** - Pre-built Java libraries

#### What are Class Libraries?

Pre-written code provided by Java that you can use:

```java
// Using Math library
import java.lang.Math;
Math.abs(-10);  // Returns 10

// Using Arrays library
import java.util.Arrays;
Arrays.sort(myArray);  // Sorts the array
```

**Common Libraries:**
- `java.lang` - Basic language features
- `java.util` - Utilities, collections
- `java.math` - Mathematical operations
- `java.io` - Input/Output operations

#### Why Libraries are Needed at Runtime:

- Your bytecode may use library methods (like `Arrays.sort()`)
- JVM needs the actual library code to execute
- JRE provides both JVM AND the libraries

#### Key Point:

> **With only JRE, you can RUN any Java program, but you CANNOT WRITE/COMPILE code**

---

### 2.3 JDK (Java Development Kit)

**JDK = JRE + Development Tools**

#### Components:

| Component | Purpose |
|-----------|---------|
| JRE | Runtime environment (JVM + Libraries) |
| Programming Language | Java syntax, keywords, etc. |
| Compiler (`javac`) | Converts .java → .class |
| Debugger | Debug your code |
| Other tools | Documentation tools, etc. |

#### Key Point:

> **Download JDK to develop Java applications. You get everything (JRE + JVM included)**

---

### Summary: JVM vs JRE vs JDK

| Feature | JVM | JRE | JDK |
|---------|-----|-----|-----|
| Execute bytecode | ✓ | ✓ | ✓ |
| Class libraries | ✗ | ✓ | ✓ |
| Compile code | ✗ | ✗ | ✓ |
| Write code | ✗ | ✗ | ✓ |
| Debug | ✗ | ✗ | ✓ |
| Platform | Dependent | Dependent | Dependent |

**Remember:**
- Java **code/bytecode** = Platform **Independent**
- JVM, JRE, JDK = Platform **Dependent**

---

## 3. Java Editions

| Edition | Full Name | Purpose |
|---------|-----------|---------|
| **JSE** | Java Standard Edition | Core Java (what we learn as developers) |
| **JEE** | Java Enterprise Edition | Large-scale applications (e-commerce, etc.) |
| **JME** | Java Micro/Mobile Edition | Mobile applications |

### JSE (Java Standard Edition)
- Core Java
- Classes, Objects, Multithreading
- Basic programming features

### JEE (Java Enterprise Edition)
- JSE + Additional APIs
- **Transactional API** (rollback, commit)
- **Servlets, JSP** (JavaServer Pages)
- **Persistence API** (database management)
- Used for: E-commerce, large enterprise applications

### JME (Java Micro Edition)
- APIs for mobile applications
- Resource-constrained environments
- Also called: Jakarta EE (newer name)

> **Note:** As beginners, we work with **JSE (Java Standard Edition)**

---

## 4. Downloading JDK

### Steps:
1. Go to Google
2. Search: "Java 8 download" (or any version)
3. Go to **Oracle Archive Downloads**
4. Select your OS (Windows/Mac/Linux)
5. Download and install

### Verify Installation:
```bash
java -version
```
This shows the installed Java version.

> **Important:** Always download from **Oracle's official website**

---

## 5. First Java Program

### File Structure Rules:

1. **File name = Class name**
    - File: `Employee.java`
    - Class: `public class Employee`

2. **One file can have only ONE public class**

### Class Structure:

```java
public class ClassName {
    // 1. Variables (data members)
    
    // 2. Methods (functions)
    
    // 3. Constructors
    
    // 4. Nested/Inner Classes
}
```

---

### The Main Method

```java
public static void main(String[] args) {
    // Code starts here
}
```

#### Why Each Keyword?

| Keyword | Reason |
|---------|--------|
| `public` | JVM needs to call it from outside the package |
| `static` | JVM can call without creating an object |
| `void` | Returns nothing |
| `main` | JVM looks for this specific name |
| `String[] args` | Command-line arguments |

#### Execution Flow:
```
JVM starts → Looks for main() method → Starts executing from there
```

> **Main method is the STARTING POINT of any Java program**

#### Why Static?

```java
// Without static - need object
ClassName obj = new ClassName();
obj.methodName();

// With static - no object needed
ClassName.methodName();
```

JVM doesn't create an object - it directly calls `ClassName.main()`

---

### Complete Example:

```java
// File: Employee.java

public class Employee {
    
    public static void main(String[] args) {
        
        // This is a single-line comment
        
        /*
         This is a
         multi-line comment
        */
        
        // Variable declaration
        int a = -10;
        
        // Print output
        System.out.println("This is my first program");
        System.out.println("Output of a is: " + a);
    }
}
```

### Comments in Java:

```java
// Single line comment

/*
 Multi-line
 comment
*/
```

---

## 6. Compiling and Running Java Programs

### Step-by-Step:

```bash
# Step 1: You have source file
Employee.java

# Step 2: Compile (creates bytecode)
javac Employee.java

# Step 3: Bytecode file created
Employee.class

# Step 4: Run with JVM
java Employee

# Output:
# This is my first program
# Output of a is: -10
```

### Important Concept:

```bash
# Change code: a = -11 (in .java file)

java Employee
# Output: -10  ← Still old value!

# Why? Because .class file not updated!

# Must recompile:
javac Employee.java

java Employee
# Output: -11  ← Now updated!
```

> **JVM reads .class (bytecode), NOT .java (source code)**

---

## 7. Quick Reference Diagram

```
                    JAVA OVERVIEW
                         │
         ┌───────────────┼───────────────┐
         │               │               │
    Components       Editions        Execution
         │               │               │
    ┌────┴────┐     ┌────┴────┐     ┌────┴────┐
    │         │     │         │     │         │
   JDK       JRE   JSE   JEE   JME  .java    .class
    │         │     │               compile   run
    │    ┌────┴──┐  │                  │       │
    │   JVM  Libs Core              javac    java
    │    │       Java                  │       │
  Tools  │                         bytecode  JVM
         │
    JIT Compiler
```

---

## 8. Key Interview Points

1. **What is Java?**
    - Platform-independent, OOP language
    - WORA: Write Once, Run Anywhere

2. **What makes Java platform-independent?**
    - Bytecode is platform-independent
    - JVM (platform-dependent) interprets bytecode

3. **JVM vs JRE vs JDK?**
    - JVM: Executes bytecode
    - JRE: JVM + Libraries (run programs)
    - JDK: JRE + Dev tools (develop programs)

4. **Why is main() method static?**
    - JVM calls it without creating an object

5. **What is bytecode?**
    - Intermediate code (.class file)
    - Not machine code, not source code
    - Can be run by any JVM

6. **Compilation process:**
    - `.java` → (javac) → `.class` → (java/JVM) → Output

---

## 9. Commands Summary

| Command | Purpose |
|---------|---------|
| `javac FileName.java` | Compile Java source to bytecode |
| `java FileName` | Run bytecode with JVM |
| `java -version` | Check installed Java version |

---

## 10. File Extensions

| Extension | What it is |
|-----------|------------|
| `.java` | Source code (human-readable) |
| `.class` | Bytecode (JVM-readable) |