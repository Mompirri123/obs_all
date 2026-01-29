**Created:** *26.11.25, 21:08*

**Status:** #atomic 

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #functions
    - #lambdafunctions
    
- **Topic Tags:**
    - #anonymousfunctions
    - #captures
    - #moderncpp
    
**Links / Tags:** 
- **Relevance Links:**
	- Functions in C++
	
- **Topic Tags:**
	- [[Lambda Capture Modes in C++]]
	- [[Generic Lambdas in C++]]
	- [[Lambdas with STL Algorithms in C++]]
	
---

# Lambda Functions - (C++11)

> Small anonymous function we can write inline

- *lambda function* also called as *lambda*
- Small, Anonymous function you can write directly in your code
	- Anonymous -> No name
- Perfect for short, one-off operations

### Syntax

- `[capture-list] (parameters) -> return_type {// function body };`
	- `[capture-list]`
		- Variables from outside scope that **lambda** can access
		- details in [[Lambda Capture Modes in C++]]
	- `(parameters)`
		- list of parameters that **lambda** takes as input
	- `return_type` 
		- (optional) type of the return value
	- `{// function body}` 
		- the body of the function
	
 - Can also be written as 
	`[capture-list] (parameter-list) -> return_type {};`


#### Example 
```C++
int multiplier = 3;

auto times = [multiplier] (int x, int y)  -> int {return (x * multiplier);};
// same as
// auto times = [multiplier] (int x, int y) {return (x * multiplier);};


std::cout << times(5) << std::endl;
```

- `[multiplier]` - *capture list*
	- variable from outside that can be accessed
	- `multiplier` is accessed by value, so lambda gets its own copy
- `(int x, int y)` - *parameter list*
	- lambda takes two parameters named `x`, `y` of type `int`
- `-> int` - *return type*
	- return type
- `(x * multiplier);` - *body*
	- code executed when lambda is called


# References:
