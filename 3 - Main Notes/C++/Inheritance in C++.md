**Created:** 27.11.25, 14:06

**Status:** 

**Hashtags:**
- #CPP (Code Language)
- #inheritance  (Primary Related TOpic)
- #OOPS 
- #advancedcpp 
- #multilevelinheritance
- #multipleinheritance
- #multipleinheritance 
- #cppclasses
- #diamondproblem

**Links / Tags:**1 
- **Relevance Links:**

- **Topic Tags:**
	- OOPS in C++
	- [[Access Specifiers in C++]]


# Inheritance

- Allows a class to reuse attributes and methods from another class
	
- Can be grouped into 2 categories
	- **derived class** (child)
		- Class that inherits from another class
	- **base class** (parent)
		- Class being inherited from
	
- To inherit from a class, we use `:`
- The `Child` class inherits the attributes and methods from the `Parent` class

### Example

```cpp
// Parent / Base Class
class Vehicle
{
public:
	string brand = "Ford";
	
	void honk()
	{
		cout << "Tuut, Tuut!" << endl;
	}
};

// Derived Class
Class Car: public Vehicle
{
public:
	string model = "Mustang";
};

int main()
{
	Car myCar;
	
	myCar.honk();
	cout << myCar.brand + " " + myCar.model;
	
	return (0);
}

```


## Advantages of Inheritance

- Useful for code re-usability:
	- reuse attributes and methods of existing class
	- Directly when creating a new Class

# Multi - level Inheritance

- Class can also be derived from a class that's derived from another class
	
- Class A -> Class B (Intermediate - derived from A) -> Class C (Most derived - derived from B)

## Key Points

- Chain of Inheritance:
	- A -> B -> C
- C gets features from both B and A
- If B inherits from A, and C inherits from B, then C also inherits from A
- Constructor calls follow the chain: A-> B -> C
- Access rules apply at each levels

## Inheritance Flow

- Chain of Inheritance:
	- Animal -> Mammal -> Dog
	 
- Dog (Most derived) can access:
	- Animal's protected/public + Mammals protected/public + all its own
	
- Animal can access:
	- all its own
	  
- Main can access:
	- Public members of any class

### Example 

```cpp
// Base Class (Parent)
Class MyClass
{
public:
	Void myFunc()
	{
		cout << "parent class content" << endl;
	}
};

// Derived Class (Child)
class MyChild: public MyClass
{
}

// Derived class from a Derived Class (grandchild)  
class MyGrandChild: public MyChild {  
};

int main()
{
	MyGrandChild myObj;
	myObj.myFunc();
	
	return (0);
}

/* Output:

Parent class content
*/

```


# Multiple Inheritance

- **Definition:**
- Class derived from more than one base class
	- Done using a **comma-separated list**

### Structure

- Class A            Class B
    ↑                        ↑
    └── Class C ──┘

## Syntax

- `class Derivedclass : access-specifier Base1, access-specifier Base2 { // class body }`

#### Example:

```cpp
#include <iostream>
using namespace std;

class Animal
{
public:
	void eat() {cout << "Eating" << endl};
};

class Mammal()
{
public:
	void drink() {cout << "milk" << endl};
};

class Bird()
{
public:
	void fly() {cout << "Flying" << endl};
};

class Bat()
{
public:
	void navigate() {cout << "Using Sonar" << endl};
};

int main()
{
	Bat bat;
	
	bat.eat();
	bat.drink();
	bat.fly();
	bat.navigate();
}

```


## Key characteristic of Multi-Inheritance

- Combines features from multiple classes
- **Constructor Calls:**
	- Base classes constructed in declaration order
- **Access Control:**
	- Can have different access specifiers for each base

## Common Issues

- **Diamond Problem**
	- When 2 base classes inherit from same grandparent
- **Ambiguity**
	- Same function name in multiple classes
- **Complexity**
	- Harder to maintain and understand

### Ambiguity Resolution

- Explicitly specifying which base class is the method from
	 - can resolve the Ambiguity from Multi-Inheritance

```cpp
class ClassA { public: void show() {cout << "from A" << endl};};
class ClassB { public: void show() {cout << "from A" << endl};};
class ClassC : public A, public B {};

ClassC obj;
// Explicitly specify which base class
obj.ClassA::show();
obj.ClassB::show();
```






# References


## Closely Related Notes

### Next:

### Prev:
