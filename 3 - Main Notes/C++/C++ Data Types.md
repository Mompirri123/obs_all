**Created:** 02.12.25, 04:27

**Status:** #map

**Hashtags:**
- #cpp
- #datatypes
- #overview

**Links / Tags:** 
- **Relevance Links:**
	- [[Primitive Integral Types in C++]]
	- [[Floating-Point Types in C++]]
	- [[Character and String Types in C++]]
	- [[Pointer and Reference Types in C++]]
	- [[Fixed-Width Integer Types in C++]]
	- [[User-Defined Types in C++]]
	- [[Special Types in C++]]
	- [[C++ Time & Date Datatypes, Timestamps]]
	- [[Generic Types in C++]]
	  
- **Topic Tags:**
	- #types
	- #numeric
	- #characters
	- #pointers
	- #userdefined
	- #templatetypes
	- #specialtypes
	  
---

# C++ Data Types

> Data types define **what kind of data a variable can hold**
> how much **memory it occupies**, and **what operations are allowed** on it

- Every object in C++ has a **type**
- The type determines:
    - memory layout
    - valid operations
    - copying and conversion rules
    - lifetime behavior
    - Storage duration
    
- C++ provides:
	- a rich set of **built-in data types**, and
	- powerful mechanisms to **define your own data types**
	  

## Built-in vs User-defined types

- **Built-in types** are provided by the language itself  
    (e.g. integers, floating-point numbers, pointers)
    
- **User-defined types** are created by programmers to:
    - model real-world entities
    - group related data
    - express abstractions
    - control ownership and behavior
    
→ See: **[[User-Defined Types in C++]]**

## Types vs value categories

- In C++, behavior is not determined by **type alone**
- It also depends on the **value category**:
    - lvalue
    - rvalue
    - prvalue / xvalue (modern C++)
      
- Value categories affect:
    - copying vs moving
    - binding to references
    - overload resolution
      
→ See: [[Value Categories in C++]]

## Data type families in C++

- Primitive numeric types  
	- [[Primitive Integral Types in C++]]
	- [[Floating-Point Types in C++]]
    
- Characters & strings 
	- [[Character and String Types in C++]]
    
- Fixed-width & size-related types  
	- [[Fixed-Width Integer Types in C++]]
    
- Pointers & references 
	- [[Pointer and Reference Types in C++]]
    
- User-defined types
	- [[User-Defined Types in C++]]
    
- Template-based instantiated types
	- [[Generic Types in C++]]
    
- Special / meta types
	- [[Special Types in C++]]
    
- Time & date-specific types  
	- [[C++ Time & Date Datatypes, Timestamps]]
	  
---

## Overview Table

| Datatype Category                                                             | Type / Feature                                                                                                                                                                                              |
| ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **[[Primitive Integral Types in C++\|Primitive Integral Types]]**             | `bool`, `char`, `signed char`, `unsigned char`, `short`, `unsigned short`, `int`, `unsigned int`, `long`, `unsigned long`, `long long`, `unsigned long long`                                                |
| **[[Floating-Point Types in C++\|Floating-Point Types]]**                     | `float`, `double`, `long double`                                                                                                                                                                            |
| **[[Character and String Types in C++\|Character / String Types]]**           | `char`, `wchar_t`, `char16_t`, `char32_t`, `std::string`, `std::string_view`                                                                                                                                |
| **[[Fixed-Width Integer Types in C++\|Fixed-Width Integer Types (cstdint)]]** | `int8_t`, `int16_t`, `int32_t`, `int64_t`, `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t`, `size_t`, `ptrdiff_t`                                                                                             |
| **[[C++ Time & Date Datatypes, Timestamps]]**                                 | `std::time_t`, `std::tm`, `std::clock_t`, `std::chrono::duration`, `std::chrono::time_point`, `std::chrono::system_clock`, `std::chrono::steady_clock`, `std::chrono::milliseconds`, `std::chrono::seconds` |
| **[[Pointer and Reference Types in C++\|Pointer/Reference Datatypes]]**       | `int*`, `std::nullptr_t`, `int&`, `int&&`                                                                                                                                                                   |
| **[[User-Defined Types in C++\|User-Defined Datatypes]]**                     | `struct`, `class`, `enum`, `enum class`, `union`                                                                                                                                                            |
| **[[Generic Types in C++\|Template-based instantiated types]]**               | `std::optional<T>`, `std::variant<T...>`, `std::pair<T, U>`, `std::tuple<T...>`, `std::array<T, N>`                                                                                                         |
| **[[Special Types in C++\|Special Types]]**                                   | `void`, `auto`, `decltype`, `std::nullptr_t`, `std::byte`                                                                                                                                                   |

# References

- 
