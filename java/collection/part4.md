# Java Collections Framework - Part 4: Map & HashMap

## 1. Why Map is NOT a Child of Collection?

### The Fundamental Difference
- **Collection Interface**: Deals with single values
    - List: [value1, value2, value3]
    - Set: {value1, value2, value3}
    - Queue: value1 → value2 → value3

- **Map Interface**: Deals with key-value pairs
    - Map: {key1→value1, key2→value2, key3→value3}
    - Requires completely different methods
    - No point inheriting Collection methods

## 2. Map Interface Overview

### Properties
1. **Object that maps keys to values**
2. **Cannot contain duplicate keys**
3. **Each key maps to at most one value**
4. **Value can be duplicate**

### Basic Structure Example
```
Key → Value
1   → "SJ"
2   → "KJ"
3   → "PJ"
4   → "KJ"  (duplicate value allowed)
```

### Core Map Methods

```java
public interface Map<K, V> {
    // Size operations
    int size();              // Number of key-value mappings
    boolean isEmpty();       // true if no mappings
    
    // Query operations
    boolean containsKey(Object key);
    boolean containsValue(Object value);
    V get(Object key);      // Returns value for key
    
    // Modification operations
    V put(K key, V value);   // Insert/update mapping
    V remove(Object key);    // Remove mapping
    V putIfAbsent(K key, V value);
    
    // Bulk operations
    void putAll(Map<? extends K, ? extends V> m);
    void clear();
    
    // Views
    Set<K> keySet();         // All keys
    Collection<V> values();  // All values
    Set<Map.Entry<K, V>> entrySet();  // All entries
}
```

## 3. HashMap Internal Design

### Data Structure
HashMap internally uses an **array of Node<K,V>** (implements Map.Entry<K,V>)

```java
// Simplified Node structure
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;     // Hash of the key
    final K key;        // Key
    V value;            // Value
    Node<K,V> next;     // Next node (for collision handling)
}

// Internal array
Node<K,V>[] table;  // Array of nodes (buckets)
```

### Visual Representation
```
HashMap Internal Structure:

Index  Array (Buckets)
  0 → [Node] → [Node] → [Node]  (Linked List for collisions)
  1 → [Node]
  2 → null
  3 → [Node] → [Node]
 ...
 15 → [Node]

Each Node contains: [hash | key | value | next]
```

## 4. How put() Method Works

### Steps for put(key, value):

```java
public V put(K key, V value) {
    // Step 1: Calculate hash of key
    int hash = hash(key);
    
    // Step 2: Find bucket index
    int index = hash % table.length;
    
    // Step 3: Check if key exists
    // - If exists: update value
    // - If not: add new node
    
    // Step 4: Handle collision if needed
    // - Use chaining (linked list)
    // - Convert to tree if threshold reached
}
```

### Example: put() Operation
```java
Map<Integer, String> map = new HashMap<>();

// put(1, "SJ")
// 1. hash(1) = 1234567
// 2. index = 1234567 % 16 = 7
// 3. table[7] = new Node(1234567, 1, "SJ", null)

// put(17, "PJ") - Collision example
// 1. hash(17) = 9876543
// 2. index = 9876543 % 16 = 7 (same as key 1!)
// 3. Collision! Chain it: table[7].next = new Node(...)
```

## 5. How get() Method Works

### Steps for get(key):

```java
public V get(Object key) {
    // Step 1: Calculate hash of key
    int hash = hash(key);
    
    // Step 2: Find bucket index
    int index = hash % table.length;
    
    // Step 3: Search in bucket
    // - If linked list: iterate and compare keys
    // - If tree: binary search
    
    // Step 4: Return value or null
}
```

## 6. Collision Handling

### Chaining (Linked List)
- When collision occurs, nodes are linked together
- Forms a linked list at that bucket index

### Tree-ification
- **Treeify Threshold**: 8
- When chain length ≥ 8, convert to balanced tree (Red-Black Tree)
- Improves worst-case from O(n) to O(log n)

```
Before (Linked List):
Index 7 → Node1 → Node2 → ... → Node8

After (Tree):
Index 7 →     Node4
             /     \
          Node2    Node6
          /  \     /  \
       Node1 Node3 Node5 Node7
```

## 7. Load Factor & Rehashing

### Load Factor
- **Default**: 0.75
- **Formula**: Threshold = Capacity × Load Factor
- **Example**: 16 × 0.75 = 12

### Rehashing Process
```
Initial: Capacity = 16, Threshold = 12
When 13th element is added:
1. Double capacity: 16 → 32
2. Create new array[32]
3. Rehash all existing entries
4. New threshold = 32 × 0.75 = 24
```

## 8. Key Concepts

### Default Values
- **Initial Capacity**: 16
- **Load Factor**: 0.75
- **Treeify Threshold**: 8
- **Untreeify Threshold**: 6

### hashCode() and equals() Contract

#### Contract 1:
```java
if (obj1.equals(obj2)) {
    // Then: obj1.hashCode() == obj2.hashCode()
}
```

#### Contract 2:
```java
if (obj1.hashCode() == obj2.hashCode()) {
    // Does NOT guarantee: obj1.equals(obj2)
}
```

## 9. Time Complexity Analysis

| Operation | Average Case | Worst Case | Note |
|-----------|-------------|------------|------|
| put() | O(1) | O(log n) | O(n) if all linked list |
| get() | O(1) | O(log n) | O(n) if all linked list |
| remove() | O(1) | O(log n) | O(n) if all linked list |
| containsKey() | O(1) | O(log n) | |

### Why O(log n) Worst Case?
- Linked list converts to tree after threshold
- Tree operations are O(log n)

## 10. Complete HashMap Example

```java
public class HashMapExample {
    public static void main(String[] args) {
        Map<Integer, String> map = new HashMap<>();
        
        // Basic operations
        map.put(null, "test");     // null key allowed
        map.put(0, null);          // null value allowed
        map.put(1, "A");
        map.put(2, "B");
        
        // putIfAbsent
        map.putIfAbsent(0, "Zero"); // Replaces null value
        map.putIfAbsent(3, "C");    // Adds new entry
        
        // Iteration using entrySet()
        for (Map.Entry<Integer, String> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " : " + entry.getValue());
        }
        
        // Check operations
        System.out.println("Is empty: " + map.isEmpty());    // false
        System.out.println("Size: " + map.size());           // 5
        System.out.println("Contains key 3: " + map.containsKey(3)); // true
        
        // Get operations
        System.out.println("Get key 1: " + map.get(1));     // "A"
        System.out.println("Get or default: " + 
            map.getOrDefault(9, "Default"));                 // "Default"
        
        // Remove operation
        String removed = map.remove(null);  // Removes and returns "test"
        
        // Iterate keys only
        for (Integer key : map.keySet()) {
            System.out.println("Key: " + key);
        }
        
        // Iterate values only
        for (String value : map.values()) {
            System.out.println("Value: " + value);
        }
    }
}
```

## 11. HashMap vs HashTable

| Feature | HashMap | HashTable |
|---------|---------|-----------|
| **Thread Safety** | Not thread-safe | Thread-safe (synchronized) |
| **Null Keys** | One null key allowed | No null keys |
| **Null Values** | Multiple null values allowed | No null values |
| **Performance** | Faster | Slower (due to synchronization) |
| **Legacy** | Java 1.2+ | Java 1.0 (Legacy) |
| **Iteration** | Fail-fast iterator | Enumerator (not fail-fast) |

## 12. Thread-Safe Alternatives

### For HashMap:
1. **HashTable** - Fully synchronized (legacy)
2. **ConcurrentHashMap** - Better concurrency (preferred)
3. **Collections.synchronizedMap()** - Wrapper approach

```java
// Thread-safe alternatives
Map<K, V> map1 = new Hashtable<>();
Map<K, V> map2 = new ConcurrentHashMap<>();
Map<K, V> map3 = Collections.synchronizedMap(new HashMap<>());
```

## Interview Key Points

### 1. HashMap Internal Working
- Array of Node<K,V> (buckets)
- Collision handling via chaining
- Tree-ification after threshold

### 2. Time Complexity
- Average: O(1) for all operations
- Worst: O(log n) due to tree conversion

### 3. Load Factor Impact
- Controls when to resize
- Trade-off: Memory vs Performance
- Lower load factor = More memory, Less collisions

### 4. Why HashMap is Fast
- Direct array access via hashing
- Balanced trees for long chains
- Automatic resizing

### 5. Common Pitfalls
```java
// Wrong - Infinite loop possible in multi-threaded environment
Map<String, String> map = new HashMap<>();  // Not thread-safe

// Right - For concurrent access
Map<String, String> map = new ConcurrentHashMap<>();
```

### 6. Key Requirements
- Must properly implement hashCode() and equals()
- hashCode() must be consistent
- Equal objects must have equal hash codes

## Best Practices

1. **Initial Capacity**: Set if size is known
   ```java
   Map<String, Integer> map = new HashMap<>(1000);
   ```

2. **Use Interface Type**:
   ```java
   Map<K, V> map = new HashMap<>();  // Good
   HashMap<K, V> map = new HashMap<>();  // Not recommended
   ```

3. **Immutable Keys**: Prefer immutable objects as keys
   ```java
   Map<String, Value> good = new HashMap<>();  // String is immutable
   Map<StringBuilder, Value> bad = new HashMap<>();  // Mutable - avoid
   ```

4. **Null Handling**: Be careful with null keys/values
   ```java
   // Check before get
   if (map.containsKey(key)) {
       Value v = map.get(key);
   }
   ```

## Summary
- HashMap provides O(1) average performance
- Uses array + linked list + tree structure
- Not thread-safe (use ConcurrentHashMap for concurrency)
- Allows one null key and multiple null values
- Load factor controls resize behavior
- Proper hashCode() and equals() implementation crucial