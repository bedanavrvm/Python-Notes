# Introduction to C++

C++ is a high-performance, general-purpose programming language created by Bjarne Stroustrup as an extension of the C programming language.

## Key Features

- **Performance**: Direct manipulation of hardware and memory.
- **Multi-paradigm**: Supports procedural, object-oriented, and generic programming.
- **Standard Library (STL)**: Rich set of algorithms and data structures.

## Compilation Process

Unlike interpreted languages (Python, JS), C++ is compiled.

1.  **Preprocessing**: Handles `#include`, `#define`.
2.  **Compilation**: Translates C++ to Assembly.
3.  **Assembly**: Translates Assembly to Machine Code (Object files `.o` / `.obj`).
4.  **Linking**: Combines object files and libraries into an executable.

## Hello World

Syntax typically involves including the Input/Output stream library and using the `std::cout` object.

```cpp
#include <iostream>

int main() {
    std::cout << "Hello, World!" << std::endl;
    return 0;
}
```

### Key Differences from Python/JS

- **Static Typing**: Types are checked at compile time.
- **Manual Memory Management**: You have control (and responsibility) over memory.
- **Entry Point**: Execution always starts in the `main()` function.
