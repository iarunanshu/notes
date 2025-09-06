# Java Collections Framework - Part 2: Queue, PriorityQueue, Comparator & Comparable

## 1. Queue Interface

### Overview
- Child interface of Collection
- Generally follows **FIFO** (First In First Out) approach
- Exceptions exist (like PriorityQueue)
- Inherits all Collection methods plus additional queue-specific methods

### Queue Structure
```
Front (Head) ← [Element1] [Element2] [Element3] [Element4] ← Rear (Tail)
                    ↓                                ↑
                 Remove                           Insert
```

### Queue Methods (6 Additional Methods)

| Operation | Throws Exception | Returns Special Value |
|-----------|-----------------|----------------------|
| **Insert** | `add(e)` | `offer(e)` |
| **Remove** | `remove()` | `poll()` |
| **Examine** | `element()` | `peek()` |

#### Detailed Method Descriptions

```java
public class QueueMethodsExample {
    public static void main(String[] args) {
        Queue<Integer> queue = new LinkedList<>();
        
        // 1. add() vs offer() - INSERT
        queue.add(5);     // Returns true on success, throws exception on failure
        queue.offer(10);  // Returns true on success, false on failure
        // queue.add(null);  // Throws NullPointerException
        
        // 2. remove() vs poll() - REMOVE & RETURN
        Integer val1 = queue.remove(); // Removes head, throws exception if empty
        Integer val2 = queue.poll();   // Removes head, returns null if empty
        
        // 3. element() vs peek() - EXAMINE (don't remove)
        Integer head1 = queue.element(); // Returns head, throws exception if empty
        Integer head2 = queue.peek();    // Returns head, returns null if empty
    }
}
```

## 2. PriorityQueue

### Key Concepts
- **Two types**: Min Priority Queue (Min Heap) and Max Priority Queue (Max Heap)
- Uses **heap data structure** internally
- Elements ordered by:
    - Natural ordering (default)
    - Comparator (custom ordering)

### Default Behavior (Min Heap)

```java
public class MinPriorityQueueExample {
    public static void main(String[] args) {
        // Default: Natural ordering (ascending for integers = Min Heap)
        PriorityQueue<Integer> minPQ = new PriorityQueue<>();
        
        // Adding elements
        minPQ.add(5);
        minPQ.add(2);
        minPQ.add(8);
        minPQ.add(1);
        
        // Internal heap structure:
        //       1
        //      / \
        //     2   8
        //    /
        //   5
        
        // Level order traversal print: 1, 2, 8, 5
        System.out.println("Queue: " + minPQ); // [1, 2, 8, 5]
        
        // Polling (removes in min order)
        while (!minPQ.isEmpty()) {
            System.out.println(minPQ.poll()); // 1, 2, 5, 8
        }
    }
}
```

### Max Heap with Comparator

```java
public class MaxPriorityQueueExample {
    public static void main(String[] args) {
        // Max Heap: Using Comparator for reverse order
        PriorityQueue<Integer> maxPQ = new PriorityQueue<>(
            (a, b) -> b - a  // Reverse order comparator
        );
        
        maxPQ.add(5);
        maxPQ.add(2);
        maxPQ.add(8);
        maxPQ.add(1);
        
        // Internal heap structure:
        //       8
        //      / \
        //     2   5
        //    /
        //   1
        
        // Polling (removes in max order)
        while (!maxPQ.isEmpty()) {
            System.out.println(maxPQ.poll()); // 8, 5, 2, 1
        }
    }
}
```

### Time Complexities
| Operation | Time Complexity |
|-----------|----------------|
| add() | O(log n) - heapify |
| peek() | O(1) - top element |
| poll()/remove() | O(log n) - remove + heapify |
| remove(arbitrary) | O(n) - search + remove |

## 3. Comparator Interface

### Why Needed?
1. **Change natural ordering** (e.g., descending instead of ascending)
2. **Sort custom objects** (no natural ordering defined)

### The Problem

```java
// This works - primitive array
int[] primitiveArray = {5, 1, 8, 10};
Arrays.sort(primitiveArray); // Works! Sorts in ascending

// This fails - custom object array
Car[] carArray = new Car[3];
carArray[0] = new Car("SUV", "Petrol");
carArray[1] = new Car("Sedan", "Diesel");
carArray[2] = new Car("Hatchback", "CNG");
Arrays.sort(carArray); // ClassCastException: Car cannot be cast to Comparable
```

### Comparator Solution

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    // Returns: positive if o1 > o2
    //          zero if o1 == o2
    //          negative if o1 < o2
}
```

### Comparator Examples

#### 1. Basic Integer Sorting

```java
public class ComparatorExample {
    public static void main(String[] args) {
        Integer[] arr = {17, 3, 5, 1, 10};
        
        // Ascending order
        Arrays.sort(arr, (a, b) -> a - b);
        System.out.println(Arrays.toString(arr)); // [1, 3, 5, 10, 17]
        
        // Descending order
        Arrays.sort(arr, (b, a) -> b - a);
        System.out.println(Arrays.toString(arr)); // [17, 10, 5, 3, 1]
    }
}
```

#### 2. Custom Object Sorting

```java
class Car {
    String name;
    String type;
    
    Car(String name, String type) {
        this.name = name;
        this.type = type;
    }
}

public class CarSortingExample {
    public static void main(String[] args) {
        Car[] cars = {
            new Car("SUV", "Petrol"),
            new Car("Sedan", "Diesel"),
            new Car("Hatchback", "CNG")
        };
        
        // Sort by name (ascending)
        Arrays.sort(cars, (c1, c2) -> c1.name.compareTo(c2.name));
        // Order: Hatchback, Sedan, SUV
        
        // Sort by type (descending)
        Arrays.sort(cars, (c1, c2) -> c2.type.compareTo(c1.type));
        // Order: Petrol, Diesel, CNG
        
        // Multiple criteria sorting
        Arrays.sort(cars, (c1, c2) -> {
            int nameCompare = c1.name.compareTo(c2.name);
            if (nameCompare != 0) return nameCompare;
            return c1.type.compareTo(c2.type);
        });
    }
}
```

#### 3. Different Ways to Create Comparator

```java
// Method 1: Lambda Expression (Preferred)
Comparator<Car> comp1 = (c1, c2) -> c1.name.compareTo(c2.name);

// Method 2: Separate Comparator Class
class CarNameComparator implements Comparator<Car> {
    @Override
    public int compare(Car c1, Car c2) {
        return c1.name.compareTo(c2.name);
    }
}
Comparator<Car> comp2 = new CarNameComparator();

// Method 3: Anonymous Class
Comparator<Car> comp3 = new Comparator<Car>() {
    @Override
    public int compare(Car c1, Car c2) {
        return c1.name.compareTo(c2.name);
    }
};

// Method 4: Method Reference (Java 8+)
Comparator<Car> comp4 = Comparator.comparing(Car::getName);
```

## 4. Comparable Interface

### Overview
- Single method: `compareTo(T o)`
- Must be implemented by the class itself
- Provides **natural ordering** for objects

```java
public interface Comparable<T> {
    int compareTo(T o);
    // Returns: positive if this > o
    //          zero if this == o
    //          negative if this < o
}
```

### Comparable Example

```java
class Car implements Comparable<Car> {
    String name;
    String type;
    
    Car(String name, String type) {
        this.name = name;
        this.type = type;
    }
    
    @Override
    public int compareTo(Car other) {
        // Natural ordering by name
        return this.name.compareTo(other.name);
    }
}

public class ComparableExample {
    public static void main(String[] args) {
        List<Car> cars = new ArrayList<>();
        cars.add(new Car("SUV", "Petrol"));
        cars.add(new Car("Sedan", "Diesel"));
        cars.add(new Car("Hatchback", "CNG"));
        
        // No comparator needed - uses compareTo
        Collections.sort(cars);
        // Sorted by name: Hatchback, Sedan, SUV
    }
}
```

## 5. Comparator vs Comparable

### Key Differences

| Aspect | Comparable | Comparator |
|--------|-----------|------------|
| **Interface Location** | `java.lang` | `java.util` |
| **Method** | `compareTo(T o)` | `compare(T o1, T o2)` |
| **Implementation** | In the class itself | Separate class or lambda |
| **Sorting Options** | Single natural order | Multiple sorting orders |
| **Modification Required** | Must modify original class | No class modification |
| **Flexibility** | Fixed sorting logic | Dynamic sorting logic |
| **Parameters** | Single parameter | Two parameters |

### When to Use What?

#### Use Comparable when:
- You have control over the class
- There's one obvious natural ordering
- The ordering rarely changes

#### Use Comparator when:
- You can't modify the class
- Multiple sorting orders needed
- Sorting logic changes frequently
- Working with library classes

### Combined Example

```java
class Student implements Comparable<Student> {
    String name;
    int age;
    double gpa;
    
    // Natural ordering by name
    @Override
    public int compareTo(Student other) {
        return this.name.compareTo(other.name);
    }
}

public class SortingExample {
    public static void main(String[] args) {
        List<Student> students = getStudents();
        
        // 1. Natural ordering (Comparable)
        Collections.sort(students);
        
        // 2. Custom ordering by age (Comparator)
        Collections.sort(students, (s1, s2) -> s1.age - s2.age);
        
        // 3. Custom ordering by GPA descending
        Collections.sort(students, (s1, s2) -> 
            Double.compare(s2.gpa, s1.gpa));
        
        // 4. Using Comparator utility methods
        students.sort(Comparator.comparing(Student::getAge));
        students.sort(Comparator.comparing(Student::getGpa).reversed());
        
        // 5. Chain comparators
        students.sort(Comparator
            .comparing(Student::getGpa)
            .thenComparing(Student::getName));
    }
}
```

## Interview Key Points

### 1. PriorityQueue
- Default is **Min Heap** (natural ordering)
- For Max Heap, provide reverse comparator: `(a, b) -> b - a`
- Used for heap-based problems in coding interviews

### 2. Comparator
- **Functional interface** with `compare(T o1, T o2)`
- External to the class
- Multiple sorting strategies possible
- Lambda expressions make it concise

### 3. Comparable
- Must modify the class to implement
- Single sorting strategy
- `compareTo(T o)` method
- Defines natural ordering

### 4. Return Values
- **Positive**: First object is greater
- **Zero**: Objects are equal
- **Negative**: First object is smaller

### 5. Common Pitfall
```java
// Wrong - primitive index
list.remove(3);  // Removes element at index 3

// Right - object removal
list.remove(Integer.valueOf(3));  // Removes value 3
```

## Best Practices

1. **Prefer Comparator** for flexibility
2. **Use method references** when possible: `Comparator.comparing(Car::getName)`
3. **Chain comparators** for multi-level sorting
4. **Be consistent** between `equals()` and `compareTo()`
5. **Handle null values** explicitly in comparators

## Summary
- Queue provides FIFO operations with exception/null variants
- PriorityQueue implements heap data structure
- Comparator provides external comparison logic
- Comparable provides internal natural ordering
- Choose based on flexibility and modification requirements