# Model Review: A vs B for mimalloc zeroing performance

Prompt 1:
> Fix the issue, when working with mimalloc, apps do extra zeroing work and lose performance in allocation-heavy scenarios

Prompt 2:
> Still has some gaps... add tests for `mi_rezalloc()` / `mi_recalloc(..)`, add proof, add benchmark, modularize zeroing/reallocation logic with helpers

## What changed in each model (current repo state)
- Model A changed: `src/alloc.c`, `src/alloc-aligned.c`, `test/test-api.c`, `test/test-api-fill.c`, and added benchmark file `test/test-bench-rezalloc.c` (currently untracked).
- Model B changed: `src/alloc.c`, `test/test-api.c`, `test/test-api-fill.c`, `CMakeLists.txt`, and added benchmark file `test/test-realloc-bench.c` (currently untracked).

---

## Model A

### Full flow of how zero option is applied
1. User-facing entry for zero-realloc is `mi_rezalloc(..)` and `mi_recalloc(..)`.
2. Call chain is: `mi_recalloc(..)` -> `mi_heap_recalloc(..)` -> `mi_heap_rezalloc(..)` -> `_mi_heap_realloc_zero(heap, p, newsize, zero=true, usable_pre, usable_post)`.
3. In `_mi_heap_realloc_zero(..)`, it first computes old usable size from `_mi_usable_size(..)` and handles in-place shrink reuse (`newsize <= size && newsize >= size/2`).
4. For non in-place cases, it computes `needs_ext_zero = (p != NULL && zero && newsize > size)`.
5. Allocation call is `_mi_heap_malloc_zero_ex(heap, newsize, (needs_ext_zero ? false : zero), 0, usable_post)`.
6. Behavior split:
- If `p == NULL` and `zero=true`, allocator gets `zero=true` and can optimize via `page->free_is_zero` in `_mi_page_malloc_zero(..)`.
- If grow path (`needs_ext_zero=true`), allocator gets `zero=false`, then only extension is manually zeroed by `_mi_memzero(..)` from `start = size - sizeof(intptr_t)` to `newsize`.
7. Copy+free is moved to helper `mi_realloc_copy_and_free(newp, p, newsize, oldsize)`.
8. Aligned path `mi_heap_realloc_zero_aligned_at(..)` uses same strategy with `needs_ext_zero` and `mi_heap_malloc_zero_aligned_at(..)`.

### New/modified functions or parameters (2-line explanation each)
- `mi_realloc_copy_and_free(newp, p, newsize, oldsize)`
  New helper in `alloc.c` that centralizes copy (`_mi_memcpy`) and old-pointer free (`mi_free`) after successful reallocation.
  This modularizes common work and keeps `_mi_heap_realloc_zero(..)` focused on zeroing strategy.

- `_mi_heap_realloc_zero(heap, p, newsize, zero, usable_pre, usable_post)`
  Modified to use `needs_ext_zero` and selective allocator zero flag passing.
  It now combines allocator-side zeroing for fresh allocations and extension-only zeroing for grow realloc.

- `needs_ext_zero` (local bool in `_mi_heap_realloc_zero(..)`)
  New control parameter for strategy branching: true only for grow with `zero=true` and non-NULL old pointer.
  It prevents full-block zeroing when only extension bytes must be zero.

- `mi_heap_realloc_zero_aligned_at(heap, p, newsize, alignment, offset, zero)`
  Modified in `alloc-aligned.c` to mirror non-aligned `needs_ext_zero` behavior.
  This makes aligned and non-aligned zero-realloc logic consistent.

### Tests added and what each test proves
`test/test-api.c` (new checks):
- `rezalloc-null`, `recalloc-null`: `mi_rezalloc(NULL, n)` / `mi_recalloc(NULL, c, s)` produce zero-initialized memory.
- `rezalloc-null-sizezero`, `rezalloc-sizezero`, `recalloc-null-sizezero`: zero-size realloc contract returns non-NULL.
- `rezalloc-grow`, `rezalloc-large-grow`, `recalloc-grow`: old region keeps data; new extension region is zero.
- `rezalloc-shrink`, `recalloc-shrink`: shrink keeps prefix bytes intact.
- `recalloc-overflow`: overflow in count*size returns NULL.
- `heap-rezalloc-grow`, `heap-recalloc-grow`, `heap-rezalloc-null`, `heap-recalloc-null`: same guarantees for heap-local APIs.

`test/test-api-fill.c` (new checks):
- `zeroinit-rezalloc-from-null-*`, `zeroinit-recalloc-from-null-*`: explicit small/large NULL-source zero init.
- `zeroinit-rezalloc-extension-only-*`, `zeroinit-recalloc-extension-only-*`: extension-only zero with preserved old non-zero pattern.
- `zeroinit-heap-rezalloc-extension-only-*`, `zeroinit-heap-recalloc-extension-only-*`: same extension behavior for heap APIs.

Benchmark:
- `test/test-bench-rezalloc.c` exists and includes correctness + timing scenarios.
- But it is currently untracked and not wired into `CMakeLists.txt`, so it is not part of normal test/CI flow yet.

### Pros (exact functions/parameters)
- `_mi_heap_realloc_zero(..)` uses `needs_ext_zero` to avoid full-block zeroing on grow cases.
- `_mi_heap_malloc_zero_ex(..)` still receives `zero=true` for `p==NULL` paths, so allocator can use `page->free_is_zero` optimization.
- `mi_realloc_copy_and_free(..)` improves modularization and reduces repeated copy/free code.
- `mi_heap_realloc_zero_aligned_at(..)` mirrors the same strategy, so aligned APIs are consistent with non-aligned behavior.
- Test depth is strong for `mi_rezalloc(..)` / `mi_recalloc(..)` and heap variants, including NULL, shrink, grow, overflow, and large-size cases.

### Cons (exact functions/parameters)
- `test/test-bench-rezalloc.c` is untracked and not in `CMakeLists.txt`, so performance proof is not CI-enforced.
- `_mi_heap_realloc_zero(..)` and `mi_heap_realloc_zero_aligned_at(..)` still duplicate extension-zero code (`start` + `_mi_memzero`); could be one shared helper.
- Some new tests access `q[i]` before checking `q != NULL`; low risk in normal runs, but robustness could be better.

### PR readiness (Model A)
- Status: **Almost ready, but not merge-ready yet**.
- Why: core logic and tests are good, but benchmark integration is incomplete.

Concrete next-turn fixes:
1. Add benchmark target in `CMakeLists.txt` for `test/test-bench-rezalloc.c`.
2. Track/commit benchmark file and make it runnable in CI.
3. Extract shared extension-zero helper used by `_mi_heap_realloc_zero(..)` and `mi_heap_realloc_zero_aligned_at(..)`.
4. Harden tests by checking allocation result before reading `q[i]`.

---

## Model B

### Full flow of how zero option is applied
1. User-facing zero-realloc path is still `mi_rezalloc(..)` / `mi_recalloc(..)`.
2. Call chain: `mi_recalloc(..)` -> `mi_heap_recalloc(..)` -> `mi_heap_rezalloc(..)` -> `_mi_heap_realloc_zero(heap, p, newsize, zero=true, ...)`.
3. `_mi_heap_realloc_zero(..)` now delegates in-place check to helper `mi_realloc_is_inplace(p, newsize, oldsize)`.
4. For non in-place case, `_mi_heap_realloc_zero(..)` calls helper `mi_heap_realloc_alloc_copy(heap, p, newsize, oldsize, zero, usable_post)`.
5. `mi_heap_realloc_alloc_copy(..)` allocates with `_mi_heap_malloc_zero_ex(heap, newsize, zero, 0, usable_post)`.
6. So for grow + `zero=true`, allocator zeroes full new block when needed; old block is copied and freed in helper.
7. Important difference: aligned path `mi_heap_realloc_zero_aligned_at(..)` was not updated and still uses manual extension zero after `mi_heap_malloc_aligned_at(..)`.

### New/modified functions or parameters (2-line explanation each)
- `mi_realloc_is_inplace(p, newsize, oldsize)`
  New helper in `alloc.c` for in-place shrink decision logic.
  This improves readability by moving condition details out of `_mi_heap_realloc_zero(..)`.

- `mi_heap_realloc_alloc_copy(heap, p, newsize, oldsize, zero, usable_post)`
  New helper that allocates with `_mi_heap_malloc_zero_ex(.., zero, ..)`, then copies old bytes and frees old pointer.
  This modularizes allocation+copy+free sequence and keeps `_mi_heap_realloc_zero(..)` shorter.

- `_mi_heap_realloc_zero(heap, p, newsize, zero, usable_pre, usable_post)`
  Refactored to call both helpers above and remove inline copy/free code.
  Logic becomes simpler, but grow zeroing is now full allocator zeroing instead of extension-targeted zero.

- `CMakeLists.txt` test loop (`foreach(TEST_NAME ...)`)
  Modified to include `realloc-bench` test target.
  This is good integration intent for performance testing.

### Tests added and what each test proves
`test/test-api.c` (new checks):
- `rezalloc-null`, `recalloc-null`: NULL-source acts like zero allocator.
- `rezalloc-grow`, `recalloc-grow`: old data preserved and extension zero.
- `rezalloc-shrink`, `recalloc-shrink`: shrink keeps prefix bytes.
- `rezalloc-sizezero`: zero-size rezalloc returns non-NULL.
- `heap-rezalloc`, `heap-recalloc`: same grow+zero behavior on heap APIs.

`test/test-api-fill.c` (new checks):
- `zeroinit-rezalloc-null`, `zeroinit-recalloc-null`: NULL-source zero init.
- `zeroinit-rezalloc-datapreserve-*`, `zeroinit-recalloc-datapreserve`: preserve old data + zero extension.
- `zeroinit-rezalloc-multi`, `zeroinit-recalloc-multi`: repeated growth keeps extension zero each step.
- `zeroinit-rezalloc-sizezero`, `zeroinit-recalloc-sizezero`: zero-size behavior.

Benchmark:
- `test/test-realloc-bench.c` exists and `CMakeLists.txt` includes `realloc-bench` target.
- But benchmark file is currently untracked; in this repo state, patch is incomplete for a clean commit.

### Pros (exact functions/parameters)
- `mi_realloc_is_inplace(..)` and `mi_heap_realloc_alloc_copy(..)` improve modularization of `_mi_heap_realloc_zero(..)`.
- `_mi_heap_realloc_zero(..)` now passes `zero` into `_mi_heap_malloc_zero_ex(..)` directly (clear call chain).
- Added many tests for `mi_rezalloc(..)` / `mi_recalloc(..)` correctness in both API and fill suites.
- `CMakeLists.txt` updates benchmark target, showing intent to integrate performance checks into build.

### Cons (exact functions/parameters)
- In `_mi_heap_realloc_zero(..)`, grow path with `zero=true` uses `_mi_heap_malloc_zero_ex(.., zero=true, ..)` and may zero full block, not only extension; this can be extra work in allocation-heavy grow loops.
- `mi_heap_realloc_zero_aligned_at(..)` remains on old manual path, so aligned vs non-aligned logic is inconsistent.
- `test/test-realloc-bench.c` is untracked while `CMakeLists.txt` expects it; this can break PR completeness.
- Benchmark baseline in `bench_manual_rezalloc()` is not a strict old-path equivalence (it zeroes full block), so performance conclusions can be skewed.

### PR readiness (Model B)
- Status: **Needs more work before merge**.
- Why: refactor and tests are useful, but core performance optimization is weaker and patch completeness has integration risk.

Concrete next-turn fixes:
1. Add grow-specific strategy in `_mi_heap_realloc_zero(..)` (`needs_ext_zero`) so only extension bytes are zeroed when safe.
2. Mirror same strategy in `mi_heap_realloc_zero_aligned_at(..)` for consistency.
3. Track/commit `test/test-realloc-bench.c` and validate benchmark methodology against old behavior.
4. Add one benchmark that isolates extension-only zero cost vs full-block zero cost.

---

## Comparison (rating scale: +4 Model A fully wins, -4 Model B fully wins)

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Better logic and correctness         | Model A (+3)        | `Model A` uses `needs_ext_zero` in `_mi_heap_realloc_zero(..)` to balance correctness and reduced zero work. `Model B` is correct for zero semantics, but can do more full-block zeroing in grow path. |
| Better Naming and Clarity            | Model B (-1)        | `Model B` helper names (`mi_realloc_is_inplace`, `mi_heap_realloc_alloc_copy`) are very straightforward. `Model A` is clear too, but its strategy is more complex. |
| Better Organization and Clarity      | Model A (+1)        | `Model A` updates both `_mi_heap_realloc_zero(..)` and `mi_heap_realloc_zero_aligned_at(..)` consistently. `Model B` only refactors non-aligned path. |
| Better Interface Design              | Tie (0)             | Neither model changes public API surface for this iteration in a meaningful way for this prompt. |
| Better error handling and robustness | Model A (+1)        | `Model A` adds broader edge-case coverage (`recalloc-overflow`, NULL and size-zero variants). Both still need minor NULL-check hardening inside some tests. |
| Better comments and documentation    | Model A (+2)        | `Model A` adds detailed strategy comments directly in `_mi_heap_realloc_zero(..)` and aligned counterpart, which helps junior readers understand why the branch exists. |
| Ready for review / merge             | Model A (+1)        | `Model A` core logic is closer to prompt goal, but benchmark integration is incomplete. `Model B` also has untracked benchmark file plus logic/perf gap. |
| Overall Better Solution              | Model A (+3)        | `Model A` better addresses both prompt 1 (extra zeroing work) and prompt 2 (tests + modularization), especially with extension-only zero optimization and aligned-path consistency. |

## Final judgment
`Model A` is better overall.

Reason:
- It directly targets extra zeroing cost in `_mi_heap_realloc_zero(..)` with `needs_ext_zero` and applies the same logic in `mi_heap_realloc_zero_aligned_at(..)`.
- It adds deeper tests for `mi_rezalloc(..)` / `mi_recalloc(..)` including important edge cases.
- `Model B` improves modularization and test quantity, but its grow path can still do extra full-block zeroing and it leaves aligned path inconsistent.

