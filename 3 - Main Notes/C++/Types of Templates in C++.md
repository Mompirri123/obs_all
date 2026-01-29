**Created:** 06.12.25, 02:53

**Status:** #map

**Hashtags:**
- #cpp
- #templates
- #genericprogramming
- #map
- #overview

**Links / Tags:**
- **Relevance Links:**
  - Templates in C++          <!-- parent, plain text on purpose -->
  - [[Function Templates in C++]]
  - [[Class Templates in C++]]
  - [[Advanced Template Features in C++]]
  - [[Template Array Example in C++]]
- **Topic Tags:**
  - #templatetypes
  - #functiontemplates
  - #classtemplates
  - #advancedtemplates

# Types of Templates in C++

> Sub-map under templates in C++.

This note groups the **main kinds of templates** you’re using and their examples.

## Function Templates

- Note: [[Function Templates in C++]]
- One function definition that works for many types.
- Example ideas:
	- `getMax<T>` for different numeric or string types
	- algorithms that don’t care about concrete types, only operations like `<` or `+`

## Class Templates

- Note: [[Class Templates in C++]]
- Define a **family of classes** parameterised by types.
- Example ideas:
	- `Box<T>` wrapper
	- `Pair<Key, Value>` for key–value pairs

## Advanced Template Features

- Note: [[Advanced Template Features in C++]]
- Special tools to fine-tune template behaviour:
	- template specialisation
	- partial specialisation
	- default template parameters

## Template Array Example

- Note: [[Template Array Example in C++]]
- Concrete example of:
	- type template parameter `T`
	- non-type template parameter `Size`
- Very similar mental model to `std::array<T, N>`.

# References

- 
