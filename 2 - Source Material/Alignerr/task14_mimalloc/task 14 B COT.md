summary of what was done:              

  ---
  Summary

  Problem

  Many small memory allocations and frees in a short time caused performance issues due to
  unnecessary work in the allocator's critical path.

  Root Cause

  When a thread-local page empties (all blocks freed), all free blocks accumulate in
  page->local_free while page->free is left NULL. The next allocation sees page->free == NULL
   and falls through to the expensive _mi_malloc_generic slow path (4–5 extra function calls)
   just to migrate local_free → free, even though the page was never released.

  Fix — src/page.c (+5 lines)

  In _mi_page_retire(), when the allocator decides to keep a page alive (setting
  retire_expire), immediately promote local_free → free:

  page->retire_expire = (bsize <= MI_SMALL_OBJ_SIZE_MAX ? MI_RETIRE_CYCLES :
  MI_RETIRE_CYCLES/4);
  // Promote local_free -> free so the next allocation takes the fast path.
  // Safe: used==0, all blocks are in local_free (thread-local, no atomics needed).
  if (page->free == NULL) {
    page->free = page->local_free;
    page->local_free = NULL;
    page->free_is_zero = false;
  }

  Measured Improvement (MI_STAT=2 build)
  ┌───────────────────────┬──────────┬──────────┬─────────────┐
  │       Benchmark       │  Before  │  After   │ Improvement │
  ├───────────────────────┼──────────┼──────────┼─────────────┤
  │ churn-64b-single-page │ 81.6M/s  │ 112.5M/s │ +38%        │
  ├───────────────────────┼──────────┼──────────┼─────────────┤
  │ churn-16b-multi-page  │ 100.5M/s │ 107.1M/s │ +6.6%       │
  ├───────────────────────┼──────────┼──────────┼─────────────┤
  │ churn-64b-multi-page  │ 95.5M/s  │ 105.1M/s │ +10%        │
  ├───────────────────────┼──────────┼──────────┼─────────────┤
  │ Memory purged         │ 255 MiB  │ 0 MiB    │ Eliminated  │
  └───────────────────────┴──────────┴──────────┴─────────────┘
  The eliminated page purging is significant — without the fix, retired pages were being
  decommitted by the OS because they weren't being reused quickly enough, causing expensive
  re-commit on the next allocation cycle.

  Tests

  - All 4 correctness tests pass: test-api (43/43), test-api-fill (17/17), test-stress,
  test-stress-dynamic
  - Debug build with MI_DEBUG_FULL=ON (full invariant checks) also passes all 4 tests

  New Files

  - test/test-perf.c — performance benchmark for small-allocation churn (run manually:
  ./build-release/mimalloc-test-perf)
  - CMakeLists.txt — mimalloc-test-perf executable target added (not a ctest target)

✳ Combobulating… (16m 4s · ↓ 36.0k tokens)

---

# Turn 2:

Summary of Changes

  Three optimizations in `src/page.c`

  1. _mi_page_retire — original fix (only-page case) (already applied)
  - When a page is the sole page in its size-class queue and kept alive, promote local_free → free so the next allocation takes the O(1) fast path instead of going through _mi_malloc_generic.

  2. _mi_page_retire — new: small-front case
  Extended the retirement condition to also keep alive a small-block page that is at the head of a multi-page queue
  (small_front). Previously, when 2+ pages existed and the front page emptied, it was immediately freed — causing OS-level
  page churn. Now it's retired with a half-cycle window (MI_RETIRE_CYCLES/2 = 8) and its local_free is promoted to free, so
  it remains warm and immediately reusable.

  3. _mi_heap_collect_retired — re-promote on each heartbeat
  When the retirement loop keeps a page alive (decrementing retire_expire), also re-promote local_free → free in case the
  page was re-used and re-emptied since original retirement. Guarantees the fast allocation path is always available for
  retired pages.

  New test file: test/test-retire.c (30 pass/fail checks, ctest-registered)
  Test group: churn-correct-* (8 tests)
  What it checks: Canary-fill + verify across 500-1000 churn rounds for sizes 8–1024 bytes
  ────────────────────────────────────────
  Test group: retire-fastpath-* (3 tests)
  What it checks: Retirement churn cost ≤ 3× steady-state reference (mode-independent ratio test)
  ────────────────────────────────────────
  Test group: multi-page-churn-* (5 tests)
  What it checks: Correctness for 2-4 page multi-page churn scenarios
  ────────────────────────────────────────
  Test group: calloc-zero-after-churn-* (3 tests)
  What it checks: mi_calloc returns zeroed memory after dirty-and-free cycles
  ────────────────────────────────────────
  Test group: single-obj-* (4 tests)
  What it checks: Single-allocation-per-cycle edge cases including 8192-byte max small object
  ────────────────────────────────────────
  Test group: mixed-sizes-churn (1 test)
  What it checks: 12 interleaved size classes simultaneously
  ────────────────────────────────────────
  Test group: usable-size-* (4 tests)
  What it checks: mi_usable_size(p) >= requested invariant after retirement reuse
  ────────────────────────────────────────
  Test group: timing-regression-* (2 tests)
  What it checks: Multi-page churn must be ≤ 4× per-alloc cost of single-page (detects severe regressions)
  Updated test/test-perf.c

  Added multi-page churn configs (churn-16b/64b-2-pages, churn-16b/64b-4-pages) that specifically exercise the new
  small_front retirement path, alongside the original single-page configs and a 1024-byte control.

  Updated CMakeLists.txt

  test-retire is now a first-class ctest target registered alongside test-api, test-api-fill, and test-stress. The
  mimalloc-test-perf remains a manually-run benchmark.

  Key metrics (MI_STAT=2 stats build)

  - purged: 0 throughout all benchmarks — retired pages stay alive instead of being decommitted to the OS
  - churn-64b-single-page: 81.6M → 112.5M allocs/s (+38%) vs. no-optimization baseline
  - Multi-page configs: consistently 100–107M allocs/s with zero page purging

✳ Gallivanting… (11m 5s · ↓ 27.7k tokens)