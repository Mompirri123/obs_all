---
aliases:
  - std::visit
  - std-visit
  - std visit
  - std  visit
---
**Created:** 06.12.25, 11:08

**Status:** #atomic

**Hashtags:**

- #cpp
- #genericprogramming
- #datatypes

**Links / Tags:**

- **Relevance Links:**
    
    - Generic Types in C++ <!-- parent, plain text on purpose -->
    - std variant in C++ <!-- parent, plain text on purpose -->
    - Templates in C++ <!-- template mechanism, plain text -->
        
    - [[Template Mental Models and Use Cases in C++]]
    
- **Topic Tags:**
    
    - #stdvisit
    - #visitorpattern
    - #patternmatching

# std::visit in C++

- `std::visit` is the **standard way to read a [[std variant in C++|std::variant]]**.  
- It calls a callable (visitor) with the **currently active type** inside the variant.

## Basic idea

- You have a `std::variant<A, B, C>`.
- You write a visitor with `operator()(A)`, `operator()(B)`, `operator()(C)`.
- `std::visit(visitor, myVariant);` calls the correct overload.

### Example with visitor struct
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

## Key points

- Printer has one operator() overload per type in the variant.
- std::visit chooses the correct overload based on the active type at runtime.
- The compiler checks that all cases are handled correctly.
- Checking and using the result
- std::visit returns whatever your visitor returns.

## Checking and using the result

- `std::visit` returns whatever your visitor returns.

```cpp
struct ToString
{
	string operator()(int value) const
	{
		return "int: " + to_string(value);
	}
	string operator()(double value) const
	{
		return "double: " + to_string(value);
	}
	string operator()(const string& value) const
	{
		return "string: " + value;
	}
};

int main()
{
	NumberOrText v = 3.14;
	string s = visit(ToString{}, v);
	cout << s << '\n';     // "double: 3.140000"
}
```

## When to use std::visit

- Whenever you need to **do something** with the value inside a `std::variant`
    
- When you want:
    - one central place that handles all types
    - the compiler to complain if you forget a case
    
- It gives you a **pattern-matching style** over variant types, without using inheritance or virtual functions.

# References

- 
