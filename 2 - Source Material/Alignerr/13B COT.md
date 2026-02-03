
  Summary

  Problem: The duplicate filename handling in collect_output_files() was non-deterministic. When duplicate filenames were found, it used enumerate(src_paths) to assign indices, but the order of src_paths could vary between runs (due to pool.map processing order and other factors), causing the same input to produce different output filenames on different runs.

  Solution: Modified smallpond/execution/task.py:2957 to:
  1. Sort the source paths by their full string path before assigning indices
  2. Create a mapping from path to index based on the sorted order
  3. Use this mapping to assign consistent indices regardless of the processing order

  Changes made:
  4. smallpond/execution/task.py: Fixed the duplicate filename handling logic to sort paths before assigning indices, ensuring deterministic output filenames.
  5. tests/test_execution.py: Added test_data_sink_deterministic_filename_on_conflict() test that verifies:
    - The index assignment is based on sorted path order
    - Shuffling input order doesn't change the output filenames
    - The same input always produces the same output