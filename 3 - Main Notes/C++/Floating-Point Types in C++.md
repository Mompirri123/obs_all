---
aliases:
  - Floating Point Types in C++
---
**Created:** 03.12.25, 01:53

**Status:** #atomic

**Hashtags:**
- #cpp
- #datatypes
- #atomic
- #numeric

**Links / Tags:** 
- **Relevance Links:**
    - [[Primitive Integral Types in C++]]     <!-- numeric sibling, tightly related -->
    - [[IEEE754]]                             <!-- floating-point standard -->

- **Topic Tags:**
    - #floatingpoint
    - #precision
    - #rounding
    - #ieee754

---

# Floating-Point Types in C++

> part of C++ data types.

Floating-point types represent real numbers with fractional parts, but with limited precision.

## Types

- `float` – single-precision (usually 32-bit)  
- `double` – double-precision (usually 64-bit; default choice)  
- `long double` – extended precision (implementation-dependent)

## Key ideas

- **precision vs range**  
  - `float` → less precision, more rounding error  
  - `double` → standard choice for most work  
  - `long double` → higher precision only when needed

- **rounding & representation errors**  
  - many decimal values cannot be represented exactly  
  - comparisons often require tolerance (epsilon), not `==`

- **Use cases**  
  - geometry, simulations, physics → usually `double`  
  - memory-sensitive large arrays (graphics, ML tensors) → sometimes `float`

# References

-
