**Created:** *<span class ="color-green">10.12.25, 09:25</span>*

**Note Type:** #atomic 

**Hashtags:**
- **Relevance Tags:**
	- #cpp 
	- #controlflow 
	- #loops 
- **Topic Tags:**
	- #forloops
	- #iteration 
	- #counters

**Links / Tags:** 
- **Relevance Links:**
	- Loops in C++             %% Parent %%
	- [[while loops in C++]]
- **Topic Links:**
	- [[Range Based For Loop in C++]]

---

# 'for' loops in C++

> Part of C++ loop mechanisms

- a `for` loop repeats code using a clear **start → check → update** pattern

## Basic 'for' loop

```cpp
for (initialisation; condition; perLoopUpdate)
{
	// body
}
```

- Translates to :
	- run *Initialisation* once at the start of the loop
	- check the condition
	- if true $\rightarrow$ run body
	- then do the update
	- repeat until condition fails (checked at start of each iteration)

#### basic example

```cpp
for (int i = 0; i < n; i++)
{
	sum += i;
}
```

## When to use a for loop

- counting or stepping
- iterating by index
- when you know the range or number of iterations
- situations where initialisation / condition / update belong together

## empty parts are allowed

```cpp
for (;;)
{
	// will be an infinite loop unless we use a 'break' statement
}

for (int i = 0; i<n; )
{
	// manual increment can be given inside
}
```

# Internal References

## [[Range Based For Loop in C++|Range based for loops]]

- Cleaner syntax for "go through every element in a container"
- when you don't need the index

# External References
