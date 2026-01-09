# Functions

Functions are the primary method of code reuse in Python. In JavaScript.info style, we view functions not just as subroutines, but as "first-class citizens"—they can be assigned to variables, passed as arguments, and returned from other functions.

## Function Basics & def

A function is defined using the `def` keyword, followed by the function name, parentheses `()`, and a colon `:`. The code block inside the function is indented.

```python
def greet(name):
    """Return a greeting message."""
    return f"Hello, {name}!"

# Calling the function
message = greet("Alice")
print(message)  # Hello, Alice!
```

### Function Naming Conventions

- Use lowercase with underscores: `calculate_total`, `get_user_data`
- Be descriptive: `calculate_total` is better than `calc`
- Avoid Python keywords: don't name functions `list`, `dict`, `str`

### The return Statement

If a function does not have an explicit `return` statement, it returns `None` by default. Unlike some languages, Python allows you to return multiple values separated by commas, which actually returns a single tuple.

```python
def get_coordinates():
    return 10, 20  # Returns (10, 20)

x, y = get_coordinates()  # Unpacking

def no_return():
    print("This function returns None")

result = no_return()
print(result)  # None
```

### Early Returns

Functions can have multiple return statements for different conditions:

```python
def classify_number(num):
    if num > 0:
        return "positive"
    elif num < 0:
        return "negative"
    else:
        return "zero"
```

## Arguments: Flexibility in Input

Python functions are incredibly flexible regarding how they receive data.

### 1. Positional vs. Keyword Arguments

- **Positional**: Arguments assigned based on their order
- **Keyword**: Arguments assigned by explicitly naming the parameter

```python
def describe_pet(animal_type, pet_name):
    print(f"I have a {animal_type} named {pet_name}.")

# Positional (Order matters)
describe_pet("Hamster", "Harry")

# Keyword (Order does NOT matter)
describe_pet(pet_name="Goldie", animal_type="Fish")

# Mixed (Positional first, then keyword)
describe_pet("Cat", pet_name="Whiskers")
```

### Default Parameters

You can provide default values for parameters:

```python
def greet(name, greeting="Hello"):
    return f"{greeting}, {name}!"

print(greet("Alice"))  # Hello, Alice!
print(greet("Bob", "Hi"))  # Hi, Bob!

# Common pitfall: mutable default arguments
def add_item(item, items=[]):  # BAD: list is shared across calls
    items.append(item)
    return items

# Better approach
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

### 2. Arbitrary Arguments: *args and **kwargs

Sometimes you don't know how many arguments will be passed.

- `*args`: Collects extra positional arguments into a tuple
- `**kwargs`: Collects extra keyword arguments into a dictionary

```python
def make_pizza(size, *toppings, **details):
    print(f"Making a {size} inch pizza with:")
    for topping in toppings:
        print(f"- {topping}")
    print(f"Delivery details: {details}")

make_pizza(12, "mushrooms", "peppers", "onions", 
           address="123 Python St", express=True)

def log_message(level, message, *args, **kwargs):
    timestamp = kwargs.get('timestamp', 'now')
    print(f"[{timestamp}] {level}: {message}")
    if args:
        print(f"Additional info: {args}")

log_message("INFO", "System started", "v1.0", timestamp="12:00")
```

### Argument Unpacking

You can unpack sequences and dictionaries into function arguments:

```python
def describe_person(name, age, city):
    print(f"{name} is {age} years old and lives in {city}")

# Unpacking a list/tuple
person_data = ["Alice", 25, "New York"]
describe_person(*person_data)

# Unpacking a dictionary
person_dict = {"name": "Bob", "age": 30, "city": "Boston"}
describe_person(**person_dict)
```

## Scope: The LEGB Rule

Scope determines where a variable is accessible. Python follows the LEGB rule to resolve variable names. When you reference a name, Python looks for it in this specific order:

- **Local (L)**: Defined inside the current function
- **Enclosing (E)**: Defined in the local scope of any enclosing functions (relevant in nested functions)
- **Global (G)**: Defined at the top level of the script or module
- **Built-in (B)**: Pre-installed names like `print`, `len`, `range`

```python
x = "global"  # Global scope

def outer():
    x = "enclosing"  # Enclosing scope

    def inner():
        x = "local"  # Local scope
        print(x)  # Finds local first

    inner()
    print(x)  # Finds enclosing

outer()
print(x)  # Finds global
```

### The global and nonlocal Keywords

To modify a variable outside the local scope, you must explicitly declare it.

```python
count = 0  # Global variable

def increment():
    global count
    count += 1  # Modifies global variable

def outer():
    message = "Hello"

    def inner():
        nonlocal message
        message = "Hi"  # Modifies enclosing variable
        print(message)

    inner()
    print(message)

increment()
print(count)  # 1

outer()
```

## Higher-Order Functions

Since functions are first-class citizens, you can pass them as arguments and return them from other functions.

```python
def apply_operation(func, numbers):
    """Apply a function to a list of numbers."""
    return [func(num) for num in numbers]

def square(x):
    return x * x

def cube(x):
    return x ** 3

squares = apply_operation(square, [1, 2, 3, 4])
cubes = apply_operation(cube, [1, 2, 3, 4])

# Returning functions
def create_multiplier(factor):
    """Return a function that multiplies by factor."""
    def multiplier(x):
        return x * factor
    return multiplier

double = create_multiplier(2)
triple = create_multiplier(3)

print(double(5))  # 10
print(triple(5))  # 15
```

## Closures

A closure is a function object that remembers values in enclosing scopes even when they are not present in memory. This allows functions to maintain state between calls.

```python
def make_multiplier(factor):
    """Return a function that multiplies by factor."""
    def multiplier(x):
        return x * factor  # 'factor' is captured from enclosing scope
    return multiplier

# Each closure maintains its own 'factor'
double = make_multiplier(2)
triple = make_multiplier(3)

print(double(10))  # 20
print(triple(10))  # 30

# Practical example: configuration
def create_greeter(greeting):
    def greeter(name):
        return f"{greeting}, {name}!"
    return greeter

hello = create_greeter("Hello")
goodbye = create_greeter("Goodbye")

print(hello("Alice"))    # Hello, Alice!
print(goodbye("Bob"))   # Goodbye, Bob!
```

### Closure Applications

```python
# Memoization with closures
def memoize(func):
    cache = {}

    def memoized_func(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return memoized_func

@memoize
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)
```

## Function Composition

Function composition is the process of combining simple functions to build more complex operations, often by passing functions as arguments.

```python
def compose(f, g):
    """Return a function that is f(g(x))."""
    return lambda x: f(g(x))

def add_one(x):
    return x + 1

def multiply_by_two(x):
    return x * 2

# Compose functions
add_then_multiply = compose(multiply_by_two, add_one)
multiply_then_add = compose(add_one, multiply_by_two)

print(add_then_multiply(5))   # multiply_by_two(add_one(5)) = 12
print(multiply_then_add(5))   # add_one(multiply_by_two(5)) = 11

# Practical composition with built-in functions
def process_text(text):
    """Chain multiple string operations."""
    # Remove whitespace, convert to lowercase, then capitalize
    return text.strip().lower().capitalize()

print(process_text("   HELLO WORLD   "))  # Hello world
```

### Composition Patterns

```python
# Pipeline pattern
def pipeline(data, *functions):
    """Apply multiple functions in sequence."""
    result = data
    for func in functions:
        result = func(result)
    return result

def clean_data(data):
    return pipeline(data, 
                    lambda x: [item for item in x if item],  # Remove None
                    lambda x: sorted(set(x)),               # Remove duplicates
                    lambda x: x[:5])                          # Take first 5

messy_data = [1, 2, None, 2, 3, None, 4, 1]
clean = clean_data(messy_data)
print(clean)  # [1, 2, 3, 4]
```

## Recursion

Recursion is a technique where a function calls itself to solve problems that can be broken down into smaller instances of the same problem.

### Basic Recursion

```python
def factorial(n):
    """Calculate factorial recursively."""
    if n <= 1:
        return 1
    return n * factorial(n - 1)

print(factorial(5))  # 120

def fibonacci(n):
    """Calculate nth Fibonacci number recursively."""
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

print(fibonacci(10))  # 55
```

### Recursive vs Iterative

```python
# Recursive approach (elegant but may hit recursion limit)
def sum_recursive(lst):
    if not lst:
        return 0
    return lst[0] + sum_recursive(lst[1:])

# Iterative approach (more efficient for large lists)
def sum_iterative(lst):
    total = 0
    for item in lst:
        total += item
    return total

# Tail recursion optimization (Python doesn't optimize automatically)
def sum_tail_recursive(lst, acc=0):
    if not lst:
        return acc
    return sum_tail_recursive(lst[1:], acc + lst[0])
```

### Practical Recursion Examples

```python
# Directory traversal
def list_files(directory, extension=None):
    """Recursively list files in directory."""
    import os
    files = []

    for item in os.listdir(directory):
        full_path = os.path.join(directory, item)
        if os.path.isdir(full_path):
            files.extend(list_files(full_path, extension))
        elif extension is None or item.endswith(extension):
            files.append(full_path)

    return files

# Tree processing
def tree_depth(tree):
    """Calculate depth of nested structure."""
    if not isinstance(tree, (list, tuple)):
        return 0
    if not tree:
        return 1
    return 1 + max(tree_depth(item) for item in tree)

# Example usage
nested = [1, [2, [3, [4, 5]], 6]
print(tree_depth(nested))  # 3
```

### Recursion Best Practices

```python
import sys

# Increase recursion limit if needed
sys.setrecursionlimit(2000)

# Always have base case
def safe_recursive(data, depth=0):
    if depth > 1000:  # Prevent infinite recursion
        raise RecursionError("Too deep")
    if not data:  # Base case
        return 0
    return safe_recursive(data[1:], depth + 1) + 1
```

## Decorators

Decorators are functions that modify other functions. They are a powerful way to extend function behavior.

```python
def timing_decorator(func):
    """Decorator to time function execution."""
    import time

    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(f"{func.__name__} took {end - start:.4f} seconds")
        return result
    return wrapper

@timing_decorator
def slow_function():
    import time
    time.sleep(0.1)
    return "Done"

slow_function()

# Multiple decorators
def uppercase_decorator(func):
    def wrapper(*args, **kwargs):
        result = func(*args, **kwargs)
        return result.upper()
    return wrapper

@uppercase_decorator
@timing_decorator
def greet(name):
    return f"Hello, {name}!"

print(greet("Alice"))
```

## Lambda Functions (Anonymous Functions)

A lambda function is a small, anonymous function defined without a name. It is restricted to a single expression.

**Syntax**: `lambda arguments: expression`

```python
# Standard function
def square(x):
    return x * x

# Equivalent lambda
square_lambda = lambda x: x * x

print(square_lambda(5))  # 25
```

### When to Use Lambdas?

Lambdas are best used as "throw-away" functions, often passed as arguments to higher-order functions like `map()`, `filter()`, or `sorted()`.

```python
pairs = [(1, 'one'), (2, 'two'), (3, 'three'), (4, 'four')]

# Sort list based on second element of tuple (alphabetical)
pairs.sort(key=lambda pair: pair[1])
print(pairs)  # [(4, 'four'), (1, 'one'), (3, 'three'), (2, 'two')]

# Using with map()
numbers = [1, 2, 3, 4, 5]
squares = list(map(lambda x: x ** 2, numbers))

# Using with filter()
even_numbers = list(filter(lambda x: x % 2 == 0, numbers))

# Using with reduce()
from functools import reduce
product = reduce(lambda x, y: x * y, numbers, 1)
```

### Lambda Limitations

- Can only contain expressions (no statements)
- Cannot have multiple lines
- No assignments or `print` statements
- Limited readability for complex logic

## Function Documentation

### Docstrings

Docstrings provide documentation for your functions:

```python
def calculate_area(length, width):
    """
    Calculate the area of a rectangle.

    Args:
        length (float): The length of the rectangle
        width (float): The width of the rectangle

    Returns:
        float: The area of the rectangle

    Raises:
        ValueError: If length or width is negative

    Example:
        >>> calculate_area(5, 3)
        15
    """
    if length < 0 or width < 0:
        raise ValueError("Dimensions must be positive")
    return length * width
```

### Type Hints

Python 3.5+ supports type hints for better code documentation:

```python
from typing import List, Dict, Optional, Union

def process_data(
    data: List[int], 
    multiplier: float = 1.0
) -> List[float]:
    """
    Process a list of integers.

    Args:
        data: List of integers to process
        multiplier: Factor to multiply each number by

    Returns:
        List of processed floats
    """
    return [x * multiplier for x in data]

# Complex type hints
from typing import Dict, List, Optional

def find_user(
    user_id: int, 
    database: Dict[int, str]
) -> Optional[str]:
    """Find user by ID or return None if not found."""
    return database.get(user_id)
```

## Function Best Practices

### Do One Thing

Functions should have a single, clear responsibility:

```python
# Bad: Does too many things
def process_user_input(input_data):
    # Validate input
    # Parse data
    # Save to database
    # Send email
    # Log activity
    pass

# Good: Single responsibility
def validate_input(input_data):
    pass

def parse_data(input_data):
    pass

def save_to_database(data):
    pass

def send_notification(user, message):
    pass

def log_activity(activity):
    pass
```

### Pure Functions

Pure functions have no side effects and always return the same output for the same input:

```python
# Pure function (predictable, testable)
def add(a, b):
    return a + b

# Impure function (side effects)
def add_and_print(a, b):
    result = a + b
    print(f"Result: {result}")  # Side effect
    return result
```

### Error Handling

```python
def safe_divide(a, b):
    """Safely divide two numbers."""
    try:
        return a / b
    except ZeroDivisionError:
        return float('inf')  # Return infinity for division by zero
    except TypeError:
        raise ValueError("Both arguments must be numbers")
```

## Summary

- `def` creates named functions; `return` sends back data (or `None`)
- `*args` and `**kwargs` allow for a variable number of inputs
- LEGB is the "search map" Python uses to find your variables
- Lambdas are concise, one-line functions used for simple operations
- Functions are first-class citizens that can be passed as arguments and returned
- Decorators provide a clean way to modify function behavior
- Type hints and docstrings improve code documentation and maintainability

## Important Keywords

### **First-Class Citizens**

Objects that can be assigned to variables, passed as arguments, and returned from other functions. Functions in Python have this property.

### **Positional Arguments**

Arguments passed to a function based on their position in the function call, matched to parameters in the same order.

### **Keyword Arguments**

Arguments passed to a function by explicitly naming the parameter, allowing any order and improving readability.

### **Default Parameters**

Parameters that automatically take a specified value if no argument is provided in the function call.

### **Arbitrary Arguments**

Special syntax (`*args`, `**kwargs`) that allows functions to accept any number of additional positional or keyword arguments.

### **LEGB Rule**

The order Python uses to resolve variable names: Local, Enclosing, Global, Built-in scopes.

### **Scope**

The region of code where a variable is accessible and can be referenced.

### **Global Scope**

Variables defined at the top level of a module or script, accessible throughout the program.

### **Local Scope**

Variables defined inside a function, only accessible within that function.

### **Enclosing Scope**

Variables defined in an outer function that are accessible to nested functions (closure).

### **Higher-Order Functions**

Functions that take other functions as arguments or return functions as results.

### **Decorator**

A function that modifies or extends the behavior of another function without changing its source code.

### **Lambda Function**

Anonymous, single-expression function defined with `lambda` keyword, often used for simple operations.

### **Pure Function**

A function that always produces the same output for the same input and has no side effects.

### **Side Effects**

Changes to program state outside the function's return value, such as modifying global variables or printing.

### **Type Hints**

Optional annotations that specify the expected types of function parameters and return values.

### **Docstrings**

String literals that occur as the first statement in a module, function, class, or method definition, used for documentation.

### **Argument Unpacking**

Syntax (`*args`, `**kwargs`) that expands sequences or dictionaries into positional or keyword arguments.

### **Closure**

Function object that remembers values in enclosing scopes even when they are not present in memory.

### **Function Composition**

Combining simple functions to build more complex operations, often by passing functions as arguments.

### **Recursion**

A technique where a function calls itself to solve problems that can be broken down into smaller instances of the same problem.
