**Created:** *<span class ="color-green">26.12.25, 15:20</span>*

**Note Type:**

**Hashtags:**
- **Relevance Tags:**
	- 
- **Topic Tags:**
	- 

**Links / Tags:** 
- **Relevance Links:**
	- Type conversion and Casting in C++
	- CPP00
- **Topic Links:**
	- [[static cast in C++]]
	- [[signed vs unsigned types in C++]]
	- [[undefined behavior in C++]]
	- [[std::cctype functions]]
---

# cpp00 ex00 - New Concepts

## Why we need casting with `std::toupper`

#### Why does *std::toupper* even cause issues
```cpp
std::cout << std::toupper(str[i])
```

- This would throw an error, because `std::toupper` doesn't return `char`
	- It takes an `int`
	- returns an `int`
- Signature of `std::toupper` is:
```cpp
int  std::toupper(int c);
```
- This means we need to give an `int` and not a `char` to `std::toupper`

#### What happens to `char` when passed to `toupper`
- `char` is promoted to an `int` 
- If `char` is **signed** (common)
- values above 127 can be **negative**
- But, std::toupper expects values representable as `unsigned char` 
	- Which would be in the `int` range of `0` - `255`
- when `char` is promoted to `int`
	- its **signedness is preserved**
    - values may fall outside the valid range for `std::toupper`

#### Why this breaks `std::toupper`
- If a **negative** value is passed (**not EOF**):
	- behaviour is undefined
	- function may
		- crash
		- read invalid memory
		- return wrong result
- This is dangerous because:
	- The bug depends on input
	- It may occur only on some systems
	- very hard to debug this issue

#### Why Casting fixes it
```cpp
static_cast<unsigned char>(ch)
```
- This forces:
	- values into range `0` - `255`
	- no negative numbers
	- `toupper` behaves correctly
- So casting guarantees:
	- value is `0` - `255`
	- promoted to `int` safely
	- meets `toupper`'s requirement

#### Simple synopsis
- `std::toupper` takes an `int`
- but that `int` must hold a value valid for `unsigned char`
- `char` may be **signed** -> **unsafe** to use with `std::toupper`
- casting to `unsigned char` fixes the value
- `toupper` returns `int`
- casting result back to `char` fixes printing
- so we end up with
```cpp
std::cout << static_cast<char>(
    std::toupper(static_cast<unsigned char>(ch))
);
```

#### Why use `static_cast`
##### Why not Implicit Casting
```cpp
std::toupper(ch) // Relies on implicit casting -> ❌
```
- Problem:
	- implicit promotion keeps the **signedness**
	- negative values may still be passed
	- undefined behavior is still possible
	- intent is **not visible**
- So implicit casts are:
	- unsafe
	- unclear
	- easy to misuse

##### Why not C-style casting
```cpp
(char)std::toupper((unsigned char)ch); // ❌
```

- Tries multiple conversions silently
- may perform:
	- `static_cast`
	- `const_cast`
	- `reinterpret_cast`
- hides dangerous conversions
- harder to read and debug
- So, this is generally **discouraged** in C++ though  **not completely wrong**

##### Why not `reinterpret_cast` cast
```cpp
reinterpret_cast<unsigned char&>(ch); // ❌
```
- Reinterprets raw memory
- ignores value semantics
- leads to undefined behavior
- never correct for numeric conversions

##### Why not `const_cast`
```cpp
const_cast<unsigned char>(ch) // Illegal ❌
```
- `const_cast` only removed `const` / `volatile`
- can not change numeric type
- completely unrelated here

##### Why `static_cast` is the right tool
- Performs only **safe, compile-time checked** conversions
- doesn't remove `const`
- doesn't reinterpret memory
- makes the conversion **explicit** and **intentional**
- Clearly stating:
	- "I am converting the *value* in a safe, well-defined manner"
	
- Here, we need to:
	- convert **value types**
	- without changing memory layout
	- without removing `const`
	- without reinterpretation
- This is exactly what `static_cast` is for

##### Synopsis on why use `static_cast`
- We use `static_cast` because it performs an **explicit, safe, compile-time checked value conversion**
- unlike implicit casts or C-style casts
	- which can hide errors or cause undefined behavior
- So Simply said:
	- `char` → may be signed → unsafe
	- `unsigned char` → always 0–255 → safe
	- `std::toupper` works on `int`, but expects unsigned-char values
	- `static_cast` makes both intent and safety explicit

> **Rule of thumb:**  
> Always cast to `unsigned char` before calling character classification or conversion functions


# Internal References



# External References
