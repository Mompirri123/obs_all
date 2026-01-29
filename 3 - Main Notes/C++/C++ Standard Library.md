**Created:** *<span class ="color-green">10.01.26, 07:40</span>

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #standardlibrary
	- #overview
- **Topic Tags:**
	- #stl
	- #io
	- #concurrency
	- #filesystem
	- #numerics
	- #utilities

**Links / Tags:** 
- **Relevance Links:**
	- C++
- **Topic Links:**
	- [[C++ STL]]
	- [[C++ IO Libraries]]
	- [[C++ Concurrency Libraries]]
	- [[C++ Filesystem Libraries]]
	- [[C++ Numeric and Math Libraries]]
	- [[C++ Utility Libraries]]
	- [[C++ Date and Time Libraries]]

---

# C++ Standard Library

> Library facilities specified and guaranteed by the C++ standard.

The C++ Standard Library provides a collection of **portable, reusable,
well-defined components** that complement the core language.

These facilities are:
- standardized by ISO/IEC
- available through headers
- designed to work seamlessly with the C++ type system and memory model

The standard library is **not a single monolithic library**, but a set of
logically grouped facilities covering data structures, algorithms, IO,
concurrency, numerics, and general-purpose utilities.

---

## Conceptual Components

### **[[C++ STL]]**
- The **generic programming core** of the standard library
- Includes:
	- containers
	- iterators
	- algorithms
	- function objects
- Forms the foundation for many higher-level facilities

---

### **[[C++ IO Libraries]]**
- Stream-based input and output
- Console, file, and formatted IO
- Examples:
	- `<iostream>`
	- `<fstream>`
	- `<sstream>`

---

### **[[C++ Concurrency Libraries]]**
- Facilities for multi-threaded programming
- Includes:
	- threads
	- mutexes
	- atomics
	- futures and async
- Designed to support low-level and high-level concurrency patterns

---

### **[[C++ Filesystem Libraries]]**
- Portable filesystem access and manipulation
- Paths, directories, files, permissions
- Introduced in C++17

---

### **[[C++ Numeric and Math Libraries]]**
- Mathematical and numeric utilities
- Includes:
	- `<cmath>`
	- `<numeric>`
	- random number generation
	- numeric limits

---

### **[[C++ Utility Libraries]]**
- General-purpose support facilities
- Includes:
	- type traits
	- optional, variant, tuple
	- time utilities
	- error handling helpers
	- memory utilities

---

### **[[C++ Date and Time Libraries]]**
- Time measurement and date handling facilities
- Includes:
	- `<ctime>` (C-style)
	- `<chrono>` (modern C++)
- Used for timestamps, durations, clocks, and scheduling

---

## Design Philosophy

The C++ Standard Library emphasises:
- zero-cost abstractions
- strong type safety
- deterministic resource management
- performance comparable to hand-written code

Library components are designed to be:
- composable
- efficient
- independent of specific platforms

---

# Related Notes
> Things you might want to think about alongside this note, but not because of it

- Language semantics
- Memory and lifetime
- Performance engineering
- Systems programming

# References
- ISO/IEC C++ Standard (Library clauses)
- [cppreference – C++ standard library](https://en.cppreference.com/w/cpp/standard_library)
