# Chapter 8: Operations and Operators

Bash provides a rich set of operators for arithmetic, logical, and bitwise operations. Understanding operator precedence and how to combine them correctly is essential for writing robust scripts.

## 8.1. Arithmetic Operators

Arithmetic operations in Bash can be performed using `let`, `$(( ))`, or `$[ ]` constructs.

{% tabs %}
{% tab title="Basic Operators" %}

| Operator | Meaning | Example |
|----------|---------|---------|
| `+` | Addition | `let "a = 5 + 3"` → 8 |
| `-` | Subtraction | `let "a = 5 - 3"` → 2 |
| `*` | Multiplication | `let "a = 5 * 3"` → 15 |
| `/` | Division (integer) | `let "a = 15 / 3"` → 5 |
| `%` | Modulo (remainder) | `let "a = 17 % 5"` → 2 |
| `**` | Exponentiation | `let "a = 2 ** 3"` → 8 |

**Syntaxes:**
```bash
# Using let
let "result = 5 + 3"

# Using double parentheses (preferred)
result=$(( 5 + 3 ))

# Using $[ ] (deprecated)
result=$[ 5 + 3 ]

# Arithmetic in conditions
if (( a > b )); then
  echo "a is greater"
fi
```

{% endtab %}
{% tab title="Assignment Operators" %}

| Operator | Meaning | Example |
|----------|---------|---------|
| `+=` | Add and assign | `let "a += 5"` → `a = a + 5` |
| `-=` | Subtract and assign | `let "a -= 3"` → `a = a - 3` |
| `*=` | Multiply and assign | `let "a *= 2"` → `a = a * 2` |
| `/=` | Divide and assign | `let "a /= 2"` → `a = a / 2` |
| `%=` | Modulo and assign | `let "a %= 3"` → `a = a % 3` |

**Example:**
```bash
a=10
let "a += 5"
echo $a           # 15

result=20
(( result *= 2 ))
echo $result      # 40
```

{% endtab %}
{% tab title="Increment/Decrement" %}

| Operator | Meaning | Effect |
|----------|---------|--------|
| `++var` | Pre-increment | Increments, then uses new value |
| `var++` | Post-increment | Uses old value, then increments |
| `--var` | Pre-decrement | Decrements, then uses new value |
| `var--` | Post-decrement | Uses old value, then decrements |

**Example:**
```bash
a=5

(( ++a ))        # Pre-increment: a becomes 6, then used
echo $a          # 6

(( a++ ))        # Post-increment: old value used, then a becomes 7
echo $a          # 7

(( --a ))        # Pre-decrement: a becomes 6, then used
echo $a          # 6

(( a-- ))        # Post-decrement: old value used, then a becomes 5
echo $a          # 5
```

Pre- vs. post-decrement have different side effects in conditional contexts.

{% endtab %}
{% endtabs %}

---

## 8.2. Bitwise Operators

Bitwise operators manipulate individual bits and are seldom used in shell scripts. Their chief use is in manipulating values read from ports or sockets. "Bit flipping" is more relevant to compiled languages like C and C++.

{% tabs %}
{% tab title="Bitwise Operations" %}

| Operator | Meaning | Example |
|----------|---------|---------|
| `<<` | Left shift (multiply by 2 per shift) | `let "a = 5 << 2"` → 20 |
| `>>` | Right shift (divide by 2 per shift) | `let "a = 20 >> 2"` → 5 |
| `&` | Bitwise AND | `let "a = 12 & 10"` → 8 |
| `\|` | Bitwise OR | `let "a = 12 \| 10"` → 14 |
| `^` | Bitwise XOR | `let "a = 12 ^ 10"` → 6 |
| `~` | Bitwise NOT | `let "a = ~5"` → -6 |

**Example:**
```bash
# Left shift: multiply by 4 (2^2)
let "result = 5 << 2"
echo $result      # 20

# Bitwise AND
let "result = 12 & 10"  # 1100 & 1010 = 1000 (8 in decimal)
echo $result      # 8

# Bitwise OR
let "result = 12 | 10"  # 1100 | 1010 = 1110 (14 in decimal)
echo $result      # 14
```

{% endtab %}
{% tab title="Bitwise Assignment" %}

| Operator | Meaning |
|----------|---------|
| `<<=` | Left-shift-equal |
| `>>=` | Right-shift-equal |
| `&=` | AND-equal |
| `\|=` | OR-equal |
| `^=` | XOR-equal |

**Example:**
```bash
a=5
let "a <<= 2"     # a = a << 2
echo $a           # 20
```

{% endtab %}
{% endtabs %}

---

## 8.3. Logical Operators

Logical operators combine conditions and control program flow.

{% tabs %}
{% tab title="Logical NOT, AND, OR" %}

| Operator | Meaning | Usage |
|----------|---------|-------|
| `!` | Logical NOT | `if [ ! -f "$file" ]` |
| `&&` | Logical AND (short-circuit) | `[ cond1 ] && [ cond2 ]` or `[[ cond1 && cond2 ]]` |
| `\|\|` | Logical OR (short-circuit) | `[ cond1 ] \|\| [ cond2 ]` or `[[ cond1 \|\| cond2 ]]` |
| `-a` | AND (no short-circuit) | `[ cond1 -a cond2 ]` |
| `-o` | OR (no short-circuit) | `[ cond1 -o cond2 ]` |

**Key Difference:** `&&` and `||` short-circuit (stop evaluating once result is known), while `-a` and `-o` do not.

{% endtab %}
{% endtabs %}

### Example 8-3: Compound Condition Tests

```bash
#!/bin/bash
# Compound conditions using && and ||

a=24
b=47

# Test 1: Using && (AND)
if [ "$a" -eq 24 ] && [ "$b" -eq 47 ]
then
  echo "Test #1 succeeds (both conditions true)"
else
  echo "Test #1 fails"
fi

echo

# Test 2: Using || (OR)
if [ "$a" -eq 98 ] || [ "$b" -eq 47 ]
then
  echo "Test #2 succeeds (at least one condition true)"
else
  echo "Test #2 fails"
fi

echo

# Test 3: Using -a (AND, no short-circuit)
if [ "$a" -eq 24 -a "$b" -eq 47 ]
then
  echo "Test #3 succeeds"
else
  echo "Test #3 fails"
fi

echo

# Test 4: Using -o (OR, no short-circuit)
if [ "$a" -eq 98 -o "$b" -eq 47 ]
then
  echo "Test #4 succeeds"
else
  echo "Test #4 fails"
fi

echo

# Test 5: String comparison with &&
a=rhino
b=crocodile
if [ "$a" = rhino ] && [ "$b" = crocodile ]
then
  echo "Test #5 succeeds"
else
  echo "Test #5 fails"
fi

# && and || in arithmetic context
echo
echo "Arithmetic context:"
echo "1 && 2 = $(( 1 && 2 ))"        # 1 (both true)
echo "3 && 0 = $(( 3 && 0 ))"        # 0 (one false)
echo "4 || 0 = $(( 4 || 0 ))"        # 1 (at least one true)
echo "0 || 0 = $(( 0 || 0 ))"        # 0 (both false)

exit 0
```

**Important:** In single brackets `[ ]`, use `&&` and `||` **outside** the brackets, not inside:
```bash
# Correct:
if [ "$a" -eq 24 ] && [ "$b" -eq 47 ]

# Wrong:
if [ "$a" -eq 24 && "$b" -eq 47 ]  # Syntax error!

# In double brackets, && and || work inside:
if [[ "$a" -eq 24 && "$b" -eq 47 ]]
```

---

## 8.4. Numerical Constants

Bash interprets numbers in different bases depending on their prefix or notation.

{% tabs %}
{% tab title="Number Bases" %}

| Prefix | Base | Example | Result |
|--------|------|---------|--------|
| (none) | Decimal (base 10) | `32` | 32 |
| `0` | Octal (base 8) | `032` | 26 (3×8 + 2) |
| `0x` or `0X` | Hexadecimal (base 16) | `0x32` | 50 (3×16 + 2) |
| `BASE#` | Custom base (2-64) | `2#11101` | 29 |

**Examples:**
```bash
# Decimal
let "dec = 32"
echo $dec             # 32

# Octal (base 8)
let "oct = 032"
echo $oct             # 26

# Hexadecimal (base 16)
let "hex = 0x32"
echo $hex             # 50

# Binary (base 2)
let "bin = 2#111100111001101"
echo $bin             # 31181

# Base 32
let "b32 = 32#77"
echo $b32             # 231

# Base 64
let "b64 = 64#@_"
echo $b64             # 4031

# Mixed bases
echo $((36#zz))       # 1295 (base 36)
echo $((2#10101010))  # 170 (binary)
echo $((16#AF16))     # 44822 (hex)
```

{% endtab %}
{% endtabs %}

### Example 8-4: Representation of Numerical Constants

```bash
#!/bin/bash
# numbers.sh: Numbers in different bases

# Decimal (default)
let "dec = 32"
echo "Decimal: $dec"

# Octal (prefix 0)
let "oct = 032"
echo "Octal 032 = $oct in decimal"

# Hexadecimal (prefix 0x)
let "hex = 0x32"
echo "Hex 0x32 = $hex in decimal"

# More hex examples
echo "Hex 0x9abc = $((0x9abc))"

# Custom bases (BASE#NUMBER, where BASE is 2-64)
let "bin = 2#111100111001101"
echo "Binary = $bin"

let "b32 = 32#77"
echo "Base-32 = $b32"

let "b64 = 64#@_"
echo "Base-64 = $b64"

# Multiple custom bases in one command
echo "Various bases: $((36#zz)) $((2#10101010)) $((16#AF16)) $((53#1aA))"
# Output: 1295 170 44822 3375

# Note: Digits must be in range for the specified base
# let "bad_oct = 081"  # Error: 8 is not valid in octal
# Octal uses only 0-7

exit 0
```

---

## 8.5. The Double-Parentheses Construct `(( ))`

The `(( ... ))` construct permits arithmetic expansion and C-style variable manipulation.

{% tabs %}
{% tab title="Features" %}

The `(( ... ))` construct supports:
- **Arithmetic evaluation:** `(( a = 5 + 3 ))`
- **C-style increment/decrement:** `(( a++ ))`, `(( ++a ))`
- **C-style operators:** `<`, `<=`, `>`, `>=`, `==`, `!=`
- **Conditional evaluation:** Returns 0 (success) if result is non-zero
- **Ternary operator:** `(( result = a > b ? a : b ))`

**Advantages over `let`:**
- More readable C-like syntax
- Spaces around operators are permitted
- Can be used in conditions without needing `[ ]`

{% endtab %}
{% endtabs %}

### Example 8-5: C-Style Variable Manipulation

```bash
#!/bin/bash
# c-vars.sh: C-style variable manipulation

echo "Initial assignment:"
(( a = 23 ))
echo "a = $a"

echo
echo "Post-increment (use value, then increment):"
(( a++ ))
echo "a after a++ = $a"  # 24

echo
echo "Post-decrement (use value, then decrement):"
(( a-- ))
echo "a after a-- = $a"  # 23

echo
echo "Pre-increment (increment, then use value):"
(( ++a ))
echo "a after ++a = $a"  # 24

echo
echo "Pre-decrement (decrement, then use value):"
(( --a ))
echo "a after --a = $a"  # 23

echo
echo "Compound assignment operators:"
(( a += 10 ))
echo "a after += 10 = $a"  # 33

echo
echo "C-style ternary operator:"
(( t = a < 45 ? 7 : 11 ))
echo "a < 45 ? 7 : 11 = $t"  # 7 (since a=33)

echo
echo "Arithmetic in conditions:"
if (( a > 30 ))
then
  echo "a is greater than 30"
fi

echo
echo "Pre- vs. post-decrement in conditions:"
n=1
let --n && echo "Pre-decrement: True" || echo "Pre-decrement: False"
n=1
let n-- && echo "Post-decrement: True" || echo "Post-decrement: False"

exit 0
```

---

## 8.6. Comma Operator

The comma operator chains together two or more arithmetic operations. All operations are evaluated, but only the result of the **last** operation is returned.

```bash
# Set t1 to result of last operation (15 - 4 = 11)
let "t1 = ((5 + 3, 7 - 1, 15 - 4))"
echo "t1 = $t1"                    # t1 = 11

# Set a and calculate t2
let "t2 = ((a = 9, 15 / 3))"
echo "t2 = $t2, a = $a"            # t2 = 5, a = 9
```

The comma operator is mainly useful in `for` loops for managing multiple loop variables.

---

## 8.7. Operator Precedence

Operator precedence determines the order in which operations are evaluated. Higher-precedence operators execute before lower-precedence ones.

### Precedence Table (Highest to Lowest)

| Precedence | Operators | Meaning |
|-----------|-----------|---------|
| **HIGHEST** | `var++`, `var--` | Post-increment, post-decrement |
| | `++var`, `--var` | Pre-increment, pre-decrement |
| | `!`, `~` | Logical/bitwise NOT |
| | `**` | Exponentiation |
| | `*`, `/`, `%` | Multiply, divide, modulo |
| | `+`, `-` | Add, subtract |
| | `<<`, `>>` | Bitwise shifts |
| | `-z`, `-n` | String null tests |
| | `-e`, `-f`, `-t`, `-x`, etc. | File tests |
| | `<`, `-lt`, `>`, `-gt`, `<=`, `-le`, `>=`, `-ge` | Comparison |
| | `-nt`, `-ot`, `-ef` | File comparison |
| | `==`, `-eq`, `!=`, `-ne` | Equality/inequality |
| | `&` | Bitwise AND |
| | `^` | Bitwise XOR |
| | `\|` | Bitwise OR |
| | `&&`, `-a` | Logical AND |
| | `\|\|`, `-o` | Logical OR |
| | `?:` | Ternary operator |
| | `=`, `+=`, `-=`, `*=`, `/=`, `%=`, etc. | Assignment |
| **LOWEST** | `,` | Comma operator |

### Key Precedence Rules

1. **"My Dear Aunt Sally"** — Multiply, Divide, Add, Subtract for arithmetic operations
2. **Compound logical operators** (`&&`, `||`, `-a`, `-o`) have **low precedence**
3. **Order of evaluation** for equal-precedence operators is usually **left-to-right**
4. **Use parentheses** to disambiguate complex expressions

### Example 8-6: Understanding Precedence

```bash
#!/bin/bash
# Understanding operator precedence

# Clear precedence example
echo "5 + 3 * 2 = $(( 5 + 3 * 2 ))"       # 11 (multiply first)
echo "5 * 3 + 2 = $(( 5 * 3 + 2 ))"       # 17 (multiply first)
echo "(5 + 3) * 2 = $(( (5 + 3) * 2 ))"   # 16 (explicit grouping)

echo

# Logical operator precedence
a=5
b=3

# This is ambiguous: -o has lower precedence than -gt
if [ -z "$a" -o "$a" -gt "$b" -a -n "$b" ]
then
  echo "Hard to read!"
fi

# Better: explicit grouping
if [[ -z "$a" ]] || ( [[ "$a" -gt "$b" ]] && [[ -n "$b" ]] )
then
  echo "Much clearer!"
fi

echo

# Real example from /etc/init.d/functions
# while [ -n "$remaining" -a "$retry" -gt 0 ]
# This evaluates as:
# while [ (-n "$remaining") -a ("$retry" -gt 0) ]
# The -n and -gt operators execute first (higher precedence)
# Then -a combines the results (lower precedence)

remaining="something"
retry=5

while [ -n "$remaining" -a "$retry" -gt 0 ]
do
  echo "Loop iteration (remaining=$remaining, retry=$retry)"
  (( retry-- ))
  [ "$retry" -eq 3 ] && remaining=""  # Exit after a few iterations
done

exit 0
```

---

## 8.8. Best Practices for Complex Expressions

To avoid confusion or errors in complex sequences of operations:

1. **Use parentheses** to group conditions explicitly
2. **Break long expressions** into readable sections
3. **Prefer `[[ ]]` over `[ ]`** for modern Bash scripts
4. **Use `&&` and `||` outside brackets** with single brackets
5. **Document complex logic** with comments

**Bad (unclear):**
```bash
if [ "$v1" -gt "$v2" -o "$v1" -lt "$v2" -a -e "$filename" ]
then
  # What's the intended logic here?
  echo "Hard to read"
fi
```

**Good (explicit grouping):**
```bash
if [[ "$v1" -gt "$v2" ]] || [[ "$v1" -lt "$v2" ]] && [[ -e "$filename" ]]
then
  echo "Conditions are clearly grouped"
fi
```

---

## Exercises

1. **Write a script** using the double-parentheses construct to perform basic arithmetic (add, multiply, divide) and demonstrate both pre- and post-increment operations.

2. **Create a script** that converts numbers between different bases (decimal, binary, octal, hexadecimal) using the custom base notation `BASE#NUMBER`.

3. **Demonstrate bitwise operations** by writing a script that performs left-shift, right-shift, AND, and OR operations on user-provided integers.

4. **Write a script** that uses the ternary operator `?:` to classify numbers as positive, negative, or zero.

5. **Create a complex condition test** that combines multiple file tests, integer comparisons, and logical operators, then rewrite it with explicit parentheses for clarity.

6. **Implement a loop** using `(( ))` construct with C-style variable manipulation (pre/post increment) and demonstrate the difference in a conditional context.

---

## Summary

- **Arithmetic operators:** `+`, `-`, `*`, `/`, `%`, `**` (exponentiation)
- **Assignment operators:** `+=`, `-=`, `*=`, `/=`, `%=`
- **Increment/decrement:** `++var` (pre), `var++` (post), `--var` (pre), `var--` (post)
- **Bitwise operators:** `<<` (left shift), `>>` (right shift), `&` (AND), `|` (OR), `^` (XOR), `~` (NOT)
- **Logical operators:** `!` (NOT), `&&` (AND with short-circuit), `||` (OR with short-circuit), `-a` (AND), `-o` (OR)
- **Numerical bases:** Decimal (no prefix), Octal (`0` prefix), Hexadecimal (`0x` prefix), Custom (`BASE#` notation)
- **Double parentheses `(( ))`:** C-style arithmetic and variable manipulation
- **Operator precedence:** High (post-increment) to low (comma) — use parentheses for clarity
- **Ternary operator:** `(( result = condition ? true_value : false_value ))`
