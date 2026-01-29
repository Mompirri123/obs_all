**Created:** *26.11.25, 20:26*

**Status:** 

**Hashtags:**
- #CPP 
- #advancedcpp 
- #OOPS 
- #polymorphism 
- #compiletimepolymorphism 
- #functionoverloading 

**Links / Tags:** 
- **Relevance Links:**
	- 

- **Topic Tags:**
	- C++ Language Features

# Function Overloading

#### Definition:

- Multiple function can have the same name
	- As long as their parameters are: 
		1. Parameters of different type
		2. Parameter count is different
	
- Let's us use same function name for similar tasks

#### Examples:

1. **Without Function Overloading:**

- Two different functions based on the type of the input parameter

```C++
int plusFuncInt(int x, int y)
{  
  return x + y;
}  
  
double plusFuncDouble(double x, double y)
{  
  return x + y;
}  
  
int main()
{
  int myNum1 = plusFuncInt(8, 5);
  double myNum2 = plusFuncDouble(4.3, 6.26);
  
  cout << "Int: " << myNum1 << "\n";
  cout << "Double: " << myNum2;

  return 0;  
}

```

2. **With Function Overloading:**

- Instead of defining two functions that should do the same thing
- It is better to overload one
	- `plusFunc` is overloaded here to work with both `int` and `double`


```C++
int plusFunc(int x, int y) {  
  return x + y;  
}  
  
double plusFunc(double x, double y) {  
  return x + y;  
}  
  
int main() {
  int myNum1 = plusFunc(8, 5);  
  double myNum2 = plusFunc(4.3, 6.26);  
  
  cout << "Int: " << myNum1 << "\n";  
  cout << "Double: " << myNum2;  
  return 0;  
}

```

# Uses of Function Overloading

- **Intuitive:** Don't need to remember different function names
- **Type Safety:** Compiler chooses the right type based on arguments
- **Code Organisation:** Related functionality grouped under one name
- **Backward Compatibility:** Add new parameters without breaking existing code


# References


## Closely Related Notes

### Next:

### Prev:

