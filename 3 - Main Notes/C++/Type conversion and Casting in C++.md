**Created:** <span class ="color-green">24.12.25, 13:41</span>

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #typesystem
	- #casting
	- #map
- **Topic Tags:**
	- #typeconversion
	- #explicitcast
	- #implicitcast
	  
**Links / Tags:** 
- **Relevance Links:**
	- C++ Type System
- **Topic Links:**
	- [[Implicit Type Conversions in C++]]
	- [[Explicit Type Casting in C++]]
	- [[C-Style Casts in C++]]
	- [[User-Defined Conversions in C++]]
	- [[Choosing the appropriate Cast in C++]]	
---

> how C++ converts values from one type to another
> **hub** for all type-conversion mechanisms in C++.

# Type Conversion and Casting in C++

- Type casting is the process of converting a value from one data type to another
- In C++ there are 4 main methods of type casting:

## 1. Implicit type conversions
- [[Implicit Type Conversions in C++]]
	- automatic conversions performed by the compiler
	- numeric promotions
	- pointer/reference adjustments
	- conversions during expressions and function calls
	
## 2. Explicit type casting
- [[Explicit Type Casting in C++]]
	- programmer-requested conversions
	- safer and more expressive than C-style casts
	
- ### C++ cast operators
	- `static_cast`
	- `const_cast`
	- `reinterpret_cast`
	- `dynamic_cast`
	  
## 3. C-style casts
- [[C-Style Casts in C++]]
	- legacy syntax: `(type)expression`
	- mixes multiple cast meanings
	- discouraged in modern C++
	  
## 4. User-defined conversions
- [[User-Defined Conversions in C++]]
	- conversion constructors
	- conversion operators (`operator T()`)
	- interaction with overload resolution


# What type of casting to choose

## Choosing the appropriate cast
- [[Choosing the appropriate Cast in C++]]
	- decision rules
	- safety vs intent
	- compile-time vs runtime checks
	
# Internal References
- CV-Qualifiers in C++
- Constructors in C++

# External References
