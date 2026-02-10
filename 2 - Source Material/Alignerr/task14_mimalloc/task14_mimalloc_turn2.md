# Review Update: Model A vs Model B (Prompt 1 + Prompt 2 + Prompt 3)

Prompts reviewed:
1. `Fix the following issue where, many small memory allocations and frees which could be avoided by the allocator happen in a short time and are resulting in performance issues. Optimise the allocator and test the improvements`
2. `There are still some gaps, currently we only improve for the only page in queue condition, add improvements for small-blocks of pages etc.;. Add tests that check explicitly for pass / fail while benchmarks check for numerical improvements`
3. `The implementation is on track but has some issues, the code should be modularised better and needs to be easy to follow. benchmark tests should be manual and shouldnt run within ctest. Needs explicit tests for newly added functions, also lacks indepth edgecase handling and tests`

Scoring scale used in table:
- `+4` = Model A fully wins
- `-4` = Model B fully wins

Note for this pass:
- This is a read-only review (no builds/tests executed here), based on current source files.
- Backward compatibility was not used as a penalty.

## Model A

### Full flow of how the optimization works
No new public runtime option was added. Behavior is automatic in allocator internals.

1. User call chain starts at `mi_malloc(size_t size)` in `A/src/alloc.c:214`.
2. It goes to `mi_heap_malloc(...)` in `A/src/alloc.c:210`, then to `_mi_page_malloc_zero(...)` in `A/src/alloc.c:52` for fast-page allocation.
3. In `_mi_page_malloc_zero(...)`, when `page->free == NULL`, Model A calls `_mi_page_local_free_to_free(page)` (`A/src/alloc.c:60`).
- Trigger: fast free list is empty.
- Action: helper moves `page->local_free -> page->free` (`A/src/alloc.c:38-47`).
- Impact: same function can continue with fast pop instead of immediate `_mi_malloc_generic(...)` fallback.
4. Free call chain starts in `mi_free(...)`, then `mi_free_block_local(...)` (`A/src/free.c:31`).
5. `mi_free_block_local(...)` pushes freed block to `page->local_free` (`A/src/free.c:45-46`), and when `--page->used == 0`, calls `_mi_page_retire(page)` (`A/src/free.c:47-48`).
6. In `_mi_page_retire(...)` (`A/src/page.c:500`), Model A splits retirement handling:
- Sole-page case uses `_mi_page_retire_set_expire(...)` (`A/src/page.c:515-523`).
- Small-block multi-page case (`bsize <= MI_SMALL_OBJ_SIZE_MAX`) may retain one page briefly (`A/src/page.c:526-543`) and move it to queue front.
7. `_mi_page_retire_set_expire(...)` (`A/src/page.c:483`) sets `page->retire_expire`, promotes `local_free -> free` using `_mi_page_retire_promote_free(...)` (`A/src/page.c:469`), and updates retired-range tracking.

Simple summary:
- A optimizes both allocation-time reclaim (`_mi_page_local_free_to_free`) and retirement-time retain/reuse (`_mi_page_retire_set_expire`).

### New or modified functions/parameters
Modified/added in allocator path:
- `A/src/alloc.c`: `_mi_page_local_free_to_free(mi_page_t* page)` (new helper)
- `A/src/alloc.c`: `_mi_page_malloc_zero(...)` (modified call path)
- `A/src/page.c`: `_mi_page_retire_promote_free(mi_page_t* page)` (new helper)
- `A/src/page.c`: `_mi_page_retire_set_expire(...)` (new helper)
- `A/src/page.c`: `_mi_page_retire(...)` (modified logic)

Modified/added in tests/build:
- `A/test/test-churn.c` (new pass/fail file)
- `A/test/test-perf.c` (manual benchmark)
- `A/CMakeLists.txt:742-765` adds `mimalloc-test-churn` as ctest and keeps `mimalloc-test-perf` manual only.

Key fields/parameters used in behavior:
- `page->free`
- `page->local_free`
- `page->retire_expire`
- `pq->first`, `pq->last`
- `bsize` from `mi_page_block_size(page)`

### Tests added and what each proves
Pass/fail tests file: `A/test/test-churn.c`

Explicit tests for newly added code paths:
1. `_mi_page_local_free_to_free` path tests:
- `test_local_free_to_free_basic()` (`A/test/test-churn.c:95`)
- `test_local_free_to_free_free_is_zero()` (`A/test/test-churn.c:126`)
- `test_local_free_to_free_multiple_cycles()` (`A/test/test-churn.c:153`)
- `test_local_free_to_free_empty_page()` (`A/test/test-churn.c:182`)
What these prove: local list promotion works, calloc behavior stays correct, repeated cycles do not lose blocks, and empty case still falls back safely.

2. `_mi_page_retire_promote_free` (sole-page retire) tests:
- `test_retire_promote_page_reuse()` (`A/test/test-churn.c:209`)
- `test_retire_promote_calloc_zero()` (`A/test/test-churn.c:241`)
- `test_retire_promote_memory_integrity()` (`A/test/test-churn.c:267`)
What these prove: same-page reuse after full-free, zeroing correctness, no corruption during reused-page churn.

3. `_mi_page_retire_set_expire` / multi-page retirement tests:
- `test_multi_page_retire_reuse()` (`A/test/test-churn.c:303`)
- `test_multi_page_retire_guard()` (`A/test/test-churn.c:351`)
- `test_multi_page_retire_integrity()` (`A/test/test-churn.c:379`)
What these prove: multi-page reuse works, “one retired page per queue” guard behavior, and data integrity under multi-page churn.

Edge-case depth (strong in A):
- size boundaries: 1 byte (`A/test/test-churn.c:410`), 1024 (`A/test/test-churn.c:431`), 8192 (`A/test/test-churn.c:451`), 8193 (`A/test/test-churn.c:471`)
- `mi_malloc(0)` (`A/test/test-churn.c:489`)
- aligned churn (`A/test/test-churn.c:505`)
- realloc churn (`A/test/test-churn.c:525`)
- interleaved sizes (`A/test/test-churn.c:557`)
- private heap (`A/test/test-churn.c:581`)
- forced collect during churn (`A/test/test-churn.c:606`)
- usable-size consistency (`A/test/test-churn.c:648`)
- duplicate block check (`A/test/test-churn.c:668`)

Benchmark file: `A/test/test-perf.c` (manual, numerical only).

### Pros (Model A)
- Strong modularization for prompt-3 in allocator internals:
  - alloc helper `_mi_page_local_free_to_free(...)` in `A/src/alloc.c:38`
  - retire helpers `_mi_page_retire_promote_free(...)` and `_mi_page_retire_set_expire(...)` in `A/src/page.c:469` and `A/src/page.c:483`.
- Clear optimization call chain:
  - `mi_free_block_local(...)` fills `page->local_free` (`A/src/free.c:45-46`),
  - next `mi_malloc(...)` can reclaim via `_mi_page_local_free_to_free(...)` before generic path.
- Very deep edge-case pass/fail coverage in `A/test/test-churn.c`.
- Benchmark is manual, not ctest (`A/CMakeLists.txt:754-765`) which matches prompt-3.

### Cons (Model A)
- `_mi_page_retire(...)` still has many branches (`A/src/page.c:500-547`) and queue-front movement (`A/src/page.c:540`), so junior maintainers may need time to understand all paths.
- `A/test/test-churn.c` is very large; maintainability of tests can be difficult without splitting by theme/file.
- Build artifacts are present in working tree (PR hygiene issue, though not allocator logic issue).

### PR readiness (Model A)
Status: **Almost ready, but one cleanup iteration recommended**.

Concrete next-turn fixes for A:
1. Split `A/test/test-churn.c` into smaller files (`test-churn-promote.c`, `test-churn-retire.c`, `test-churn-edges.c`) to keep reviewer load low.
2. Add short comments at each early return in `_mi_page_retire(...)` to state exactly why page is retained/freed.
3. Remove build artifacts from PR diff.

---

## Model B

### Full flow of how the optimization works
No new public runtime option was added. Behavior is automatic.

1. Allocation call chain starts in `mi_malloc(size_t size)` (`B/src/alloc.c:208`) and reaches `_mi_page_malloc_zero(...)` (`B/src/alloc.c:31`).
2. In `_mi_page_malloc_zero(...)`, if `page->free == NULL`, B calls shared helper `_mi_page_promote_local_free(page)` from `B/include/mimalloc/internal.h:525` (`B/src/alloc.c:40`).
- Trigger: free list empty on fast path.
- Action: helper moves `page->local_free -> page->free` only when safe (`free==NULL && local_free!=NULL`).
- Impact: next block pop continues fast path; fallback to `_mi_malloc_generic(...)` only if still empty.
3. Free path uses `mi_free_block_local(...)` (`B/src/free.c:31`) same as A:
- freed block goes to `page->local_free` (`B/src/free.c:45-46`),
- `used==0` triggers `_mi_page_retire(page)` (`B/src/free.c:47-48`).
4. In `_mi_page_retire(...)` (`B/src/page.c:492`), B modularizes retirement logic:
- helper `mi_page_retire_track(...)` tracks retired range (`B/src/page.c:463`),
- helper `mi_page_retire_small_in_queue(...)` handles small multi-page retain branch (`B/src/page.c:477`).
5. Small multi-page helper behavior:
- Trigger: `bsize <= MI_SMALL_OBJ_SIZE_MAX` and no retired page already at queue front (`B/src/page.c:478-480`).
- Action: set `retire_expire = MI_RETIRE_CYCLES/4`, promote local free, move page to queue front, track retire range (`B/src/page.c:482-488`).
- Impact: reduces free/re-acquire churn while bounding retained-page count.

Simple summary:
- B uses shared promotion helper across alloc and retire paths, and isolates small-page retire logic into dedicated helper function.

### New or modified functions/parameters
Modified/added in allocator path:
- `B/include/mimalloc/internal.h`: `_mi_page_promote_local_free(mi_page_t* page)` (new shared helper)
- `B/src/alloc.c`: `_mi_page_malloc_zero(...)` uses shared helper
- `B/src/page.c`: `mi_page_retire_track(...)` (new helper)
- `B/src/page.c`: `mi_page_retire_small_in_queue(...)` (new helper)
- `B/src/page.c`: `_mi_page_retire(...)` simplified around helpers

Modified/added in tests/build:
- `B/test/test-churn.c` (pass/fail)
- `B/test/test-perf.c` (manual benchmark)
- `B/CMakeLists.txt:727-752` adds churn into test loop; perf manual only.

Key fields/parameters used:
- `page->free`
- `page->local_free`
- `page->retire_expire`
- `pq->first`, `pq->last`
- `bsize`

### Tests added and what each test proves
Pass/fail file: `B/test/test-churn.c`

Explicit tests for newly added functions/paths:
1. `_mi_page_promote_local_free` path:
- `test_promote_basic()` (`B/test/test-churn.c:53`)
- `test_promote_sizes()` (`B/test/test-churn.c:70`)
- `test_promote_noop_when_free_nonempty()` (`B/test/test-churn.c:88`)
What these prove: promotion triggers in churn, works across sizes, and behaves as no-op when `page->free` is already non-empty.

2. sole-page retire path:
- `test_retire_sole_page_reuse()` (`B/test/test-churn.c:111`)
- `test_retire_sole_repeated()` (`B/test/test-churn.c:128`)
What these prove: repeated reuse and stability when page is sole page in queue.

3. small multi-page retire path:
- `test_retire_multi_page_integrity()` (`B/test/test-churn.c:155`)
- `test_retire_multi_size()` (`B/test/test-churn.c:183`)
What these prove: multi-page churn correctness and multi-size-class behavior.

Edge cases covered in B:
- zero-size (`B/test/test-churn.c:209`)
- single object churn (`B/test/test-churn.c:220`)
- extension boundary (`B/test/test-churn.c:234`)
- calloc after dirty (`B/test/test-churn.c:251`)
- aligned churn (`B/test/test-churn.c:279`)
- partial free (`B/test/test-churn.c:299`)
- realloc churn (`B/test/test-churn.c:327`)
- fresh heap (`B/test/test-churn.c:355`)
- large objects (`B/test/test-churn.c:379`)
- interleaved (`B/test/test-churn.c:403`)

Benchmark file: `B/test/test-perf.c` (manual numerical benchmark).

### Pros (Model B)
- Better modular reuse than A for promotion helper:
  - `_mi_page_promote_local_free(...)` is in shared internal header (`B/include/mimalloc/internal.h:525`), used by alloc and retire logic.
- `_mi_page_retire(...)` is easier to follow due to clear helper extraction (`mi_page_retire_track`, `mi_page_retire_small_in_queue`) in `B/src/page.c:463` and `B/src/page.c:477`.
- Explicit pass/fail tests for new helper/paths are present in `B/test/test-churn.c` with simpler structure than A.
- Benchmark is manual only in CMake, satisfying prompt-3 (`B/CMakeLists.txt:742-752`).

### Cons (Model B)
- Edge-case depth is good but less complete than A:
  - missing explicit boundary checks like 8192/8193 split and duplicate-block uniqueness test present in A.
- `B/test/test-perf.c` has fewer benchmark scenarios than A (A includes single-object and partial-free benchmark configs).
- As with A, build artifacts are present in working tree (PR hygiene issue).

### PR readiness (Model B)
Status: **Almost ready, also needs one cleanup iteration**.

Concrete next-turn fixes for B:
1. Add boundary-focused tests for `MI_SMALL_OBJ_SIZE_MAX` and `MI_SMALL_OBJ_SIZE_MAX+1`.
2. Add duplicate-block uniqueness test after churn promotion path.
3. Expand benchmark matrix with single-object and partial-free patterns.
4. Remove build artifacts from PR diff.

---

## Comparison Table

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution              | Model A (`+1`)      | A and B both satisfy prompt-3 core asks, but A provides deeper edge-case validation around the same optimization paths, so confidence is slightly higher for churn corner cases. |
| Better logic and correctness         | Model A (`+1`)      | Runtime logic is comparable, but A adds more guard-oriented and boundary-oriented correctness tests tied to its new helpers (`_mi_page_local_free_to_free`, `_mi_page_retire_set_expire`). |
| Better Naming and Clarity            | Model B (`-1`)      | B’s helper names and separation (`_mi_page_promote_local_free`, `mi_page_retire_track`, `mi_page_retire_small_in_queue`) are easier for a junior reader to map to behavior. |
| Better Organization and Clarity      | Model B (`-1`)      | B keeps allocator code paths cleaner by extracting shared helper into `internal.h` and reducing repeated list-promotion code in source files. |
| Better Interface Design              | Model B (`-1`)      | No public API changes in either model, but B’s internal interface reuse (`_mi_page_promote_local_free`) is cleaner and reduces duplicate logic. |
| Better error handling and robustness | Model A (`+2`)      | A has wider pass/fail edge coverage (boundary sizes, forced collect, usable-size consistency, duplicate-block check), so bug-detection net is stronger. |
| Better comments and documentation    | Model A (`+1`)      | A’s `test-churn.c` explains each new path and test target in more detail, helping reviewer understanding without opening multiple files. |
| Ready for review / merge             | Tie (`0`)           | Both are close and both now keep benchmarks manual. Each still needs final polish (mainly PR hygiene + a few targeted test additions). |

## Final judgment
Model A is slightly better overall for this prompt sequence because it validates more edge behaviors after adding churn optimizations.

Model B is better in code modularization style and readability, and is also close to merge quality.

Best practical merge strategy:
1. Keep B’s modular helper structure in allocator internals.
2. Keep A’s deeper edge-case test set and bring missing edge tests into B-style structure.
3. Ensure only source/test/CMake files are in final PR (no build artifacts).
