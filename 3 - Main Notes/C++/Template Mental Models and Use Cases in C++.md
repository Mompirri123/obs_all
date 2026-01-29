**Created:** 05.12.25, 03:54

**Status:** #atomic

**Hashtags:**

- #cpp
- #templates
- #genericprogramming
- #mentalmodels
- #atomic

**Links / Tags:**

- **Relevance Links:**
    
    - Templates in C++ <!-- parent, plain text on purpose -->
    - [[Generic Types in C++]]
    - [[C++ STL]]
    - [[Template Best Practices in C++]]
    
- **Topic Tags:**
    
    - #templatedesign


# Template Mental Models and Use Cases in C++

> Part of templates in C++.

Instead of memorising syntax, think in **pictures** of how templates behave.

## Mental Model 1: “Code Generator”

- A template is like a **code generator**:
    
    - you write a pattern once
    - the compiler creates concrete versions like `Box<int>`, `Box<string>`, `getMax<double>`
    
- Each `T` you use creates a **separate type or function** at compile time.


## Mental Model 2: “Algorithms, not types”

- Generic programming focuses on **what operations** are needed, not the exact type.
    
    - “I need something I can compare with `<` and copy”
    - not “I need an `int` or a `double`”
    
- This is the idea behind many STL algorithms:
    
    - `std::sort`, `std::find`, `std::accumulate` work with many containers and value types.

## Mental Model 3: “Lego blocks”

- Templates + STL = **Lego blocks** for types and algorithms.
    
- You combine:
    
    - containers: `std::vector<T>`, `std::array<T, N>`, `std::list<T>`
        
    - algorithms: `std::sort`, `std::transform`, `std::for_each`
        
    - adaptors/wrappers: `std::optional<T>`, `std::variant<T...>`, `std::pair<T, U>`
    
- Most of the time you don’t write advanced templates, you **reuse** existing ones.

## Typical Use Cases for Templates

### 1. Containers

- Need to store “a collection of T”:
    
    - `std::vector<T>`, `std::list<T>`, `std::array<T, N>`
    
- Your own custom containers (like `Array<T, Size>`) follow the same pattern.


### 2. Algorithms

- One algorithm, many element/container types:
    
    - sorting, searching, filtering, transforming
    
- Templates let you write it once and reuse it everywhere.


### 3. Smart Pointers and Resource Wrappers

- `std::unique_ptr<T>`, `std::shared_ptr<T>`:
    
    - same ownership logic for **any** T
    
- RAII wrappers (lock guards, file handles, etc.) often use templates.


### 4. Math and Utility Code

- `Vector2<T>`, `Matrix<T>` for numeric code
    
- Utility helpers like `clamp<T>`, `lerp<T>`, etc.
  

## When not to use templates

- When behaviour is radically different per type:
    
    - sometimes simple overloading or runtime polymorphism (virtual functions) is clearer
    
- When you only ever need one or two concrete types:
    
    - `int` and `double` only? two overloads might be simpler than a template explosion
    
- When it makes error messages unreadable and debugging painful.


## Quick Checklist

- Before writing a template, ask:

	- Is the **algorithm the same** for all types I care about?
	- Can I describe in one line what I expect from the template `T`?
	- Will this be reused often enough to justify a template?

If “yes” to these → a template is probably a good fit.

# References
