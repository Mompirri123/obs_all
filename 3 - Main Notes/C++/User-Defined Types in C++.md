**Created:** 03.12.25, 05:04

**Status:** #atomic

**Hashtags:**
- #cpp
- #datatypes
- #atomic
- #oops

**Links / Tags:** 
- **Relevance Links:**
	- OOPS in C++
	- [[Templates in C++]]
	- [[enums in C++]]
	- [[structs in C++]]
	- [[Classes in C++]]
	- [[Class vs Struct in C++]]
	- [[Union in C++]]

- **Topic Tags:**
	- #struct
	- #class
	- #enum
	- #enumclass
	- #union

# User-Defined Types in C++

> Part of C++ data types.

User-defined types let you model your own concepts using the language’s type system.

## struct and class

- `struct` – default public members  
- `class` – default private members  

Otherwise they are the same; you choose based on style and intent.

## enum and enum class

- `enum` – unscoped enumeration; names leak into the surrounding scope.  
- `enum class` – scoped enumeration; safer, must be qualified (e.g. `Color::Red`).

## union

- `union` – several members share the same memory. Only one is active at a time.  
  Useful for low-level code; can be tricky to use safely.

# References

- 
