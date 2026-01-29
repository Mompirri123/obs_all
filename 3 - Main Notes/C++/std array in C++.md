**Created:** 06.12.25, 13:55

**Status:** #atomic

**Hashtags:**

- #cpp
- #genericprogramming
- #datatypes

**Links / Tags:**

- **Relevance Links:**
    
    - Generic Types in C++ <!-- parent, plain text on purpose -->
    - C++ Data Types <!-- parent, plain text on purpose -->
    - Templates in C++ <!-- template mechanism, plain text -->
        
    - [[C++ STL]]
    
- **Topic Tags:**
    
    - #stdarray
    - #fixedsizearray
    - #stl

---
# std::array in C++

> Part of generic types in C++.

`std::array<T, N>` is a **fixed-size array** that behaves like a C array but has a nice STL-style interface.

- size `N` is known at compile time
    
- elements are stored **contiguously** (like `T[N]`)
    
- you get `.size()`, iterators, and range-for support

---
## Basic usage

```cpp
#include <array>
#include <iostream>
using namespace std;

int main()
{
    array<int, 3> nums = {1, 2, 3};
    
    for (int x : nums)
        cout << x << " ";
        
    cout << "\nsize: " << nums.size() << endl;
}
```

- `array<int, 3>` = array of 3 int numbers
- can be default-initialised, brace-initialised, or filled later

## Accessing elements

```cpp
int main()
{
    array<int, 3> nums = {10, 20, 30};
    
    cout << nums[0] << endl;      // 10 (no bounds check)
    cout << nums.at(1) << endl;   // 20 (throws if out of range)
    
    nums[2] = 99;
    cout << nums[2] << endl;      // 99
}
```

- `operator[]` – fast, no bounds checking
- `at()` – bounds checking, throws `std::out_of_range` on bad index

## Using with algorithms

Because `std::array` has `.begin()` and `.end()`, you can use STL algorithms easily:
```cpp
#include <algorithm>
#include <array>
#include <iostream>
using namespace std;

int main()
{
    array<int, 5> nums = {5, 2, 4, 1, 3};
    
    sort(nums.begin(), nums.end());
    
    for (int x : nums)
        cout << x << " ";     // 1 2 3 4 5
}
```

## When to use std::array

Use `std::array<T, N>` when:

- the size is **fixed at compile time**
    
- you want:
    - contiguous storage
    - STL compatibility
    - safer interface than raw arrays

If the size can change at runtime, prefer `std::vector<T>` instead.

---
## Mental model

- `std::array<T, N>` = “C array with methods”:
    
    - same memory layout as `T[N]`
        
    - plus `.size()`, iterators, compatibility with algorithms

---
# References