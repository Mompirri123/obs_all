# Model B Summary — Code Context Snippet for `ic()`

## What Model B did
- Added an **optional source snippet** under normal `ic()` output.
- Uses a `showContext` flag and a `contextLines` count to control the snippet.

## How it did it
- Adds `showContext` to the `IceCreamDebugger` constructor and
  `configureOutput(showContext=...)`.
- Adds `contextLines` as a class attribute (default 3).
- In `_format()`, if `showContext` is true, it appends `_formatSnippet()`.
- `_formatSnippet()` uses `inspect.getframeinfo(callFrame, context=2*n+1)` to
  grab lines around the call site and prints them with `>>>` marker and
  line numbers. It also aligns snippet indentation with the current prefix.

## Tests (including edge cases)
- Adds a dedicated snippet test section:
  - Disabled-by-default (no snippet lines).
  - Enabled via `configureOutput(showContext=True)`.
  - Enabled via constructor flag.
  - Custom line counts by modifying `contextLines`.
  - Marker alignment checks (`>>>` vs spaces).
  - Sequential line-number check.
  - No-arg `ic()` snippet generation.
- Edge cases **not** covered:
  - Missing source file or REPL contexts where `code_context` is None.
  - Large files or multi-line expressions that might shift snippet bounds.

## New imports / functions / fields
- **No new imports** for this feature.
- **New field:** `showContext` (bool) on the debugger.
- **New class attribute:** `contextLines` (int, default 3).
- **New method:** `_formatSnippet()`.
- **configureOutput** extended with `showContext`.

## Good about the approach
- Model B does a good job at making the snippet indentation match the
  prefix width. That keeps the snippet visually aligned with `ic|` output.
- The `showContext` flag is simple and easy to reason about.
- Tests check formatting details like marker placement and sequential
  line numbers, which helps prevent subtle regressions.

## Bad about the approach
- `contextLines` is a **class attribute**, so changing it affects all
  instances. That’s more global than needed and can surprise users.
- The snippet depends on `inspect.getframeinfo(..., context=...)`.
  In some runtime environments `code_context` is missing, which would
  cause the snippet to silently disappear (no tests for that).
- Uses `showContext` (name) while the feature is about **code snippets**;
  the naming is a bit less explicit than `includeCodeContext`.

## Other notes specific to Model B
- Does not use `linecache`; relies on Python’s inspect machinery to fetch
  the context block.
