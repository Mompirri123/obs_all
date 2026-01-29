**Created:** *<span class ="color-green">21.12.25, 23:27</span>*

**Note Type:**

**Hashtags:**
- **Relevance Tags:**
	- #cpp 
	- #functions 
	- #lambdafunctions 
- **Topic Tags:**
	- #captures 
	- #capturemodes
	- #moderncpp 

**Links / Tags:** 
- **Relevance Links:**
	- Lambda Functions in C++
- **Topic Links:**
	-  [[Capturing this in C++]]
	  
---

# Lambda Capture Modes in C++

> How a lambda sees and uses variables from the surrounding scope

- Capture list is the `[...]` part at start of Lambda

```cpp
[capture-list] (parameters) {/* body */};
```

- It tells the Lambda **which variables** it is allowed to use and **how** it gets them 
	- by value or by reference


## 1. Capture nothing - `[]`
- `[]` -> captures nothing

```cpp
int x = 9;

auto f = [] ()
{
	// Cannot use x here - not captured
};
```
- safest form
- lambda can only see:
	- its own **parameters**
	- global/static variables
- we use this when the lambda does not need outer variables


## 2. Capture by value - `[=]`, `[...]`

#### `[=]` -> capture used variables by value
```cpp
int a = 1;
int b = 2;

auto f = [=] ()
{
	//
	// a and b are copied into the lambda
	std::cout << a + b << "\n";
};
```
- each used outer variable is **copied**
- inside lambda you see **const-like copy**
	- by default

#### `[...]` -> capture a specific variables by value
```cpp
int x = 10;
int y = 20;

auto f = [x]()
{
	std::cout << x << "\n";
	// y not captured -> cannot use y
};
```
- only `x` is captured
- useful when you want to be explicit and avoid surprises

## 3. Capture by reference - `[&]`, `[&x, &y, ...]`

#### `[&]` -> Capture used variables by reference
```cpp
int counter = 0;

auto inc = [&] ()
{
	++counter; //modifies original counter
};

inc();
std::cout << counter; // 1
```
- outer variables are not copied
- lambda works on **original** values

#### `[&x, &y, ...]` -> Capture given variables by reference
```cpp
int x = 10;
int y = 20;

auto f = [&x] ()
{
	x += 5;   // modifies original x
	// y not captured -> can't use y
}
```

- Capturing by reference is a bit dangerous
	- the lambda **outlives** the variables (e.g. stored and used later)
	- you then have **dangling reference**


## 4. Mixed Captures - `[x, &y, ...]`

- We can mix value and reference captures:
```cpp
int x = 10;
int y = 20;

auto f = [x, &y] ()
{
	// x is a copy
	// y is a reference
	
	std::cout << x << ", " << y << "\n";
	y++;
};
```
- `x` stays same outside, `y` changes


## 5. Initialiser Captures (C++14) - `[z = x + y]`

- We can create a new captured variable:

```cpp
int x = 2;
int y = 3;

auto f = [z = x + y] ()
{
	std::cout << z << "\n";
};
```
- `z` is a new variable inside the lambda
- initialised with `x + y` at the moment of capture

##### Useful to move-only things
```cpp
auto ptr = std::make_unique<int>(42);

auto f = [p = std::move(ptr)]()
{
	std::cout << *p << "\n";
};
```


## 6. Capturing `this`

see: [[Capturing this in C++]]

- Inside a member function:
```cpp
class counter
{
private:
	int value = 0;
	
public:
	void run()
	{
		auto inc = [this]()
		{
			value++;
		}
		
		inc();
	}
};
```
- `[this]` captures the **this pointer**
- lambda can access `value` and other members

## C++20 detail - capturing `*this` by value
##### Before C++20:
- `[this]` -> capture `this` pointer (always by reference)
- `[=]` inside a member -> also capture `this` by **reference** implicitly

##### Before C++20:
- `[=]` inside a member behaves as if it captures `*this` by **value**
	- **copy of the object**, not the raw pointer
- there is also an explicit `[*this]` capture (C++ 17)
	- makes a **copy** of current object for this lambda

#### Example:
```cpp
class Widget
{
private:
	int value = 0;
	
public:
	void demo()
	{
		// C++20: this lambda keeps a *copy* of the current widget
		auto f = [=]()
		{
			// works on the captured copy of *this*
			std::cout << value << "\n";
		};
		
		f();
	}
}
```
- (C++98 - C++20): `[]` implied **pointer** capture (like `[this]`)
- (C++ 20 +): `[=]` effectively captures a **copy** of the object (`*this`)
	- makes it safer in some cases
	

# Quick Cheat Sheet

| Capture form | What it does                                                     | Typical use                                 | Notes                                         |
| ------------ | ---------------------------------------------------------------- | ------------------------------------------- | --------------------------------------------- |
| `[]`         | captures **nothing**                                             | lambda uses only its params / globals       | safest, very explicit                         |
| `[=]`        | captures **used** outer vars by **value** (and `*this` in C++20) | small lambdas that just read outer values   | doesn’t see later changes                     |
| `[&]`        | captures **used** outer vars by **reference**                    | callbacks that must modify outer state      | risk of dangling refs if lambda outlives vars |
| `[x]`        | captures `x` by value                                            | explicit, avoid accidental captures         | good for clarity                              |
| `[&x]`       | captures `x` by reference                                        | modify just `x` from outer scope            | be sure `x` lives long enough                 |
| `[x, &y]`    | mix: `x` by value, `y` by reference                              | fine-tuned control                          | common pattern                                |
| `[z = expr]` | creates new captured var `z`, initialised with `expr`            | move-only types, precomputed values         | needs C++14+                                  |
| `[this]`     | captures `this` pointer                                          | member functions needing full object access | classic OO capture                            |
| `[*this]`    | captures a **copy** of `*this` (C++17+)                          | snapshot of object state inside lambda      | safer if object may later change              |

# External References
