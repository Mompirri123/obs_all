# Model A Summary — Code Context Snippet for `ic()`

## What Model A did
- Added an **optional code-context snippet** appended under normal `ic()` output.
- Introduced toggles to turn the snippet on/off and to control how many lines
  above and below the call site are shown.

## How it did it
- Adds two new configuration fields on `IceCreamDebugger`:
  - `includeCodeContext` (bool) to enable/disable snippet output.
  - `numCodeContextLines` (int) to control how many lines above/below.
- In `_format()`, after the normal output is built, it appends a snippet when
  `includeCodeContext` is true.
- `_formatCodeContext()` uses `inspect.getframeinfo()` to find the file and
  line number, then uses `linecache.getline()` to fetch surrounding lines.
- Snippet lines are printed as:
  - `    >>> 0123 | <line>` for the call site
  - `       0122 | <line>` for surrounding lines

## Tests (including edge cases)
- Adds tests around the snippet behavior:
  - Disabled-by-default check (no `>>>` marker in output).
  - Enabled with args and with no args (snippet is appended after main line).
  - `numCodeContextLines` affects line count.
  - Line-number format includes a `|` separator.
  - `configureOutput(includeCodeContext=...)` toggles the feature on/off.
- Edge cases **not** covered:
  - Missing source files / REPL environments (linecache returns empty).
  - Very small files (near start/end bounds) beyond basic trimming.

## New imports / functions / fields
- **New import:** `linecache` (to read source lines by number).
- **New constants:** `DEFAULT_INCLUDE_CODE_CONTEXT`, `DEFAULT_NUM_CODE_CONTEXT_LINES`.
- **New fields:** `includeCodeContext`, `numCodeContextLines` on the debugger.
- **New method:** `_formatCodeContext()`.
- **configureOutput** extended with `includeCodeContext` and `numCodeContextLines`.

## Good about the approach
- Model A does a good job at separating concerns by adding
  `includeCodeContext` as a toggle for the feature, while
  `numCodeContextLines` controls how many lines to print. This makes the
  feature easy to read and easy to configure.
- Uses `linecache` to fetch the exact lines by number, so the snippet can still
  render even when the normal AST-based source parsing fails.
- Tests cover the basic on/off flow and formatting markers, which is enough to
  catch regressions in the core output shape.

## Bad about the approach
- Snippet indentation is fixed to four spaces and **does not align to the
  `ic` prefix**, so output can look misaligned for custom prefixes.
- No test for source-unavailable cases (e.g., frozen apps, REPL), so behavior
  is unknown in those common scenarios.
- Uses `linecache` without explicit cache invalidation; if files change during
  runtime, the snippet may show stale lines.

## Other notes specific to Model A
- The new options are **per-instance** (stored on `IceCreamDebugger`), which is
  consistent with other `configureOutput` settings.
