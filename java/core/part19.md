# Comprehensive Notes: Operators in Java

---

## 1. Basic Terminology

### What is an Operator?
An **operator** indicates what action to perform (addition, subtraction, multiplication, etc.)

### What is an Operand?
An **operand** is the item on which the action is performed. Operands can be:
- **Variables**: `a + b` (where a and b are variables)
- **Constants**: `5 + 3` (where 5 and 3 are constant values)

### What is an Expression?
An **expression** consists of one or more operands and zero or more operators.

```
Example: 5 + 3 * 4 + a
         ↑   ↑   ↑   ↑
      operands and operators combined = Expression
```

---

## 2. Categories of Operators in Java

Java has **9 categories** of operators:

| # | Category | Example |
|---|----------|---------|
| 1 | Arithmetic Operators | `+`, `-`, `*`, `/`, `%` |
| 2 | Relational Operators | `==`, `!=`, `>`, `<`, `>=`, `<=` |
| 3 | Logical Operators | `&&`, `||` |
| 4 | Unary Operators | `++`, `--`, `!`, `+`, `-` |
| 5 | Assignment Operators | `=`, `+=`, `-=`, `*=`, `/=` |
| 6 | Bitwise Operators | `&`, `|`, `^`, `~` |
| 7 | Bitwise Shift Operators | `<<`, `>>`, `>>>` |
| 8 | Ternary Operator | `? :` |
| 9 | Type Comparison Operator | `instanceof` |

---

## 3. Arithmetic Operators

Basic mathematical operations:

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| `+` | Addition | `5 + 3` | `8` |
| `-` | Subtraction | `5 - 3` | `2` |
| `*` | Multiplication | `5 * 3` | `15` |
| `/` | Division | `5 / 2` | `2` (integer division) |
| `%` | Modulus | `5 % 2` | `1` (remainder) |

```java
public class ArithmeticExample {
    public static void main(String[] args) {
        int a = 5, b = 2;
        System.out.println("Division: " + (a / b));  // Output: 2
        System.out.println("Modulus: " + (a % b));   // Output: 1
    }
}
```

---

## 4. Relational Operators

**Purpose:** Compares two operands and returns `true` or `false`

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `==` | Equal to | `4 == 7` | `false` |
| `!=` | Not equal to | `4 != 7` | `true` |
| `>` | Greater than | `4 > 7` | `false` |
| `<` | Less than | `4 < 7` | `true` |
| `>=` | Greater than or equal | `4 >= 7` | `false` |
| `<=` | Less than or equal | `4 <= 7` | `true` |

```java
int a = 4, b = 7;
System.out.println(a == b);  // false
System.out.println(a != b);  // true
System.out.println(a < b);   // true
```

---

## 5. Logical Operators

**Purpose:** Combines two or more conditions and returns `true` or `false`

### Logical AND (`&&`)

| Condition 1 | Condition 2 | Result |
|-------------|-------------|--------|
| false | false | false |
| false | true | false |
| true | false | false |
| true | true | **true** |

> **Short-circuit evaluation:** If first condition is `false`, second condition is NOT evaluated (result is already `false`)

### Logical OR (`||`)

| Condition 1 | Condition 2 | Result |
|-------------|-------------|--------|
| false | false | false |
| false | true | **true** |
| true | false | **true** |
| true | true | **true** |

> **Short-circuit evaluation:** If first condition is `true`, second condition is NOT evaluated (result is already `true`)

### Example:
```java
int a = 4, b = 7;

// Logical AND
System.out.println((a < 3) && (a != b));  // false && true = false
System.out.println((a > 3) && (a != b));  // true && true = true

// Logical OR
System.out.println((a < 3) || (a != b));  // false || true = true
System.out.println((a > 3) || (a != b));  // true || (not evaluated) = true
```

---

## 6. Unary Operators

**Purpose:** Works on a **single operand** only

| Operator | Name | Description |
|----------|------|-------------|
| `++` | Increment | Increases value by 1 |
| `--` | Decrement | Decreases value by 1 |
| `+` | Unary Plus | Makes value positive |
| `-` | Unary Minus | Makes value negative |
| `!` | Logical NOT | Inverts boolean value |

### Increment/Decrement - Prefix vs Postfix

| Type | Syntax | Behavior |
|------|--------|----------|
| **Postfix** | `a++` or `a--` | Return value FIRST, then increment/decrement |
| **Prefix** | `++a` or `--a` | Increment/decrement FIRST, then return value |

### Example:
```java
int a = 5;

// Postfix: Return first, then increment
System.out.println(a++);  // Prints: 5, then a becomes 6

// Prefix: Increment first, then return
System.out.println(++a);  // a becomes 7, Prints: 7

// Postfix decrement
System.out.println(a--);  // Prints: 7, then a becomes 6

// Prefix decrement
System.out.println(--a);  // a becomes 5, Prints: 5
```

### Logical NOT (`!`)
```java
boolean flag = true;
System.out.println(!flag);  // Output: false (inverts the value)
```

### Unary Plus and Minus
```java
int a = 5;
System.out.println(-a);  // Output: -5
System.out.println(+a);  // Output: 5
```

---

## 7. Assignment Operators

**Purpose:** Assigns a new value to a variable

| Operator | Example | Equivalent To |
|----------|---------|---------------|
| `=` | `a = 5` | Assign 5 to a |
| `+=` | `a += 5` | `a = a + 5` |
| `-=` | `a -= 5` | `a = a - 5` |
| `*=` | `a *= 5` | `a = a * 5` |
| `/=` | `a /= 5` | `a = a / 5` |
| `%=` | `a %= 5` | `a = a % 5` |

### Example:
```java
int a = 5;
int variable = 0;

variable += a;   // variable = 0 + 5 = 5
System.out.println(variable);  // 5

variable -= 3;   // variable = 5 - 3 = 2
System.out.println(variable);  // 2

variable *= a;   // variable = 2 * 5 = 10
System.out.println(variable);  // 10

variable /= a;   // variable = 10 / 5 = 2
System.out.println(variable);  // 2
```

---

## 8. Bitwise Operators (Important!)

**Purpose:** Works on bit level (1s and 0s) - Very fast as processor directly supports binary operations

| Operator | Name | Symbol |
|----------|------|--------|
| AND | Bitwise AND | `&` |
| OR | Bitwise OR | `|` |
| XOR | Bitwise XOR | `^` |
| NOT | Bitwise NOT | `~` |

### Bitwise AND (`&`)
Returns 1 only when **BOTH bits are 1**

| Bit A | Bit B | A & B |
|-------|-------|-------|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | **1** |

### Bitwise OR (`|`)
Returns 1 when **ANY bit is 1**

| Bit A | Bit B | A \| B |
|-------|-------|--------|
| 0 | 0 | 0 |
| 0 | 1 | **1** |
| 1 | 0 | **1** |
| 1 | 1 | **1** |

### Bitwise XOR (`^`)
Returns 1 when bits are **DIFFERENT**

| Bit A | Bit B | A ^ B |
|-------|-------|-------|
| 0 | 0 | 0 |
| 0 | 1 | **1** |
| 1 | 0 | **1** |
| 1 | 1 | 0 |

### Example:
```java
int a = 4;  // Binary: 0100
int b = 6;  // Binary: 0110

// Bitwise AND
//   0100
// & 0110
// ------
//   0100 = 4
System.out.println(a & b);  // Output: 4

// Bitwise OR
//   0100
// | 0110
// ------
//   0110 = 6
System.out.println(a | b);  // Output: 6

// Bitwise XOR
//   0100
// ^ 0110
// ------
//   0010 = 2
System.out.println(a ^ b);  // Output: 2
```

---

### Bitwise NOT (`~`) - Detailed Explanation

**Purpose:** Flips all bits (0 → 1, 1 → 0)

#### Important Concepts:

1. **MSB (Most Significant Bit):** Leftmost bit - determines the **sign**
    - MSB = 0 → Positive number
    - MSB = 1 → Negative number

2. **LSB (Least Significant Bit):** Rightmost bit

3. **Formula for Bitwise NOT:** `~n = -(n + 1)`

#### Example:
```java
int a = 4;  // Binary: 0100
System.out.println(~a);  // Output: -5
```

**Why -5?**

```
Original:     0100 (which is +4)
After NOT:    1011

Computing decimal value of 1011:
Position:     3  2  1  0
Bits:         1  0  1  1
Values:      -8  0  2  1  (MSB is negative because it's sign bit)

Result: -8 + 0 + 2 + 1 = -5
```

**Using Formula:** `~4 = -(4 + 1) = -5` ✓

#### Two's Complement (Finding negative representation):

To find `-5` in binary:
1. Start with `5` = `0101`
2. **First Complement** (flip bits): `1010`
3. **Second Complement** (add 1): `1010 + 1 = 1011`

So `-5` = `1011` ✓

> **Note:** In Java, integers are **signed** (no unsigned integers). The MSB always represents the sign.

---

## 9. Bitwise Shift Operators (Important!)

**Purpose:** Shifts bits of a number left or right

| Operator | Name | Symbol |
|----------|------|--------|
| Left Shift | Shift bits left | `<<` |
| Right Shift (Signed) | Shift bits right | `>>` |
| Right Shift (Unsigned) | Shift bits right | `>>>` |

### Memory Trick:
Look at where the arrows point:
- `<<` arrows point **LEFT** → Left Shift
- `>>` arrows point **RIGHT** → Right Shift

---

### Left Shift (`<<`)

- Shifts bits to the **left**
- Fills empty positions on **right with 0**
- **Effect:** Multiplies by 2 for each shift

```
a = 4         →  0100
a << 1        →  1000 = 8  (4 × 2)
a << 2        →  10000 = 16 (4 × 4)
```

```java
int a = 4;
System.out.println(a << 1);  // Output: 8  (4 * 2^1)
System.out.println(a << 2);  // Output: 16 (4 * 2^2)
```

> **Note:** There is NO unsigned left shift because LSB (which gets filled) has no significance for sign.

---

### Right Shift - Signed (`>>`)

- Shifts bits to the **right**
- Fills empty positions on **left with the sign bit (MSB)**
- **Effect:** Divides by 2 for each shift (preserves sign)

```
a = 4         →  0100
a >> 1        →  0010 = 2  (4 ÷ 2)
```

For negative numbers:
```
a = -5        →  1011
a >> 1        →  1101 (MSB filled with 1, preserving negative sign)
```

---

### Right Shift - Unsigned (`>>>`)

- Shifts bits to the **right**
- **Always fills empty positions with 0** (regardless of sign)
- Can turn negative numbers positive

```
a = -5        →  1011
a >>> 1       →  0101 (MSB filled with 0, not sign bit)
```

---

### Summary of Shift Operators:

| Operation | Fill Behavior | Mathematical Effect |
|-----------|---------------|---------------------|
| `<<` (Left) | Fill right with 0 | Multiply by 2ⁿ |
| `>>` (Signed Right) | Fill left with sign bit | Divide by 2ⁿ (preserve sign) |
| `>>>` (Unsigned Right) | Fill left with 0 | Divide by 2ⁿ (always positive) |

---

## 10. Ternary Operator

**Purpose:** Mimics if-else condition in a single line

### Syntax:
```
result = (condition) ? expressionIfTrue : expressionIfFalse;
```

### Equivalent if-else:
```java
// Using if-else
int max;
if (a > b) {
    max = a;
} else {
    max = b;
}

// Using ternary operator
int max = (a > b) ? a : b;
```

### Example:
```java
int a = 4, b = 5;
int maxValue = (a > b) ? a : b;
System.out.println(maxValue);  // Output: 5

// Another example
int maxValue2 = (b > a) ? b : a;
System.out.println(maxValue2);  // Output: 5
```

---

## 11. Type Comparison Operator (`instanceof`)

**Purpose:** Checks if an object is an instance of a specific class

### Syntax:
```java
object instanceof ClassName
```

**Returns:** `true` or `false`

### Example:
```java
class Parent { }
class ChildClass1 extends Parent { }
class ChildClass2 extends Parent { }

public class Main {
    public static void main(String[] args) {
        // Create ChildClass2 object
        Parent obj = new ChildClass2();
        
        System.out.println(obj instanceof ChildClass2);  // true
        System.out.println(obj instanceof ChildClass1);  // false
        System.out.println(obj instanceof Parent);       // true
        
        // With String
        String val = "Hello";
        System.out.println(val instanceof String);       // true
        
        // With Object type
        Object unknownObj = new ChildClass1();
        System.out.println(unknownObj instanceof ChildClass2);  // false
        System.out.println(unknownObj instanceof ChildClass1);  // true
    }
}
```

### Common Use Cases:
1. **Parent-Child relationships:** Determining which child type an object is
2. **Object type checking:** When receiving `Object` type and need to identify actual class

---

## 12. Operator Precedence and Associativity

### Precedence Table (High to Low Priority):

| Priority | Operator Type | Operators | Associativity |
|----------|---------------|-----------|---------------|
| 1 (Highest) | Postfix | `expr++`, `expr--` | Left to Right |
| 2 | Unary | `++expr`, `--expr`, `+`, `-`, `~`, `!` | Right to Left |
| 3 | Multiplicative | `*`, `/`, `%` | Left to Right |
| 4 | Additive | `+`, `-` | Left to Right |
| 5 | Shift | `<<`, `>>`, `>>>` | Left to Right |
| 6 | Relational | `<`, `>`, `<=`, `>=`, `instanceof` | Left to Right |
| 7 | Equality | `==`, `!=` | Left to Right |
| 8 | Bitwise AND | `&` | Left to Right |
| 9 | Bitwise XOR | `^` | Left to Right |
| 10 | Bitwise OR | `|` | Left to Right |
| 11 | Logical AND | `&&` | Left to Right |
| 12 | Logical OR | `||` | Left to Right |
| 13 | Ternary | `? :` | Right to Left |
| 14 (Lowest) | Assignment | `=`, `+=`, `-=`, etc. | Right to Left |

### What is Associativity?
When two operators have the **same precedence**, associativity determines evaluation order.

### Example 1: Different Precedence
```java
int result = 5 + 2 * 3;
// Multiplication (*) has higher precedence than Addition (+)
// So: 5 + (2 * 3) = 5 + 6 = 11
```

### Example 2: Same Precedence (Left to Right)
```java
int result = 5 * 2 / 2;
// * and / have same precedence
// Associativity: Left to Right
// So: (5 * 2) / 2 = 10 / 2 = 5
```

### Example 3: Assignment (Right to Left)
```java
int a, b, c;
a = b = c = 5;
// Assignment associativity: Right to Left
// So: a = (b = (c = 5))
// c gets 5, then b gets 5, then a gets 5
```

---

## 13. Complex Expression Evaluation

### Steps to Evaluate:
1. Replace variables with their current values
2. Handle prefix operations (modify value, then use)
3. Handle postfix operations (use value, then modify)
4. Apply precedence rules
5. Apply associativity for same-precedence operators

### Example:
```java
int a = 4;
int result = a + a++ + ++a * --a + a--;
```

**Step-by-step evaluation:**

| Expression | Value Used | a After |
|------------|------------|---------|
| `a` | 4 | 4 |
| `a++` | 4 (postfix: use then increment) | 5 |
| `++a` | 6 (prefix: increment then use) | 6 |
| `--a` | 5 (prefix: decrement then use) | 5 |
| `a--` | 5 (postfix: use then decrement) | 4 |

**Calculation:**
```
= 4 + 4 + (6 * 5) + 5
= 4 + 4 + 30 + 5
= 43
```

Final value of `a` = 4

---

## 14. Quick Reference Summary

| Operator Type | Key Points |
|---------------|------------|
| **Arithmetic** | Basic math: `+`, `-`, `*`, `/`, `%` |
| **Relational** | Compare and return `true`/`false` |
| **Logical** | Combine conditions; short-circuit evaluation |
| **Unary** | Single operand; prefix vs postfix matters |
| **Assignment** | `a += b` equals `a = a + b` |
| **Bitwise** | Operates on bits; `~n = -(n+1)` |
| **Shift** | `<<` multiply by 2ⁿ; `>>` divide by 2ⁿ |
| **Ternary** | `condition ? ifTrue : ifFalse` |
| **instanceof** | Type checking; returns boolean |

---

## 15. Practice Problem

```java
int x = 2;
int z = ++x + ++x / x++ - 1;
// What is the value of z?
```

**Try solving this using the steps above!**

---