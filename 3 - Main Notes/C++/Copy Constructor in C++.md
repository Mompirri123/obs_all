**Created:** *<span class ="color-green">10.12.25, 02:33</span>*

**Note Type:** #atomic 

**Hashtags:**
- **Relevance Tags:**
	- #cpp 
	- #OOPS 
	- #classdesign 
- **Topic Tags:**
	- #copyconstructor
	- #copysemantics 
	- #specialmemberfunctions 

**Links / Tags:** 
- **Relevance Links:**
	- Classes and Objects in C++ %% Parent %%
	- Canonical Forms in C++ %% Lifetime cluster %%
	- Copy Semantics %% Shallow vs Deep %%
- **Topic Links:**
	- [[Default Copy Constructor in C++]]
	- [[User-Defined Copy Constructor in C++]]
	- [[Disabling Copy Constructor in C++]]
	- [[Copy Assignment Operator in C++]]
	- [[Rules of Three Five and Zero in C++]]

---

# Copy Constructor in C++

> Special Constructor used to create a new Object from an existing object of same class

- A **Special Member Function**
- Controls how an Object is *copied at creation time*

## Basic Syntax

```cpp
class MyClass
{
public:
	MyClass(const MyClass &other);
}

MyClass::MyClass(const MyClass &other);
```

**Parts:**
- `MyClass`
	- name of the class
	- constructor must have the **same name** as the class
- `const MyClass &other`
	- source object we are copying from
	- `const` $\rightarrow$ we promise not to modify `other`
	- reference `&` $\rightarrow$ no extra copy just for the parameter

**function**
- Creates a **new** object
- Initialises it using an **existing** object (`other`)
- part of the **special member functions** cluster
	(constructor, destructor, copy/move operations)

## When is the copy constructor called?

#### Typical Cases:

1. **Direct Copy Initialisation**
2. **Copy Initialisation**
3. **Pass by Value**
4. **Return by Value** (maybe optimised away, or moved)

```cpp
MyClass a; // existing object

MyClass b(a);  // 1) direct copy initialization
MyClass c = a; // 2) copy initialization

void func(MyClass x);
func(a);       // 3) pass by value → copy

MyClass make();
MyClass d = make(); // 4) return by value (may call copy constructor(ctor), or elide / move)
```

- Copy Constructor $\rightarrow$ Creating new from existing
- Copy Assignment $\rightarrow$ Replace existing from existing

see: [[Copy Assignment Operator in C++]]

## Sub notes

- [[Default Copy Constructor in C++]]
	- Compiler-generated, member wise (shallow) copy
- [[User-Defined Copy Constructor in C++]]
	- Write the body (often for deep copy of resources)
- [[Disabling Copy Constructor in C++]]
	- deleting the copy constructor to make the type non-copyable

# External References
