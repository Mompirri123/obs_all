[[task 11]]
[[task 12]]
[[task 13]]
# Task Ideas

## Idea 1
#### Problem to solve
- When there is a task that needs GPUs and there isn't one available/free, then the executor gets struck inside a loop while waiting for a GPU. While it’s waiting, it doesn’t check for  any stop signals or new messages, so it can respond slowly and other tasks can get stuck behind it
#### prompt
- Fix the following issue, when there is a task that needs GPUs and there isn't one available/free, then the executor gets struck inside a loop while waiting for a GPU. While it is waiting for a GPU to be available, other tasks get struck behind it
#### approach to follow
- **Simple: requeue GPU tasks instead of blocking**
    
    - If acquire_gpu fails, push the item back to the queue with a short delay/backoff and continue the main loop. This keeps stop/probe handling responsive.
- **Better: non‑blocking wait with periodic control checks**
    
    - Replace the tight while with a loop that:
        - checks stop/probe messages
        - calls collect_finished_works()
        - sleeps briefly
        - retries acquire_gpu
    - This still waits, but stays responsive.
- **Best: event/condition on GPU release**
    
    - Add a Condition/event that’s signaled when GPUs are released.
    - The executor can wait(timeout) and still process control messages.
 
## Idea 2
- DataFrame.to_pandas() / to_arrow() use ThreadPoolExecutor while count() explicitly avoids it due to “duckdb results will be mixed up.” This likely hits concurrency bugs or non‑determinism.

## template
- Review Model A and Model B changes under folders A and B for this prompt:
	- Fix the following issue, when there is a task that needs GPUs and there isn't one available/free, then the executor gets struck inside a loop while waiting for a GPU. While it is waiting for a GPU to be available, other tasks get struck behind it
- For each model, explicitly describe the full flow of how the option is applied (functions, parameters, and call chain), list new or modified functions/parameters, and explain which tests were added and what each test proves (including edge cases). Provide pros/cons that reference the exact functions or parameters responsible. Include PR readiness for each model and concrete next‑turn fixes. Finally, output a comparison table (using the provided rating scale) plus a justification of which model is better and why. Write everything into `/Users/home/obs_all/task11_turn1.md` and also copy it into `/Users/home/obs_all/2 - Source Material/Alignerr/task11/`