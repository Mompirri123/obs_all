---
aliases:
  - OCF in C++
  - OCF C++
  - Orthodox Canonical Form
  - OCF
  - Classic Canonical Form in C++
---
**Created:** 08.12.25, 05:30

**Note Type:** #concept

**Hashtags:**

- #cpp
- #canonicalform
- #orthodox
- #special_member_functions

**Links / Tags:**

- **Relevance Links:**
    - Canonical Forms in C++
    - Classes and Objects in C++
	    - Constructors in C++
- **Topic Links:**
	- [[Rule of Three]]
    - [[Default Constructor in C++|Default Constructor]]
	- [[Copy Constructor in C++|Copy Constructor]]
    - [[Copy Assignment Operator in C++ |Copy Assignment Operator]]
    - [[Destructors in C++ |Destructor]]

---

# Orthodox Canonical Form in C++

> Classic C++98 idiom for making object lifetime behaviour explicit and safe

## What is Orthodox Canonical Form (OCF)

- Orthodox Canonical Form is a **traditional C++98 design idiom**
- It formalises how a class manages:
    - creation
    - copying
    - assignment    
    - destruction
    
- A class following OCF **explicitly declares** the core special member functions
- This makes ownership, copying, and cleanup rules **visible and intentional**

- OCF is closely related to the **[[Rule of Three]]**.


## The four functions in Orthodox Canonical Form

- A class following OCF explicitly defines:
	- [[Default Constructor]]
	- [[Copy Constructor in C++|Copy Constructor]]
	- [[Copy Assignment Operator in C++|Copy Assignment Operator]]
	- [[Destructor]]
	  
- Together, these four fully define the object’s **basic lifetime semantics**.
	- so its **lifetime behaviour is fully defined and visible**
	  
- In C++11 and later, a “modern” canonical form may also add:
    - Move constructor
    - Move assignment operator
      
- But the core idea stays the same:
	- the class clearly controls how it is **created, copied, moved, and destroyed.**
	
---

## Why Orthodox Canonical Form exists

- ***Without OCF***:
	- the compiler may implicitly generate copy and assignment operations
	- those defaults perform **shallow copies**
	- shallow copies are often wrong for:
	    - raw pointers
	    - ownership of memory
	    - file handles
	    - mutexes
	    - other resources
	
- ***With OCF***:
	- copying is intentional
	- assignment is intentional
	- destruction is explicit
	- ownership rules are obvious to the reader
	  

> OCF makes object lifetime **predictable and auditable**

## Classic canonical form (C++98 style)

```cpp
class Buffer
{
private:
    int* data;
    size_t size;
    
public:
    Buffer();                               // default constructor
    Buffer(const Buffer& other);            // copy constructor
    Buffer& operator=(const Buffer& other); // copy assignment
    ~Buffer();                              // destructor
};
```

- This is called _orthodox_ because it follows the traditional, explicit C++98 object model

---

## What problem Orthodox Canonical Form solves

- Without OCF:
    - the compiler may generate copy and assignment behaviour automatically
    - that behaviour may be incorrect for classes that own resources
    - ownership rules are hidden and unclear
- With OCF:
    - copy behaviour is intentional
    - assignment behaviour is intentional
    - destruction behaviour is visible
    - object lifetime is easy to reason about

---

## Responsibility of each function for OCF

### Default Constructor

- defines how a valid object is created with no arguments
- ensures the object starts in a usable state

---

### Copy Constructor

- defines how a **new object** is created from an existing object
- called when:

```cpp
T b(a);
T b = a; // b is just created
```

---

### Copy Assignment Operator

- defines how an **existing object** is overwritten by another object
- called when:

```cpp
b = a; // b was created well before assignment
```

---

### Destructor

- defines how the object cleans up when its lifetime ends
- called automatically when:
    - an object goes out of scope
    - an object is deleted
    - a temporary object is destroyed

---

## Key idea

- Orthodox Canonical Form is about **making object lifetime visible**
- no hidden compiler-generated behaviour
- no ambiguity about copying, assignment, or destruction

---

## Internal References
- [[Rule of Three]]
- [[Special Member Functions in C++]]

## External References
- cppreference.com — Special member functions
- C++98 standard notes on copy semantics