# Review: Model A vs Model B for prompt 1

Prompt reviewed:
> Fix the issue where malformed encoded URLs can silently slip through a component but then suddenly warn in another component

I reviewed only current code under `A/` and `B/` and did a read-only comparison.

## Quick Scope and Verification

- Model A changed `A/furl/furl.py` and added `A/tests/test_strict_mode.py`.
- Model B changed `B/furl/furl.py` and `B/tests/test_furl.py`.
- Validation commands run:
1. `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_strict_mode.py` in `A` -> 12 passed.
2. `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py::TestStrictMode` in `B` -> 10 passed.

Backward compatibility note (as requested): I give credit if a model preserves compatibility, but I do not reduce score if it does not.

---

## Model A Review

### Full flow of how `strict=True` is applied

1. Entry point is `furl.furl.__init__(..., strict=False)` in `A/furl/furl.py:1430`.
2. `furl.furl.__init__()` passes `strict` into `URLPathCompositionInterface.__init__()`, `QueryCompositionInterface.__init__()`, and `FragmentCompositionInterface.__init__()` in `A/furl/furl.py:1439-1441`.
3. `furl.furl.load(url)` in `A/furl/furl.py:1452` parses URL, then calls `self.path.load(tokens.path)`, `self.query.load(tokens.query)`, and `self.fragment.load(tokens.fragment)` in `A/furl/furl.py:1475-1477`.
4. Path branch:
`Path.load()` in `A/furl/furl.py:530` calls `Path._segments_from_path(path)` in `A/furl/furl.py:688` for string input.
5. Inside `Path._segments_from_path()`:
first it calls `has_valid_percent_encoding(path)` in `A/furl/furl.py:700`; if malformed and `self.strict`, it warns once (`Malformed percent encoding...`) in `A/furl/furl.py:701-706`.
6. Then it still does context validation via `is_valid_encoded_path_segment(segment)` in `A/furl/furl.py:711`; `warned` flag in `A/furl/furl.py:709` avoids duplicate warnings.
7. Query branch:
`Query._items()` calls `Query._extract_items_from_querystr(querystr)` for string input in `A/furl/furl.py:1141-1142`.
8. Inside `_extract_items_from_querystr()`:
first it calls `has_valid_percent_encoding(querystr)` in `A/furl/furl.py:1154`; if malformed and `self.strict`, it warns once (`Malformed percent encoding...`) in `A/furl/furl.py:1155-1167`.
9. Then per pair, it checks `is_valid_encoded_query_key(key)` and `is_valid_encoded_query_value(value)` in `A/furl/furl.py:1175-1176`; `warned` in `A/furl/furl.py:1173` prevents duplicate warnings.
10. Fragment branch:
`Fragment.load()` in `A/furl/furl.py:1265` routes fragment to `self._path.load(...)` or `self._query.load(...)` based on `?` and `=` rules in `A/furl/furl.py:1272-1294`, so strict behavior comes from `Path`/`Query` code above.

### New/modified functions and parameters

1. `has_valid_percent_encoding(s)` (new) in `A/furl/furl.py:245`.
- Adds a component-neutral malformed `%` checker (`%HH` only).
- Used before context-specific validators, so malformed percent encoding is caught early and consistently.

2. `Path._segments_from_path(self, path)` (modified) in `A/furl/furl.py:688`.
- Adds early malformed check and warning with message `Malformed percent encoding...`.
- Adds `warned` flow control so only one warning appears per input path string.

3. `Query._extract_items_from_querystr(self, querystr)` (modified) in `A/furl/furl.py:1150`.
- Adds early malformed check with dedicated warning message.
- Adds `warned` flow control to avoid repeated warnings for each invalid pair.

4. Parameter behavior: `strict` (existing parameter, behavior expanded).
- No new public parameter was introduced; existing `strict` now enforces malformed-percent consistency in more places.
- This is good reuse of existing interface.

### Tests added and what each test proves

File: `A/tests/test_strict_mode.py`

`TestStrictModeConsistency`:
1. `test_malformed_percent_encoding_path` proves malformed `%` patterns in `Path(..., strict=True)` always warn (`%3Z`, `%GG`, `%`, `%3`, `test%`, `test%1`).
2. `test_malformed_percent_encoding_query` proves same malformed cases warn in `Query(..., strict=True)`.
3. `test_malformed_percent_encoding_fragment` proves `Fragment(..., strict=True)` also warns, so fragment routing path/query still catches malformed input.
4. `test_programmatic_assignment_no_warning` proves direct decoded assignment (`url.query.params[...]`, `url.path.segments = ...`) does not produce false strict warnings.
5. `test_context_appropriate_validation_path` proves path-specific invalid chars still warn in `Path('folder/?', strict=True)`.
6. `test_context_appropriate_validation_query` proves `Query('key=?', strict=True)` does not wrongly complain about `?` in query context.
7. `test_non_strict_mode_no_warnings` proves `strict=False` keeps silent behavior.
8. `test_full_url_with_malformed_encoding` proves end-to-end `furl.furl(..., strict=True)` catches malformed encoding during URL parse.

`TestStrictModeEdgeCases`:
9. `test_valid_percent_encoding` proves valid encodings (`%20`, `%3F`) do not trigger malformed warnings.
10. `test_mixed_valid_invalid_encoding` proves mixed valid + invalid (`%20` plus `%3Z`) still warns.
11. `test_percent_at_end` proves dangling `%` is detected.
12. `test_percent_with_one_char` proves short `%2` sequence is detected.

### Pros (with exact function/parameter references)

1. Strong consistency improvement: `has_valid_percent_encoding()` is reused in both `Path._segments_from_path()` and `Query._extract_items_from_querystr()`.
2. Warning noise is controlled: `warned` in `Path._segments_from_path()` and `Query._extract_items_from_querystr()` gives one warning per input string.
3. Strict flow is clear: existing `strict` parameter from `furl.furl.__init__()` propagates naturally through `Path`/`Query`/`Fragment` constructors.
4. Test coverage is broad and includes edge cases not usually covered (`%`, `%3`, mixed valid-invalid).

### Cons (with exact function/parameter references)

1. `@static_vars(regex=...)` on `has_valid_percent_encoding()` is unused (`A/furl/furl.py:244`), which can confuse future maintainers.
2. In `Query._extract_items_from_querystr()`, `except Exception` around suggestion generation (`A/furl/furl.py:1161`) is too broad and hides specific failure types.
3. Warning suggestion generation in `Path._segments_from_path()` and `Query._extract_items_from_querystr()` can over-encode already-valid `%HH` parts in some mixed inputs, so message may be less precise.

### PR readiness (Model A)

Status: **Mostly ready for review**, but I would ask for one cleanup pass before merge.

Concrete next-turn fixes:
1. Remove or use the unused `regex` static var in `has_valid_percent_encoding()`.
2. Replace broad `except Exception` in `Query._extract_items_from_querystr()` with narrow exceptions.
3. Add one focused test for multi-pair malformed query dedupe, e.g. `a=%3Z&b=%3Z` should warn once.

---

## Model B Review

### Full flow of how `strict=True` is applied

1. Entry point is `furl.furl.__init__(..., strict=False)` in `B/furl/furl.py:1377`.
2. `strict` is passed into `URLPathCompositionInterface`, `QueryCompositionInterface`, and `FragmentCompositionInterface` in `B/furl/furl.py:1386-1389`.
3. `furl.furl.load(url)` in `B/furl/furl.py:1399` then calls `self.path.load(tokens.path)`, `self.query.load(tokens.query)`, `self.fragment.load(tokens.fragment)` in `B/furl/furl.py:1422-1424`.
4. Path branch changed:
`Path.load()` in `B/furl/furl.py:500` calls `Path._segments_from_path(path)` for string paths in `B/furl/furl.py:514`.
5. Inside modified `Path._segments_from_path()` in `B/furl/furl.py:658`:
`any_invalid` is set when any segment fails `is_valid_encoded_path_segment(segment)` in `B/furl/furl.py:670-674`.
6. After loop, if `self.strict and any_invalid`, one warning is raised using whole processed path suggestion in `B/furl/furl.py:677-689`.
7. Query branch is unchanged:
`Query._extract_items_from_querystr()` in `B/furl/furl.py:1115` still validates each pair and warns inside loop in `B/furl/furl.py:1122-1130`.
8. Fragment branch unchanged:
`Fragment.load()` in `B/furl/furl.py:1212` routes to `self._path.load(...)` or `self._query.load(...)`, so strict warning behavior still depends on Path/Query internals.

### New/modified functions and parameters

1. `Path._segments_from_path(self, path)` (modified) in `B/furl/furl.py:658`.
- Warning is moved after segment processing and deduped with `any_invalid`.
- Suggestion now uses processed full path, improving warning message quality.

2. `TestStrictMode` (new test class) in `B/tests/test_furl.py:2370`.
- Adds strict-mode tests for path/query/fragment and full URL behavior.
- Validates one-warning behavior for multi-invalid path segments.

3. Parameter behavior: `strict` (existing parameter, no API change).
- No new parameter added.
- Change is internal to how `strict=True` warning is emitted in path parsing.

### Tests added and what each test proves

File section: `B/tests/test_furl.py:2370-2464`

1. `test_path_improperly_encoded_warns_once` proves one warning for malformed path segment.
2. `test_path_multiple_invalid_segments_warns_once` proves path warning dedupe and full corrected suggestion text for multi-segment case.
3. `test_path_valid_encoded_no_warning` proves valid encoded path does not warn.
4. `test_path_non_strict_no_warning` proves `strict=False` stays silent.
5. `test_query_improperly_encoded_warns` proves invalid query key encoding warns.
6. `test_query_improperly_encoded_value_warns` proves invalid query value encoding warns.
7. `test_query_valid_encoded_no_warning` proves valid query encoding is accepted.
8. `test_fragment_path_improperly_encoded_warns` proves fragment path route warns.
9. `test_fragment_query_improperly_encoded_warns` proves fragment query route warns.
10. `test_furl_path_and_query_each_warn_once` proves full URL with bad path+query produces 2 warnings (one per component).

Edge-case gap:
- No direct tests for `%` end, `%3` short sequence, or mixed valid+invalid sequence.
- No direct test for repeated invalid query pairs warning dedupe (`a=%3Z&b=%3Z`).

### Pros (with exact function/parameter references)

1. Good fix quality in `Path._segments_from_path()` using `any_invalid` gives one warning per input path.
2. Warning suggestion quality improved in `Path._segments_from_path()` because it is built from full processed segments.
3. `strict` interface is unchanged and behavior is still predictable through existing constructors.

### Cons (with exact function/parameter references)

1. `Query._extract_items_from_querystr()` stayed unchanged; it still warns inside the pair loop and can emit multiple warnings for one input query.
2. No shared malformed-percent helper means consistency logic is split by component instead of centralized.
3. Test set does not cover malformed `%` edge cases like dangling `%` or short `%3`, and does not cover duplicate-warning behavior for multiple bad query pairs.

Observed behavior example (manual probe):
- In Model B, `Query('a=%3Z&b=%3Z', strict=True)` produced **2 warnings** from `Query._extract_items_from_querystr()` loop.
- In Model A, same input produced **1 warning** due `warned` flow control.

### PR readiness (Model B)

Status: **Not fully ready for merge for this prompt goal**.

Concrete next-turn fixes:
1. Add a shared malformed-percent validator (like `has_valid_percent_encoding()`) and call it in `Query._extract_items_from_querystr()` and path flow.
2. Add query warning dedupe flag in `Query._extract_items_from_querystr()` so one query string gives one warning.
3. Add explicit edge-case tests for `%`, `%3`, `%GG`, and multi-pair malformed query strings.

---

## Comparison Table

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution              | **Model A (+3)**    | Model A addresses malformed percent handling in both `Path._segments_from_path()` and `Query._extract_items_from_querystr()` with a shared helper, not only path-level warning behavior. |
| Better logic and correctness         | **Model A (+3)**    | Model A adds early malformed check and one-warning control in both path and query. Model B improves only path; query still warns per invalid pair loop. |
| Better Naming and Clarity            | **Model A (+2)**    | `has_valid_percent_encoding()` and explicit message split (`Malformed...` vs `Incorrectly...`) communicate intent clearly. Model B naming is fine but narrower. |
| Better Organization and Clarity      | **Model A (+2)**    | Model A centralizes malformed detection logic and keeps strict tests in a dedicated file (`tests/test_strict_mode.py`). |
| Better Interface Design              | **Tie (0)**         | Both keep existing public interface and reuse `strict` parameter without breaking calling style. |
| Better error handling and robustness | **Model A (+3)**    | Model A dedupes warnings in both path and query; Model B query path still emits repeated warnings for one input string with multiple bad pairs. |
| Better comments and documentation    | **Model A (+2)**    | Model A has clearer scenario-oriented tests and comments about malformed encoding consistency and edge conditions. |
| Ready for review / merge             | **Model A (+1)**    | Model A is close; needs minor cleanup. Model B still needs another logic pass in `Query._extract_items_from_querystr()` for full prompt alignment. |

---

## Final Judgment

**Model A is better for this prompt.**

Why:
1. The fix is broader and more aligned to "silent in one component, warns in another" by applying malformed-percent logic in both path and query flows.
2. The strict warning behavior is more controlled (single warning per input string).
3. Tests are stronger on edge cases and consistency.

If I score overall on the requested scale (`+4` = A fully wins, `-4` = B fully wins):

**Final score: +3 (Model A clearly better, but not perfect).**
