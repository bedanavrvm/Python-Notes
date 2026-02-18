# Binary & Machine Interaction

Computers do not understand decimal numbers, text, or images directly. At the lowest level, every piece of data is represented as a series of 1s and 0s—binary. This chapter explores how these bits are organized, manipulated, and interpreted by hardware and software to perform complex computations.

---

## 1. Foundations of Number Systems

A number system is a way of representing values using a specific set of symbols. Modern computing relies on **positional notation**, where the value of a digit depends on its position relative to the base (radix).

### Positional Notation
In a base-$n$ system, the $i$-th position (starting from 0) has a weight of $n^i$.
- **Decimal (Base-10):** $123 = 1 \cdot 10^2 + 2 \cdot 10^1 + 3 \cdot 10^0$
- **Binary (Base-2):** $101_2 = 1 \cdot 2^2 + 0 \cdot 2^1 + 1 \cdot 2^0 = 5_{10}$

### Integer Ranges (Formal Derivation)
The range of values a system can represent depends on the number of bits ($N$) and the chosen representation.

- **Unsigned $N$-bit range:** $\mathbf{[0,\ 2^N - 1]}$.
  - With $N$ bits, there are $2^N$ possible combinations. Since we start at 0, the maximum is $2^N - 1$.
- **Signed $N$-bit range (Two's Complement):** $\mathbf{[-2^{N-1},\ 2^{N-1} - 1]}$.
  - One bit is reserved for the sign. The positive side is one less because it includes zero. The "extra" negative value comes from the fact that $0$ is considered non-negative.

### Common Bases in Computing
| Base | Name | Symbols | Usage |
|------|------|---------|-------|
| 2    | Binary | 0, 1 | Machine level storage/logic |
| 8    | Octal | 0–7 | Unix permissions (historically used for 36-bit systems) |
| 10   | Decimal| 0–9 | Human interaction |
| 16   | Hexadecimal | 0–9, A–F | Memory addresses, color codes, instruction sets |

---

## 2. Binary Arithmetic

Binary arithmetic follows the same principles as decimal arithmetic, but with only two digits.

### Addition Rules
- $0+0 = 0$
- $0+1 = 1$
- $1+0 = 1$
- $1+1 = 0$ (carry 1)
- $1+1+1 = 1$ (carry 1)

### Multiplication (Shift and Add)
Binary multiplication works like decimal long multiplication but is simpler since you only multiply by 0 or 1.
- Multiplying by **1** copies the value.
- Multiplying by **0** results in zero.
- Each partial product is shifted left by its position.

**Example ($13 \times 5$):**
```text
      1101  (13)
    x 0101  (5)
    ------
      1101  (1101 x 1)
     0000_  (1101 x 0, shifted)
    1101__  (1101 x 1, shifted)
    ------
   1000001  (65)
```

### Division (Shift and Subtract)
Binary division follows a "Shift and Subtract" approach similar to long division.
1. Align the divisor with the leftmost bits of the dividend.
2. If the divisor fits, put a **1** in the quotient and subtract.
3. If not, put a **0** and shift to the next bit.

**Example ($11 \div 3$):**
```text
Dividend: 1011 (11)
Divisor:  0011 (3)

1. Can 0011 fit into 1?   No (0)
2. Can 0011 fit into 10?  No (0)
3. Can 0011 fit into 101? Yes (1) -> 101 - 011 = 10
4. Bring down 1: 101
5. Can 0011 fit into 101? Yes (1) -> 101 - 011 = 10 (Remainder)

Quotient: 0011 (3), Remainder: 10 (2)
```

### Subtraction via Two's Complement
Hardware does not have a "subtraction" circuit separate from addition. To compute $A - B$, the CPU performs $A + (\sim B + 1)$.
- Any **carry-out** bit beyond the register width is simply discarded.
- This works because $2^N - B$ is the Two's Complement representation of $-B$ in a modular system.

### Overflow vs. Carry (CPU Flags)
Hardware tracks the outcome of arithmetic using dedicated flags in a status register:
- **Carry Flag (CF):** Set when an *unsigned* operation results in a carry-out. Indicates the result is $> 2^N - 1$.
- **Overflow Flag (OF):** Set when a *signed* operation results in a sign error (e.g., adding two positives produces a negative).
- **Zero Flag (ZF):** Set if the result is exactly zero.
- **Sign Flag (SF):** Set if the result is negative (matches the MSB).

> [!NOTE]
> CPU flags are used by conditional jump instructions. For example, x86 uses `jb` (Jump if Below) for unsigned and `jl` (Jump if Less) for signed comparisons.

---

## 3. Signed Number Representations

How do we represent negative numbers?

- **Sign-Magnitude:** Use the leftmost bit (MSB) as a sign (0 for +, 1 for -). Problems: Two zeros (+0 and -0), hardware complexity.
- **One’s Complement:** Invert all bits. Problem: Still has two zeros.
- **Two’s Complement:** The industry standard. Invert bits and add 1.

---

## 4. Two’s Complement Deep Dive

### Computing Negative Numbers
To find $-X$:
1. Write the binary for $X$.
2. Invert all bits (NOT operation).
3. Add 1 to the result.

### Key Characteristics
- **All 1s is -1:** In any bit-width, `111...11` represents -1.
- **Minimum Value Edge Case (INT_MIN):** The most negative number ($1 \dots 0$) has no positive counterpart. `abs(INT_MIN)` is problematic because $2^{N-1}$ exceeds the positive range.
- **Wraparound:** Adding 1 to the maximum signed integer ($01 \dots 1$) results in the minimum signed integer ($10 \dots 0$).

---

## 5. Signed vs. Unsigned & Type Rules

### Fixed-Width Integer Types
To write cross-platform code, systems programmers use `<stdint.h>` types:
- `int8_t`, `int32_t`: Guaranteed exact width.
- `uint64_t`: Unsigned 64-bit.
- `size_t`: Unsigned type for sizes/indices, width varies by platform (32-bit vs 64-bit).

**Data Models:**
- **ILP32:** `int`, `long`, and pointers are all 32-bit.
- **LP64:** `int` is 32-bit, `long` and pointers are 64-bit (standard for 64-bit Linux/macOS).

### Sign Extension
### Sign Extension
When promoting a smaller signed integer to a larger width (e.g., `int8_t` to `int32_t`), the CPU performs **Sign Extension** by copying the sign bit into all new upper bits to preserve the value.

#### Sign Extension Example
Converting an 8-bit `-5` to 16-bit:
- **8-bit -5:** `11111011`
- **16-bit -5:** `11111111 11111011` (MSB `1` is copied to the left)

Converting an 8-bit `+5` to 16-bit:
- **8-bit +5:** `00000101`
- **16-bit +5:** `00000000 00000101` (MSB `0` is copied to the left)

### Integer Promotion & Comparison Pitfalls
In languages like C/C++, values smaller than `int` (char, short) are promoted to `int` before arithmetic operations.
- **Rank Hierarchy:** `unsigned long` > `long` > `unsigned int` > `int`.
- **The Pitfall:** If you compare a `signed int` and `unsigned int`, the signed value is promoted to unsigned.
```c
int a = -1;
unsigned int b = 1;
if (a < b) { /* This represents: (4,294,967,295 < 1) -> FALSE! */ }
```

---

## 6. Bit-Level Operations

Bitwise operators manipulate individual bits within a number, providing extreme efficiency.

| Operator | Action | Logic |
|----------|--------|-------|
| `&` (AND) | Masking | 1 if both bits are 1 |
| `\|` (OR)  | Setting | 1 if either bit is 1 |
| `^` (XOR) | Toggling | 1 if bits are different |
| `~` (NOT) | Inverting | Flips 1 to 0 and 0 to 1 |

### Masking and Shifting
- **Masking:** Use `&` to extract specific bits (e.g., `x & 0xFF`).
- **Left Shift (`<<`):** Multiplies by powers of 2. `x << 1` is $x \cdot 2$.
- **Right Shift (`>>`):** Divides by powers of 2.
  - **Logical:** Fills with 0s.
  - **Arithmetic:** Fills with the sign bit to preserve negative values. In C, right-shifting a signed negative value is often **implementation-defined**.

### Advanced Bitwise Logic
- **Bit Rotation (Circular Shift):** Bits shifted out one end reappear on the other.
  - `01011001` Rotate Left 1 -> `10110010`
  - `01011001` Rotate Right 1 -> `10101100`
- **Population Count:** Counting the "on" bits.
  - `PopCount(10110100)` = 4
- **XOR Swap:** `a ^= b; b ^= a; a ^= b;`.

---

## 7. Binary and Memory

### Endianness
Defines the order of bytes in multi-byte data types.
- **Big-Endian:** MSB first (Network Byte Order).
- **Little-Endian:** LSB first (x86 systems).

> [!TIP]
> Always use `htons` (host-to-network-short) or similar functions when serializing binary data for network transmission to avoid endianness mismatches.

### Memory Layout & Alignment
- **Padding:** Compilers insert empty bytes between struct members to align them to natural boundaries.
- **Bit Fields:** C allows packing multiple small fields into a single integer: `struct { unsigned int flag : 1; }`.
- **Pointers as Integers:** Pointers are often cast to `uintptr_t` for arithmetic. Address space includes virtual memory mappings, heap, and stack.

---

## 8. Floating-Point Representation (IEEE 754)

## 8. Floating-Point Representation (IEEE 754)

Real numbers use a format similar to scientific notation: $(-1)^{sign} \cdot mantissa \cdot 2^{exponent-bias}$.
A 32-bit `float` layout consists of:
- **Sign (1 bit):** 0 for +, 1 for -.
- **Exponent (8 bits):** Stored with a **bias** (127) to represent both small and large values without a sign bit.
- **Mantissa (23 bits):** Represents the significant digits (with an implicit leading 1).

#### Floating-Point Example ($12.5$)
1. **Binary:** $12.5 = 1100.1_2$
2. **Normalize:** $1.1001 \times 2^3$
3. **Sign:** $positive \rightarrow 0$
4. **Exponent:** $3 + 127 = 130 \rightarrow 10000010_2$
5. **Mantissa:** $1001000\dots$ (fractional part)
**Full Float:** `0 | 10000010 | 10010000000000000000000`

### Precision & Special Values
- **Denormal Numbers:** Very small numbers near zero where the leading 1 is lost.
- **NaN (Not a Number):** Result of invalid operations (like $0/0$).
- **Infinity:** Result of overflow (like $1/0$).
- **Precision:** $0.1$ has no exact binary representation, causing `0.1 + 0.2 != 0.3`.

---

## 9. Overflow & Underflow

### Unsigned Wraparound
Unsigned integers use modular arithmetic ($mod\ 2^N$). If you add 1 to the max value, it becomes 0.

### Signed Overflow & Undefined Behavior (UB)
In C/C++, signed overflow is **Undefined Behavior**.
- **Compiler Optimization:** Modern compilers assume UB *never* happens. If a loop relies on overflow to terminate, the compiler may optimize it away.
- **Safety:** Use flags like `-fwrapv` to force the compiler to treat signed overflow as two's complement wraparound.

---

## 10. Binary in Hardware

### Logic Gates
The building blocks of CPUs: AND, OR, NOT, and XOR.

### Adders
- **Half-Adder:** Adds two bits.
- **Full-Adder:** Adds two bits plus a carry-in.
- **ALU (Arithmetic Logic Unit):** The engine of the CPU that executes all arithmetic and logic.

---

## 11. Performance & Bit Tricks

### Why Powers of Two Matter
Memory blocks and disk sectors are sized in powers of two because it replaces slow division with fast bit masking.

### Bit Tricks Explained
- **Check Power of Two:** `(x & (x - 1)) == 0`.
  - Mathematically: A power of two has exactly one bit set (`1000`). $x - 1$ flips that bit and sets all lower bits (`0111`). Their AND is zero.
- **Hamming Weight:** Using bitwise tricks to count 1s.
- **Odd Check:** `(x & 1)` checks the LSB.

---

## Summary

- **Bases:** Binary is fundamental; Hexadecimal is for readability.
- **Two's Complement:** Industry standard for signed integers.
- **Hardware Flags:** CF, OF, ZF, SF guide conditional execution.
- **Type Promotion:** Critical to understand to avoid comparison bugs.
- **IEEE 754:** Floating-point is an approximation of real numbers.

## Important Keywords

### **Two's Complement**
A method of representing signed integers by inverting bits and adding one.

### **Endianness**
The order of bytes in memory (Big-Endian vs. Little-Endian).

### **Sign Extension**
Copying the sign bit when converting to a larger integer type.

### **ALU**
The status-flag-driven core of the CPU that handles calculations.

### **Undefined Behavior (UB)**
Code behavior not defined by the language spec, often exploited for optimization.
