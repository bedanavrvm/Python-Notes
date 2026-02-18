# Basics & Fundamentals

C++ is a compiled, statically typed language. This means you must explicitly declare the type of each variable before you use it (with some exceptions like `auto`), and the compiler checks for type errors before the program runs.

In this chapter, we cover the fundamental building blocks of C++: variables, types, initialization, and basic I/O.

## Comments

C++ supports both single-line and multi-line comments.

```cpp
// This is a single-line comment. Everything after the // is ignored.

/*
 * This is a multi-line comment.
 * Good for longer explanations or temporarily disabling blocks of code.
 */
```

> **Best Practice**: Use `//` for single-line comments in code. Save `/* ... */` for file headers or temporarily commenting out blocks of code during debugging, as nested multi-line comments are not supported.

## Variables & Initialization

A variable is a named storage location in memory. In C++, variables have a **storage duration**, **scope**, and **linkage**.

### Initialization Styles

C++ has evolved several ways to initialize variables. It is crucial to understand the differences.

#### 1. C-Style Assignment (Copy Initialization)
Traditional, but can be ambiguous with objects.

```cpp
int w = 5;       // Copy initialization
std::string s = "Hello";
```

#### 2. Constructor Initialization (Direct Initialization)
Uses parentheses.

```cpp
int x(10);       // Direct initialization
std::string s("Hello");
```

> **Warning (The "Most Vexing Parse")**: `int x();` is NOT a variable declaration; it is a function declaration returning an int. This ambiguity is why Uniform Initialization was introduced.

#### 3. Uniform Initialization (List Initialization) - Modern & Recommended
Uses curly braces `{}`. Introduced in C++11 to provide a consistent syntax for everything.

**Key Benefit**: Prevents "narrowing conversions". The compiler will throw an error if data would be lost (e.g., float to int).

```cpp
int y{15};       // Valid
int z = {20};    // Also valid

// Narrowing check:
// int n{3.14};  // ERROR: Narrowing conversion from double to int
int n(3.14);     // Allowed (n becomes 3), but dangerous
```

#### 4. Zero Initialization
Empty braces initialize variables to their default "zero" value.

```cpp
int i{};         // i is 0
float f{};       // f is 0.0
int* p{};        // p is nullptr
```

## Primitive Data Types

C++ provides a rich set of built-in types. The C++ standard guarantees *minimum* sizes, but actual sizes depend on the compiler and architecture (e.g., 32-bit vs 64-bit).

### 1. Integer Types

Whole numbers. Can be `signed` (default) or `unsigned` (positive only).

| Type | Min Size | Typical (64-bit) | Range (Signed) | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| `char` | 1 byte | 1 byte | -128 to 127 | ASCII characters, bytes |
| `short` | 2 bytes | 2 bytes | -32,768 to 32,767 | Small counts (rarely used) |
| `int` | 2 bytes | 4 bytes | +/- 2 billion | **Default integer type** |
| `long` | 4 bytes | 4 or 8 bytes | Varies (use long long for 64-bit) | System-dependent size |
| `long long` | 8 bytes | 8 bytes | +/- 9 quintillion | Large counts, 64-bit IDs |

#### Signed vs Unsigned
- **signed**: Can hold positive and negative numbers.
- **unsigned**: Can ONLY hold non-negative numbers. It trades the negative range to double the positive range.

> **Pitfall**: Be careful mixing signed and unsigned integers in arithmetic or comparisons. The signed value may be implicitly promoted to unsigned, causing extremely large values for negatives (underflow).

```cpp
int s = -1;
unsigned int u = 1;
if (s < u) {
    // This might NOT execute!
    // -1 cast to unsigned becomes MAX_UINT, which is > 1.
}
```

### 2. Fixed-Width Integers (`<cstdint>`)

For precise control over size (critical for networking, binary file formats, and cross-platform compatibility), include `<cstdint>`.

```cpp
#include <cstdint>

int8_t  tiny = 120;     // Exactly 1 byte
int16_t small = 30000;  // Exactly 2 bytes
int32_t normal = 10000; // Exactly 4 bytes
int64_t huge = 5000LL;  // Exactly 8 bytes

uint64_t id = 0xFFFFFFFFFFFFFFFF; // Unsigned 64-bit
```

### 3. Floating Point Types

Numbers with fractional parts, following IEEE 754 standard (usually).

| Type | Min Size | Precision | Usage |
| :--- | :--- | :--- | :--- |
| `float` | 4 bytes | ~7 digits | Graphics (GPU), low-memory envs |
| `double` | 8 bytes | ~15 digits | **Default for math** |
| `long double` | 8 bytes | ~18-30 digits | High-precision scientific calc |

```cpp
float f = 3.14f;      // 'f' suffix is required!
double d = 3.14;      // Default is double
long double ld = 3.14L; // 'L' suffix
```

> **Note**: `3.14` is a `double`. `3.14f` is a `float`. Initializing a float with a double literal (`float f = 3.14`) causes a conversion, potentially losing precision.

### 4. Boolean

The `bool` type stores truth values.
  - `true` (promotes to 1)
  - `false` (promotes to 0)

```cpp
bool isReady = true;
std::cout << isReady; // Prints "1" by default
std::cout << std::boolalpha << isReady; // Prints "true"
```

### 5. Character Types

- `char`: 1 byte. ASCII.
- `wchar_t`: Wide character (size varies, 2 or 4 bytes).
- `char8_t` (C++20): UTF-8 character.
- `char16_t` (C++11): UTF-16 character.
- `char32_t` (C++11): UTF-32 character.

```cpp
char letter = 'A';    // Single quotes for method char literals
```

## Type Inference (`auto`)

Since C++11, the `auto` keyword lets the compiler deduce the type from the initializer.

Why/When:
- **Good**: For complex types (iterators, lambdas) or when the type is obvious (`auto x = 5;`).
- **Bad**: When it obscures the API (`auto result = calculate();` - what is result?).
- **Rule**: `auto` variables *must* be initialized.

```cpp
auto a = 10;          // int
auto b = 3.14;        // double
auto c = "hello";     // const char*
auto d = std::string("hello"); // std::string

// Const/Reference with auto rules
const auto& ref = a;  // const int&
```

## `decltype` (Advanced)

Sometimes you want the type of an expression *without evaluating it*.

```cpp
int x = 0;
decltype(x) y = 5; // y is type int because x is int
```

## Literals & Suffixes

Suffixes tell the compiler exactly what type a literal is.

```cpp
auto a = 10;    // int
auto b = 10u;   // unsigned int
auto c = 10L;   // long
auto d = 10LL;  // long long
auto e = 10uLL; // unsigned long long

auto f = 3.14;  // double
auto g = 3.14f; // float
auto h = 3.14L; // long double
```

## Constants

### `const` (Runtime Constant)
Promises that a variable will not be modified after initialization. The compiler enforces this.

```cpp
const int limits = 100;
// limits = 101; // Compilation Error
```

### `constexpr` (Compile-Time Constant) - C++11
A stronger guarantee. The value is computed *during compilation*. This allows the value to be used in places where C++ requires a constant expression (like array sizes or template arguments).

```cpp
constexpr double gravity = 9.81;
constexpr int square(int x) { return x * x; }

int arr[square(5)]; // Valid: square(5) is 25 at compile time!
```

## Type Conversion (Casting)

### Implicit Conversion (Coercion)
Happens automatically.
- **Promotion**: Generally safe (e.g., `float` -> `double`, `short` -> `int`).
- **Demotion**: Risky (e.g., `double` -> `int`), loses data.

### Explicit Casting

**Avoid C-style casts** `(type)val`. They are hard to read and too powerful (unsafe). Use C++ named casts:

#### 1. `static_cast`
The standard cast for value conversions. It performs checks at compile time.

```cpp
double pi = 3.14159;
int num = static_cast<int>(pi); // Explicitly truncate
```

#### 2. `reinterpret_cast`
DANGEROUS. Reinterprets the raw bits of memory. Used for low-level systems programming.

```cpp
int* p = new int(65);
char* ch = reinterpret_cast<char*>(p);
```

#### 3. `const_cast`
Removes the `const` qualifier. Rarely used; suggests design flaw if needed often.

## Input / Output (I/O)

C++ uses streams from `<iostream>` for I/O.
- `std::cout`: Standard Output (buffered)
- `std::cin`: Standard Input
- `std::cerr`: Standard Error (unbuffered, appears immediately)
- `std::clog`: Standard Logging (buffered)

### Formatting Output (`<iomanip>`)

Include `<iomanip>` to formatting manipulators.

```cpp
#include <iostream>
#include <iomanip>

int main() {
    double pi = 3.1415926535;

    // Set precision
    std::cout << std::fixed << std::setprecision(2) << pi << std::endl; // 3.14

    // Set width (padding)
    std::cout << std::setw(10) << "Right" << std::endl;
    std::cout << std::left << std::setw(10) << "Left" << std::endl;

    // Hexadecimal output
    int check = 255;
    std::cout << std::hex << check << std::endl; // ff
    std::cout << std::dec << check << std::endl; // 255
    
    return 0;
}
```

## Scope & Lifetime

Variables have a specific "lifetime" and visibility.

1. **Local Scope (Automatic Storage)**: Defined inside a block `{}`. Created when execution enters the block, destroyed when it leaves.
2. **Global Scope (Static Storage)**: Defined outside functions. Created at program start, destroyed at program end.
3. **Static Local**: Defined inside a function with `static`. Initialized ONCE (first call), retains value across calls.

```cpp
void counter() {
    static int count = 0; // Initialized only once
    count++;
    std::cout << count << " ";
}

int main() {
    counter(); // 1
    counter(); // 2
    counter(); // 3
}
```

## Important Keywords

### `sizeof`
Returns the size (in bytes) of a type or variable.
`std::cout << sizeof(int); // 4`

### `typedef` / `using` (Type Aliasing)
Create a new name for an existing type. `using` is the modern, preferred syntax.

```cpp
// Old C-style
typedef unsigned long long u64;

// Modern C++
using u64 = unsigned long long;

u64 id = 123456789;
```

### `volatile`
Tells the compiler NOT to optimize memory access to this variable. Used in embedded systems (hardware registers) or multi-threading (though `std::atomic` is preferred there).

## Summary

- C++ variables must be declared with a **type**, though `auto` can deduce it.
- Use **Uniform Initialization** (`int x{5}`) to prevent narrowing errors.
- **Fixed-width integers** (`int32_t`) are safer for cross-platform code than raw `int`/`long`.
- Understand the difference between **signed** and **unsigned** to avoid arithmetic bugs.
- Prefer **C++ style casts** (`static_cast`) over C style casts `(int)`.
- Use `const` by default, and `constexpr` for values known at compile time.
