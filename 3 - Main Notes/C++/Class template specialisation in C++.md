**Created:** 05.12.25, 02:20

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


# Class template specialisation in C++

#### Key ideas:

- Generic template covers **most** types.
- Specialisation is used when a **particular type** needs different behaviour.
- Overusing specialisation can make code harder to follow, so use it when it really adds clarity.
  
```cpp
// Generic version
template <typename T>
class Printer
{
public:
    void print(T value)
    {
        cout << "Value: " << value << endl;
    }
};

// Specialised version for string
template <>
class Printer<string>
{
public:
    void print(string value)
    {
        cout << "String: \"" << value << "\"" << endl;
    }
};
```

```cpp
int main()
{
    Printer<int> intPrinter;
    Printer<string> stringPrinter;
    intPrinter.print(42);           // uses generic version
    stringPrinter.print("hello");   // uses specialised version
}
```

## [[Partial Specialisation in C++|Partial Specialisation]] (class templates only)

- You can also specialise **some** template parameters and keep others generic

# References
