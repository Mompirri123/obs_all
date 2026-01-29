**Created:** 03.12.25, 01:56

**Status:** #atomic

**Hashtags:**
- #cpp
- #datatypes
- #text

**Links / Tags:** 
- **Relevance Links:**
  - [[C++ STL]]

- **Topic Tags:**
  - #characters
  - #strings
  - #unicode
  - #stdstring

# Character and String Types in C++

> Part of C++ data types.

C++ has several character types for different encodings, plus string container types.

## Character types

- `char` – smallest addressable unit of the execution character set  
- `signed char` / `unsigned char` – char with explicit sign  
- `wchar_t` – wide character (implementation-defined size)  
- `char16_t` – [[UTF-16]] code units  
- `char32_t` – [[UTF-32]] code units

## String types

- `std::string` – owning dynamic string of `char`  
- `std::wstring` – wide string (`wchar_t`)  
- `std::u16string`, `std::u32string` – [[UTF-16]]/[[UTF-32]] strings  
- `std::string_view` – non-owning string view (pointer + length)

## Key ideas

- Use `std::string` for most text in modern C++.
- Use `std::string_view` for read-only views without allocations.
- For serious Unicode, combine `char8_t`/`char16_t`/`char32_t` with libraries that understand encodings.

# References

- 
