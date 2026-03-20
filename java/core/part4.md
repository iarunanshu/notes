# Float & Double in Memory (IEEE 754) - Notes

---

## 1. The Problem: Why Float Values Are Not Exact

```java
float f = 0.7f;
System.out.println(f);

// Expected: 0.7
// Actual:   0.6999999880790710
```

**Why does this happen?**
- Floats/Doubles are stored in **IEEE 754** format
- Some decimal numbers **cannot be exactly represented** in binary
- This is a **frequent Java interview question**

> **Industry Rule:** Never use `float`/`double` for precise calculations (especially currency). Use **BigDecimal** instead!

---

## 2. IEEE 754 Format Structure

### Float (32 bits)

```
┌──────┬──────────────────┬─────────────────────────────────────┐
│ Sign │     Exponent     │             Mantissa                │
│ 1 bit│      8 bits      │             23 bits                 │
└──────┴──────────────────┴─────────────────────────────────────┘
   ↓          ↓                        ↓
   0/1     Biased exp          Fractional part
(+/-)    (actual + 127)       (after decimal)
```

### Double (64 bits)

```
┌──────┬──────────────────┬─────────────────────────────────────────────────┐
│ Sign │     Exponent     │                   Mantissa                      │
│ 1 bit│     11 bits      │                   52 bits                       │
└──────┴──────────────────┴─────────────────────────────────────────────────┘
```

### Comparison:

| Component | Float (32-bit) | Double (64-bit) |
|-----------|----------------|-----------------|
| Sign bit | 1 bit | 1 bit |
| Exponent | 8 bits | 11 bits |
| Mantissa | 23 bits | 52 bits |
| **Bias** | **127** (2⁷ - 1) | **1023** (2¹⁰ - 1) |

---

## 3. Understanding the Components

### Sign Bit
- `0` = Positive number
- `1` = Negative number

### Exponent (with Bias)
- Stores the power of 2
- Uses **bias** instead of two's complement for negative exponents
- **Float bias = 127**
- **Double bias = 1023**
- Stored value = Actual exponent + Bias

### Mantissa (Significand)
- The fractional part after the decimal point
- Assumes a leading `1.` (normalized form)
- Only stores digits **after** the decimal point

---

## 4. Converting Decimal to IEEE 754 (Step-by-Step)

### Example 1: Store `4.125f` in Memory

#### Step 1: Convert to Binary

**Integer part (4):**
```
4 ÷ 2 = 2, remainder 0
2 ÷ 2 = 1, remainder 0
1 ÷ 2 = 0, remainder 1

Read bottom-up: 100
```

**Fractional part (0.125):**
```
0.125 × 2 = 0.25  → 0
0.25  × 2 = 0.50  → 0
0.50  × 2 = 1.00  → 1

Read top-down: 001
```

**Result:** `4.125` = `100.001` in binary

---

#### Step 2: Normalize (Scientific Notation)

Convert to form: `1.xxxxx × 2^exponent`

```
100.001 = 1.00001 × 2²
          ↑
    Move decimal 2 places left
    So exponent = +2
```

---

#### Step 3: Add Bias to Exponent

```
Biased Exponent = Actual Exponent + Bias
                = 2 + 127
                = 129
```

**Why Bias?**
- IEEE 754 doesn't use two's complement for exponent
- Bias allows representing negative exponents as positive numbers
- Example: If exponent is -3 → Stored as 127 + (-3) = 124

---

#### Step 4: Fill the 32 Bits

| Component | Value | Binary |
|-----------|-------|--------|
| Sign | Positive | `0` |
| Exponent | 129 | `10000001` |
| Mantissa | 00001 | `00001000000000000000000` |

**Final IEEE 754 representation of 4.125f:**
```
0 | 10000001 | 00001000000000000000000
```

---

## 5. Converting IEEE 754 Back to Decimal

### Formula:
```
Value = (-1)^sign × (1 + mantissa) × 2^(exponent - bias)
```

### Verify 4.125f:

```
Sign bit = 0 → (-1)⁰ = 1 (positive)

Exponent = 10000001 = 129
Actual exponent = 129 - 127 = 2

Mantissa = 00001000000000000000000
         = 2⁻⁵ = 0.03125

Value = 1 × (1 + 0.03125) × 2²
      = 1 × 1.03125 × 4
      = 4.125 ✓
```

---

## 6. Example 2: Why 0.7f Becomes 0.699999...

### Step 1: Convert 0.7 to Binary

```
0.7 × 2 = 1.4  → 1
0.4 × 2 = 0.8  → 0
0.8 × 2 = 1.6  → 1
0.6 × 2 = 1.2  → 1
0.2 × 2 = 0.4  → 0
0.4 × 2 = 0.8  → 0  ← Pattern repeats!
0.8 × 2 = 1.6  → 1
0.6 × 2 = 1.2  → 1
...
```

**Result:** `0.7` = `0.101100110011001100110011...` (repeating!)

> **Key Insight:** 0.7 has an **infinite repeating binary representation** - it can NEVER be exactly stored!

---

### Step 2: Normalize

```
0.1011001100110011... = 1.011001100110011... × 2⁻¹
                                              ↑
                              Move decimal 1 place right
                              Exponent = -1
```

---

### Step 3: Add Bias

```
Biased Exponent = -1 + 127 = 126
```

---

### Step 4: Store in 32 Bits

```
Sign: 0 (positive)

Exponent: 126 = 01111110

Mantissa: 01100110011001100110011 (23 bits - truncated!)
          ↑
          Infinite sequence gets CUT OFF here!
```

**Stored value:**
```
0 | 01111110 | 01100110011001100110011
```

---

### Step 5: Convert Back (What We Actually Get)

```
Sign = 0 → positive

Exponent = 126 - 127 = -1

Mantissa calculation:
Position:  -1   -2   -3   -4   -5   -6   -7   -8   -9  -10  -11  ...
Binary:     0    1    1    0    0    1    1    0    0    1    1   ...
Value:     0  +0.25+0.125+ 0  + 0 +0.015+0.007+ 0  + 0 +0.0009+...

Mantissa ≈ 0.399414... (approximate due to truncation)

Value = 1 × (1 + 0.399414...) × 2⁻¹
      = 1.399414... × 0.5
      ≈ 0.699707...
```

**That's why we get 0.699999... instead of 0.7!**

---

## 7. Mantissa Calculation Reference

For mantissa bits: `b₁b₂b₃b₄b₅...`

```
Mantissa = b₁×2⁻¹ + b₂×2⁻² + b₃×2⁻³ + b₄×2⁻⁴ + ...
```

| Position | Power | Decimal Value |
|----------|-------|---------------|
| 1st bit | 2⁻¹ | 0.5 |
| 2nd bit | 2⁻² | 0.25 |
| 3rd bit | 2⁻³ | 0.125 |
| 4th bit | 2⁻⁴ | 0.0625 |
| 5th bit | 2⁻⁵ | 0.03125 |
| ... | ... | ... |

---

## 8. Quick Conversion Summary

### Decimal → IEEE 754:

```
Step 1: Convert decimal to binary
        - Integer: Divide by 2, read remainders bottom-up
        - Fraction: Multiply by 2, read integers top-down

Step 2: Normalize to 1.xxxx × 2^exp

Step 3: Biased Exponent = Actual Exponent + Bias
        - Float bias = 127
        - Double bias = 1023

Step 4: Assemble: [Sign][Exponent][Mantissa]
```

### IEEE 754 → Decimal:

```
Value = (-1)^sign × (1 + mantissa) × 2^(exponent - bias)
```

---

## 9. Float vs Double Summary

| Aspect | Float | Double |
|--------|-------|--------|
| Size | 32 bits | 64 bits |
| Sign | 1 bit | 1 bit |
| Exponent | 8 bits | 11 bits |
| Mantissa | 23 bits | 52 bits |
| Bias | 127 | 1023 |
| Precision | ~7 decimal digits | ~15 decimal digits |
| Suffix | `f` or `F` | `d` or `D` (optional) |

---

## 10. Why Use BigDecimal?

### Problem with float/double:
```java
float a = 0.3f;
float b = 0.1f;
System.out.println(a - b);  // 0.20000002 (NOT 0.2!)

double x = 0.3;
double y = 0.1;
System.out.println(x - y);  // 0.19999999999999998
```

### Solution - BigDecimal:
```java
import java.math.BigDecimal;

BigDecimal a = new BigDecimal("0.3");
BigDecimal b = new BigDecimal("0.1");
System.out.println(a.subtract(b));  // 0.2 (EXACT!)
```

> **Always use BigDecimal for:**
> - Financial/Currency calculations
> - Any situation requiring exact decimal precision

---

## 11. Interview Key Points

1. **Float = 32-bit IEEE 754** (1 sign + 8 exponent + 23 mantissa)
2. **Double = 64-bit IEEE 754** (1 sign + 11 exponent + 52 mantissa)
3. **Bias used instead of two's complement** for exponent
    - Float bias = 127
    - Double bias = 1023
4. **Why 0.7f ≠ 0.7?** → Infinite repeating binary, gets truncated
5. **Numbers that CAN be exactly represented:** Powers of 2 and their sums
    - 0.5 (2⁻¹) ✓
    - 0.25 (2⁻²) ✓
    - 0.125 (2⁻³) ✓
    - 4.125 (4 + 0.125) ✓
6. **Numbers that CANNOT be exactly represented:** Most decimals
    - 0.1, 0.2, 0.3, 0.7, etc. ✗
7. **Solution:** Use **BigDecimal** for precision-critical calculations

---

## 12. Visual Summary

```
        STORING 0.7f IN MEMORY
        
0.7 (decimal)
        ↓
0.1011001100110011... (binary - INFINITE!)
        ↓
1.011001100110011... × 2⁻¹ (normalized)
        ↓
┌───┬──────────┬─────────────────────────┐
│ 0 │ 01111110 │ 01100110011001100110011 │
└───┴──────────┴─────────────────────────┘
 sign  exp=126      23 bits (TRUNCATED!)
        ↓
0.6999999880790710 (retrieved - NOT 0.7!)

        ═══════════════════════
        PRECISION LOST DUE TO
        TRUNCATION OF INFINITE
        BINARY REPRESENTATION
        ═══════════════════════
```