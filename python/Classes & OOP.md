# Classes & Object-Oriented Programming (OOP)

Python is fundamentally object-oriented. In fact, every string, integer, and function you have used so far is an object. This chapter moves from using objects to designing them.

## Class Syntax and self

A Class is a blueprint; an Instance (Object) is a house built from that blueprint.

### The Role of self

In Python, `self` represents the specific instance of an object. It is the first argument to every instance method, allowing code to distinguish between "my data" and "the class's data."

```python
class Robot:
    def __init__(self, name, model):
        # Attributes (Instance Variables)
        self.name = name
        self.model = model

    def identify(self):
        return f"I am {self.name}, a {self.model} unit."

# Creating an instance
r1 = Robot("R2-D2", "Astromech")
print(r1.identify())
```

### Class and Instance Attributes

```python
class Car:
    # Class attribute (shared by all instances)
    wheels = 4

    def __init__(self, brand, color):
        # Instance attributes (unique to each instance)
        self.brand = brand
        self.color = color

car1 = Car("Toyota", "Red")
car2 = Car("Honda", "Blue")

print(car1.wheels)  # 4 (class attribute)
print(car1.brand)   # Toyota (instance attribute)

# Modify class attribute
Car.wheels = 6
print(car1.wheels)  # 6 (affects all instances)
```

## Inheritance and MRO

Inheritance allows a class (Child) to derive attributes and methods from another class (Parent). This promotes code reuse.

### Method Resolution Order (MRO)

Python supports Multiple Inheritance, which can get complex. To decide which method to run if multiple parents have the same name, Python uses the MRO (accessible via `ClassName.mro()`).

```python
class Machine:
    def power_on(self):
        print("System Online.")

class Drone(Machine):  # Inherits from Machine
    def fly(self):
        print("Taking off...")

my_drone = Drone()
my_drone.power_on()  # Inherited method
my_drone.fly()       # Child method

# Multiple inheritance
class Flyable:
    def fly(self):
        print("Flying with wings...")

class Swimmable:
    def swim(self):
        print("Swimming with fins...")

class Duck(Flyable, Swimmable):
    def quack(self):
        print("Quack!")

duck = Duck()
duck.fly()   # From Flyable
duck.swim()  # From Swimmable
duck.quack() # From Duck

# Check MRO
print(Duck.mro())
# [<class '__main__.Duck'>, <class '__main__.Swimmable'>, 
#  <class '__main__.Flyable'>, <class 'object'>]
```

### super() Function

The `super()` function allows you to call parent class methods:

```python
class Animal:
    def speak(self):
        return "Some sound"

class Dog(Animal):
    def speak(self):
        # Call parent method
        parent_sound = super().speak()
        return f"{parent_sound} - Woof!"

dog = Dog()
print(dog.speak())  # Some sound - Woof!
```

## Dunder (Magic) Methods

"Dunder" stands for Double Underscore. These methods allow your custom objects to hook into Python's built-in syntax (like `+`, `len()`, or `print()`).

| Method | Triggered by... | Purpose |
|---------|------------------|---------|
| `__init__` | `Object()` | Initializing object's state. |
| `__str__` | `print(obj)` | User-friendly string representation. |
| `__repr__` | `repr(obj)` | Developer-friendly debugging string. |
| `__len__` | `len(obj)` | Defines what "length" means for your object. |
| `__add__` | `obj1 + obj2` | Overloading addition operator. |
| `__eq__` | `obj1 == obj2` | Equality comparison. |
| `__lt__` | `obj1 < obj2` | Less-than comparison. |
| `__getitem__` | `obj[key]` | Dictionary-like access. |
| `__setitem__` | `obj[key] = value` | Dictionary-like assignment. |
| `__call__` | `obj()` | Make object callable. |

```python
class Book:
    def __init__(self, title, pages):
        self.title = title
        self.pages = pages

    def __str__(self):
        return f"'{self.title}' ({self.pages} pages)"

    def __len__(self):
        return self.pages

    def __add__(self, other):
        # Combine two books
        return Book(
            f"{self.title} & {other.title}",
            self.pages + other.pages
        )

my_book = Book("Pythonic Ways", 300)
print(my_book)     # Calls __str__
print(len(my_book))  # Calls __len__

book2 = Book("Advanced Python", 400)
combined = my_book + book2  # Calls __add__
print(combined.pages)  # 700
```

### Comparison Methods

```python
class Student:
    def __init__(self, name, gpa):
        self.name = name
        self.gpa = gpa

    def __eq__(self, other):
        if not isinstance(other, Student):
            return False
        return self.gpa == other.gpa

    def __lt__(self, other):
        if not isinstance(other, Student):
            return NotImplemented
        return self.gpa < other.gpa

    def __repr__(self):
        return f"Student(name={self.name!r}, gpa={self.gpa})"

s1 = Student("Alice", 3.8)
s2 = Student("Bob", 3.5)
s3 = Student("Charlie", 3.8)

print(s1 == s2)  # False
print(s1 == s3)  # True
print(s1 < s2)   # False
print(s2 < s1)   # True

# Sorting works with __lt__
students = [s1, s2, s3]
sorted_students = sorted(students)
print(sorted_students)  # Sorted by GPA
```

## Properties: @property, Getters, and Setters

In languages like Java, you write `getAge()` and `setAge()`. In Python, we use the `@property` decorator. This allows you to access a method like an attribute while still maintaining control over validation.

```python
class Employee:
    def __init__(self, salary):
        self._salary = salary  # The underscore suggests it's 'private'

    @property
    def salary(self):
        """The Getter"""
        return f"${self._salary:,}"

    @salary.setter
    def salary(self, value):
        """The Setter with validation"""
        if value < 0:
            raise ValueError("Salary cannot be negative!")
        self._salary = value

emp = Employee(50000)
print(emp.salary)    # Access like an attribute -> $50,000
emp.salary = 60000  # Validates through the setter

# Read-only property
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def area(self):
        """Read-only calculated property"""
        import math
        return math.pi * self._radius ** 2

circle = Circle(5)
print(circle.area)  # 78.53981633974483
# circle.area = 100  # AttributeError: can't set attribute
```

### Computed Properties

```python
class Rectangle:
    def __init__(self, width, height):
        self.width = width
        self.height = height

    @property
    def area(self):
        return self.width * self.height

    @property
    def perimeter(self):
        return 2 * (self.width + self.height)

rect = Rectangle(5, 3)
print(f"Area: {rect.area}, Perimeter: {rect.perimeter}")
```

## Class vs. Static Methods

- `@classmethod`: Receives the class (`cls`) as first argument. Used for "factory methods" that create instances in different ways.
- `@staticmethod`: Receives neither `self` nor `cls`. It's just a regular function that lives inside the class namespace because it's logically related.

```python
class DateConverter:
    @staticmethod
    def is_valid_year(year):
        return 1900 <= year <= 2100

    @classmethod
    def from_string(cls, date_string):
        """Factory method to create DateConverter from string"""
        day, month, year = map(int, date_string.split('/'))
        if cls.is_valid_year(year):
            return cls(day, month, year)
        raise ValueError("Invalid year")

# Static method usage
print(DateConverter.is_valid_year(2024))  # True

# Class method usage
date_obj = DateConverter.from_string("25/12/2024")
print(date_obj)  # DateConverter instance
```

## Advanced OOP Concepts

### Encapsulation

```python
class BankAccount:
    def __init__(self, account_number, balance):
        self._account_number = account_number  # Protected
        self.__balance = balance              # Private

    def deposit(self, amount):
        if amount > 0:
            self.__balance += amount

    def get_balance(self):
        return self.__balance

    # Name mangling makes __balance hard to access
    # _BankAccount__balance is the actual name

account = BankAccount("12345", 1000)
account.deposit(500)
print(account.get_balance())  # 1500
# print(account.__balance)  # AttributeError
```

### Polymorphism

```python
class Shape:
    def area(self):
        raise NotImplementedError("Subclasses must implement area()")

class Rectangle(Shape):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

class Circle(Shape):
    def __init__(self, radius):
        self.radius = radius

    def area(self):
        import math
        return math.pi * self.radius ** 2

def print_area(shape):
    print(f"Area: {shape.area()}")

# Polymorphism: same interface, different implementations
rect = Rectangle(5, 3)
circle = Circle(2)

print_area(rect)    # Rectangle's area method
print_area(circle)   # Circle's area method
```

### Abstract Base Classes

```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def make_sound(self):
        pass

    @abstractmethod
    def move(self):
        pass

class Dog(Animal):
    def make_sound(self):
        return "Woof!"

    def move(self):
        return "Running"

class Bird(Animal):
    def make_sound(self):
        return "Tweet!"

    def move(self):
        return "Flying"

# Cannot instantiate abstract class
# animal = Animal()  # TypeError: Can't instantiate abstract class

dog = Dog()
bird = Bird()

print(dog.make_sound())  # Woof!
print(bird.move())      # Flying
```

### Composition over Inheritance

```python
class Engine:
    def start(self):
        print("Engine starting...")

class Wheels:
    def rotate(self):
        print("Wheels rotating...")

class Car:
    def __init__(self):
        self.engine = Engine()  # Composition
        self.wheels = Wheels()  # Composition

    def drive(self):
        self.engine.start()
        self.wheels.rotate()
        print("Car moving...")

# Car HAS-A engine and wheels, rather than IS-A
car = Car()
car.drive()
```

## Special OOP Patterns

### Singleton Pattern

```python
class Singleton:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
        return cls._instance

    def __init__(self):
        # Initialize only once
        pass

s1 = Singleton()
s2 = Singleton()
print(s1 is s2)  # True (same instance)
```

### Observer Pattern

```python
class Subject:
    def __init__(self):
        self._observers = []

    def attach(self, observer):
        self._observers.append(observer)

    def detach(self, observer):
        self._observers.remove(observer)

    def notify(self, message):
        for observer in self._observers:
            observer.update(message)

class Observer:
    def update(self, message):
        print(f"Received: {message}")

# Usage
subject = Subject()
observer1 = Observer()
observer2 = Observer()

subject.attach(observer1)
subject.attach(observer2)
subject.notify("Hello Observers!")
```

### Data Classes

```python
from dataclasses import dataclass
from typing import List

@dataclass
class Person:
    name: str
    age: int
    emails: List[str] = None

    def __post_init__(self):
        if self.emails is None:
            self.emails = []

# Auto-generates __init__, __repr__, __eq__, etc.
person = Person("Alice", 25, ["alice@email.com"])
print(person)  # Person(name='Alice', age=25, emails=['alice@email.com'])
```

## Summary

- Classes use `self` to maintain instance state
- Inheritance lets you build hierarchies; MRO handles lookup logic
- Magic Methods allow your objects to behave like native Python types
- Properties encapsulate data while keeping a clean, attribute-like API
- Abstract classes enforce interfaces and prevent instantiation
- Composition often preferred over inheritance for flexibility
- Design patterns solve common OOP problems elegantly

## Important Keywords

### **Class**

Blueprint for creating objects, defining attributes and methods that instances will inherit.

### **Object**

Instance of a class, containing its own data and behavior defined by the class blueprint.

### **Instance**

Specific occurrence of an object created from a class, with its own unique state.

### **self**

Reference to the current instance within instance methods, allowing access to instance attributes.

### **Inheritance**

Mechanism where a child class acquires attributes and methods from a parent class.

### **Method Resolution Order (MRO)**

Algorithm Python uses to determine which method to call in multiple inheritance scenarios.

### **super()**

Function that allows access to parent class methods from child class methods.

### **Dunder Methods**

Special methods with double underscores that hook into Python's built-in operations and syntax.

### **Magic Methods**

Alternative term for dunder methods, enabling custom objects to integrate with Python's data model.

### **@property**

Decorator that transforms methods into read-only or computed attributes with getter/setter semantics.

### **Getter**

Method that retrieves attribute values, typically used with property decorators for controlled access.

### **Setter**

Method that modifies attribute values, often including validation logic before assignment.

### **@classmethod**

Decorator that binds method to class rather than instance, receiving class as first argument.

### **@staticmethod**

Decorator that creates method independent of class or instance, essentially a namespaced function.

### **Encapsulation**

OOP principle of bundling data and methods that operate on that data within a single unit.

### **Polymorphism**

Ability of different objects to respond to same method call in appropriate ways for their type.

### **Abstract Base Class**

Class designed to be inherited from but not instantiated directly, defining interface requirements.

### **@abstractmethod**

Decorator marking methods that must be implemented by concrete subclasses.

### **Composition**

OOP principle of building complex objects by combining simpler objects, favoring "has-a" over "is-a" relationships.

### **Singleton**

Design pattern ensuring only one instance of a class can ever exist during program execution.

### **Observer Pattern**

Behavioral design pattern where objects (observers) are notified of state changes in another object (subject).

### **Data Class**

Special class type that automatically generates common methods like `__init__`, `__repr__`, and `__eq__`.

### **Name Mangling**

Python's mechanism for making "private" attributes harder to access from outside the class.

### **Interface**

Contract defining methods that implementing classes must provide, though Python uses duck typing instead.

### **Duck Typing**

Python's philosophy that object suitability is determined by available methods and properties, not explicit type.

### **Multiple Inheritance**

Feature allowing a class to inherit from multiple parent classes simultaneously.

### **Method Overriding**

Child class providing its own implementation of a method inherited from a parent class.

### **Method Overloading**

Providing multiple implementations of the same method with different parameters (limited in Python).

### **Constructor**

Special method (`__init__`) called when creating new instance of a class.

### **Destructor**

Special method (`__del__`) called when object is about to be destroyed by garbage collector.

### **Instance Variable**

Data attribute unique to each object instance, storing individual object state.

### **Class Variable**

Data attribute shared among all instances of a class, defined at class level.

### **Object Identity**

Unique identifier for each object, accessible via `id()` function, distinguishing between different instances.

### **Object Equality**

Custom comparison behavior implemented via `__eq__` method to define when two objects are considered equal.

### **Object Representation**

String representation of objects defined by `__str__` (user-friendly) and `__repr__` (developer-friendly).

### **Attribute Access**

Mechanisms for controlling access to object attributes, including properties and descriptors.

### **Protocol**

Informal interface defined by set of methods that objects must implement to be considered compatible.

### **Mixin**

Class providing functionality that can be added to other classes through multiple inheritance.

### **Factory Method**

Pattern where objects are created without specifying exact class, often using class methods.

### **Decorator Pattern**

Structural pattern that adds new functionality to objects dynamically without altering their structure.

### **Strategy Pattern**

Behavioral pattern that encapsulates algorithms, making them interchangeable within client code.

### **Object Lifecycle**

Complete sequence of object creation, usage, and destruction phases in Python programs.

### **Memory Management**

Process of allocating and deallocating memory for objects, handled automatically by Python's garbage collector.

### **Reference Counting**

Python's memory management technique tracking how many references exist to determine when to deallocate objects.

### **Garbage Collection**

Automatic process of reclaiming memory from objects that are no longer referenced.

### **Weak References**

References that don't prevent garbage collection, useful for caching and circular references.

### **Metaclass**

Class that creates classes, allowing customization of class creation process and behavior.

### **Descriptor Protocol**

Interface defining how attribute access is customized through `__get__`, `__set__`, and `__delete__` methods.

### **Attribute Validation**

Process of ensuring attribute values meet certain criteria before assignment, often using property setters.

### **Type Hints**

Syntax for specifying expected types of attributes, method parameters, and return values.

### **Protocol Implementation**

Concrete class providing all required methods for a particular protocol, enabling polymorphic behavior.
