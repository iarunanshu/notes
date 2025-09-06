# Java Multithreading - Part 1 Study Notes

## 1. Process vs Thread - Core Concepts

### Process
- **Definition**: An instance of a program that is getting executed
- **Key Characteristics**:
  - Has its own resources (memory, threads, etc.)
  - OS allocates resources when process is created
  - Each process is independent - they don't share resources
  - Can run parallelly with other processes

**Example Flow:**
```
test.java → javac test.java → bytecode → java test → JVM creates new process
```

### Thread
- **Definition**: Smallest sequence of instructions executed by CPU independently
- Also known as "lightweight process"
- **Key Characteristics**:
  - Multiple threads can exist within one process
  - When process is created, it starts with one thread (main thread)
  - From main thread, we can create multiple threads for concurrent tasks
  - Threads share some resources within same process

## 2. JVM Memory Architecture

### When Process is Created:
1. New JVM instance is allocated to that process
2. Each JVM instance has its own memory areas

### Memory Areas in JVM:

#### Shared Among Threads:
- **Code Segment**: 
  - Stores compiled bytecode/machine code
  - Read-only
  - All threads share same code segment

- **Data Segment**: 
  - Stores global and static variables
  - Threads can read/modify (needs synchronization)

- **Heap Memory**:
  - Objects created with 'new' keyword stored here
  - Shared among threads of same process
  - NOT shared between different processes
  - Can be configured: `-Xms256m` (min) `-Xmx2g` (max)

#### Thread-Specific (Not Shared):
- **Stack**: 
  - Each thread has its own
  - Manages method calls and local variables

- **Register**:
  - Stores intermediate values during execution
  - Used by JIT compiler for optimization
  - Crucial for context switching

- **Program Counter (PC)**:
  - Points to current instruction being executed
  - Increments after successful execution
  - Points to address in code segment

## 3. Execution Flow

### Complete Flow from Code to Execution:

1. **Compilation**: `javac Main.java` → Generates bytecode
2. **Execution**: `java Main` → Creates process
3. **JVM Instance**: Allocated to process with memory areas
4. **Conversion**: JVM uses interpreter/JIT to convert bytecode → machine code
5. **Thread Creation**: JVM creates required threads (main, thread1, thread2, etc.)
6. **Memory Assignment**: Each thread gets stack, register, PC
7. **Machine Code Storage**: Stored in code segment
8. **CPU Execution**: OS scheduler assigns threads to CPU for execution

## 4. Context Switching

### Definition
Process where CPU switches between different threads/processes

### How it Works:
1. OS allocates time slice to each thread (e.g., 1 second)
2. When time expires, thread saves its state in register
3. OS switches to next thread
4. New thread loads its state from register
5. Continues execution from where it left off

### Single vs Multiple CPUs:
- **Single CPU**: Context switching creates illusion of parallel execution
- **Multiple CPUs**: Actual parallel execution possible
  - If threads ≤ CPU cores: True parallel execution
  - If threads > CPU cores: Context switching still required

## 5. Multithreading Definition & Concepts

### Definition
Allows a program to perform multiple operations simultaneously using threads that share resources but execute independently

### Benefits:
1. **Improved Performance**: Task parallelism
2. **Better Responsiveness**: Faster client responses
3. **Resource Sharing**: Efficient resource utilization

### Challenges:
1. **Concurrency Issues**: 
   - Deadlocks
   - Data inconsistency
   - Need for synchronization (overhead)
2. **Complexity**: Code becomes more complex
3. **Debugging Difficulty**: Hard to test and debug concurrent code

## 6. Multitasking vs Multithreading

| Aspect | Multitasking | Multithreading |
|--------|--------------|----------------|
| Definition | Multiple processes running | Multiple threads within a process |
| Resource Sharing | No sharing between processes | Threads share resources |
| Memory | Each process has own memory | Threads share heap, code, data segments |
| Independence | Completely independent | Share some resources, have some independent |
| Example | Process1, Process2 | Thread1, Thread2 in same process |

## Key Interview Points to Remember:

1. **Process creation**: Happens when you execute `java ClassName`
2. **Main thread**: Automatically created when process starts
3. **JVM instances**: Each process gets its own JVM instance
4. **Memory sharing**: Threads share heap, code segment, data segment but have individual stack, register, PC
5. **Context switching**: Essential for concurrent execution with limited CPU cores
6. **True parallelism**: Only possible when CPU cores ≥ number of threads

## Code Example Referenced:
```java
public class MultithreadingLearning {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName()); // Prints: main
    }
}
```

---