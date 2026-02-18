# Memory Management

Understanding memory is the key to C++.

## Stack vs Heap

- **Stack**: Automatic memory. Fast, limited size. Function variables live here.
- **Heap**: Dynamic memory. Managed manually (or via smart pointers). Slower, large size.

## Manual Management (`new` / `delete`)

Use `new` to allocate on the heap and returns a pointer. You MUST call `delete` to free it.

```cpp
int* p = new int(10); // Allocate int on heap
// ... use p ...
delete p; // Free memory. CRITICAL!
p = nullptr; // Safety practice
```

**Memory Leak**: Forgetting to `delete` causes a leak.
**Double Free**: Calling `delete` twice crashes the program.

## Smart Pointers (C++11)

**Modern C++ strongly discourages raw `new`/`delete`.** Use Smart Pointers (`<memory>`) instead. They automatically delete memory when they go out of scope (RAII).

### `std::unique_ptr`

Sole ownership. Cannot be copied, only moved. Efficient and safe.

```cpp
#include <memory>

std::unique_ptr<int> p = std::make_unique<int>(10);
// No delete needed! Automatically freed when p goes out of scope.
```

### `std::shared_ptr`

Shared ownership via reference counting. Deleted when the last owner is gone.

```cpp
std::shared_ptr<int> p1 = std::make_shared<int>(20);
std::shared_ptr<int> p2 = p1; // Both point to same memory
```

## RAII (Resource Acquisition Is Initialization)

This is the core C++ idiom. Resources (memory, file handles, locks) are acquired in a constructor and released in a destructor.

Why/when:
- Prevents leaks.
- Exception-safe (destructors run during stack unwinding).
