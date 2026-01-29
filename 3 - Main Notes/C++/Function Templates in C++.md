**Created:** 03.12.25, 05:24

**Status:** #atomic

**Hashtags:**
- #cpp
- #templates
- #genericprogramming
- #functions
- #atomic

**Links / Tags:** 
- **Relevance Links:**
  - User-Defined Types in C++
  - Templates in C++

- **Topic Tags:**
  - #functiontemplates
  - #typededuction
  - #compiletimepolymorphism

# Function Templates in C++

> Part of templates in C++.

Function templates let you write one function that works with many types. The compiler generates specific instances per type at **compile time**.

## Basic Syntax

```cpp
template <typename T>
T functionName(T parameter)
{
    // Code using type T
}
```

## Practical Example

```cpp
template <typename T> T getMax(T a, T b)
{
	return (a > b) ? a : b;
}

int main()
{
	cout << getMax<int>(10, 20);
	// T = int     cout << getMax<double>(3.14, 2.99);
	// T = double     cout << getMax<string>("apple", "banana"); // works with strings
}```

## Type Deduction

The compiler can deduce the template parameter from arguments:

```cpp
getMax(10, 20);       // T deduced as int
getMax(3.14, 2.99);   // T deduced as double
// T deduced as int getMax(3.14, 2.99);     
// T deduced as double
```

#### Key idea:
- you usually don’t need to write `<int>` explicitly unless deduction fails.
