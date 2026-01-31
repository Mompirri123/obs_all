# PR Readiness — Turn 2 (Inline Snippet Ordering)

## Model A
**Status:** Not PR‑ready.

**Blockers:**
- Snippet ordering is incorrect: context lines are printed above the `ic|` line,
  so the call site and debug output are not inline.
- Negative `numCodeContextLines` is not validated.
- Tests do not cover inline ordering or negative counts.

**What’s needed:**
- Insert `ic|` output immediately after the `>>>` marker line.
- Clamp negative `numCodeContextLines` to 0.
- Add tests for both behaviors.

## Model B
**Status:** PR‑ready for turn‑2 requirements.

**Why:**
- Inline ordering is implemented and tested.
- Negative line counts are clamped and tested.
- No new imports added and file read happens once per call.

**Non‑blocking gaps to be aware of:**
- Snippet lines do **not** align to custom `ic.prefix` widths (fixed 4‑space indent).
- No tests for missing‑source environments (REPL/frozen apps).
- Reads the entire file into memory per call when snippet is enabled (heavy for large files).
- Snippet can be stale if the source file changes between runs.
