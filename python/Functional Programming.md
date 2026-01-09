# Functional Programming Tools

Python is not a purely functional language like Haskell, but it borrows powerful concepts from the functional paradigm. These tools allow you to write code that is more memory-efficient, modular, and elegant.

## Iterators and Iterables

To understand how Python loops work under the hood, we must distinguish between an Iterable and an Iterator.

- **Iterable**: An object capable of returning its members one at a time (e.g., Lists, Tuples, Strings). It implements the `__iter__` method.
- **Iterator**: The object that actually performs the traversal. It remembers its state (where it is in the sequence) and implements the `__next__` method.

### How it Works

When you use a for loop, Python calls `iter()` on the collection to get an iterator, and then calls `next()` repeatedly until a `StopIteration` exception is raised.

```python
nums = [1, 2]
it = iter(nums)

print(next(it))  # 1
print(next(it))  # 2

# print(next(it))  # Raises StopIteration
```

### Custom Iterators

You can create your own iterator classes:

```python
class CountUpTo:
    def __init__(self, max):
        self.max = max
        self.current = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.current >= self.max:
            raise StopIteration
        self.current += 1
        return self.current

# Usage
counter = CountUpTo(3)
for num in counter:
    print(num)  # 1, 2, 3
```

## Generators (yield)

Generators are a simple way to create iterators using functions. Instead of returning a value and exiting, a generator function yields a value and pauses its execution, maintaining its local state.

### Memory Efficiency

Unlike a List, which stores all its elements in memory, a Generator calculates each value on the fly (Lazy Evaluation).

```python
def count_up_to(max):
    count = 1
    while count <= max:
        yield count
        count += 1

counter = count_up_to(1000000)
print(next(counter))  # 1
print(next(counter))  # 2

# Only 1 and 2 have been "calculated" and stored; the rest don't exist yet
```

### Generator Expressions

Concise syntax for creating generators:

```python
# List comprehension (creates full list in memory)
squares_list = [x**2 for x in range(1000000)]

# Generator expression (creates generator object)
squares_gen = (x**2 for x in range(1000000))

print(squares_gen)  # <generator object <genexpr> at 0x...>
print(next(squares_gen))  # 0
print(next(squares_gen))  # 1
```

### Advanced Generator Patterns

```python
# Infinite generator
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Pipeline generators
def filter_even(numbers):
    for num in numbers:
        if num % 2 == 0:
            yield num

def square(numbers):
    for num in numbers:
        yield num ** 2

# Chain generators
def process_data(data):
    return square(filter_even(data))

# Usage
fib = fibonacci()
print(next(fib))  # 0
print(next(fib))  # 1
print(next(fib))  # 1

data = range(10)
result = list(process_data(data))  # [0, 4, 16, 36, 64]
```

## Decorators

A decorator is a function that takes another function as an argument and extends its behavior without explicitly modifying it. In JavaScript terms, this is a "Wrapper."

### Basic Syntax

Decorators use the `@decorator_name` "syntactic sugar" above the function definition.

```python
def my_decorator(func):
    def wrapper():
        print("Something is happening before the function is called.")
        func()
        print("Something is happening after the function is called.")
    return wrapper

@my_decorator
def say_hello():
    print("Hello!")

say_hello()
```

### Decorators with Arguments

To decorate functions that take arguments, we use `*args` and `**kwargs` inside the wrapper.

```python
def log_execution(func):
    def wrapper(*args, **kwargs):
        print(f"Calling {func.__name__} with {args}")
        return func(*args, **kwargs)
    return wrapper

@log_execution
def add(a, b):
    return a + b

print(add(5, 10))
```

### Advanced Decorator Patterns

#### Decorators with Arguments

```python
def repeat(times):
    def decorator(func):
        def wrapper(*args, **kwargs):
            result = None
            for _ in range(times):
                result = func(*args, **kwargs)
            return result
        return wrapper
    return decorator

@repeat(3)
def greet(name):
    print(f"Hello, {name}!")

greet("Alice")  # Prints greeting 3 times
```

#### Class-based Decorators

```python
class Timer:
    def __init__(self, func):
        self.func = func
        self.count = 0

    def __call__(self, *args, **kwargs):
        import time
        start = time.time()
        result = self.func(*args, **kwargs)
        end = time.time()
        self.count += 1
        print(f"{self.func.__name__} executed {self.count} times in {end - start:.4f}s")
        return result

@Timer
def slow_function():
    import time
    time.sleep(0.1)
    return "Done"

slow_function()  # Will print execution time
```

#### Preserving Function Metadata

```python
import functools

def decorator(func):
    @functools.wraps(func)
    def wrapper(*args, **kwargs):
        """Wrapper docstring"""
        return func(*args, **kwargs)
    return wrapper

@decorator
def my_function():
    """Original function docstring"""
    pass

print(my_function.__name__)  # my_function (preserved)
print(my_function.__doc__)   # Original function docstring (preserved)
```

## Higher-Order Functions

Functions that take other functions as arguments or return functions.

### map()

Apply a function to all items in an iterable:

```python
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))
print(squared)  # [1, 4, 9, 16, 25]

# With regular function
def double(x):
    return x * 2

doubled = list(map(double, numbers))
print(doubled)  # [2, 4, 6, 8, 10]
```

### filter()

Select items from an iterable based on a condition:

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
evens = list(filter(lambda x: x % 2 == 0, numbers))
print(evens)  # [2, 4, 6, 8, 10]

# With regular function
def is_prime(n):
    if n < 2:
        return False
    for i in range(2, int(n**0.5) + 1):
        if n % i == 0:
            return False
    return True

primes = list(filter(is_prime, numbers))
print(primes)  # [2, 3, 5, 7]
```

### reduce()

Apply a function cumulatively to items of an iterable:

```python
from functools import reduce

numbers = [1, 2, 3, 4, 5]
product = reduce(lambda x, y: x * y, numbers)
print(product)  # 120

# With initial value
sum_with_start = reduce(lambda x, y: x + y, numbers, 10)
print(sum_with_start)  # 25 (10 + 1 + 2 + 3 + 4 + 5)
```

## Functional Programming with Built-in Functions

### all() and any()

```python
numbers = [2, 4, 6, 8, 10]
print(all(x % 2 == 0 for x in numbers))  # True (all even)

mixed = [1, 2, 3, 4, 5]
print(any(x % 2 == 0 for x in mixed))  # True (some even)
```

### zip()

Combine multiple iterables:

```python
names = ["Alice", "Bob", "Charlie"]
ages = [25, 30, 35]
cities = ["NYC", "LA", "Chicago"]

combined = list(zip(names, ages, cities))
print(combined)  # [('Alice', 25, 'NYC'), ('Bob', 30, 'LA'), ('Charlie', 35, 'Chicago')]

# Unzip
names, ages, cities = zip(*combined)
```

### enumerate()

Add index to iteration:

```python
fruits = ["apple", "banana", "cherry"]
for index, fruit in enumerate(fruits):
    print(f"{index}: {fruit}")

# With custom start
for index, fruit in enumerate(fruits, start=1):
    print(f"{index}. {fruit}")
```

## Lambda Functions

Anonymous functions for simple operations:

```python
# Basic lambda
add = lambda x, y: x + y
print(add(5, 3))  # 8

# With built-in functions
numbers = [1, 2, 3, 4, 5]
squared = list(map(lambda x: x**2, numbers))

# In sorting
people = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30},
    {"name": "Charlie", "age": 20}
]

# Sort by age
sorted_by_age = sorted(people, key=lambda person: person["age"])
```

## Functional Programming Patterns

### Function Composition

```python
def compose(f, g):
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
```

### Currying

```python
def curry(func):
    def curried(*args, **kwargs):
        if len(args) + len(kwargs) >= func.__code__.co_argcount:
            return func(*args, **kwargs)
        return lambda *more_args, **more_kwargs: curried(*(args + more_args), **{**kwargs, **more_kwargs})
    return curried

@curry
def add_three(a, b, c):
    return a + b + c

add_5 = add_three(2, 3)  # Partially applied
result = add_5(10)  # 15
```

### Memoization

```python
from functools import lru_cache

@lru_cache(maxsize=128)
def fibonacci(n):
    if n < 2:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Manual memoization
def memoize(func):
    cache = {}
    def wrapper(*args):
        if args not in cache:
            cache[args] = func(*args)
        return cache[args]
    return wrapper

@memoize
def expensive_calculation(x):
    print(f"Calculating for {x}")
    return x ** 2

print(expensive_calculation(5))  # Calculates
print(expensive_calculation(5))  # Returns cached result
```

## Practical Applications

### Data Processing Pipeline

```python
def process_data(data):
    """Functional data processing pipeline."""
    return list(
        map(
            lambda x: x ** 2,
            filter(
                lambda x: x % 2 == 0,
                map(
                    lambda x: x + 1,
                    data
                )
            )
        )
    )

numbers = [1, 2, 3, 4, 5]
result = process_data(numbers)  # [4, 16, 36]
```

### Configuration Management

```python
def validate_config(config):
    """Validate configuration using functional approach."""
    required_keys = ['host', 'port', 'database']

    return all(
        key in config and config[key] is not None
        for key in required_keys
    )

config = {
    'host': 'localhost',
    'port': 5432,
    'database': 'mydb'
}

is_valid = validate_config(config)
```

### Event Handling

```python
class EventManager:
    def __init__(self):
        self.handlers = {}

    def on(self, event_name):
        def decorator(func):
            if event_name not in self.handlers:
                self.handlers[event_name] = []
            self.handlers[event_name].append(func)
            return func
        return decorator

    def emit(self, event_name, *args, **kwargs):
        if event_name in self.handlers:
            for handler in self.handlers[event_name]:
                handler(*args, **kwargs)

# Usage
events = EventManager()

@events.on('user_login')
def log_login(user):
    print(f"User {user} logged in")

@events.on('user_login')
def send_welcome_email(user):
    print(f"Sending welcome email to {user}")

events.emit('user_login', 'Alice')
```

## Summary

- Iterables are collections you can loop over; Iterators are the "pointers" that move through them
- Generators use `yield` to produce values one by one, saving massive amounts of memory
- Decorators allow you to "wrap" functions with extra logic (like logging, timing, or authentication) cleanly
- Higher-order functions like `map`, `filter`, and `reduce` enable functional data transformation
- Lambda functions provide concise anonymous function syntax
- Functional patterns like composition, currying, and memoization enable powerful abstractions

## Important Keywords

### **Iterable**

Object that can return its members one at a time, implementing the `__iter__` method for iteration.

### **Iterator**

Object that performs actual traversal, maintaining state and implementing the `__next__` method.

### **Generator**

Function that uses `yield` to produce values lazily, pausing execution between value generation.

### **Lazy Evaluation**

Strategy where expressions are evaluated only when their values are needed, improving memory efficiency.

### **Decorator**

Function that modifies or extends other functions without explicitly changing their source code.

### **Higher-Order Function**

Function that takes other functions as arguments or returns functions as results.

### **Lambda Function**

Anonymous function defined with `lambda` keyword, typically used for short, simple operations.

### **map()**

Built-in function that applies a function to all items in an iterable, returning an iterator.

### **filter()**

Built-in function that selects items from an iterable based on a predicate function condition.

### **reduce()**

Function that cumulatively applies a function to items of an iterable, reducing to single value.

### **Function Composition**

Process of combining simple functions to build more complex operations.

### **Currying**

Technique of transforming a function with multiple arguments into sequence of functions each taking single argument.

### **Memoization**

Optimization technique that stores results of expensive function calls to avoid redundant computations.

### **Closure**

Function object that remembers values in enclosing scopes even when they are not present in memory.

### **Pure Function**

Function that always returns same output for same input and has no side effects.

### **Side Effects**

Changes to system state outside function's local scope, such as modifying global variables or I/O operations.

### **Immutable**

Objects whose state cannot be modified after creation, essential for functional programming paradigms.

### **First-Class Citizen**

Functions that can be passed as arguments, returned from other functions, and assigned to variables.

### **Functional Programming**

Programming paradigm that treats computation as evaluation of mathematical functions, avoiding state and mutable data.

### **Generator Expression**

Concise syntax for creating generators using parentheses instead of brackets in comprehensions.

### **Iterator Protocol**

Interface consisting of `__iter__` and `__next__` methods that enables iteration over custom objects.

### **StopIteration**

Exception raised by iterators to signal that iteration is complete.

### **Syntactic Sugar**

Syntax that makes code easier to read or express, like the `@decorator` syntax.

### **Function Wrapping**

Technique of enclosing a function with additional logic while preserving original functionality.

### **Pipeline Pattern**

Data processing approach where output of one operation becomes input to next, enabling chained transformations.

### **Partial Application**

Technique of fixing some arguments of a function, creating new function with fewer parameters.

### **Recursion**

Function calling itself to solve problems by breaking them into smaller instances of same problem.

### **Tail Recursion**

Recursive call that is final operation in function, allowing optimization in some languages.

### **Functional Purity**

Property of functions having no side effects and always producing same output for same input.

### **Stateless Programming**

Approach where functions don't maintain state between calls, improving predictability and testability.

### **Declarative Programming**

Style expressing logic without describing control flow, focusing on what rather than how.

### **Imperative Programming**

Style describing step-by-step instructions for how computation should be performed.

### **Function Chaining**

Pattern of calling multiple functions in sequence where output of one becomes input to next.

### **Anonymous Function**

Function without a name, typically used for short-lived operations or as arguments to higher-order functions.
