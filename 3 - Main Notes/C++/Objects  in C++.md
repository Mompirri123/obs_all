**Created:** 13.12.25, 05:18

**Note Type:** #atomic

**Hashtags:**
- #cpp
- #objects
- #oops
- #atomic

**Links / Tags:** 
- **Relevance Links:**
    - Classes in C++            <!-- parent -->
    - [[Const Objects in C++]]

- **Topic Tags:**
    - #instances
    - #memory
    - #lifetime

---

# Objects in C++

> part of object-oriented programming in C++.

an **object** is a concrete **instance of a class**.

a class defines *what* exists.  
an object is *that thing in memory*.

---

## What is an Object?

- created from a class  
- occupies memory  
- has its **own copy** of data members  
- can call the class’s member functions  

the same class can create **many independent objects**.

---

## Object Creation

```cpp
Student s1;
Student s2;
```

- `s1` and `s2` are **different objects**
- both follow same class definition
	- but, store separate data
	  
- Memory is allocated when the **Object is Created**, not when the class is defined

## Object Lifetime

- An object's lifetime begins when it is created
- It ends when it goes out of scope or is destroyed
- Constructors run at creation 
- Destructors run at destruction

# Internal References


# External References
