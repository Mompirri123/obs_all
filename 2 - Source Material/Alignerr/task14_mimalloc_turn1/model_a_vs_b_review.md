# Review Update: Model A vs Model B (Prompt 1)

Prompt reviewed:
`Fix the following issue where, many small memory allocations and frees which could be avoided by the allocator happen in a short time and are resulting in performance issues. Optimise the allocator and test the improvements`

Scoring scale used in table:
- `+4` = Model A fully wins
- `-4` = Model B fully wins

Backward compatibility note used for this review:
- I did not subtract points only because a change can affect backward compatibility.
- If a model added compatibility-safe behavior, I counted that as a positive.

## Model A

### Findings First (JR-level review)
1. High risk: `A/src/page.c:487` keeps more pages retired with `is_high_capacity_small` in `_mi_page_retire(mi_page_t* page)`. This can increase memory retention and delay page release when churn is high.
2. High risk: `A/src/page.c:245` swaps `page->local_free` and `page->free` in `_mi_page_free_collect(mi_page_t* page, bool force)` when `force=false`. This is non-standard list handling and can create unstable reuse order across collect calls.
3. Medium risk: `A/src/page.c:1014` changes `_mi_malloc_generic(...)` housekeeping trigger from `100` to `250` via `heap->generic_count`, so deferred free and delayed free handling run less often.
4. Medium risk: `A/src/page.c:640` changes `MI_MIN_EXTEND` from `4` to `8`, which may touch/commit more memory per page extension.
5. Medium confidence gap: benchmark is not integrated in CMake/CTest (`A/test/test-small-alloc-bench.c` is standalone; no target in `A/CMakeLists.txt`).

### Full flow of how the optimization is applied
There is no new user option/flag. The optimization is automatic in allocator internals.

1. Allocation call chain starts in `mi_malloc(size_t size)` at `A/src/alloc.c:213`.
2. `mi_malloc()` calls `mi_heap_malloc()` at `A/src/alloc.c:209`, then `_mi_heap_malloc_zero()` at `A/src/alloc.c:205`, then internal fast path (`_mi_page_malloc_zero(...)`).
3. In `_mi_page_malloc_zero(mi_heap_t* heap, mi_page_t* page, size_t size, bool zero, size_t* usable)` at `A/src/alloc.c:31`, Model A adds this branch:
- If `page->free == NULL` and `page->local_free != NULL`, move `local_free -> free` (`A/src/alloc.c:41-45`) and allocate directly.
- Else fallback to `_mi_malloc_generic(...)` (`A/src/alloc.c:48`).
4. Free call chain starts in `mi_free(void* p)` at `A/src/free.c:177`, then `mi_free_ex(...)`, then `mi_free_block_local(...)` at `A/src/free.c:31`.
5. `mi_free_block_local(...)` pushes freed blocks to `page->local_free` (`A/src/free.c:45-46`), and when page becomes empty (`--page->used == 0`), it calls `_mi_page_retire(page)` (`A/src/free.c:47-48`).
6. In `_mi_page_retire(...)` at `A/src/page.c:470`, Model A broadens keep-alive condition:
- keep if only page in queue (`is_only_page`), or
- keep if small block page with `page->capacity >= 4` (`is_high_capacity_small`) (`A/src/page.c:489-491`).
7. In `_mi_page_free_collect(mi_page_t* page, bool force)` at `A/src/page.c:217`, when both lists exist and `force=false`, Model A swaps `page->local_free` and `page->free` (`A/src/page.c:245-251`) to prioritize recent frees.
8. In `_mi_malloc_generic(...)` at `A/src/page.c:1001`, admin work runs every `250` calls (`A/src/page.c:1014`) instead of baseline `100`.
9. In page extension path, `MI_MIN_EXTEND` is changed to `8` (`A/src/page.c:640`).

### New or modified functions/parameters
Modified:
- `_mi_page_malloc_zero(...)` in `A/src/alloc.c:31`
- `_mi_page_free_collect(mi_page_t* page, bool force)` in `A/src/page.c:217`
- `_mi_page_retire(mi_page_t* page)` in `A/src/page.c:470`
- `_mi_malloc_generic(...)` in `A/src/page.c:1001` (`heap->generic_count` threshold usage)
- `MI_MIN_EXTEND` macro in `A/src/page.c:640`

Added:
- `A/test/test-small-alloc-bench.c`

No new public API parameter was added.

### Tests added and what each test proves
File: `A/test/test-small-alloc-bench.c`

1. `test_pingpong(size_t alloc_size, size_t iterations)` at `A/test/test-small-alloc-bench.c:35`
- Proves repeated `mi_malloc`/`mi_free` can run in tight loop for fixed size.
- Edge case: smallest tested size is 16 bytes (`main` uses 16/64/256).

2. `test_batch_lifo(size_t alloc_size, size_t batch_size, size_t iterations)` at `A/test/test-small-alloc-bench.c:50`
- Proves batch allocate + reverse free pattern stability.
- Edge case: reverse-order free path pressure.

3. `test_interleaved(size_t alloc_size, size_t pool_size, size_t iterations)` at `A/test/test-small-alloc-bench.c:75`
- Proves mixed alloc/free under pseudo-random index churn.
- Edge case: half prefilled pool and repeated allocate/free toggling.

4. `test_multisize(size_t iterations)` at `A/test/test-small-alloc-bench.c:111`
- Proves mixed small size-class pattern runs without crash.
- Edge case: several size classes (`8..256`).

5. `test_mt_pingpong(...)` at `A/test/test-small-alloc-bench.c:141`
- Proves 4-thread churn executes and measures wall time.
- Edge case: multi-thread contention pattern.

6. `test_realloc_pattern(size_t iterations)` at `A/test/test-small-alloc-bench.c:165`
- Proves repeated growth realloc loops work.
- Edge case: `mi_realloc(NULL, sz)` then incremental growth.

7. `test_correctness(void)` at `A/test/test-small-alloc-bench.c:182`
- Proves basic alloc/free, write/read integrity, `mi_malloc(0)` non-crash behavior, and `mi_calloc` zeroing.
- Edge case: `size=0` allocation and data integrity loops.

Important gap in A tests:
- No pass/fail performance threshold against baseline.
- Reuse check in `test_correctness` is non-assertive (`p1 == p2` is “good” but not required) at `A/test/test-small-alloc-bench.c:193-204`.
- Not integrated into CMake/CTest.

### Pros (function/parameter specific)
- Strong fast-path idea in `_mi_page_malloc_zero(...)` (`A/src/alloc.c:39-46`) directly avoids `_mi_malloc_generic(...)` when blocks exist in `page->local_free`.
- Good workload coverage in `A/test/test-small-alloc-bench.c` (single-thread, multi-thread, multisize, realloc).
- Adds correctness checks (`test_correctness`) in same benchmark file.

### Cons (function/parameter specific)
- `_mi_page_free_collect(...)` swap logic (`A/src/page.c:245-251`) is hard to reason about and may cause list-order oscillation.
- `_mi_page_retire(...)` broader keep-alive (`A/src/page.c:487-494`) can hold pages longer than needed.
- `heap->generic_count` threshold change in `_mi_malloc_generic(...)` (`A/src/page.c:1014`) can delay maintenance actions.
- `MI_MIN_EXTEND` increase (`A/src/page.c:640`) may increase touched memory for small pages.
- Benchmark is not build-system integrated.

### PR readiness (Model A)
Status: Not ready for merge.

Why:
- Multiple allocator heuristics changed in one patch (`_mi_page_malloc_zero`, `_mi_page_free_collect`, `_mi_page_retire`, `_mi_malloc_generic`, `MI_MIN_EXTEND`) without isolated proof per change.
- Benchmark exists but is manual and has no baseline assertion threshold.

### Concrete next-turn fixes (Model A)
1. Keep only one allocator behavior change first (start with `_mi_page_malloc_zero` local promotion) and revert heuristic extras temporarily.
2. Revert `_mi_page_free_collect(...)` swap path or guard it behind strict condition with explicit invariant checks.
3. Revert `MI_MIN_EXTEND` to `4` unless memory-impact data is added.
4. Revert generic threshold to `100` unless delayed-free regression data proves safe.
5. Add CMake target and optional CI mode for the benchmark, plus reproducible before/after script.

---

## Model B

### Findings First (JR-level review)
1. Low/medium risk: no correctness test added for the new retire-time promotion path (`page->local_free -> page->free`), only performance benchmark.
2. Low risk: benchmark is manual (not CTest), so CI does not enforce perf signal.
3. Positive: allocator change is focused to one place (`_mi_page_retire(...)`) and keeps baseline behavior elsewhere (`_mi_page_malloc_zero`, `_mi_page_free_collect`, `_mi_malloc_generic`).

### Full flow of how the optimization is applied
There is no new user option/flag. The optimization is automatic and narrow.

1. Free path starts at `mi_free(void* p)` in `B/src/free.c:177`.
2. `mi_free()` reaches `mi_free_block_local(...)` in `B/src/free.c:31`, which pushes block to `page->local_free` (`B/src/free.c:45-46`).
3. When page becomes empty (`--page->used == 0`), allocator calls `_mi_page_retire(page)` (`B/src/free.c:47-48`).
4. Model B change is inside `_mi_page_retire(mi_page_t* page)` at `B/src/page.c:462`:
- If page is only page in queue (`pq->first==page && pq->last==page`) and `page->free == NULL`, it promotes `page->local_free` to `page->free` (`B/src/page.c:484-487`).
5. Next allocation starts at `mi_malloc(size_t size)` in `B/src/alloc.c:203` and reaches `_mi_page_malloc_zero(...)` in `B/src/alloc.c:31`.
6. Because `page->free` was pre-populated at retire time, `_mi_page_malloc_zero(...)` can pop directly from `page->free` and skip `_mi_malloc_generic(...)` fallback (`B/src/alloc.c:37-40`).

This directly targets the prompt’s churn case: free-all then allocate-again quickly.

### New or modified functions/parameters
Modified:
- `_mi_page_retire(mi_page_t* page)` in `B/src/page.c:462`
- `CMakeLists.txt` adds benchmark target `mimalloc-test-perf` in `B/CMakeLists.txt:742-751`

Added:
- `B/test/test-perf.c`

No public API parameter added.

### Tests added and what each test proves
File: `B/test/test-perf.c`

Main test routine:
- `bench_churn(size_t obj_size, size_t n_objects, int rounds)` at `B/test/test-perf.c:45`
- Pattern: alloc-all -> free-all -> alloc-all -> free-all.
- What it proves: allocator can handle repeated churn and reports throughput (`Mallocs/sec`).

Configs:
- single-page churn: `16`, `64`, `256` byte objects (`B/test/test-perf.c:98-100`)
- multi-page churn: `16`, `64` byte objects (`B/test/test-perf.c:101-102`)
- warmup before measurement (`B/test/test-perf.c:115`)

Edge cases covered:
- single-page and multi-page regimes.
- repeated free-all/alloc-all cycles.

Important gap in B tests:
- No correctness assertions for allocator invariants tied to the new retire path.
- No pass/fail threshold for performance regression.
- No multithread churn case.

### Pros (function/parameter specific)
- Very focused allocator change in `_mi_page_retire(...)` (`B/src/page.c:482-488`), easy to reason and lower regression surface.
- Keeps baseline logic in `_mi_page_malloc_zero(...)` (`B/src/alloc.c:37-40`) and `_mi_page_free_collect(...)` (`B/src/page.c:217-245`).
- Benchmark integrated into build via `mimalloc-test-perf` target (`B/CMakeLists.txt:742-751`).
- Benchmark scenarios are directly aligned with retire-churn issue (`B/test/test-perf.c:43-75`).

### Cons (function/parameter specific)
- New behavior in `_mi_page_retire(...)` lacks dedicated correctness test.
- Benchmark only prints metrics; no automatic threshold/failure condition.
- Benchmark not added as CTest (`B/CMakeLists.txt` comment says manual run).

### PR readiness (Model B)
Status: Closer to ready, but still needs one more test pass.

Why:
- Allocator change is targeted and understandable.
- Build integration is better than A.
- Needs at least one correctness-focused test around retire-time promotion behavior.

### Concrete next-turn fixes (Model B)
1. Add correctness test that forces `mi_free()` -> `_mi_page_retire()` -> next `mi_malloc()` reuse path and validates no corruption.
2. Add multithread churn variant to `test-perf.c` or a separate test file.
3. Add optional perf threshold mode (env var based) for CI perf gating.
4. Keep benchmark manual by default, but add documented script for before/after comparison.

---

## Comparison Table

| **Question**                         | **Which is better** | **Reasoning / Justification** | **Score (A:+4, B:-4)** |
| ------------------------------------ | ------------------- | ----------------------------- | ---------------------- |
| Overall Better Solution              | Model B             | Model B targets the exact churn point in `_mi_page_retire()` with minimal side effects, while Model A changes multiple allocator heuristics at once. | -3 |
| Better logic and correctness         | Model B             | Model B keeps core allocator control paths stable (`_mi_page_malloc_zero`, `_mi_page_free_collect`, `_mi_malloc_generic`) and modifies only retire-time promotion. | -2 |
| Better Naming and Clarity            | Model B             | Comments in `B/src/page.c:482-484` explain safety and intent directly. Model A has more moving parts and harder reasoning across functions. | -1 |
| Better Organization and Clarity      | Model B             | One focused allocator change + one benchmark target in `B/CMakeLists.txt:742-751`; Model A mixes several independent tuning knobs in one patch. | -2 |
| Better Interface Design              | Model B             | No public API changes in either model, but B improves dev workflow by adding build target `mimalloc-test-perf` without changing allocator API. | -1 |
| Better error handling and robustness | Model B             | Smaller change surface lowers regression risk. Model A introduces potential memory/maintenance side effects via `MI_MIN_EXTEND`, retire broadening, and generic threshold change. | -2 |
| Better comments and documentation    | Model A             | `A/test/test-small-alloc-bench.c` has richer scenario descriptions and explicit correctness section; B comments are good but narrower. | +1 |
| Ready for review / merge             | Model B             | B is near-ready with one missing correctness test. A should be split/reduced before merge due to broad heuristic changes. | -2 |

## Final judgment
Model B is better for this prompt.

Why Model B wins:
- The issue is short-time small alloc/free churn.
- Model B addresses this directly where churn creates empty-page retire transitions: `_mi_page_retire()` in `B/src/page.c:462`.
- Model B does not broadly retune unrelated allocator heuristics.
- Model B also adds a build-integrated benchmark target (`B/CMakeLists.txt:742-751`).

What Model A still did well:
- Strong benchmark coverage breadth in `A/test/test-small-alloc-bench.c`.
- Useful idea in `_mi_page_malloc_zero()` local-free promotion.

But final recommendation:
- Prefer Model B as base.
- Borrow Model A’s broader benchmark ideas after narrowing and validating each allocator change independently.

## Verification notes from this pass
- `ctest -N` in `A/build` shows `Total Tests: 0`.
- `ctest -N` in `B/build-release` shows 4 existing tests.
- Ran existing `B/build-release/mimalloc-test-perf` binary and it produced churn throughput output; benchmark runs successfully.
