**Created:** 27.11.25, 20:50

**Status:**

**Hashtags:**
- #CPP 
- #advancedcpp 
- #OOPS 
- #virtualfunctions 
- #polymorphism 
- #runtimepolymorphism 

**Links / Tags:** 
- **Relevance Links:**

- **Topic Tags:**
	- [[Abstract Classes in C++]]

# Virtual Functions in C++

- **Definition**:
	- Member functions that can be overridden in derived classes to enable runtime polymorphism.
- **Purpose**:
	- Ensure the correct function is called based on the **actual object type** at runtime, not the pointer type.

## 🎯 Core Problem Solved

### Without Virtual (Static Binding)

```cpp
Base* ptr = new Derived();
ptr->show(); // Always calls Base::show() - WRONG!
```

### With Virtual (Dynamic Binding)

```cpp

Base* ptr = new Derived();
ptr->show(); // Calls Derived::show() - CORRECT!
```

## ⚡ Quick Syntax

### Declaration

- `virtual returnType functionName(parameters);`

- The `virtual` keyword is used to "enable runtime Polymorphism"
- Tells the computer
	- Check the actual object type at runtime, not the pointer type
- Must be declared in the **Base Class**

### Override

- `returnType functionName(parameters) override;`

- The `override` keyword is used to explicitly state you're overriding a virtual function
- Provides **compile-time safety**
	- compiler checks if base class actually has this virtual function
- Must be used in the **Derived Class**

## 🚀 Essential Example

```cpp
class Animal
{
public:
    // VIRTUAL: Tells compiler "check actual object type at runtime"
    virtual void speak() { cout << "Animal sound"; }
};

class Dog : public Animal
{
public:
    // OVERRIDE: Explicitly replacing base implementation
    // Compiler verifies Animal::speak() exists and is virtual
    void speak() override { cout << "Woof!"; } 
};

```

### WHAT'S HAPPENING:

 1. Compiler sees `animal->speak()` and checks if `speak()` is *virtual*
 2. Since `speak()` is *virtual*, 
	 - it DOESN'T call `Animal::speak()` immediately
 3. At RUNTIME:
	 - checks what `animal` actually points to (a `Dog` object)
 4. Calls `Dog::speak()` because that's the *actual object type*
 5. Output: 
	 - "Woof!" instead of "Animal sound"

- **WITHOUT VIRTUAL:**
	- Would call `Animal::speak()` regardless of actual object type

### Usage:

- `Animal* animal = new Dog();` 
	- Base pointer pointing to Derived object
	  
- `animal->speak();` 
	- "Woof!" (Runtime decision)

## 🔑 Key Rules

- **`virtual`** in base class
- **`override`** in derived class (optional but recommended)
- **Runtime decision** - based on object type, not pointer type
- **Virtual destructors** needed when deleting through base pointers

## 💀 Virtual Destructors

- ALWAYS use **virtual destructors in base classes**

```cpp
class Base
{
public:
    virtual ~Base() {} // ← Essential!
};

Base* ptr = new Derived();
delete ptr; // Calls both Derived::~Derived() AND Base::~Base()
```

## 📋 Cheat Sheet

| Situation                                        | Result                         |
| ------------------------------------------------ | ------------------------------ |
| `Base* ptr = new Derived(); ptr->func();`        | Calls `Base::func()`           |
| `Base* ptr = new Derived(); ptr->virtualFunc();` | Calls `Derived::virtualFunc()` |
| `delete basePointer` without virtual destructor  | **Memory leak!**               |

## 🎓 Simple Mental Model

- **Virtual**  $\iff$  **"Check what I'm actually pointing to at runtime"**

# References


## Closely Related Notes

- 

### Next:

### Prev:
