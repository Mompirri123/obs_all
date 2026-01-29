**Created:** 08.12.25, 05:58

**Note Type:**

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #OOPS 
	- #classdesign
	- #map
- **Topic Tags:**
	- #specialmemberfunctions
	- #lifetime
	- #raii
**Links / Tags:** 
- **Relevance Links:**
	- Classes and Objects in C++        <!-- parent, plain text on purpose -->
	- [[Orthodox Canonical Form in C++]]
	- [[Modern Canonical Forms in C++]]
	- [[Rules of Three Five and Zero in C++]]
- **Topic Links:**
	- 

---
# Canonical Forms in C++

> How to give a class a clear, well defined lifetime in C++.

- “Canonical form” answers:
	
	- how is this class **constructed**?
	- how is this class **copied** or **moved**?
	- how is this class **destroyed**?
	
- Different eras of C++ favour different canonical shapes.

## Main variants

- [[Orthodox Canonical Form in C++]]  
	- C++98 style: default ctor, copy ctor, copy assignment, destructor.

- [[Modern Canonical Forms in C++]]  
	- C++11 and later:
	- resource owning form (rule of five)
	- rule of zero form (no special members).

## Related design rules

- [[Rules of Three Five and Zero in C++]]  
  Guidelines that tell you when you must define special member functions.

# References
