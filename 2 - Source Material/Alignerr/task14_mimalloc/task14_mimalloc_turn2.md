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
- If `page->free == NULL`, it now checks `page->local_free` first.
- If `page->local_free != NULL`, it moves `local_free -> free` and allocates directly (`A/src/alloc.c:42-47`).
- Only if still empty, it calls `_mi_malloc_generic(...)` (`A/src/alloc.c:49`).
4. Free starts in `mi_free(void* p)` and goes to `mi_free_block_local(...)` in `A/src/free.c:31`.
5. `mi_free_block_local(...)` pushes block to `page->local_free` (`A/src/free.c:45-46`).
6. When `--page->used == 0`, `mi_free_block_local(...)` calls `_mi_page_retire(page)` (`A/src/free.c:47-48`).
7. Model A changed `_mi_page_retire(mi_page_t* page)` in `A/src/page.c:462`:
- Original condition kept: only page in queue.
- Added small-block multi-page path: if `bsize <= MI_SMALL_OBJ_SIZE_MAX`, it may retain page briefly (`A/src/page.c:498-521`).
- It promotes `local_free -> free` when retiring (`A/src/page.c:506-510`).
- It moves page to queue front with `mi_page_queue_move_to_front(heap, pq, page)` (`A/src/page.c:513`) so `_mi_heap_collect_retired(...)` can expire it later.

### New or modified functions/parameters
Modified functions:
- `_mi_page_malloc_zero(...)` in `A/src/alloc.c:31`
- `_mi_page_retire(mi_page_t* page)` in `A/src/page.c:462`
- CMake test wiring in `A/CMakeLists.txt:742-753`

Added file:
- `A/test/test-perf.c`

Important modified parameters/fields:
- `page->free`
- `page->local_free`
- `page->retire_expire`
- `bsize` (from `mi_page_block_size(page)`)
- `pq->first` / `pq->last`

### Tests added and what each test proves
File: `A/test/test-perf.c` (pass/fail + benchmarks in one executable)

Pass/fail tests (via `CHECK(...)` / `CHECK_BODY(...)` from `A/test/testhelper.h:33-47`):
1. `test_churn_page_reuse()` (`A/test/test-perf.c:68`)
- Proves churn can reuse the same page after free-all.
- Edge case: checks same 64KiB page alignment region.

2. `test_churn_memory_integrity()` (`A/test/test-perf.c:92`)
- Proves data is writable/readable after churn.
- Edge case: verify byte-level content across rounds.

3. `test_churn_calloc_zero()` (`A/test/test-perf.c:125`)
- Proves `mi_calloc()` remains zeroed after churn.

4. `test_single_object_churn()` (`A/test/test-perf.c:156`)
- Proves extreme 1-alloc/1-free churn does not break correctness.

5. `test_multi_size_churn()` (`A/test/test-perf.c:178`)
- Proves mixed size-class churn still behaves correctly.

6. `test_multi_page_churn()` (`A/test/test-perf.c:200`)
- Proves multi-page small-block churn correctness.

7. `test_churn_realloc()` (`A/test/test-perf.c:229`)
- Proves `mi_realloc()` content preservation in churn path.

Numerical benchmark section:
- `bench_churn(...)` (`A/test/test-perf.c:268`) and `run_benchmarks()` (`A/test/test-perf.c:309`) print throughput.

CMake integration:
- `mimalloc-test-perf` target + `add_test(NAME test-perf ...)` in `A/CMakeLists.txt:742-753`.

### Pros (with exact functions/parameters)
- Directly removes one slow-path call for churn in `_mi_page_malloc_zero(...)` when `page->local_free` is available (`A/src/alloc.c:39-50`).
- Improves beyond only-page condition in `_mi_page_retire(...)` by handling small-block multi-page churn (`A/src/page.c:498-521`).
- Adds both pass/fail correctness and numeric benchmark coverage in one place (`A/test/test-perf.c`).

### Cons (with exact functions/parameters)
- `_mi_page_retire(...)` adds more branch complexity (`A/src/page.c:498-521`) and `mi_page_queue_move_to_front(...)`, so behavior is harder to reason than baseline.
- `test-perf` is also a `ctest` test (`A/CMakeLists.txt:752`) while it includes heavy benchmarks, which can increase CI runtime variance.
- There is no dedicated isolated test file just for retirement logic (all bundled into `test-perf.c`).

### PR readiness (Model A)
Status: **Needs one more iteration before merge**.

Reason:
- Core optimization is useful and prompt-aligned.
- But production test lane should avoid running long benchmark loops as normal `ctest` pass/fail.

### Concrete next-turn fixes for Model A
1. Split `A/test/test-perf.c` into:
- `test-retire.c` (pass/fail only, fast)
- `test-perf.c` (numeric benchmark only, manual or separate perf job).
2. Keep `_mi_page_malloc_zero(...)` local reclaim path, but add targeted assertions/tests around list state transitions (`page->free`, `page->local_free`).
3. Add a deterministic assertion-oriented test specifically for the new small-block multi-page retirement branch in `_mi_page_retire(...)`.

---

## Model B

### Full flow (how the optimization is applied)
There is no new public option flag. Behavior is automatic.

1. Free starts in `mi_free()` and reaches `mi_free_block_local(...)` in `B/src/free.c:31`.
2. `mi_free_block_local(...)` pushes to `page->local_free` (`B/src/free.c:45-46`), and when empty (`--page->used == 0`) calls `_mi_page_retire(page)` (`B/src/free.c:47-48`).
3. Model B changed `_mi_page_retire(mi_page_t* page)` in `B/src/page.c:462`:
- `only_page`: old condition (`pq->first==page && pq->last==page`).
- `small_front`: new condition for small-block queue head (`bsize <= MI_SMALL_OBJ_SIZE_MAX && pq->first == page`) (`B/src/page.c:483-485`).
- If either condition true, retain page with `page->retire_expire` and promote `local_free -> free` (`B/src/page.c:489-497`).
4. Model B also changed `_mi_heap_collect_retired(...)` in `B/src/page.c:514`:
- While page stays retired, if `page->free == NULL && page->local_free != NULL`, it re-promotes `local_free -> free` (`B/src/page.c:527-533`) to keep next allocation on fast path.
5. Allocation fast path in `B/src/alloc.c:_mi_page_malloc_zero(...)` is unchanged (`B/src/alloc.c:37-40`): when `page->free == NULL`, fallback directly to `_mi_malloc_generic(...)`.

### New or modified functions/parameters
Modified functions:
- `_mi_page_retire(mi_page_t* page)` in `B/src/page.c:462`
- `_mi_heap_collect_retired(mi_heap_t* heap, bool force)` in `B/src/page.c:514`
- CMake wiring in `B/CMakeLists.txt:727-743`

Added files:
- `B/test/test-retire.c` (pass/fail retire-focused test suite)
- `B/test/test-perf.c` (benchmark)

Important modified parameters/fields:
- `only_page`, `small_front`
- `page->retire_expire`
- `page->free`
- `page->local_free`
- `pq->first` / `pq->last`

### Tests added and what each test proves
1. Pass/fail tests: `B/test/test-retire.c`
- `churn_correctness(...)` (`B/test/test-retire.c:67`) checks canary data integrity across sizes.
- `retirement_fast_path(...)` (`B/test/test-retire.c:100`) checks relative timing ratio for churn vs steady-state.
- `multi_page_churn_correctness(...)` (`B/test/test-retire.c:161`) checks 2–4 page churn correctness.
- `calloc_zeroinit_after_churn(...)` (`B/test/test-retire.c:190`) verifies zero-init after churn reuse.
- `single_object_churn(...)` (`B/test/test-retire.c:221`) covers one-object edge case.
- `mixed_size_churn(...)` (`B/test/test-retire.c:240`) checks mixed small sizes.
- `usable_size_after_churn(...)` (`B/test/test-retire.c:272`) checks `mi_usable_size >= requested`.
- `timing_regression_multi_page(...)` (`B/test/test-retire.c:297`) checks per-allocation multi-page timing bound.

CMake integration:
- Added `retire` into normal test loop (`B/CMakeLists.txt:727`), so it runs in `ctest`.

2. Numerical benchmark: `B/test/test-perf.c`
- `bench_churn(...)` (`B/test/test-perf.c:45`) and multiple configs (`B/test/test-perf.c:96-103`) print throughput.
- It is intentionally manual (not `ctest`) in `B/CMakeLists.txt:742-752`.

### Pros (with exact functions/parameters)
- Small-block gap from prompt 2 is addressed explicitly with `small_front` in `_mi_page_retire(...)` (`B/src/page.c:484`).
- Adds dedicated pass/fail test file (`B/test/test-retire.c`) and keeps benchmarks separate in `B/test/test-perf.c`.
- Keeps benchmark out of `ctest` (`B/CMakeLists.txt:742` comment), which is cleaner for CI stability.
- Adds extra retired-page free-list promotion in `_mi_heap_collect_retired(...)` (`B/src/page.c:527-533`).

### Cons (with exact functions/parameters)
- `_mi_page_malloc_zero(...)` fast path in `B/src/alloc.c:37-40` was not improved, so one direct churn optimization path is missing versus Model A.
- `small_front` optimization is narrower than A’s multi-page branch because it requires `pq->first == page` (`B/src/page.c:484`).
- `B/test/test-retire.c` claims double-free/used-count invariants in header comments (`B/test/test-retire.c:17`) but no explicit double-free test is implemented.
- Several pass/fail checks depend on timing ratios (`ratio_limit`, timing regression), which can still be environment-sensitive.

### PR readiness (Model B)
Status: **Also needs one more iteration before merge, but test structure is cleaner**.

Reason:
- Good separation between deterministic tests and benchmark target.
- Still missing alloc fast-path improvement and some comment-to-test alignment.

### Concrete next-turn fixes for Model B
1. Add local reclaim in allocation fast path (`_mi_page_malloc_zero(...)`) similar to A, guarded by clear invariants.
2. Either widen small-block retire condition (beyond `pq->first == page`) or add data proving current `small_front` is enough.
3. Replace timing-only pass/fail checks with more structural assertions where possible.
4. Align `test-retire.c` header claims with actual implemented checks (or add missing invariant tests).

---

## Comparison Table

| Question of which is / has | Answer Given | Justification Why? |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution | Model A (`+1`) | Model A covers more churn paths (`_mi_page_malloc_zero` + `_mi_page_retire` multi-page small-block logic), so it addresses prompt intent more directly. |
| Better logic and correctness | Model A (`+1`) | In runtime allocator logic, A removes one extra fallback to `_mi_malloc_generic(...)` by reclaiming `page->local_free` inline in `_mi_page_malloc_zero(...)`. |
| Better Naming and Clarity | Model B (`-1`) | `only_page` and `small_front` naming in `B/src/page.c` is clearer to follow than A’s larger conditional branch with queue move-to-front behavior. |
| Better Organization and Clarity | Model B (`-2`) | B separates pass/fail tests (`test-retire.c`) and benchmark (`test-perf.c`), while A mixes both in one `test-perf.c` executable. |
| Better Interface Design | Model B (`-1`) | Neither adds public API, but B keeps CI-facing tests and perf benchmark separated in CMake, which is cleaner for users and maintainers. |
| Better error handling and robustness | Model B (`-1`) | B’s dedicated `retire` test target in `ctest` is a stronger default safety net than A’s benchmark-heavy `test-perf` ctest integration. |
| Better comments and documentation | Model B (`-1`) | B has clearer inline intent comments in `_mi_page_retire(...)` and detailed structured test sections in `test-retire.c`. |
| Ready for review / merge | Model B (`-1`) | B is closer because test structure is cleaner; A should first split deterministic tests from heavy benchmark execution in `ctest`. |

## Final judgment
Model A is slightly better on raw optimization coverage for the requested churn issue, but Model B is better organized for testing and CI usage.

If choosing one base branch:
- Choose **Model A** if priority is maximizing allocator-path optimization now.
- Choose **Model B** if priority is test/CI structure and lower integration friction.

Practical recommendation:
- Combine A’s allocation fast-path reclaim (`_mi_page_malloc_zero(...)`) with B’s split test strategy (`test-retire.c` pass/fail + manual `test-perf.c`).
