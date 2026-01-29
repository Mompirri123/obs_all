**Created:** *27.11.25, 03:50*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #CPP , #OOPS  , #constructors  
- **Topic Tags:**
	- #specialmemberfunctions    
	- #raii    
	- #classdesign

**Links / Tags:** 
- **Relevance Links:** 
	- Classes  in C++
	
- **Topic Links:**
	- [[Types of Constructors in C++]] 
	- [[Constructor Overloading in C++]]
	- [[Member Initialiser List in C++]]
	- [[Advanced Constructor Concepts in C++]]
	- [[Destructors in C++]]

# Constructors in C++

## Definition

- A constructor is a special function that runs automatically when an object is created
- It has the same name as the class and no return type.

```cpp
class MyClass
{
public:
	MyClass(); // constructor 
};
```

#### Example: Defining and creating a class using constructor

```cpp
class MyClass
{
public:
	// A public Constructor to create the class
	MyClass()
	{
		cout << "Hello World!";
	}
}

int main()
{
	MyClass myObj; // will call the constructor
	return 0;
}
```

## Key Rules

- Constructor must have **same name** as the class
- no `return` type
- usually `public`
- automatically called when the object is created

## Why Constructors Matter

- **Automatic initialisation**  
  Objects start in a valid state immediately after creation.

- **[[Encapsulation in C++|Encapsulation]] and invariants**  
  The constructor controls how the object is set up and can enforce invariants.

- **RAII (Resource Acquisition Is Initialisation)**  
  Acquire resources in the constructor, release them in the destructor.

- **[[Constructor Overloading in C++|Constructor Overloading]]**  
  Provide multiple ways to create an object with different parameter sets.

- **Default values**  
  Allow creating objects with sensible defaults when no arguments are given.

# Related Notes

## Sub-maps

- [[Types of Constructors in C++]]  
    Different constructor kinds: default, parameterised, copy, move, delegating, converting.
    
- [[Advanced Constructor Concepts in C++]]  
    Initialiser lists, const members, explicit, default/delete, etc.

# External References
