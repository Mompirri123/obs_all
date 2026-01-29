**Created:** 10.12.25, 14:46*

**Note Type:** #atomic

**Hashtags:**

- **Relevance Tags:**
    - #cpp 
    - #cvqualifiers
- **Topic Tags:**
    - #const
    - #immutability

**Links / Tags:**

- **Relevance Links:**
    - Const in C++
- **Topic Links:**
    - [[Const Placement Rules in C++]]
    - [[Const Member Functions in C++]]
    - [[Const Objects in C++]]
    - [[Pointer and Reference Types in C++]]

---
# Const Keyword in C++

- `const` is a promise you make to the compiler
- It means: **this thing must not be changed after this point**
- The compiler then _enforces_ that promise and raises an error if your code tries to break it.

---

## What `const` actually protects

- A common mistake is thinking `const` always protects a variable
	- That is **not true**
- `const` qualifies a **type**. it makes _that part_ of the type read-only
	- to decode it, use the **right-left rule** (read from the name outward)
	- `const`  **protects whatever is immediately to its right** (or to its left if nothing is on the right)
- “immediately to the right” is a **helpful heuristic**, not a law

What gets protected depends on where `const` is written

The details of this rule are explained in:
- [[Const Placement Rules in C++]]

---

# what kinds of things can be protected using `const`

## Const on variables

```
const int x = 9;
```

Meaning:

- `x` is initialised once
- any later attempt to modify `x` is rejected

```
x = 10;   // ❌ compiler error
```

Use this when a value should never change after creation

---

## Const on function parameters

```
void foo(const std::string& s);
```

Meaning:

- the function promises not to modify `s`
- the caller’s data is protected

Why this matters:

- avoids accidental changes
- allows passing temporaries and const objects
- improves readability and trust in the API

---

## Const with pointers

```
int* const p;        // pointer cannot change
const int* p;        // value cannot change
const int* const p;  // neither can change
```

- These look similar but mean **very different things**
- Instead of memorising, always ask:
	- what is being protected from modification?

Full explanation lives in:

- [[Const Placement Rules in C++]]

---

## Const with references

```
const int& ref = x;
```

Meaning:

- `ref` cannot be used to modify `x`
- but `x` itself may still change through other means

This is heavily used for:

- function parameters
- avoiding copies
- read-only access

---

## Const member functions

```
int getValue() const;
```

Meaning:

- this function promises not to modify the object
- inside the function, `this` is treated as a pointer to const

Why it matters:

- allows the function to be called on const objects
- enables const-correct APIs

Details live in:

- Const Member Functions in C++

---

## Const objects

```
const MyClass obj;
```

Meaning:

- `obj` itself cannot be modified
- only `const` member functions may be called

This enforces read-only usage at the object level.

See:
- [[Const Objects in C++]]

---

## Key mental model

Every time you see `const`, ask:

> **What exactly is not allowed to change?**

- a value?
- a pointed value?
- a pointer?
- an object?
- a function’s behaviour?
- The answer depends on **placement**, not on the keyword itself.

---

# Internal References

# External References