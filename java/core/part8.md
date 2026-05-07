# Comprehensive Notes: Java Memory Management

## Introduction

Java Memory Management is a critical topic for:
- Technical interviews
- Real-world application development and performance optimization

**Key Point:** JVM (Java Virtual Machine) is responsible for managing memory in Java.

---

## 1. Types of Memory in Java

JVM divides RAM into **two main types** of memory:

| Stack Memory | Heap Memory |
|--------------|-------------|
| Smaller in size | Larger in size |
| Multiple copies (one per thread) | Single shared copy |
| Stores temporary/local data | Stores objects |

---

## 2. Stack Memory

### What is Stored in Stack?

1. **Temporary/Local Variables**
    - Variables created inside methods with limited scope (within `{}`)
    - Destroyed when scope ends

2. **Separate Memory Block (Frame) for Each Method**
    - Each method gets its own memory frame
    - All variables/references created inside that method are stored in its frame

3. **Primitive Data Types**
    - `int`, `float`, `char`, `boolean`, etc.
    - Actual values are stored directly

4. **References to Heap Objects**
    - Only the reference (memory address) is stored
    - Actual object resides in heap

### Key Characteristics of Stack

| Feature | Description |
|---------|-------------|
| **Thread-specific** | Each thread has its own stack memory |
| **Scope-based visibility** | Variables only visible within their scope |
| **LIFO Order** | Last In, First Out - variables deleted in reverse order of creation |
| **Automatic cleanup** | Variables deleted when scope ends |
| **Error on overflow** | `StackOverflowError` when stack is full |

---

## 3. Heap Memory

### What is Stored in Heap?

- **Objects** created using `new` keyword
- **String Pool** (for string literals)

### Key Characteristics

- Shared across all threads
- Larger than stack
- Managed by Garbage Collector
- `OutOfMemoryError` when heap is full

---

## 4. Practical Example: Memory Allocation

### Sample Code:
```java
public class MemoryManagement {
    public static void main(String[] args) {
        int primitive = 10;                    // Primitive
        Person personObj = new Person();       // Object
        String stringLiteral = "24";           // String literal
        MemoryManagement memObj = new MemoryManagement();
        memObj.memoryManagementTest(personObj);
    }
    
    public void memoryManagementTest(Person personObj) {
        Person personObj2 = personObj;         // Reference to same object
        String stringLiteral2 = "24";          // Same literal from pool
        String stringLiteral3 = new String("24"); // New String object
    }
}
```

### Step-by-Step Memory Allocation:

#### Step 1: Main Method Invoked
```
STACK                          HEAP
┌─────────────────┐           ┌─────────────────┐
│ main() frame    │           │                 │
│                 │           │                 │
└─────────────────┘           └─────────────────┘
```

#### Step 2: Primitive Variable Created
```
STACK                          HEAP
┌─────────────────┐           ┌─────────────────┐
│ main() frame    │           │                 │
│  primitive = 10 │           │                 │
└─────────────────┘           └─────────────────┘
```
*Primitive value stored directly in stack*

#### Step 3: Person Object Created
```
STACK                          HEAP
┌─────────────────┐           ┌─────────────────┐
│ main() frame    │           │  ┌───────────┐  │
│  primitive = 10 │           │  │ Person    │  │
│  personObj ─────────────────┼─►│ Object    │  │
└─────────────────┘           │  └───────────┘  │
                              └─────────────────┘
```
*Object in heap, reference in stack*

#### Step 4: String Literal Created
```
STACK                          HEAP
┌─────────────────┐           ┌─────────────────────────┐
│ main() frame    │           │  ┌───────────┐          │
│  primitive = 10 │           │  │ Person    │          │
│  personObj ─────────────────┼─►│ Object    │          │
│  stringLiteral ─────────────┼──┼───────────┼─┐        │
└─────────────────┘           │  └───────────┘ │        │
                              │  String Pool   ▼        │
                              │  ┌───────────────┐      │
                              │  │     "24"      │      │
                              │  └───────────────┘      │
                              └─────────────────────────┘
```

#### Step 5: MemoryManagement Object & Method Call
```
STACK                          HEAP
┌─────────────────────┐       ┌─────────────────────────┐
│ memoryMgmtTest()    │       │                         │
│  personObj ─────────────┐   │  ┌───────────┐          │
│  personObj2 ────────────┼───┼─►│ Person    │          │
│  stringLiteral2 ────────┼───┼──┼─Object────┼──┐       │
│  stringLiteral3 ────────┼───┼──┼───────────┼─┐│       │
├─────────────────────┤   │   │  └───────────┘ ││       │
│ main() frame        │   │   │                ││       │
│  primitive = 10     │   │   │  ┌───────────┐ ││       │
│  personObj ─────────────┘   │  │ MemMgmt   │ ││       │
│  stringLiteral ─────────────┼──│ Object    │ ││       │
│  memObj ────────────────────┼─►└───────────┘ ││       │
└─────────────────────┘       │                ││       │
                              │  String Pool   ▼│       │
                              │  ┌────────────┐ │       │
                              │  │    "24"    │◄┘       │
                              │  └────────────┘         │
                              │  ┌────────────┐         │
                              │  │ String Obj │         │
                              │  │   "24"     │◄────────┘
                              │  └────────────┘         │
                              └─────────────────────────┘
```

### Memory Deallocation (LIFO Order):

1. **When `memoryManagementTest()` ends:**
    - `stringLiteral3` reference removed
    - `stringLiteral2` reference removed
    - `personObj2` reference removed
    - `personObj` (parameter) reference removed
    - Entire method frame removed

2. **When `main()` ends:**
    - `memObj` reference removed
    - `stringLiteral` reference removed
    - `personObj` reference removed
    - `primitive` removed
    - Main frame removed

3. **Garbage Collector cleans unreferenced objects in heap**

---

## 5. Types of References

### 5.1 Strong Reference (Default)
```java
Person pObj = new Person();
```
- **Behavior:** Object will NOT be garbage collected as long as strong reference exists
- **Most commonly used** in applications

### 5.2 Weak Reference
```java
WeakReference<Person> weakPObj = new WeakReference<>(new Person());
```
- **Behavior:** Object CAN be garbage collected even if weak reference exists
- As soon as GC runs, object is freed
- Returns `null` if accessed after GC

### 5.3 Soft Reference
```java
SoftReference<Person> softPObj = new SoftReference<>(new Person());
```
- **Behavior:** Similar to weak, but GC only collects when memory is urgently needed
- GC may keep it alive if sufficient space exists

### 5.4 Phantom Reference
- Used for cleanup operations before object is collected
- Rarely used in practice

### Comparison Table:

| Reference Type | GC Behavior | Use Case |
|---------------|-------------|----------|
| **Strong** | Never collected while referenced | Normal usage |
| **Weak** | Collected immediately when GC runs | Caching (short-term) |
| **Soft** | Collected only when memory is low | Caching (memory-sensitive) |
| **Phantom** | For pre-collection cleanup | Resource cleanup |

---

## 6. Ways to Make Object Eligible for GC

### Method 1: Nullifying Reference
```java
Person obj = new Person();
obj = null;  // Now eligible for GC
```

### Method 2: Reassigning Reference
```java
Person obj1 = new Person();  // Object A
Person obj2 = new Person();  // Object B
obj1 = obj2;  // Object A now eligible for GC
```

### Method 3: Object Created Inside Method
```java
void method() {
    Person obj = new Person();
}  // obj eligible for GC after method ends
```

---

## 7. Heap Memory Structure

```
┌─────────────────────────────────────────────────────────┐
│                      HEAP MEMORY                        │
├────────────────────────────┬────────────────────────────┤
│      YOUNG GENERATION      │      OLD GENERATION        │
│                            │      (Tenured Space)       │
│  ┌──────┬───────┬───────┐  │                            │
│  │ Eden │  S0   │  S1   │  │   Long-lived objects       │
│  │      │(Surv) │(Surv) │  │   (promoted from Young)    │
│  └──────┴───────┴───────┘  │                            │
├────────────────────────────┴────────────────────────────┤
│                     Minor GC                 Major GC   │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                   METASPACE (Non-Heap)                  │
│   • Class metadata                                      │
│   • Static variables (class variables)                  │
│   • Constants (static final)                            │
└─────────────────────────────────────────────────────────┘
```

### Young Generation Components:

| Component | Purpose |
|-----------|---------|
| **Eden** | Where ALL new objects are created first |
| **S0 (Survivor 0)** | Holds surviving objects after GC |
| **S1 (Survivor 1)** | Alternate survivor space |

### Object Lifecycle Example:

#### Initial State:
```
Eden: [O1] [O2] [O3] [O4] [O5]    S0: []    S1: []    Old: []
```

#### After 1st GC (O2, O5 have no references):
```
Eden: []    S0: [O1(age:1)] [O3(age:1)] [O4(age:1)]    S1: []    Old: []
```
*Mark: O2, O5 | Sweep: Delete O2, O5, Move survivors to S0*

#### New Objects Created + 2nd GC (O4, O7 no references):
```
Eden: []    S0: []    S1: [O1(age:2)] [O3(age:2)] [O6(age:1)]    Old: []
```
*Survivors moved to S1 alternately, age incremented*

#### After 3rd GC (O1 reaches threshold age of 3):
```
Eden: []    S0: [O3(age:3)] [O6(age:2)] [O9(age:1)]    S1: []    Old: [O1]
```
*O1 promoted to Old Generation*

### Key Concepts:

| Concept | Description |
|---------|-------------|
| **Age** | Number of GC cycles an object has survived |
| **Threshold** | Age limit before promotion to Old Gen (configurable) |
| **Promotion** | Moving object from Young to Old Generation |
| **Minor GC** | GC in Young Generation (fast, frequent) |
| **Major GC** | GC in Old Generation (slow, infrequent) |

---

## 8. Metaspace vs PermGen

| Feature | PermGen (Before Java 8) | Metaspace (Java 8+) |
|---------|------------------------|---------------------|
| **Location** | Part of Heap | Outside Heap (Native Memory) |
| **Size** | Fixed, not expandable | Auto-expandable |
| **Error** | `OutOfMemoryError: PermGen space` | `OutOfMemoryError: Metaspace` (rare) |
| **Stores** | Class metadata, constants, static vars | Same |

---

## 9. Garbage Collection Algorithms

### 9.1 Mark and Sweep Algorithm

**Two Phases:**

1. **Mark Phase:**
    - Traverse all objects
    - Mark objects with no references as "eligible for deletion"

2. **Sweep Phase:**
    - Delete marked objects
    - Free up memory

### 9.2 Mark-Sweep-Compact Algorithm

**Three Phases:**

```
Before Compaction:
[Obj1] [FREE] [Obj2] [FREE] [FREE] [Obj3] [FREE]

After Compaction:
[Obj1] [Obj2] [Obj3] [FREE] [FREE] [FREE] [FREE]
```

- **Benefit:** Contiguous free memory for new object allocation
- **Drawback:** Additional time for compaction

---

## 10. Types of Garbage Collectors

### 10.1 Serial GC
```
Application Threads: ████████░░░░░░░░████████
GC Thread (Single):         ▓▓▓▓▓▓▓▓
                            ↑ PAUSE ↑
```
- **Threads:** Single GC thread
- **Pause:** Stop-The-World (all app threads pause)
- **Use Case:** Small applications, single-core machines

### 10.2 Parallel GC (Default in Java 8)
```
Application Threads: ████████░░░░░░████████
GC Threads (Multi):         ▓▓▓▓▓▓
                            ▓▓▓▓▓▓
                            ↑PAUSE↑
```
- **Threads:** Multiple GC threads (based on CPU cores)
- **Pause:** Still Stop-The-World, but shorter
- **Benefit:** Faster cleanup due to parallelism

### 10.3 CMS (Concurrent Mark-Sweep)
```
Application Threads: ████████████████████████
GC Threads:              ▓▓▓▓▓▓▓▓▓▓▓▓
                    (Concurrent - minimal pause)
```
- **Threads:** GC runs concurrently with application
- **Pause:** Minimal (not guaranteed zero)
- **Drawback:** No compaction (memory fragmentation)

### 10.4 G1 (Garbage First) - Default in Java 9+
```
Application Threads: ████████████████████████
GC Threads:              ▓▓▓▓▓▓▓▓▓▓▓▓
                    (Concurrent + Compaction)
```
- **Best of both:** Concurrent + Compaction
- **Goal:** Predictable pause times
- **Benefit:** Better throughput and lower latency

### Comparison Summary:

| GC Type | Threads | Pause Time | Compaction | Best For |
|---------|---------|------------|------------|----------|
| **Serial** | 1 | Long | Yes | Small apps |
| **Parallel** | Multiple | Medium | Yes | Throughput-focused |
| **CMS** | Concurrent | Short | No | Low-latency |
| **G1** | Concurrent | Predictable | Yes | Balanced (default) |

---

## 11. GC and Application Performance

### Key Terminology:

| Term | Definition |
|------|------------|
| **Throughput** | Requests processed per unit time |
| **Latency** | Response time for a request |
| **Pause Time** | Time when application is stopped for GC |

### Impact of GC:

```
High Pause Time → Lower Throughput → Higher Latency
Low Pause Time  → Higher Throughput → Lower Latency
```

### Example:
- **Before optimization:** 1000 requests/minute
- **After reducing GC pause:** 1500 requests/minute (50% improvement)

---

## 12. Important Points to Remember

### About `System.gc()`:
```java
System.gc();  // Request to run GC
```
- Only a **suggestion** to JVM
- **No guarantee** GC will run
- JVM has full control over when to run GC
- This is why Java has **Automatic Memory Management**

### Common Errors:

| Error | Cause |
|-------|-------|
| `StackOverflowError` | Stack memory exhausted (e.g., infinite recursion) |
| `OutOfMemoryError: Java heap space` | Heap memory exhausted |
| `OutOfMemoryError: Metaspace` | Too many classes loaded |

---

## 13. Quick Reference Diagram

```
┌──────────────────────────────────────────────────────────────┐
│                         JVM MEMORY                           │
├─────────────────────────────┬────────────────────────────────┤
│          STACK              │             HEAP               │
│  (Per Thread)               │          (Shared)              │
│                             │                                │
│  ┌───────────────────┐      │   ┌──────────────────────┐     │
│  │ Method Frame 3    │      │   │   Young Generation   │     │
│  │  - local vars     │      │   │  ┌─────┬─────┬─────┐ │     │
│  │  - references ────┼──────┼──►│  │Eden │ S0  │ S1  │ │     │
│  ├───────────────────┤      │   │  └─────┴─────┴─────┘ │     │
│  │ Method Frame 2    │      │   ├──────────────────────┤     │
│  │  - local vars     │      │   │   Old Generation     │     │
│  │  - references ────┼──────┼──►│   (Tenured Space)    │     │
│  ├───────────────────┤      │   └──────────────────────┘     │
│  │ Method Frame 1    │      │                                │
│  │  (main)           │      │   ┌──────────────────────┐     │
│  │  - primitives     │      │   │    String Pool       │     │
│  │  - references ────┼──────┼──►│                      │     │
│  └───────────────────┘      │   └──────────────────────┘     │
│                             │                                │
│         LIFO                │        Garbage Collected       │
└─────────────────────────────┴────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              │   METASPACE (Non-Heap)        │
              │   - Class metadata            │
              │   - Static variables          │
              │   - Constants                 │
              └───────────────────────────────┘
```

---

## 14. Interview Tips

1. **Always mention** JVM manages memory automatically
2. **Explain** difference between Stack and Heap clearly
3. **Know** the object lifecycle (Eden → Survivor → Old)
4. **Understand** why GC is "expensive" (Stop-The-World)
5. **Be familiar** with different GC types and when to use them
6. **Remember** `System.gc()` is just a suggestion, not a command