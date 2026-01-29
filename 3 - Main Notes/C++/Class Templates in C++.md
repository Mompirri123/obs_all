**Created:** 03.12.25, 07:50

**Status:** #atomic

**Hashtags:**

- #cpp
- #templates    
- #genericprogramming    
- #classes    
- #atomic    

**Links / Tags:**

- **Relevance Links:**
    - Templates in C++       <!-- parent -->
    - [[User-Defined Types in C++]]
    
- **Topic Tags:**
    - #classtemplates
    - #genericcontainers
    - #multipletypeparameters

# Class Templates in C++

> Part of templates in C++.

- Class templates let you define a **family of classes**
	- that work with different types without duplicating code.
	  
- The compiler generates a concrete class for each type you instantiate.

## Basic Syntax

```cpp
template <typename T> class ClassName
{ 
	// Members using type T
};
```

#### Key ideas:
- `template <typename T>` introduces a template parameter `T`
- `ClassName<T>` becomes a real type once you substitute a concrete type

## Simple Container Example: Box

```cpp
template <typename T> class Box
{
private:
	T content;
public:
	Box(T value) : content(value) {}
	
	T getValue() const { return content; }
	void setValue(T newValue) { content = newValue;}
}; 

// Usage:
int main()
{
	Box<int> intBox(42);                // box of int
	Box<string> stringBox("Hello");     // box of string
	Box<double> doubleBox(3.14);        // box of double
	
	cout << intBox.getValue() << endl;
	cout << stringBox.getValue() << endl;
}
```

#### Key ideas:

- `Box<int>` and `Box<string>` are different types generated from the same template.
- All logic is written once, reused for many types.

## Multiple Type Parameters: Pair

```cpp
template <typename Key, typename Value>

class Pair
{
public:
	Key first;
	Value second;
	Pair(Key k, Value v) : first(k), second(v) {}
	
	void display() const
	{
		cout << first << ": " << second << endl;
	}
};


int main
{
	Pair<string, int> person("John", 25);    // name + age
	Pair<int, double> measurement(1, 99.5);  // id + value
	person.display();
	measurement.display();
}
```

#### Key ideas:
- You can have more than one template parameter (`Key`, `Value`)
- This pattern is the basis of many STL types like `std::pair` and `std::map` (key/value)

## When To Use Class Templates

- Use class templates when:
	- You build **reusable containers or wrappers** (like `Box`, arrays, smart pointers)
	- The **behaviour is the same** for many different element types
	- You design **data structures** where the algorithm does not depend on the concrete type of elements