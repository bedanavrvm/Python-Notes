# C++ Best Practices

Writing "Modern C++" means unlearning "C with Classes".

## Resource Management (RAII)

**Rule #1**: Never manage resources manually.
- Use `std::unique_ptr` and `std::shared_ptr`.
- Use `std::vector` instead of raw arrays (`int arr[10]`).
- Use `std::string` instead of `char*`.

## The Rule of Zero

Classes should not manage resources directly if possible. Rely on members (like `std::string` or `std::vector`) that already handle cleanup.

If you *must* manage a resource, follow the **Rule of Five**:
1. Destructor
2. Copy Constructor
3. Copy Assignment Operator
4. Move Constructor
5. Move Assignment Operator

## Const Correctness

Make everything `const` unless it needs to change.
- `const` parameters.
- `const` methods (`int getValue() const;`).
- `const` local variables.

## Type Inference

- Use `auto` to avoid verbosity, but don't overuse it if it obscures types.
- **Always** use `auto` for iterators and lambdas.

## Algorithms over Loops

Prefer `<algorithm>` over raw loops.

```cpp
// Bad
for (int i = 0; i < v.size(); i++) {
    if (v[i] == 5) return true;
}

// Good
if (std::find(v.begin(), v.end(), 5) != v.end()) return true;
```

## Compilation Speed

- Use forward declarations in headers where possible to reduce dependency chains.
- Use `#pragma once` or include guards.
