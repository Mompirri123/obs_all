**Created:** *<span color ="cyan"></span>* <span style="color: green; font-style:italic;"> 10.02.26, 00:00 </span>

**UUID:**
42bafde0-09b7-4930-802c-7516a183d063

**Task ID:**
42bafde0-09b7-4930-802c-7516a183d063

**git rev-parse HEAD:**
8ff03b636192e25db17eaaff29e6f75acc9a662b

**Github Link:**
https://github.com/microsoft/mimalloc

**interface code:**
cc_agentic_coding


---
# Saves and Logs - task 14

## Turn 1

### Enhancement:

- Fix, CPU overhead and throughput drops
	- Find and optimise the reset/reuse hot path causing this overhead
	- Check with tests and a before/after comparison 
- multiple allocations and frees are done in a small time by allocator which in turn slows performance improve allocator performance and test for improvement

- [[task 14 A COT]]
- [[task 14 B COT]]

### Turn 1 Prompt:
Fix the following issue where, many small memory allocations and frees which could be avoided by the allocator happen in a short time and are resulting in performance issues. Optimise the allocator and test the improvements

### Turn 1 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	- `if mi_likely(page->local_free != NULL)` checks `local_free` before falling back to the expensive `_mi_malloc_generic`, this helps rapid alloc/free patterns where blocks are in local_free but not yet migrated to free and is also mentioned by model A comments, focuses on performance boost using a faster way when there are available free blocks. Great when there are frequent `alloc`(s) and `free`(s).
	- Good performance benchmarks added in `test-small-alloc-bench.c` and then called in the files `main()`. also checks `alloc()` and `free()` are working properly, `calloc()` inserts 0's properly, and `mi_malloc(0)` not crashing is also checked for. This not only checks for speed but also for safe allocation and free by the allocator. Benchmarks for before and after tests.
	   
- Cons:
	- Tries to implement and improve things more than the scope of the prompt. The problem to solve is about small `alloc()` / `free()` combinations in a short time that could be avoided but A tries to go and improve multiple things like order in which memory can be reused (which of the `free`d blocks to be used first), rules for when to delete a given memory and when is it not needed and how often is the memory cleaned etc;. So in simple terms it not only tries to make memory use faster but also does un asked changes
	- Tests are not complete as they are only benchmarks and also don't compare with the old allocator behaviour before model's changes. More memory being held at once cause of more minimum memory clear size, risky for not having much benefits. unpredictable allocator behaviour with the combo of `free` and `local_free` having different order of reusing memory
	  
#### Model B:
- Pros:
	- Fixes the problem of many small allocations and frees happening by changing `local_free -> free` given `if (page->free == NULL)`, this means next allocations can reuse memory directly avoiding the need to create a new `malloc()` and increasing performance. Also removes / pops memory directly from `page->free` directly when available which means faster handling of `malloc` + `free` blocks. As `_mi_page_retire` is the only function thats changed the changes are simple and easy to maintain and keeps other parts of the allocator unchanged. Doesn't create more data structures (lists enums structs etc;), so there is no memory usage added. Has after and before benchmarks, as it runs `alloc` and `free` repeatedly and compares the performance for before and after, showing improvements as it also integrates the tests into build (Cmake)
	
- Cons:
	- Improves performance only for cases where everything is freed and then allocated, other patterns are not touched, this means the performance boost is very small and does not improve allocations in general cases or for long `alloc`'s. Though benchmarks test and prove performance improvements, there is no test that directly checks the `local->free`to `free` change alone directly. As all the tests added are benchmarks, there are no tests that show a *pass / fail*. Has all the build artefacts present after the model ran and if PR readiness is what we are aiming for this is bad. Though the main drawback of model B is that it only optimises the execution path when the page is the only page in the queue, checked `if (pq->last==page && pq->first==page)` which means no other condition is improved

#### Justification for best overall

I think Model B is better than Model A. The difference between the two models can simply be said like, Model A tries to improve multiple stuff (trying to avoid `_mi_malloc_generic()`, `MI_MIN_EXTEND` etc.;) more than what the prompt is aiming for and under delivers most of them, where as Model B picks one particular thing that could be improved and implements changes and improvements for it neatly. That said, Model B also needs improvements as there are no clear tests to show pass / fail, but only benchmarks and this bench marks also do not check for previous vs new. Also the particular change of `localfree->free` to `->free` is not explicitly checked whether it is the correct thing to do. Model A has better benchmark coverage than Model B but does not compliment for B's better handling of (`local->free`to `->free`) at-least one of the improve able scenarios to a better extent.

---

## Turn 2

### Problems to fix:
- Add a check that forces to follow the 

### Turn 2 Prompt:
There are still some gaps, currently we only improve for the only page in queue condition, add improvements for small-blocks of pages etc.;. Add tests that check explicitly for pass / fail while benchmarks check for numerical improvements.

### Turn 2 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	- if within the `_mi_page_malloc_zero()` function, if the `page->free` is empty, but if `local_free` is not, this memory blocks are moved and are immediately allocated. This stops a `..._generic()` call for a very commonly seen memory allocation pattern. It Not only covers the 'only page in queue' condition, but also when there are small memory blocks belonging to multiple pages, which means better performance improvements. Adds pass / fail tests as requested. Handles multiple cases within `mi_page_retire()`
	
- Cons:
	- `_mi_page_retire()` has many cases and is more complex and hence harder to maintain. Benchmarks are included in `ctest`which means they are not manual and will run during regular test checks and can make the tests very unnecessarily long and it would be better to split tests into separate files depending on type and exclude benchmarks from running in normal test runs. Does not clean build artefacts (as we are checking for PR ready code, I see this as a con). 


#### Model B:
- Pros:
	- Well done tests like `churn_correctness()` and `multi_page_churn_correctness()` together check whether any written memory is damaged after multiple `alloc()`and `free()` cycles / calls. Good separation of tests with benchmarks in  `test-pref.c` and `test-retire` having the pass / fail, retire checking tests. `small_front` and `only_page` conditions both use "shorter retirement window for the non-singleton case so we don't hold and excess committed memory when the multi-page queue shrinks naturally" as said in the `mi_page_block_size()`  comments. `_mi_page_retire()` covers not only the 'only page in queue' condition but also multi page case (small group), so more performance improvements. Changes are simpler to understand. Explains the implementation well in comments.`small_front` explicitly handles the case where there are small memory blocks case.
	
- Cons:
	- If there are blocks only in `local_free`, then allocator still uses the slower `..generic`, which could be improved using pre existing functions if allowed to. Works only when page(s) is at the front of the queue. Mostly timing based tests and needs more explicit checks. Talks about having double free checks but there is none. also `test-retire.c`  could be more rigid and check for more edge cases. Does not clean build artefacts (as we are checking for PR ready code, I see this as a con)

#### Justification for best overall
  - I think Model A is slightly better than B. This is because Model A has better optimisation for the short term small alloc/free blocks which is our main requirement. That said Model B outperforms model B in terms of structure and simplicity. It has better organisation of tests with different types of tests separated and also avoids running the benchmark tests with the normal test sequence. that Model A's `_mi_page_malloc_zero()` and `_mi_page_retire()`are well designed and perform better than model A optimisations, but the tests are not separated and the benchmarks tests run on a normal test run which makes the tests very slow. So, Model A has better functionality and Model B has better modularisation , tests and code structure. The best would be to have a combination of Model A's performance optimisations combined with Model B's better tests and organisation. So in short, Model A's better functionality makes it slightly better than Model B

---

## Turn 3

#### Model A:
- Pros:
	- if within the `_mi_page_malloc_zero()` function, if the `page->free` is empty, but if `local_free` is not, this memory blocks are moved and are immediately allocated. This stops a `..._generic()` call for a very commonly seen memory allocation pattern. It Not only covers the 'only page in queue' condition, but also when there are small memory blocks belonging to multiple pages, which means better performance improvements. Adds pass / fail tests as requested. Handles multiple cases within `mi_page_retire()`
	
- Cons:
	- `_mi_page_retire()` has many cases and is more complex and hence harder to maintain. Benchmarks are included in `ctest`which means they are not manual and will run during regular test checks and can make the tests very unnecessarily long and it would be better to split tests into separate files depending on type and exclude benchmarks from running in normal test runs. Does not clean build artefacts (as we are checking for PR ready code, I see this as a con). 

overall:
- `_mi_page_retire()` has many cases and is more complex and hence harder to maintain. Benchmarks are included in `ctest`which means they are not manual and will run during regular test checks and can make the tests very unnecessarily long and it would be better to split tests into separate files depending on type and exclude benchmarks from running in normal test runs. Does not clean build artefacts (as we are checking for PR ready code, I see this as a con). 

## Problems to fix:

-  Split tests by purpose
    - Keep pass/fail checks in [test-retire.c](https://file+.vscode-resource.vscode-cdn.net/Users/home/.vscode/extensions/openai.chatgpt-0.4.71-darwin-arm64/webview/# "test-retire.c").
    - Keep speed numbers in [test-perf.c](https://file+.vscode-resource.vscode-cdn.net/Users/home/.vscode/extensions/openai.chatgpt-0.4.71-darwin-arm64/webview/# "test-perf.c") 
    - Do not run test-perf in normal ctest.
    - 
- Modularise _mi_page_retire()
    - Break big logic into small helper functions.
    - Keep same behaviour, but make each condition easy to read.
    - 
-  Add direct tests for new logic
    - Add a test for _mi_page_malloc_zero() case: page->free == NULL and page->local_free != NULL.
    - Add a test for small multi-page retire path in _mi_page_retire().
- Add safety checks
    - Add asserts after list moves (local_free -> free) to confirm correct state
      
- Remove build artefacts and temp files.
	
- Keep only source, test, and CMake changes
	
- Yo how many build files !!!

### Turn 3 Prompt:
 
 - The implementation is on track but has some issues, the code should be modularised better and needs to be easy to follow. benchmark tests should be manual and shouldnt run within ctest. Needs explicit tests for newly added functions, also lacks indepth edgecase handling and tests
 
### Turn 3 Eval Table:

| Question of which is / has           | Answer Given | Justoification Why? |
| ------------------------------------ | ------------ | ------------------- |
| Overall Better Solution              |              |                     |
| Better logic and correctness         |              |                     |
| Better Naming and Clarity            |              |                     |
| Better Organization and Clarity      |              |                     |
| Better Interface Design              |              |                     |
| Better error handling and robustness |              |                     |
| Better comments and documentation    |              |                     |
| Ready for review / merge             |              |                     |

### Pros and cons

#### Model A:
- Pros:
	- 
	
- Cons:
	- 


#### Model B:
- Pros:
	- 
	
- Cons:
	- 
	
#### Justification for best overall

