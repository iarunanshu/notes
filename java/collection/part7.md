# Java Streams - Complete Notes

## Overview
- **Stream**: A pipeline through which data elements pass and can perform various operations (sorting, filtering, etc.)
- Most useful for **bulk processing** with capability for parallel processing
- Real benefit comes when dealing with large datasets

## Three Steps of Stream Processing

### Step 1: Create a Stream
- Open/create a stream from a data source (collection, array, etc.)
- Original data is never modified

### Step 2: Intermediate Operations (0 or more)
- Transform stream into another stream
- **Lazy in nature** - only executed when terminal operation is invoked
- Can chain multiple operations together
- Examples: filter, map, sorted, distinct

### Step 3: Terminal Operation (exactly 1)
- Triggers processing of the stream
- Produces final result
- Closes the stream after execution
- Examples: collect, count, forEach, reduce

## Ways to Create Streams

### 1. From Collection
```java
List<Integer> salaries = Arrays.asList(3000, 4000, 10000);
Stream<Integer> stream = salaries.stream();
```

### 2. From Array
```java
Integer[] array = {3000, 4000, 10000};
Stream<Integer> stream = Arrays.stream(array);
```

### 3. Using Static Method
```java
Stream<Integer> stream = Stream.of(3000, 4000, 10000);
```

### 4. Using Stream Builder
```java
Stream.Builder<Integer> builder = Stream.builder();
builder.add(1000).add(2000);
Stream<Integer> stream = builder.build();
```

### 5. Using Stream.iterate()
```java
Stream<Integer> stream = Stream.iterate(1000, n -> n + 5000).limit(5);
// Creates: 1000, 6000, 11000, 16000, 21000
```

## Intermediate Operations

### 1. **filter()**
- Filters elements based on predicate
- Returns stream with elements matching condition
```java
stream.filter(salary -> salary > 3000)
```

### 2. **map()**
- Transforms each element
- Uses Function interface (accepts one value, returns one value)
```java
stream.map(name -> name.toLowerCase())
```

### 3. **flatMap()**
- Flattens complex collections (e.g., List of Lists)
- Converts nested structure to single stream
```java
listOfLists.stream().flatMap(list -> list.stream())
```

### 4. **distinct()**
- Removes duplicate elements from stream
```java
stream.distinct()
```

### 5. **sorted()**
- Sorts elements in natural order or using Comparator
```java
stream.sorted() // Natural order
stream.sorted((v1, v2) -> v2 - v1) // Descending order
```

### 6. **peek()**
- Performs action on each element without modifying stream
- Useful for debugging/printing
```java
stream.peek(System.out::println)
```

### 7. **limit()**
- Truncates stream to specified size
```java
stream.limit(3) // Keep only first 3 elements
```

### 8. **skip()**
- Skips first n elements
```java
stream.skip(3) // Skip first 3 elements
```

### 9. **mapToInt(), mapToLong(), mapToDouble()**
- Converts to primitive stream types
- Returns IntStream, LongStream, or DoubleStream
```java
stringList.stream().mapToInt(Integer::parseInt)
```

## Terminal Operations

### 1. **forEach()**
- Performs action on each element
- Does not return value
```java
stream.forEach(System.out::println)
```

### 2. **toArray()**
- Converts stream to array
```java
Object[] array = stream.toArray();
Integer[] intArray = stream.toArray(Integer[]::new);
```

### 3. **reduce()**
- Performs associative aggregation (sum, multiply, etc.)
- Returns Optional
```java
Optional<Integer> sum = stream.reduce((v1, v2) -> v1 + v2);
```

### 4. **collect()**
- Collects stream elements into collection
```java
List<Integer> list = stream.collect(Collectors.toList());
```

### 5. **min() / max()**
- Finds minimum/maximum based on comparator
```java
Optional<Integer> min = stream.min((v1, v2) -> v1 - v2);
```

### 6. **count()**
- Returns number of elements in stream
```java
long count = stream.count();
```

### 7. **anyMatch() / allMatch() / noneMatch()**
- Checks if elements match predicate
- Returns boolean
```java
boolean hasAny = stream.anyMatch(value -> value > 3);
```

### 8. **findFirst() / findAny()**
- findFirst: Returns first element
- findAny: Returns any random element
```java
Optional<Integer> first = stream.findFirst();
```

## Important Concepts

### Lazy Evaluation
- Intermediate operations are **lazy** - only executed when terminal operation is invoked
- Stream doesn't process until terminal operation triggers it

### Sequential Processing
- Stream processes elements one at a time through the pipeline
- For operations not requiring all data (filter, map), element passes through entire pipeline before next element starts
- Operations like sorted() require all elements before proceeding

### Stream Reusability
- **Streams can only be used once**
- After terminal operation, stream is closed
- Attempting to reuse throws: "stream has already been operated upon or closed"

## Parallel Streams

### Creating Parallel Stream
```java
numbers.parallelStream() // Instead of numbers.stream()
```

### Key Points
- Uses **Fork-Join Pool** technique
- Divides task into subtasks that run concurrently
- Takes advantage of multi-core CPUs
- Faster for bulk processing
- Uses **Spliterator** with trySplit() method to divide data

### Performance Comparison
- Sequential: Processes elements one by one
- Parallel: Processes multiple elements simultaneously
- Real benefit seen with large datasets

## Best Practices
1. Use streams for bulk data processing
2. Chain operations for readability
3. Prefer method references where possible
4. Be cautious with parallel streams for small datasets
5. Remember streams don't modify original data
6. Terminal operation must be present to execute stream

## Interview Key Points
- Understand three steps of stream processing
- Know difference between intermediate and terminal operations
- Understand lazy evaluation
- Be able to explain parallel vs sequential processing
- Know that streams cannot be reused after terminal operation
- Understand that original collection is never modified