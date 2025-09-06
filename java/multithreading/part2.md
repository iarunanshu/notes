# Java Multithreading - Part 2 Study Notes

## 1. Thread Creation Methods

### Two Ways to Create Threads:
1. **Implementing Runnable Interface**
2. **Extending Thread Class**

### Why Two Different Ways?
- **Java Limitation**: A class can only extend ONE parent class
- **Java Flexibility**: A class can implement MULTIPLE interfaces
- If your class already extends another class, you can still add threading capability via Runnable interface

### Method 1: Implementing Runnable Interface

**Architecture:**
```
Runnable (Functional Interface)
    ↓
Thread class implements Runnable
    ↓
Your class implements Runnable
```

**Steps:**
1. Create a class implementing Runnable
2. Override the `run()` method
3. Create instance of your class
4. Pass instance to Thread constructor
5. Call `start()` on thread object

**Code Example:**
```java
// Step 1: Create class implementing Runnable
class MultithreadingLearning implements Runnable {
    @Override
    public void run() {
        System.out.println("Code executed by thread: " + 
            Thread.currentThread().getName());
    }
}

// Usage
Runnable runnableObj = new MultithreadingLearning();
Thread thread = new Thread(runnableObj);
thread.start(); // Internally calls run()
```

**Lambda Alternative:**
```java
Thread thread = new Thread(() -> {
    System.out.println("Code executed by thread: " + 
        Thread.currentThread().getName());
});
thread.start();
```

### Method 2: Extending Thread Class

**Steps:**
1. Create class extending Thread
2. Override `run()` method
3. Create instance of your class
4. Call `start()` directly on instance

**Code Example:**
```java
class MultithreadingLearning extends Thread {
    @Override
    public void run() {
        System.out.println("Code executed by thread: " + 
            Thread.currentThread().getName());
    }
}

// Usage
MultithreadingLearning thread = new MultithreadingLearning();
thread.start();
```

### Industry Preference
- **Runnable Interface is preferred** because:
    - Allows class to extend another class
    - More flexible design
    - Better separation of concerns

## 2. Thread Lifecycle

### Thread States:

```
NEW → RUNNABLE → TERMINATED
         ↓↑
    BLOCKED/WAITING/TIMED_WAITING
```

### State Descriptions:

1. **NEW**
    - Thread created but not started
    - Just an object in memory

2. **RUNNABLE**
    - Thread ready to run or running
    - Actually includes two sub-states:
        - **Runnable**: Waiting for CPU time
        - **Running**: Currently executing (has CPU)

3. **BLOCKED**
    - Waiting for I/O operations
    - Waiting to acquire lock held by another thread
    - **Releases all monitor locks**

4. **WAITING**
    - Thread calls `wait()` method
    - Needs `notify()` or `notifyAll()` to resume
    - **Releases all monitor locks**

5. **TIMED_WAITING**
    - Thread calls `sleep(time)` or `wait(time)`
    - Automatically returns to RUNNABLE after time expires
    - **Does NOT release monitor locks**

6. **TERMINATED**
    - Thread completed execution
    - Cannot be restarted

## 3. Monitor Locks

### Definition
- Ensures only one thread enters synchronized section at a time
- Each object in Java has one monitor lock

### Key Concepts:
- **Synchronized methods/blocks** use monitor locks
- Lock is on the **object**, not the method
- Different objects have different locks

### Example:
```java
class MonitorLockExample {
    // Synchronized method - locks on 'this' object
    synchronized void task1() {
        System.out.println("Inside task1");
        Thread.sleep(10000);
    }
    
    // Synchronized block - also locks on 'this' object
    void task2() {
        System.out.println("Before synchronized");
        synchronized(this) {
            System.out.println("Inside synchronized");
        }
    }
    
    // Non-synchronized method
    void task3() {
        System.out.println("Task 3 - no lock needed");
    }
}
```

### Important Points:
- If Thread1 holds lock on an object, Thread2 must wait for same object's lock
- Threads working on different objects don't interfere
- Lock is released when exiting synchronized block/method

## 4. Inter-Thread Communication

### Key Methods:

1. **wait()**
    - Makes thread wait until notified
    - Must be called inside synchronized block
    - Releases monitor lock

2. **notify()**
    - Wakes up one waiting thread
    - Must be called inside synchronized block

3. **notifyAll()**
    - Wakes up all waiting threads
    - Must be called inside synchronized block

### Example: Producer-Consumer Pattern
```java
class SharedResource {
    private boolean itemAvailable = false;
    
    synchronized void addItem() {
        itemAvailable = true;
        notifyAll(); // Wake up waiting consumers
    }
    
    synchronized void consumeItem() {
        while (!itemAvailable) { // while loop prevents spurious wakeup
            try {
                wait(); // Wait until item available
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        itemAvailable = false;
        System.out.println("Item consumed");
    }
}
```

### Spurious Wakeup
- Threads can wake up without being notified (system noise)
- **Solution**: Use `while` loop instead of `if` to recheck condition

## 5. Context Switching

### Definition
- CPU switching between different threads
- Creates illusion of parallel execution on single CPU

### How it Works:
1. OS allocates time slice to each thread
2. Thread saves state in register when time expires
3. Next thread loads its state from register
4. Continues from where it left off

### Single vs Multiple CPUs:
- **Single CPU**: Context switching required for concurrency
- **Multiple CPUs**: True parallel execution possible
    - If threads ≤ cores: True parallelism
    - If threads > cores: Context switching still needed

## 6. Thread Methods Summary

| Method | Purpose | Notes |
|--------|---------|-------|
| `start()` | Starts thread execution | Moves thread to RUNNABLE |
| `run()` | Contains thread's task | Called by start() internally |
| `sleep(ms)` | Pauses thread for specified time | TIMED_WAITING, keeps locks |
| `wait()` | Makes thread wait | WAITING, releases locks |
| `notify()` | Wakes one waiting thread | Must be in synchronized |
| `notifyAll()` | Wakes all waiting threads | Must be in synchronized |

## 7. Best Practices

1. **Prefer Runnable over extending Thread**
2. **Always use synchronized for shared resources**
3. **Use while loops with wait() to handle spurious wakeups**
4. **Release resources properly in finally blocks**
5. **Avoid deprecated methods (stop(), suspend())**

## 8. Assignment: Producer-Consumer Problem

### Requirements:
- Producer and Consumer share fixed-size buffer (queue)
- Producer generates data and adds to buffer
- Consumer removes and processes data from buffer
- **Producer must wait if buffer is full**
- **Consumer must wait if buffer is empty**

### Key Points to Implement:
- Use synchronized methods
- Use wait() when buffer full (producer) or empty (consumer)
- Use notifyAll() after adding/removing items
- Handle queue size limits

### Skeleton Structure:
```java
class Buffer {
    private Queue<Integer> queue;
    private int maxSize;
    
    synchronized void produce(int item) {
        while (queue.size() == maxSize) {
            wait(); // Buffer full, wait
        }
        queue.add(item);
        notifyAll(); // Notify waiting consumers
    }
    
    synchronized int consume() {
        while (queue.isEmpty()) {
            wait(); // Buffer empty, wait
        }
        int item = queue.remove();
        notifyAll(); // Notify waiting producers
        return item;
    }
}
```

## Key Interview Topics Covered:
1. ✅ Two ways to create threads and why
2. ✅ Thread lifecycle and states
3. ✅ Monitor locks and synchronization
4. ✅ wait(), notify(), notifyAll()
5. ✅ Context switching
6. ✅ Producer-Consumer problem pattern
