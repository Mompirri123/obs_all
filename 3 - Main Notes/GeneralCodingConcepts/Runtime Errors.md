**Created:** *<span class ="color-green">08.12.25, 09:43</span>*

**Note Type:**

**Hashtags:**
- **Relevance Tags:**
	- 
- **Topic Tags:**
	- 

**Links / Tags:** 
- **Relevance Links:**
	- Types of Errors in C++
	
- **Topic Links:**
	- 

---

# Runtime Errors

## Definition

A run-time error is a problem that happens **while the program is running**, after the code has already compiled successfully.

---

## What causes run-time errors?

These come from situations the compiler can’t predict.  
The code is valid, but something goes wrong when it actually runs.

---

### 1. Invalid memory access

- Using a pointer that points nowhere (null pointer).
- Going out of array bounds.
- Using deleted or uninitialised memory.

These usually crash the program.

---

### 2. Operations that are not allowed

- Dividing by zero.
- Taking modulo of zero.
- Overflow/underflow during calculations.

The compiler cannot always catch these earlier.

---

### 3. Exceptions that are not handled

- If a function throws an exception and nothing catches it → program ends.
```cpp
throw std::runtime_error("fail");
```

### 4. Bad input or unexpected values

- Program expects a valid input, but gets something else.	
	- Converting invalid text to a number.
	- Opening a file that doesn't exist.
	- Network request failing.

These do not break compilation but break execution.

---

### 5. Logic that depends on real conditions

- Anything that needs real-time info:
	- File system
	- User input
	- Network
	- Hardware  
- If those conditions fail → run-time error.

## Examples:

```cpp
int x = 0;
int y= 10/x; // division by zero

int arr[3];
arr[10] = 5; //out of bounds access

int *p = nullptr;
*p =5; // invalid memory write
```


## How to deal with run-time errors

- Add proper checks (e.g., `if (ptr != nullptr)`, `if (x != 0)`).
- Validate all input.
- Use exceptions and handle them correctly.
- Test with edge cases.
- Use tools like sanitizers (`-fsanitize=address`, etc.) to find memory bugs.

# Internal References



# External References
