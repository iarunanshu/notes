# Java Collections Framework - Part 3: Deque, List & Their Implementations

## 1. Deque Interface (Double-Ended Queue)

### Overview
- **Deque** = Double-Ended Queue
- Extends Queue interface
- Addition and removal can be done from **both ends**
- Can be used to implement both Queue and Stack

### Deque Structure
```
     ← Add/Remove              Add/Remove →
Front [Element1] [Element2] [Element3] [Element4] Rear
     ← First/Head              Last/Tail →
```

### Deque Methods (12 New Methods)

| Operation | First (Head) | | Last (Tail) | |
|-----------|-------------|---|-------------|---|
| | **Throws Exception** | **Returns Special Value** | **Throws Exception** | **Returns Special Value** |
| **Insert** | `addFirst(e)` | `offerFirst(e)` | `addLast(e)` | `offerLast(e)` |
| **Remove** | `removeFirst()` | `pollFirst()` | `removeLast()` | `pollLast()` |
| **Examine** | `getFirst()` | `peekFirst()` | `getLast()` | `peekLast()` |

### Inherited Queue Methods Behavior
```java
// Queue methods delegate to Deque methods
add(e)     → addLast(e)      // Add at rear
offer(e)   → offerLast(e)    // Add at rear
remove()   → removeFirst()   // Remove from front
poll()     → pollFirst()     // Remove from front
element()  → getFirst()      // Examine front
peek()     → peekFirst()     // Examine front
```

### Using Deque as Stack
```java
// Stack operations (LIFO)
push(e)    → addFirst(e)     // Push to top
pop()      → removeFirst()   // Pop from top

// Example: Deque as Stack
Deque<Integer> stack = new ArrayDeque<>();
stack.push(1);    // Same as addFirst(1)
stack.push(2);    // Stack: [2, 1]
stack.push(3);    // Stack: [3, 2, 1]
stack.pop();      // Returns 3, Stack: [2, 1]
```

## 2. ArrayDeque Implementation

### Overview
- Concrete class implementing Deque interface
- Can be used as both Queue and Stack
- Resizable array implementation
- **NOT thread-safe**

### ArrayDeque Examples

#### As Queue (FIFO)
```java
public class ArrayDequeAsQueue {
    public static void main(String[] args) {
        ArrayDeque<Integer> queue = new ArrayDeque<>();
        
        // Adding at rear
        queue.addLast(1);   // [1]
        queue.addLast(5);   // [1, 5]
        queue.addLast(10);  // [1, 5, 10]
        
        // Removing from front
        System.out.println(queue.removeFirst()); // 1
        System.out.println(queue.removeFirst()); // 5
        System.out.println(queue.removeFirst()); // 10
    }
}
```

#### As Stack (LIFO)
```java
public class ArrayDequeAsStack {
    public static void main(String[] args) {
        ArrayDeque<Integer> stack = new ArrayDeque<>();
        
        // Adding at front
        stack.addFirst(1);   // [1]
        stack.addFirst(5);   // [5, 1]
        stack.addFirst(10);  // [10, 5, 1]
        
        // Removing from front
        System.out.println(stack.removeFirst()); // 10
        System.out.println(stack.removeFirst()); // 5
        System.out.println(stack.removeFirst()); // 1
    }
}
```

### Time Complexity
| Operation | Time Complexity | Note |
|-----------|----------------|------|
| Insert (first/last) | O(1) amortized | O(n) when resize needed |
| Remove (first/last) | O(1) | |
| Peek/Get (first/last) | O(1) | |
| Space | O(n) | |

### Resize Mechanism
- Initial capacity: 8
- When full: doubles capacity
- Creates new array, copies elements (O(n) operation)

## 3. List Interface

### Overview
- Ordered collection of objects
- Allows duplicate values
- **Index-based access** (0-based indexing)
- Data can be inserted, removed, or accessed from **anywhere**

### List vs Queue
| Aspect | Queue | List |
|--------|-------|------|
| Access | Only at ends (front/rear) | Any position via index |
| Structure | FIFO/LIFO based | Index-based (0, 1, 2...) |
| Underlying DS | Various | Array-based |

### List Methods (Additional to Collection)

```java
public interface List<E> extends Collection<E> {
    // Positional access
    void add(int index, E element);     // Insert at index, shift right
    E get(int index);                    // Retrieve element
    E set(int index, E element);         // Replace element (no shift)
    E remove(int index);                  // Remove, shift left
    
    // Search
    int indexOf(Object o);               // First occurrence
    int lastIndexOf(Object o);           // Last occurrence
    
    // Bulk operations
    boolean addAll(int index, Collection<? extends E> c);
    
    // List iteration
    ListIterator<E> listIterator();
    ListIterator<E> listIterator(int index);
    
    // View
    List<E> subList(int fromIndex, int toIndex);
    
    // Modification
    void replaceAll(UnaryOperator<E> operator);
    void sort(Comparator<? super E> c);
}
```

### Key Method Differences

#### add() vs set()
```java
List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4));

// add() - shifts elements right
list.add(2, 100);  // [1, 2, 100, 3, 4]

// set() - replaces element
list.set(2, 200);  // [1, 2, 200, 3, 4]
```

## 4. ArrayList Implementation

### Overview
- Resizable array implementation of List
- Fast random access O(1)
- **NOT thread-safe**
- Maintains insertion order

### ArrayList Example
```java
public class ArrayListExample {
    public static void main(String[] args) {
        List<Integer> list1 = new ArrayList<>();
        
        // Adding elements at specific indices
        list1.add(0, 100);  // [100]
        list1.add(1, 200);  // [100, 200]
        list1.add(2, 300);  // [100, 200, 300]
        
        // Adding another collection
        List<Integer> list2 = Arrays.asList(400, 500, 600);
        list1.addAll(2, list2);  // [100, 200, 400, 500, 600, 300]
        
        // Replace all elements
        list1.replaceAll(val -> val * -1);  // Negate all
        
        // Sorting
        list1.sort((a, b) -> a - b);  // Ascending order
        
        // Get element
        Integer element = list1.get(2);  // Get index 2
        
        // Set (replace) element
        list1.set(2, -4000);  // Replace index 2
        
        // Remove element
        list1.remove(2);  // Remove index 2, shift left
        
        // Find index
        int index = list1.indexOf(-200);  // First occurrence
        int lastIndex = list1.lastIndexOf(-200);  // Last occurrence
    }
}
```

### ListIterator
```java
public class ListIteratorExample {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        ListIterator<Integer> iterator = list.listIterator();
        
        // Forward iteration
        System.out.println("Forward:");
        while (iterator.hasNext()) {
            int value = iterator.next();
            System.out.println("Value: " + value);
            System.out.println("Next Index: " + iterator.nextIndex());
            System.out.println("Previous Index: " + iterator.previousIndex());
            
            // Add element during iteration
            if (value == 3) {
                iterator.add(30);  // Adds between current and next
            }
        }
        
        // Backward iteration
        System.out.println("\nBackward:");
        ListIterator<Integer> backIterator = list.listIterator(list.size());
        while (backIterator.hasPrevious()) {
            int value = backIterator.previous();
            System.out.println("Value: " + value);
            
            // Replace element during iteration
            if (value == 30) {
                backIterator.set(35);
            }
        }
    }
}
```

### Time Complexity
| Operation | Time Complexity | Note |
|-----------|----------------|------|
| Add at end | O(1) amortized | O(n) when resize |
| Add at index | O(n) | Requires shifting |
| Remove | O(n) | Requires shifting |
| Get by index | O(1) | Direct access |
| Search | O(n) | Linear search |
| Space | O(n) | |

## 5. LinkedList Implementation

### Overview
- Doubly-linked list implementation
- Implements **both List and Deque** interfaces
- Better for frequent insertion/deletion
- **NOT thread-safe**

### LinkedList Structure
```
null ← [Node1] ↔ [Node2] ↔ [Node3] ↔ [Node4] → null
         ↑                              ↑
        Head                           Tail
```

### LinkedList Example
```java
public class LinkedListExample {
    public static void main(String[] args) {
        LinkedList<Integer> linkedList = new LinkedList<>();
        
        // Using as Deque
        linkedList.addLast(200);   // [200]
        linkedList.addLast(300);   // [200, 300]
        linkedList.addLast(400);   // [200, 300, 400]
        linkedList.addFirst(100);  // [100, 200, 300, 400]
        
        System.out.println(linkedList.getFirst()); // 100
        
        // Using as List (with indices)
        linkedList.add(1, 150);    // [100, 150, 200, 300, 400]
        System.out.println(linkedList.get(1));  // 150
        System.out.println(linkedList.get(2));  // 200
    }
}
```

### Time Complexity
| Operation | Time Complexity | Note |
|-----------|----------------|------|
| Add at start/end | O(1) | Direct pointer update |
| Add at index | O(n) + O(1) | O(n) to find, O(1) to add |
| Remove start/end | O(1) | |
| Remove at index | O(n) + O(1) | O(n) to find, O(1) to remove |
| Get by index | O(n) | Must traverse |
| Search | O(n) | Linear traversal |
| Space | O(n) | |

## 6. Vector (Thread-Safe ArrayList)

### Overview
- **Thread-safe** version of ArrayList
- All methods are `synchronized`
- Less efficient than ArrayList due to locking
- Legacy class (since Java 1.0)

### Vector Example
```java
public class VectorExample {
    public static void main(String[] args) {
        Vector<Integer> vector = new Vector<>();
        
        // Same methods as ArrayList but thread-safe
        vector.add(1);
        vector.add(2);
        vector.add(3);
        
        // All operations are synchronized
        vector.get(0);        // Thread-safe
        vector.remove(1);     // Thread-safe
        vector.set(0, 10);    // Thread-safe
    }
}
```

## 7. Stack (LIFO)

### Overview
- Extends Vector (hence thread-safe)
- Last In First Out (LIFO) structure
- Legacy class
- Better to use Deque for new code

### Stack Example
```java
public class StackExample {
    public static void main(String[] args) {
        Stack<Integer> stack = new Stack<>();
        
        // Push elements
        stack.push(1);   // [1]
        stack.push(2);   // [1, 2]
        stack.push(3);   // [1, 2, 3]
        
        // Pop elements
        System.out.println(stack.pop());   // 3
        System.out.println(stack.pop());   // 2
        System.out.println(stack.peek());  // 1 (doesn't remove)
    }
}
```

## Comparison Summary

### Feature Comparison
| Collection | Thread-Safe | Maintains Order | Null Allowed | Duplicates | Access Type |
|------------|------------|-----------------|--------------|------------|-------------|
| ArrayDeque | No | Yes | No | Yes | Both ends |
| ArrayList | No | Yes | Yes | Yes | Index-based |
| LinkedList | No | Yes | Yes | Yes | Index + Deque |
| Vector | Yes | Yes | Yes | Yes | Index-based |
| Stack | Yes | Yes* | Yes | Yes | Top only |

*Stack maintains insertion order but retrieval is LIFO

### Thread-Safe Alternatives
| Non Thread-Safe | Thread-Safe Alternative |
|-----------------|------------------------|
| PriorityQueue | PriorityBlockingQueue |
| ArrayDeque | ConcurrentLinkedDeque |
| ArrayList | CopyOnWriteArrayList / Vector |
| LinkedList | ConcurrentLinkedDeque |
| - | Stack (inherently thread-safe) |

## Best Practices

1. **Choose the Right Implementation**
    - Random access needed? → ArrayList
    - Frequent insertion/deletion? → LinkedList
    - Queue operations? → ArrayDeque
    - Thread safety needed? → Use concurrent versions

2. **Avoid Legacy Classes**
   ```java
   // Avoid
   Stack<Integer> stack = new Stack<>();
   
   // Prefer
   Deque<Integer> stack = new ArrayDeque<>();
   ```

3. **Use Interface Types**
   ```java
   // Good
   List<String> list = new ArrayList<>();
   Deque<Integer> deque = new ArrayDeque<>();
   
   // Not recommended
   ArrayList<String> list = new ArrayList<>();
   ```

4. **Consider Thread Safety**
   ```java
   // Multi-threaded environment
   List<String> list = new CopyOnWriteArrayList<>();
   Deque<Integer> deque = new ConcurrentLinkedDeque<>();
   ```

## Interview Key Points

1. **Deque vs Queue**
    - Queue: Single-ended (FIFO)
    - Deque: Double-ended (can be FIFO or LIFO)

2. **ArrayList vs LinkedList**
    - ArrayList: Better for random access O(1)
    - LinkedList: Better for insertion/deletion O(1)

3. **Why Vector/Stack are Legacy**
    - Heavy synchronization overhead
    - Better alternatives available (concurrent collections)

4. **ListIterator Features**
    - Bidirectional traversal
    - Can add/set during iteration
    - Provides index information

5. **Amortized Complexity**
    - Average case O(1) for ArrayDeque/ArrayList add
    - Worst case O(n) when resize needed