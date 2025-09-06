# Java Collections Framework - Part 6: Set & Its Implementations

## 1. Set Interface Overview

### Key Properties
- **Collection of unique objects** - No duplicates allowed
- **Why no duplicates?** - Internally uses Map (keys are unique)
- **Can have only one null value** (for HashSet/LinkedHashSet)
- **Generally unordered** (except LinkedHashSet and TreeSet)
- **Cannot be accessed via index** (unlike List)

### Set vs List
| Feature | Set | List |
|---------|-----|------|
| Duplicates | Not allowed | Allowed |
| Null values | One null (HashSet) | Multiple nulls |
| Ordering | Generally unordered | Ordered |
| Index access | No | Yes |
| Use case | Unique elements | Ordered collection |

## 2. How Set Uses Map Internally

### The Key Insight
Set implementations use Map implementations internally:
- **HashSet** → Uses **HashMap**
- **LinkedHashSet** → Uses **LinkedHashMap**
- **TreeSet** → Uses **TreeMap**

### Internal Storage Mechanism
```java
// When you do: set.add(12)
// Internally it does: map.put(12, PRESENT)

// Inside HashSet class:
private transient HashMap<E,Object> map;
private static final Object PRESENT = new Object(); // Dummy value

public boolean add(E e) {
    return map.put(e, PRESENT) == null;
}
```

**Key Point**:
- Set elements are stored as **keys** in the internal Map
- Values are just dummy objects (PRESENT)
- We only care about keys, not values

## 3. Set Interface Methods

### Unique Set Operations
```java
public interface Set<E> extends Collection<E> {
    // Basic operations
    boolean add(E e);         // Returns false if duplicate
    boolean remove(Object o);
    boolean contains(Object o);
    
    // Set operations (Mathematical)
    boolean addAll(Collection<? extends E> c);    // Union
    boolean retainAll(Collection<?> c);           // Intersection
    boolean removeAll(Collection<?> c);           // Difference
    
    // From Collection
    int size();
    boolean isEmpty();
    void clear();
    Iterator<E> iterator();
}
```

### Mathematical Set Operations

#### Union (addAll)
```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 88, 10, 5, 12));

set1.addAll(set2);  // Union
// Result: [1, 2, 11, 33, 4, 9, 88, 10, 5, 12]
```

#### Intersection (retainAll)
```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4, 12));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 88, 10, 5, 12));

set1.retainAll(set2);  // Intersection
// Result: [11, 12]
```

#### Difference (removeAll)
```java
Set<Integer> set1 = new HashSet<>(Arrays.asList(1, 2, 11, 33, 4, 12));
Set<Integer> set2 = new HashSet<>(Arrays.asList(11, 9, 88, 10, 5, 12));

set1.removeAll(set2);  // Difference
// Result: [1, 2, 33, 4]
```

## 4. HashSet Implementation

### Overview
- Uses **HashMap** internally
- **No ordering** guaranteed
- Allows **one null** element
- **NOT thread-safe**
- Best performance: O(1) average

### HashSet Example
```java
public class HashSetExample {
    public static void main(String[] args) {
        Set<Integer> hashSet = new HashSet<>();
        
        // Adding elements
        hashSet.add(5);
        hashSet.add(2);
        hashSet.add(8);
        hashSet.add(2);  // Duplicate - returns false, not added
        hashSet.add(null); // One null allowed
        
        // Iteration - no guaranteed order
        for (Integer num : hashSet) {
            System.out.println(num);
        }
        // Possible output: null, 2, 5, 8 (order not guaranteed)
        
        // Check operations
        System.out.println(hashSet.contains(5));  // true
        System.out.println(hashSet.size());       // 4
    }
}
```

### Thread-Safe HashSet
```java
// Using ConcurrentHashMap's keySet
Set<Integer> threadSafeSet = ConcurrentHashMap.newKeySet();

// Alternative using Collections.synchronizedSet
Set<Integer> syncSet = Collections.synchronizedSet(new HashSet<>());
```

## 5. LinkedHashSet Implementation

### Overview
- Uses **LinkedHashMap** internally
- Maintains **insertion order**
- Allows **one null** element
- **NOT thread-safe**
- Performance: O(1) average with ordering overhead

### Why Only Insertion Order?
LinkedHashMap supports both insertion and access order, but LinkedHashSet only exposes insertion order:

```java
// LinkedHashSet constructor
public LinkedHashSet() {
    super(16, .75f, false);  // false = insertion order only
}
// Access order parameter is not exposed in LinkedHashSet API
```

### LinkedHashSet Example
```java
public class LinkedHashSetExample {
    public static void main(String[] args) {
        Set<Integer> linkedSet = new LinkedHashSet<>();
        
        linkedSet.add(2);
        linkedSet.add(77);
        linkedSet.add(82);
        linkedSet.add(63);
        linkedSet.add(5);
        
        // Iteration maintains insertion order
        for (Integer num : linkedSet) {
            System.out.println(num);
        }
        // Output: 2, 77, 82, 63, 5 (insertion order)
    }
}
```

## 6. TreeSet Implementation

### Overview
- Uses **TreeMap** internally (Red-Black Tree)
- Elements stored in **sorted order**
- **NO null** elements allowed
- **NOT thread-safe**
- Performance: O(log n)

### TreeSet Examples

#### Natural Ordering (Ascending)
```java
public class TreeSetNaturalOrder {
    public static void main(String[] args) {
        Set<Integer> treeSet = new TreeSet<>();
        
        treeSet.add(2);
        treeSet.add(5);
        treeSet.add(63);
        treeSet.add(77);
        treeSet.add(82);
        
        // Iteration in natural order (ascending)
        for (Integer num : treeSet) {
            System.out.println(num);
        }
        // Output: 2, 5, 63, 77, 82
    }
}
```

#### Custom Comparator (Descending)
```java
public class TreeSetCustomOrder {
    public static void main(String[] args) {
        // Descending order
        Set<Integer> treeSet = new TreeSet<>((a, b) -> b - a);
        
        treeSet.add(2);
        treeSet.add(5);
        treeSet.add(63);
        treeSet.add(77);
        treeSet.add(82);
        
        // Iteration in descending order
        for (Integer num : treeSet) {
            System.out.println(num);
        }
        // Output: 82, 77, 63, 5, 2
    }
}
```

### NavigableSet Operations
```java
NavigableSet<Integer> navSet = new TreeSet<>(Arrays.asList(10, 20, 30, 40, 50));

System.out.println(navSet.lower(30));    // 20 (strictly less)
System.out.println(navSet.floor(35));    // 30 (less or equal)
System.out.println(navSet.ceiling(35));  // 40 (greater or equal)
System.out.println(navSet.higher(30));   // 40 (strictly greater)

// Range operations
NavigableSet<Integer> subset = navSet.subSet(20, true, 40, false);
// Result: [20, 30]
```

## 7. Concurrent Modification Exception

### The Problem
```java
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
Iterator<Integer> iterator = set.iterator();

while (iterator.hasNext()) {
    Integer value = iterator.next();
    if (value == 3) {
        set.add(8);  // ConcurrentModificationException!
    }
}
```

### Solutions

#### 1. Use Thread-Safe Collection
```java
Set<Integer> set = ConcurrentHashMap.newKeySet();
// Now concurrent modifications are allowed
```

#### 2. Use Iterator's remove()
```java
Iterator<Integer> iterator = set.iterator();
while (iterator.hasNext()) {
    Integer value = iterator.next();
    if (value == 3) {
        iterator.remove();  // Safe removal
    }
}
```

#### 3. Collect changes, apply later
```java
Set<Integer> toAdd = new HashSet<>();
Iterator<Integer> iterator = set.iterator();

while (iterator.hasNext()) {
    Integer value = iterator.next();
    if (value == 3) {
        toAdd.add(8);
    }
}
set.addAll(toAdd);  // Apply after iteration
```

## 8. Comparison Summary

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| **Internal DS** | HashMap | LinkedHashMap | TreeMap (RB-Tree) |
| **Ordering** | No order | Insertion order | Sorted order |
| **Null values** | One null | One null | No nulls |
| **Performance** | O(1) avg | O(1) avg | O(log n) |
| **Memory** | Lowest | Medium | Highest |
| **Thread-safe** | No | No | No |
| **Use case** | General unique elements | Ordered unique elements | Sorted unique elements |

## 9. Time Complexity Comparison

| Operation | HashSet | LinkedHashSet | TreeSet |
|-----------|---------|---------------|---------|
| add() | O(1) avg | O(1) avg | O(log n) |
| remove() | O(1) avg | O(1) avg | O(log n) |
| contains() | O(1) avg | O(1) avg | O(log n) |
| Iteration | O(n) | O(n) | O(n) |
| Space | O(n) | O(n) | O(n) |

## 10. When to Use Which Set?

### Use HashSet when:
- Need unique elements only
- Order doesn't matter
- Best performance required
- General purpose unique collection

### Use LinkedHashSet when:
- Need unique elements with insertion order
- Predictable iteration order needed
- Slightly slower than HashSet acceptable

### Use TreeSet when:
- Need unique elements in sorted order
- Range queries needed
- NavigableSet operations required
- O(log n) performance acceptable

## 11. Practical Examples

### Remove Duplicates from List
```java
public class RemoveDuplicates {
    public static void main(String[] args) {
        List<Integer> listWithDuplicates = 
            Arrays.asList(1, 2, 3, 2, 4, 1, 5, 3);
        
        // Using HashSet (loses order)
        Set<Integer> uniqueSet = new HashSet<>(listWithDuplicates);
        
        // Using LinkedHashSet (maintains order)
        Set<Integer> orderedUniqueSet = new LinkedHashSet<>(listWithDuplicates);
        
        System.out.println(uniqueSet);        // [1, 2, 3, 4, 5] (order not guaranteed)
        System.out.println(orderedUniqueSet); // [1, 2, 3, 4, 5] (insertion order)
    }
}
```

### Finding Common Elements
```java
public class CommonElements {
    public static void main(String[] args) {
        Set<String> skills1 = new HashSet<>(
            Arrays.asList("Java", "Python", "SQL", "Git"));
        Set<String> skills2 = new HashSet<>(
            Arrays.asList("Java", "JavaScript", "SQL", "Docker"));
        
        // Find common skills (intersection)
        Set<String> commonSkills = new HashSet<>(skills1);
        commonSkills.retainAll(skills2);
        System.out.println("Common: " + commonSkills); // [Java, SQL]
        
        // Find all skills (union)
        Set<String> allSkills = new HashSet<>(skills1);
        allSkills.addAll(skills2);
        System.out.println("All: " + allSkills);
        
        // Find unique to skills1 (difference)
        Set<String> uniqueToSkills1 = new HashSet<>(skills1);
        uniqueToSkills1.removeAll(skills2);
        System.out.println("Unique to skills1: " + uniqueToSkills1); // [Python, Git]
    }
}
```

## Interview Key Points

1. **Set uses Map internally**
    - Elements stored as keys
    - Values are dummy objects
    - That's why no duplicates

2. **Why Set doesn't allow duplicates?**
    - Because Map keys are unique
    - Attempting to add duplicate returns false

3. **Thread Safety**
    - None of the basic Set implementations are thread-safe
    - Use `ConcurrentHashMap.newKeySet()` for thread-safe Set

4. **Null handling**
    - HashSet/LinkedHashSet: One null allowed
    - TreeSet: No nulls (needs comparison)

5. **Performance Trade-offs**
    - HashSet: Fastest, no order
    - LinkedHashSet: Fast with order overhead
    - TreeSet: Slower but sorted

6. **ConcurrentModificationException**
    - Occurs when modifying collection during iteration
    - Use concurrent collections or Iterator.remove()

## Best Practices

1. **Choose the right implementation**
   ```java
   // Need uniqueness only
   Set<String> set = new HashSet<>();
   
   // Need order preservation
   Set<String> set = new LinkedHashSet<>();
   
   // Need sorting
   Set<String> set = new TreeSet<>();
   ```

2. **Use interface type**
   ```java
   Set<Integer> set = new HashSet<>();  // Good
   HashSet<Integer> set = new HashSet<>();  // Not recommended
   ```

3. **Immutable Sets (Java 9+)**
   ```java
   Set<String> immutableSet = Set.of("A", "B", "C");
   ```

4. **Efficient set operations**
   ```java
   // Instead of loop for union
   set1.addAll(set2);
   
   // Instead of loop for intersection
   set1.retainAll(set2);
   ```

## Summary
- Set ensures uniqueness using Map internally
- Three main implementations for different needs
- Mathematical set operations built-in
- Choose based on ordering and performance requirements
- Remember: Set elements are Map keys internally