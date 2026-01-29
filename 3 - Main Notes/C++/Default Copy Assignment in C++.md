**Created:** *<span class ="color-green">09.12.25, 10:14</span>*

**Note Type:** #atomic

**Hashtags:**

- **Relevance Tags:**
    
    - #cpp
    - #oop
    - #classdesign
       
- **Topic Tags:**
    - #copyassignmentoperator 
    - #defaulted        
    - #ruleofzero


**Links / Tags:**

- **Relevance Links:**
    
    - Copy Assignment Operator in C++ <!-- parent, plain text -->
    - Canonical Forms in C++ <!-- lifetime cluster, plain text -->
       
- **Topic Links:**
    
    - [[Constructors in C++]] 
    - [[Destructors in C++]]
    - Rules of Three Five and Zero in C++
    
---

# Default Copy Assignment in C++

- When we don't declare a copy assignment operator for a class
- The compiler **generates one automatically**

## Implementation of Default Copy Assignment

```cpp
class Point
{
public:
	int  x;
	int  y;
}

int main()
{
	Point a{1, 2};
	Point b{3, 4};
	b = a;
	std::cout << b.x << ", " << b.y; 
}
```

- Default copy assignment does a **member wise copy**
	- Assigns each non-static data member from `other` to `this` class
	- for `Point` class example
		- `x` from `other.x`
		- `y` from `other.y`

## Syntax

- The syntax is same for both user-defined and default Copy assignment operator

```cpp
Point &Point::operator=(const Point &other)
```
- `point&`
	- returns `this` so we can chain `a = b = c`
	  
- `operator=`
	- Called when we write `b = a;`
	  
- `const Point& other`
	- Source object, passed by const reference
	  
- So we can just write `b = a` while calling the `operator=`

## Using '=default' (C++ 11)

- We can explicitly ask for the `default` behaviour
```cpp
class Point
{
public:
	int  x;
	int  y;
	
	Point &operator=(const Point&) = default;
};
```

### Use-cases for'default' call

- Document that copying is allowed
- We defined other special members (like a custom destructor) but still need default copy assignment

## When to use default copy assignment

- The class doesn't manage raw resources (no `new` or `delete`) 
- All members are:
	- value types (`int`, `double`, `enums` etc.,) 
	   or
	- RAII types (`std::string`, `std::vector`, smart pointers, etc;)
	  
- This is the **Rule of Zero** situation
	- Let member objects handle copying and destruction
	- no need for a custom `operator=`

# Internal References

- Rules of Three Five and Zero in C++

# External References
