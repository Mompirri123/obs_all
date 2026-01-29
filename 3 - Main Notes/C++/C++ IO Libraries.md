**Created:** *<span class ="color-green">11.01.26, 19:33</span>*

**Note Type:** #map 

**Hashtags:**
- **Relevance Tags:**
	- #cpp 
	- #standardlibrary 
	- #io 
	
- **Topic Tags:**
	- #streams
	- #files 
	- #formatting
	
**Links / Tags:** 
- **Relevance Links:**
	- C++ Standard Library
	
- **Topic Links:**
	- [[Console IO in C++]]
	- [[File Streams in C++]]
	- [[String Streams in C++]]
	- [[Formatted IO and manipulators in C++]]
	
---

# C++ IO Libraries

> Input and Output facilities in C++ Standard Library

- The C++ IO libraries define **stream-based mechanisms** for reading from and writing to external sources such as
	- Console
	- Files
	- Strings
	- Other Input Output (I/O) devices
	  
- They are designed around *stream abstraction*
- Stream abstractions provide a type safe, extensible and composable wat to perform IO operations
---

## Conceptual Overview

- C++ IO is built around the idea that *data flows through streams*
	- Input streams consume data
	- output streams produce data
	- streams can be changed, formatted and customised
	  
- IO facilities integrate closely with:
	- C++ type system
	- Operator Overloading
	- RALL for resource management
	  
---

## Core Stream Categories

### 1. Console IO

- Reading from standard input
- Writing to standard output and error
- Based on:
    - `std::cin`
    - `std::cout`
    - `std::cerr`
    - `std::clog`
    
→ See: [[Console IO in C++]]

---
### 2. File IO

- Stream-based reading and writing of files
- Text and binary file support
- Based on:
    - `std::ifstream`
    - `std::ofstream`
    - `std::fstream`
    
→ See: [[File Streams in C++ (fstream)]]

---
### 3. String-based IO

- Streams operating on in-memory strings
- Useful for parsing and formatting
- Based on:
    - `std::istringstream`
    - `std::ostringstream`
    - `std::stringstream`
    
→ See: [[String Streams in C++]]

---
### 4. Formatted IO and Manipulation

- Controlling formatting of stream output
- Includes:
    - width, precision, alignment
    - base (decimal, hex, octal)
    - locale-aware formatting
    
→ See: [[Formatted IO and Manipulators in C++]]

---
## Design Characteristics

The C++ IO libraries emphasise:
- type safety
- extensibility via inheritance
- RAII-based resource management
- separation of formatting from data processing

Streams can be:
- customised
- redirected
- combined with user-defined types

---

# Related Notes

> Things you might want to think about alongside this note, but not because of it

- Filesystem abstractions
- Performance optimization
- Locale and internationalisation
- System-level IO
---
# References
- ISO/IEC C++ Standard (IO library clauses)
- [cppreference – IO library](https://en.cppreference.com/w/cpp/io)
