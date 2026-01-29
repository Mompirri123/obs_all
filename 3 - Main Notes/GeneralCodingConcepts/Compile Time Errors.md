---
aliases:
  - Fixing compile time errors
---
**Created:** *<span class ="color-green">08.12.25, 08:59</span>*

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

# Compile-Time Errors

- **Definition:**
	
- An error that the compiler finds during **before the program runs**
- Stops the build and must be fixed before code can be executed

## Why do compiler-time errors occur?

#### 1. Syntax Errors

- If we broke the **basic rules** of C++.
	- Forgot a semicolon.
	- Forgot a closing brace.
    - Wrote something the compiler cannot read.
    
- Think of it like **bad grammar** in a sentence.

```cpp
// Missing ;
int x = 10
```

#### 2. Type Error

- Trying to use wrong kind of data
	- Assigning to a number
	- `double` where an `int` was expected
- Think of it like putting a US charger in an European power socket

```cpp
int X = "Hello"
```

#### 3. Name Resolution Error

- Calling a undeclared variable
- Incorrect variable name

```cpp
int num;

number = 10; // num is declared not number
```

#### 4. Wrong Function Call

- Too many / Too few arguments in function call
- Wrong types
- Compiler can't find a version that matches what we wrote

```cpp
void empty(int a){}

empty("string"); // wrong parameter type
```

#### 5. Template Issues

- Templates allow any type
	- but not all types support all operations
	- Doing `x++` on a `string`

```cpp
std::string x ="hi";

x++; //strings dont support increment / ++
```

#### 6. 'access' / 'const' Violations

- Trying to change something marked as `const`
- Trying to access a private variable from outside the class

# Fixing compile-time errors

1. **Always read the first error first.**  
    Everything after it might just be noise.
    
2. **Go to the line number.**  
    And check a few lines above it too.
    
3. **Read the message slowly.**  
    The compiler usually hints what it expected.
    
4. **Fix one thing, compile again.**  
    Don’t try to fix 10 things at once.
    
5. **Turn on warnings.**  
    Helps catch stuff before it becomes an error.

# External References
