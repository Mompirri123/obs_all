**Created:** *<span class ="color-green">10.12.25, 05:00</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #typesystem 
	- #cvqualifiers
	- #const
- **Topic Tags:**
    - #qualifiers

**Links / Tags:** 
- **Relevance Links:**
    - C++ Type System %% Parent %%
- **Topic Links:**
    - [[Const in C++]]
    - [[Volatile in C++]]

---

# CV-Qualifiers in C++

- Map note for `const` / `volatile` (CV-qualifiers) in C++.
	- [[Const in C++]]
		- Restricts modification
	- [[Volatile in C++]]
		- Restricts optimisation assumptions

## Why do we need cv qualifiers

- Protect data from accidental modification
- express clear intent (read-only vs mutable)
- enable correct overload resolution
- make APIs safe and predictable
- let the compiler catch bugs early
- support correct object-oriented design
	
- Without CV-Qualifiers, large C++ programs would be fragile and unsafe

see: [[Uses of CV-Qualifiers in C++ ]]

# References