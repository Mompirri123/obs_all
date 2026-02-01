
**UUID:**
a10bca95-e4f2-43aa-8ab8-4193204e2374

**Task ID:**


**git rev-parse HEAD:**


**interface code:**
cc_agentic_coding

**Checklist:**

---
# Saves and Logs - task 11

## Turn 1

### Turn 1 Prompt:
**_Fix the following issue, when there is a task that needs GPUs and there isn't one available/free, then the_** 

**_executor gets struck inside a loop while waiting for a GPU. While it is waiting for a GPU to be available, other_**

 **_tasks get struck behind it_**


### Turn 1 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
- the `process_gpu_deferred_queue(pool)` under `Express` puts all GPU defering logic in a single place away from the mainloop. It tries to process deferred GPU tasks before entering the main loop. This doesn't get struck while waiting for GPU's. When the GPU is not available after checking with `local_gpus`, `gpu_deferred_queue: List[WorkItem] = []` stores the tasks waiting for GPU resources.

-  CPU only tasks continue running even when GPU tasks are deferred / kept waiting. When nothing is running, the executor() still reaches its idle / sleep path.

- Cons:
-  **_The GPU tasks that are waiting are stored only in the memory, so in a situation_**

  **_where the executor would crash, these tasks are forver lost and are_**

  **_unrecoverable without retiring or requeuing them somewhere or the other. This_**

  **_could lead to data losses, that too silently. No tests / checks are added_**

#### Model B:
- Pros:
	 **_All the logic related to GPU differing is present inside a single `exec_loop`_**

  **_this makes it easier to seperate the GPU wait logic from others easily. When_**

  **_GPU tasks are waiting, CPU related taks are unaffected bu GPU defering and_**

  **_continue to be scheduked and run. 'pending_gpu_items' inside the 'execloop()'_**

  **_stores all the pending GPU related pending taks sseperatel_****_y_**
  
- Cons:
 In the case where GPU tasks exist and all the GPU are unavailable then the items list wont be empty and the executor will never hit the idle state. This will cause a running loop and directly result in CPU usage increase. This when happens is a serious issue. Also the tasks that are defered and waiting are still in memory only. lastly, literally no tests where added
#### Justification for best overall
- Add small details of:
	- Common pros and cons for A and B
	- Model A specific Pros and Cons
	- Model B specific Pros and Cons
	- why winning model is winning

### Model Choosen:
- Model A

---

## Turn 2

### Things to be fixed:
 - Tests for
	 1. GPU tasks defer while CPU tasks continue
	 2. deferred GPU tasks start after GPUs are freed
	 3. no busy‑loop when GPUs are unavailable
	 4. deferred tasks survive executor restarts or are safely requeued.
 - Loses Deferred GPU tasks if the executor exits as the deferred tasks are kept only in memory
 - tight retry loops when only GPU tasks are pending and no GPUs are available
### Turn 2 Prompt:

- The approach is good but there are still some gaps. The Deferred GPU tasks should not be lost or kept only in the memory and need to have a safety mechanism to not lose them. Avoid doing forever running, retry loops when GPU tasks are pending when there are no GPUs are available as this adds CPU load for no reason. Tests for GPU deferring related issues and edge cases

### Turn 2 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	
- Cons:


#### Model B:
- Pros:
	
- Cons:

#### Justification for best overall

---

## Turn 3

### Turn 3 Prompt:

### Turn 3 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	
- Cons:


#### Model B:
- Pros:
	
- Cons:

#### Justification for best overall



