# Review: Model A vs Model B (Prompt 1)

Prompt reviewed:
> Fix the issue, when working with mimalloc, apps do extra zeroing work and lose performance in allocation-heavy scenarios

## Model A Review

### Changed files
- `A/src/alloc.c`

### Full flow (how the zero option is applied)
- Entry path for zero-realloc is:
  `mi_recalloc()` -> `mi_heap_recalloc()` -> `mi_heap_rezalloc()` -> `_mi_heap_realloc_zero(..., zero=true)`
  at `A/src/alloc.c:369`, `A/src/alloc.c:341`, `A/src/alloc.c:337`, `A/src/alloc.c:280`.
- In `_mi_heap_realloc_zero(...)`, Model A changed allocation from `mi_heap_umalloc(...)` to `_mi_heap_malloc_zero_ex(..., zero, 0, usable_post)` at `A/src/alloc.c:304`.
- This means the `zero` parameter now goes directly into allocator fast/slow paths:
  `_mi_heap_malloc_zero_ex(...)` -> `mi_heap_malloc_small_zero(...)` (small alloc) or `_mi_malloc_generic(...)` (generic alloc)
  at `A/src/alloc.c:168`, `A/src/alloc.c:172`, `A/src/alloc.c:183`, `A/src/page.c:988`.
- Final zero behavior happens inside `_mi_page_malloc_zero(..., zero, ...)` at `A/src/alloc.c:31` / `A/src/page.c:1034`.
- In `_mi_page_malloc_zero(...)`, when `zero=true`:
  - If `page->free_is_zero` is true, it avoids full memzero and does minimal work (`block->next = 0`) at `A/src/alloc.c:66`-`A/src/alloc.c:69`.
  - Else it zeroes block once with `_mi_memzero_aligned(...)` at `A/src/alloc.c:71`.
- Model A also removed the manual second-stage `_mi_memzero(...)` branch from realloc path, so extra zeroing work in `_mi_heap_realloc_zero(...)` is removed.

### New or modified functions/parameters (2 lines each)
- `_mi_heap_realloc_zero(mi_heap_t* heap, void* p, size_t newsize, bool zero, size_t* usable_pre, size_t* usable_post)` in `A/src/alloc.c:280`
  Model A changed one call site: now uses `_mi_heap_malloc_zero_ex(..., zero, ...)` at `A/src/alloc.c:304`.
  This forwards the existing `zero` parameter to allocator internals instead of manual memset in realloc.
- Parameter usage: `zero`
  Same parameter name, but behavior is improved because it controls allocation-time zeroing path directly.
  This is important for performance because allocator can skip unnecessary clearing when pages are already zero (`page->free_is_zero`).

### Tests added and what they prove
- No new tests were added in Model A (`git -C A diff --name-only` shows only `src/alloc.c`).
- Because of this, there is no new direct proof for these edge cases:
  - `mi_rezalloc()` grow path zero guarantees,
  - `p == NULL` path in `_mi_heap_realloc_zero(...)`,
  - `newsize == 0` behavior (`A/src/alloc.c:306`),
  - “reuse same pointer” path when `newsize <= size && newsize >= size/2`.

### Pros (with exact code references)
- Better fix for the prompt issue: zero behavior now uses allocator-native zero path via `_mi_heap_malloc_zero_ex(..., zero, ...)` at `A/src/alloc.c:304`.
- Removes duplicate/manual zero logic from realloc path, reducing extra zeroing work in allocation-heavy growth scenarios.
- Keeps API surface unchanged (no unnecessary external API expansion for this task).

### Cons (with exact code references)
- No regression/perf test added to prove improvement and to lock behavior.
- Removed explicit padding-zero branch from realloc path; behavior now depends fully on allocator path correctness (`_mi_page_malloc_zero(...)` at `A/src/alloc.c:60`-`A/src/alloc.c:72`). This is likely correct, but still should be tested.

### PR readiness (Model A)
- Status: **Mostly ready, but not merge-ready without tests**.
- Concrete next-turn fixes:
  1. Add `test_rezalloc_grow_zero_tail()` in `A/test/test-api-fill.c` to verify newly grown bytes are zero for `mi_rezalloc`/`mi_recalloc`.
  2. Add edge tests for `p==NULL`, `newsize==0`, and in-place reuse branch in `_mi_heap_realloc_zero(...)`.
  3. Add a small micro-benchmark for realloc-grow loops to show reduced zeroing cost.

---

## Model B Review

### Changed files
- `B/src/alloc.c`
- `B/include/mimalloc.h`

### Full flow (how the zero option is applied)
- Zero-realloc entry path is still:
  `mi_recalloc()` -> `mi_heap_recalloc()` -> `mi_heap_rezalloc()` -> `_mi_heap_realloc_zero(..., zero=true)`
  at `B/src/alloc.c:378`, `B/src/alloc.c:350`, `B/src/alloc.c:346`, `B/src/alloc.c:284`.
- Inside `_mi_heap_realloc_zero(...)`, allocation is still `mi_heap_umalloc(...)` at `B/src/alloc.c:308`.
- `mi_heap_umalloc(...)` always calls `_mi_heap_malloc_zero_ex(..., false, ...)` at `B/src/alloc.c:240`-`B/src/alloc.c:241`, so allocator does **not** get `zero=true` in this path.
- Then manual clearing still happens in realloc with:
  `if (zero && newsize > size) { _mi_memzero(...) }` at `B/src/alloc.c:310`-`B/src/alloc.c:314`.
- So the extra zeroing pattern in the prompt is not removed in the core realloc path.

### New or modified functions/parameters (2 lines each)
- `mi_uzalloc_small(size_t size, size_t* usable)` added in `B/src/alloc.c:236`
  Calls `mi_heap_malloc_small_zero(..., true, usable)` for small zeroed allocation with returned usable size.
  This is a useful helper/API completeness improvement, but unrelated to realloc extra-zeroing problem.
- `mi_uzalloc(size_t size, size_t* block_size)` declaration added in `B/include/mimalloc.h:193`
  Exposes already-existing function as public API declaration.
  Good for header consistency, but does not affect `_mi_heap_realloc_zero(...)` flow.

### Tests added and what they prove
- No new tests were added in Model B (`git -C B diff --name-only` only shows `src/alloc.c` and `include/mimalloc.h`).
- So there is no new direct proof for:
  - realloc grow zero behavior,
  - allocation-heavy performance improvement,
  - edge cases (`p==NULL`, `newsize==0`, reuse same pointer).

### Pros (with exact code references)
- Improves API completeness: `mi_uzalloc` declaration in `B/include/mimalloc.h:193`.
- Adds missing convenience/helper function `mi_uzalloc_small(...)` in `B/src/alloc.c:236`.

### Cons (with exact code references)
- Main issue is not fixed: `_mi_heap_realloc_zero(...)` still allocates via `mi_heap_umalloc(...)` at `B/src/alloc.c:308` and does manual `_mi_memzero(...)` at `B/src/alloc.c:310`-`B/src/alloc.c:314`.
- Prompt asks to reduce extra zeroing work in allocation-heavy paths; this core path remains unchanged.

### PR readiness (Model B)
- Status: **Not ready for this prompt** (changes are mostly API additions, not core fix).
- Concrete next-turn fixes:
  1. In `_mi_heap_realloc_zero(...)`, replace `mi_heap_umalloc(...)` with `_mi_heap_malloc_zero_ex(heap, newsize, zero, 0, usable_post)`.
  2. Remove manual `_mi_memzero(...)` branch in realloc-grow path.
  3. Add tests/benchmark for realloc-grow and recalloc performance + correctness.

---

## Comparison Table (Scale: +4 = Model A fully wins, -4 = Model B fully wins)

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Better logic and correctness         | Model A (+4)        | `A/src/alloc.c:304` forwards `zero` into `_mi_heap_malloc_zero_ex(...)`, so allocator handles zeroing directly. `B/src/alloc.c:308` still uses `mi_heap_umalloc(...)` + manual zeroing. |
| Better Naming and Clarity            | Model A (+1)        | A change is very focused in one critical function. B adds clear names (`mi_uzalloc`, `mi_uzalloc_small`) but they do not explain/fix the prompt’s main path. |
| Better Organization and Clarity      | Model A (+2)        | A removes custom zero branch in realloc and reuses central allocator zero logic. This reduces duplicate logic in `_mi_heap_realloc_zero(...)`. |
| Better Interface Design              | Model B (-1)        | B improves public API surface by declaring `mi_uzalloc` in header and adding `mi_uzalloc_small`. This is useful, but not the requested main fix. |
| Better error handling and robustness | Model A (+1)        | A reuses established allocation path (`_mi_heap_malloc_zero_ex`) which already handles small/generic flows consistently. Neither model adds new error-path tests. |
| Better comments and documentation    | Tie (0)             | No meaningful comment/doc improvements in either model for this issue. |
| Ready for review / merge             | Model A (+3)        | A is close but needs tests. B does not address the core prompt issue, so it is not ready for this specific PR goal. |
| Overall Better Solution              | Model A (+4)        | A directly changes the hot realloc zero path; B mainly adds API pieces and leaves key performance issue path unchanged. |

## Final judgment
Model A is better for this prompt because it modifies the real hot path (`_mi_heap_realloc_zero(...)`) and sends the `zero` option to allocator internals, which is exactly where extra zeroing can be reduced. Model B has useful API cleanup, but it does not fix the main performance issue asked in the prompt.
