**Created:** 27.11.25, 18:47

**Status:** #atomic 

**Hashtags:**
- #CPP 
- #pointers
- #mathematical-foundations 

**Links / Tags:** 
- **Relevance Links:**

- **Topic Tags:**
	- [[Ternary Operators in C++]]
	- [[Operator precedence in C++]]
	- [[Operator Overloading in C++]]
	  
---

# Operators in C++

> Symbols that perform operations on operands.

- Operators are a **core language feature** in C++
- The define how values are:
	- combined
	- compared
	- modified
	- accessed

## Operator Categories
### 1. Arithmetic Operators
1. Used to perform mathematical calculations
2. `+`, `-`, `*`, `/`, `%`

---
### 2. Relational Operators
1. Used to compare two values
2. Return `true` or `false`
3. `==`, `!=`, `<`, `>`, `<=`, `>=`

---
### 3. Logical Operators
1. Used to combine or negate conditions
2. Work on boolean values
3. `&&`, `||`, `!`

---
### 4. Assignment Operators
1. Used to assign values to variables
2. Can combine assignment with operations
3. `=`, `+=`, `-=`, `*=`, `/=`

---
### 5. Bitwise Operators
1. Perform operations at the bit level
2. Work on binary representation of data
3. `&`, `|`, `^`, `~`, `<<`, `>>`

---
### 6. Unary and Compile-Time Operators
1. Operate on a single operand
2. Used for increment, decrement, negation, etc.
3. `++`, `--`, `+`, `-`, `!`, `~`, `sizeof`

---
### 7. Conditional (Ternary) Operator
1. Shorthand form of if–else
2. `?:`

---
### 8. Pointer Operators
1. Used to work with memory addresses and pointers
2. `&` (address-of), `*` (dereference), `->`

---
### 9. Pointer-to-Member Operators
1. Used to access class members through pointers
2. `.*`, `->*`

---
### 10. Scope Resolution Operator
1. Used to access global variables or class members
2. `::`

---
### 11. Memory Management Operators
1. Used for dynamic memory allocation and deallocation
2. `new`, `delete`, `new[]`, `delete[]`

---
### 12. Type Casting Operators
1. Used to convert one data type into another
2. `static_cast`, `dynamic_cast`, `const_cast`, `reinterpret_cast`

---
### 13. Comma Operator
1. Allows multiple expressions in a single statement
2. Evaluates left to right and returns the last value
3. `,`

---
## Key Sub-topics:
- **Operator Precedence in C++** - Order of evaluation
- **Operator Overloading in C++** - Custom operator behaviour
- **Ternary Operators in C++** - Conditional operator

---
## Basic Examples:
```cpp

int a = 5, b = 3;
int sum = a + b;        // Arithmetic
bool result = (a > b);  // Relational  

a += 2;                 // Assignment

```

---
## The `->` Operator in C++
- The `->` operator is used to access members (like functions or variables) through a [pointer](https://www.w3schools.com/cpp/cpp_pointers.asp).
- It's a shortcut for writing `(*pointer).member`
- If using a pointer to an object
	- `->` is used to access its members.

---
# References
