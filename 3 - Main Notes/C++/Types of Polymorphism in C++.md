**Created:** *<span class ="color-green">09.12.25, 23:58</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
    
    - #cpp
    - #oops
    - #polymorphism
    - #map
       
- **Topic Tags:**
    
	- #compiletimepolymorphism
    - #runtimepolymorphism
    - #functionoverloading
    - #operatoroverloading
    - #virtualfunctions
    - #abstractclasses
 

**Links / Tags:**

- **Relevance Links:**
    - Polymorphism in C++ <!-- parent, plain text -->
    - OOPS in C++ <!-- higher-level map -->
    - C++ Language Features <!-- related, plain text -->
     
- **Topic Links:**
    - [[Compile Time Polymorphism in C++]]
	    - Function Overloading in C++
	    - Operator Overloading in C++
	    - Constructor Overloading in C++
	    - Templates in C++
    - [[Runtime Polymorphism in C++]]
	    - Virtual Functions in C++
	    - Abstract Classes in C++

---

# Types of Polymorphism in C++

> Main flavours of polymorphism in C++

## 1. [[Compile Time Polymorphism in C++|Compile-time Polymorphism]]

- These are **Resolved during Compilation**
- The compiler decides which code to call based on types and parameters

### Typical Mechanisms

- Function Overloading in C++ 
	- same function name, different parameter types
	  
- Operator Overloading in C++
	- Redefine `+`, `==` etc., for user-defined types
	  
- Constructor Overloading in C++
	- Multiple Constructors for different ways to build the same class
	
- Templates in C++
	- compiler generates type-specific code from a template

## 2. [[Runtime Polymorphism in C++|Runtime Polymorphism]] (Dynamic / Late Binding)

- Resolved during **Runtime** / **Program Execution**
- Calls depend on actual object type, not just the pointer / reference type

### Typical Mechanisms

- [[Virtual Functions in C++]]
	- Calling through a base-class pointer/ reference, selects the correct override at runtime
	
- [[Abstract Classes in C++]]
	- Interface-like base classes with pure virtual functions that derived class must implement

## High-level Comparison

|Aspect|Compile time polymorphism|Runtime polymorphism|
|---|---|---|
|Resolution|at compile time|at runtime|
|Performance|faster (no virtual dispatch)|slightly slower|
|Flexibility|limited to known overloads/templates|high (depends on actual object type)|
|Mechanism|overloading, templates|virtual functions, abstract interfaces|

# External References

