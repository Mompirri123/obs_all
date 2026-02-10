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