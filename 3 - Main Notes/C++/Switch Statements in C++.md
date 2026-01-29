**Created:** *25.11.25, 20:46*

**Status:** #atomic

**Hashtags:**

- **Relevance Tags:**
    - #cpp
    - #switchstatements
    - #conditionalstatements
    - #controlflow
    
- **Topic Tags:**
    - #branches
    - #break
    - #default
    - #fallthrough


**Links / Tags:**

- **Relevance Links:**
    - Control Flow in C++ <!-- parent -->
    - C++ Language Features <!-- higher-level parent -->
    
- **Topic Links:**
    - [[If else in C++]]
    - [[Break and Continue in C++]]


# Switch Statements in C++

`switch` is used to select **one of many code blocks** based on the value of an expression.

## How switch works

- the `switch` expression is evaluated once
- its value is compared with each `case` label
- when a match is found, execution jumps to that `case`
- from there it continues **downwards** until:
    - a `break` is hit, or
    - the `switch` block ends
    
- `break` and `default` are **optional**, but very important in practice


### Example usage:

```c++
int day = 4;

switch (day) 
{
	case 1:
		cout << "monday";
		break;
	case 2:
		cout << "tuesday";
		break;
	case 3:
		cout << "wednesady";
		break;
	case 4:
		cout << "Thursday";
		break;
	case 5:
		cout << "Friday";  
	    break;  
	case 6:  
	    cout << "Saturday";  
	    break;  
	case 7:  
	    cout << "Sunday";  
	    break;
}

// Output : "Thursday"

```

## Key points

- The `switch` expression must be an **integral or enum type**  
	- (e.g. `int`, `char`, `enum`, `enum class`)
    
- Each `case` label must be a **constant expression**  
	- (no variables or runtime values)
	
- `default` runs if no `case` matches
- `break` exits the `switch` block immediately


## Fall-through (common control-flow error)

- If you **forget `break`**
	- execution “falls through” into the next case
	
- Sometimes, fall-through could be intentional
	- but, it is often an *logical bug*

```cpp
int day = 1;

switch (day) 
{
	case 1:
		cout << "monday";
	case 2:
		cout << "tuesday";
	case 3:
		cout << "wednesady";
}

// Output: "mondaytuesdaywednesday"
// as there is no break statement found after entering case1 it would jump down to case 2
// then again no break jumps it to case 3 and end after the switch statement ends
// hence the output has all the cout(s) in one go
```

# References

