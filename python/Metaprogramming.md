# Metaprogramming & Advanced Features

Metaprogramming is often described as *code that manipulates code*. In
Python, it usually means using the language's runtime objects (functions,
classes, modules) to build flexible and self-documenting systems.

This chapter focuses on practical tools used in libraries and frameworks:

- Introspection and reflection
- Type hints and static analysis
- Abstract Base Classes (ABCs)
- Descriptors and attribute control
- Class creation hooks (`__init_subclass__`)
- Metaclasses (`type`)

## Type Hinting and Static Analysis

Python is dynamically typed, but as projects grow, type-related bugs become
harder to spot. **Type hints** (PEP 484) let you annotate expected argument
and return types.

Type hints do not change runtime behavior by default. They are primarily
consumed by tools like `mypy` and by IDEs.

### Basic Syntax

```python
def process_user_id(user_id: int) -> str:
    return f"USER_{user_id}"
```

### The `typing` module

```python
from typing import Optional

def find_index(values: list[str], target: str) -> Optional[int]:
    for i, value in enumerate(values):
        if value == target:
            return i
    return None
```

Common `typing` concepts:

- `Optional[T]` means `T | None`
- `list[str]`, `dict[str, int]` are generic (parameterized) types
- `Any` opts out of type checking
- `Protocol` supports structural typing (duck typing with type checking)

### Runtime access: `__annotations__`

Python stores annotations on functions and classes.

```python
def f(x: int, y: int) -> int:
    return x + y

print(f.__annotations__)  # {'x': int, 'y': int, 'return': int}
```

If you need evaluated type hints (resolving forward references), use
`typing.get_type_hints()`.

## Introspection and Reflection

**Introspection** is examining objects at runtime (what is this object?
what attributes does it have?).

**Reflection** is changing behavior at runtime (setting attributes,
dynamically calling functions, loading modules).

Frameworks use these ideas for routing, dependency injection, ORMs, and
serialization.

### Common Introspection Tools

- `type(obj)`
- `isinstance(obj, cls)`
- `issubclass(cls, base)`
- `dir(obj)`
- `hasattr(obj, name)`
- `getattr(obj, name)` / `setattr(obj, name, value)`
- `callable(obj)`

```python
class SecretAgent:
    def __init__(self, name: str):
        self.name = name

spy = SecretAgent("Bond")

if hasattr(spy, "name"):
    print(getattr(spy, "name"))

setattr(spy, "codename", "007")
print(spy.codename)
```

### The `inspect` module

The `inspect` module lets you inspect signatures, source code, and
callability.

```python
import inspect

def greet(name: str, excited: bool = False) -> str:
    return f"Hi {name}{'!' if excited else '.'}"

sig = inspect.signature(greet)
print(sig)  # (name: str, excited: bool = False) -> str
```

## Abstract Base Classes (ABCs)

An **Abstract Base Class** defines a contract that subclasses must follow.
If a subclass fails to implement an `@abstractmethod`, Python raises an
error when you try to instantiate it.

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float:
        raise NotImplementedError

class Square(Shape):
    def __init__(self, side: float):
        self.side = side

    def area(self) -> float:
        return self.side * self.side

sq = Square(5)
print(sq.area())
```

ABCs are often used with the standard library's collection interfaces
(`collections.abc.Iterable`, `Mapping`, etc.).

## Descriptors (Attribute Control)

A **descriptor** is any object implementing one or more of:

- `__get__(self, instance, owner)`
- `__set__(self, instance, value)`
- `__delete__(self, instance)`

Descriptors are the mechanism behind `@property`, methods, and many ORM
field systems.

### `@property` is a descriptor

```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius

    @property
    def celsius(self) -> float:
        return self._celsius

    @celsius.setter
    def celsius(self, value: float) -> None:
        if value < -273.15:
            raise ValueError("Below absolute zero")
        self._celsius = value
```

### Custom descriptor example

```python
class PositiveNumber:
    def __set_name__(self, owner, name):
        self.private_name = f"_{name}"

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.private_name)

    def __set__(self, instance, value):
        if value <= 0:
            raise ValueError("Must be > 0")
        setattr(instance, self.private_name, value)

class Product:
    price = PositiveNumber()

    def __init__(self, price: float):
        self.price = price

p = Product(10)
# p.price = -1  # ValueError
```

## Class Creation Hooks: `__init_subclass__`

`__init_subclass__` runs automatically when a class is subclassed. This is a
lightweight alternative to metaclasses for many use cases.

```python
class PluginBase:
    registry: dict[str, type] = {}

    def __init_subclass__(cls, **kwargs):
        super().__init_subclass__(**kwargs)
        PluginBase.registry[cls.__name__] = cls

class MyPlugin(PluginBase):
    pass

print(PluginBase.registry)  # {'MyPlugin': <class '__main__.MyPlugin'>}
```

## Metaclasses (Class Creation Control)

In Python, classes are objects too. The default metaclass is `type`.

A **metaclass** can customize:

- class creation (`__new__`)
- class initialization (`__init__`)
- instance creation (`__call__`)

### Singleton via metaclass

```python
class Singleton(type):
    _instances: dict[type, object] = {}

    def __call__(cls, *args, **kwargs):
        if cls not in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kwargs)
        return cls._instances[cls]

class Database(metaclass=Singleton):
    pass

db1 = Database()
db2 = Database()
print(db1 is db2)
```

### When to avoid metaclasses

Metaclasses are powerful, but often unnecessary. Prefer:

- `__init_subclass__` for subclass registration and validation
- decorators for wrapping behavior
- descriptors for attribute logic

## Summary

- Type hints improve maintainability via static analysis.
- Introspection/reflection enable dynamic behavior in frameworks.
- ABCs formalize contracts for class hierarchies.
- Descriptors power advanced attribute behavior (including `@property`).
- `__init_subclass__` and metaclasses customize class creation.

## Important Keywords

### **Metaprogramming**

Code that inspects, generates, or modifies other code structures.

### **Type Hint / Annotation**

Metadata attached to code (parameters/returns/attributes) describing types.

### **Static Analysis**

Analyzing code without running it (e.g., `mypy`, IDE type checking).

### **Introspection**

Examining an object's structure at runtime (attributes, methods, types).

### **Reflection**

Modifying or using program structure dynamically (e.g., `setattr`).

### **Annotation Storage (`__annotations__`)**

Where Python stores annotations for functions/classes.

### **Abstract Base Class (ABC)**

Base class defining a contract, often using `@abstractmethod`.

### **Descriptor**

Object controlling attribute access via `__get__` / `__set__` / `__delete__`.

### **`@property`**

Built-in descriptor that turns methods into managed attributes.

### **`__set_name__`**

Descriptor hook called when the owning class is created.

### **`__init_subclass__`**

Hook that runs automatically when a subclass is defined.

### **Metaclass**

"Class of a class"; controls how classes are constructed.

### **`type`**

The default metaclass in Python and also a callable to create classes.

### **`__new__` / `__init__` / `__call__`**

Customization points for creating classes and instances.
