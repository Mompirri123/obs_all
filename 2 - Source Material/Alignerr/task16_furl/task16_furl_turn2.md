# Review Update (Prompt 1 + Prompt 2): Model A vs Model B

Backward compatibility was **not** scored positive or negative, per instruction.

## Model A

### Full flow (how the fix works)
1. Caller uses `f.set(origin='http://[::1]:6767')` in `furl.set()` at `A/furl/furl.py:1632`.
2. Inside `furl.set()`, it applies `self.origin = origin` at `A/furl/furl.py:1731`.
3. `origin.setter` at `A/furl/furl.py:1582` splits scheme and host part with `origin.split('://', 1)`.
4. `origin.setter` calls new helper `extract_host_and_port(host_port)` at `A/furl/furl.py:1593`.
5. `extract_host_and_port()` at `A/furl/furl.py:138` parses:
- `'[ipv6]'`
- `'[ipv6]:port'`
- `'host:port'`
- `'host'`
and raises `ValueError` for invalid format like `'[::1]x:8080'`.
6. Returned values are applied in `origin.setter`:
- `self.host = host` first
- `self.port = port` only when `port is not None`
7. Validation happens in `host.setter` (`A/furl/furl.py:1480`) and `port.setter` (`A/furl/furl.py:1508`).
8. Final URL is rebuilt through `url -> tostr() -> netloc` (`A/furl/furl.py:1598`, `A/furl/furl.py:1831`, `A/furl/furl.py:1525`).

Important behavior note:
- In `origin.setter`, setting `self.host` before `self.port` can leave partial state if `port` is invalid.

### New/modified functions and parameters
- `extract_host_and_port(host_port)` (new) in `A/furl/furl.py:138`.
  - Central helper to parse host/port including IPv6 bracket form.
  - Replaces duplicated split logic previously in setters.

- `furl.netloc(self, netloc)` setter (modified) in `A/furl/furl.py:1541`.
  - Now calls `extract_host_and_port(netloc)`.
  - Keeps existing `port` then `host` assignment order for side-effect safety.

- `furl.origin(self, origin)` setter (modified) in `A/furl/furl.py:1582`.
  - Now calls `extract_host_and_port(host_port)`.
  - But assignment order is `host` then `port`, so invalid `port` can partially update state.

Parameters:
- No new public parameter.
- Existing `origin` and `netloc` parsing behavior changed via shared helper.

### Tests added/changed and what they prove
Changed file: `A/tests/test_furl.py`.

In `TestFurl.test_netloc()`:
- `f.netloc = '[::1]x:8080'` raises `ValueError`.
- `f.netloc = '[::1]invalid:80'` raises `ValueError`.
- Proves helper rejects characters between `]` and `:` for `netloc` path.

In `TestFurl.test_origin()`:
- Positive IPv6 cases:
  - `'http://[::1]:8081'`
  - `'http://[::1]'`
  - `'http://[2001:db8::1]:9000'`
  Proves correct parse + URL reconstruction with path/query/fragment.

- Negative origin cases:
  - `'http://[::1]x:8080'`, `'[::1]invalid:80'`
  - `'http://[::1]:notaport'`, `'http://[::1]:99999999'`
  Proves invalid format/port raise `ValueError`.

Missing test:
- No explicit `origin` no-side-effects assertion after `ValueError`.

### Pros (Model A)
- Good refactor: duplicate split logic removed using `extract_host_and_port()`.
- `netloc.setter` and `origin.setter` now share the same parse path, easier to maintain.
- Added both positive and negative origin tests in `TestFurl.test_origin()`.
- Added extra malformed `netloc` tests in `TestFurl.test_netloc()`.

### Cons (Model A)
- In `origin.setter` (`A/furl/furl.py:1594`), assignment order is `self.host` then `self.port`; invalid port can leave changed host.
- Helper is named `extract_host_and_port` (not private-style), while prompt asked internal helper.
- Missing `origin` rollback/no-side-effect test for failure cases.

### PR readiness (Model A)
- **Not ready for merge yet**.
- Reason: correctness risk in `origin.setter` partial update on invalid port.

### Concrete next-turn fixes (Model A)
1. In `origin.setter`, set `self.port` before `self.host` when `port is not None`.
2. Add tests in `TestFurl.test_origin()` to assert unchanged `f.host`, `f.port`, and `f.url` after invalid origin.
3. Rename helper to private style (example: `_parse_host_port`) for internal intent clarity.

---

## Model B

### Full flow (how the fix works)
1. Caller uses `f.set(origin='http://[::1]:6767')` in `furl.set()` at `B/furl/furl.py:1632`.
2. `furl.set()` applies `self.origin = origin` at `B/furl/furl.py:1731`.
3. `origin.setter` at `B/furl/furl.py:1568` splits scheme/host using `origin.split('://', 1)`.
4. `origin.setter` calls new helper `_parse_host_port(host_port)` at `B/furl/furl.py:1579`.
5. `_parse_host_port()` at `B/furl/furl.py:1337` parses IPv6 and non-IPv6 host/port forms and raises `ValueError` for malformed `']...:'` patterns.
6. Returned values are applied in side-effect-safe order in `origin.setter`:
- `self.port = port` first (if provided)
- then `self.host = host`
7. Validation still goes through `port.setter` and `host.setter`.
8. URL rebuild still goes through `url -> tostr() -> netloc`.

Important behavior note:
- `origin.setter` explicitly protects state on invalid port by assigning `port` first.

### New/modified functions and parameters
- `_parse_host_port(host_port)` (new) in `B/furl/furl.py:1337`.
  - Internal helper for host/port parsing (IPv6 + non-IPv6).
  - Used by both `origin.setter` and `netloc.setter`.

- `furl.netloc(self, netloc)` setter (modified) in `B/furl/furl.py:1527`.
  - Replaced custom split logic with `_parse_host_port(netloc)`.
  - Keeps existing safety order for assignment.

- `furl.origin(self, origin)` setter (modified) in `B/furl/furl.py:1568`.
  - Uses `_parse_host_port(host_port)`.
  - Adds explicit side-effect-safe assignment order for invalid port handling.

Parameters:
- No new public parameter.
- Existing `origin` and `netloc` parsing behavior updated via one helper.

### Tests added/changed and what they prove
Changed file: `B/tests/test_furl.py`.

In `TestFurl.test_origin()`:
- Positive IPv6 cases:
  - `'http://[::1]:8081'`
  - `'http://[::1]'`
  - `'http://[2001:db8::1]:9000'`
  Proves regression fix works and URL rebuild is correct.

- No-side-effects negatives:
  - Start from `f = furl.furl('http://[2001:db8::1]:9000/path')`
  - Apply invalid origin `'http://[::1]:notaport'` and `'[::1]bad:8080'`
  - Assert `f.host` and `f.port` unchanged.
  Proves `origin.setter` protects state on failure.

- Additional port negatives:
  - `'http://[::1]:99999'`, `'http://[::1]:0'`
  Proves out-of-range and zero ports raise `ValueError`.

### Pros (Model B)
- Clean internal helper `_parse_host_port()` directly matches prompt intent (single internal parser).
- `origin.setter` uses safer order (`port` then `host`) at `B/furl/furl.py:1583`, avoiding partial updates.
- Added explicit no-side-effect tests in `TestFurl.test_origin()`.
- Good balance of positive + negative origin tests for IPv6.

### Cons (Model B)
- No new `test_netloc()` negative cases for malformed `'[::1]x:8080'` style (A added this).
- `_parse_host_port()` error text is generic (`Invalid host:port`) and less specific than A for missing bracket cases.

### PR readiness (Model B)
- **Ready for review/merge**.
- Reason: duplicate logic removed, origin negatives improved, and state-safety tested.

### Concrete next-turn fixes (Model B)
1. Add one or two `test_netloc()` malformed separator negatives for parity with `origin` tests.
2. Improve helper error message specificity for easier debugging (`missing ']'`, etc.).
3. Optional: small helper comment that parser is reused by both setters.

---

## Test run summary (local)
- Model A: `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py::TestFurl::test_origin tests/test_furl.py::TestFurl::test_netloc` -> **2 passed**.
- Model B: `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py::TestFurl::test_origin tests/test_furl.py::TestFurl::test_netloc` -> **2 passed**.

Also manually validated side-effect difference on invalid origin port:
- In `A`, after `f.origin='http://[::1]:notaport'`, `host` changed unexpectedly.
- In `B`, same invalid assignment keeps previous `host` and `port` unchanged.

---

## Comparison Table (Scale: `+4` = Model A fully wins, `-4` = Model B fully wins)

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution              | Model B (`-3`)      | Both removed duplicated parsing, but `B/origin.setter` is safer on failure and has explicit no-side-effect tests. |
| Better logic and correctness         | Model B (`-3`)      | In `A/origin.setter`, `self.host` is set before `self.port`; invalid port can partially mutate state. `B` avoids this by setting `port` first. |
| Better Naming and Clarity            | Model B (`-1`)      | `_parse_host_port()` clearly signals internal helper intent; `extract_host_and_port()` looks more public. |
| Better Organization and Clarity      | Model B (`-2`)      | `B` refactor is cohesive: helper + setter safety + tests aligned to that behavior. |
| Better Interface Design              | Model B (`-1`)      | No API changes in both, but `B` better preserves object consistency for `origin` setter usage. |
| Better error handling and robustness | Model B (`-4`)      | `B` has both implementation and test proof for no-side-effects in `TestFurl.test_origin()`; `A` currently fails this scenario. |
| Better comments and documentation    | Model A (`+1`)      | `A/extract_host_and_port()` has more detailed doc and examples; `B` comments are shorter. |
| Ready for review / merge             | Model B (`-3`)      | `B` is merge-ready; `A` should fix `origin.setter` assignment order and add rollback-style tests first. |

## Final judgment
**Model B is better after prompt 2.**

Main reason: `Model B` completes the refactor and also fixes the important failure-path behavior in `origin.setter`. `Model A` still has a correctness gap when `origin` contains an invalid port, because `host` can change before the exception is raised.
