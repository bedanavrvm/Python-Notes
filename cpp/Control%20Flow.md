# Control Flow

## Conditionals

### If / Else

Standard if/else logic applies.

```cpp
if (score > 90) {
    cout << "A";
} else if (score > 80) {
    cout << "B";
} else {
    cout << "C";
}
```

### Switch Statement

Switch statements work on integral types (int, char, enum).

```cpp
switch (grade) {
    case 'A':
        cout << "Excellent";
        break;
    case 'B':
        cout << "Good";
        break;
    default:
        cout << "Unknown";
}
```

> [!NOTE] C++17 Attribute
> Use `[[fallthrough]]` to explicitly indicate intentional fallthrough to suppress compiler warnings.

## Loops

### Standard For Loop

```cpp
for (int i = 0; i < 5; ++i) {
    cout << i << " ";
}
```

### While Loop

```cpp
while (condition) {
    // ...
}
```

### Range-based For Loop (C++11)

Preferred for iterating over containers (arrays, vectors).

Why/when:
- Safer (bounds checking handled by container iterators implicitly).
- Cleaner syntax.

```cpp
std::vector<int> numbers = {1, 2, 3, 4, 5};

// Deduce type with auto
for (auto num : numbers) {
    cout << num << " ";
}

// Modify elements by reference
for (auto& num : numbers) {
    num *= 2;
}

// Read-only access by const reference (Efficient)
for (const auto& num : numbers) {
    cout << num << " ";
}
```
