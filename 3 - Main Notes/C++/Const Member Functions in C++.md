**Created:** 13.12.25, 08:13

**Note Type:** #atomic 

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #objects
	- #classes
- **Topic Tags:**
	- #const 
	- #methods
	- #constcorrectness
	  
**Links / Tags:** 
- **Relevance Links:**
	- Objects in C++
	- Classes in C++
- **Topic Links:**
	- [[Const Objects in C++]]
	- 


---

# Const Member Functions in C++

> rules for member functions that do not modify object state.

a **const member function** promises not to change the observable state of the object it is called on.

---
## syntax

```cpp
class Student {
public:
    int getAge() const;
};
```

the `const` appears **after** the function signature.

---

## what const means here

inside a const member function:

- `this` is treated as `const Student*`
- data members cannot be modified
- only other `const` member functions can be called

this restriction is enforced at **compile time**.

---

## why const member functions matter

- allow functions to be called on **const objects**
- express intent: “this function only reads state”
- improve safety and prevent accidental modification
- enable better API design

---

## const and function overloading

member functions can be overloaded based on constness:

```cpp
void print();
void print() const;
```

- non-const object → prefers non-const version
- const object → can only call the const version

this is a very **common C++ pattern.**

## common mistake

```cpp
int getAge() const
{
    age++;   // ❌ not allowed
    return age;
}
```

because modifying data members violates the const promise.

## relation to const objects

- const objects can **only** call const member functions
- non-const objects can call both
- both concepts work together to enforce const-correctness

see: [[Const Objects in C++]]

## key takeaway

- const member functions do not change object state
- `this` becomes a pointer to const
- they are essential for const-correct C++ code

---

# Internal References

# External References