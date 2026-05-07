# Comprehensive Notes: Control Flow Statements in Java

---

## Overview

Control Flow Statements are divided into **3 types**:

| Type | Purpose | Examples |
|------|---------|----------|
| **Decision Making Statements** | Execute code based on conditions | `if`, `if-else`, `switch` |
| **Iterative Statements (Loops)** | Repeat code multiple times | `for`, `while`, `do-while`, `for-each` |
| **Branching Statements** | Control loop execution | `break`, `continue` |

> **Note:** Switch Expression was introduced in **Java 12**. Earlier JDK versions won't support it.

---

# Part 1: Decision Making Statements

---

## 1. Simple If Statement (if-then)

**Purpose:** Execute code block only if condition is `true`

### Syntax:
```java
if (condition) {
    // Code executes only if condition is true
}
```

### Flow:
```
    [Condition]
        |
    Is True?
      /    \
   Yes      No
    |        |
 Execute   Skip
  Block    Block
    |        |
    +---+---+
        |
   Rest of Code
```

### Example:
```java
public class Main {
    public static void main(String[] args) {
        int value = 10;
        
        if (value > 8) {
            System.out.println("Only executes when value > 8");
        }
        
        System.out.println("Rest of code executes always");
    }
}
```

**Output:**
```
Only executes when value > 8
Rest of code executes always
```

---

## 2. If-Else Statement

**Purpose:** Execute one block if condition is `true`, another block if `false`

### Syntax:
```java
if (condition) {
    // Executes if condition is TRUE
} else {
    // Executes if condition is FALSE
}
```

### Example:
```java
int value = 7;

if (value > 8) {
    System.out.println("Value is greater than 8");
} else {
    System.out.println("Value is less than or equal to 8");
}
```

**Output:**
```
Value is less than or equal to 8
```

---

## 3. If-Else-If Ladder

**Purpose:** Check multiple conditions in sequence (top to bottom)

### Syntax:
```java
if (condition1) {
    // Executes if condition1 is true
} else if (condition2) {
    // Executes if condition2 is true
} else if (condition3) {
    // Executes if condition3 is true
} else {
    // Executes if NO condition is true (optional)
}
```

### Flow:
```
      [condition1]
           |
       Is True? --Yes--> Execute Block 1 --> Exit
           |
          No
           |
      [condition2]
           |
       Is True? --Yes--> Execute Block 2 --> Exit
           |
          No
           |
      [condition3]
           |
       Is True? --Yes--> Execute Block 3 --> Exit
           |
          No
           |
    [else block] --> Execute Default --> Exit
```

### Example:
```java
int value = 13;

if (value == 1) {
    System.out.println("Value is 1");
} else if (value == 2) {
    System.out.println("Value is 2");
} else if (value == 3) {
    System.out.println("Value is 3");
} else {
    System.out.println("Value is " + value);  // This executes
}
```

**Output:**
```
Value is 13
```

> **Note:** The `else` block is optional. You can have just `if` and `else if` without `else`.

---

## 4. Nested If Statement

**Purpose:** Place if-else statements inside other if-else blocks

### Syntax:
```java
if (condition1) {
    // Outer if block
    if (condition2) {
        // Inner if block
    } else {
        // Inner else block
    }
} else {
    // Outer else block
}
```

### Example:
```java
int value = 13;

if (value > 8) {
    System.out.println("Value is greater than 8");
    
    // Nested if-else
    if (value < 15) {
        System.out.println("Value is greater than 8 but less than 15");
    } else {
        System.out.println("Value is 15 or greater");
    }
} else {
    System.out.println("Value is 8 or less");
}
```

**Output:**
```
Value is greater than 8
Value is greater than 8 but less than 15
```

> **Note:** There's no limit on nesting levels. You can have if-else inside if-else inside if-else, etc.

---

## 5. Switch Statement

**Purpose:** Similar to if-else-if ladder, but cleaner for multiple equality checks

### Syntax:
```java
switch (expression) {
    case value1:
        // Code for value1
        break;
    case value2:
        // Code for value2
        break;
    case value3:
        // Code for value3
        break;
    default:
        // Code if no case matches
}
```

### Flow Diagram:
```
      [Expression]
           |
    Match case 1? --Yes--> Execute Code 1 --> break --> Exit
           |
          No
           |
    Match case 2? --Yes--> Execute Code 2 --> break --> Exit
           |
          No
           |
    Match case 3? --Yes--> Execute Code 3 --> break --> Exit
           |
          No
           |
       [default] --> Execute Default Code --> Exit
```

### Basic Example:
```java
int a = 1, b = 2;

switch (a + b) {  // Expression evaluates to 3
    case 1:
        System.out.println("a + b is 1");
        break;
    case 2:
        System.out.println("a + b is 2");
        break;
    case 3:
        System.out.println("a + b is 3");  // This executes
        break;
    default:
        System.out.println("a + b is " + (a + b));
}
```

**Output:**
```
a + b is 3
```

---

### Important: The `break` Statement in Switch

**Without `break`:** Code "falls through" to the next case

### Example - Without Break (Fall-through):
```java
int a = 1, b = 2;  // a + b = 3

switch (a + b) {
    case 1:
        System.out.println("a + b is 1");
    case 2:
        System.out.println("a + b is 2");
    case 3:
        System.out.println("a + b is 3");  // Matches here
        // No break - falls through!
    case 4:
        System.out.println("a + b is 4");  // Also executes!
        // No break - falls through!
    default:
        System.out.println("a + b is " + (a + b));  // Also executes!
}
```

**Output:**
```
a + b is 3
a + b is 4
a + b is 3
```

> **Warning:** Forgetting `break` is a common bug. Always use `break` unless fall-through is intentional.

---

### Default Placement

`default` can be placed **anywhere** in the switch (not just at the end)

### Example - Default in Middle:
```java
int a = 1, b = 2;  // a + b = 3

switch (a + b) {
    case 1:
        System.out.println("a + b is 1");
        break;
    case 2:
        System.out.println("a + b is 2");
        break;
    default:
        System.out.println("a + b is " + (a + b));
        break;  // Break is REQUIRED here if not at end!
    case 4:
        System.out.println("a + b is 4");
        break;
}
```

> **Important:** If `default` is NOT at the end, you MUST use `break` to prevent fall-through.

---

### Combining Cases

Multiple cases can share the same code block:

### Method 1 - Stacked Cases:
```java
String month = "March";

switch (month) {
    case "January":
    case "February":
    case "March":
        System.out.println("Month is in Quarter 1");
        break;
    case "April":
    case "May":
    case "June":
        System.out.println("Month is in Quarter 2");
        break;
    // ... more cases
}
```

### Method 2 - Comma-Separated (Java 14+):
```java
switch (month) {
    case "January", "February", "March":
        System.out.println("Month is in Quarter 1");
        break;
    case "April", "May", "June":
        System.out.println("Month is in Quarter 2");
        break;
}
```

---

### Switch Statement Rules

| Rule | Description |
|------|-------------|
| **No Duplicate Cases** | Two cases cannot have the same value |
| **Type Matching** | Expression type and case value type must be same |
| **Literals/Constants Only** | Case values must be literals or `final` constants |
| **All Cases Optional** | Not required to handle all possible values |
| **Nested Switch Allowed** | Can have switch inside switch |

### Case Values Must Be Constants:
```java
int value = 1;

// ❌ ERROR - variable is not constant
switch (2 + 1 - 2) {
    case value:  // Error: constant expression required
        break;
}

// ✅ CORRECT - use final keyword
final int value = 1;

switch (2 + 1 - 2) {
    case value:  // Works! value is now constant
        break;
}
```

---

### Supported Data Types in Switch

**Total: 10 Data Types**

| Category | Types |
|----------|-------|
| **Primitive** | `int`, `short`, `byte`, `char` |
| **Wrapper Classes** | `Integer`, `Short`, `Byte`, `Character` |
| **Others** | `String`, `Enum` |

> **Not Supported:** `float`, `double`, `long`, `boolean`

### Example with Enum:
```java
enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }

Day day = Day.FRIDAY;
int output = 0;

switch (day) {
    case MONDAY:
        output = 1;
        break;
    case TUESDAY:
        output = 2;
        break;
    // Not all cases handled - that's OK for switch statement
}

System.out.println(output);  // Output: 0 (no case matched)
```

---

## 6. Switch Expression (Java 12+)

**Purpose:** Switch that **returns a value** - cleaner, no break needed

### Key Differences from Switch Statement:

| Feature | Switch Statement | Switch Expression |
|---------|------------------|-------------------|
| Returns Value | ❌ No | ✅ Yes |
| Break Required | ✅ Yes | ❌ No |
| All Cases Required | ❌ No | ✅ Yes (or default) |
| Arrow Syntax | ❌ No | ✅ Yes |

---

### Using Arrow (`->`) Syntax:

```java
int value = 1;

String output = switch (value) {
    case 1 -> "One";
    case 2 -> "Two";
    case 3 -> "Three";
    default -> "Unknown";
};

System.out.println(output);  // Output: One
```

### Key Points:
- Arrow `->` automatically returns the value (no `break` needed)
- Must handle ALL possible cases (or use `default`)
- Expression ends with semicolon after closing brace

---

### Must Cover All Cases:

```java
int value = 1;

// ❌ ERROR - doesn't cover all cases
String output = switch (value) {
    case 1 -> "One";
    case 2 -> "Two";
    // Error: switch expression does not cover all possible input values
};

// ✅ CORRECT - has default
String output = switch (value) {
    case 1 -> "One";
    case 2 -> "Two";
    default -> "Unknown";
};
```

---

### Using `yield` for Code Blocks:

When you need multiple statements in a case, use `yield` to return:

```java
int value = 1;

String output = switch (value) {
    case 1 -> {
        // Multiple statements allowed in block
        System.out.println("Processing case 1");
        yield "One";  // yield returns the value
    }
    case 2 -> {
        System.out.println("Processing case 2");
        yield "Two";
    }
    default -> "Unknown";  // Single expression - no yield needed
};
```

### Summary - Arrow vs Yield:

| Scenario | Syntax |
|----------|--------|
| Single expression | `case 1 -> "One";` |
| Code block | `case 1 -> { /* code */ yield "One"; }` |

---

# Part 2: Iterative Statements (Loops)

---

## 1. For Loop

**Purpose:** Execute code a specific number of times

### Syntax:
```java
for (initialization; condition; increment/decrement) {
    // Code to execute
}
```

### Three Parts:
| Part | Purpose | Example |
|------|---------|---------|
| **Initialization** | Set starting value | `int i = 1` |
| **Condition** | Check before each iteration | `i <= 10` |
| **Increment/Decrement** | Update after each iteration | `i++` |

### Flow:
```
[Initialize] --> [Check Condition]
                      |
                  Is True?
                   /    \
                 Yes     No
                  |       |
            [Execute]   Exit
              Block     Loop
                  |
            [Increment]
                  |
                  +-------> [Check Condition]
```

### Example:
```java
for (int val = 1; val <= 10; val++) {
    System.out.println(val);
}
// Output: 1 2 3 4 5 6 7 8 9 10
```

### Execution Order:
1. `int val = 1` (initialize - runs once)
2. `val <= 10` (condition check - true)
3. Execute code block (print 1)
4. `val++` (increment - val becomes 2)
5. `val <= 10` (condition check - true)
6. ... repeat until condition is false

---

### Nested For Loops:

**Use Case:** 2D arrays, matrices, grids

```java
for (int x = 1; x <= 3; x++) {        // Outer loop
    for (int y = 1; y <= 3; y++) {    // Inner loop
        System.out.println("x=" + x + ", y=" + y);
    }
}
```

**Output:**
```
x=1, y=1
x=1, y=2
x=1, y=3
x=2, y=1
x=2, y=2
x=2, y=3
x=3, y=1
x=3, y=2
x=3, y=3
```

> **Note:** For each iteration of outer loop, inner loop runs completely.

### Matrix Visualization:
```
     y=1  y=2  y=3
x=1 [ 1 ][ 2 ][ 3 ]
x=2 [ 4 ][ 5 ][ 6 ]
x=3 [ 7 ][ 8 ][ 9 ]
```

---

## 2. While Loop

**Purpose:** Execute code while condition is true (check condition first)

### Syntax:
```java
// Initialize variable before loop
int value = 1;

while (condition) {
    // Code to execute
    // Increment/decrement inside loop
    value++;
}
```

### Example:
```java
int value = 1;

while (value <= 5) {
    System.out.println(value);
    value++;
}
// Output: 1 2 3 4 5
```

### Flow:
```
[Initialize]
     |
     v
[Check Condition] <----+
     |                 |
 Is True?              |
  /    \               |
Yes     No             |
 |       |             |
Execute  Exit          |
Block    Loop          |
 |                     |
[Increment] ---------->+
```

---

## 3. Do-While Loop

**Purpose:** Execute code **at least once**, then check condition

### Syntax:
```java
int value = 1;

do {
    // Code executes AT LEAST ONCE
    value++;
} while (condition);  // Note: semicolon at end!
```

### Example:
```java
int value = 1;

do {
    System.out.println(value);
    value++;
} while (value <= 5);
// Output: 1 2 3 4 5
```

### Key Difference from While:

| While Loop | Do-While Loop |
|------------|---------------|
| Checks condition FIRST | Executes code FIRST |
| May execute 0 times | Executes AT LEAST once |

### Example - Condition False Initially:
```java
int value = 10;

// While loop - won't execute
while (value <= 5) {
    System.out.println("While: " + value);
    value++;
}
// Output: (nothing)

// Do-While loop - executes once
value = 10;
do {
    System.out.println("Do-While: " + value);
    value++;
} while (value <= 5);
// Output: Do-While: 10
```

---

## 4. For-Each Loop (Enhanced For Loop)

**Purpose:** Iterate over arrays or collections easily

### Syntax:
```java
for (dataType variable : arrayOrCollection) {
    // Use variable
}
```

### Example with Array:
```java
int[] numbers = {1, 2, 3, 4, 5};

for (int val : numbers) {
    System.out.println(val);
}
// Output: 1 2 3 4 5
```

### Flow:
```
[Get first element] --> [Assign to variable]
         |                      |
         |                  [Execute Block]
         |                      |
[Move to next element] <--------+
         |
   More elements?
      /    \
    Yes     No
     |       |
   Repeat   Exit
```

### Comparison - Regular For vs For-Each:

```java
int[] arr = {1, 2, 3, 4, 5};

// Regular for loop
for (int i = 0; i < arr.length; i++) {
    System.out.println(arr[i]);
}

// For-each loop (cleaner)
for (int val : arr) {
    System.out.println(val);
}
```

> **Note:** For-each is read-only. You cannot modify array elements directly.

---

## Loop Comparison Summary

| Feature | for | while | do-while | for-each |
|---------|-----|-------|----------|----------|
| **Use When** | Known iterations | Unknown iterations | At least once | Iterating collections |
| **Condition Check** | Before | Before | After | N/A |
| **Min Executions** | 0 | 0 | 1 | 0 |
| **Index Access** | ✅ Yes | ✅ Yes | ✅ Yes | ❌ No |

---

# Part 3: Branching Statements

---

## 1. Break Statement

**Purpose:** Exit from the **immediate** loop or switch

### Example:
```java
for (int val = 1; val <= 10; val++) {
    if (val == 3) {
        break;  // Exit loop when val is 3
    }
    System.out.println(val);
}
System.out.println("Loop ended");
```

**Output:**
```
1
2
Loop ended
```

### Break in Nested Loops:
`break` only exits the **innermost** loop

```java
for (int i = 1; i <= 3; i++) {           // Outer loop
    for (int j = 1; j <= 3; j++) {       // Inner loop
        if (j == 2) {
            break;  // Only breaks inner loop
        }
        System.out.println("i=" + i + ", j=" + j);
    }
}
```

**Output:**
```
i=1, j=1
i=2, j=1
i=3, j=1
```

### Visual Flow:
```
i=1: j=1 (print) → j=2 (break inner) → back to outer
i=2: j=1 (print) → j=2 (break inner) → back to outer
i=3: j=1 (print) → j=2 (break inner) → back to outer
Exit outer loop
```

---

## 2. Continue Statement

**Purpose:** Skip current iteration and continue with next iteration

### Example:
```java
for (int val = 1; val <= 10; val++) {
    if (val == 3) {
        continue;  // Skip printing 3
    }
    System.out.println(val);
}
```

**Output:**
```
1
2
4
5
6
7
8
9
10
```

> **Note:** Only `3` is skipped, loop continues with other values.

---

## Break vs Continue Comparison

| Feature | Break | Continue |
|---------|-------|----------|
| **Action** | Exits the loop entirely | Skips current iteration only |
| **Loop continues?** | ❌ No | ✅ Yes |
| **Use case** | Stop when condition met | Skip specific values |

### Visual Comparison:
```
Loop: 1, 2, 3, 4, 5

With break at 3:     1, 2, [STOP]
With continue at 3:  1, 2, [SKIP], 4, 5
```

---

# Quick Reference Summary

## Decision Making Statements

| Statement | Use Case |
|-----------|----------|
| `if` | Execute if condition is true |
| `if-else` | Execute one of two blocks |
| `if-else-if` | Check multiple conditions |
| `nested if` | Complex conditional logic |
| `switch` | Multiple equality checks |
| `switch expression` | Return value from switch (Java 12+) |

## Loops

| Loop | Use Case |
|------|----------|
| `for` | Known number of iterations |
| `while` | Unknown iterations, check first |
| `do-while` | At least one execution needed |
| `for-each` | Iterate arrays/collections |

## Branching

| Statement | Effect |
|-----------|--------|
| `break` | Exit loop immediately |
| `continue` | Skip to next iteration |

---

## Common Interview Points

1. **Switch supported types:** `int`, `short`, `byte`, `char`, `Integer`, `Short`, `Byte`, `Character`, `String`, `Enum`

2. **Switch Expression (Java 12+):** Must cover all cases, uses `->` arrow, returns value

3. **Break in nested loops:** Only exits immediate/innermost loop

4. **Do-while vs While:** Do-while executes at least once

5. **For-each limitations:** Cannot modify elements, no index access

---