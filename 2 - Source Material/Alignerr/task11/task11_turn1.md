# Turn 1 Evaluation — furl Ordered Query Option (Model A vs Model B)

## Process Flow and Function-Level Notes

### Shared baseline behavior (both models)
- The determinism feature already exists via `Query.encode(sort=True)` and `furl.furl.tostr(query_sort=True)`. Both models lean on this existing API rather than adding a new option name.
- Key implementation points:
  - `furl.furl.Query.encode(..., sort=False)` controls ordering of query params.
  - `furl.furl.furl.tostr(query_sort=True)` passes `query_sort` into query encoding.
  - Sorting is done by key (`items = sorted(items, key=lambda item: str(item[0]))`) in `Query.encode`.

### Model A — How it works (flow)
1. **Query encoding**: no code changes in `Query.encode`; it already accepts `sort`.
2. **URL building**: `furl.furl.tostr(query_sort=True)` triggers sorted query output.
3. **Tests** expand coverage around `Query.encode(sort=True)` and `furl.furl.tostr(query_sort=True)`.

**Relevant functions and parameters**
- `Query.encode(sort=True/False)`:
  - Used in tests to verify deterministic ordering, quoting, delimiters, and stability.
- `furl.furl.tostr(query_sort=True)`:
  - Used in tests to confirm whole-URL determinism without changing defaults.

### Model B — How it works (flow)
1. **Query encoding**: unchanged implementation; still uses `Query.encode(sort=True)`.
2. **URL building**: `furl.furl.tostr(query_sort=True)` used for deterministic URL strings.
3. **Tests** focus on deterministic ordering, stable sorting, mixed key types, and combined options.

**Relevant functions and parameters**
- `Query.encode(sort=True/False)`:
  - Tests for stable ordering with duplicate keys and deterministic output across insertion orders.
- `furl.furl.tostr(query_sort=True)`:
  - Tests for deterministic full URLs and interaction with quote/delimiter options.

---

## Evaluation Table

| Question of which is / has | Answer Given | Justoification Why? |
| --- | --- | --- |
| Overall Better Solution | A slightly better than B | Both use the existing sort option, but A keeps a wider edge‑case test net (dont_quote, empty keys, special chars, “no mutation”). |
| Better logic and correctness | A slightly better than B | A’s tests cover more boundary conditions that are easy to regress. |
| Better Naming and Clarity | Tie | No new names; both keep `sort` / `query_sort`. |
| Better Organization and Clarity | Tie | No structural changes; test additions only. |
| Better Interface Design | Tie | Both preserve default behavior and rely on the existing optional flags. |
| Better error handling and robustness | A slightly better than B | A retains more edge‑case tests that protect against encoding regressions. |
| Better comments and documentation | B slightly better than A | B’s comments explicitly explain stability and determinism, though A is still fine. |
| Ready for review / merge | A slightly better than B | A’s broader test coverage reduces risk; B drops some important cases. |

---

## Model A — Pros and Cons (with function references)

**Pros**
- Model A does a good job at validating the existing `Query.encode(sort=True)` behavior across multiple encoding modes (`quote_plus`, `dont_quote`, special characters), which reduces regression risk in core query serialization.
- Tests verify deterministic output for different insertion orders and ensure `sort=False` still preserves insertion order, so default behavior is unchanged.
- `furl.furl.tostr(query_sort=True)` tests confirm deterministic full URL strings while keeping `.url` default behavior intact.

**Cons**
- No new implementation was added; changes are mostly tests and doc wording.
- It does not add new tests for stable ordering of duplicate keys in the same explicit style as Model B.

---

## Model B — Pros and Cons (with function references)

**Pros**
- Model B adds clear tests that show stable ordering with duplicate keys using `Query.encode(sort=True)`.
- Tests demonstrate deterministic output regardless of insertion order, which aligns with the prompt’s “same inputs → same URL string” requirement.
- `furl.furl.tostr(query_sort=True)` tests cover combinations with `query_quote_plus` and `query_delimiter`.

**Cons**
- Model B removes several edge‑case tests present in A (dont_quote, empty keys, special characters, “sort does not mutate original order”), which weakens coverage for real‑world encoding issues.
- Like A, it does not add new implementation; only tests and comments change.

---

## PR Readiness (by model)

**Model A — PR status: Ready**
- Changes are limited to tests and doc wording.
- Coverage is broad and protects against regressions in key query encoding behaviors.

**Model B — PR status: Almost ready**
- Tests are good but narrower; removal of edge‑case tests is a coverage regression.
- Would be PR‑ready after restoring missing edge‑case tests or adding equivalent replacements.

---

## How to improve in a further turn (if needed)

**Model A improvements**
- Add explicit tests for stable ordering of duplicate keys (like B), to make that property unmistakable.
- Add a small test that shows deterministic output when keys include `None` values mixed with empty strings.

**Model B improvements**
- Restore edge‑case tests for `dont_quote`, empty keys, and special characters.
- Add a regression test to ensure `sort=True` does not mutate the original param order.

---

## Prompt template you could have given (clear and unambiguous)

"Review Model A and Model B changes under folders A and B for this prompt: add an optional deterministic ordering when building URLs, without changing defaults. For each model, explicitly describe the full flow of how the option is applied (functions, parameters, and call chain), list new or modified functions/parameters, and explain which tests were added and what each test proves (including edge cases). Provide pros/cons that reference the exact functions or parameters responsible. Include PR readiness for each model and concrete next‑turn fixes. Finally, output a comparison table (using the provided rating scale) plus a justification of which model is better and why. Write everything into `/Users/home/obs_all/task11_turn1.md` and also copy it into `/Users/home/obs_all/2 - Source Material/Alignerr/task11/`."
