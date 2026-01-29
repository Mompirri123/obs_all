---
aliases:
  - std::tuple
  - std-tuple
  - std tuple
---
**Created:** 06.12.25, 12:43

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
        
    - [[std pair in C++]]
    
- **Topic Tags:**
    
    - #stdtuple 
    - #heterogeneousbundle 
    - #fixedsizegroup

---

# std::tuple in C++

> Part of generic types in C++.

`std::tuple<T...>` is a **fixed-size bundle** of values, where each element can have a different type.

- generalises `std::pair<T, U>` to more than two elements
- keeps all values together as one object

## Basic usage

```cpp
#include <tuple>
#include <string>
#include <iostream>
using namespace std;

int main()
{
    tuple<string, int, double> data("Alice", 3, 4.5);

    string name;
    int count;
    double value;

    tie(name, count, value) = data;

    cout << name  << endl;  // "Alice"
    cout << count << endl;  // 3
    cout << value << endl;  // 4.5
}

```

## Access by index

- You can access elements by compile-time index using `std::get`

```cpp
#include <tuple>
#include <string>
#include <iostream>
using namespace std;

int main()
{
    auto t = make_tuple(string("Bob"), 7, 2.5)
    
    cout << get<0>(t) << endl;  // "Bob"
    cout << get<1>(t) << endl;  // 7
    cout << get<2>(t) << endl;  // 2.5
}
```

- index must be known at compile time (`get<0>`, not `get<i>` with a variable)
- type and index are tied together in the type system

## When to use std::tuple

- Good use cases:
	
	- returning several values from a function
	    - e.g. parsed result + error count + status
	- grouping a few related values **only used together**
	- quick prototyping where creating a full struct is too heavy
	  
- Be careful when:
	- there are many elements and it becomes unclear what each position means  
	    → a named `struct` is usually better
    
	- the same type appears multiple times  
	    → it’s easy to mix up positions
    

## std::pair vs std::tuple

- use [[std pair in C++]] when you just need **two values**
- use `std::tuple` when you need **three or more** and they form one logical bundle

## Mental model

- think of `std::tuple<T1, T2, T3>` as a small **fixed record**:
    
    - slot 0: `T1`
	- slot 1: `T2`    
    - slot 2: `T3`
    
- you trade **names** for compactness; good in small, local contexts where the meaning is obvious

# References

