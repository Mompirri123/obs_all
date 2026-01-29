---
aliases:
  - AI
  - Machine Intelligence
---
**Created:** 06.12.25, 11:57

**Status:** #atomic

**Hashtags:**

- #CPP  
- #genericprogramming
- #datatypes


**Links / Tags:**

- **Relevance Links:**
    
    - Generic Types in C++ <!-- parent, plain text on purpose -->
    - C++ Data Types <!-- parent, plain text on purpose -->
    - Templates in C++ <!-- template mechanism, plain text -->
        
    - [[std tuple in C++]]
    
- **Topic Tags:**
    
    - #stdpair
    - #twovalues
    - #simpletuple

# std::pair in C++

> Part of generic types in C++.

- `std::pair<T, U>` stores **two related values**, possibly with different types.
	
- It is a small, generic struct with:
    
    - `first` – first value
    - `second` – second value

## Basic Usage

```cpp
#include <utility>
#include <string>
#include <iostream>
using namespace std;

int main()
{
    pair<string, int> person("John", 25);

    cout << person.first  << endl;   // "John"
    cout << person.second << endl;   // 25
}
```

## Returning multiple values

- You can use `std::pair` as a simple return type:

```cpp
pair<bool, int> findValue()
{
    // found?  value
    return {true, 42};
}

int main()
{
    auto result = findValue();
    
    if (result.first)
    {
        cout << "Value: " << result.second << endl;
    }
}
```

### Helpers: make_pair and auto

```cpp
auto p = make_pair(string("pi"), 3.14);
// p is pair<string, double>

cout << p.first  << endl;   // "pi"
cout << p.second << endl;   // 3.14
```

- Using `auto` and `make_pair` avoids repeating long template parameters.

## When to use std::pair

- When you need to pass or return **two small related values**:
	- name + age
    - success flag + result
    - key + value
    
- When a **full struct or class** would be overkill.
	
- If you have more than two values or the meaning is unclear, consider:
	
	- a proper `struct` with named fields, or
	- [[std tuple in C++]] for more elements.

## Mental model

- Think of `std::pair<T, U>` as a tiny **2-slot container**:
    
    - slot 1 → `first` of type `T`
    - slot 2 → `second` of type `U`

# References

https://en.cppreference.com/w/cpp/utility/pair.html
