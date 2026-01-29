**Created:** <span class ="color-green">13.12.25, 13:55</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #constructors
	- #oops
	- #map
- **Topic Tags:**
	- #specialmemberfunctions
	- #objectlifecycle
	- #initialization

**Links / Tags:** 
- **Relevance Links:**
	- Constructors in C++        <!-- parent -->
- **Topic Links:**
	- [[Default Constructor in C++]]
	- [[Parameterised Constructor]]
	- [[Copy Constructor in C++]]
	- [[Move Constructor in C++]]
	- [[Delegating Constructors in C++]]
	- [[Converting Constructors in C++]]

---

# Types of Constructors in C++

> this note classifies constructor types.  

constructors come in different **forms**, each solving a different object-creation problem.
each type has (or will have) its own focused note.

## Core constructor types

### Default constructor
- called when no arguments are provided
- creates an object with default state

→ [[Default Constructor in C++]]

### Parameterised constructor
- takes parameters to initialise the object
- most common constructor form

→ [[Parameterised Constructor]]

## Copy and move semantics

### Copy constructor
- creates a new object from an existing one
- used in copying, pass-by-value, return-by-value

→ [[Copy Constructor in C++]]

### Move constructor
- transfers ownership of resources
- avoids expensive deep copies
- used with temporaries and rvalues

→ [[Move Constructor in C++]]

## Advanced / structural forms

### Delegating constructors
- one constructor calls another constructor of the same class
- reduces duplicated initialisation logic

→ [[Delegating Constructors in C++]]

### Converting constructors
- single-argument constructors
- allow implicit or explicit type conversion
- often paired with `explicit`

→ [[Converting Constructors in C++]]

## Notes on classification

- all constructors are **member functions**
	
- they differ by:
	- parameters
	- how objects are created
	- how resources are handled
	  
- some constructors interact closely with:
	- copy / assignment semantics
	- const correctness
	- initialiser lists

those relationships are handled in sibling notes.

# External References

-
