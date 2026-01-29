**Created:** *<span class ="color-green">09.12.25, 15:29</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #cpp
	- #algorithms
	- #dsa
	- #overview
- **Topic Tags:**
	- #sorting
	- #searching
	- #graphs
	- #dp
	- #math

**Links / Tags:** 
- **Relevance Links:**
	- C++ Data Structures and Algorithms
	- Algorithms
- **Topic Links:**
	- Algorithmic Categories
		- [[C++ Sorting Algorithms]]
		- [[C++ Searching Algorithms]]
		- [[C++ Graph Algorithms]]
		- [[C++ String Algorithms]]
		- [[C++ Dynamic Programming]]
		- [[C++ Mathematical Algorithms]]
		- [[C++ Optimization Techniques]]
	- [[Algorithmic Complexity]]
	  
---

# C++ Algorithms

> Algorithmic techniques and problem-solving patterns in C++.

Algorithms define **procedures for solving problems and transforming data**
in a finite sequence of steps.

In C++, algorithms are influenced by:
- the underlying data structures
- iterator- and range-based abstractions
- value, reference, and move semantics
- time and space complexity constraints

They range from **generic algorithms provided by the standard library** to
**custom, problem-specific implementations**.

---

## i. Algorithm Categories

Algorithms in C++ can be organized into the following major conceptual groups.

### 1. [[C++ Sorting Algorithms|Sorting Algorithms]]
- Sorting algorithms reorder data according to a defined ordering relation
- Fundamental to many other algorithms
- serve as building blocks for more complex solutions.

### 2. [[C++ Searching Algorithms|Searching Algorithms]]

- Searching algorithms locate elements or patterns within data structures
- They vary based on
	- data organization
	- ordering
	- performance requirements.

### 3. [[C++ Graph Algorithms|Graph Algorithms]]

Graph algorithms operate on graph-structured data to solve problems involving
connectivity, paths, cycles, and flows.

### 4. [[C++ String Algorithms|String Algorithms]]

String algorithms process sequences of characters and are used for pattern
matching, text processing, and parsing.

### 5. [[C++ Dynamic Programming|Dynamic Programming]]

Dynamic programming algorithms solve problems by breaking them into
overlapping sub-problems and storing intermediate results.

### 6. [[C++ Mathematical Algorithms|Mathematical Algorithms]]

Mathematical algorithms perform numerical and number-theoretic computations,
often forming the foundation for cryptography and optimization.

### 7. [[C++ Optimization Techniques|Optimization Techniques]]

Optimization techniques improve algorithm efficiency by reducing redundant
work or exploiting problem structure.

---

## ii. [[Algorithm Complexity|Complexity of Algorithms]]

- Algorithmic complexity describes
	- **how the cost of an algorithm grows** with input size
	  
- In the context of C++ algorithms, complexity analysis is used to:
	- compare different algorithmic approaches
	- understand scalability limits
	- reason about performance trade-offs
	- choose appropriate data structures and algorithms together
	  
- Complexity is usually expressed in terms of:
	- **time complexity** — how execution time grows
	- **space complexity** — how memory usage grows
	  
- These costs are analyzed independently of hardware
	- focusing on **growth rates**
	- rather than absolute runtime.
	  
- Key ideas associated with algorithm complexity include:
	- asymptotic analysis
	- best, average, and worst-case behavior
	- amortized complexity
	- trade-offs between time and space
	  
- 🖇️***Detailed discussion***:- [[Algorithmic Complexity]]
---

# Related Notes
> Things you might want to think about alongside this note, but not because of it

- Computational complexity
- Problem-solving strategies
- Performance engineering

---
# References
- Algorithm textbooks
- [cp-algorithms](https://cp-algorithms.com/)
- [cppreference – algorithms](https://en.cppreference.com/w/cpp/algorithm)
