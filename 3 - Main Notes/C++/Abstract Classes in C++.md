**Created:** 27.11.25, 20:50

**Status:** 

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #advancedcpp
    - #cppclasses
      
- **Topic Tags:**
    - #abstractclasses
    - #purevirtualfunctions
    - #virtualfunctions
    - #polymorphism  

**Links / Tags:**

- **Relevance Links:**
    - Classes and Objects in C++ <!-- parent, plain text -->
    - Polymorphism in C++ <!-- parent, plain text -->
    - OOPS in C++ <!-- higher-level OOP map, plain text -->
    - Inheritance in C++
      
- **Topic Links:**
    - [[Virtual Functions in C++]]
    - [[Pure Virtual Functions in C++]]
    - [[Interfaces in C++]] <!-- if you make this note later -->

# Abstract Classes

- **Definition**
	- Classes that **cannot be instantiated**
	- serve as interfaces for derived classes
	
- **Purpose**
	- Force derived classes to implement specific functionality

## Properties of Abstract Class

- Requires **PURE virtual functions** (`= 0`)
- **No implementation** for pure Virtual Functions in the base class
- **Must be overridden** in concrete derived classes
- Used as **interfaces/blueprints**

## Pure Virtual Function

```cpp
virtual void function() = 0; // Makes class abstract
```

## Key Points

- `= 0` $\Rightarrow$ Pure Virtual (no Implementation in this class)
- Class becomes abstract
- cannot create objects of abstract class
- Derived classes must implement all pure virtual functions to become concrete

## Examples:

#### Basic Abstract Class

```cpp
class Shape // Abstract Class
{
public:
	virtual void draw() = 0; // pure virtual
};

// Shape s; // ERROR: Cannot instantiate
```

#### Complete Implementation

```cpp
class Animal
{
public:
	virtual void speak = 0; // Must Implement
	virtual void sleep(){}  // Optional override
	void breathe() {}       // Inherited as-is
};

class Dog : public Animal
{
public:
	// MUST implement the pure virtual funtion speak=0
    void speak() override {cout << "Woof!" << endl;}
};

int main()
{
	// Animal a;  // Still abstract
	Dog dog;  // OK - implements all pure virtual functions
	
	dog.speak() // Output: "Woof!"
}
```

- The `virtual void speak = 0;` says that `speak` must be implemented in the derived class
- Then in the derived class `Dog`
	- The `override` keyword is used to modify `speak`
		- making it clear we are overriding a virtual function
	- In this particular example, `void speak() override {cout << "Woof!" << endl;}`

#### Practical Usage

```cpp
class Database
{  // Abstract interface
public:
    virtual void connect() = 0;
    virtual void query(string sql) = 0;
    virtual ~Database() = default;
};

// As `query` is now a pure virtual function
// we must override it all the classes derived from Database

class MySQL : public Database
{
public:
    void connect() override { /* MySQL connection */ }
    void query(string sql) override { /* MySQL query */ }
};

class PostgreSQL : public Database
{
public:
    void connect() override { /* PostgreSQL connection */ }
    void query(string sql) override { /* PostgreSQL query */ }
};
```

- The **Base Class** `Database` is used as
	- an **Interface** / **Contract** that all database types must follow
- Then the **Child Classes** `MySQL` and `PostgreSQL` provide specific implementations for each database system

##### 🎯 How It Works in Practice

```cpp
// Usage in application:
void useDatabase(Database* db) {  // Accepts ANY Database type
    db->connect();    // Calls MySQL::connect() or PostgreSQL::connect()
    db->query("SELECT * FROM users");  // Calls appropriate query implementation
}
// Can switch between databases easily:
useDatabase(new MySQL());      // Uses MySQL
useDatabase(new PostgreSQL()); // Uses PostgreSQL - same interface!
```

- **Benefits:**
	- Write code once that works with any database
	- Easy to add new database types later
	- Consistent API across all database implementations

**This is polymorphism in action!** 🔥

# References

- [Pure Virtual Functions and Abstract Classes in C++ - Geeks for Geeks](https://www.geeksforgeeks.org/cpp/pure-virtual-functions-and-abstract-classes/)
