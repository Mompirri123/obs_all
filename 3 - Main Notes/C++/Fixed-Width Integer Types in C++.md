**Created:** 03.12.25, 04:52

**Status:** #atomic

**Hashtags:**
- #cpp
- #datatypes
- #numeric

**Links / Tags:** 
- **Relevance Links:**
  - Primitive Integral Types in C++
  - [[Pointer and Reference Types in C++]]

- **Topic Tags:**
  - #fixedwidth
  - #cstdint
  - #size_t
  - #ptrdiff_t

# Fixed-Width Integer Types in C++

> Part of C++ data types.

Fixed-width types give you exact bit sizes, useful for low-level code and binary formats.

## `<cstdint>` integer types

- `int8_t`, `int16_t`, `int32_t`, `int64_t`  
- `uint8_t`, `uint16_t`, `uint32_t`, `uint64_t`

These are `typedef`'s with **guaranteed widths** if the platform supports them.

## Size-related types

- `size_t` – unsigned type large enough to represent sizes of any object  
- `ptrdiff_t` – signed type for difference between pointers

## When to use

- Binary protocols, file formats, embedded systems → `int32_t`, `uint8_t`, etc.  
- Loops over containers and array indices → `size_t`.  
- Pointer arithmetic differences → `ptrdiff_t`.

# References

- 
