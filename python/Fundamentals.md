# Python Fundamentals

Now that the environment is set up, we move to the building blocks of the
language: variables, core data types, type conversion, and operators. In
Python, everything is an object, and the way we handle data is different
from statically typed languages like C++ or Java.

## Variables & Dynamic Typing

In many languages, a variable is a "container" of a specific type. In
Python, a variable is better described as a label (name) that refers to
an object in memory. Assignment (`=`) binds a name to an object; it does
not copy the object itself.

### Dynamic Typing

Python is dynamically typed. This means you do not need to declare the
type of a variable when you create one. The type is associated with the
value (the object), not the variable name. Python still has strong types,
so operations must make sense for the underlying objects.

```python
x = 42         # x is now an integer
x = "Hello"    # x is now a string
```

In the example above, we didn't "change" the type of x. We simply moved
the label x from an integer object to a string object. If you want extra
clarity, you can add optional type hints, but they are not enforced at
runtime.

### Object Identity

Every object in Python has a unique identity, a type, and a value. You
can check these using built-in functions:

```python
item = "Coffee"

print(type(item))  # <class 'str'>
print(id(item))    # 14070... (unique ID in this interpreter)
```

Use `is` to compare identities and `==` to compare values:

```python
a = [1, 2, 3]
b = a
c = [1, 2, 3]

print(a is b)  # True (same object)
print(a is c)  # False (different objects)
print(a == c)  # True (same values)
```

#### Memory Management

Python uses reference counting for memory management. When an object's
reference count drops to zero, it becomes eligible for garbage
collection. A cyclic garbage collector also cleans up objects that
reference each other. This is why you don't need to manually allocate or
deallocate memory in Python.

<details>
<summary>Show reference counting example</summary>

```python
import sys

a = "Hello"
print(sys.getrefcount(a))  # Shows reference count

b = a  # Increases reference count
del b   # Decreases reference count
```

</details>

## Data Types

Python has several built-in data types. For now, we will focus on the
"scalars" (single values). Container types like lists, tuples, and
dictionaries are covered later.

### 1. Integers (int)

Whole numbers, positive or negative, without decimals. In Python,
integers have arbitrary precision, meaning they can be as large as your
computer's memory allows.

```python
big_number = 10**100  # Googol (1 followed by 100 zeros)
readable = 1_000_000  # Underscores improve readability
negative = -42
zero = 0
```

#### Integer Operations

Bitwise operators are commonly used for flags and low-level work.

```python
# Bitwise operations
a = 5  # binary: 101
b = 3  # binary: 011

print(a & b)  # 1 (AND)
print(a | b)  # 7 (OR)
print(a ^ b)  # 6 (XOR)
print(~a)     # -6 (NOT)
print(a << 1)  # 10 (left shift)
print(a >> 1)  # 2 (right shift)
```

### 2. Floating Point Numbers (float)

Numbers with a decimal point. Python follows the IEEE 754 standard for
floating-point arithmetic (similar to double in C).

```python
pi = 3.14159
scientific = 2e3  # 2000.0
negative_float = -0.001
```

#### Floating Point Precision

Floating point numbers are stored in binary, so some decimals cannot be
represented exactly. Compare floats using rounding or `math.isclose`.

{% tabs %}

{% tab title="float" %}

```python
# Be careful with floating point arithmetic
result = 0.1 + 0.2  # 0.30000000000000004 (not exactly 0.3)
print(round(result, 2))  # 0.3 (rounded)

import math
print(math.isclose(result, 0.3))  # True
```

{% endtab %}

{% tab title="Decimal" %}

```python
# For precise decimal arithmetic, use Decimal
from decimal import Decimal
precise = Decimal("0.1") + Decimal("0.2")  # 0.3 exactly
```

{% endtab %}

{% endtabs %}

### 3. Booleans (bool)

Represents truth values: `True` or `False`.

> **Note:** In Python, booleans are a subclass of integers. `True` behaves
> like `1` and `False` behaves like `0`.

```python
is_active = True
is_complete = False

# Boolean arithmetic
print(True + True)   # 2
print(False * 5)     # 0
```

#### Truthiness

In Python, many objects can be evaluated in a boolean context. Empty
containers and `0` are falsy; most other values are truthy.

```python
# Truthy values
if "hello":      # Non-empty strings
if [1, 2, 3]:   # Non-empty lists
if 42:          # Non-zero numbers
    print("This is truthy")

# Falsy values
if not ("" or [] or {} or 0 or None):
    print("This is falsy")
```

### 4. Strings (str)

Unicode character sequences. Strings can be enclosed in single (`'`) or
double (`"`) quotes. Strings are immutable, so operations create new
string objects rather than modifying the original.

```python
name = "Guido"
multiline = """This is a 
multiline string"""
raw_string = r"C:\path\to\file"  # Raw string (no escape sequences)
```

#### String Operations

Use f-strings when you want readable, modern formatting.

```python
# String concatenation
first = "Hello"
second = "World"
greeting = first + " " + second  # "Hello World"
```

{% tabs %}

{% tab title="f-string" %}

```python
name = "Alice"
age = 25
formatted = f"{name} is {age} years old"  # f-string (Python 3.6+)
```
{% endtab %}

{% tab title="% formatting" %}

```python
name = "Alice"
age = 25
old_style = "%s is %d years old" % (name, age)
```
{% endtab %}

{% tab title=".format()" %}

```python
name = "Alice"
age = 25
format_method = "{} is {} years old".format(name, age)
```
{% endtab %}

{% endtabs %}

```python
# String methods
text = "  Python Programming  "
print(text.strip())        # "Python Programming"
print(text.upper())        # "  PYTHON PROGRAMMING  "
print(text.lower())        # "  python programming  "
print(text.replace("Python", "Java"))  # "  Java Programming  "
print(text.split())        # ["Python", "Programming"]
print(" ".join(["Hello", "World"]))  # "Hello World"
```

#### String Slicing

Slicing uses the `[start:stop:step]` pattern. The start index is
inclusive and the stop index is exclusive.

```python
text = "Python Programming"
print(text[0])        # 'P'
print(text[0:6])      # 'Python'
print(text[7:])        # 'Programming'
print(text[-3])        # 'i'
print(text[-9:-1])     # 'Programmin'
print(text[::2])       # 'Pto rgamn' (every second character)
print(text[::-1])      # 'gnimmargorP nohtyP' (reversed)
```

## Type Conversion (Casting)

Since Python is dynamically typed, you often need to convert data from
one type to another—especially when dealing with user input (`input()`
returns a string).

### Implicit Conversion

Python performs some conversions automatically to prevent data loss. This usually happens in mixed-type arithmetic.

```python
x = 10    # int
y = 2.5   # float

result = x + y 
print(result)  # 12.5 (Automatically converted to float)
```

### Explicit Conversion

You can manually convert types using constructor functions:

- `int()` - converts to an integer
- `float()` - converts to a float
- `str()` - converts to a string
- `bool()` - converts to a boolean
- `list()` - converts to a list
- `tuple()` - converts to a tuple

```python
age_str = "25"
age_int = int(age_str)  # Converts string to integer

# Warning: This will raise a ValueError if the string isn't a valid number
# invalid = int("abc")

# Safe conversion with error handling
def safe_int_convert(value, default=0):
    try:
        return int(value)
    except (ValueError, TypeError):
        return default

result = safe_int_convert("abc", 0)  # Returns 0 instead of crashing
```

> **Tip:** `bool("False")` is `True` because any non-empty string is
> truthy. Convert with explicit checks when parsing user input.

## Basic Operators

### 1. Arithmetic Operators

Python provides standard math operators, plus a few unique ones. `/`
always performs true division, while `//` floors the result.

| Operator | Name | Example | Result |
|----------|------|---------|--------|
| + | Addition | 5 + 2 | 7 |
| - | Subtraction | 5 - 2 | 3 |
| * | Multiplication | 5 * 2 | 10 |
| / | True Division | 5 / 2 | 2.5 |
| // | Floor Division | 5 // 2 | 2 (removes decimal) |
| % | Modulo | 5 % 2 | 1 (remainder) |
| ** | Exponentiation | 5 ** 2 | 25 |

#### Advanced Arithmetic

```python
# Multiple assignment
a, b, c = 1, 2, 3

# Augmented assignment
count = 0
count += 1  # count = count + 1
count *= 2  # count = count * 2
count -= 1  # count = count - 1
count /= 2  # count = count / 2

# Math functions
import math
print(math.sqrt(16))    # 4.0
print(math.ceil(3.2))   # 4
print(math.floor(3.8))   # 3
print(math.pi)          # 3.141592653589793
```

### 2. Comparison Operators

These always return a Boolean (True or False).

- `==` Equal to
- `!=` Not equal to
- `>` Greater than / `<` Less than
- `>=` Greater or equal / `<=` Less or equal

```python
age = 25
print(age == 25)    # True
print(age != 30)    # True
print(age > 18)     # True
print(age <= 25)    # True

# Chaining comparisons
score = 85
print(80 <= score <= 90)  # True (between 80 and 90, inclusive)
```

### 3. Logical Operators

In Python, logical operators use plain English words, making them highly
readable.

- `and`: True if both are true
- `or`: True if at least one is true
- `not`: Inverts boolean value

```python
is_adult = True
has_ticket = False

can_enter = is_adult and has_ticket  # False
must_wait = not can_enter           # True

# Short-circuit evaluation
def expensive_function():
    print("This won't be called!")
    return True

# If first condition is False, second isn't evaluated
result = False and expensive_function()  # Only prints nothing
result = True or expensive_function()    # Only prints nothing
```

## Summary

- Variables are labels pointing to objects; they don't have fixed types.
- Dynamic typing means names can be rebound, while objects keep their
  own types.
- Floats can have precision limits; use `Decimal` or `math.isclose` when
  accuracy matters.
- Floor division (`//`) and exponentiation (`**`) are specific Python
  syntax features.
- Logical operators use English words (`and`, `or`, `not`).
- String manipulation is powerful with built-in methods and slicing.
- Error handling is essential for safe type conversion.

## Important Keywords

### **Dynamic Typing**

Variable types are determined at runtime rather than being declared in
advance. Provides flexibility but requires runtime type checking.

### **Object Identity**

Every object in Python has a unique identifier, type, and value. The
`id()` function returns a unique identity for the object's lifetime.

### **Reference Counting**

Memory management technique where Python tracks how many variables
reference an object. Objects are deleted when count reaches zero.

### **Garbage Collection**

Automatic memory management that frees memory occupied by objects no longer referenced by any variables.

### **Arbitrary Precision**

Python integers can grow to any size limited only by available memory, unlike fixed-size integers in other languages.

### **IEEE 754**

Standard for floating-point arithmetic that Python follows, defining how decimal numbers are stored and calculated.

### **Unicode**

Character encoding standard that supports text from all writing systems. Python 3 strings are Unicode by default.

### **Truthiness**

Concept where non-zero numbers, non-empty containers, and non-None
objects evaluate to True in boolean contexts.

### **String Slicing**

Syntax for extracting substrings using start, stop, and step parameters:
`[start:stop:step]` (stop is exclusive).

### **Type Casting**

Explicit conversion of data from one type to another using constructor
functions like `int()`, `float()`, `str()`.

### **Short-circuit Evaluation**

Optimization where logical operators stop evaluating as soon as the result is determined.

### **Floor Division**

Division operator (`//`) that returns the largest integer less than or equal to the exact division result.

### **Modulo Operation**

Remainder operation (`%`) that returns what's left after integer division.

### **F-strings**

Modern string formatting syntax using `f"prefix{variable}suffix"` for embedding expressions directly in strings.

### **Bitwise Operations**

Operations that manipulate individual bits of integers: AND (`&`), OR
(`|`), XOR (`^`), NOT (`~`), left shift (`<<`), right shift (`>>`).

### **Raw Strings**

String literals prefixed with `r` that disable escape sequence processing, useful for file paths and regular expressions.
