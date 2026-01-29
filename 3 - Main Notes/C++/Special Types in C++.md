**Created:** 03.12.25, 05:09

**Status:** #atomic

**Hashtags:**
- #cpp
- #data_types
- #atomic

**Links / Tags:** 
- **Relevance Links:**
  - [[Generic Types in C++]]
  - [[Pointer and Reference Types in C++]]

- **Topic Tags:**
  - #void
  - #auto
  - #decltype
  - #nullptr_t
  - #stdbyte

# Special Types in C++

> Part of C++ data types.

These types are more about language mechanics than business data.

## void

- Used as:
  - function return type when nothing is returned: `void f();`  
  - pointer to “object of unknown type”: `void*`.

## auto

- Tells the compiler to **deduce the type** from the initialiser.  
- Useful to avoid long template types and keep code flexible.

## decltype

- Yields the type of an expression without evaluating it.  
- Often used in templates and generic programming.

## std::nullptr_t

- Type of `nullptr`.  
- Helps overload resolution distinguish null pointer from integer `0`.

## std::byte

- Raw byte type representing data, not a character or integer.  
- Useful for low-level memory and buffer manipulation.

# References

- 
