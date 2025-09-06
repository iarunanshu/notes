# Java Collections Framework - Part 1: Introduction & Core Interfaces

## Overview
Java Collections Framework (JCF) provides a unified architecture for representing and manipulating collections of objects. Introduced in Java 1.2, it standardizes how we work with groups of objects.

## 1. What is Java Collections Framework?

### Definition
- **Collection**: Group of objects/elements
- **Framework**: Architecture to manage these groups
- **Package**: `java.util`
- **Since**: Java 1.2

### Components
```java
// Collection - group of objects
List<Integer> numbers = new ArrayList<>();  // Collection of integers

// Framework provides:
// - Interfaces (List, Set, Queue, Map)
// - Implementations (ArrayList, HashSet, LinkedList)
// - Algorithms (sort, search, shuffle)
```

## 2. Why Do We Need Collections Framework?

### Problem Before JCF (Pre-Java 1.2)

**Available Collections**:
- Array
- Vector
- Hashtable

**Main Problem**: No common interface

```java
// Different ways to add/read for different collections

// Array - Direct index access
int[] array = new int[4];
array[0] = 1;              // Write
int value = array[0];      // Read

// Vector - Method calls
Vector<Integer> vector = new Vector<>();
vector.add(1);             // Write (different method!)
int value = vector.get(0); // Read (different method!)

// Hashtable - Yet another way
Hashtable<String, Integer> table = new Hashtable<>();
table.put("key", 1);       // Write (different again!)
int value = table.get("key"); // Read

// Problem: Need to remember different methods for each collection!
```

### Solution with JCF

```java
// Common interface for all collections
List<Integer> arrayList = new ArrayList<>();
List<Integer> linkedList = new LinkedList<>();
List<Integer> vector = new Vector<>();

// Same methods work for all!
arrayList.add(1);   // Same method
linkedList.add(1);  // Same method
vector.add(1);      // Same method

// Same for other operations
arrayList.remove(0);
linkedList.remove(0);
vector.remove(0);
```

## 3. Collections Framework Hierarchy

### Complete Hierarchy
```
                    Iterable (interface)
                        ↓
                    Collection (interface)
                    ↙    ↓    ↘
                List   Queue   Set
                 ↓       ↓      ↓
    ArrayList  PriorityQueue  HashSet
    LinkedList  ArrayDeque    TreeSet
    Vector                    LinkedHashSet
    Stack

    Map (separate hierarchy - not under Collection)
     ↓
    HashMap
    TreeMap
    LinkedHashMap
    Hashtable
```

### Color Coding
- **Light Blue**: Interfaces
- **Pink/Purple**: Concrete Classes

## 4. Iterable Interface (Added in Java 1.5)

### Purpose
Used to traverse collections

### Key Methods

#### 1. iterator() Method (Java 1.5)
Returns an Iterator object with three methods:

```java
public class IteratorExample {
    public static void main(String[] args) {
        List<Integer> values = new ArrayList<>();
        values.add(1);
        values.add(2);
        values.add(3);
        values.add(4);
        
        // Get iterator
        Iterator<Integer> iterator = values.iterator();
        
        // Iterator methods:
        // hasNext() - returns true if more elements exist
        // next() - returns next element
        // remove() - removes last returned element
        
        while (iterator.hasNext()) {
            Integer value = iterator.next();
            System.out.println("Value: " + value);
            
            // Remove element with value 3
            if (value == 3) {
                iterator.remove();
            }
        }
        
        // Verify removal
        System.out.println("After removal: " + values); // [1, 2, 4]
    }
}
```

#### 2. Enhanced For Loop
Any collection implementing Iterable can use for-each loop:

```java
List<Integer> values = Arrays.asList(1, 2, 3, 4);

// Enhanced for loop (syntactic sugar for iterator)
for (Integer value : values) {
    System.out.println(value);
}
```

#### 3. forEach() Method (Java 1.8)
Uses lambda expressions:

```java
List<Integer> values = Arrays.asList(1, 2, 3, 4);

// forEach with lambda
values.forEach(value -> System.out.println(value));

// Method reference
values.forEach(System.out::println);

// With more logic
values.forEach(value -> {
    if (value % 2 == 0) {
        System.out.println(value + " is even");
    }
});
```

### Historical Note
- Iterator **object** existed since Java 1.2 (in Collection interface)
- Iterable **interface** added in Java 1.5 for better abstraction
- forEach() method added in Java 1.8

## 5. Collection Interface

### Purpose
Root interface for all collections (except Map)

### Common Methods (Java 1.2)

```java
public class CollectionMethodsExample {
    public static void main(String[] args) {
        List<Integer> values = new ArrayList<>();
        values.add(2);
        values.add(3);
        values.add(4);
        
        // 1. size() - returns number of elements
        System.out.println("Size: " + values.size()); // 3
        
        // 2. isEmpty() - check if empty
        System.out.println("Empty? " + values.isEmpty()); // false
        
        // 3. contains() - search for element
        System.out.println("Contains 3? " + values.contains(3)); // true
        System.out.println("Contains 5? " + values.contains(5)); // false
        
        // 4. add() - insert element
        values.add(5);
        System.out.println("After add: " + values); // [2, 3, 4, 5]
        
        // 5. remove() - two ways
        // By index (primitive int)
        values.remove(3); // Removes element at index 3
        
        // By object (Integer object)
        values.remove(Integer.valueOf(3)); // Removes value 3
        
        // 6. addAll() - add another collection
        Stack<Integer> stackValues = new Stack<>();
        stackValues.add(6);
        stackValues.add(7);
        stackValues.add(8);
        
        values.addAll(stackValues);
        System.out.println("After addAll: " + values); // [2, 4, 6, 7, 8]
        
        // 7. containsAll() - check if all elements present
        System.out.println("Contains all? " + 
            values.containsAll(stackValues)); // true
        
        // 8. removeAll() - remove all elements from another collection
        values.removeAll(stackValues);
        System.out.println("After removeAll: " + values); // [2, 4]
        
        // 9. clear() - remove all elements
        values.clear();
        System.out.println("After clear: " + values); // []
        
        // 10. toArray() - convert to array
        List<String> list = Arrays.asList("a", "b", "c");
        String[] array = list.toArray(new String[0]);
    }
}
```

### Java 1.8 Additions

```java
// stream() and parallelStream()
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Stream for filtering and processing
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Parallel stream for concurrent processing
numbers.parallelStream()
    .forEach(System.out::println);
```

## 6. Collection vs Collections

### Collection (Interface)
```java
// Part of framework hierarchy
public interface Collection<E> extends Iterable<E> {
    boolean add(E e);
    boolean remove(Object o);
    int size();
    // ... other methods
}
```

### Collections (Utility Class)
```java
// Utility class with static methods
public class Collections {
    public static void sort(List list) { }
    public static int binarySearch(List list, Object key) { }
    public static void reverse(List list) { }
    // ... all static methods
}

// Usage example
List<Integer> numbers = Arrays.asList(3, 1, 4, 1, 5);

// Using Collections utility methods
int max = Collections.max(numbers);        // 5
int min = Collections.min(numbers);        // 1
Collections.sort(numbers);                 // [1, 1, 3, 4, 5]
Collections.reverse(numbers);              // [5, 4, 3, 1, 1]
Collections.shuffle(numbers);              // Random order
int frequency = Collections.frequency(numbers, 1); // 2
```

## 7. Complete Example

```java
import java.util.*;

public class CollectionFrameworkDemo {
    public static void main(String[] args) {
        // Create collection
        List<String> fruits = new ArrayList<>();
        
        // Add elements
        fruits.add("Apple");
        fruits.add("Banana");
        fruits.add("Orange");
        
        // Three ways to iterate
        
        // 1. Using Iterator
        System.out.println("Using Iterator:");
        Iterator<String> iterator = fruits.iterator();
        while (iterator.hasNext()) {
            System.out.println("  " + iterator.next());
        }
        
        // 2. Using enhanced for loop
        System.out.println("\nUsing for-each:");
        for (String fruit : fruits) {
            System.out.println("  " + fruit);
        }
        
        // 3. Using forEach with lambda
        System.out.println("\nUsing forEach:");
        fruits.forEach(fruit -> System.out.println("  " + fruit));
        
        // Collection operations
        System.out.println("\nCollection operations:");
        System.out.println("Size: " + fruits.size());
        System.out.println("Contains 'Apple': " + fruits.contains("Apple"));
        System.out.println("Is empty: " + fruits.isEmpty());
        
        // Using Collections utility
        Collections.sort(fruits);
        System.out.println("Sorted: " + fruits);
        
        Collections.reverse(fruits);
        System.out.println("Reversed: " + fruits);
    }
}
```

## Key Interview Points

### 1. Why Collections Framework?
- **Before**: Different methods for different collections (array[0] vs vector.get(0))
- **After**: Common interface, same methods for all collections

### 2. Iterable vs Collection
- **Iterable**: For traversal (iterator, forEach)
- **Collection**: Extends Iterable, adds manipulation methods (add, remove, size)

### 3. Iterator Methods
- `hasNext()`: Check if more elements
- `next()`: Get next element
- `remove()`: Remove last returned element

### 4. Collection vs Collections
- **Collection**: Interface in framework
- **Collections**: Utility class with static methods

### 5. Three Ways to Iterate
1. Iterator (Java 1.2)
2. Enhanced for-loop (Java 1.5)
3. forEach with lambda (Java 1.8)

## Best Practices

1. **Program to Interface**
```java
// Good
List<String> list = new ArrayList<>();

// Not recommended
ArrayList<String> list = new ArrayList<>();
```

2. **Use Appropriate Collection**
- Need ordering? Use List
- Need uniqueness? Use Set
- Need FIFO/LIFO? Use Queue/Deque

3. **Clean Iterator Usage**
```java
// Always check hasNext() before next()
while (iterator.hasNext()) {
    Object element = iterator.next();
    // process
}
```

4. **Prefer forEach for Simple Iteration**
```java
// Clean and concise
list.forEach(System.out::println);
```

## Summary
- Collections Framework provides unified architecture for collections
- Solves the "different methods" problem of pre-Java 1.2 collections
- Iterable enables traversal, Collection adds manipulation
- Common methods across all collections improve developer productivity
- Collections utility class provides additional algorithms