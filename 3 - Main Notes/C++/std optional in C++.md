---
aliases:
  - std::optional in C++
---
**Created:** 06.12.25, 04:40

**Status:** #atomic

**Hashtags:**
- #cpp
- #genericprogramming
- #datatypes
- #atomic

**Links / Tags:**
- **Relevance Links:**
    - Generic Types in C++        <!-- parent, plain text on purpose -->
    - C++ Data Types              <!-- parent, plain text on purpose -->
    - Templates in C++            <!-- mechanism, plain text -->
    
    - [[Template Mental Models and Use Cases in C++]]
      
- **Topic Tags:**
    - #stdoptional
    - #maybevalue
    - #expressiveapis

# std::optional in C++

> Part of generic types in C++.

- `std::optional<T>` represents **“a value of type T or nothing”**.

## Basic idea

- Avoids magic values like `-1`, `0`, or `nullptr` to mean “not found”.
- Makes it explicit when a result **might not exist**.

## Example

```cpp
#include <optional>
#include <iostream>
using namespace std;

optional<int> findIndex(bool ok)
{
    if (!ok) return nullopt;
    return 42;
}

int main()
{
    optional<int> result = findIndex(true);

    if (result.has_value())
    {
        cout << "Index: " << *result << endl;
    }
    else
    {
        cout << "No index found" << endl;
    }
}
```

### Key points

- `optional<T>` can be:
    
    - “engaged” (contains a T)
    - “disengaged” (empty)
    
- Access:
    
    - `has_value()`
    - `operator*` to get the value
    - `value_or(defaultValue)` to fall back

# References
