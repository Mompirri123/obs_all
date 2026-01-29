**Created:** 05.12.25, 02:26

**Links / Tags:** 
**Links / Tags:** 
- **Relevance Links:**
	- #cpp
	- #templates
	- #genericprogramming
	- #advancedcpp  
	- #atomic 

- **Topic Tags:**
	- #classtemplates 
	- #classtemplatespecialisation
	- #partialspecialisation

# Partial Specialisation in C++

- You can also specialise **some** template parameters and keep others generic
	- (class templates only)
  
```cpp
// Generic
template <typename T, typename U>
class PairWrapper
{
    // generic implementation
};

// Partial specialisation when both types are the same
template <typename T>
class PairWrapper<T, T>
{
    // specialised implementation for (T, T)
};
```

# References
