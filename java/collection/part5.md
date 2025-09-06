# Java Collections Framework - Part 5: LinkedHashMap & TreeMap

## 1. LinkedHashMap Overview

### Key Differences from HashMap
- **HashMap**: Does NOT maintain any order
- **LinkedHashMap**: Maintains order (2 types)
    1. **Insertion Order** (default)
    2. **Access Order** (optional)

### Two Types of Ordering

#### 1. Insertion Order
- Elements retrieved in the same order they were inserted
- Example: Insert [1, 2, 3, 4] → Retrieve [1, 2, 3, 4]

#### 2. Access Order
- Less frequently used → More frequently used
- Recently accessed elements move to the end
- Useful for caching implementations (LRU Cache)

## 2. LinkedHashMap Internal Design

### How It Differs from HashMap

LinkedHashMap extends HashMap but adds **doubly linked list** functionality.

#### HashMap Node Structure
```java
class Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;  // For collision chaining
}
```

#### LinkedHashMap Entry Structure
```java
class Entry<K,V> extends HashMap.Node<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;   // For collision chaining
    Entry<K,V> before; // For doubly linked list
    Entry<K,V> after;  // For doubly linked list
}
```

### Visual Representation

```
LinkedHashMap Structure:

Array (Buckets) - Same as HashMap:
[0] → [1, A] → [21, B]
[1] → [23, C] → [25, E]  
[2] → [141, D]

Doubly Linked List (Maintains Order):
null ← [1, A] ↔ [21, B] ↔ [23, C] ↔ [141, D] ↔ [25, E] → null
        ↑                                            ↑
       Head                                        Tail
```

### How Insertion Works

When inserting elements:
1. **HashMap operations** happen normally (hash, index, collision handling)
2. **Additionally**, maintain doubly linked list pointers

Example sequence:
```java
put(1, "A")    // Head: [1,A], Tail: [1,A]
put(21, "B")   // Head: [1,A] ↔ [21,B], Tail: [21,B]
put(23, "C")   // Head: [1,A] ↔ [21,B] ↔ [23,C], Tail: [23,C]
put(141, "D")  // Head: [1,A] ↔ ... ↔ [141,D], Tail: [141,D]
put(25, "E")   // Head: [1,A] ↔ ... ↔ [25,E], Tail: [25,E]
```

## 3. LinkedHashMap Examples

### Insertion Order (Default)
```java
public class LinkedHashMapInsertionOrder {
    public static void main(String[] args) {
        Map<Integer, String> linkedMap = new LinkedHashMap<>();
        
        linkedMap.put(1, "A");
        linkedMap.put(21, "B");
        linkedMap.put(23, "C");
        linkedMap.put(141, "D");
        linkedMap.put(25, "E");
        
        // Iteration maintains insertion order
        for (Map.Entry<Integer, String> entry : linkedMap.entrySet()) {
            System.out.println(entry.getKey() + " : " + entry.getValue());
        }
        // Output: 1:A, 21:B, 23:C, 141:D, 25:E (insertion order)
        
        // Compare with HashMap
        Map<Integer, String> hashMap = new HashMap<>();
        hashMap.put(1, "A");
        hashMap.put(21, "B");
        hashMap.put(23, "C");
        hashMap.put(141, "D");
        hashMap.put(25, "E");
        
        // HashMap doesn't guarantee order
        // Output might be: 1:A, 21:B, 23:C, 25:E, 141:D (random order)
    }
}
```

### Access Order
```java
public class LinkedHashMapAccessOrder {
    public static void main(String[] args) {
        // Constructor: (initialCapacity, loadFactor, accessOrder)
        Map<Integer, String> map = new LinkedHashMap<>(16, 0.75f, true);
        
        map.put(1, "A");
        map.put(21, "B");
        map.put(23, "C");
        map.put(141, "D");
        map.put(25, "E");
        
        // Access element 23
        map.get(23);  // Moves 23 to the end
        
        // Iteration: Less frequently → More frequently used
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " : " + entry.getValue());
        }
        // Output: 1:A, 21:B, 141:D, 25:E, 23:C
        // (23 moved to end because it was accessed)
    }
}
```

### LRU Cache Implementation Using LinkedHashMap
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;
    
    public LRUCache(int capacity) {
        super(capacity, 0.75f, true);  // true for access order
        this.capacity = capacity;
    }
    
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size() > capacity;  // Remove least recently used
    }
}
```

## 4. LinkedHashMap Time Complexity

| Operation | Time Complexity | Note |
|-----------|----------------|------|
| put() | O(1) average | Same as HashMap |
| get() | O(1) average | Same as HashMap |
| remove() | O(1) average | Same as HashMap |
| Iteration | O(n) | Maintains order |
| Space | O(n) | Extra space for linked list pointers |

**Thread Safety**: NOT thread-safe

### Making LinkedHashMap Thread-Safe
```java
Map<K, V> syncMap = Collections.synchronizedMap(new LinkedHashMap<>());
```

## 5. TreeMap Overview

### Key Features
- Implements **NavigableMap** interface
- Stores entries in **sorted order**
- Uses **Red-Black Tree** (self-balancing BST)
- Sorting based on:
    - Natural ordering of keys
    - Custom Comparator

### Interface Hierarchy
```
Map
 └── SortedMap
      └── NavigableMap
           └── TreeMap (concrete class)
```

## 6. TreeMap Internal Structure

### Node Structure
```java
class Entry<K,V> {
    K key;
    V value;
    Entry<K,V> parent;
    Entry<K,V> left;
    Entry<K,V> right;
    boolean color;  // RED or BLACK for Red-Black tree
}
```

### Visual Representation
```
TreeMap (Binary Search Tree):

        [4, "SJ"]
        /        \
   [1, "PJ"]    [5, "KJ"]
              /
         [3, "MJ"]
```

### How Insertion Works
1. Compare with root
2. Go left if smaller, right if larger
3. Insert at appropriate leaf position
4. Rebalance tree if needed (Red-Black tree properties)

## 7. TreeMap Examples

### Natural Ordering (Ascending)
```java
public class TreeMapNaturalOrder {
    public static void main(String[] args) {
        Map<Integer, String> treeMap = new TreeMap<>();
        
        treeMap.put(21, "B");
        treeMap.put(1, "A");
        treeMap.put(13, "C");
        treeMap.put(5, "D");
        treeMap.put(11, "E");
        
        // Iteration in sorted order (ascending)
        for (Map.Entry<Integer, String> entry : treeMap.entrySet()) {
            System.out.println(entry.getKey() + " : " + entry.getValue());
        }
        // Output: 1:A, 5:D, 11:E, 13:C, 21:B
    }
}
```

### Custom Comparator (Descending)
```java
public class TreeMapCustomOrder {
    public static void main(String[] args) {
        // Descending order comparator
        Map<Integer, String> treeMap = new TreeMap<>((k1, k2) -> k2 - k1);
        
        treeMap.put(21, "B");
        treeMap.put(1, "A");
        treeMap.put(13, "C");
        treeMap.put(5, "D");
        treeMap.put(11, "E");
        
        // Iteration in descending order
        for (Map.Entry<Integer, String> entry : treeMap.entrySet()) {
            System.out.println(entry.getKey() + " : " + entry.getValue());
        }
        // Output: 21:B, 13:C, 11:E, 5:D, 1:A
    }
}
```

## 8. SortedMap Methods

```java
public class SortedMapMethods {
    public static void main(String[] args) {
        SortedMap<Integer, String> map = new TreeMap<>();
        map.put(5, "A");
        map.put(11, "B");
        map.put(13, "C");
        map.put(21, "D");
        
        // SortedMap specific methods
        System.out.println(map.firstKey());      // 5
        System.out.println(map.lastKey());       // 21
        
        // headMap(toKey) - exclusive
        SortedMap<Integer, String> head = map.headMap(13);
        System.out.println(head);  // {5=A, 11=B}
        
        // tailMap(fromKey) - inclusive
        SortedMap<Integer, String> tail = map.tailMap(13);
        System.out.println(tail);  // {13=C, 21=D}
    }
}
```

## 9. NavigableMap Methods

```java
public class NavigableMapMethods {
    public static void main(String[] args) {
        NavigableMap<Integer, String> map = new TreeMap<>();
        map.put(1, "A");
        map.put(21, "B");
        map.put(23, "C");
        map.put(25, "D");
        map.put(141, "E");
        
        // Lower methods (strictly less than)
        System.out.println(map.lowerEntry(23));  // 21=B
        System.out.println(map.lowerKey(23));    // 21
        
        // Floor methods (less than or equal)
        System.out.println(map.floorEntry(24));  // 23=C
        System.out.println(map.floorEntry(23));  // 23=C
        
        // Ceiling methods (greater than or equal)
        System.out.println(map.ceilingEntry(24)); // 25=D
        System.out.println(map.ceilingEntry(23)); // 23=C
        
        // Higher methods (strictly greater than)
        System.out.println(map.higherEntry(23));  // 25=D
        System.out.println(map.higherKey(23));    // 25
        
        // Poll methods (retrieve and remove)
        System.out.println(map.pollFirstEntry()); // 1=A (removed)
        System.out.println(map.pollLastEntry());  // 141=E (removed)
        
        // Descending view
        NavigableMap<Integer, String> descMap = map.descendingMap();
        // Iterates in reverse order
        
        // Range views with inclusive/exclusive control
        NavigableMap<Integer, String> subMap = 
            map.subMap(21, true, 25, false);  // [21, 25)
    }
}
```

## 10. TreeMap Time Complexity

| Operation | Time Complexity | Note |
|-----------|----------------|------|
| put() | O(log n) | Tree insertion |
| get() | O(log n) | Tree search |
| remove() | O(log n) | Tree deletion |
| containsKey() | O(log n) | Tree search |
| first/lastKey() | O(log n) | Tree traversal |
| Space | O(n) | |

**Thread Safety**: NOT thread-safe

## 11. Comparison: HashMap vs LinkedHashMap vs TreeMap

| Feature | HashMap | LinkedHashMap | TreeMap |
|---------|---------|---------------|---------|
| **Ordering** | No order | Insertion/Access order | Sorted order |
| **Null Keys** | One null key | One null key | No null keys |
| **Null Values** | Multiple | Multiple | Multiple |
| **Performance** | O(1) average | O(1) average | O(log n) |
| **Memory** | Lowest | Medium (extra pointers) | Highest (tree nodes) |
| **Use Case** | General purpose | LRU cache, ordered data | Sorted data, range queries |
| **Data Structure** | Array + LinkedList/Tree | Array + DLL | Red-Black Tree |

## 12. When to Use Which Map?

### Use HashMap when:
- No ordering required
- Best performance needed
- General purpose key-value storage

### Use LinkedHashMap when:
- Need to maintain insertion order
- Implementing LRU cache
- Predictable iteration order needed

### Use TreeMap when:
- Need sorted order
- Range queries (headMap, tailMap, subMap)
- NavigableMap operations needed
- Floor/ceiling operations required

## 13. Practical Examples

### LRU Cache with LinkedHashMap
```java
public class LRUCacheExample {
    private final LinkedHashMap<Integer, String> cache;
    private final int capacity;
    
    public LRUCacheExample(int capacity) {
        this.capacity = capacity;
        this.cache = new LinkedHashMap<Integer, String>(capacity, 0.75f, true) {
            @Override
            protected boolean removeEldestEntry(Map.Entry<Integer, String> eldest) {
                return size() > capacity;
            }
        };
    }
    
    public void put(int key, String value) {
        cache.put(key, value);
    }
    
    public String get(int key) {
        return cache.get(key);
    }
}
```

### Range Queries with TreeMap
```java
public class RangeQueryExample {
    public static void main(String[] args) {
        TreeMap<Integer, String> scores = new TreeMap<>();
        scores.put(95, "A+");
        scores.put(85, "A");
        scores.put(75, "B");
        scores.put(65, "C");
        scores.put(55, "D");
        
        // Get all scores between 70 and 90
        NavigableMap<Integer, String> goodScores = 
            scores.subMap(70, true, 90, true);
        System.out.println(goodScores); // {75=B, 85=A}
        
        // Get highest failing grade (below 60)
        Map.Entry<Integer, String> highestFail = scores.lowerEntry(60);
        System.out.println(highestFail); // 55=D
    }
}
```

## Interview Key Points

1. **LinkedHashMap = HashMap + Doubly Linked List**
    - Maintains order with minimal performance impact
    - Perfect for LRU cache implementation

2. **TreeMap = Red-Black Tree**
    - Always sorted
    - O(log n) operations
    - No null keys allowed

3. **Access Order in LinkedHashMap**
    - Constructor parameter controls ordering
    - Useful for cache eviction policies

4. **TreeMap Use Cases**
    - When you need sorted data
    - Range queries
    - Floor/ceiling operations

5. **Performance Trade-offs**
    - HashMap: Fastest, no order
    - LinkedHashMap: Fast with order overhead
    - TreeMap: Slower but sorted