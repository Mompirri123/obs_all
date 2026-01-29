**Created:** 03.12.25, 10:00

**Status:** #atomic

**Hashtags:**

- #cpp
- #templates
- #genericprogramming
- #datastructures
- #atomic

**Links / Tags:**
- **Relevance Links:**
    - Templates in C++ <!-- parent, plain text on purpose -->
    - C++ Data Structures
      
- **Topic Tags:**
    - #genericarray
    - #templatedtypes
    - #nonstlcontainer

# Template Array Example in C++

> Part of templates in C++.

- This is a simple example of a **generic fixed-size array** using templates.
- It uses:
    - a **type template parameter** `T`
    - a **non-type template parameter** `Size` for the array size at compile time

## Simple Template Array

```cpp
template <typename T, int Size = 10>
class Array
{
private:
    T elements[Size];

public:
    void set(int index, T value)
    {
        if (index >= 0 && index < Size)
            elements[index] = value;
    }

    T get(int index) const
    {
        return elements[index];
    }
};
```

- **Usage**
```cpp
int main()
{
    Array<int, 5> intArray;        // integer array of size 5
    Array<double> doubleArray;     // double array of size 10 (default)

    intArray.set(0, 42);
    intArray.set(1, 7);

    doubleArray.set(0, 3.14);
    doubleArray.set(1, 2.71);

    cout << intArray.get(0) << endl;     // 42
    cout << intArray.get(1) << endl;     // 7

    cout << doubleArray.get(0) << endl;  // 3.14
}
```

### Key ideas:
- `T` is a *type parameter*
- `Size` is a *non-type template parameter*
	- must be a compile-time constant
- `Array<int, 5>` and `Array<double, 10>` are different types
- pattern is similar to `std::array<T, N>`, but simplified

# References
