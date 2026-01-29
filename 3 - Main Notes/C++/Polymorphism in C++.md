**Created:** 27.11.25, 16:50

**Status:** 

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #oops
    - #polymorphism
       
- **Topic Tags:**
    - #compiletimepolymorphism
    - #runtimepolymorphism
    - #functionoverloading
    - #operatoroverloading
    - #virtualfunctions
    - #abstractclasses

**Links / Tags:**

- **Relevance Links:**
	- OOPS in C++ <!-- parent, plain text -->
    - Classes and Objects in C++ <!-- related, plain text -->
    - C++ Language Features <!-- related, plain text -->
       
- **Topic Links:**
    - [[Types of Polymorphism in C++]]
	    - Compile Time Polymorphism in C++
		    - Function Overloading in C++
		    - Operator Overloading in C++
		    - Constructor Overloading in C++
		    - Templates in C++
	    - Runtime Polymorphism in C++
		    - Virtual Functions in C++
		    - Abstract Classes in C++

---

# Polymorphism in C++

- Polymorphism $\implies$ "Many forms"
- Ability to process objects differently based on their data type or class
- Same Interface, different Implementations / behaviour depending on
	- data type
	- actual class of the object
- same function / call site
	- different implementation behind it

## [[Types of Polymorphism in C++|Types of Polymorphism]]

> See: [[Types of Polymorphism in C++]] for detailed information

- Run time Polymorphism in C++
	- Function Overloading
	- Operator Overloading
	- Constructor Overloading
	  
- Compile time Polymorphism in C++
	- Virtual Functions
	- base-class pointers / Interfaces
	- abstract classes as interfaces


### Key Differences

| Aspect   $\Downarrow$ | Compile-time    $\Downarrow$               | Runtime    $\Downarrow$ |
| --------------------- | ------------------------------------------ | ----------------------- |
| **Resolution**        | During compilation                         | During execution        |
| **Performance**       | Faster                                     | Slightly slower         |
| **Flexibility**       | Limited                                    | High                    |
| **Mechanism**         | Function overloading, Operator overloading | Virtual functions       |

### When to use Which?

- **Compile-time Polymorphism**
	- When behaviour is known at compile time
	- when we want zero or minimal runtime overhead
	- ex: maths libraries using templates, small overload sets
	
- **Runtime Polymorphism**
	- When behaviour depends on actual object type at runtime
	- When we want to work through Interfaces / Base classes
	- ex: GUI widgets, Plug-in systems, `Database *db` with multiple implementations

### Mental Models

- **Compile-time Polymorphism**
	- The compiler decides which function to call before program runs
- **Runtime Polymorphism**
	- The program checks the actual object at runtime and calls matching function via `virtual`

# References

## Closely Related Notes

### Next:

### Prev:
