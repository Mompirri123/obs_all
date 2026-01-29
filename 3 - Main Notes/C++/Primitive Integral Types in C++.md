**Created:** 03.12.25, 01:49

**Status:** #atomic

**Hashtags:**
- #cpp
- #datatypes
- #atomic
- #numeric

**Links / Tags:** 
- **Relevance Links:**
	- [[Floating-Point Types in C++]]
	- [[Fixed-Width Integer Types in C++]]
	  
- **Topic Tags:**
	- #integraltypes
	- #signed
	- #unsigned
	- #integerranges

# Primitive Integral Types in C++

> Part of C++ data types.

Primitive integral types represent **whole numbers** and **simple boolean** values.
They differ mainly in **size** and whether they are **signed** or **unsigned**.

## List of primitive integral types

- `bool` – logical true/false  
- `char` – typically a small integer used for characters  
- `signed char` – small signed integer  
- `unsigned char` – small unsigned integer  
- `short`, `unsigned short` – small integers  
- `int`, `unsigned int` – “default” integer size for the platform  
- `long`, `unsigned long` – larger integers  
- `long long`, `unsigned long long` – guaranteed at least 64 bits

## Key ideas

- **Signed vs unsigned**  
	- *Signed*: can store negative and positive values
		- `int`, `long` etc.,
	- *Unsigned*: only non-negative, but larger max value in same number of bits.
		- `unsigned int`, `unsigned long` etc.,
	  
- **Overflow**  
	- Behaviour of signed overflow is undefined.  
	- Unsigned overflow wraps modulo $2^n$.
	  
- **Type choice**  
	- `int` for most general counting.  
	- `size_t` (see fixed-width note) for sizes and indexes.  
	- wider types only when we really need them.

# References

- 
