# Tasks in Mimalloc

- [ ] `src/options.c` boolean env parsing should use exact token matching instead of substring matching. User issue: invalid values can be interpreted as valid flags, causing surprising config behavior.
- [x] `src/options.c` strict `stderr` routing behavior cleanup. User issue: error logs are missing, delayed, or appear in unexpected sinks, making production incidents hard to diagnose  
      [[task 15]]
- [ ] `src/options.c` reduce/avoid CRT dependency (`strtol`) in early option init path. User issue: preload or constrained-runtime startup can behave inconsistently when allocator init touches CRT paths.
- [ ] `src/prim/emscripten/prim.c` track first-use zeroed memory state instead of always conservative `is_zero=false`. User issue: WebAssembly apps do extra zeroing work and lose performance in allocation-heavy workloads.
- [ ] `src/arena.c` choose current NUMA node when unset (`-1`). User issue: default configuration gives inconsistent locality, so identical deployments have different performance profiles.
- [ ] `src/prim/unix/prim.c` replace hard-coded huge page size with query/config fallback. User issue: workload uses more CPU and memory bandwidth than expected on some Linux hosts, with inconsistent performance across machines.  
    
- [ ] `include/mimalloc/internal.h` free-list alignment validation. User issue: app crashes randomly with hard-to-reproduce heap corruption after long runtimes, even in code that appears correct
- [x] `src/heap.c` heap reset strategy optimization. User issue: short-lived allocation bursts cause avoidable CPU overhead, reducing throughput in services with frequent heap churn.  
      [[task 14]]
- [ ] `src/prim/unix/prim.c` improve NUMA node discovery on platforms with known gaps (for example DragonFly TODO). User issue: allocator falls back to weaker NUMA decisions and gets lower locality/throughput.
- [ ] `src/page.c` reclaim-path full-queue push behavior tuning. User issue: memory reuse is less efficient under pressure, causing periodic latency spikes and higher tail latencies.
- [ ] `src/bitmap.c` weak CAS optimization and validation. User issue: at high thread counts, allocator overhead spikes and request latency regresses due to contention
- [ ] `src/prim/unix/prim.c` improve/verify huge-page NUMA binding strategy. User issue: multi-socket servers show unstable latency and throughput drops because memory lands on remote NUMA nodes.
- [ ] `src/arena.c` separate “large-page” semantics from `memid.is_pinned`. User issue: certain arena configurations get suboptimal policy decisions because pinned and large-page states are conflated.
- [ ] `src/alloc.c` guarded allocation plus padding compatibility work. User issue: users enabling both guard-style safety and padding-style checks cannot get both protections together as expected.