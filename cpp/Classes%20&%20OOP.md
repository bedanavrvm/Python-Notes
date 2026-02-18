# Classes & OOP

## Basic Class Syntax

Classes are private by default, Structs are public by default.

```cpp
class Person {
private:
    std::string name;
    int age;

public:
    // Constructor
    Person(string n, int a) : name(n), age(a) {
        // Initialization list usage is preferred
    }

    // Destructor (Optional, mostly for manual cleanup)
    ~Person() {
        cout << "Destroying Person";
    }

    void greet() {
        cout << "Hello, I'm " << name;
    }
};

int main() {
    Person p("Alice", 30); // Stack allocation
    p.greet();
    return 0;
}
```

## Inheritance

```cpp
class Student : public Person { // Public inheritance
public:
    int studentID;

    Student(string n, int a, int id) : Person(n, a), studentID(id) {}
};
```

## Polymorphism & Virtual Functions

To enable polymorphism (overriding functions), declare the base function as `virtual`.

**Critical**: If a class has virtual functions, it MUST have a **virtual destructor**.

```cpp
class Shape {
public:
    virtual void draw() { cout << "Generic Shape"; }
    virtual ~Shape() {} // Essential for proper cleanup!
};

class Circle : public Shape {
public:
    void draw() override { cout << "Circle"; } // 'override' ensures safety
};

void render(Shape* s) {
    s->draw(); // Calls correct function at runtime (Dynamic Dispatch)
}
```
