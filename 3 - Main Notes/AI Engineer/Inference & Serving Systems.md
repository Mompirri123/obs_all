**Created:**  *<span class ="color-green">09.01.26, 03:49</span>*

**Note Type:** #map

**Hashtags:**
- **Relevance Tags:**
    - #aiengineer
    - #map
- **Topic Tags:**
    - #inference
    - #serving
    - #systems
    - #latency
    - #deployment
      
**Links / Tags:**
- **Relevance Links:**
    - AI Engineer
    
- **Topic Links:**
    - [[Inference in AI Systems]]
    - [[Serving Systems for AI Models]]
	- [[Model Interfaces and APIs]]
	  
---

# Inference & Serving Systems

> Inference & serving are the part of an AI system where a trained model is **actually used**.

- This is where:
	- inputs are sent to a model
	- predictions or generations are produced
	- results are returned to users or other systems
	  
- Unlike training, this part of the system must:
	- respond within time / latency limits
	- handle many requests reliably
	- run continuously in production
	- deal with failures, cost, and scale
	  
- Is about **running models safely and efficiently in the real world**.


## 1. Inference (runtime execution)

> What happens when a trained model is executed

- [[Inference in AI Systems]]

Talks about:
- runtime execution
- latency vs throughput
- state and context
- output behaviour
- failure visibility
---

## 2. Serving (system exposure)
> How inference is exposed, scaled, and operated.

- [[Serving Systems for AI Models]]

Talks about:
- request handling
- deployment topology
- scaling strategies 
- availability guarantees
---

## 3. Interfaces and Integration
> Boundary between inference systems and external callers.

- [[Model Interfaces and APIs]]
---

# Related Notes

> Contextually adjacent, not owned here

- Data Foundations for AI    
- Systems Design for AI
- MLOps
- Model Lifecycle for AI Systems
---

# References
