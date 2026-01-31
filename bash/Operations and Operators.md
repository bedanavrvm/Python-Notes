# Chapter 6: Operations and Operators

Bash provides a rich set of operators for arithmetic, logical, and bitwise operations. Understanding operator precedence and how to combine them correctly is essential for writing robust scripts.

In programming, **operations** are actions performed on data (adding numbers, comparing values), and **operators** are the symbols that represent these operations. Mastering operators enables you to write concise, expressive code that manipulates data effectively.

---

## 6.1. Arithmetic Operators and Contexts

Arithmetic operations in Bash can be performed using several constructs. It's important to understand when and why to use each.

### Arithmetic Evaluation Constructs

Bash provides three main ways to perform arithmetic:

| Construct | Syntax | Usage | Notes |
|-----------|--------|-------|-------|
| `let` | `let "expr"` | Variable assignment | Older, explicit syntax |
| `(( ))` | `(( expr ))` | Arithmetic or condition | **Preferred**, most readable |
| `$[ ]` | `$[ expr ]` | Command substitution | **Deprecated**, avoid |
| `$(( ))` | `result=$(( expr ))` | Command substitution | Same as `$[ ]`, preferred syntax |

**Why `(( ))` is preferred:**
- More readable than `let`
- Can be used in conditions without `[ ]`
- Allows spaces around operators
- More familiar to C/Java developers

### Basic Arithmetic Operators

| Operator | Meaning | Example | Result |
|----------|---------|---------|--------|
| `+` | Addition | `(( 5 + 3 ))` | 8 |
| `-` | Subtraction | `(( 5 - 3 ))` | 2 |
| `*` | Multiplication | `(( 5 * 3 ))` | 15 |
| `/` | Division (integer) | `(( 15 / 3 ))` | 5 |
| `%` | Modulo (remainder) | `(( 17 % 5 ))` | 2 |
| `**` | Exponentiation | `(( 2 ** 3 ))` | 8 |
| `-` | Unary minus | `(( -5 ))` | -5 |
| `+` | Unary plus | `(( +5 ))` | 5 |

### Important Distinction: Integer Division

Bash performs **integer division** — it truncates decimal results:

```bash
#!/bin/bash

# Integer division (decimal truncated)
echo "15 / 4 = $(( 15 / 4 ))"      # Output: 3 (not 3.75)
echo "17 / 5 = $(( 17 / 5 ))"      # Output: 3 (not 3.4)

# For decimal results, use bc command
echo "15 / 4 = $(echo "scale=2; 15/4" | bc)"  # Output: 3.75
```

### Using Arithmetic in Different Contexts

```bash
#!/bin/bash

# Context 1: Variable assignment
result=$(( 5 + 3 ))
echo "Result: $result"              # Output: 8

# Context 2: In conditions
if (( 5 > 3 )); then
  echo "5 is greater than 3"
fi

# Context 3: Arithmetic substitution
echo "10 + 20 = $(( 10 + 20 ))"

# Context 4: With variables
count=5
(( count = count + 1 ))
echo "New count: $count"            # Output: 6
```

---

## 6.2. Assignment and Compound Assignment Operators

Assignment operators modify a variable based on its current value.

### Assignment Operators

| Operator | Meaning | Example | Equivalent |
|----------|---------|---------|------------|
| `=` | Assign | `a=5` | Direct assignment |
| `+=` | Add and assign | `a += 5` | `a = a + 5` |
| `-=` | Subtract and assign | `a -= 3` | `a = a - 3` |
| `*=` | Multiply and assign | `a *= 2` | `a = a * 2` |
| `/=` | Divide and assign | `a /= 2` | `a = a / 2` |
| `%=` | Modulo and assign | `a %= 3` | `a = a % 3` |

### Practical Examples

```bash
#!/bin/bash

# Using +=
count=10
(( count += 5 ))
echo "Count after += 5: $count"     # 15

# Using -=
score=100
(( score -= 20 ))
echo "Score after -= 20: $score"    # 80

# Using *= (scaling)
price=29
(( price *= 2 ))
echo "Price doubled: $price"        # 58

# Chaining operations
total=0
(( total += 50 ))                   # 50
(( total += 30 ))                   # 80
(( total -= 10 ))                   # 70
echo "Final total: $total"          # 70
```

**Why compound assignment matters:**
- **Readability**: `count += 1` is clearer than `count=$(( count + 1 ))`
- **Brevity**: Fewer characters to type
- **Correctness**: Less chance of typos like `count = count + 1` (which shadows the variable)

---

## 6.3. Increment and Decrement Operators

These operators change a variable by 1. The prefix and postfix forms have **different** behaviors in certain contexts.

### Increment/Decrement Operators

| Operator | Meaning | Behavior |
|----------|---------|----------|
| `++var` | Pre-increment | Increment **first**, then use value |
| `var++` | Post-increment | Use value **first**, then increment |
| `--var` | Pre-decrement | Decrement **first**, then use value |
| `var--` | Post-decrement | Use value **first**, then decrement |

### Understanding Pre vs. Post

**Pre-increment/decrement:** The value changes **before** being used in the expression

```bash
a=5
b=$(( ++a ))
echo "a=$a, b=$b"      # a=6, b=6 (a incremented first, then assigned to b)
```

**Post-increment/decrement:** The value changes **after** being used in the expression

```bash
a=5
b=$(( a++ ))
echo "a=$a, b=$b"      # a=6, b=5 (value assigned to b first, then a incremented)
```

### Practical Use Cases

```bash
#!/bin/bash

# Loop counter (most common use)
count=0
while (( count < 5 )); do
  echo "Count: $count"
  (( count++ ))
done

# Decrement (countdown)
countdown=3
while (( countdown > 0 )); do
  echo "T minus $countdown"
  (( countdown-- ))
done
echo "Blastoff!"

# Pre vs. post matters in assignments
a=10
b=$(( ++a ))
echo "Pre-increment: a=$a, b=$b"    # a=11, b=11

a=10
b=$(( a++ ))
echo "Post-increment: a=$a, b=$b"   # a=11, b=10
```

---

## 6.4. Bitwise Operators

Bitwise operators manipulate individual bits and are rarely used in shell scripts. They're primarily useful for:
- Manipulating hardware registers
- Processing binary flags
- Bit-level data manipulation

### Bitwise Operators

| Operator | Meaning | Example | Binary Example |
|----------|---------|---------|-----------------|
| `<<` | Left shift (multiply by 2 per shift) | `(( 5 << 2 ))` | `0101` << 2 = `10100` (20) |
| `>>` | Right shift (divide by 2 per shift) | `(( 20 >> 2 ))` | `10100` >> 2 = `0101` (5) |
| `&` | Bitwise AND | `(( 12 & 10 ))` | `1100` & `1010` = `1000` (8) |
| `\|` | Bitwise OR | `(( 12 \| 10 ))` | `1100` \| `1010` = `1110` (14) |
| `^` | Bitwise XOR | `(( 12 ^ 10 ))` | `1100` ^ `1010` = `0110` (6) |
| `~` | Bitwise NOT | `(( ~5 ))` | ~`0101` = `...1010` (-6) |

### Understanding Bitwise Operations

**Bitwise AND (`&`)**: Result has 1 where **both** inputs have 1

```bash
12 = 1100
10 = 1010
---------
 8 = 1000  (only position 3 has 1 in both)
```

**Bitwise OR (`|`)**: Result has 1 where **at least one** input has 1

```bash
12 = 1100
10 = 1010
---------
14 = 1110  (positions 1,2,3 have at least one 1)
```

**Bitwise XOR (`^`)**: Result has 1 where inputs **differ**

```bash
12 = 1100
10 = 1010
---------
 6 = 0110  (positions 1,2 differ between inputs)
```

**Left Shift (`<<`)**: Shift bits left, fill with zeros (multiplies by 2^n)

```bash
5 << 2:
0101 << 2 = 10100 = 20  (5 * 4 = 20)
```

### Bitwise Assignment Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `<<=` | Left-shift and assign | `(( a <<= 2 ))` |
| `>>=` | Right-shift and assign | `(( a >>= 2 ))` |
| `&=` | AND and assign | `(( a &= 0xFF ))` |
| `\|=` | OR and assign | `(( a \|= 0x01 ))` |
| `^=` | XOR and assign | `(( a ^= mask ))` |

### Example 6-1: Bitwise Operations

```bash
#!/bin/bash

# Left shift (multiply by powers of 2)
echo "5 << 1 = $(( 5 << 1 ))"       # 10 (5 * 2)
echo "5 << 2 = $(( 5 << 2 ))"       # 20 (5 * 4)
echo "5 << 3 = $(( 5 << 3 ))"       # 40 (5 * 8)

# Right shift (divide by powers of 2)
echo "20 >> 1 = $(( 20 >> 1 ))"     # 10 (20 / 2)
echo "20 >> 2 = $(( 20 >> 2 ))"     # 5  (20 / 4)

# Bitwise AND (mask operation)
echo "12 & 10 = $(( 12 & 10 ))"     # 8

# Bitwise OR (set bits)
echo "12 | 10 = $(( 12 | 10 ))"     # 14

# Bitwise XOR (toggle bits)
echo "12 ^ 10 = $(( 12 ^ 10 ))"     # 6
```

---

## 6.5. Logical Operators

Logical operators combine conditions and are essential for decision-making in scripts.

### Logical Operators

| Operator | Meaning | Short-circuits | Example |
|----------|---------|---|---------|
| `!` | NOT (negation) | N/A | `if [[ ! -f "$file" ]]` |
| `&&` | AND (both must be true) | **Yes** | `[[ cond1 && cond2 ]]` |
| `\|\|` | OR (at least one true) | **Yes** | `[[ cond1 \|\| cond2 ]]` |
| `-a` | AND (within `[ ]`) | No | `[ cond1 -a cond2 ]` |
| `-o` | OR (within `[ ]`) | No | `[ cond1 -o cond2 ]` |

### Understanding Short-Circuit Evaluation

**Short-circuit AND (`&&`):** If first condition is **false**, second is never evaluated

```bash
# Second condition not checked (file doesn't exist)
if [ -f "$file" ] && grep "pattern" "$file"; then
  echo "Pattern found"
fi
# grep never runs if file doesn't exist
```

**Short-circuit OR (`||`):** If first condition is **true**, second is never evaluated

```bash
# Second command not run (file exists)
[ -f "$file" ] || cp "$template" "$file"
# Copy only happens if file doesn't exist
```

### Logical NOT (`!`)

Reverses the truth value of a condition:

```bash
# NOT operator
if [ ! -f "$filename" ]; then
  echo "$filename does not exist"
fi

# Equivalent long form
if [ -f "$filename" ]; then
  # do nothing
else
  echo "$filename does not exist"
fi
```

### Example 6-2: Compound Conditions

```bash
#!/bin/bash

a=24
b=47

# Test 1: AND (both must be true)
if [ "$a" -eq 24 ] && [ "$b" -eq 47 ]; then
  echo "✓ Both conditions are true (AND)"
fi

# Test 2: OR (at least one must be true)
if [ "$a" -eq 99 ] || [ "$b" -eq 47 ]; then
  echo "✓ At least one condition is true (OR)"
fi

# Test 3: NOT
if [ ! -z "$a" ]; then
  echo "✓ a is not empty (NOT)"
fi

# Test 4: Complex logic
if [ "$a" -gt 20 ] && ([ "$b" -lt 50 ] || [ "$b" -eq 47 ]); then
  echo "✓ Complex: a > 20 AND (b < 50 OR b == 47)"
fi

# Test 5: Guard pattern (fail fast)
if [ -f "$file" ] && [ -r "$file" ] && grep -q "pattern" "$file"; then
  echo "✓ File exists, is readable, and contains pattern"
fi
```

**Critical Rule with Single Brackets:**
In `[ ]`, use `&&` and `||` **outside** the brackets, not inside:

```bash
# Correct:
if [ "$a" -eq 24 ] && [ "$b" -eq 47 ]; then

# Wrong (syntax error):
if [ "$a" -eq 24 && "$b" -eq 47 ]; then

# In double brackets, && and || work inside:
if [[ "$a" -eq 24 && "$b" -eq 47 ]]; then
```

---

## 6.6. Numerical Constants and Bases

Bash interprets numbers in different bases depending on their prefix or notation.

### Number Bases

| Notation | Base | Example | Decimal Value |
|----------|------|---------|---|
| Decimal | 10 | `32` | 32 |
| Octal | 8 | `032` | 26 (3×8 + 2) |
| Hexadecimal | 16 | `0x32` | 50 (3×16 + 2) |
| Binary | 2 | `2#11101` | 29 |
| Custom | 2-64 | `BASE#number` | Varies |

### Using Different Bases

```bash
#!/bin/bash

# Decimal (default, no prefix)
echo "Decimal 32 = $((32))"

# Octal (prefix 0, digits 0-7)
echo "Octal 032 = $((8#32))"        # 26 (easier to read with base prefix)

# Hexadecimal (prefix 0x, digits 0-9, a-f)
echo "Hex 0x20 = $((0x20))"         # 32
echo "Hex 0xFF = $((0xFF))"         # 255

# Binary (2#, digits 0-1)
echo "Binary 2#11101 = $((2#11101))"  # 29

# Base 16 (same as hex)
echo "Base16 $(( 16#1F ))"          # 31

# Base 8 (same as octal)
echo "Base8 $(( 8#37 ))"            # 31

# Base 32
echo "Base32 $(( 32#10 ))"          # 32
```

### Why Different Bases Matter

```bash
#!/bin/bash

# Common use: file permissions in octal
chmod 755 script.sh              # Octal: read+write+execute, read+execute, read+execute

# Color values in hexadecimal
color=0xFF0000                   # Red in RGB

# Network masks
mask=$((0xFF000000))             # Common network mask

# Bit flags
flag=$((2#1010))                 # Set specific bits
```

---

## 6.7. The Double-Parentheses Construct `(( ))`

The `(( ... ))` construct is Bash's primary arithmetic mechanism, providing:
- Arithmetic evaluation and assignment
- C-style variable manipulation
- Conditional testing without `[ ]`
- The ternary operator

### Features of `(( ))`

```bash
#!/bin/bash

# Feature 1: Arithmetic assignment (with spaces, no need for quotes)
(( result = 5 + 3 ))
echo "Result: $result"

# Feature 2: C-style operators (not just -gt, -eq)
(( count = 0 ))
if (( count < 5 )); then          # Using < instead of -lt
  echo "Count is less than 5"
fi

# Feature 3: Pre/post increment and decrement
(( count++ ))
(( ++count ))
(( --count ))

# Feature 4: Ternary operator (inline if-then-else)
(( grade = (score >= 90) ? 1 : 0 ))

# Feature 5: Complex expressions with readable syntax
(( result = (a + b) * c / d - e ** 2 ))
```

### Ternary Operator: Inline Conditionals

The ternary operator provides compact if-then-else logic:

```bash
# Syntax: (( result = condition ? value_if_true : value_if_false ))

score=85
(( grade = (score >= 90) ? 'A' : (score >= 80) ? 'B' : 'C' ))
echo "Grade: $grade"

# More complex example
(( age = 25 ))
(( status = (age < 18) ? 0 : (age < 65) ? 1 : 2 ))
# status = 1 (adult, working age)
```

### Example 6-3: C-Style Variable Manipulation

```bash
#!/bin/bash

echo "Initial assignment:"
(( a = 23 ))
echo "a = $a"

echo
echo "Post-increment:"
(( a++ ))
echo "a after a++ = $a"            # 24

echo
echo "Pre-increment:"
(( ++a ))
echo "a after ++a = $a"            # 25

echo
echo "Compound assignment:"
(( a += 10 ))
echo "a after += 10 = $a"          # 35

echo
echo "Ternary operator:"
(( result = (a > 30) ? 100 : 50 ))
echo "a > 30 ? 100 : 50 = $result"  # 100
```

---

## 6.8. Operator Precedence

Operator precedence determines the order in which operations are evaluated. **Higher-precedence** operators execute before **lower-precedence** ones.

### Precedence Hierarchy (Highest to Lowest)

| Precedence | Operators | Example |
|-----------|-----------|---------|
| **HIGHEST** | Post-increment/decrement (`var++`, `var--`) | First evaluated |
| | Pre-increment/decrement (`++var`, `--var`) | |
| | Unary (`!`, `~`, `+`, `-`) | |
| | Exponentiation (`**`) | `2 ** 3 ** 2` = `2 ** (3 ** 2)` |
| | Multiplication, division, modulo (`*`, `/`, `%`) | `6 / 2 * 3` = `(6 / 2) * 3` = 9 |
| | Addition, subtraction (`+`, `-`) | `2 + 3 * 4` = `2 + (3 * 4)` = 14 |
| | Bitwise shifts (`<<`, `>>`) | |
| | Bitwise AND (`&`) | |
| | Bitwise XOR (`^`) | |
| | Bitwise OR (`\|`) | |
| | Relational (`<`, `>`, `<=`, `>=`) | |
| | Equality (`==`, `!=`, `-eq`, `-ne`) | |
| | Logical AND (`&&`, `-a`) | Short-circuits |
| | Logical OR (`\|\|`, `-o`) | Short-circuits |
| | Ternary (`condition ? true : false`) | Right-associative |
| | Assignment (`=`, `+=`, `-=`, etc.) | Right-associative |
| **LOWEST** | Comma (`,`) | Last evaluated |

### Key Precedence Rules

**Rule 1: Multiplication and Division Before Addition and Subtraction**

```bash
echo $(( 2 + 3 * 4 ))    # 14, not 20
# Evaluates as: 2 + (3 * 4)
```

**Rule 2: Use Parentheses for Clarity**

```bash
# Ambiguous
if (( a || b && c )); then

# Clear
if (( a || (b && c) )); then
# Logical AND has higher precedence than OR
```

**Rule 3: Left-to-Right for Equal Precedence**

```bash
echo $(( 10 - 5 + 2 ))   # 7, not 3
# Evaluates as: (10 - 5) + 2, not 10 - (5 + 2)
```

**Rule 4: Exponentiation is Right-Associative**

```bash
echo $(( 2 ** 3 ** 2 ))  # 512 (2^9)
# Evaluates as: 2 ** (3 ** 2), not (2 ** 3) ** 2
```

### Example 6-4: Understanding Precedence

```bash
#!/bin/bash

echo "Arithmetic precedence:"
echo "5 + 3 * 2 = $(( 5 + 3 * 2 ))"       # 11 (multiply first)
echo "(5 + 3) * 2 = $(( (5 + 3) * 2 ))"   # 16 (explicit grouping)

echo
echo "Logical operator precedence:"
a=1
b=0
c=1

# AND has higher precedence than OR
if (( a || b && c )); then
  echo "true (parsed as: a || (b && c))"
fi

# With explicit grouping
if (( (a || b) && c )); then
  echo "true"
fi

echo
echo "Exponentiation (right-associative):"
echo "2 ** 3 ** 2 = $(( 2 ** 3 ** 2 ))"   # 512 (2^9)
echo "(2 ** 3) ** 2 = $(( (2 ** 3) ** 2 ))"  # 64 (8^2)
```

---

## 6.9. The Comma Operator

The comma operator chains multiple operations together. All operations execute, but only the **last result** is returned.

```bash
# Returns 11 (result of last operation)
(( result = (5 + 3, 7 - 1, 15 - 4) ))
echo "$result"                     # 11

# Useful in for loops with multiple variables
for (( i = 0, j = 100; i < 10; i++, j-- )); do
  echo "i=$i, j=$j"
done
```

---

## Programming Keywords and Concepts

### Essential Operators and Keywords

| Operator | Type | Purpose |
|---|---|---|
| `(( ))` | Arithmetic | Evaluate expressions and conditions |
| `$(( ))` | Substitution | Embed arithmetic in strings/assignments |
| `+`, `-`, `*`, `/`, `%`, `**` | Arithmetic | Basic math operations |
| `++`, `--` | Increment/decrement | Add or subtract 1 |
| `+=`, `-=`, `*=`, `/=`, `%=` | Compound assignment | Modify and assign in one operation |
| `<<`, `>>`, `&`, `\|`, `^`, `~` | Bitwise | Bit-level manipulation |
| `&&`, `\|\|`, `!` | Logical | Combine and negate conditions |
| `?:` | Ternary | Inline if-then-else |

### Core Programming Concepts

**1. Operator Precedence**
- Determines order of evaluation in complex expressions
- Multiplication/division before addition/subtraction
- Use parentheses for clarity and correctness
- Prevents subtle bugs from unexpected evaluation order

**2. Arithmetic Contexts**
- `(( ))` — Preferred for all arithmetic in Bash
- `let` — Older syntax, less readable
- `$[ ]` — Deprecated, avoid
- Each context returns a value suitable for different uses

**3. Pre vs. Post Increment/Decrement**
- **Pre** (`++var`): Increment first, use new value
- **Post** (`var++`): Use old value, then increment
- Usually equivalent in simple statements
- Difference matters in complex expressions

**4. Short-Circuit Evaluation**
- `&&` and `||` stop evaluating once result is known
- Allows "guard" patterns: `[ -f "$file" ] && grep pattern "$file"`
- Prevents errors from invalid operations
- More efficient (skips unnecessary evaluation)

**5. Ternary Operator for Inline Logic**
- `condition ? true_value : false_value`
- Compact alternative to if-then-else
- Can be nested for complex decisions
- Improves readability vs. multiple if statements

**6. Bitwise Operations for Bit Manipulation**
- Left/right shift for multiplying/dividing by powers of 2
- AND for masking (keeping only certain bits)
- OR for setting bits
- XOR for toggling bits
- Rarely used in scripts but essential in systems programming

**7. Logical Operators for Decision Combining**
- AND (&&) — all conditions must be true
- OR (||) — at least one condition must be true
- NOT (!) — reverses truth value
- Enables expressing complex business logic

**8. Number Base Conversions**
- Octal (base 8): Used for permissions
- Hexadecimal (base 16): Used for colors, addresses
- Binary (base 2): Used for bit flags
- Custom bases: Sometimes used for data encoding

**9. Assignment Operators for Brevity**
- Compound operators (`+=`, `-=`, etc.) more readable than `a = a + b`
- Reduces variable references and typos
- Essential idiom in Bash programming
- Combined with loops for counters

**10. Expression Evaluation Strategy**
- Use `(( ))` for all arithmetic in modern Bash
- Parenthesize subexpressions for clarity
- Break complex expressions into intermediate variables
- Always test edge cases and boundaries

---

## Best Practices for Safe Arithmetic

### Pattern 1: Explicit Grouping

```bash
# Bad: unclear what condition is being tested
if (( a > 5 || b < 10 && c == 1 )); then

# Good: explicit grouping
if (( a > 5 || (b < 10 && c == 1) )); then
```

### Pattern 2: Intermediate Variables

```bash
# For clarity in complex calculations
(( width = 100 ))
(( height = 50 ))
(( area = width * height ))
(( doubled = area * 2 ))
echo "Doubled area: $doubled"
```

### Pattern 3: Guard Patterns

```bash
# Fail fast if precondition not met
[ "$count" -gt 0 ] || { echo "Invalid count"; exit 1; }
# Only continue if count is positive
```

### Pattern 4: Ternary for Simple Choices

```bash
# Better than if-then-else for single choice
(( status = (age >= 18) ? 1 : 0 ))
```

---

## Summary

Operators are the building blocks of expressions:

- **Arithmetic operators** (`+`, `-`, `*`, `/`, `%`, `**`) perform calculations
- **Assignment operators** (`=`, `+=`, `-=`, etc.) modify variables
- **Increment/decrement** (`++`, `--`) change values by 1, with pre/post variants
- **Bitwise operators** (`<<`, `>>`, `&`, `|`, `^`, `~`) manipulate individual bits
- **Logical operators** (`&&`, `||`, `!`) combine and negate conditions
- **`(( ))`** is the preferred arithmetic construct in Bash for readability and flexibility
- **Operator precedence** determines evaluation order; use parentheses for clarity
- **Short-circuit evaluation** in `&&` and `||` prevents errors and improves efficiency
- **Ternary operator** (`condition ? true : false`) provides inline conditionals

Master operators, and you'll write more expressive, efficient, and maintainable Bash scripts.
