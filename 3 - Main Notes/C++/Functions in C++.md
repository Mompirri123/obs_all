**Created:** 27.11.25, 13:41

**Status:** 

**Hashtags:**
- #CPP 
- #functions

**Links / Tags:** 
- **Relevance Links:**
	- C++ Language Features

- **Topic Tags:**
	- [[Lambda Functions in C++]]
	- [[Function Overloading in C++]]
	- [[Friend Functions in C++]]
	- [[Methods in C++]]
	- [[Virtual Functions in C++]]
	- [[Function Templates in C++]]


# Functions in C++

- A function is a block of code which only runs when it is called.
- You can pass data, known as parameters, into a function.
- Functions are used to perform:
	- a specific task through a single call
	- important for reusing code:
		- Define the code once
		- and use it many times

## Create a Function

- C++ provides some pre-defined functions, such as `main()` which is used to execute code
- Can also create own functions to perform certain actions
- To create (often referred to as _declare_) a function:
	- specify the name of the function, followed by parentheses `()`

### Syntax
```cpp
void myFunction()
{  
  // code to be executed  
}  
```

#### Example Explained
- `myFunction()` is the name of the function
- `void` means that the function does not have a return value
- inside the function (the body), add code that defines what the function should do

## Call a Function

- Declared functions are not executed immediately
	- They are "saved for later use"
	- will be executed later, when they are called
	
- To call a function:
	- write the function's name followed by two parentheses `()` and a semicolon `;`
	
- In the following example, `myFunction()` is used to print a text (the action), when it is called

#### Example 1
```cpp
Inside `main`, call `myFunction()`:

// Create a function  
void myFunction() {  
  cout << "I just got executed!";  
}  
  
int main() {  
  myFunction(); // call the function  
  return 0;  
}  
  
// Outputs "I just got executed!"  
```
- A function can be called multiple times:

#### Example 2
```cpp
void myFunction() {  
  cout << "I just got executed!\n";  
}  
  
int main() {  
  myFunction();
  myFunction();
  myFunction();

  return 0;  
}

/* Output:-

I just got executed!  
I just got executed!  
I just got executed!  

*/
```
  

## Function Declaration and Definition

- A C++ function consists of 2 parts:
	- **Declaration:** the return type, the name of the function, and parameters (if any)
	- **Definition:** the body of the function (code to be executed)

```cpp

void myFunction()
{ // **declaration**  
  // the body of the function (**definition**)  

}  
```
- **Note:**
	- If a user-defined function, such as `myFunction()` is declared after the `main()` function, **an error will occur**:
	- unless *forward declared*

#### Example
```cpp
int main()
{  
  myFunction();  
  return 0;  
}  
  
void myFunction()
{  
  cout << "I just got executed!";  
}

/* Error */
```

### Forward Declaration / Function Prototyping

- It is possible to separate the declaration and the definition of the function
	- This is done for code optimisation
- C++ programs can have
	- *function declaration* above `main()`
	- and *function definition* below `main()`

### Advantages of Forward Declaration

- **Modularity** - Separate interface from implementation
- **Compilation** - Faster compilation (only recompile changed .cpp files)
- **Libraries** - Hide implementation details
- **Linking** - Multiple files can use the same function declaration

#### Example
```cpp
// **Function declaration**  
void myFunction();  
  
// The main method  
int main()
{
  myFunction();  // **call** the function  
  return 0;  
}
  
// **Function definition**  
void myFunction()
{
  cout << "I just got executed!";
}
```

## Parameters and Arguments

### Parameters

- Information can be passed to functions as a parameter
- Parameters act as variables inside the function
- Parameters are specified after the function name, inside the parentheses
- You can add as many parameters as you want, just separate them with a comma:

### Syntax

```cpp
void functionName(parameter1, parameter2, parameter3)
{  
  // code to be executed  
}  
```

- The following example has a function that takes a `string` called **fname** as parameter.
- When the function is called, we pass along a first name
	- which is used inside the function to print the full name:

#### Example
```cpp
void myFunction(string fname)
{  
  cout << fname << " Refsnes\n";
}  
  
int main()
{  
  myFunction("Liam");
  myFunction("Jenny");
  myFunction("Anja");
  
  return 0;  
}

// Liam Refsnes
// Jenny Refsnes
// Anja Refsnes
```

### Arguments

- When a **parameter** is passed to the function, it is called an **argument**
- So, from the example above: 
	- `fname` is a **parameter**
	- `Liam`, `Jenny` and `Anja` are **arguments**

# References

