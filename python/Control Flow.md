# Control Flow

Control flow is the order in which individual statements, instructions, or function calls are executed. In Python, this is governed by conditional logic and loops. One of Python's most distinct features is that it uses indentation to define blocks of code, rather than curly braces {}.

## Conditional Branching

The if statement allows you to execute a block of code only if a specific condition is met.

### if, elif, and else

- `if`: The initial test
- `elif`: Short for "else if." It allows you to check multiple expressions
- `else`: The "catch-all" that executes if none of the previous conditions were true

```python
age = 20

if age < 13:
    print("You are a child.")
elif age < 20:
    print("You are a teenager.")
else:
    print("You are an adult.")
```

### Nested Conditionals

You can nest if statements inside each other for more complex logic:

```python
age = 25
has_license = True

if age >= 18:
    if has_license:
        print("You can drive.")
    else:
        print("You need a license to drive.")
else:
    print("You're too young to drive.")
```

### Conditional Expressions (Ternary Operator)

Python provides a concise way to write simple if-else statements:

```python
# Traditional if-else
if age >= 18:
    status = "adult"
else:
    status = "minor"

# Ternary operator
status = "adult" if age >= 18 else "minor"

# Nested ternary
message = "senior" if age >= 65 else "adult" if age >= 18 else "minor"
```

## Truthiness and Falsiness

In Python, every object has an inherent boolean value. This is known as Truthiness. The following values are considered Falsy (they evaluate to False in an if statement):

- `None`
- `False`
- Zero of any numeric type: `0`, `0.0`, `0j`
- Empty sequences and collections: `''`, `()`, `[]`, `{}`, `set()`, `range(0)`

Everything else is generally considered Truthy.

```python
# Examples of truthiness checks
if "hello":      # Truthy (non-empty string)
    print("String is truthy")

if []:          # Falsy (empty list)
    print("This won't print")

if 0:           # Falsy (zero)
    print("This won't print")

if [1, 2, 3]:   # Truthy (non-empty list)
    print("List is truthy")
```

### Custom Truthiness

You can define custom truthiness for your classes by implementing the `__bool__` or `__len__` methods:

```python
class Container:
    def __init__(self, items):
        self.items = items

    def __bool__(self):
        return len(self.items) > 0

    # Alternative: __len__ method also affects truthiness
    # def __len__(self):
    #     return len(self.items)

empty_container = Container([])
full_container = Container([1, 2, 3])

if empty_container:
    print("Empty container is truthy")
else:
    print("Empty container is falsy")

if full_container:
    print("Full container is truthy")
```

## Loops

Loops allow you to repeat a block of code. Python provides two primary types: `while` and `for`.

### 1. The while Loop

The while loop repeats as long as a condition remains True.

```python
count = 5
while count > 0:
    print(f"Countdown: {count}")
    count -= 1  # Decrementing is crucial to avoid infinite loops
```

#### while Loop with else

A while loop can have an `else` clause that executes when the loop completes normally (not via break):

```python
number = 0
while number < 5:
    print(number)
    number += 1
else:
    print("Loop completed normally")
```

#### Infinite Loops and Break

```python
while True:
    user_input = input("Enter 'quit' to exit: ")
    if user_input.lower() == 'quit':
        break
    print(f"You entered: {user_input}")
```

### 2. The for Loop and the in Keyword

Unlike languages that use a counter-based for loop (like `for(i=0; i<10; i++)`), Python's for loop is an iterator. It traverses any "iterable" object (like a list, a string, or a range).

The `in` keyword tells Python to pick the next element from the collection and assign it to the variable.

```python
fruits = ["apple", "banana", "cherry"]

for fruit in fruits:
    print(f"I like {fruit}")
```

#### Iterating Over Different Types

```python
# Strings
for char in "Python":
    print(char)

# Dictionaries (iterates over keys by default)
person = {"name": "Alice", "age": 25, "city": "New York"}
for key in person:
    print(f"{key}: {person[key]}")

# Dictionary items
for key, value in person.items():
    print(f"{key}: {value}")

# Lists with index
fruits = ["apple", "banana", "cherry"]
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# Multiple lists
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
for name, age in zip(names, ages):
    print(f"{name} is {age} years old")
```

### The range() Function

To loop a specific number of times, we use the `range(start, stop, step)` function.

```python
# range(5) generates numbers from 0 to 4
for i in range(5):
    print(i)

# range(1, 6) generates numbers from 1 to 5
for i in range(1, 6):
    print(i)

# range(0, 10, 2) generates even numbers from 0 to 8
for i in range(0, 10, 2):
    print(i)

# Reverse range
for i in range(5, 0, -1):
    print(i)
```

### 3. Loop Control: break and continue

- `break`: Terminate the loop immediately
- `continue`: Skip the rest of the current iteration and move to the next one

```python
# Example with break
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
for num in numbers:
    if num == 6:
        break
    print(num)  # Prints 1, 2, 3, 4, 5

# Example with continue
for num in numbers:
    if num % 2 == 0:
        continue
    print(num)  # Prints odd numbers: 1, 3, 5, 7, 9
```

#### Loop with else Clause

```python
for i in range(5):
    print(i)
else:
    print("Loop completed without break")

for i in range(5):
    if i == 3:
        break
    print(i)
else:
    print("This won't print because of break")
```

## match Statements (Structural Pattern Matching)

Introduced in Python 3.10, the match statement is Python's version of the switch statement found in other languages, but it is significantly more powerful.

### Basic Syntax

The `_` case acts as a wildcard (similar to `default` in other languages).

```python
status = 404

match status:
    case 200:
        print("Success")
    case 404:
        print("Not Found")
    case 500 | 501:  # The | operator means "or"
        print("Server Error")
    case _:
        print("Unknown Status")
```

### Pattern Matching with Data

The match statement can also "deconstruct" data structures, which is where it truly shines compared to a standard if/else block.

```python
point = (0, 5)  # A coordinate (x, y)

match point:
    case (0, 0):
        print("At the origin")
    case (0, y):
        print(f"On the Y-axis at {y}")
    case (x, 0):
        print(f"On the X-axis at {x}")
    case (x, y):
        print(f"At point {x}, {y}")
```

In the example above, `case (0, y)` not only matches a tuple starting with 0 but also binds the second value to the variable `y` for use inside that block.

### Advanced Pattern Matching

```python
# Matching with guards
data = {"type": "user", "name": "Alice", "age": 25}

match data:
    case {"type": "user", "age": age} if age >= 18:
        print(f"Adult user: {age} years old")
    case {"type": "user", "age": age} if age < 18:
        print(f"Minor user: {age} years old")
    case {"type": "admin"}:
        print("Admin user")
    case _:
        print("Unknown data type")

# Matching with classes
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

p = Point(3, 4)

match p:
    case Point(x=0, y=y):
        print(f"On Y-axis at {y}")
    case Point(x=x, y=0):
        print(f"On X-axis at {x}")
    case Point(x=x, y=y):
        print(f"Point at ({x}, {y})")

# Matching with lists
data = [1, 2, 3, 4, 5]

match data:
    case []:
        print("Empty list")
    case [first]:
        print(f"Single element: {first}")
    case [first, second]:
        print(f"Two elements: {first}, {second}")
    case [first, *rest]:
        print(f"First: {first}, rest: {rest}")
```

### Nested Loops

Loops placed inside other loops, useful for processing multi-dimensional data structures.

```python
# Example: Multiplication table
for i in range(1, 4):
    for j in range(1, 4):
        print(f"{i} × {j} = {i * j}")
    print()  # Empty line between rows

# Example: Processing 2D list
matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]]
for row in matrix:
    for element in row:
        print(element, end=" ")
    print()
```

#### Loop Control in Nested Loops

```python
# Using break in nested loops
for i in range(3):
    for j in range(3):
        if i == 1 and j == 1:
            break  # Only breaks inner loop
        print(f"({i}, {j})")

    # Using flag to break outer loop
    found = False
    for i in range(3):
        for j in range(3):
            if i == 1 and j == 1:
                found = True
                break
        if found:
            break
```

### Loop Optimization

Techniques for improving loop performance and readability.

#### List Comprehensions

```python
# Traditional loop
squares = []
for i in range(10):
    squares.append(i ** 2)

# List comprehension (faster and more readable)
squares = [i ** 2 for i in range(10)]

# With condition
even_squares = [i ** 2 for i in range(10) if i % 2 == 0]
```

#### Generator Expressions

```python
# List comprehension (creates entire list in memory)
sum_squares = sum([i ** 2 for i in range(1000000)])

# Generator expression (lazy evaluation, memory efficient)
sum_squares = sum(i ** 2 for i in range(1000000))
```

#### Built-in Functions

```python
# Instead of manual loops
numbers = [1, 2, 3, 4, 5]

# Manual sum
total = 0
for num in numbers:
    total += num

# Built-in sum (faster)
total = sum(numbers)

# Manual filtering
evens = []
for num in numbers:
    if num % 2 == 0:
        evens.append(num)

# Built-in filter (more efficient)
evens = list(filter(lambda x: x % 2 == 0, numbers))
```

### Short-circuit Evaluation

Behavior where logical operators stop evaluating as soon as the result is determined.

```python
# AND operator: stops at first False
def expensive_check():
    print("Expensive check called")
    return True

# Second function not called because first is False
result = False and expensive_check()  # Only prints nothing

# OR operator: stops at first True
result = True or expensive_check()  # Only prints nothing

# Practical usage
def is_valid_user(username, password):
    # Check username first (cheaper) before password validation
    return username and len(password) >= 8

# Safe dictionary access
user_data = {"name": "Alice"}
# Won't raise KeyError if key doesn't exist
name = user_data.get("name") and user_data["name"].upper()
```

#### Performance Implications

```python
# Inefficient: checks expensive condition even when cheap one fails
if expensive_function() and simple_check():
    pass

# Efficient: cheap check first
if simple_check() and expensive_function():
    pass

# Default values with short-circuiting
value = user_input or "default"  # Uses default if input is falsy
```

## Exception Handling in Control Flow

Control flow isn't just about conditions and loops—it's also about handling errors gracefully.

```python
# Basic try-except
try:
    result = 10 / 0
except ZeroDivisionError:
    print("Cannot divide by zero")

# Multiple exception types
try:
    value = int("abc")
except ValueError:
    print("Invalid number format")
except TypeError:
    print("Wrong type for conversion")

# try-except-else-finally
try:
    result = 10 / 2
except ZeroDivisionError:
    print("Division error")
else:
    print(f"Result: {result}")  # Only runs if no exception
finally:
    print("This always runs")  # Cleanup code
```

## Summary

- Indentation is syntactically required to define control blocks
- `if/elif/else` handles conditional logic based on Truthiness
- `for` loops are iterators that use the `in` keyword to traverse collections
- `match` statements provide powerful structural pattern matching (Python 3.10+)
- Python's control flow is designed for readability and expressiveness
- Exception handling is an important part of robust control flow management

## Important Keywords

### **Indentation**

Python uses whitespace (typically 4 spaces) to define code blocks, unlike other languages that use curly braces or keywords.

### **Truthiness**

Concept where objects can be evaluated in boolean contexts, with specific values considered falsy and others truthy.

### **Iterable**

Any object that can be looped over, including lists, tuples, strings, dictionaries, sets, and custom objects with `__iter__`.

### **Iterator**

Object that implements the iterator protocol (`__iter__` and `__next__` methods), allowing sequential access to elements.

### **Range**

Built-in function that generates sequences of numbers, commonly used in for loops for counting.

### **Pattern Matching**

Advanced feature (Python 3.10+) that allows destructuring and matching complex data structures.

### **Structural Pattern Matching**

Type of pattern matching that examines the structure of data rather than just values, enabling powerful conditional logic.

### **Guard Conditions**

Additional conditions in match cases using `if` clauses to add more specific matching criteria.

### **Short-circuit Evaluation**

Behavior where logical operators (`and`, `or`) stop evaluating as soon as the result is determined.

### **Exception Handling**

Mechanism for managing runtime errors using `try`, `except`, `else`, and `finally` blocks.

### Control Flow

Order in which statements are executed, including conditional branching, looping, and error handling.

#### Types of Control Flow

1. **Sequential**: Statements executed one after another
2. **Selection**: Choose between different paths (if/elif/else, match)
3. **Iteration**: Repeat blocks of code (while, for)
4. **Exception Handling**: Manage errors and special conditions

```python
# Example combining multiple control flow types
def process_data(data):
    try:
        if not data:  # Selection
            return None

        processed = []
        for item in data:  # Iteration
            if item > 0:  # Selection
                processed.append(item ** 2)

        return processed
    except TypeError as e:  # Exception Handling
        print(f"Error processing data: {e}")
        return []
```

### **Nested Loops**

Loops placed inside other loops, useful for processing multi-dimensional data structures.

### **Loop Optimization**

Techniques like using list comprehensions or generator expressions instead of explicit loops for better performance.

### **Conditional Expression**

Ternary operator syntax `value_if_true if condition else value_if_false` for concise conditional assignments.

### **Enumeration**

Process of getting both index and value when iterating, typically using the `enumerate()` function.

### **Zip Function**

Built-in function that combines multiple iterables, allowing simultaneous iteration over them.
