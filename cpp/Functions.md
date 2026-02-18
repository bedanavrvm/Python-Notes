# Functions

## Declaration vs Definition

In C++, functions must be declared before they are used.

- **Declaration (Header)**: Tells the compiler the function exists (name, return type, params).
- **Definition (Body)**: The actual code implementation.

```cpp
// Declaration
int add(int a, int b);

int main() {
    int result = add(5, 3);
    return 0;
}

// Definition
int add(int a, int b) {
    return a + b;
}
```

## Parameter Passing

### Pass by Value

Default behavior. A copy of the argument is made. Modifications inside the function do not affect the original variable.

```cpp
void increment(int x) {
    x++; // Only modifies local copy
}
```

### Pass by Reference

Using `&`, the function receives a reference to the original variable. No copy is made.

Why/when:
- Use when you need to modify the argument.
- Use `const` reference for large objects to avoid copying overhead (see below).

```cpp
void increment(int& x) {
    x++; // Modifies original variable
}
```

### Pass by Const Reference

Standard practice for passing large objects (like `std::string`, `std::vector`, classes) that should *not* be modified.

```cpp
void printMessage(const std::string& msg) {
    // msg is read-only reference
    // msg = "New"; // Error
    cout << msg;
}
```

## Overloading

You can define multiple functions with the same name but different parameter lists.

```cpp
int add(int a, int b) { return a + b; }
double add(double a, double b) { return a + b; }
```

## Default Arguments

You can provide default values for trailing parameters.

```cpp
void log(std::string msg, int level = 1) {
    cout << "[" << level << "] " << msg;
}

log("Error");    // level = 1
log("Fatal", 0); // level = 0
```
