**Created:** *<span class ="color-green">09.01.26, 03:16</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
	- #math
	- #linearalgebra
	- #vectors
	- #aiengineer
- **Topic Tags:**
	- #features
	- #representations
	- #dimensions

**Links / Tags:** 
- **Relevance Links:**
	- Mathematics
	- Linear Algebra for AI
	- Vectors for AI
- **Topic Links:**
	- [[Vector as Representation]]
	- [[Vector Dimensionality]]
	- [[Feature Vectors]]
	- [[Parameter Vectors]] 
	- [[Batched Vector Representations]] 
	- [[Ordered Representations]]
---

# Vector Basics for AI

> Core meaning of **vectors as used in AI systems**.  
> This note covers concepts that usually **do not need further splitting**.

---

## 1. What a Vector Means in AI

- In AI, a vector is:
	- an ordered list of numbers
	- representing **something concrete**:
		- a data point
		- a feature set
		- a model output
		- an embedding
	
- A vector is **not**:
	- an abstract mathematical object
	- a direction-only geometric arrow (that intuition comes later)

 ***Visit***: [[Vector as Representation]]  → what it means for vectors to “represent” data in AI systems
---

## 2. Dimensionality

- vector length = number of features
- dimension = information capacity
- higher dimension ≠ better by default

In AI:
- dimensions are chosen or learned
- unused dimensions still affect computation
- dimensionality impacts:
  - similarity
  - generalization
  - efficiency

***Visit***:  [[Vector Dimensionality]] -> in - depth notes on vector dimensionality

---

## 3. Feature Vectors vs Parameter Vectors

### Feature vectors
- represent inputs or intermediate representations
- examples:
  - image pixels
  - word embeddings
  - sensor readings
- often have semantic meaning

***Visit***: [[Feature Vectors]]  
→ how features are encoded as vectors in AI

### Parameter vectors
- represent learnable values
- examples:
  - weights
  - biases
- usually not interpreted semantically

***Visit***: [[Parameter Vectors]]  
→ vectors used to store model parameters

---

## 4. Single Vector vs Batch of Vectors

- single vector → one sample
- batch → collection of vectors processed together

In practice:
- models almost always operate on batches
- batching affects:
  - performance
  - numerical stability
  - gradient behavior

***Visit***: [[Batched Vector Representations]]  
→ how batching changes computation and learning

---

## 5. Ordering Matters

- vector positions are meaningful
- swapping entries changes meaning
- vectors are **not** sets or bags

Ordering is critical for:
- feature engineering
- embedding alignment
- model correctness

***Visit***: [[Ordered Representations]]  
→ why index position matters in vector-based models

---

## Explicitly Out of Scope
- vector space axioms
- basis theory
- coordinate-free formulations
- proofs

---

# Closely Related concepts
- Vector Geometry & Similarity
- Vector Representations in AI
- Matrices for AI

# External References
## 3. Feature Vectors vs Parameter Vectors

### Feature vectors
- represent inputs or intermediate representations
- examples:
  - image pixels
  - word embeddings
  - sensor readings
- often have semantic meaning

***Visit***: [[Feature Vectors]]  → how features are encoded as vectors in AI

### Parameter vectors
- represent learnable values
- examples:
	- weights
	- biases
- usually not interpreted semantically

***Visit***: [[Parameter Vectors]]  → vectors used to store model parameters

---

## 4. Single Vector vs Batch of Vectors

- single vector → one sample
- batch → collection of vectors processed together

In practice:
- models almost always operate on batches
- batching affects:
  - performance
  - numerical stability
  - gradient behavior

***Visit***: [[Batched Vector Representations]]  → how batching changes computation and learning

---

## 5. Ordering Matters

- vector positions are meaningful
- swapping entries changes meaning
- vectors are **not** sets or bags

Ordering is critical for:
- feature engineering
- embedding alignment
- model correctness

***Visit***: [[Ordered Representations]]  → why index position matters in vector-based models

---

## Explicitly Out of Scope
- vector space axioms
- basis theory
- coordinate-free formulations
- proofs

---

# Closely Related concepts
- Vector Geometry & Similarity
- Vector Representations in AI
- Matrices for AI

# External References