# Data Structures (Deep Dive)

In Python, choosing the right data structure is half the battle. This section explores four built-in collection types. A critical concept to keep in mind throughout this chapter is mutability: whether an object can be changed after it is created.

## Lists: Mutable Sequences

Lists are ordered, mutable collections that can hold a mix of different data types. They are the "workhorse" of Python.

### 1. Methods and Manipulation

Lists come with a rich set of built-in methods to modify their contents:

```python
fruits = ["apple", "banana"]

fruits.append("cherry")    # Adds to end: ["apple", "banana", "cherry"]
fruits.insert(1, "orange") # Inserts at index 1: ["apple", "orange", "banana", "cherry"]
fruits.remove("banana")    # Removes first occurrence
popped_item = fruits.pop() # Removes and returns last item
```

#### More List Methods

```python
numbers = [3, 1, 4, 1, 5, 9, 2]

# Sorting
numbers.sort()           # Sorts in place: [1, 1, 2, 3, 4, 5, 9]
sorted_numbers = sorted(numbers)  # Returns new sorted list

# Counting and finding
count = numbers.count(1)  # 2
index = numbers.index(4)   # 2

# Extending and clearing
numbers.extend([6, 7, 8])  # Add multiple items
numbers.clear()               # Remove all items

# Reversing
numbers.reverse()  # Reverse in place
reversed_list = list(reversed(numbers))  # Return reversed iterator
```

### 2. Slicing

Slicing allows you to retrieve a subset of a list using the syntax `list[start:stop:step]`.

```python
nums = [0, 10, 20, 30, 40, 50]

print(nums[1:4])   # [10, 20, 30] (index 4 is excluded)
print(nums[:3])     # [0, 10, 20] (from start)
print(nums[::2])    # [0, 20, 40] (every second item)
print(nums[::-1])    # [50, 40, 30, 20, 10, 0] (reverse list)
print(nums[2:])      # [20, 30, 40, 50] (from index 2 to end)
print(nums[-3:-1])   # [30, 40] (last 3 elements)
```

### 3. Nesting

Lists can contain other lists, creating multi-dimensional structures (matrices).

```python
matrix = [
    [1, 2, 3],
    [4, 5, 6]
]
print(matrix[1][0])  # 4

# 3D matrix
cube = [
    [[1, 2], [3, 4]],
    [[5, 6], [7, 8]]
]
print(cube[1][0][1])  # 6
```

#### Matrix Operations

```python
# Transpose a matrix
def transpose(matrix):
    return [[row[i] for row in matrix] for i in range(len(matrix[0]))]

matrix = [[1, 2, 3], [4, 5, 6]]
transposed = transpose(matrix)  # [[1, 4], [2, 5], [3, 6]]

# Flatten nested list
def flatten(nested_list):
    result = []
    for item in nested_list:
        if isinstance(item, list):
            result.extend(flatten(item))
        else:
            result.append(item)
    return result

nested = [[1, 2], [3, [4, 5]], 6]
flat = flatten(nested)  # [1, 2, 3, 4, 5, 6]
```

## Tuples: Immutable Sequences

Tuples are similar to lists but with one major difference: they are immutable. Once created, you cannot add, remove, or change elements.

### Packing and Unpacking

Tuples are often used for "packing" multiple values into a single variable and "unpacking" them back out.

```python
# Packing
point = (10, 20, 30)

# Unpacking
x, y, z = point
print(x)  # 10
```

### Extended Unpacking (Python 3+)

```python
first, *middle, last = [1, 2, 3, 4, 5]
# first=1, middle=[2, 3, 4], last=5

# Function return unpacking
def get_coordinates():
    return (10, 20, 30)

x, y, z = get_coordinates()

# Swapping variables (Pythonic way)
a, b = 5, 10
a, b = b, a  # a=10, b=5
```

### Tuple Methods and Operations

```python
# Creating tuples
empty = ()
single = (1,)  # Note the comma
nested = (1, (2, 3), 4)

# Tuple operations
t1 = (1, 2, 3)
t2 = (4, 5, 6)
combined = t1 + t2  # (1, 2, 3, 4, 5, 6)
repeated = t1 * 2    # (1, 2, 3, 1, 2, 3)

# Built-in methods
numbers = (1, 2, 3, 2, 1)
count = numbers.count(2)     # 2
index = numbers.index(3)     # 2
```

### When to Use Tuples

- **Fixed data**: Coordinates, RGB values, database records
- **Dictionary keys**: Tuples are hashable, can be dictionary keys
- **Function returns**: Multiple return values
- **Data integrity**: Prevent accidental modification

```python
# Dictionary keys
point_dict = {
    (0, 0): "origin",
    (1, 0): "unit_x",
    (0, 1): "unit_y"
}

# Data integrity
def get_user_info():
    return ("Alice", 25, "alice@example.com")  # Can't be accidentally modified
```

## Dictionaries: Key-Value Pairs

Dictionaries (dict) are unordered (technically ordered by insertion since Python 3.7), mutable mappings. They work like a real-world dictionary or a JSON object.

### Key-Value Logic and Hashing

- **Keys must be hashable** (immutable types like strings, numbers, or tuples)
- **Values can be anything**
- **Performance**: Looking up a key in a dictionary is O(1)—extremely fast—because Python uses a hash table internally

```python
user = {
    "name": "Guido",
    "role": "BDFL",
    "active": True
}

print(user["name"])        # Accessing value
user["role"] = "Retired"  # Updating value
print(user.get("email", "N/A"))  # Safe access with default value
```

### Dictionary Methods and Operations

```python
# Creating dictionaries
empty = {}
from_keys = dict.fromkeys(['a', 'b', 'c'], 0)  # {'a': 0, 'b': 0, 'c': 0}
from_pairs = dict([('x', 1), ('y', 2)])  # {'x': 1, 'y': 2}

# Adding and removing
user['email'] = "guido@python.org"
del user['active']
removed = user.pop('role', None)  # Remove with default

# Iteration
for key in user:           # Iterate over keys
    print(f"{key}: {user[key]}")

for key, value in user.items():  # Iterate over key-value pairs
    print(f"{key} = {value}")

# Dictionary views
keys = user.keys()      # dict_keys(['name', 'role', 'active'])
values = user.values()  # dict_values(['Guido', 'BDFL', True])
items = user.items()    # dict_items([('name', 'Guido'), ...])
```

### Nested Dictionaries

```python
# Complex data structures
database = {
    "users": {
        "alice": {
            "name": "Alice Smith",
            "age": 25,
            "posts": 42
        },
        "bob": {
            "name": "Bob Jones",
            "age": 30,
            "posts": 15
        }
    },
    "total_posts": 57
}

# Accessing nested data
alice_age = database["users"]["alice"]["age"]
alice_posts = database["users"]["alice"].get("posts", 0)

# Safe nested access
def safe_get(dictionary, *keys, default=None):
    """Safely get nested dictionary values."""
    current = dictionary
    for key in keys:
        if isinstance(current, dict) and key in current:
            current = current[key]
        else:
            return default
    return current

age = safe_get(database, "users", "alice", "age", 0)  # 25
```

### Dictionary Comprehensions

```python
# Create from list comprehension
squares = {x: x**2 for x in range(5)}  # {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}

# Filter and transform
scores = {"alice": 85, "bob": 92, "charlie": 78}
high_scores = {name: score for name, score in scores.items() if score >= 80}

# Invert dictionary
original = {"a": 1, "b": 2, "c": 3}
inverted = {v: k for k, v in original.items()}  # {1: 'a', 2: 'b', 3: 'c'}
```

## Sets: Unique Collections

A set is an unordered collection of unique elements. They are used to eliminate duplicates and perform mathematical set operations.

### Set Theory Math

Python sets support operations identical to mathematical set theory:

```python
a = {1, 2, 3, 4}
b = {3, 4, 5, 6}

print(a | b)  # Union: {1, 2, 3, 4, 5, 6}
print(a & b)  # Intersection: {3, 4}
print(a - b)  # Difference: {1, 2}
print(a ^ b)  # Symmetric Difference: {1, 2, 5, 6}

# Subset and superset
c = {1, 2}
print(c.issubset(a))  # True
print(a.issuperset(c))  # True
```

### Set Methods and Operations

```python
# Creating sets
empty = set()
from_list = set([1, 2, 2, 3, 1])  # {1, 2, 3}
from_string = set("hello")  # {'h', 'e', 'l', 'o'}

# Adding and removing
numbers = {1, 2, 3}
numbers.add(4)        # {1, 2, 3, 4}
numbers.remove(2)      # {1, 3, 4} (raises KeyError if not found)
numbers.discard(2)     # {1, 3, 4} (no error if not found)

# Set operations
set1 = {1, 2, 3}
set2 = {3, 4, 5}
set1.update(set2)      # {1, 2, 3, 4, 5}
set1.intersection_update(set2)  # {3}
set1.difference_update(set2)   # {1, 2}
```

### Practical Set Applications

```python
# Remove duplicates from list
def remove_duplicates(items):
    return list(set(items))

duplicates = [1, 2, 2, 3, 1, 4, 3]
unique = remove_duplicates(duplicates)  # [1, 2, 3, 4]

# Find common elements
def common_elements(list1, list2):
    return list(set(list1) & set(list2))

list1 = [1, 2, 3, 4, 5]
list2 = [4, 5, 6, 7, 8]
common = common_elements(list1, list2)  # [4, 5]

# Membership testing
allowed_users = {"alice", "bob", "charlie"}
current_user = "alice"

if current_user in allowed_users:
    print("Access granted")
else:
    print("Access denied")
```

## Comprehensions: The "Pythonic" Way

Comprehensions provide a concise way to create new collections based on existing ones. They replace messy for loops.

### List Comprehension

**Syntax**: `[expression for item in iterable if condition]`

{% tabs %}

{% tab title="Instead of this" %}

```python
# Instead of this
squares = []
for x in range(5):
    squares.append(x**2)
```

{% endtab %}

{% tab title="Do this" %}

```python
# Do this
squares = [x**2 for x in range(5)]  # [0, 1, 4, 9, 16]

# With a condition
evens = [x for x in range(10) if x % 2 == 0]

# Nested comprehension
matrix = [[i*j for j in range(3)] for i in range(3)]
# [[0, 0, 0], [1, 1, 1], [2, 2, 2]]

# Flattening with comprehension
nested = [[1, 2], [3, [4, 5]], 6]
flat = [item for sublist in nested for item in sublist]
```

{% endtab %}

{% endtabs %}

### Dictionary & Set Comprehensions

You can apply the same logic to create dictionaries and sets:

```python
# Dict Comprehension
square_dict = {x: x**2 for x in range(3)}  # {0: 0, 1: 1, 2: 4}

# Set Comprehension
unique_chars = {char.upper() for char in "apple"}  # {'A', 'P', 'L', 'E'}

# Complex dict comprehension
words = ["apple", "banana", "cherry"]
word_lengths = {word: len(word) for word in words}  # {'apple': 5, 'banana': 6, 'cherry': 6}

# Conditional dict comprehension
numbers = [1, 2, 3, 4, 5]
even_squares = {x: x**2 for x in numbers if x % 2 == 0}  # {2: 4, 4: 16}
```

## Advanced Data Structure Concepts

### Mutability vs Immutability

```python
# Mutable (can be changed)
my_list = [1, 2, 3]
my_list[0] = 100  # OK
my_dict = {"a": 1}
my_dict["b"] = 2  # OK

# Immutable (cannot be changed)
my_tuple = (1, 2, 3)
# my_tuple[0] = 100  # TypeError!
my_string = "hello"
# my_string[0] = "H"  # TypeError!
```

### Memory and Performance

<details>
<summary>Show memory and performance snippet</summary>

```python
import sys

# Memory usage
list_size = sys.getsizeof([1, 2, 3, 4, 5])
tuple_size = sys.getsizeof((1, 2, 3, 4, 5))
dict_size = sys.getsizeof({"a": 1, "b": 2})

# Performance comparison
import timeit

# List lookup (O(n))
list_lookup = """
my_list = list(range(1000))
999 in my_list
"""

# Set lookup (O(1))
set_lookup = """
my_set = set(range(1000))
999 in my_set
"""

print(timeit.timeit(list_lookup, number=1000))
print(timeit.timeit(set_lookup, number=1000))
```

</details>

### Choosing the Right Data Structure

<details>
<summary>Show choosing-the-right-structure checklist</summary>

```python
# Use Lists when:
# - Order matters
# - Duplicates are allowed
# - You need to modify frequently
# - You need indexing/slicing

# Use Tuples when:
# - Data should not change
# - You need dictionary keys
# - Returning multiple values from functions

# Use Dictionaries when:
# - You need key-value mapping
# - Fast lookups are important
# - Data has unique identifiers

# Use Sets when:
# - You need unique elements
# - You need mathematical set operations
# - Membership testing is frequent
# - Order doesn't matter
```

</details>

## Summary

- Lists are mutable and ordered (use for sequences)
- Tuples are immutable (use for fixed data/records)
- Dictionaries are key-value maps (use for fast lookups)
- Sets store unique items and support mathematical operations
- Comprehensions make code more readable and often more performant

## Important Keywords

### **Mutability**

Property of an object that determines whether its value can be changed after creation. Lists and dictionaries are mutable; tuples and strings are immutable.

### **Sequence**

Ordered collection that can be indexed and sliced. Lists, tuples, and strings are sequences.

### **Hashable**

Object that can be used as a dictionary key or set element. Must be immutable and have a `__hash__()` method.

### **O(1) Complexity**

Constant time complexity, meaning operation time doesn't grow with input size. Dictionary lookups and set operations are O(1).

### **Slicing**

Syntax for extracting subsets of sequences using `start:stop:step` notation.

### **Packing**

Combining multiple values into a single tuple or list using `*` operator or comma separation.

### **Unpacking**

Extracting values from sequences into separate variables using assignment or `*` operator.

### **Nested Structures**

Data structures contained within other data structures, creating multi-dimensional representations like matrices or trees.

### **Comprehension**

Concise syntax for creating new collections by transforming and filtering existing ones.

### **Set Theory**

Mathematical principles of union, intersection, difference, and symmetric difference applied to Python sets.

### **Dictionary Comprehension**

Creating dictionaries using expression syntax `{key_expr: value_expr for item in iterable if condition}`.

### **List Comprehension**

Creating lists using expression syntax `[expression for item in iterable if condition]`.

### **Set Comprehension**

Creating sets using expression syntax `{expression for item in iterable if condition}`.

### **Immutable**

Objects whose state cannot be modified after creation, providing data integrity and thread safety.

### **Ordered Collection**

Data structure that maintains insertion order and allows access by position/index.

### **Collection**

Generic term for container objects that hold multiple items like lists, tuples, sets, and dictionaries.

### **Hash Table**

Underlying data structure used by dictionaries and sets for O(1) average-time lookups.

### **Time Complexity**

Measure of how algorithm execution time scales with input size, crucial for choosing appropriate data structures.
