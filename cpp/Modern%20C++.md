# Modern C++ (C++11/14/17/20)

## Lambda Expressions

Anonymous functions.

`[capture](parameters) -> return_type { body }`

- `[]`: No capture
- `[=]`: Capture all by value
- `[&]`: Capture all by reference

```cpp
auto add = [](int a, int b) { return a + b; };
cout << add(2, 3); // 5

int x = 10;
auto addX = [x](int a) { return a + x; }; // Captures x by value
```

## Move Semantics & R-value References

Optimizes performance by avoiding deep copies of temporary objects.

- **L-value**: Has a name and address (e.g., `x`).
- **R-value**: Temporary, no name (e.g., `5`, `x + y`).
- **`std::move`**: Casts an L-value to an R-value, allowing resources to be "stolen" (moved).

```cpp
std::string a = "Hello";
std::string b = std::move(a); // a is now empty/valid state, b owns the data
```

## `constexpr`

Computes values at **compile time**.

```cpp
constexpr int factorial(int n) {
    return (n <= 1) ? 1 : (n * factorial(n - 1));
}
// factorial(5) is computed by the compiler, not at runtime.
```

## Structured Binding (C++17)

Unpack tuples or structs easily (like Python/JS destructuring).

```cpp
std::pair<int, int> p = {1, 2};
auto [x, y] = p;
```
