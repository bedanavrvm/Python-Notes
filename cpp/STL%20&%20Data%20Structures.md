# STL & Data Structures

The Standard Template Library (STL) provides powerful container classes and algorithms.

## Containers

### Vectors (`std::vector`)

Dynamic array. Fast access, O(1) push_back.

```cpp
#include <vector>

std::vector<int> numbers = {1, 2, 3};
numbers.push_back(4); // Add to end

cout << numbers[0];   // Access via index
cout << numbers.size(); // Length
```

### Maps (`std::map` / `std::unordered_map`)

Key-value pairs.

- `map`: Sorted (Tree-based), O(log n).
- `unordered_map`: Unsorted (Hash-based), O(1) avg.

```cpp
#include <map>

std::map<string, int> ages;
ages["Alice"] = 30;
ages["Bob"] = 25;

if (ages.find("Charlie") == ages.end()) {
    cout << "Not found";
}
```

### Strings (`std::string`)

Mutable string class.

```cpp
string s = "Hello";
s += " World";
cout << s.length();
```

## Algorithms (`<algorithm>`)

Works on ranges defined by iterators.

```cpp
#include <algorithm>
#include <vector>

std::vector<int> v = {5, 2, 8, 1};

// Sort
std::sort(v.begin(), v.end());

// Find
auto it = std::find(v.begin(), v.end(), 8);
if (it != v.end()) {
    cout << "Found 8 at index: " << std::distance(v.begin(), it);
}
```
