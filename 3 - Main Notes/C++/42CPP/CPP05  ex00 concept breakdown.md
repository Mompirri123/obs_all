**Created:** *<span class ="color-green">23.12.25, 08:56</span>*

**Note Type:**

**Hashtags:**
- **Relevance Tags:**
	- 
- **Topic Tags:**
	- 

**Links / Tags:** 
- **Relevance Links:**
	- 
- **Topic Links:**
	- 
	  
---

# CPP05  ex00 concept breakdown

## Problem overview

- You must implement a `Bureaucrat` class with:​
	- A constant name
	- An integer grade in the range (1 = highest, 150 = lowest)​
	- Grade validation in constructor and when incrementing/decrementing
		- using custom exceptions
	- Getters, grade manipulation functions, and `operator<<` overload for formatted printing
	  

## Core C++ concepts involved
Here are the main concepts you’ll touch in ex00, split into small units:

1. **Class definition basics**
    - Declaring a class in a header (`Bureaucrat.hpp`).​
    - Private data members vs public member functions.
    - Constructors and destructor declarations.
    
2. **Const data members and initialization**
    - Meaning of a `const` data member (e.g. `const std::string _name;`).​
    - Why it must be initialised in a constructor initialiser list.
    - Consequences: name cannot change after construction.
    
3. **Constructor initialiser lists**
    - Syntax and purpose of initialiser lists.
    - Initialising both `const` members and regular members.
    - Using the initialiser list plus runtime checks to validate grade and throw if invalid.
    
4. **Encapsulation and getters**
    - Providing `getName()` and `getGrade()` to access private members.​
    - Marking getters as `const` member functions.
    - Returning by value vs by reference (for `std::string`).
    
5. **Value constraints and invariants**
    - Defining and enforcing allowed range for grade (1…150).​
    - Keeping the object in a valid state at all times (class invariant).
    - Where to check: constructor, `incrementGrade()`, `decrementGrade()`.
    
6. **Exceptions: basic concept**
    - What an exception is and why it’s used for invalid grades.
    - `try` / `catch` blocks and control flow when throwing.
    - Using `std::exception` as a base type for catching.​
    
7. **Custom exception classes**
    - Declaring nested exception classes like `Bureaucrat::GradeTooHighException` / `GradeTooLowException` (or similarly named).​
    - Inheriting from `std::exception`
    - Overriding `what()` to provide an error message
    
8. **Throwing and catching exceptions**
    - Using `throw` in the constructor and in grade modification functions
    - Catching with `catch (std::exception &e)` and using `e.what()`
    - Understanding stack unwinding at a conceptual level (no need for deep detail yet)
    
9. **Incrementing/decrementing with non‑intuitive ordering**
    - Interpreting the problem’s grading logic:
        - Lower numeric value = better grade
        - “Incrementing” a grade means moving toward 1 (e.g. 3 → 2)
    - Ensuring boundaries are respected (no <1, no >150)
      
10. **Operator overloading: `operator<<`**
    - How to declare a non-member `operator<< (std::ostream &, const Bureaucrat &)`
    - Returning the `std::ostream &` to allow chaining
    - Formatting output exactly as required: `"name, bureaucrat grade X"`
    
11. **Separation of interface and implementation**
    - Declarations in `.hpp`, definitions in `.cpp`.​
    - Include guards in headers.
    - Avoiding definitions in headers (except for trivial inline or templates, which you don’t need here).
    
12. **Makefile and compilation model**
    - Compiling multiple translation units (`main.cpp`, `Bureaucrat.cpp`)
    - Linking them into a single executable via the Makefile.
    - Ensuring compilation with flags like `-Wall -Wextra -Werror -std=c++98`
    
13. **Testing and main function design**
    - Writing `main.cpp` that:
        - Creates valid `Bureaucrat` objects.
        - Tries constructing with invalid grades and catches exceptions.​
        - Tests boundary behavior (1 and 150) for increment/decrement.
        
    - Printing results to verify everything.
    
14. **Orthodox Canonical Form (conceptual for later)**
    
    - Understanding what “Orthodox Canonical Form” means 
	    - (default ctor, copy ctor, assignment operator, destructor), even though ex00 is simple
    - How it applies to classes generally in Module 05 (exceptions excluded)
    

## Concept 1 - Classes definition basics

# External References
