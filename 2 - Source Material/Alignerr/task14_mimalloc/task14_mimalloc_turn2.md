# Review Update: Model A vs Model B (Prompt 1 + Prompt 2)

Prompt 1:
`Fix the following issue where, many small memory allocations and frees which could be avoided by the allocator happen in a short time and are resulting in performance issues. Optimise the allocator and test the improvements`

Prompt 2:
`There are still some gaps, currently we only improve for the only page in queue condition, add improvements for small-blocks of pages etc.;. Add tests that check explicitly for pass / fail while benchmarks check for numerical improvements`

Scoring scale used in table:
- `+4` = Model A fully wins
- `-4` = Model B fully wins

Backward compatibility rule used in this review:
- I gave credit when compatibility-safe choices were made.
- I did not subtract points only because backward compatibility may change.

## Model A

### Full flow (how the optimization is applied)
There is no new public option flag. Behavior is automatic.

1. Allocation starts in `mi_malloc(size_t size)` in `A/src/alloc.c:214`.
2. `mi_malloc()` calls `mi_heap_malloc()` in `A/src/alloc.c:210`, then small allocation fast-path reaches `_mi_page_malloc_zero(...)` in `A/src/alloc.c:31`.
3. Model A changed `_mi_page_malloc_zero(mi_heap_t* heap, mi_page_t* page, size_t size, bool zero, size_t* usable)`:
- Trigger: `page->free == NULL`.
- Action: it checks `page->local_free`; if not null, it moves `page->local_free` to `page->free` (`A/src/alloc.c:42-47`).
- Impact: next block is popped from `page->free` in the same function call, so allocator avoids fallback to `_mi_malloc_generic(...)` in this churn case.
4. Free starts in `mi_free(void* p)` and goes to `mi_free_block_local(...)` in `A/src/free.c:31`.
5. `mi_free_block_local(...)` pushes block to `page->local_free` (`A/src/free.c:45-46`).
6. When `--page->used == 0`, `mi_free_block_local(...)` calls `_mi_page_retire(page)` (`A/src/free.c:47-48`).
7. Model A changed `_mi_page_retire(mi_page_t* page)` in `A/src/page.c:462`:
- Original retain case is still present: if page is the only page in this size queue.
- Added small-block multi-page case: when `bsize <= MI_SMALL_OBJ_SIZE_MAX`, page can be retained briefly (`A/src/page.c:498-521`).
- It promotes `page->local_free -> page->free` in retire path (`A/src/page.c:506-510`).
- It moves page to queue front with `mi_page_queue_move_to_front(heap, pq, page)` (`A/src/page.c:513`) so `_mi_heap_collect_retired(...)` can manage expiry from queue head.

In simple words:
- In A, churn can be faster in two places: allocation-time reclaim (`_mi_page_malloc_zero`) and retirement-time keep/reuse (`_mi_page_retire`).

### New or modified functions/parameters
Modified functions:
- `_mi_page_malloc_zero(...)` in `A/src/alloc.c:31`
- `_mi_page_retire(mi_page_t* page)` in `A/src/page.c:462`
- CMake test wiring in `A/CMakeLists.txt:742-753`

Added file:
- `A/test/test-perf.c`

Important modified parameters/fields:
- `page->free`: allocation pop-list
- `page->local_free`: newly freed local blocks list
- `page->retire_expire`: retention countdown for retired pages
- `bsize` (from `mi_page_block_size(page)`): used to classify small-block behavior
- `pq->first` / `pq->last`: queue position checks

### Tests added and what each test proves
File: `A/test/test-perf.c` (pass/fail + benchmarks in one executable)

Pass/fail tests (via `CHECK(...)` / `CHECK_BODY(...)` from `A/test/testhelper.h:33-47`):
1. `test_churn_page_reuse()` (`A/test/test-perf.c:68`)
- Proves free-all then alloc-all can reuse same page region.
- Edge case: checks 64KiB page alignment identity (`page_a == page_b`).

2. `test_churn_memory_integrity()` (`A/test/test-perf.c:92`)
- Proves no corruption after churn cycles by writing then verifying byte values.

3. `test_churn_calloc_zero()` (`A/test/test-perf.c:125`)
- Proves `mi_calloc()` still returns zeroed memory after churn and reuse.

4. `test_single_object_churn()` (`A/test/test-perf.c:156`)
- Proves extreme alloc/free one-by-one loop remains correct.

5. `test_multi_size_churn()` (`A/test/test-perf.c:178`)
- Proves correctness across size classes in repeated churn.

6. `test_multi_page_churn()` (`A/test/test-perf.c:200`)
- Proves correctness when small allocations span multiple pages.

7. `test_churn_realloc()` (`A/test/test-perf.c:229`)
- Proves reallocation path preserves old content during churn.

Numerical benchmark section:
- `bench_churn(...)` (`A/test/test-perf.c:268`) and `run_benchmarks()` (`A/test/test-perf.c:309`) print throughput.

CMake integration:
- `mimalloc-test-perf` target + `add_test(NAME test-perf ...)` in `A/CMakeLists.txt:742-753`.

### Pros (with exact functions/parameters)
- In `_mi_page_malloc_zero(...)`, when `page->free` is empty but `page->local_free` has blocks, A moves blocks and allocates immediately in same call (`A/src/alloc.c:39-50`).
  This directly removes one slow fallback call to `_mi_malloc_generic(...)` for common churn patterns.
- In `_mi_page_retire(...)`, A covers not only “only page in queue” but also a small-block multi-page case (`A/src/page.c:498-521`).
  This means churn optimization is not limited to single-page queues.
- A adds pass/fail correctness checks and numeric benchmark output in one file (`A/test/test-perf.c`), so both correctness and speed are tested.

### Cons (with exact functions/parameters)
- `_mi_page_retire(...)` has more branch paths and queue mutation (`mi_page_queue_move_to_front`) (`A/src/page.c:513`), so maintenance and reasoning become harder.
- `test-perf` is registered in `ctest` (`A/CMakeLists.txt:752`) although it also runs large benchmarks. This can make regular CI test duration noisy.
- A has no separate short deterministic retire-only test file; retire correctness is mixed with benchmark workload in `A/test/test-perf.c`.

### PR readiness (Model A)
Status: **Needs one more iteration before merge**.

Reason:
- Runtime optimization is strong and prompt-aligned.
- Test packaging should be split so CI correctness lane is deterministic and fast.

### Concrete next-turn fixes for Model A
1. Split tests into:
- `test-retire.c` for pass/fail deterministic checks.
- `test-perf.c` for numeric performance only.
2. Keep allocation-time reclaim in `_mi_page_malloc_zero(...)`, but add explicit invariant checks around `page->free` and `page->local_free` transitions.
3. Add targeted tests for the small-block multi-page retire branch in `_mi_page_retire(...)`.

---

## Model B

### Full flow (how the optimization is applied)
There is no new public option flag. Behavior is automatic.

1. Free starts in `mi_free()` and reaches `mi_free_block_local(...)` in `B/src/free.c:31`.
2. `mi_free_block_local(...)` pushes freed block to `page->local_free` (`B/src/free.c:45-46`).
3. When page becomes empty (`--page->used == 0`), it calls `_mi_page_retire(page)` (`B/src/free.c:47-48`).
4. Model B changed `_mi_page_retire(mi_page_t* page)` in `B/src/page.c:462` with two conditions:
- `only_page`: this page is both `pq->first` and `pq->last`.
- `small_front`: this is a small-object page (`bsize <= MI_SMALL_OBJ_SIZE_MAX`) and it is queue head (`pq->first == page`) even when queue has other pages.
5. If `only_page || small_front` is true, B retains page for some cycles by setting `page->retire_expire` (`B/src/page.c:489-490`) and promotes `page->local_free -> page->free` (`B/src/page.c:493-497`).
6. Why this matters:
- Next `mi_malloc()` for same size class can hit the same page and pop directly from `page->free`.
- This avoids freeing and re-allocating warm page metadata too often during short bursts.
7. Model B also changed `_mi_heap_collect_retired(...)` in `B/src/page.c:514`:
- While page is still retired and alive, if it has blocks in `page->local_free` but `page->free` is empty, B re-promotes local list to free list (`B/src/page.c:527-533`).
- So repeated retire/collect cycles still keep next allocation on fast list-pop path.
8. Allocation fast path in `B/src/alloc.c:_mi_page_malloc_zero(...)` is unchanged (`B/src/alloc.c:37-40`):
- If `page->free == NULL`, it still directly falls back to `_mi_malloc_generic(...)`.

In simple words:
- B improves retirement behavior and queue-head small-page reuse, but does not add allocation-time local reclaim logic.

### New or modified functions/parameters
Modified functions:
- `_mi_page_retire(mi_page_t* page)` in `B/src/page.c:462`
- `_mi_heap_collect_retired(mi_heap_t* heap, bool force)` in `B/src/page.c:514`
- CMake wiring in `B/CMakeLists.txt:727-743`

Added files:
- `B/test/test-retire.c` (pass/fail retire-focused test suite)
- `B/test/test-perf.c` (benchmark)

Important modified parameters/fields:
- `only_page`: singleton queue condition
- `small_front`: small-block queue-head condition
- `page->retire_expire`: retained-page lifetime
- `page->free` / `page->local_free`: reusable block lists
- `pq->first` / `pq->last`: queue-position checks

### Tests added and what each test proves
1. Pass/fail tests: `B/test/test-retire.c`
- `churn_correctness(...)` (`B/test/test-retire.c:67`) verifies data canary integrity for repeated churn by size.
- `retirement_fast_path(...)` (`B/test/test-retire.c:100`) checks churn path cost relative to steady-state path with ratio limit.
- `multi_page_churn_correctness(...)` (`B/test/test-retire.c:161`) verifies correctness in 2-page and 4-page churn.
- `calloc_zeroinit_after_churn(...)` (`B/test/test-retire.c:190`) verifies zero-init after reuse.
- `single_object_churn(...)` (`B/test/test-retire.c:221`) verifies single-object edge pattern.
- `mixed_size_churn(...)` (`B/test/test-retire.c:240`) verifies interleaved size-class churn correctness.
- `usable_size_after_churn(...)` (`B/test/test-retire.c:272`) verifies `mi_usable_size()` invariant.
- `timing_regression_multi_page(...)` (`B/test/test-retire.c:297`) checks multi-page per-alloc slowdown bound.

CMake integration:
- Adds `retire` into standard `ctest` loop (`B/CMakeLists.txt:727-740`).

2. Numerical benchmark: `B/test/test-perf.c`
- `bench_churn(...)` (`B/test/test-perf.c:45`) + config matrix (`B/test/test-perf.c:96-103`) output throughput numbers.
- Kept manual by design (not in `ctest`) (`B/CMakeLists.txt:742-752`).

### Pros (with exact functions/parameters)
- Prompt-2 small-block gap is concretely addressed in `_mi_page_retire(...)` with `small_front` (`B/src/page.c:484`).
  Explanation: when a just-emptied small page is at queue head, B retains it (`page->retire_expire`) and keeps reusable blocks in `page->free`, so next same-size allocation can reuse hot page data instead of cold page reacquire.
- `_mi_heap_collect_retired(...)` re-promotes `local_free -> free` (`B/src/page.c:527-533`).
  Explanation: even after retire-cycle housekeeping, page stays ready for immediate list-pop allocation path.
- B separates correctness tests (`test-retire.c`) and benchmarks (`test-perf.c`), which is cleaner for development flow and CI behavior.
- B keeps benchmark manual in CMake, so regular `ctest` remains focused on deterministic pass/fail checks.

### Cons (with exact functions/parameters)
- `_mi_page_malloc_zero(...)` in `B/src/alloc.c:37-40` still does not reclaim `page->local_free` inline.
  Effect: if page has reusable blocks only in `local_free`, this function still jumps to `_mi_malloc_generic(...)` instead of using same-call local reclaim.
- `small_front` path is narrower than a general multi-page optimization because it needs `pq->first == page` (`B/src/page.c:484`).
  Effect: pages not at head do not get this retire retention behavior.
- Header comment in `B/test/test-retire.c:17` mentions double-free/used-count invariants, but explicit double-free test case is not present.
- Some pass/fail decisions are timing-ratio based (`retirement_fast_path`, `timing_regression_multi_page`), which can vary by machine/load.

### PR readiness (Model B)
Status: **Needs one more iteration before merge, but test structure is cleaner**.

Reason:
- Runtime behavior is improved in retire path and small-front case.
- Test organization is strong.
- Missing alloc-time reclaim path and minor test-claim alignment.

### Concrete next-turn fixes for Model B
1. Add local reclaim to allocation fast path in `_mi_page_malloc_zero(...)`:
- when `page->free == NULL` and `page->local_free != NULL`, move list and allocate in same call.
2. Validate/extend `small_front` coverage:
- either widen condition or provide data that queue-head-only policy is enough.
3. Add explicit structural tests (not only timing thresholds) for retire-list behavior.
4. Align `test-retire.c` intro claims with actually implemented checks.

---

## Comparison Table

| Question of which is / has | Answer Given | Justification Why? |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution | Model A (`+1`) | A optimizes two runtime moments: allocation-time reclaim in `_mi_page_malloc_zero(...)` and retirement-time retention in `_mi_page_retire(...)`. B optimizes mostly retirement-time behavior. |
| Better logic and correctness | Model A (`+1`) | In A, if `page->free` is empty but `page->local_free` has blocks, same function call can still allocate (`A/src/alloc.c:39-50`) without generic fallback. That is direct churn-path correctness/perf logic. |
| Better Naming and Clarity | Model B (`-1`) | B uses explicit names `only_page` and `small_front` in `B/src/page.c`, making trigger conditions easier for junior readers to map to queue state. |
| Better Organization and Clarity | Model B (`-2`) | B separates deterministic pass/fail checks (`test-retire.c`) from numeric benchmark (`test-perf.c`), so readers and CI can reason about correctness and performance independently. |
| Better Interface Design | Model B (`-1`) | Neither changes public API, but B’s CMake integration is cleaner: `test-retire` in `ctest`, `test-perf` manual. That gives clearer user/developer workflow. |
| Better error handling and robustness | Model B (`-1`) | B’s default test lane avoids running heavy benchmark loops as regular test cases, reducing flaky CI behavior while still keeping correctness checks active. |
| Better comments and documentation | Model B (`-1`) | B’s `_mi_page_retire(...)` comments explain “why this branch exists” and “memory hold-time tradeoff” directly near logic, which helps when reading without prior context. |
| Ready for review / merge | Model B (`-1`) | B is closer: cleaner test packaging and focused retire-path changes. A still needs test split for stable PR checks. |

## Final judgment
Model A is slightly stronger on raw optimization coverage for short-term small alloc/free churn.
Model B is stronger on test structure and readability for maintainers.

If choosing one base branch:
- Choose **Model A** if immediate runtime optimization breadth is main goal.
- Choose **Model B** if maintainability and cleaner CI test strategy are main goal.

Best combined direction:
- Keep A’s allocation-time local reclaim in `_mi_page_malloc_zero(...)`.
- Keep B’s split test strategy (`test-retire.c` for pass/fail, `test-perf.c` for numbers).
