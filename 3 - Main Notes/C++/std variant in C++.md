---
aliases:
  - std::variant in C++
  - std::variant
  - std-variant
  - std variant
---
**Created:** 06.12.25, 05:57
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
        
    - [[Template Mental Models and Use Cases in C++]]
        
- **Topic Tags:**
    - #stdvariant
    - #sumtypes
    - #eitheror

---
# std::variant in C++

> Part of generic types in C++.

- `std::variant<T...>` represents a value that can be **one of several types**.
- It is a **type safe union**
	- with the compiler keeping track of which alternative is active.

## Basic Idea

- We declare a variant with several possible types
	- If the variant will hold exactly one of those types at all points

```cpp
#include <variant>
#include <string>

using namespace std;

using NumberOrText = variant<int, double, string>
```

- At any time, the variant holds exactly one of these:
	
	- `int`
	- `double`
	- `string`

### Example with [[std visit in C++|std::visit]]

```cpp
#include <variant>
#include <string>
#include <iostream>
using namespace std;

using NumberOrText = variant<int, double, string>;

struct Printer
{
	void operator()(int value) const
	{
		cout << "int: " << value << '\n';
	}
	
	void operator()(double value) const
	{
		cout << "double: " << value << '\n';
	}
	
	void operator()(const string& value) const
	{
		cout << "string: " << value << '\n';
	}
};

int main()
{
	NumberOrText v = 10;
	visit(Printer{}, v);   // int: 10
	
	v = 3.14;
	visit(Printer{}, v);   // double: 3.14
	
	v = string("hello");
	visit(Printer{}, v);   // string: hello
}
```

## Accessing the active value

Common helpers:

- `holds_alternative<T>(v)`
    - check if the active value is of type `T`
    
- `get<T>(v)`
    - get the value as `T` (throws if wrong type)
    
- `get_if<T>(&v)`
    - returns pointer to `T` if active, otherwise `nullptr`

```cpp
NumberOrText v = 3.14;

if (holds_alternative<double>(v))
{
	double x = get<double>(v);
	cout << "double value: " << x << '\n';
}
```

## When std::variant is useful

- When something can be **one of a small set** of types:
    - result can be `int` or `error code` type
    - configuration option can be `bool`, `int`, or `string`
    
- When you want to **avoid inheritance** and virtual functions and still model alternatives.
    
- When you want the compiler to **force you to handle all cases** via `std::visit`.
