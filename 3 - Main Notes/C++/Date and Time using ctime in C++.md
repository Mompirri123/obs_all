**Created:** 01.12.25, 18:18

**Status:** #

**Hashtags:**
- #CPP 
- #Time 

**Links / Tags:** 
- **Relevance Links:**
	- C++ Standard Library
	- C++ Date and Time Libraries
	  
- **Topic Tags:**
	- 
	  
---
# Date and Time using in C++

> Working with dates and times using the C-style `<ctime>` library in C++.

- The `<ctime>` header provides access to **legacy C-style time facilities**
- Allows programs to obtain the
	- current time represent timestamps
	- convert them into human-readable forms.
	  
- These facilities are inherited from C and remain available in C++
	- primarily for compatibility and interoperability with older code
	  

## Including the `ctime` library

```cpp
// Import the data/time library
#include <ctime>
```

## Getting Current Date and Time

- `time()` - Calendar time as timestamp
- `ctime()` - Converts timestamp into a readable string

## ⚡ Two ways to use "time()" function

- The time function both `returns` and `stores`  time
- So we can both
	- read the time from its `return` value
	- store and read it from `time_t` variable

### 1. Using Parameter (Traditional):

- A parameter is passed to `store`
- Passing address to store the timestamp
- The traditional way of doing things

```cpp
#include <iostream>
#include <ctime>

int main()
{
	// Get the timestamp to current date and time
	time_t timestamp; // declare a time stamp variable
	time(&timestamp); // pass ADDRESS to store timestamp
	
	// Display the date and time represented by the timestamp
	std::cout << ctime(&timestamp); // Output: "Mon Dec 1 22:57:45 2025"
	return (0);
}
```

#### What happens:

- `&timestamp` = address of timestamp variable
- `time()` stores result at this address
- Also returns the same value (often ignored)

### 2. Using Return Value (Modern)

```cpp
#include <iostream>
#include <ctime>

int main()
{
	time_t timestamp = time(NULL);  // pass NULL, get RETURN value
	std::cout << ctime(&timestamp); // Display: "Mon Dec 1 22:59:42 2025"
	return (0);
}
```

### 🔍 Key Differences
| Aspect                    | Method 1 (Parameter)            | Method 2 (Return Value)  |
| ------------------------- | ------------------------------- | ------------------------ |
| **Syntax**                | `time(&timestamp)`              | `timestamp = time(NULL)` |
| **What's passed**         | `&timestamp`Address of variable | `NULL` / `nullptr`       |
| **How value is obtained** | Written to parameter            | Returned by function     |
| **Common use**            | Older C-style code              | Modern C++ style         |

#### Pro Tip: Even Cleaner

```cpp
time_t now = time(nullptr);  // Use nullptr instead of NULL (C++11+)
std::cout << ctime(&now);
```

### 💡 Simple Example (Both Methods)

```cpp
#include <iostream>
#include <ctime>

int main()
{
	// Method 1: Parameter-based
	time_t ts1; 
	time(&ts1);
	std::cout << "Method 1:" << ctime(&ts1);
	
	// Method 2: Return-value based
	time_t ts2 = time(nullptr);
	std::cout << "Method 2:" << ctime(&ts2);
	
	// Both produce identical output!
	return(0);
}
```

# References
