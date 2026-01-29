**Created:** *27.11.25, 04:21*

**Status:** 

**Hashtags:**
- #CPP 
- #cppclasses 
- #OOPS 
- #constructors 
- #polymorphism 
- #compiletimepolymorphism
- #constructoroverloading


**Links / Tags:** 
- **Relevance Links:**
	- 

- **Topic Tags:**
	- 

# Constructor Overloading

## Definition:

- Can have more than one constructor in same class
	- This is called as **Constructor Overloading**
	  
- Each constructor must have 
	- Different number of parameters or
	- Different type of parameters
	
- The compiler knows which one to use when creating an object


## Why Use Constructor Overloading?

- To give flexibility when creating objects
- To set default or custom values
- To reduce repetitive code
- To create objects with custom values.

## Example: Constructor with multiple Operators

```cpp

class Car
{
public:
	string _brand;
	string _model;
	
	Car()
	{
		_brand = "unknown";
		_model = "unknown";
	}
	
	Car(string brand, string model)
	{
		_brand = brand;
		_model = model;
	}
};

int main()
{
	 Car car1;  
	 Car car2("BMW", "X5");  
	 Car car3("Ford", "Mustang");
	 
	 cout << "Car1: " << car1.brand << " " << car1.model << endl;
	 cout << "Car2: " << car2.brand << " " << car2.model << endl;
	 cout << "Car3: " << car3.brand << " " << car3.model << endl;
	 
	 return 0;
}

// Car1: Unknown Unknown  
// Car2: BMW X5  
// Car3: Ford Mustang

```


# References


## Closely Related Notes

### Next:

### Prev:

