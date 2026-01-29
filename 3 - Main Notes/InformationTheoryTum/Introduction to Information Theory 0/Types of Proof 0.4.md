
19.11.25 18:55

Status:

Tags:
- [[Introduction to Information Theory 0]] 



---
# Types of Proof 0.4
%% page 21 (42) %%

- Proof may contain more than one type of argument
	- As the proof may contain within it several different sub-proofs

## Proof by Construction

- A way to prove a statement stating that a particular type of object exists
- This is done by demonstrating how to construct the said object
- This technique is called as ***Proof by Construction***

### Theorem 2.22
- For each even number $n$ greater than $2$, there exists a 3-regular graph with $n$ nodes
	
- **Proof:**
- Let $n$ be an even number greater than $2$.
- Construct graph $G = (V, E)$ with $n$ nodes as follows
	- The set of nodes of $G$ is $V = \{0, 1, 2, ... , n-1\}$
	- The set of edges is the set 
		- $E = \{\{i,\ i+1\}\ |\ for\ 0\leq i \leq n-2 \}\ \cup \{\{n-1,\ 0\}\ \cup \{\{i,\ i + n/2 \}\ |\ for\ 0\leq i \leq n/2 - 1\}   \}$
		  
	- Picture the nodes of this graph written consecutively around the circumference of a circle
	- In that case the edges described in the top line of $E$ go between adjacent pairs around the circle
	- The edges described in the bottom line of $E$ go between nodes on opposite sides of the circle
	- This mental picture clearly shows every node in $G$ has degree 3
		
	- Example:
		- **Nodes arranged in circle**: 1 → 2 → 3 → 4 → 5 → 6 → back to 1
			
		- **Adjacent edges** (top line): (1,2), (2,3), (3,4), (4,5), (5,6), (6,1)
			
		- **Opposite edges** (bottom line): (1,4), (2,5), (3,6)
			
		- **Each node degree**: 2 (from adjacent) + 1 (from opposite) = 3
			
		- **Degree Calculation**
			- Node 1: connected to 2, 6 (adjacent) + 4 (opposite) = degree 3
			- Node 2: connected to 1, 3 (adjacent) + 5 (opposite) = degree 3
			- Node 3: connected to 2, 4 (adjacent) + 6 (opposite) = degree 3
			- Node 4: connected to 3, 5 (adjacent) + 1 (opposite) = degree 3
			- Node 5: connected to 4, 6 (adjacent) + 2 (opposite) = degree 3
			- Node 6: connected to 5, 1 (adjacent) + 3 (opposite) = degree 3
%% Creates a 3-regular graph arranged in a circle with opposite connections%%
```mermaid
graph LR
    1 --> 2
    2 --> 3
    3 --> 4
    4 --> 5
    5 --> 6
    6 --> 1
    
    1 --> 4
    2 --> 5
    3 --> 6
    
    linkStyle 6,7,8 stroke:#ff0000,stroke-width:2px
```

## Proof By Contradiction

- Common form of argument for proving a theorem
- Assume that the theorem is false
	- show that this assumption leads to an obviously false consequence
	- called a contradiction
- Used frequently in everyday reasoning

### Example 0.23
%% page 22 (43) %%
- Jack sees Jill, who has just come in from outdoors
- On observing that she is completely dry, he knows that it's not raining
- His "proof" that it's not raining is that:
	
- *if it were raining* (assumption that the statement is false)
	
- *Jill would be wet* (the obviously false consequence)
	
- $\therefore$ it must not be raining

### Theorem 0.24
%% page 22 (43) %%
- **Prove:** $\sqrt{2}$  is irrational
	
- **Proof**
- **Assuming Contradiction**
	- Assuming, $\sqrt{2}$  is rational $\implies \sqrt{2} = m / n\ \ni\ m, n \in \mathbb{N}$
	  
- **Proving the failure of contradiction statement**
	- when both $m$ and $n$ are integers
	- After diving them with their common factor
		- Atleast one of $m$ and $n$ is an odd number post division
	
- $\because \sqrt{2} = m / n\ \implies\ 2 = m^2 / n^2\ \implies\ 2n^2 = m^2$
	
- $\because m^2 = 2 * n^2 \implies m^2$ is even
- $\because m$ is even
	- we can write $m = k\ |\ k \in \mathbb{z}$
	  
- so, $2 n^2 = (2k)^2$
- $n^2 = 2k^2\ \implies\ n$ is even
- we have established that both $m$ and $n$ are even
	- But, we had earlier deduced that at-least one $m$ and $n$ must be odd
- A contradiction which means 
	- $\sqrt{2}$ is not rational $\implies \sqrt{2}$ is irrational
	  
- Hence, we proved $\sqrt{2}$ is irrational using **Proof of Contradiction**


## Proof by Induction

### Core Concept

- Method to prove that property $\mathcal{P}(k)$ holds for all natural numbers $k = 1, 2, 3, \ldots$
    
- Works for any infinite set, commonly $\mathbb{N} = {1, 2, 3, \ldots}$

### Two Essential Parts

#### 1. Basis Step

- Prove $\mathcal{P}(1)$ is true
    
- The **foundation** of the induction

#### 2. Induction Step

- Prove that **for each $i \geq 1$**:  
	
	- If $\mathcal{P}(i)$ is true, then $\mathcal{P}(i+1)$ is true
    
- Contains the **induction hypothesis**: assumption that $\mathcal{P}(i)$ is true

### How It Works

```mermaid
graph TB
    A[Basis: P₁] --> B[Induction: P₁ → P₂]
    B --> C[P₂]
    C --> D[Induction: P₂ → P₃]
    D --> E[P₃]
    E --> F[Induction: P₃ → P₄]
    F --> G[P₄]
    G --> H[...]
    
    I[Induction Step: For any i, P_i → P_i+1] -.-> B
    I -.-> D
    I -.-> F
```

### Key Insight

- **Basis** gives us the first domino: $\mathcal{P}(1)$
    
- **Induction step** ensures each domino knocks down the next
    
- Together they prove $\mathcal{P}(k)$ for **all** $k$
    

### Variations & Generalizations

#### Different Starting Point

- Basis can start at any value $b$ (not necessarily 1)
    
- Proves $\mathcal{P}(k)$ for every $k \geq b$
    

#### Strong Induction

- **Stronger induction hypothesis**: Assume $\mathcal{P}(j)$ is true for **every $j \leq i$**
    
- Use this to prove $\mathcal{P}(i+1)$
    
- Particularly useful for recursive definitions
    

### Standard Proof Format

text

_Basis:_ Prove that 𝒫(1) is true.

[Proof details...]

_Induction step:_ For each i ≥ 1, assume that 𝒫(i) is true 
and use this assumption to show that 𝒫(i+1) is true.

[Proof details...]

### Mortgage Example Setup

**Prove:** For each $t \geq 0$:  
Pt=PMt−Y(Mt−1M−1)Pt​=PMt−Y(M−1Mt−1​)

### Proof by Induction

#### Basis Step ($t = 0$)

**Already proven:**

- $P_0 = PM^0 - Y\left(\frac{M^0 - 1}{M - 1}\right)$
    
- $M^0 = 1$ ⇒ $P_0 = P(1) - Y(0) = P$ ✓
    
- Matches definition $P_0 = P$
    

#### Induction Step

**Induction Hypothesis:** Assume formula holds for $t = k$:  
Pk=PMk−Y(Mk−1M−1)Pk​=PMk−Y(M−1Mk−1​)

**Goal:** Prove formula for $t = k + 1$:  
Pk+1=PMk+1−Y(Mk+1−1M−1)Pk+1​=PMk+1−Y(M−1Mk+1−1​)

**Proof:**

1. **Start with recurrence relation:**  
    Pk+1=M⋅Pk−YPk+1​=M⋅Pk​−Y
    
2. **Substitute induction hypothesis:**  
    Pk+1=M[PMk−Y(Mk−1M−1)]−YPk+1​=M[PMk−Y(M−1Mk−1​)]−Y
    
3. **Distribute and simplify:**  
    Pk+1=PMk+1−Y(Mk+1−MM−1)−YPk+1​=PMk+1−Y(M−1Mk+1−M​)−Y
    
4. **Combine the $-Y$ terms:**  
    Pk+1=PMk+1−Y(Mk+1−MM−1+1)Pk+1​=PMk+1−Y(M−1Mk+1−M​+1)
    
5. **Simplify the fraction:**  
    Mk+1−MM−1+1=Mk+1−M+M−1M−1=Mk+1−1M−1M−1Mk+1−M​+1=M−1Mk+1−M+M−1​=M−1Mk+1−1​
    
6. **Final result:**  
    Pk+1=PMk+1−Y(Mk+1−1M−1)✓Pk+1​=PMk+1−Y(M−1Mk+1−1​)✓
    

### Conclusion

By mathematical induction, the formula holds for all $t \geq 0$.





---
# References



---
# Closely Related Notes

## Next:
- Introduction to Information theory 0.5

## Prev:
- Definitions, Theorems, and proofs - 0.3

## Closely Related Notes:
