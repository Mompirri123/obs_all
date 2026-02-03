Summary:

  Problem: In smallpond/execution/task.py:2957, when duplicate filenames were detected, the code used enumerate(src_paths) to assign index-based suffixes (data.0.parquet, data.1.parquet, etc.). The src_paths list is built via ThreadPoolExecutor.map(), whose iteration order can vary between runs. This meant the same input could produce different output filenames each time.

  Fix:
  smallpond/execution/task.py — Replaced index-based naming with hash-based naming:

  - Added import hashlib
  - Changed the dedup logic
	  - from:#
		  - f"{p.stem}.{idx}{p.suffix}" for idx, p in enumerate(src_paths)
	  - to:
		  - f"{p.stem}.{hashlib.sha256(str(p).encode()).hexdigest()[:16]}{p.suffix}" for p in src_paths
  - Each file's full source path is hashed with SHA-256, and the first 16 hex characters (64 bits) are used as a suffix. Since each source file has a unique full path, the output name is now:
  - Deterministic — same input path always produces the same output name
  - Order-independent — doesn't depend on thread pool execution order
  - Collision-resistant — 64-bit prefix is more than sufficient for realistic file counts

  tests/test_execution.py — Updated and added tests:

  - Updated test_data_sink_avoid_filename_conflicts to verify all output filenames are unique
  - Added test_data_sink_deterministic_filenames that verifies:
    a. Same inputs produce the same output names across repeated runs
    b. Shuffling input order still maps each path to the same output name
    c. All output names are unique

✻ Simmering… (esc to interrupt · 20m 23s · ↓ 5.1k tokens)