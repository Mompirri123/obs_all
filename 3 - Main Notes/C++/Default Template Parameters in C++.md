**Created:** 05.12.25, 02:30

**Links / Tags:** 
- **Relevance Links:**

- **Topic Tags:**


# Default Template Parameters in C++

#### Key ideas:

- Default container type can be given for a template type
- Default's reduce noise when a “usual” type is common.
- You can still override the default explicitly when needed.
- You can give template parameters **default types**, making templates easier to use

```cpp
template <typename T = int>
class Container
{
private:
    T value;

public:
    Container(T v) : value(v) {}

    T get() const { return value; }
};
```

```cpp
int main()
{
	Container<> defaultContainer(5);          // uses T = int (default)
    Container<double> doubleContainer(3.14);
    cout << defaultContainer.get() << endl;   // 5
    cout << doubleContainer.get() << endl;    // 3.14
}
```


# References
