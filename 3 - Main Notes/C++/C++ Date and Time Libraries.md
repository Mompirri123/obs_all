**Created:** *<span class ="color-green">11.01.26, 18:09</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #standardlibrary
	- #time
	- #date
	- #overview
- **Topic Tags:**
	- #ctime
	- #chrono
	- #timestamps
	- #clocks
	- #durations
	
**Links / Tags:** 
- **Relevance Links:**
	- C++ Standard Library
- **Topic Links:**
	- [[Date and Time using ctime in C++]]
	- [[Chrono Library in C++]]
	- [[Clocks and Durations in C++]]
	- [[Time Points and Timestamps in C++]]
	- [[Time Formatting and Parsing in C++]]
	
---

# C++ Date and Time Libraries

> Time and date facilities provided by the C++ standard library.

- C++ provides multiple facilities for working with
	- **time measurement**
	- **timestamps**
	- **date-related concepts**
- These have evolved over time
	- ranging from **legacy C-style APIs** to **modern, type-safe C++ abstractions**
	  
- Time and date support in C++ is used for:
	- measuring elapsed time
	- working with wall-clock time
	- scheduling and timeouts
	- logging and timestamps
	- performance measurement
	  
---

## Conceptual Components

### **Legacy C-style APIs**
- **[[Date and Time using ctime in C++]]**
	- C-style time facilities inherited from C
	- `time_t`, `time()`, `ctime()`, `tm`
	- Simple and widely supported, but limited and error-prone
	  
---

### **Modern C++ Time Facilities**
- **[[Chrono Library in C++]]**
	- Type-safe time utilities introduced in C++11
	- Durations, clocks, and time points
	- Preferred for modern C++ code
	  
- **[[Clocks and Durations in C++]]**
	- `system_clock`, `steady_clock`, `high_resolution_clock`
	- Measuring time intervals and performance
	  
- **[[Time Points and Timestamps in C++]]**
	- Representing specific moments in time
	- Conversions between clocks and durations
	  
---

### **Formatting and Parsing**
- **[[Time Formatting and Parsing in C++]]**
	- Converting time values to and from human-readable forms
	- Locale and formatting considerations
	- Modern formatting support (C++20+)
	  
---

## Design Considerations

- Prefer **`<chrono>`** facilities in modern C++
- Use `<ctime>` only when interacting with legacy APIs or C libraries
- Choose clocks carefully based on:
	- precision requirements
	- monotonicity
	- performance constraints
	  
---

# Related Notes
> Things you might want to think about alongside this note, but not because of it

- Concurrency and timing
- Performance engineering
- Logging systems
- Operating system time APIs

# References
- ISO/IEC C++ Standard
- [cppreference – chrono](https://en.cppreference.com/w/cpp/chrono)
- [cppreference – ctime](https://en.cppreference.com/w/cpp/header/ctime)
