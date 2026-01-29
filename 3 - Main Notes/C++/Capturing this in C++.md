**Created:** *<span class ="color-green">22.12.25, 06:11</span>*

**Note Type:** #atomic

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #lambdafunctions
	- #oop
- **Topic Tags:**
	- #this
	- #captures
	- #objectlifetime

**Links / Tags:** 
- **Relevance Links:**
	- Lambda Capture Modes in C++
	- Classes in C++
- **Topic Links:**
	- [[Const Member Functions in C++]]
	  
---

# Capturing `this` in C++

> how lambdas access the current object inside member functions.

- `this` is a pointer to the **current object**.
- When a lambda is written inside a member function
- it does **not** automatically have access to the object 
- we must decide **how** to capture it.
---

## Capturing `this` explicitly

```cpp
class Counter
{
private:
	int value = 0;
	
public:
	void run()
	{
		auto inc = [this]()
		{
			value++;
		};
		
		inc();
	}
}
```
- `[this]` captures the **pointer** to object
- the lambda works on the **original object**
- any modification affects the real instance
- This is the **classic and most common** form

---
## Lifetime Risk
- Because `[this]` captures a pointer:
	- if lambda is stored
	- and the object is destroyed
	- calling the lambda causes **undefined behaviour**

```cpp
auto f ;
{
	Counter c;
	f = [this]() { value++;} // ❌ dangerous
}
// c is destroyed here
f(); //dangling `this`
```

So:
> Capturing `this` is safe only if the object **outlives the lambda**

---

## Implicit capture of `this`
inside a member function:
```cpp
auto f = [=]()
{
	value++;
};
```

- What happens depends on the C++ version

#### Before C++ 20
- `[this]` -> captures `this` by pointer
- `[=]` -> also captures `this` by pointer
- no way to capture the object by value
	
- So, both `[this]`, `[=]` behave similarly for `this`

#### C++ 20 and later
- C++20 changes the meaning:
```cpp
auto f = [=] ()
{
	std::cout << value <<"\n";
};
```
- `[=]` captures a copy of `*this`
- the lambda owns its own snapshot of the object
- changes do not affect the original object
There is also an explicit form:
```cpp
auto f = [*this]()
{
	std::cout << value << "\n";
};
```
This makes the intent very clear

---

## `[this]` vs `[*this]`
|Capture|What is captured|Effect|
|---|---|---|
|`[this]`|pointer to object|modifies original|
|`[*this]`|copy of object|works on snapshot|
|`[=]` (C++20+)|copy of object|same as `[*this]`|

see: [[capturing using this in const functions]]

# External References
