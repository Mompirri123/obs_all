**Created:** *<span class ="color-green">09.12.25, 13:36</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #oop
    - #classdesign
- **Topic Tags:**
    - #copysemantics
    - #shallowcopy
    - #deepcopy
      
**Links / Tags:** 
- **Relevance Links:**
	- Classes and Objects in C++        <!-- Parent -->
    - Memory management in C++          <!-- Parent -->
    - Inheritance in C++
- **Topic Links:**
	- [[Shallow Copy in C++]]
	- [[Deep Copy in C++]]
	- [[Copy Constructor in C++|Copy constructor]]
	- [[Copy Assignment Operator in C++]]
	- [[Canonical Forms in C++]]
	- [[Rules of Three Five and Zero in C++]]

---

# Copy semantics in C++

> How objects are **copied** in C++: 
> 	what exactly is duplicated when you do `a = b;` or `MyClass c = b;`.

Two main ideas:

- [[Shallow Copy in C++]]
	- copies **member values as-is** 
	- including pointer addresses
	  
- [[Deep Copy in C++]]
	- copy the **data behind** the pointers / resources
	  
- These matter most when a class **owns resources** (heap memory, file handles, etc.).

# Internal References

- [[Copy Constructor in C++]]
	- copying when creating a new object
	  
- [[Copy Assignment Operator in C++]]
	- copying into an existing object
	
- [[Canonical Forms in C++]]
	- how copy semantics fit lifetime design
	  
- [[Rules of Three Five and Zero in C++]]
	- 

# External References
