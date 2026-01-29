**Created:** <span class ="color-green">13.12.25, 15:05</span>*

**Note Type:** #atomic

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #constructors
	- #atomic
- **Topic Tags:**
	- #defaultconstructor
	- #initialization
	- #objectcreation

**Links / Tags:** 
- **Relevance Links:**
	- Types of Constructors in C++        <!-- parent -->
- **Topic Links:**
	- [[Member Initialiser List in C++]]

---

# Default Constructor in C++

a **default constructor** is a constructor that can be called **with no arguments**.

it is used when an object is created without providing any parameters.

---

## what counts as a default constructor

### implicit default constructor

if you do **not** define any constructor, the compiler may generate one:

```cpp
class A
{
    int x;
};

A a;   // default constructor is implicitly generated
```

- The `A a` calls the defaults constructor is implicitly called
- This constructor:
	- Does not initialise fundamental types
	- Value Initialises members only in some contexts

### User-defined default constructor
```cpp
class A
{
public:
	A()
	{
	// custom setup
	// we can write something we want to do here
	// where as the default just initalizes the members / bases and nothing more
	}
};
```

- Now the compiler will not generate its own default constructor

### Default Constructor and member Initialisation

```cpp
class A
{
	int x;
public:
	A() : x(0) {} //member initialiser list
};
```

- Preferred way to give members a known initial state
- avoids uninitialised values

see: [[Member Initialiser List in C++]]

### When a default constructor is deleted

- A default constructor is **not available** if:
- The is having:
	- a reference member
	- a const member without a default initialiser
	- a base class without a default constructor
- and no suitable constructor provided

##### Example:
```cpp
class A
{
	int& r;
}
```

- `A a;` No default constructor possible

## Why Default Constructors matter

- required by many containers (`std::vector`, `std::array` in some cases)
- needed for value-initialisation
- simplifies generic and template code
- enables objects to be created in stages

# Key Takeaways

- Default Constructor = callable with no arguments
- can be compiler-generated or user-defined
- not always available
- often works together with Initialiser lists

