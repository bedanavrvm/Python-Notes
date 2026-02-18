# Pointers & References

This is often the most challenging part of C++ coming from Python/JS.

## Pointers

A pointer is a variable that stores the **memory address** of another variable.

- `&`: Address-of operator (Get address)
- `*`: Dereference operator (Get value at address)

```cpp
int x = 10;
int* ptr = &x; // ptr stores address of x

cout << ptr;  // Prints memory address (e.g., 0x7ffd...)
cout << *ptr; // Prints value (10)

*ptr = 20;    // Changes x to 20
```

### Null Pointers

Always initialize pointers. If a pointer doesn't point to anything valid, use `nullptr` (C++11). Avoid `NULL`.

```cpp
int* ptr = nullptr;
```

## References

A reference is an **alias** for an existing variable. It must be initialized when declared and cannot be reseated (made to refer to a different variable).

```cpp
int x = 10;
int& ref = x; // ref is an alias for x

ref = 20;     // x becomes 20
```

### Pointers vs References

| Feature | Pointer (`*`) | Reference (`&`) |
| :--- | :--- | :--- |
| Can be `nullptr`? | Yes | No |
| Can be reseated? | Yes | No |
| Syntax | Explicit dereferencing (`*ptr`) | Implicit usage |
| Arithmetic? | Yes (pointer arithmetic) | No |

## Const Correctness

`const` can change meaning depending on placement relative to `*`.

- `const int* p`: Pointer to **constant data** (Data cannot change, pointer can move).
- `int* const p`: **Constant pointer** to data (Pointer cannot move, data can change).
- `const int* const p`: Constant pointer to constant data (Neither can change).

Rule of thumb: Read from right to left.

```cpp
int x = 10;
const int* p1 = &x; // "p1 is a pointer to an int that is const"
// *p1 = 20; // Error
p1++; // OK

int* const p2 = &x; // "p2 is a const pointer to an int"
*p2 = 20; // OK
// p2++; // Error
```
