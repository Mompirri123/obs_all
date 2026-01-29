**Created:** 02.12.25, 10:121

**Status:** #

**Hashtags:**
- #errorhandling 
- #compiletimeerrors
- #runtimeerrors
- #danglingpointers
- #logicalerrors

# Types of Errors in C++

- Errors are classified into types depending on why and where the Error occurs

## 1. [[Compile Time Errors]]

- Errors preventing program from compiling
- Common Compile Time Errors:
	- Missing semicolon
	- Using undeclared variables
	- Mismatched types

#### Example: Compile Time Error

```cpp
int x = 5 // Missing semi-colon
std::cout << y; // Undeclared y
int x = "Hello"; // Type Mismatch
```

## 2. [[Runtime Errors]]

- Runtime errors occur when the program compiles but crashes or behaves unexpectedly
- Common Runtime Errors:
	- Dividing by zero
	- Accessing out-of-bounds array elements
		- Also called as [[Access Violation in C++|Access Violation]]
	- Using deleted memory (dangling pointer)

#### Examples: Common Runtime Errors

##### Un-computable logic
```cpp
int a = 10;
int b = 0;

int result = a/b; // Throws an error
std::cout << result;
```

##### Accessing out-of-bounds elements
```cpp
int members[3] = {1, 2, 3};
std::cout << nums[3]; // no element on index 3
```

##### Using Deleted memory
```cpp
int* ptr = new int(10);
delete ptr;
std::cout << *ptr; // danglling pointer
```

## 3. Linktime Errors

- Occur during the **linking phase** after compilation
- When object files are combined into executable
- Common Link-time Errors:
	- Missing function definitions
	- Multiple definitions of same function
	- Library linking issues
	- Incorrect include paths

#### Example: Linktime Error

```cpp
// File1.cpp
void functionA(); // Declaration only

// File2.cpp
int main()
{
	functionA(); // ERROR: functionA never defined
}
```

## 4. Logical Errors

- Program runs but produces wrong results
- Hardest to detect
- Program compiles and runs
- incorrect output / behaviour
- Algorithmic mistakes

```cpp
int average(int a, int b)
{
	return a + b / 2;
	// WRONG for average we needed parantheses
	// corret => (a + b) / 2;
}

// test:
std::cout << average(10, 20); // Returns 20 instead of 15
```

# Quick Reference Table

| Error Type    | When Detected | Example            | Fix Approach             |
| ------------- | ------------- | ------------------ | ------------------------ |
| **Syntax**    | Compile-time  | Missing `;`        | Check compiler errors    |
| **Type**      | Compile-time  | `int x = "text"`   | Fix type declarations    |
| **Link-time** | Linking       | Undefined function | Check includes/libraries |
| **Runtime**   | Execution     | Divide by zero     | Add validation checks    |
| **Logical**   | Testing       | Wrong calculation  | Debug algorithm logic    |
# References


## Closely Related Notes

- 

### Next:

### Prev:
