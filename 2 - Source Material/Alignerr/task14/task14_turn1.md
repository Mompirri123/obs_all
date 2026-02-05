**Scope**
Review Model A and Model B for:
“Fix the issue where the cleanup is improper. Make sure partial outputs and temp directories are removed properly when a task is terminated. Add tests to check for this.”
Focus: `smallpond/execution/task.py` termination cleanup and related tests.

**Model A**
**Flow (how the option is applied)**
1. `Session.shutdown()` in `smallpond/dataframe.py` writes job status based on `finished` and `_terminated`.
2. If `_terminated` is `True`, `Session.shutdown()` logs “terminated” and **does not** call `RuntimeContext.cleanup(...)` or `RuntimeContext.cleanup_partial(...)`.
3. `Task.cleanup()` in `smallpond/execution/task.py` only disables profiling and calls `Task.clean_complex_attrs()`.
4. There is **no** `Task.terminate()` implementation in `smallpond/execution/task.py` to remove `Task.runtime_output_abspath` or `Task.temp_abspath` when a task is terminated.
5. Executor termination flow (`Executor.exec_loop()` → `StopWorkItem` → `SimplePoolTask.terminate()`) kills the worker process but does not invoke `Task.cleanup()` or any per‑task cleanup.

**New or modified functions/parameters**
- `Session._terminated` flag in `smallpond/dataframe.py` and shutdown logic that preserves runtime directories when terminated.
- No new termination cleanup functions in `smallpond/execution/task.py`.

**Tests added/updated and what they prove**
- `tests/test_session.py`:
  - `test_shutdown_no_cleanup_on_termination` proves `Session.shutdown()` preserves `queue_root`, `staging_root`, and `temp_root` when `_terminated` is set.
  - `test_shutdown_no_cleanup_on_termination_with_partial_output` proves partial outputs in `staging_root` are preserved when terminated.
  - `test_terminated_flag_prevents_cleanup_even_when_tasks_finished` proves `_terminated` overrides successful completion and prevents cleanup.

**Pros**
- Clear explicit behavior in `Session.shutdown()` for termination via `_terminated` flag.
- Tests document preservation behavior on termination in `tests/test_session.py`.

**Cons**
- The prompt requires cleanup on task termination, but `Session.shutdown()` in `smallpond/dataframe.py` explicitly **avoids** cleanup when `_terminated` is `True`.
- `Task.cleanup()` in `smallpond/execution/task.py` does not remove `runtime_output_abspath` or `temp_abspath` for terminated tasks.
- There is no `Task.terminate()` in `smallpond/execution/task.py`, so the executor’s termination path (`SimplePoolTask.terminate()`) does not clean per‑task outputs.
- Tests validate preserving partial outputs on termination, which is opposite of the prompt.

**PR readiness**
- Not ready. Model A implements preservation on termination, not cleanup.

**Concrete next‑turn fixes**
1. Add `Task.terminate()` in `smallpond/execution/task.py` to remove `runtime_output_abspath` and `temp_abspath`.
2. Update the executor termination path to call a cleanup hook after `SimplePoolTask.terminate()`.
3. Replace termination‑preservation tests with tests that assert cleanup on termination.

**Model B**
**Flow (how the option is applied)**
1. `Session.shutdown()` in `smallpond/dataframe.py` writes job status based on `finished`.
2. If `finished` is `False`, `Session.shutdown()` calls `RuntimeContext.cleanup_partial()` which removes `queue_root`, `temp_root`, and `staging_root` but preserves `output_root` and `log_root`.
3. `Task.cleanup()` in `smallpond/execution/task.py` calls `Task._cleanup_partial_output()` when `WorkStatus` is `FAILED` or `CRASHED`, removing `runtime_output_abspath` and `temp_abspath`.
4. Executor termination (`StopWorkItem` → `SimplePoolTask.terminate()`) still kills the worker process and does not directly call a `Task.terminate()` method.

**New or modified functions/parameters**
- `RuntimeContext.cleanup_partial()` in `smallpond/execution/task.py` (already present) is used from `Session.shutdown()`.
- `Task._cleanup_partial_output()` in `smallpond/execution/task.py` removes partial output and temp on crash/failure.

**Tests added/updated and what they prove**
- `tests/test_session.py`:
  - `test_shutdown_cleanup_partial_on_failure` verifies `Session.shutdown()` calls `RuntimeContext.cleanup_partial()` when tasks fail.
  - `test_cleanup_partial_removes_temp_and_staging` verifies `RuntimeContext.cleanup_partial()` removes `queue_root`, `temp_root`, and `staging_root` but not `output_root` or `log_root`.
  - `test_task_cleanup_removes_partial_output_on_crash` verifies `Task._cleanup_partial_output()` is triggered for `WorkStatus.CRASHED` and removes `runtime_output_abspath` and `temp_abspath`.
  - `test_task_cleanup_preserves_output_on_success` verifies successful tasks keep outputs.

**Pros**
- `Task._cleanup_partial_output()` in `smallpond/execution/task.py` ensures partial outputs and temp directories are removed on crash/failure.
- `Session.shutdown()` cleans runtime directories on failure via `RuntimeContext.cleanup_partial()`.
- Tests validate runtime and task‑level cleanup behavior on failure/crash.

**Cons**
- The prompt is specifically about **task termination**, but there is no `Task.terminate()` hook in `smallpond/execution/task.py` and the executor kill path does not trigger cleanup.
- Cleanup on termination is only indirectly covered as “failure” in `Session.shutdown()`, not a per‑task termination flow.
- No test explicitly simulates a `StopWorkItem` termination and asserts cleanup of `Task.runtime_output_abspath` and `Task.temp_abspath`.

**PR readiness**
- Partially ready. The failure/crash cleanup is correct, but task termination cleanup is not explicitly handled or tested.

**Concrete next‑turn fixes**
1. Add `Task.terminate()` in `smallpond/execution/task.py` and call it from the executor termination path after `SimplePoolTask.terminate()`.
2. Add a test that simulates termination (e.g., set `WorkStatus.EXEC_FAILED` or directly call `Task.terminate()`) and verifies `runtime_output_abspath` and `temp_abspath` are removed.
3. Clarify in `Session.shutdown()` comments that termination should use the same cleanup as failure for task outputs.

**Comparison table**
| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              | -3 (Model B) | Model B at least removes partial outputs on failure/crash via `Task._cleanup_partial_output()` and `RuntimeContext.cleanup_partial()`. Model A preserves partial outputs on termination. |
| Better logic and correctness         | -3 (Model B) | `Task.cleanup()` in B removes `runtime_output_abspath` and `temp_abspath` for crashes; A does not. |
| Better Naming and Clarity            | -1 (Model B) | `cleanup_partial()` and `_cleanup_partial_output()` in B are aligned with their behavior; A’s termination behavior conflicts with the prompt intent. |
| Better Organization and Clarity      | -1 (Model B) | Cleanup logic is centralized in `RuntimeContext.cleanup_partial()` and `Task._cleanup_partial_output()` in B. |
| Better Interface Design              | 0 (Tie)      | Neither model adds a `Task.terminate()` API, so interface coverage is incomplete in both. |
| Better error handling and robustness | -2 (Model B) | B cleans partial outputs on failure/crash; A leaves them in more cases. |
| Better comments and documentation    | -1 (Model B) | B’s tests document cleanup behavior, while A’s tests document preservation, which is contrary to the prompt. |
| Ready for review / merge             | -2 (Model B) | B is closer but still missing explicit task‑termination cleanup tests and hook. |

**Overall justification**
Model B is closer to the prompt because it cleans partial outputs and temp directories on failure/crash via `Task._cleanup_partial_output()` and `RuntimeContext.cleanup_partial()`. Model A explicitly keeps partial outputs on termination in `Session.shutdown()`, which is the opposite of the requested behavior. Neither model fully implements task‑termination cleanup because there is no `Task.terminate()` hook in `smallpond/execution/task.py` or a test that validates cleanup for a terminated task. Backward compatibility is not a concern here, so there is no penalty for changing cleanup behavior.

**Test status**
I did not run tests in this review.
