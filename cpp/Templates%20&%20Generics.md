# Templates & Generics

Templates allow you to write generic code that works with any data type.

## Function Templates

Start with `template <typename T>`. The compiler generates a specific version of the function for each type used.

```cpp
template <typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    cout << add(5, 3);       // instantiated as add<int>
    cout << add(2.5, 3.1);   // instantiated as add<double>
    return 0;
}
```

## Class Templates

Useful for generic containers (like `std::vector` itself).

```cpp
template <typename T>
class Box {
private:
    T value;
public:
    Box(T v) : value(v) {}
    T getValue() { return value; }
};

Box<int> intBox(10);
Box<string> strBox("Hello");
```

## Template Specialization

You can define a specific implementation for a particular type.

```cpp
template <>
class Box<bool> {
    // specialized implementation for bool
};
```
