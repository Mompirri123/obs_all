**Created:** *<span class ="color-green">09.12.25, 16:08</span>*

**Note Type:**

**Hashtags:**
- **Relevance Tags:**
	- 
- **Topic Tags:**
	- 

**Links / Tags:** 
- **Relevance Links:**
	- 
- **Topic Links:**
	- 

---

# Access Violation in C++

- Type of error that occurs when a program attempts to access an illegal memory location

## Common cases for Access Violation

- Dereferencing a null or invalid pointer
- Accessing an array out of bounds
- Reading or writing to memory freed by user or Operating System

## Issues caused by Access Violation

- Leads to unpredictable behaviour, Application crashes, or data corruption
- Difficult to debug as the error may not manifest immediately

## Prevention of Access Violation

- Always initialize pointers before use
- Use smart pointers (like `std::unique_ptr`, `std::shared_ptr`) to manage memory automatically
- Perform bounds checking when accessing arrays or buffers
- Use tools like Valgrind or AddressSanitizer to detect memory issues during development

# External References

