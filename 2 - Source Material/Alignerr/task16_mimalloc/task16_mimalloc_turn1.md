# Task 16 mimalloc - Model A vs Model B Review (Simple English)

## Scope and score rule
- Prompt reviewed: "Fix the issue where WebAssembly apps do extra zeroing and lose performance during allocation-heavy work."
- I reviewed only the real code changes inside folders `A` and `B`, and I traced where those changes are used.
- Backward compatibility note: if a model keeps backward compatibility, I give credit. If it does not, I do not remove credit for this prompt.
- Score rule in table: `+4` means Model A clearly wins, `-4` means Model B clearly wins.

## Model A

### Changed file
- `A/src/prim/wasi/prim.c`

### Full flow (how this fix is used end to end)
1. Memory request starts in `mi_os_prim_alloc_at(..., bool* is_zero)` at `A/src/os.c:211`.
2. `mi_os_prim_alloc_at()` calls `_mi_prim_alloc(..., bool* is_zero, void** addr)` at `A/src/os.c:220`.
3. Model A changed `_mi_prim_alloc()` at `A/src/prim/wasi/prim.c:122` to set `*is_zero = true`.
4. That `is_zero` value is stored in `memid.initially_zero` by `_mi_memid_create_os(..., is_zero, ...)` at `A/src/os.c:332` and `A/include/mimalloc/internal.h:915`.
5. Then it is copied to `segment->free_is_zero` at `A/src/segment.c:886`.
6. Then it is copied to `page->free_is_zero` at `A/src/page.c:699`.
7. In `_mi_page_malloc_zero()` at `A/src/alloc.c:66`, if `page->free_is_zero` is true, mimalloc skips full `_mi_memzero_aligned(...)`. If false, it calls `_mi_memzero_aligned(...)` at `A/src/alloc.c:71`.
8. Segment metadata zeroing also uses this value: in `mi_segment_alloc()` at `A/src/segment.c:918`, metadata zeroing is skipped when `segment->memid.initially_zero` is true.

Extra path Model A also changed:
1. `_mi_os_commit_ex(..., bool* is_zero, ...)` calls `_mi_prim_commit(..., &os_is_zero)` at `A/src/os.c:460`.
2. Model A changed `_mi_prim_commit()` at `A/src/prim/wasi/prim.c:135` to set `*is_zero = true`.
3. This keeps commit path behavior in line with WASI zero-initialized memory behavior.

### New or modified functions/parameters (2 lines each)
- `_mi_prim_alloc(..., bool* is_zero, void** addr)` in `A/src/prim/wasi/prim.c:122`.
  - Changed `*is_zero` from `false` to `true`.
  - This is the main fix because it tells the rest of mimalloc that new WASI memory is already zero.

- `_mi_prim_commit(void* addr, size_t size, bool* is_zero)` in `A/src/prim/wasi/prim.c:135`.
  - Changed `*is_zero` from `false` to `true`.
  - This makes commit behavior match the same zero-memory assumption.

- Comment in `mi_memory_grow()` in `A/src/prim/wasi/prim.c:50`.
  - Removed uncertain text marker `(?)` in the comment.
  - No code behavior change.

### Tests added and what they prove
- Added tests: **none**.
- What new tests prove: **nothing new**, because no test was added.
- Missing important tests and edge cases:
  - A WASI test to prove `_mi_prim_alloc(..., is_zero)` makes `_mi_page_malloc_zero()` skip `_mi_memzero_aligned(...)` for first zero-alloc use.
  - A test for commit path to prove `_mi_prim_commit(..., is_zero)` stays correct for aligned/commit scenarios.

### Pros and cons (with exact function names)
Pros:
- Main issue fix is done in `_mi_prim_alloc(..., bool* is_zero, ...)` at `A/src/prim/wasi/prim.c:122`.
- Benefit reaches both block allocation and segment metadata through `memid.initially_zero` (`A/src/segment.c:886`, `A/src/segment.c:918`).
- `_mi_prim_commit(..., bool* is_zero)` is also updated at `A/src/prim/wasi/prim.c:135`, so behavior is more complete.

Cons:
- In `_mi_prim_alloc()` at `A/src/prim/wasi/prim.c:125-127`, `*is_zero` is set to true before checking success; if allocation fails, output can still be true.
- No tests were added, so future regressions are harder to catch.

### PR readiness and concrete next-turn fixes
- PR readiness: **close, but not safe to merge without tests**.
- Next-turn fixes:
1. Add WASI test to show `mi_zalloc()`/`mi_calloc()` path avoids extra zeroing when `page->free_is_zero` is true.
2. Add commit-path regression test for `_mi_prim_commit(..., bool* is_zero)`.
3. Small cleanup: in `_mi_prim_alloc()`, set `*is_zero` after success check.

---

## Model B

### Changed file
- `B/src/prim/wasi/prim.c`

### Full flow (how this fix is used end to end)
1. Memory request starts in `mi_os_prim_alloc_at(..., bool* is_zero)` at `B/src/os.c:211`.
2. It calls `_mi_prim_alloc(..., bool* is_zero, void** addr)`.
3. Model B changed `_mi_prim_alloc()` at `B/src/prim/wasi/prim.c:122` to set `*is_zero = (*addr != NULL)` after `mi_prim_mem_grow(...)`.
4. On success, that value is stored in `memid.initially_zero` (`B/src/os.c:332`, `B/include/mimalloc/internal.h:915`).
5. Then it goes to `segment->free_is_zero` (`B/src/segment.c:886`) and `page->free_is_zero` (`B/src/page.c:699`).
6. Then `_mi_page_malloc_zero()` at `B/src/alloc.c:66-71` can skip `_mi_memzero_aligned(...)` when page memory is already zero.
7. Segment metadata zeroing in `mi_segment_alloc()` at `B/src/segment.c:918` also benefits.
8. Model B did not change `_mi_prim_commit()` (`B/src/prim/wasi/prim.c:137-140`), so commit path still reports `*is_zero = false`.

### New or modified functions/parameters (2 lines each)
- `_mi_prim_alloc(..., bool* is_zero, void** addr)` in `B/src/prim/wasi/prim.c:122`.
  - Sets `*is_zero = (*addr != NULL)`.
  - This fixes the main allocation-time issue and gives cleaner output for failure case.

### Tests added and what they prove
- Added tests: **none**.
- What new tests prove: **nothing new**.
- Missing important tests and edge cases:
  - No WASI test to prove redundant zeroing is removed in allocation-heavy path.
  - No commit-path test, and `_mi_prim_commit(..., bool* is_zero)` still returns false.

### Pros and cons (with exact function names)
Pros:
- Main fix exists in `_mi_prim_alloc(..., bool* is_zero, void** addr)` at `B/src/prim/wasi/prim.c:122`.
- Failure handling for output is cleaner because `*is_zero` depends on `(*addr != NULL)`.

Cons:
- `_mi_prim_commit(..., bool* is_zero)` is unchanged at `B/src/prim/wasi/prim.c:137-140`, so behavior is less complete than Model A.
- No tests were added.

### PR readiness and concrete next-turn fixes
- PR readiness: **partly ready, but incomplete and not proven by tests**.
- Next-turn fixes:
1. Update `_mi_prim_commit(..., bool* is_zero)` in `B/src/prim/wasi/prim.c` to set true for WASI.
2. Add WASI regression test for `is_zero` flow to `page->free_is_zero` behavior.
3. Add a small benchmark or guard test for allocation-heavy zero-allocation workload.

---

## Comparison table (rating: +4 A wins, -4 B wins)

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution              | Model A (+2)        | Both fix the main `_mi_prim_alloc(..., is_zero)` point, but Model A also updates `_mi_prim_commit(..., is_zero)` so the full behavior is more complete. |
| Better logic and correctness         | Model A (+2)        | Model A changes both `_mi_prim_alloc()` and `_mi_prim_commit()`. Model B leaves commit path unchanged. |
| Better Naming and Clarity            | Model B (-1)        | In `_mi_prim_alloc()`, `*is_zero = (*addr != NULL)` is very clear and easy to read. |
| Better Organization and Clarity      | Tie (0)             | Both models make small, focused changes in one file. |
| Better Interface Design              | Model B (-1)        | Model B ties output value to success in `_mi_prim_alloc()`, which is cleaner output behavior. |
| Better error handling and robustness | Model B (-1)        | Model B avoids returning `is_zero=true` when allocation fails; Model A can do that in failure path. |
| Better comments and documentation    | Model A (+1)        | Model A adds clearer comments for WASI assumptions in `_mi_prim_alloc()` and `_mi_prim_commit()`. |
| Ready for review / merge             | Model A (+1)        | Both need tests, but Model A is functionally more complete for this prompt. |

## Final judgment
Model A is better for this prompt because it handles the zero-memory signal in both `_mi_prim_alloc()` and `_mi_prim_commit()`. This gives a more complete fix path.

Model B still solves the main allocation-time issue, but it leaves one path incomplete and has no tests, so it is weaker right now.
