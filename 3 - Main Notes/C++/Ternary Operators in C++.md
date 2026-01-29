**Created:** *25.11.25, 19:33*

**Status:** #atomic

**Hashtags:**

- **Relevance Tags:**
    - #cpp
    - #ternaryoperators
    - #conditionalstatements
    
- **Topic Tags:**
    
    - #operators
    - #syntax
    - #readability
    

**Links / Tags:**

- **Relevance Links:**
    
    - Operators in C++ <!-- parent, plain text -->
    - Control Flow in C++ <!-- if/switch cluster, plain text -->
    
- **Topic Links:**
    
    - [[Polymorphism in C++]]
      
---

# Ternary Operator

- Short hand for `if... else` statements
- Known as **Ternary Operators**

- Returns a value based on condition
	- If condition is true, returns first value
	- If condition is false, returns second value

- **Syntax :**
	- `variable = (condition) ? expressionIfTrue : expressionIfFalse`
	
	- Example:
```C++
	int time = 20;
	string result = (time < 18) ? "Good day" : "Good evening"
```
-  The above expression is similar to the following if else statement
```C++
	int time = 20;
	
	if (time < 18)
	{
		string result = "Good day";
	}
	else
	{
		string result = "Good evening";
	}
```

- Ternary operator can also be used within the if... else statement

- When to use If... else or Ternary:
	- short and simple conditions : Ternary;
	- longer and more complex : if else;


# Nested Ternary

- Ternary Operators can be nested to handle more than 2 outcomes
	- Though it can make code harder to read

```C++
int time = 22;

string message = (time < 12) ? "Good morning"
	: (time < 18) ? "Good Afternoon"
	: "Good evening"

cout << message;
```

- The above Nested Ternary is similar to the following `if... else` statement
```c++
int time = 22;
string message;

if (time < 12)
{
    message = "Good morning";
}
else if (time < 18)
{
    message = "Good afternoon";
}
else
{
    message = "Good evening";
}

cout << message;

```


# References


## Closely Related Notes

### Next:

### Prev:

