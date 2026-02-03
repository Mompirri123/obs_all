**Scope**
Review of Model A vs Model B changes for duplicate filename handling in `smallpond/execution/task.py` and tests in `tests/test_execution.py`.

**Model A**
**Flow (duplicate filename handling)**
1. A user configures `DataSinkNode.__init__(..., type=..., manifest_only=..., output_path=...)` in `smallpond/logical/node.py`.
2. `DataSinkNode.create_task(...)` decides the execution shape.
3. For `type="copy"` or `type="link_or_copy"`, `DataSinkNode.create_task(...)` calls `DataSinkNode._create_phase1_task(...)` and then creates a phase‑2 `DataSinkTask(..., type="link_manifest")`.
4. For `type="link"`, `DataSinkNode.create_task(...)` creates a single `DataSinkTask(..., type="link_manifest")`.
5. For `type="manifest"`, `DataSinkNode.create_task(...)` creates a single `DataSinkTask(..., type="manifest")`.
6. `DataSinkTask.run()` in `smallpond/execution/task.py` calls `DataSinkTask.collect_output_files(...)`.
7. `DataSinkTask.collect_output_files(...)` builds `src_paths` from `dataset.resolved_paths` of `self.input_datasets`.
8. It detects duplicate filenames using `set(p.name for p in src_paths)`.
9. If duplicates exist, it renames by appending a hash in `DataSinkTask.collect_output_files(...)`: `f"{p.stem}.{hashlib.sha256(str(p).encode()).hexdigest()[:16]}{p.suffix}"`.
10. It applies the renamed `dst_paths` only when `sink_type` is `"copy"`, `"link_or_copy"`, or `"link_manifest"` via `create_link_or_copy(...)`. For `sink_type == "manifest"`, it keeps `output_paths = src_paths` and only writes the manifest file.

**New or modified functions/parameters**
- Modified `DataSinkTask.collect_output_files(...)` in `smallpond/execution/task.py` to use `hashlib.sha256(...)` for deterministic suffixes.
- Added `import hashlib` in `smallpond/execution/task.py`.
- Tests updated in `tests/test_execution.py`.

**Tests added/updated and what they prove**
- Added `TestExecution.test_data_sink_deterministic_filenames` in `tests/test_execution.py`.
  - Proves that `hashlib.sha256(str(p).encode())[:16]` in `DataSinkTask.collect_output_files(...)` is deterministic for the same `Path` list.
  - Edge case: order independence by shuffling `src_paths` and checking each `Path` maps to the same output name.
  - Edge case: uniqueness by asserting `len(set(result1)) == len(result1)`.
- Kept and strengthened `TestExecution.test_data_sink_avoid_filename_conflicts` with explicit uniqueness checks on `link_path` and `copy_path` outputs.

**Pros**
- Deterministic per‑path naming in `DataSinkTask.collect_output_files(...)` because `hashlib.sha256(str(p).encode())` is order‑independent.
- Uniqueness is guaranteed by the hash suffix in `DataSinkTask.collect_output_files(...)`.
- Test coverage explicitly checks determinism and uniqueness in `TestExecution.test_data_sink_deterministic_filenames`.

**Cons**
- The hash is based on `str(p)` in `DataSinkTask.collect_output_files(...)`, so if the full path changes between runs (for example, different staging roots), the output name still changes even if the logical input is the same.
- The test `TestExecution.test_data_sink_deterministic_filenames` re‑implements the logic instead of calling `DataSinkTask.collect_output_files(...)`, so it is a logic‑level test, not an integration test.

**PR readiness**
- Not fully ready. It fixes determinism, but I would want one more test that exercises `DataSinkTask.collect_output_files(...)` end‑to‑end for duplicate filenames.

**Concrete next‑turn fixes**
1. Factor the rename logic into a helper like `DataSinkTask._resolve_duplicate_filenames(...)` and call it from `collect_output_files(...)` and tests.
2. Add an integration‑style test that creates a small `ParquetDataSet` with duplicate filenames and asserts stable output names across runs.
3. Consider hashing only `Path.name` (not full path) if the requirement is “same logical input file name => same output name”.

**Model B**
**Flow (duplicate filename handling)**
1. A user configures `DataSinkNode.__init__(..., type=..., manifest_only=..., output_path=...)` in `smallpond/logical/node.py`.
2. `DataSinkNode.create_task(...)` chooses single‑phase or two‑phase `DataSinkTask` creation based on `type`.
3. `DataSinkTask.run()` calls `DataSinkTask.collect_output_files(...)` in `smallpond/execution/task.py`.
4. `DataSinkTask.collect_output_files(...)` builds `src_paths` from `dataset.resolved_paths` of `self.input_datasets`.
5. It detects duplicate filenames using `set(p.name for p in src_paths)`.
6. If duplicates exist, it computes a deterministic index by sorting: `sorted_paths = sorted(src_paths, key=lambda p: str(p))` and `path_to_idx = {str(p): idx for idx, p in enumerate(sorted_paths)}`.
7. It renames with `f"{p.stem}.{path_to_idx[str(p)]}{p.suffix}"` inside `DataSinkTask.collect_output_files(...)`.
8. It applies `dst_paths` for `sink_type` in `("copy", "link_or_copy", "link_manifest")`. For `sink_type == "manifest"`, it keeps `output_paths = src_paths` and writes a manifest file.

**New or modified functions/parameters**
- Modified `DataSinkTask.collect_output_files(...)` in `smallpond/execution/task.py` to use deterministic index assignment.
- Removed `import hashlib` from `smallpond/execution/task.py`.
- Tests updated in `tests/test_execution.py`.

**Tests added/updated and what they prove**
- Added `TestExecution.test_data_sink_deterministic_filename_on_conflict` in `tests/test_execution.py`.
  - Proves that the index mapping derived from sorted paths is deterministic across shuffled input order.
  - Edge case: expected sorted order is asserted explicitly to lock in the sorting rule.
- `TestExecution.test_data_sink_avoid_filename_conflicts` no longer checks uniqueness of output filenames, so uniqueness is not asserted in B.

**Pros**
- Deterministic across ordering changes because of `sorted_paths` in `DataSinkTask.collect_output_files(...)`.
- Avoids the runtime cost and opacity of `hashlib.sha256(...)`.

**Cons**
- The index comes from the full `sorted_paths` list in `DataSinkTask.collect_output_files(...)`, so adding or removing other files changes indexes for all later files (global renumbering).
- Determinism still depends on `str(p)`; if full paths change between runs and the sort order changes, names can still change.
- Test coverage regresses by removing the uniqueness checks in `TestExecution.test_data_sink_avoid_filename_conflicts`.
- The new test `TestExecution.test_data_sink_deterministic_filename_on_conflict` re‑implements the same algorithm instead of calling `DataSinkTask.collect_output_files(...)` directly.

**PR readiness**
- Not fully ready. The logic fix is good, but the test coverage regression should be fixed before merge.

**Concrete next‑turn fixes**
1. Restore the uniqueness assertions in `TestExecution.test_data_sink_avoid_filename_conflicts`.
2. Add an integration‑style test that runs `DataSinkTask.collect_output_files(...)` on real temporary files to verify deterministic names and uniqueness together.
3. Consider limiting renaming to only duplicated `Path.name` entries to avoid renaming unique files when a single conflict exists.

**Comparison table**
| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              | Model A      | `DataSinkTask.collect_output_files(...)` in A makes names deterministic without global renumbering; it also keeps stronger tests. |
| Better logic and correctness         | Model A      | A avoids index renumbering in `DataSinkTask.collect_output_files(...)` and produces stable per‑path names; B’s index shifts when the set changes. |
| Better Naming and Clarity            | Model B      | Index‑based output names in `DataSinkTask.collect_output_files(...)` are shorter and easier to read than hash suffixes. |
| Better Organization and Clarity      | Tie          | Both changes are localized to `DataSinkTask.collect_output_files(...)` with similar structure. |
| Better Interface Design              | Tie          | Neither model changes `DataSinkNode.__init__(...)` or task parameters; the interface stays the same. |
| Better error handling and robustness | Model A      | Hash suffix in `DataSinkTask.collect_output_files(...)` is independent of other files, so adding/removing paths does not renumber existing outputs. |
| Better comments and documentation    | Model B      | The added comments in `DataSinkTask.collect_output_files(...)` explicitly explain determinism and order independence. |
| Ready for review / merge             | Model A      | A keeps uniqueness assertions and adds a determinism test; B removes a uniqueness check and should restore it. |

**Overall justification**
With your clarified intent, the goal is to fix the unstable naming; index‑based naming is allowed if it is stable. Model A updates `DataSinkTask.collect_output_files(...)` to a deterministic per‑path hash, which avoids global renumbering and keeps stronger tests for uniqueness and determinism. Model B fixes order instability but still uses a global index, so adding/removing other files can change names for unchanged paths. Backward compatibility is not a concern here, so Model A’s naming change is acceptable. The main remaining gap in A is that it hashes `str(p)`, so different absolute paths for the same logical file can still change names; that can be refined if needed.

**Test status**
I did not run tests in this review.
