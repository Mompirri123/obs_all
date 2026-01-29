**Created:** *<span class ="color-green">09.12.25, 08:12</span>*

**Note Type:** #atomic 

**Hashtags:**

- **Relevance Tags:**
    - #cpp
    - #OOPS 
    - #classdesign
    - #specialmemberfunctions
    
- **Topic Tags:**
    - #copyassignmentoperator
    - #assignmentoperator
    - #operatoroverloading 
    - #copy

**Links / Tags:**

- **Relevance Links:**
    - Classes and Objects in C++ <!-- main parent, plain text -->
    - Canonical Forms in C++ <!-- lifetime cluster, plain text -->
    - Operator Overloading in C++
    - Classic Canonical Form in C++
    - Rules of Three Five and Zero in C++
      ￼ ￼
- **Topic Links:**
	- [[Default Copy Assignment in C++]]
	- [[Overloading Copy Assignment Operator in C++]]


---

# Copy Assignment Operator in C++

> Assigns one existing object’s contents to another existing object of the same class.

- It is both:
	- a **special member function** (controls object lifetime)
	- an **overloaded operator** (`operator=`)

## Kinds of Copy Assignment Operators

- [[Default Copy Assignment in C++|Default Copy Assignment Operator]]
	- Generated automatically by compiler
	- When we don't write our own
	  
- [[Overloading Copy Assignment Operator in C++|Overloaded Copy Assignment Operator]]
	- [[Overloading Copy Assignment Operator in C++|User Defined Copy Assignment Operator]]
	- Replaces or extends the default behaviour

## Typical signature / Syntax

```cpp
Myclass& operator=(const MyClass& other);
```

### Syntax Breakdown

- `MyClass&`
	- `return` type
	- returns a reference to `*this`
	- Let's us chain assignments `a = b = c`
	  
- `operator=`
	- function name used by compiler to write `a = b`
	- i.e when we write `a = b` it changes to
		- `a.operator=(b);` // for member versions
		- `operator=(a, b);` // for non-member versions
		  
	  - The function signature is typically
		  - `MyClass& MyClass::operator=(const MyClass &other)`
	
  - `const MyClass &other`
	  - `&other` is address / reference to the source object we are copying from
	  - `const` promises not to modify `other` inside the assignment operator
	  - reference(`&`) $\rightarrow$ no extra copy, just for the parameter

# Internal References




# External References
