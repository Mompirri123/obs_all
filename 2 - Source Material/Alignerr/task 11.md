
**UUID:**
a10bca95-e4f2-43aa-8ab8-4193204e2374

**Task ID:**
a10bca95-e4f2-43aa-8ab8-4193204e2374

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

### Working
- Persistent `WorkQueueOnFileSystem` at `gpu_deferred_queue` and a flag `gpu_recently_released` and a flag `gpu_recently_released`
- `Executor.exec_loop(pool)`:
	- when `gpu_recently_released` is True, it calls `Executor.process_gpu_deferred_queue(pool)` and resets the flag
	- It pops new work from `WorkQueue.pop(...)` and iterates each `WorkItem`
	- 


### Pros and cons

#### Model A:
- Pros:
	- `__init__` creates a persistent `WorkQueueOnFilesystem` in `gpu_deferred_queue` and also a flag named `gpu_rencently_released`, using these and working with `WorkQueueOnFileSystem` that was created, it is making sure that the waiting / deferred tasks do not get lost when there is a crash from `executor()` or even on restart. This solves one of the main issues. A safety mechanism is also added, as there is a persistent queue (a queue that doesnt get lost on `executor()` crash or restart) `process_gpu_deferred_queue` which means data doesn't get lost 
	- `collect_finished_works` manipulates `gpu_deferred_queue(pool)`, this makes sure that the deferred processes are avoiding loops and are now event-based instead
- Cons:
- `exec_loop()` calls `process_gpu_deferred_queue()` , only when the value for `gpu_recently_released` is `True`. But in a situation of restart, `gpu_recently_released` is `False` which means `process_gpu_deferred_queue` is not called, so tasks deferred though are lost not on crash, but they do get struck forever in case of restart
- `process_gpu_works() ` re enqueues a task at the front of `process_gpu_deferred_queue` to the end of it after checking if there is any place in the persistent queue. This means a "First In" GPU task could get reordered. which is against "FirstInFirstServe"
- Many tests do not test things rigidly, like the `gpu_recently_released` which is toggled manually. this could have used the `collect_finished_works()`


#### Model B:
- Pros:
- `drain_pending_gpu_works` follows FirstInFirstOut order and when it meets an unsatisfied task, it stops. This keeps the GPU tasking order correct (FIFO)
- `flush_pending_gpu_works`requeues deferred GPU tasks back into the `wq` when there is a shut down.  This means no silent losses when the `èxecutor()`exits normally
- Tests for FIFO, checks whether CPU tasks not getting disturbed if there is any GPU deferring
	
- Cons:
	- `drain_pending_gpu_works` is memory only. In case the executor crashes and not a clean shutdown, deferred tasks are lost. So there is no crash safety
	- `exec_loop()`doesn't sleep when the `pending_gpu_works` is not empty which means in that case no GPUs are freed, this could cause looping and higher CPU loads which is the same as before
	- No tests to cover busy loop risks. The `TestGpuDeferralNoSpin`only checks the `gpus_freed` flag but never actually checks for looping
	

#### Justification for best overall
Model A is slightly better than B this is because it keeps deferred GPU tasks on the disk, where as B uses `pending_gpu_works` within memory and are lost on shutdown, which means a core requirement is unsatisfied by B. Both models still have gaps in different places. Model A and Model B both have weak tests though most normally tested cases are covered. Model B also doesnt solve the never sleeping issue which was existing before. So Model A though imperfect is slightly better than Model B

### Model Chosen
- Model A

---

## Turn 3

### Things to be fixed:
- in `Executor.exec_loop(pool)` a call at the start of the loop to `process_gpu_deferred_queue(pool)`should be added or `gpu_recetnly_released = TRUE` with proper conditions should be done. FirstInFirstOut policy should be followed by `process_gpu_deferred_queue(pool)` to avoid task reordering. Checks like `gpu_recently_released` which is toggled manually should have used the `collect_finished_works()`, and timing based tests should use replaced with better ways if possible
### Turn 3 Prompt:
There are still some gaps existing like in `Executor.exec_loop(pool)` a call at the start of the loop to `process_gpu_deferred_queue(pool)`should be added or `gpu_recetnly_released = TRUE` with proper conditions should be done. FirstInFirstOut policy should be followed by `process_gpu_deferred_queue(pool)` to avoid task reordering. Checks like `gpu_recently_released` which is toggled manually should have used the `collect_finished_works()`, and timing based tests are ~~to~~ too shallow for most cases

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
- `exec_loop()` checks there is available quota and any defered tasks `gpu_deferred_queue.size > 0` before calling the `process_...`to process them, this means there are no busy loops when GPUs are unavailable
- FirstInFirstServe is followed by stopping on first GPU miss and batch re-enqueueing the tail
- Manual and time related tests and flags are replaced with and now use automated ways `collect_finished_works()`  to check which also matched the actual code
- Cons:
	- The `process_gpu_deferred_queue()` pops (removes) the whole queue at once, this can be a bit heavy when the queue is large. This is not so bad but is slow when the queue size is too big


#### Model B:
- Pros:
	- only `process_gpu_deferred_queue()` goes through one item at once and keeps FirstInFirstOut order without popping whole queue which is a slightly faster logic
	- FirstInFirstOut tests are added
- Cons:
- Still has time based tests instead of automatic 
- requeuing to the front uses some file naming hacks and deletes the existing files based on the key, this is risky and can drop duplicate tasks
#### Justification for best overall


- points about model A working and Model B still having issues. A better than B


## Rating from AutoQA

### Prompt Quality Assessment

![[Pasted image 20260202115048.png]]

### Rationale Writing Quali


![[Pasted image 20260202115159.png]]