**Created:** 03.12.25, 10:03

**Status:** #atomic

**Hashtags:**

- #cpp
- #templates
- #genericprogramming
- #advancedcpp  
- #atomic 

**Links / Tags:**

- **Relevance Links:**
    - Templates in C++ <!-- parent, plain text on purpose -->
    - [[Function Templates in C++]]
    - [[Class Templates in C++]]
    - Template and Generic Types in C++
      
- **Topic Tags:**
    
    - #templatespecialisation
    - #defaulttemplateparameters
    - #advancedtemplates


# Advanced Template Features in C++

> Part of templates in C++.

- Advanced template features let you **tune behaviour** for specific types.
    
- Common tools:
    
    - **template specialisation**
    - **default template parameters**


## Template Specialisation

- Template specialisation lets you override the generic template for a specific type.

### [[Class template specialisation in C++| Class Template Specialisation]]

### Key ideas:

- Generic template covers **most** types.
- Specialisation is used when a **particular type** needs different behaviour.
- Overusing specialisation can make code harder to follow, so use it when it really adds clarity.
  
## [[Default Template Parameters in C++| Default Template Parameters]]

### Key ideas:

- Defaults reduce noise when a “usual” type is common.
- You can still override the default explicitly when needed.

### Syntax for Default Template Parameters

```cpp
template <typename T = int>
```

#### example

## When to reach for these features

- Use **specialisation** when one or a few types need **different behaviour**.
- Use **partial specialisation** for a whole family of types with a shared pattern.
- Use **default parameters** when there is a clear “most common” type and you want a simple syntax.

# References
