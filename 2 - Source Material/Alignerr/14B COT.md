# Turn 1

### summary:       

- **Problem**
	- When a task was terminated (failed or crashed), partial outputs and temp directories were not cleaned up:
	-  Task.cleanup() (task.py:907) only disabled profiling and cleaned complex attributes — it never removed the runtime_output_abspath or temp_abspath directories created during initialize()
	- Session.shutdown() (dataframe.py:163-164) did nothing on failure — it just logged a warning, leaving: 
		1. queue_root
		2. temp_root
		3. staging_root behind.
	
- Changes:
	- smallpond/execution/task.py
		- Task.cleanup(): Added a check for FAILED/CRASHED status to call _cleanup_partial_output(), which removes
		- runtime_output_abspath and temp_abspath.
		- RuntimeContext.cleanup_partial(): New method that removes queue_root, temp_root, and staging_root while
  preserving output_root and log_root for debugging.

  smallpond/dataframe.py

  - Session.shutdown(): The failure branch now calls self._runtime_ctx.cleanup_partial() instead of just logging
   a warning.

  tests/test_session.py

  - test_shutdown_cleanup_partial_on_failure: Updated existing test to verify that temp/staging/queue dirs are
  now cleaned on failure.
  - test_cleanup_partial_removes_temp_and_staging: New test verifying RuntimeContext.cleanup_partial() removes
  queue/temp/staging while preserving output/log.
  - test_task_cleanup_removes_partial_output_on_crash: New test verifying a crashed task's
  runtime_output_abspath and temp_abspath are cleaned up.
  - test_task_cleanup_preserves_output_on_success: New test verifying successful tasks preserve their output

✳ Newspapering… (6m 0s · ↓ 17.1k tokens · thinking)