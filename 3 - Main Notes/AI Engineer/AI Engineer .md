**Created:** *<span class ="color-green">08.01.26, 19:54</span>*

**Note Type:**
#map

**Hashtags:**
- **Relevance Tags**
    - #aiengineer
    
- **Topic Tags**
    - #ml
    - #dl 
    - #data  
    - #mlops    
    - #softwareengineering    
    - #math 
    - #cloud
     
**Links / Tags:**
- **Relevance Links:**
    - Foundations:
	    - [[Mathematics for AI]] — math needed for ML/DL (linear algebra, calculus, probability, statistics)
	    - [[Programming Foundations for AI]] — programming mental models and engineering habits for AI work        
	    - [[Computer Science Foundations for AI]] — DSA, OS basics, networking basics
	      
	- Data + Systems:
	    - [[Data Foundations for AI]] — data collection, cleaning, labeling, leakage
	    - [[Data Engineering for AI]] — ETL/ELT, warehouses, batch vs streaming    
	    - [[Systems Design for AI]] — serving patterns, latency, reliability
	    
	- Production / Deployment:
	    - [[MLOps]] — packaging, versioning, CI/CD, monitoring, retraining    
	    - [[Cloud for AI]] — GPUs, storage, IAM, deployment primitives
	    
	- Applied Domains:
		- [[NLP Systems]] — text, embeddings, transformers, LLM basics
		- [[Vision Systems]] — image pipelines, CNNs, detection/segmentation
		- [[Generative AI Systems]] — diffusion, LLMs, prompting, RAG, safety basics
		- [[Reinforcement Learning Systems]] — agents, reward, MDPs (optional)
	
---

# AI Engineer

> Main hub for AI Engineer notes  
> Links to second-level maps: foundations, ML/DL core, data + systems, MLOps, cloud, and applied domains.

- **Definition:**
	- process of designing and implementing AI systems
		- using pre-trained models,
		- existing AI tools to solve practical problems
	- AI Engineering focuses on designing, building, and operating intelligent systems in the real world.
	
- **Core responsibilities:**
	- integrating learning-based components into systems
	- building reliable data and inference pipelines
	- handling scale, latency, and failure modes
	- deploying, monitoring, and iterating safely
	- AI engineering = building ML systems that work in the real world:
	  
- For AI Engineering we need to:
	- understand math enough to debug models
	- build data pipelines that don’t leak or break
	- train + evaluate models correctly
	- deploy reliably and monitor drift
	- iterate fast with clean engineering habits
	
---

## 1. What AI Engineers Build

- AI Engineers build **real-world systems** that:
	- run trained models in production environments
	- integrate data pipelines and inference services
	- operate under latency, cost, and reliability constraints
	- are monitored, updated, and scaled over time
	  
- The focus is not just models, but **end-to-end AI systems**.
---

## 2. Model Lifecycle & Runtime Systems

> Core mental model: how models move from training to real usage.

- [[Model Lifecycle for AI Systems]]  
	- training → validation → deployment → monitoring
	  
- [[Inference & Serving Systems]]  
	- online/offline inference, batching, latency, runtime constraints
	  
- This is where learning-based components meet real systems.

---

## 3. Data & Systems Engineering

> Everything that makes models usable and reliable.

- [[Data Foundations for AI]]  
  data quality, leakage, splits

- [[Data Engineering for AI]]  
  ETL, schemas, versioning, batch vs streaming

- [[Systems Design for AI]]  
  latency, scaling, reliability

---

## 4. Foundations (must-have)

> Core knowledge required to reason about AI systems.

- [[Mathematics for AI]]  
  linear algebra, probability, statistics, optimization

- [[Programming Foundations for AI]]  
  Python, debugging, clean code, testing habits

- [[Computer Science Foundations for AI]]  
  DSA, complexity, OS and networking basics

---

## 5. Production & Operations

> Making AI systems stable, observable, and maintainable.

- [[MLOps]]  
  CI/CD, model registry, monitoring, retraining

- [[Cloud for AI]]  
  compute, GPUs, storage, IAM, deployment primitives

---

## 6. Domain Interaction Points

> AI Engineers interact with domains through **systems**, not raw theory.

- [[NLP Systems]]  
	- text processing, embeddings, LLM inference, retrieval, and language pipelines
	  
- [[Vision Systems]]  
	- image/video ingestion, model inference, post-processing, and latency-aware pipelines
	  
- [[Generative AI Systems]]
	- prompt handling, context management, generation control, safety, and cost constraints
	  
- [[Reinforcement Learning Systems]]
	- agents interacting with environments, reward loops, and long-running decision systems
	  
---

# References