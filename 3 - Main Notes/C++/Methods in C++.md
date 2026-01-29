**Created:** *27.11.25, 01:26*

**Status:** 

**Hashtags:**
- #CPP 
- #functions 

**Links / Tags:** 
- **Relevance Links:**
	- 

- **Topic Tags:**
	- 


# Methods

## Explanation on Methods

- **Methods** or **member functions** are functions that belong to a class
- They define the behaviours or actions that objects of the class can perform
   

## Two Ways to Define Class Methods

### 1. Inside Class Definition (Inline Methods)

- Method is 
	- *declared and defined* directly *inside the class declaration*
    
- Suitable for short, simple functions

#### Example:

```cpp
class MyClass
{
public:
	// Method declared & defined Inside the class
	void myMethod()
	{
		cout << "Hello World!";
	}
};

```

### 2. Outside Class Definition

- Method is
	- declared inside the class
	- defined outside
	  
- Uses the **Scope resolution Operator** `::`
	
- Better for Organisation, especially in large programs

#### Example:

```cpp

class MyClass
{
public:
	// Method declared INSIDE class
	void myMethod();
};

// Method definition OUTSIDE class
void MyClass::myMethod()
{
	cout << "Hello World:";
}
```


### Complete Examples of both scenarios

#### Inside class Definition:
```cpp
#include <iostream>
using namespace std;

class Calculator
{
public:
	// Methods Declared and Defined inside class
	int add(int a, int b)
	{
		return (a + b);
	}
	
	int multiply(int a, int b)
	{
		return (a * b);
	}
}

int main()
{
	Calculator calc;
	cout << calc.add(5, 3);
	cout << calc.multiply(4, 2);
	return 0;
}

```

#### Outside class Definition

```cpp

#include <iostream>
using namespace std;

class Calculator
{
private:
	// only Declarations inside the class
	int add(int a, int b);
	int multiply(int a, int b);
}

int Calculator::add(int a, int b)
{
	return (a + b);
}

int Calculator::multiply(int a, int b)
{
	return (a * b);
}

int main()
{
	MyClass myCalcy;
	
	cout << myCalcy.add(2, 10) << endl;      // Output : "20"
	cout << myCalcy.multiply(4, 5) << endl;  // Output : "20"
	return 0;
}
```


## Example Problem

- **QUESTION:**
	- Create a class `Dog` with a method `bark()` that prints `"Woof!"`.
	- Then call that method from `main()` using an object of the class.

- **Solution:**
```cpp

#include <iostream>
using namespace std;

class Dog
{
public:
	void bark();
};

Dog::bark()
{
	cout << "woof!" << endl;
}

int main()
{
	Dog myDog;
	
	myDog.bark();
	return 0;
}

```

# References


## Closely Related Notes

- Classes and Objects in C++
