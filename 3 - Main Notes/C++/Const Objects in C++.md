**Created:** *<span class ="color-green">13.12.25, 07:02</span>*

**Note Type:** #atomic 

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #const
	- #objects
- **Topic Tags:**
	- #immutability
	- #safety
	- #compilerchecks

**Links / Tags:** 
- **Relevance Links:**
	- Objects  in C++
- **Topic Links:**
	- [[Const Member Functions in C++]]
	- 

---

# Const Objects in C++

> Part of Object-related concepts in C++

- A **Const Object** is an object whose state we **cannot modify** after creation

## Basic Idea

```cpp
const Student s;
```
- `s` cannot be changed
- only [[Const Member Functions in C++|const member functions]] can be called
- The compiler enforces these rules

## What "const" really applies to

- `const` applies to the **object itself** not the class
- Same class can create `const` and `non const` objects

## Why "const" objects exist

- express intent ("this object should not change")
- Prevent accidental modification
- allow safer APIs
- Enable compiler optimisations

## Common mistakes

```cpp
const Student s;
s.setAge(20); // ❌ error! won't work if setAge is not const
```

- This fails because:
	- modifying member functions are not allowed
	- `this` is treated as `const` inside member functions

see: [[Const Member Functions in C++]]


## Relation to member functions

- const objects restrict which member functions can be called
- Only functions marked `const` are allowed


## Key Takeaways

- Const object $\implies$ read-only state
- class definition stays the same
- constness is enforced at **Compile time**

# Internal References


# External References
