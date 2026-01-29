**Created:** *<span class ="color-green">10.12.25, 01:36</span>*

**Note Type:** #atomic

**Hashtags:**
- **Relevance Tags:**
    - #cpp
    - #oops
    - #polymorphism
- **Topic Tags:**
    - #runtimepolymorphism
    - #virtualfunctions
    - #dynamicdispatch
    - #abstractclasses

**Links / Tags:**
- **Relevance Links:**
    - Polymorphism in C++              <!-- parent, plain text -->
    - Types of Polymorphism in C++    <!-- parent, plain text -->
- **Topic Links:**
    - [[Virtual Functions in C++]]
    - [[Abstract Classes in C++]]
    - [[Inheritance in C++]]

---

# Runtime Polymorphism in C++

> Polymorphism where the **program decides at runtime** which function to call  
>  - based on the **actual object type**
>  - not just the pointer/reference type.

- Uses:
    - `virtual` functions
    - base-class pointers / references
    - often abstract classes as interfaces

## Core Idea

- You talk to objects through a **base class interface**:
    - `Base *ptr` or `Base &ref`
- At runtime,
	- C++ checks **which derived type** the object really is.
- It then calls the **correct override** via the virtual function mechanism.

See: Virtual Functions in C++

## Basic Example with Virtual Functions

```cpp
#include <iostream>

class Animal
{
public:
    virtual void speak()
    {
        std::cout << "Animal speaks\n";
    }
    
    virtual ~Animal() = default;
};

class Dog : public Animal
{
public:
    void speak() override
    {
        std::cout << "Woof! Woof!\n";
    }
};

class Cat : public Animal
{
public:
    void speak() override
    {
        std::cout << "Meow!\n";
    }
};

int main()
{
    Animal *animal = nullptr;
    
    Dog dog;
    Cat cat;
    
    animal = &dog;
    animal->speak();   // prints "Woof! Woof!"
    
    animal = &cat;
    animal->speak();   // prints "Meow!"
    
    // without 'virtual', both calls would print "Animal speaks"
}
```

- `animal` is an `Animal *`, but:
    - when it points to `Dog`, `Dog::speak` runs
    - when it points to `Cat`, `Cat::speak` runs 
      
- Decision is made **at runtime** using the virtual table (vtable).

## Abstract Classes and Interfaces

- Often the base class is **abstract** (pure virtual)
See: Abstract Classes in C++

```cpp
class Shape
{
public:
    virtual double area() const = 0;   // pure virtual ⇒ abstract class
    virtual ~Shape() = default;
};

class Rectangle : public Shape
{
private:
    double width;
    double height;

public:
    Rectangle(double w, double h)
        : width(w), height(h) {}

    double area() const override
    {
        return width * height;
    }
};

```
- `Shape` cannot be instantiated (abstract)
- Any non-abstract derived class (like `Rectangle`) **must** implement `area`
- You can store different shapes via `Shape *` or `Shape &` and call `area()` polymorphically

## Typical Use Cases

- GUI frameworks: `Widget *w; w->draw();`
- Game entities: `Entity *e; e->update();`
- Database / driver interfaces: `Database *db; db->query(...);`
	
- You write code against the **interface**, not the concrete type.

## Characteristics

- **Resolution**:
	- at runtime 
	  
- **Performance**:
	- one extra indirection (virtual call), slightly slower
    
- **Flexibility**:
    - very flexible: new derived classes can be added without changing caller code
    - as long as they follow the same interface

## Mental Model

- “I only know that this is a **Shape**.”
- “At runtime,
	- the system looks at the **real object** (Rectangle, Circle, etc.)  
	- calls the correct override via `virtual`.

# Internal References

# External References
