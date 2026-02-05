**Scope**
Prompt: fix improper cleanup on task termination so partial outputs and temp directories are removed, and add tests. Backward compatibility is not required by the prompt, so I do not penalize compatibility breaks. I only credit it if present.

**Model A Review**
**What Changed**
- `Task.run_on_ray()` changes the retry marker creation to an atomic `os.open(..., O_CREAT|O_EXCL)` loop in the inner `exec_task()` in `A/smallpond/execution/task.py`.
- `DataSinkTask.run()` now builds the pool with `ThreadPoolExecutor(min(32, max(1, len(self.input_datasets))))` in `A/smallpond/execution/task.py`.
- `DataSinkTask.collect_output_files()` no longer removes pre-existing files in `runtime_output_abspath` in `A/smallpond/execution/task.py`.
- `DataFrame.to_pandas()` and `DataFrame.to_arrow()` remove `ThreadPoolExecutor` use and run sequentially in `A/smallpond/dataframe.py`.

**Flow (Call Chain) As Implemented**
- Ray path: `Task.run_on_ray()` -> inner `exec_task()` -> atomic marker creation using `task.ray_marker_path` -> `task.exec()` -> `task.output` dump. This flow is in `A/smallpond/execution/task.py` and is unrelated to termination cleanup.
- Output sink path: `Task.exec()` (from `WorkItem.exec()` in `smallpond/execution/workqueue.py`) -> `DataSinkTask.run()` -> `DataSinkTask.collect_output_files()` using a pool sized by `len(self.input_datasets)`. No cleanup of `runtime_output_abspath` happens in this path in Model A.
- DataFrame path: `DataFrame.to_pandas()` / `DataFrame.to_arrow()` -> `self._compute()` -> sequential per-dataset conversion. This is unrelated to termination cleanup.

**New or Modified Functions / Parameters**
- Modified function body: `Task.run_on_ray()` inner `exec_task()` in `A/smallpond/execution/task.py`.
- Modified function body: `DataSinkTask.run()` in `A/smallpond/execution/task.py`.
- Modified function body: `DataSinkTask.collect_output_files()` in `A/smallpond/execution/task.py`.
- Modified function bodies: `DataFrame.to_pandas()` and `DataFrame.to_arrow()` in `A/smallpond/dataframe.py`.
- No new parameters added.

**Tests Added or Changed**
- None. There are no new or changed tests in `A/tests/` that assert cleanup on termination, partial output removal, or temp directory removal.

**Pros**
- The retry slot claim in `Task.run_on_ray()` is more race-safe because `os.open(..., O_CREAT|O_EXCL)` removes the time gap between `os.path.exists()` and `Path.touch()`.
- `DataSinkTask.run()` avoids `ThreadPoolExecutor(0)` by using `max(1, len(self.input_datasets))`, which prevents a runtime error when `input_datasets` is empty.
- The comment in `DataFrame.to_pandas()` and `DataFrame.to_arrow()` explains a real risk (DuckDB thread-safety).

**Cons**
- The prompt requirement is not implemented. There is no cleanup on termination path touching `Task.clean_output()` or removing `RuntimeContext.temp_root` for a terminated task. The flow in `Task.cleanup()` and `Scheduler.clean_temp_files()` is unchanged.
- `DataSinkTask.collect_output_files()` no longer removes pre-existing files. That can leave stale partial outputs in `runtime_output_abspath` and mix old and new data.
- The changes in `DataFrame.to_pandas()` and `DataFrame.to_arrow()` are unrelated to the prompt and add risk of performance regression without tests.
- No tests validate cleanup on termination, partial output removal, or temp directory removal.

**PR Readiness**
- Not ready. The core requirement (termination cleanup) is missing and no tests were added.

**Concrete Next-Turn Fixes**
1. Add a termination cleanup path. Example: in `WorkItem.exec()` or in the executor termination path, call `Task.clean_output(force=True)` for a task that ends with `WorkStatus.CRASHED` or executor termination, and remove `RuntimeContext.temp_root` and `RuntimeContext.staging_root` if the job is being shut down. Call sites should be explicit, for example in `Scheduler.clean_temp_files()`.
2. Ensure temp output directories are removed for terminated tasks: call `remove_path(task.temp_abspath)` inside a new `Task.cleanup_on_terminate()` or directly inside the termination handling path.
3. Add tests in `tests/test_execution.py` that:
- Simulate a terminated task (raise a controlled exception mid-run) and assert `task.runtime_output_abspath` and `task.final_output_abspath` are removed after cleanup.
- Simulate a job shutdown and assert `RuntimeContext.temp_root` and `RuntimeContext.staging_root` are removed.
- Edge case: `DataSinkTask` with pre-existing output files and subdirectories should be fully removed, not only top-level files.

**Model B Review**
**What Changed**
- `DataSinkTask.collect_output_files()` removes pre-existing files and symlinks from `runtime_output_abspath` before copying/linking outputs in `B/smallpond/execution/task.py`.

**Flow (Call Chain) As Implemented**
- Output sink path: `Task.exec()` (from `WorkItem.exec()` in `smallpond/execution/workqueue.py`) -> `DataSinkTask.run()` -> `DataSinkTask.collect_output_files()` -> cleanup of top-level files in `runtime_output_abspath` -> output collection and link/copy. This is the only cleanup added and it only runs when the `DataSinkTask` executes normally.

**New or Modified Functions / Parameters**
- Modified function body: `DataSinkTask.collect_output_files()` in `B/smallpond/execution/task.py`.
- No new parameters added.

**Tests Added or Changed**
- None. There are no new or changed tests in `B/tests/` that assert cleanup on termination, partial output removal, or temp directory removal.

**Pros**
- The cleanup loop in `DataSinkTask.collect_output_files()` reduces mixing of stale files when the output directory already exists, which partially addresses “partial outputs” for re-runs.
- The change is localized to the output sink and does not affect other task types.

**Cons**
- The prompt requirement is not implemented. There is no termination cleanup path that runs when a task is killed, so `Task.clean_output()` and `RuntimeContext.temp_root` are not guaranteed to be cleaned on termination.
- Cleanup only deletes top-level files and symlinks. It does not remove subdirectories inside `runtime_output_abspath`, so partial outputs in nested directories remain.
- No tests validate cleanup on termination, partial output removal, or temp directory removal.

**PR Readiness**
- Not ready. The core requirement (termination cleanup) is missing and no tests were added.

**Concrete Next-Turn Fixes**
1. Add explicit termination cleanup: ensure `Task.clean_output(force=True)` runs when a task is terminated, and remove `RuntimeContext.temp_root` during job shutdown.
2. Expand cleanup logic to remove directories, not only files, when clearing partial outputs. Use `remove_path(runtime_output_dir)` instead of iterating only files.
3. Add tests in `tests/test_execution.py` for termination cleanup and temp directory removal, including a case with nested output directories.

**Comparison Table**
| | ------------------------------------ | ------------ | ------------------- |
| question | which is better | reasoning / justification |
| Overall Better Solution | Model B (-3) | Model B at least attempts to remove stale outputs in `DataSinkTask.collect_output_files()`, while Model A does not address the cleanup requirement and removes that cleanup path entirely. |
| Better logic and correctness | Model B (-2) | The cleanup loop in `DataSinkTask.collect_output_files()` is closer to “remove partial outputs,” even though it is not on termination. Model A’s changes are mostly unrelated. |
| Better Naming and Clarity | Tie (0) | Both changes are small and the added comments are clear; no naming improvements were made. |
| Better Organization and Clarity | Tie (0) | No meaningful refactor or structure change in either model. |
| Better Interface Design | Tie (0) | No new parameters or interfaces were added. |
| Better error handling and robustness | Model B (-1) | Deleting stale outputs in `DataSinkTask.collect_output_files()` reduces mixed output risks. Model A improves unrelated robustness in `Task.run_on_ray()` but does not address cleanup. |
| Better comments and documentation | Tie (0) | Model A adds comments about DuckDB thread-safety, Model B adds comments about output cleanup. Both are fine but not for the prompt. |
| Ready for review / merge | Tie (0) | Both are not ready due to missing termination cleanup and missing tests. |

**Which Model Is Better and Why**
Model B is better overall because it at least touches the partial-output problem by clearing files in `DataSinkTask.collect_output_files()`. Model A does not implement termination cleanup at all and removes a cleanup step, plus it introduces unrelated changes in `DataFrame.to_pandas()` and `DataFrame.to_arrow()` that are outside the prompt. Neither model adds the required tests, so both are still not ready to merge.
