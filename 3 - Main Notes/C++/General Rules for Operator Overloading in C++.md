**Created:** *<span class ="color-green">12.12.25, 05:58</span>*

**Status:** #map

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #OOPS 
    - #operatoroverloading
- **Topic Tags:**
    - #polymorphism

**Links / Tags:** 
- **Relevance Links:**
    - Operators in C++               <!-- grand parent, plain text on purpose -->
    - Classes and Objects in C++     <!-- also related, plain text -->
    - Operator Overloading in C++      <!-- parent -->
    
- **Topic Links:**
	- [[Member vs Non Member Operator Overloads in C++]]

---

# General Rules for Operator Overloading
> Fundamental constraints that **cannot break**

- We can not define new operators
	- such as `..`
	  
- Cannot redefine the meaning of operators when applied to built-in datatypes
	- `int` + `int` can not be redefined
	- Overloaded operators must either be a **non static class member** function or a **global** function

# References
