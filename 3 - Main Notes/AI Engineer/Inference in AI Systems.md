**Created:** *<span class ="color-green">09.01.26, 04:42</span>*

**Note Type:** #concept

**Hashtags:**
- **Relevance Tags:**
	- #aiengineer
	- #inference
- **Topic Tags:**
	- #runtime
	- #prediction
	- #serving

**Links / Tags:** 
- **Relevance Links:**
	- AI Engineer
	- Inference & Serving Systems
	- [[Training vs Inference in AI Systems]]
- **Topic Links:**
	- [[Online vs Offline Inference]]
	- [[Batching and Throughput]]
	- [[Latency Constraints in AI Systems]]
	- [[Determinism and Randomness]]
	- [[Tokenization and Context Handling]]
	- [[Resource Management for Inference]]
	- [[Cost Models for Inference]]
	- [[Failure Modes in Inference Systems]]
	
---

# Inference in AI Systems

## ***Definition:***
- Inference is the **runtime phase of an AI system** where a trained model:
	- receives new, unseen input data
	- applies learned parameters
	- produces predictions, classifications, or generated outputs
	  
- Unlike training
	- Inference involves the model applying what it has learned to make decisions
	- without needing examples of the exact result
	  
- In essence, inference is the AI model actively functioning
	  
- Inference is the **runtime phase of an AI system** where a trained model:
	- receives new, unseen input data
	- applies learned parameters
	- produces outputs such as predictions, classifications, or generated content
	  
- This is the phase where the model is **actually used**.


### Example
- A self-driving car recognising a stop sign on a road it has never encountered before
	- The model identifies the stop sign in a new setting, using its learned knowledge to make a decision in real-time
	  
- Here:-
	- Sensor input is captured and pre-processed
	- inference runs under strict latency limits
	- output is reliable and timely
	- failure or delay would have had real-world consequences
	  
- This entire process is *Inference*.

---

## Inference vs Training

**Training**
- learns parameters from data
- updates weights
- iterative and often offline

**Inference**
- uses fixed parameters
- does not learn
- forward computation only
- constrained by latency, cost, and reliability

***Visit***: [[Training vs Inference in AI Systems]]
-> Deep comparison between Training and Inference in AI Systems

---

## Core Questions Inference Introduces

### How is inference executed?
Real systems don’t always run inference the same way

***Visit***: [[Online vs Offline Inference]]  
→ choosing between real-time, batch, or delayed execution

---

### How many requests are handled at once?
Throughput and efficiency depend on grouping work.

***Visit***: [[Batching and Throughput]]  
→ how batching improves efficiency but affects latency

---

### How fast must results arrive?
Most production systems have strict time limits.

***Visit***: [[Latency Constraints in AI Systems]]  
→ understanding deadlines, tail latency, and trade-offs

---

### Are outputs repeatable or variable?
Some models must behave deterministically, others don’t.

***Visit***: [[Determinism and Randomness]]  
→ randomness, sampling, seeds, and reproducibility

---

### How is input context handled?
Modern models depend heavily on context and preprocessing.

***Visit***: [[Tokenization and Context Handling]]  
→ turning raw input into model-ready representations

---

## Related Notes
> Concepts commonly encountered alongside inference

- Serving Systems for AI Models
- Systems Design for AI
- MLOps
