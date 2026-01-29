**Created:** 05.12.25, 03:10

**Status:** #atomic

**Hashtags:**

- #cpp    
- #templates    
- #genericprogramming    
- #bestpractices
- #atomic

**Links / Tags:**

- **Relevance Links:**
    
    - Templates in C++ <!-- parent, plain text on purpose -->
      
- **Topic Tags:**
    
    - #dosanddonts
	- #headerorganisation  
    - #templatecleanliness


# Template Best Practices in C++

> Part of templates in C++.

- Good templates are about **clarity**
- not showing off meta-programming skills.

## Do’s ✅

- Use meaningful template parameter names
    
    - `typename ElementType` or `typename Key` / `typename Value`
    - not always just `T`, `U`, `V` everywhere
      
- Keep template interfaces **simple**
    
    - minimal number of template parameters
    - avoid deeply nested templates when not needed
    
- Put template definitions in **header files**
    - the compiler needs to see the full definition to instantiate it
       
- Document requirements on template types
    
    - e.g. “T must be copy able and support `<`”
    - add comments or use concepts later when you learn them
    
- Prefer **type deduction** when it improves readability
    
    - `auto result = getMax(a, b);` instead of repeating long template types

## Don’ts ❌

- Don’t hide templates in `.cpp` files
    - unless you really know what you are doing with explicit instantiations
    
- Don’t create templates only to avoid writing **two simple overloads**
    
- Don’t add template parameters “just in case”
    - each extra parameter makes usage and error messages harder
    
- Don’t rely on clever specialisation when a normal overload or strategy pattern is clearer


## File Organisation Example

```cpp
// MyTemplate.h
template <typename T> class MyTemplate
{
private:
	T value;

public:
	MyTemplate(T v) : value(v) {}
	T get() const { return value;} 
};
```

- For most templates, you do **not** need a separate `MyTemplate.cpp`.
- Implementation stays in the header, so all translation units see it.

## Practical Rules of Thumb

- If your template error messages are unreadable even for you → simplify.
- If you can’t explain in one sentence what a template expects from `T` → redesign or add constraints.
- If a design only ever needs one or two concrete types → templates might be overkill.


# References
