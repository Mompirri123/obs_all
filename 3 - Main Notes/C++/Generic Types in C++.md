**Created:** 06.12.25, 02:59
**Status:** #map

**Hashtags:**

- #cpp
- #genericprogramming
- #datatypes
- #map
- #overview

**Links / Tags:**

- **Relevance Links:**
    
    - C++ Data Types <!-- parent, plain text on purpose -->
    - Templates in C++ <!-- template mechanism, plain text -->
    
    - [[std optional in C++]]
    - [[std variant in C++]]
    - [[std pair in C++]]
    - [[std tuple in C++]]
    - [[std array in C++]]
    - [[C++ STL]]
    - [[Template Mental Models and Use Cases in C++]]
    
- **Topic Tags:**
    
	    - #generictypes
    - #stdoptional
    - #stdvariant
    - #stdpair
    - #stdtuple
    - #stdarray


# Generic Types in C++

- Generic types are **types that depend on other types**.  
- Examples are
	1. `std::optional<int>`
	2. `std::variant<int, string>`
	3. `std::array<double, 10>`

- One template definition can create many concrete types:
    - `std::optional<int>`, `std::optional<string>`, `std::optional<MyStruct>`
    
- They are used to:
    - make code reusable
    - keep type safety
    - express ideas like “maybe a value”, “one of many types”, “a fixed group of values”


Most common generic types you will see in modern C++ come from the standard library and are implemented with templates.

## Main generic types

- [[std optional in C++]] – value that may or may not be present
    
- [[std variant in C++]] – one of several alternative types
    
- [[std pair in C++]] – simple pair of two values
    
- [[std tuple in C++]] – fixed-size heterogeneous bundle
    
- [[std array in C++]] – fixed-size array with STL interface

# References