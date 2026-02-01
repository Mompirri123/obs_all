# Turn 2 Evaluation — GPU Deferral Safety + No-Spin (Model A vs Model B)

## Process Flow and Function-Level Notes

### Shared baseline behavior (both models)
- GPU gating still happens in `Executor.exec_loop(...)` based on `WorkItem.gpu_limit`.
- GPU quota is still managed by `Executor.acquire_gpu(quota)` and `Executor.release_gpu(...)`, with releases triggered in `Executor.collect_finished_works()`.
- No public API changes; all behavior is internal to the executor.

### Model A — How it works (flow)
1. `Executor.__init__(...)` creates a persistent `WorkQueueOnFilesystem` at `gpu_deferred_queue` and a flag `gpu_recently_released`.
2. `Executor.exec_loop(pool)`:
   - When `gpu_recently_released` is True, it calls `Executor.process_gpu_deferred_queue(pool)` and resets the flag.
   - It pops new work from `WorkQueue.pop(...)` and iterates each `WorkItem`.
   - If `WorkItem.gpu_limit > 0` and `Executor.acquire_gpu(...)` fails, the item is pushed into `Executor.gpu_deferred_queue`.
   - If pushing to `gpu_deferred_queue` fails, it marks the item `WorkStatus.FAILED` and pushes it into `cq`.
3. `Executor.collect_finished_works()` releases GPUs and sets `gpu_recently_released = True` to trigger deferred work processing.
4. On shutdown, `Executor.handle_shutdown_deferred_queue()` logs how many deferred tasks remain in the persistent queue.

**Relevant functions and parameters**
- `Executor.exec_loop(pool)`:
  - Calls `process_gpu_deferred_queue(pool)` only when `gpu_recently_released` is True.
  - Defers GPU tasks by calling `gpu_deferred_queue.push(item)` when `acquire_gpu(...)` fails.
- `Executor.process_gpu_deferred_queue(pool)`:
  - Pulls items from the persistent queue, retries `acquire_gpu(...)`, and re-enqueues when still blocked.
- `Executor.gpu_deferred_queue`:
  - Persistent, on-disk queue for deferred GPU tasks.
- `Executor.gpu_recently_released`:
  - Event flag that gates deferred-queue processing.

### Model B — How it works (flow)
1. `Executor.__init__(...)` creates an in-memory list `pending_gpu_works` and a flag `gpus_freed`.
2. `Executor.exec_loop(pool)`:
   - Pops new work from `WorkQueue.pop(...)`.
   - If `WorkItem.gpu_limit > 0` and `Executor.acquire_gpu(...)` fails, the item is appended to `pending_gpu_works`.
   - CPU-only tasks are still submitted to the pool immediately via `pool.apply_async(...)`.
   - After `collect_finished_works()` runs, if `gpus_freed` is True, it calls `drain_pending_gpu_works(pool)`.
3. `Executor.collect_finished_works()` releases GPUs and sets `gpus_freed = True`.
4. On shutdown, `Executor.flush_pending_gpu_works()` pushes all pending GPU tasks back into the main `WorkQueue`.

**Relevant functions and parameters**
- `Executor.exec_loop(pool)`:
  - Defers GPU tasks into `pending_gpu_works`.
  - Only drains when `gpus_freed` is set by `collect_finished_works()`.
- `Executor.drain_pending_gpu_works(pool)`:
  - Starts GPU tasks in FIFO order and stops on the first task it cannot satisfy.
- `Executor.flush_pending_gpu_works()`:
  - Requeues in-memory deferred items into `wq` on shutdown.
- `Executor.pending_gpu_works` and `Executor.gpus_freed`:
  - Control deferral and event-driven draining.

---

## New or Modified Functions / Parameters

### Model A
- New field: `Executor.gpu_deferred_queue: WorkQueueOnFilesystem` (persistent GPU deferral).
- New field: `Executor.gpu_recently_released: bool` (event gate for deferred processing).
- New method: `Executor.process_gpu_deferred_queue(pool)` (processes persistent deferred tasks).
- New method: `Executor.handle_shutdown_deferred_queue()` (logs persisted deferred tasks on shutdown).
- Modified `Executor.busy` to include `gpu_deferred_queue.size()`.
- `Executor.exec_loop(pool)` now pushes deferred GPU tasks into `gpu_deferred_queue` when `acquire_gpu(...)` fails.

### Model B
- New field: `Executor.pending_gpu_works: List[WorkItem]` (in-memory pending list).
- New field: `Executor.gpus_freed: bool` (event gate for draining).
- New method: `Executor.drain_pending_gpu_works(pool)`.
- New method: `Executor.flush_pending_gpu_works()`.
- Modified `Executor.exec_loop(pool)` to defer GPU tasks into `pending_gpu_works` and to call `drain_pending_gpu_works(...)` after `collect_finished_works()`.
- Modified GPU downgrade path to set `item._gpu_limit` directly.

---

## Tests Added / Modified

### Model A (A/tests/test_gpu_deferred_queue.py)
- `test_gpu_deferred_queue_is_persistent`: verifies `Executor.gpu_deferred_queue` uses `WorkQueueOnFilesystem` and the directory exists.
- `test_task_deferred_when_no_gpu_available`: verifies a GPU task can be pushed into `gpu_deferred_queue` when GPU quota is zero.
- `test_event_driven_processing`: checks the `gpu_recently_released` flag behavior (but it manually toggles the flag instead of running `collect_finished_works()`).
- `test_no_wasteful_polling_when_queue_empty`: checks `process_gpu_deferred_queue(...)` returns quickly when queue is empty (timing-based and may be flaky).
- `test_deferred_task_runs_when_gpu_available`: verifies `process_gpu_deferred_queue(...)` starts a task and assigns GPUs when available.
- `test_duplicate_task_handling`: verifies a deferred item with a key already in `running_works` is skipped.
- `test_shutdown_with_deferred_tasks`: checks `handle_shutdown_deferred_queue()` does not drop persisted tasks.
- `test_busy_property_includes_deferred_queue`: verifies `Executor.busy` uses `gpu_deferred_queue.size()`.
- `test_requeue_failure_handling`: simulates re-enqueue failure and expects failed items to be pushed to `cq`.
- `test_batch_processing_limit`: checks the deferred queue pop is limited by `ctx.usable_cpu_count * 2`.
- `test_gpu_quota_management`: sanity checks `acquire_gpu(...)` and `release_gpu(...)` quotas.
- `test_persistence_survives_recreation`: verifies deferred tasks survive executor recreation (crash recovery).

### Model B (B/tests/test_gpu_deferral.py)
- `test_gpu_task_deferred_when_no_quota`: verifies a GPU task is appended to `pending_gpu_works` when `acquire_gpu(...)` fails.
- `test_pending_gpu_works_drained_after_gpu_freed`: verifies `drain_pending_gpu_works(...)` starts a deferred GPU task after a GPU is released.
- `test_drain_stops_at_first_unsatisfiable_task`: verifies FIFO ordering and stop-on-block behavior in `drain_pending_gpu_works(...)`.
- `test_gpus_freed_flag_set_by_collect`: verifies `collect_finished_works()` sets `gpus_freed` on GPU release.
- `test_gpus_freed_flag_not_set_when_no_gpu_work_finishes`: verifies CPU-only completion does not set `gpus_freed`.
- `test_cpu_task_runs_while_gpu_task_pending`: verifies CPU tasks still enter `running_works` while GPU tasks wait.
- `test_flush_pushes_items_back_to_wq`: verifies `flush_pending_gpu_works()` requeues deferred tasks on shutdown.
- `test_flush_is_noop_when_queue_empty`: verifies flush is safe when no pending tasks exist.
- `test_downgrade_does_not_crash`: verifies the `_gpu_limit` downgrade path works.
- `test_no_gpu_system_defers_gpu_tasks`: checks the downgrade path sets `gpu_limit` to 0 on a no-GPU system.
- `test_cpu_task_runs_on_no_gpu_system`: verifies CPU tasks still run with zero GPUs.
- `test_fifo_order_preserved`: verifies FIFO ordering with partial GPU quota.
- `test_tasks_spread_across_gpus`: verifies multi-GPU draining behavior.

---

## Model A — Pros and Cons (with function references)

**Pros**
- `Executor.gpu_deferred_queue` uses `WorkQueueOnFilesystem`, so deferred GPU tasks are not lost on crash or restart.
- `Executor.process_gpu_deferred_queue(pool)` uses a persistent queue, which satisfies the safety requirement in the prompt.
- `Executor.collect_finished_works()` sets `gpu_recently_released`, so deferred processing is event-driven and avoids tight retry loops inside `process_gpu_deferred_queue(...)`.

**Cons**
- `Executor.exec_loop(pool)` only calls `process_gpu_deferred_queue(pool)` when `gpu_recently_released` is True. After a restart, `gpu_recently_released` is False, so persisted tasks in `gpu_deferred_queue` can be stuck forever.
- `Executor.process_gpu_deferred_queue(pool)` re-enqueues unsatisfied tasks to the back of the persistent queue, which can reorder GPU tasks and break FIFO fairness.
- Several tests in `A/tests/test_gpu_deferred_queue.py` are weak or flaky (e.g., `test_no_wasteful_polling_when_queue_empty` is timing-based, and `test_event_driven_processing` toggles `gpu_recently_released` manually instead of exercising `collect_finished_works()`).

---

## Model B — Pros and Cons (with function references)

**Pros**
- `Executor.drain_pending_gpu_works(pool)` enforces FIFO order and stops at the first unsatisfied task, which keeps GPU task ordering predictable.
- `Executor.flush_pending_gpu_works()` requeues deferred GPU tasks back into `wq` on shutdown, so tasks are not silently lost when the executor exits normally.
- Tests in `B/tests/test_gpu_deferral.py` cover FIFO ordering, multi-GPU behavior, and CPU tasks proceeding during GPU deferral.

**Cons**
- `Executor.pending_gpu_works` is in-memory only; if the executor crashes (not a clean shutdown), deferred tasks are lost.
- `Executor.exec_loop(pool)` does not sleep when `pending_gpu_works` is non-empty and no GPUs are freed, which causes a tight retry loop and CPU load.
- Tests do not cover the busy-loop risk; `TestGpuDeferralNoSpin` only checks the `gpus_freed` flag, not actual loop throttling.

---

## PR Readiness (by model)

**Model A — PR status: Not ready**
- Persistence is good, but `process_gpu_deferred_queue(pool)` is never triggered on restart because `gpu_recently_released` is False. This can strand tasks indefinitely.
- Tests have several weak spots and do not validate the restart recovery path inside `exec_loop(...)`.

**Model B — PR status: Not ready**
- The busy-loop risk in `Executor.exec_loop(pool)` violates the “no forever retry loop” requirement.
- Crash safety is missing because `pending_gpu_works` is in-memory only.

---

## Concrete Next-Turn Fixes

**Model A improvements**
- In `Executor.exec_loop(pool)`, add a startup pass that calls `process_gpu_deferred_queue(pool)` at least once, or set `gpu_recently_released = True` when `gpu_deferred_queue.size() > 0` and `available_gpu_quota > 0`.
- Add a FIFO policy inside `process_gpu_deferred_queue(pool)` (stop on first unsatisfied task) to avoid reordering.
- Replace timing-based checks in `test_no_wasteful_polling_when_queue_empty` with a direct assertion on calls or counters.

**Model B improvements**
- Add a sleep or backoff path in `Executor.exec_loop(pool)` when `pending_gpu_works` is non-empty and `gpus_freed` is False, to avoid busy looping.
- Add persistence for deferred GPU tasks (for example, a `WorkQueueOnFilesystem` like Model A or a requeue strategy after N retries).
- Add a test that verifies no busy-looping by asserting that the loop sleeps when only `pending_gpu_works` remain.

---

## Evaluation Table

| Question of which is / has | Answer Given | Justification Why? |
| --- | --- | --- |
| Overall Better Solution | A slightly better than B | `Executor.gpu_deferred_queue` in Model A solves crash safety, which Model B lacks. |
| Better logic and correctness | Tie (both have blockers) | Model A can strand tasks on restart; Model B can spin in `Executor.exec_loop(pool)` when only `pending_gpu_works` exist. |
| Better Naming and Clarity | Tie | Both use clear names like `gpu_deferred_queue` / `pending_gpu_works` and `gpus_freed`. |
| Better Organization and Clarity | A slightly better than B | `process_gpu_deferred_queue(pool)` keeps deferred-queue logic separate, while B mixes logic in `exec_loop(...)`. |
| Better Interface Design | Tie | No public API changes; behavior is internal. |
| Better error handling and robustness | A slightly better than B | A persists deferred tasks via `WorkQueueOnFilesystem`, but B loses tasks on crash. |
| Better comments and documentation | Tie | Both add basic comments, no major documentation updates. |
| Ready for review / merge | Tie (not ready) | Model A has restart deadlock risk; Model B has busy-loop risk and crash-loss risk. |

---

## Overall Preference Justification

Model A is slightly better because `Executor.gpu_deferred_queue` in `Executor.__init__(...)` keeps deferred GPU tasks on disk, which is a core requirement from the prompt. Model B uses `pending_gpu_works` in memory and only requeues tasks in `flush_pending_gpu_works()` on shutdown, so a crash still loses work. Both models still have critical gaps: Model A can strand deferred tasks after restart because `gpu_recently_released` is False in `Executor.exec_loop(pool)`, and Model B can spin when only deferred tasks exist because `exec_loop(...)` does not sleep in that case. If these issues are fixed, Model A would be closer to meeting the safety and no-spin requirements.
