**Created:** *27.11.25, 06:48*

**Status:** 

**Hashtags:**
- #CPP 
- #advancedcpp 
- #functions 
- #OOPS 
- #encapsulation

**Links / Tags:** 
- **Relevance Links:**
	- [[Encapsulation in C++]]

- **Topic Tags:**
	- 

# Friend Functions

- `friend` function is not a member of the class
	- but is allowed to access the class's private data
	  
- Declaration inside class with `friend` keyword
- Definition outside class like normal function
- Not a member function - called without object prefix
- **One-way friendship:**
	- friend can access class
	- but class can't access friend's privates

### Example: Friend Functions

```cpp
class Employee {
  private:
    int salary;

  public:
    Employee(int s) {
      salary = s;
    }

    // Declare friend function
    friend void displaySalary(Employee emp);
};

void displaySalary(Employee emp) {
  cout << "Salary: " << emp.salary;
}

int main() {
  Employee myEmp(50000);
  displaySalary(myEmp);
  return 0;
}
```

- `friend` function `displaySalary()` is declared inside the `Employee` class but defined outside of it
- Even though `displaySalary()` is not a member of the class
	- it can still access the private `salary`
- In the `main()` function
	- we create an `Employee` object and call the friend function to print the salary



# References


## Closely Related Notes

### Next:

### Prev:

