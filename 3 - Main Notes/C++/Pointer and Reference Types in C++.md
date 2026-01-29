**Created:** 03.12.25, 04:54

**Status:** #atomic

**Hashtags:**
- #cpp
- #datatypes
- #atomic
- #memory
**Topic Tags:**
- #pointers
- #references
- #nullptr
- #rvaluereferences

**Links / Tags:** 
- **Relevance Links:**
	- [[User-Defined Types in C++]]
	- [[Special Types in C++]]
	- [[Lvalue and Rvalue references in c++]]
	  
---

# Pointer and Reference Types in C++

> Part of C++ datatypes.

Pointers and references let you refer to objects indirectly, which is central to C++’s memory model.

## Pointers

- Raw pointer: `int* p;` – holds the address of an `int`.  
- Null pointer: `std::nullptr_t` / `nullptr` – represents “no address”.

Key points:

- You must manage lifetime and validity manually.  
- De-referencing invalid pointers → undefined behaviour.

## References

- [[Lvalue and Rvalue references in c++|Lvalue]] reference: `int& r = x;` – alias for an existing object.  
- [[Lvalue and Rvalue references in c++|Rvalue]] reference: `int&& r = foo();` – used for move semantics and perfect forwarding.

Key points:

- References must be bound on initialization and can’t be reseated.  
- Rvalue references typically appear in templates and move constructors.

---

# References

- 
