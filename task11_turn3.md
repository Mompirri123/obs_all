# Turn 2 Evaluation — GPU Deferral Follow‑ups (Model A vs Model B)

## Process Flow and Function-Level Notes

### Shared baseline behavior (both models)
- GPU gating still happens in `Executor.exec_loop(...)` based on `WorkItem.gpu_limit`.
- GPU quota is still managed by `Executor.acquire_gpu(quota)` and `Executor.release_gpu(...)`, with releases triggered in `Executor.collect_finished_works()`.
- No public API changes; all behavior is internal to `Executor`.

### Model A — How it works (flow)
1. `Executor.__init__(...)` creates a persistent `WorkQueueOnFilesystem` at `gpu_deferred_queue` and a flag `gpu_recently_released`.
2. `Executor.exec_loop(pool)`:
   - When `gpu_recently_released` is True, it calls `Executor.process_gpu_deferred_queue(pool)` and resets the flag.
   - It pops new work from `WorkQueue.pop(...)` and iterates each `WorkItem`.
   - If `WorkItem.gpu_limit > 0` and `Executor.acquire_gpu(...)` fails, the item is pushed into `Executor.gpu_deferred_queue`.
3. `Executor.collect_finished_works()` releases GPUs and sets `gpu_recently_released = True`.
4. On shutdown, `Executor.handle_shutdown_deferred_queue()` logs how many deferred tasks remain in the persistent queue.

**Relevant functions and parameters**
- `Executor.exec_loop(pool)`:
  - Calls `process_gpu_deferred_queue(pool)` only when `gpu_recently_released` is True.
  - Defers GPU tasks by calling `gpu_deferred_queue.push(item)` when `acquire_gpu(...)` fails.
- `Executor.process_gpu_deferred_queue(pool)`:
  - Pulls items from the persistent queue, retries `acquire_gpu(...)`, and re‑enqueues when still blocked.
- `Executor.gpu_deferred_queue`:
  - Persistent, on‑disk queue for deferred GPU tasks.
- `Executor.gpu_recently_released`:
  - Event flag that gates deferred processing.

### Model B — How it works (flow)
1. `Executor.__init__(...)` creates an in‑memory list `pending_gpu_works` and a flag `gpus_freed`.
2. `Executor.exec_loop(pool)`:
   - Pops new work from `WorkQueue.pop(...)`.
   - If `WorkItem.gpu_limit > 0` and `Executor.acquire_gpu(...)` fails, the item is appended to `pending_gpu_works`.
   - CPU-only tasks are still submitted to the pool via `pool.apply_async(...)`.
   - After `collect_finished_works()` runs, if `gpus_freed` is True, it calls `drain_pending_gpu_works(pool)`.
3. `Executor.collect_finished_works()` releases GPUs and sets `gpus_freed = True`.
4. On shutdown, `Executor.flush_pending_gpu_works()` pushes pending GPU tasks back into `wq`.

**Relevant functions and parameters**
- `Executor.exec_loop(pool)`:
  - Defers GPU tasks into `pending_gpu_works`.
  - Only drains when `gpus_freed` is set by `collect_finished_works()`.
- `Executor.drain_pending_gpu_works(pool)`:
  - Starts GPU tasks in FIFO order and stops on the first task it cannot satisfy.
- `Executor.flush_pending_gpu_works()`:
  - Requeues in‑memory deferred items into `wq` on shutdown.

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
- New field: `Executor.pending_gpu_works: List[WorkItem]` (in‑memory pending list).
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
- `test_event_driven_processing`: checks the `gpu_recently_released` flag, but it toggles the flag manually instead of using `collect_finished_works()`.
- `test_no_wasteful_polling_when_queue_empty`: timing-based check that `process_gpu_deferred_queue(...)` returns quickly (can be flaky).
- `test_deferred_task_runs_when_gpu_available`: verifies `process_gpu_deferred_queue(...)` starts a task and assigns GPUs when available.
- `test_duplicate_task_handling`: verifies a deferred item with a key already in `running_works` is skipped.
- `test_shutdown_with_deferred_tasks`: checks `handle_shutdown_deferred_queue()` does not drop persisted tasks.
- `test_busy_property_includes_deferred_queue`: verifies `Executor.busy` uses `gpu_deferred_queue.size()`.
- `test_requeue_failure_handling`: simulates re‑enqueue failure and expects failed items to be pushed to `cq`.
- `test_batch_processing_limit`: checks deferred queue pop is limited by `ctx.usable_cpu_count * 2`.
- `test_gpu_quota_management`: sanity checks `acquire_gpu(...)` and `release_gpu(...)` quotas.
- `test_persistence_survives_recreation`: verifies deferred tasks survive executor recreation.

### Model B (B/tests/test_gpu_deferral.py)
- `test_gpu_task_deferred_when_no_quota`: verifies a GPU task is appended to `pending_gpu_works` when `acquire_gpu(...)` fails.
- `test_pending_gpu_works_drained_after_gpu_freed`: verifies `drain_pending_gpu_works(...)` starts a deferred GPU task after GPU release.
- `test_drain_stops_at_first_unsatisfiable_task`: verifies FIFO ordering and stop-on-block behavior in `drain_pending_gpu_works(...)`.
- `test_gpus_freed_flag_set_by_collect`: verifies `collect_finished_works()` sets `gpus_freed` on GPU release.
- `test_gpus_freed_flag_not_set_when_no_gpu_work_finishes`: verifies CPU-only completion does not set `gpus_freed`.
- `test_cpu_task_runs_while_gpu_task_pending`: verifies CPU tasks still enter `running_works` while GPU tasks wait.
- `test_flush_pushes_items_back_to_wq`: verifies `flush_pending_gpu_works()` requeues deferred tasks on shutdown.
- `test_flush_is_noop_when_queue_empty`: verifies flush is safe when no pending tasks exist.
- `test_downgrade_does_not_crash`: verifies `_gpu_limit` downgrade path works.
- `test_no_gpu_system_defers_gpu_tasks`: checks the downgrade path sets `gpu_limit` to 0 on a no‑GPU system.
- `test_cpu_task_runs_on_no_gpu_system`: verifies CPU tasks still run with zero GPUs.
- `test_fifo_order_preserved`: verifies FIFO ordering with partial GPU quota.
- `test_tasks_spread_across_gpus`: verifies multi‑GPU draining behavior.

---

## Model A — Pros and Cons (with function references)

**Pros**
- `Executor.gpu_deferred_queue` uses `WorkQueueOnFilesystem`, so deferred GPU tasks are not lost on crash or restart.
- `Executor.collect_finished_works()` sets `gpu_recently_released`, so `Executor.process_gpu_deferred_queue(pool)` is event‑driven and avoids busy polling.

**Cons**
- `Executor.exec_loop(pool)` only calls `process_gpu_deferred_queue(pool)` when `gpu_recently_released` is True. After restart, `gpu_recently_released` is False, so tasks in `gpu_deferred_queue` can be stuck forever.
- `Executor.process_gpu_deferred_queue(pool)` re‑enqueues unsatisfied tasks to the back of the queue, so FIFO ordering among GPU tasks is not guaranteed.
- `test_event_driven_processing` sets `gpu_recently_released` manually instead of using `collect_finished_works()`, so it does not validate the real code path.
- `test_no_wasteful_polling_when_queue_empty` uses timing for correctness, which is brittle on slow or loaded CI.

---

## Model B — Pros and Cons (with function references)

**Pros**
- `Executor.drain_pending_gpu_works(pool)` enforces FIFO order and stops at the first unsatisfied task, so ordering is predictable.
- `Executor.flush_pending_gpu_works()` pushes deferred tasks back into `wq` on shutdown, so tasks are not lost on clean exit.
- Tests in `B/tests/test_gpu_deferral.py` cover FIFO order, multi‑GPU behavior, and CPU tasks continuing when GPU tasks are pending.

**Cons**
- `Executor.pending_gpu_works` is in memory only, so deferred tasks are lost on crash (no persistence like `WorkQueueOnFilesystem`).
- `Executor.exec_loop(pool)` does not sleep when `pending_gpu_works` is non‑empty and no GPUs are freed, so it can spin and load CPU.
- Tests do not cover the busy‑loop risk; they do not assert any sleep or throttling behavior in `exec_loop(...)`.

---

## PR Readiness (by model)

**Model A — PR status: Not ready**
- Needs a startup path in `Executor.exec_loop(pool)` to process persisted work (call `process_gpu_deferred_queue(pool)` at loop start or set `gpu_recently_released = True` when `gpu_deferred_queue.size() > 0` and `available_gpu_quota > 0`).
- Needs FIFO preservation in `process_gpu_deferred_queue(pool)` to avoid reordering.
- Needs test fixes that use `collect_finished_works()` for flag changes and avoid timing‑based assertions.

**Model B — PR status: Not ready**
- Needs persistence for `pending_gpu_works` or a safe requeue mechanism on crash.
- Needs a sleep/backoff path in `Executor.exec_loop(pool)` when only `pending_gpu_works` remain and no GPUs were freed.

---

## Concrete Next‑Turn Fixes

**Model A improvements**
- In `Executor.exec_loop(pool)`, call `process_gpu_deferred_queue(pool)` at loop start, or set `gpu_recently_released = True` when `gpu_deferred_queue.size() > 0` and `available_gpu_quota > 0` to avoid stuck tasks after restart.
- Update `process_gpu_deferred_queue(pool)` to enforce FIFO (stop on first unsatisfied task instead of re‑enqueueing to the back).
- Replace `test_event_driven_processing` with a test that calls `collect_finished_works()` to flip `gpu_recently_released`.
- Replace timing‑based `test_no_wasteful_polling_when_queue_empty` with a deterministic assertion (e.g., verify `WorkQueueOnFilesystem.pop()` is not called when size is 0).

**Model B improvements**
- Add persistence for deferred GPU tasks (e.g., a `WorkQueueOnFilesystem` similar to `gpu_deferred_queue`).
- Add a sleep/backoff in `Executor.exec_loop(pool)` when `pending_gpu_works` is non‑empty and `gpus_freed` is False.
- Add a test that verifies the sleep/backoff path is used when only `pending_gpu_works` exist.

---

## Evaluation Table

| Question of which is / has | Answer Given | Justification Why? |
| --- | --- | --- |
| Overall Better Solution | A slightly better than B | Model A has persistence via `Executor.gpu_deferred_queue`, which matches the prompt’s safety requirement. |
| Better logic and correctness | Tie (both have blockers) | Model A can strand work after restart in `Executor.exec_loop(pool)`; Model B can spin in `Executor.exec_loop(pool)` when only `pending_gpu_works` exist. |
| Better Naming and Clarity | Tie | Both use clear names like `gpu_deferred_queue`, `pending_gpu_works`, `gpu_recently_released`, and `gpus_freed`. |
| Better Organization and Clarity | A slightly better than B | Model A separates deferred logic into `process_gpu_deferred_queue(pool)`; Model B keeps most logic in `exec_loop(pool)`. |
| Better Interface Design | Tie | No public API changes in either model. |
| Better error handling and robustness | A slightly better than B | Model A persists deferred tasks in `WorkQueueOnFilesystem`; Model B loses deferred tasks on crash. |
| Better comments and documentation | Tie | Both add minimal comments, no significant docs. |
| Ready for review / merge | Tie (not ready) | Model A needs restart handling and FIFO; Model B needs no‑spin and persistence. |

---

## Overall Preference Justification

Model A is slightly better because `Executor.gpu_deferred_queue` in `Executor.__init__(...)` persists deferred GPU tasks, which is the most important safety requirement. Model B’s `pending_gpu_works` is only in memory and is lost on crash, even though `flush_pending_gpu_works()` helps on clean shutdown. Both models still fail key parts of the follow‑up prompt: Model A does not process deferred tasks after restart because `gpu_recently_released` is False in `Executor.exec_loop(pool)`, and Model B can spin in `exec_loop(pool)` when only `pending_gpu_works` exist. If Model A adds a startup drain in `exec_loop(pool)` and FIFO behavior in `process_gpu_deferred_queue(pool)`, it should become the better, mergeable option.
