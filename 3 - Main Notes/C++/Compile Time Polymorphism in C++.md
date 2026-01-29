**Created:** *<span class ="color-green">10.12.25, 00:47</span>*

**Status:** #atomic

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #oops
    - #polymorphism
- **Topic Tags:**
    - #compiletimepolymorphism
    - #functionoverloading
    - #operatoroverloading
    - #constructoroverloading
    - #templates

**Links / Tags:**
- **Relevance Links:**
    - Polymorphism in C++              <!-- higher parent -->
    - Types of Polymorphism in C++    <!-- parent -->
	    - Compile time polymorphism <!-- Current note--> 
		- Run time polymorphism        <!-- Other type of Polymorphism -->
- **Topic Links:**
    - [[Function Overloading in C++]]
    - [[Operator Overloading in C++]]
    - [[Constructor Overloading in C++]]
    - [[Templates in C++]]

---

# Compile Time Polymorphism in C++

> Polymorphism where the **compiler** decides which function or code to call.

- Decision is made **before** the program runs.
- No virtual dispatch, no runtime lookup.
- Same function name / call site → different function chosen based on **types** and **parameters**.


## Core Idea

- The compiler looks at:
    - function name
    - parameter types
    - template arguments  
  and picks or generates the right implementation.

- Everything is fixed at **compile time**.

---

## Main Mechanisms

### 1. [[Function Overloading in C++|Function Overloading]]

Same function name, different parameter lists.

```cpp
void print(int value)      {}
void print(double value)   {}
void print(const std::string &value) {}
```

- The compiler picks the best match at compile time

see: [[Function Overloading in C++]]

### 2. [[Operator Overloading in C++|Operator Overloading]]

- Redefining operators for user-defined types
- At compile time, the compiler knows `a + b` means `vector::operator+`

see: [[Operator Overloading in C++]]

```cpp
class Vector
{
public:
    int x;
    int y;
    
    Vector(int x, int y) : x(x), y(y) {}
    
    Vector operator+(const Vector &other) const
    {
        return Vector(x + other.x, y + other.y);
    }
};

int main()
{
    Vector a{1, 2};
    Vector b{3, 4};
    Vector c = a + b;   // compiler chooses Vector::operator+
}
```

### 3. [[Constructor Overloading in C++|Constructor Overloading]]

- Multiple constructors for different ways to build the same object
- Compiler chooses which constructor to call based on arguments

see: [[Constructor Overloading in C++]]

```cpp
class Rectangle
{
private:
    int width;
    int height;

public:
    Rectangle()
        : width(0), height(0) {}

    Rectangle(int w, int h)
        : width(w), height(h) {}

    Rectangle(int size)
        : width(size), height(size) {}   // square
};
```


### 4. [[Templates in C++|Templates]] (Generic Programming)

- Templates generate *type -specific* code at compile time

see: [[Templates in C++]]

```cpp
template <typename T>
T add(T a, T b)
{
    return a + b;
}

int main()
{
    int i = add(1, 2);          // generates int version
    double d = add(1.5, 2.5);   // generates double version
}
```
- compiler generates `add<int>` and `add<double>` from same template `T add(T a, T b)`

## Characteristics

- **Resolution**
	- Resolved during compilation
	  
- **Performance**
	- fast (no virtual table lookup)
	  
- **Flexibility**
	- Limited to what is known when compiling
	  
- **Error feedback**
	- errors show up at compile time
	- often long template errors, but still compile-time

## Mental Model

- “The compiler looks at the call and **decides the exact function** before the program starts"
- Overloads and templates are like **multiple versions** of a function that the compiler chooses from

# References 
