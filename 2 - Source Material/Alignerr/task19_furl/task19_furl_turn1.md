# Updated Review: Prompt 1 (`origin` vs `netloc` parsing consistency)

Prompt reviewed:

> Fix, inconsistent parsing paths leading to creation of surprising differences between `origin` and `netloc` assignments.

Scale reminder for table scores:

- `+4` = Model A fully wins
- `-4` = Model B fully wins

I reviewed current code only (not memory) in `A/` and `B/`.

## Model A Review

### Changed files

- `A/furl/furl.py:1550` (modified `origin` setter)

### Full flow: how `origin` is applied

1. Entry path 1: direct assignment calls `furl.origin` setter in `A/furl/furl.py:1551`.
2. Entry path 2: `furl.__init__(origin=...)` in `A/furl/furl.py:1367` calls `self.set(...)` in `A/furl/furl.py:1382`, then `set()` applies `self.origin = origin` in `A/furl/furl.py:1731`.
3. Inside `origin` setter (`A/furl/furl.py:1551`):
- If `origin is None`, it clears with `self.scheme = self.netloc = None`.
- Else it splits by `://`. If scheme exists, it assigns `self.scheme` immediately.
- It parses `host_port`. If IPv6 bracket is present, it uses `rfind()` + `rsplit(':', 1)` when port is right after `]`.
- Otherwise it sets `self.host` only, or sets both `self.host, self.port`.
4. Final URL output comes from `self.url` / `tostr()` and uses `scheme`, `host`, `port` state.

### New or modified functions/parameters (2-line explanation each)

- `furl.origin(self, origin)` setter in `A/furl/furl.py:1551` (modified).
  It now uses IPv6-aware split logic (`rsplit`) so `http://[::1]:8080` can parse like `netloc`. It tries to avoid wrong split on the first colon inside IPv6.

- Parameter behavior touched: `origin` parameter used by `furl.set(origin=...)` in `A/furl/furl.py:1632` and `furl.__init__(origin=...)` in `A/furl/furl.py:1367`.
  Signature is unchanged, but runtime behavior changed because setter logic changed. No new parameter was added.

### Tests added and what they prove

- No new tests were added in Model A.
- Existing `test_origin` in `A/tests/test_furl.py:1714` still checks malformed IPv6 and invalid ports raise errors.
- Missing proof in Model A:
- No positive IPv6 parse assertion for `origin` like `http://[::1]:8080`.
- No test that failed `origin` update keeps old URL unchanged.
- No test for stale old port when new origin has no explicit port.

### Pros (with exact function/parameter names)

- In `origin` setter (`A/furl/furl.py:1551`), using `rsplit(':', 1)` is better than old left split for IPv6 with port.
- The IPv6 branch in `origin` setter (`A/furl/furl.py:1562`) reduces one major parse mismatch with `netloc` for `host='[::1]'` and `port='8080'`.

### Cons (with exact function/parameter names)

- `origin` setter (`A/furl/furl.py:1551`) still keeps stale old port when new `origin` has no port (example: from `http://good.com:999/path` to `origin='http://example.com'` becomes `http://example.com:999/path`). `netloc` setter does not do this.
- `origin` setter assigns `self.scheme` early and uses direct `self.host, self.port = ...`; if `self.port` raises, object can be partially changed. Example: invalid port leaves host changed.
- No matching tests were added in `A/tests/test_furl.py` to lock the expected parity with `netloc`.

### PR readiness

- Not ready for merge.
- It fixes part of the issue (IPv6 split) but still leaves surprising `origin` vs `netloc` differences and partial-update risk.

### Concrete next-turn fixes

1. In `A/furl/furl.py:1551`, parse into local `scheme/host/port` first, then assign in safe order (`port` before `host`, and scheme last), like `netloc` style.
2. In `A/furl/furl.py:1551`, when `origin` has no explicit port, clear old port state so `origin` and `netloc` behave the same.
3. Add tests in `A/tests/test_furl.py:1714`:
- `origin='http://example.com'` after starting with explicit port should not keep old `:999`.
- failed origin set with invalid port should keep full original URL unchanged.
- positive IPv6 cases (`[::1]` and `[::1]:8080`) for host/port/url checks.

## Model B Review

### Changed files

- `B/furl/furl.py:1551` (modified `origin` setter)
- `B/tests/test_furl.py:1714` (expanded `test_origin`)

### Full flow: how `origin` is applied

1. Entry path 1: direct assignment calls `furl.origin` setter in `B/furl/furl.py:1551`.
2. Entry path 2: `furl.__init__(origin=...)` in `B/furl/furl.py:1367` calls `self.set(...)` in `B/furl/furl.py:1382`, then `set()` applies `self.origin = origin` in `B/furl/furl.py:1752`.
3. Inside `origin` setter (`B/furl/furl.py:1551`):
- It first validates using `urllib.parse.urlsplit(...)` (with `http://` prefix when no scheme).
- It parses to local `scheme`, `host`, `port` with IPv6 bracket-aware logic using `rfind()` and `rsplit(':', 1)`.
- It applies state in safer order: set port first, then host, then scheme.
- If no explicit port was present, it clears stored port (`self._port = None`) so default comes from scheme.
4. URL output then uses the final consistent state in `self.url` / `tostr()`.

### New or modified functions/parameters (2-line explanation each)

- `furl.origin(self, origin)` setter in `B/furl/furl.py:1551` (modified).
  It now mirrors `netloc` parse rules more closely and adds pre-validation with `urlsplit`. It also avoids partial object updates by using local variables then safe assignment order.

- `test_origin()` in `B/tests/test_furl.py:1714` (modified).
  New checks verify positive IPv6 parsing and failure rollback behavior. This directly tests the bug area instead of only testing exceptions.

- Parameter behavior touched: `origin` parameter in `furl.set(origin=...)` and `furl.__init__(origin=...)`.
  No signature change, but behavior is more consistent with `netloc` setter when parsing/applying host and port.

### Tests added and what each test proves

Added in `B/tests/test_furl.py:1730` to `B/tests/test_furl.py:1758`:

- IPv6 positive case with port:
  `f.origin = 'http://[::1]:8080'` proves `origin` setter now stores `host='[::1]'` and `port=8080` correctly and keeps path.

- IPv6 positive case without explicit port:
  `f.origin = 'http://[::1]'` proves default port behavior works (`port==80`) and URL rendering is correct.

- Error rollback case:
  invalid port in `origin` raises `ValueError` and URL remains unchanged, proving no partial update remains after failure.

Edge-case coverage status:

- Good: IPv6 host with and without explicit port.
- Good: invalid port side-effect rollback.
- Still missing: direct test where old non-default port exists and new origin has no port (should clear old port).

### Pros (with exact function/parameter names)

- `origin` setter in `B/furl/furl.py:1551` now follows same parse shape as `netloc` setter (`B/furl/furl.py:1496`), reducing inconsistent behavior.
- `urlsplit` pre-validation in `B/furl/furl.py:1555` catches malformed IPv6 early and clearly.
- Safe apply order in `B/furl/furl.py:1583` to `B/furl/furl.py:1594` avoids partial updates when `port` or `host` is invalid.
- Added tests in `B/tests/test_furl.py:1730` to `B/tests/test_furl.py:1758` validate both success and failure paths.

### Cons (with exact function/parameter names)

- `origin` setter uses `self._port = None` directly in `B/furl/furl.py:1591`, which bypasses `port` property setter path. It works, but mixes direct field write with property APIs.
- Parse logic is now duplicated between `origin` setter (`B/furl/furl.py:1551`) and `netloc` setter (`B/furl/furl.py:1496`), so future maintenance can drift.

### PR readiness

- Ready with minor follow-up.
- Core issue is addressed better than Model A, and test coverage was improved for this area.

### Concrete next-turn fixes

1. Add one targeted test in `B/tests/test_furl.py:1714` for stale-port reset case:
   start with `http://good.com:999/path`, set `origin='http://example.com'`, expect `http://example.com/path`.
2. Optional cleanup: extract shared host/port parsing helper used by both `netloc` and `origin` setters to avoid code drift.
3. Optional style cleanup: document why `self._port = None` is intentional, or refactor to avoid direct private-field write.

## Comparison Table

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution              | `-3 (Model B)`      | Model B fixes more of the real inconsistency and adds tests. Model A fixes IPv6 split only, but still leaves stale-port mismatch and partial-update risk in `origin` setter. |
| Better logic and correctness         | `-4 (Model B)`      | `B/furl/furl.py:1551` handles parse/validation/apply order safely and keeps `origin` close to `netloc`. In `A/furl/furl.py:1551`, `origin='http://example.com'` can keep old `:999`, which is still surprising and inconsistent. |
| Better Naming and Clarity            | `-1 (Model B)`      | Model B uses clear local names (`scheme`, `host`, `port`) and comments around assignment order. Model A is shorter but less explicit about failure safety. |
| Better Organization and Clarity      | `-2 (Model B)`      | Model B separates parse phase from apply phase in `origin` setter. Model A does direct mutations while parsing, which is harder to reason about on exceptions. |
| Better Interface Design              | `-1 (Model B)`      | Public API is unchanged in both, but Model B behavior of `origin` is closer to `netloc` expectations for users. That makes interface behavior more predictable. |
| Better error handling and robustness | `-4 (Model B)`      | Model B validates early and avoids partial state updates (`B/furl/furl.py:1555`, `B/furl/furl.py:1583`). Model A can partially update host/scheme when port parse fails. |
| Better comments and documentation    | `-1 (Model B)`      | Model B adds comments explaining why apply order matters and how no-port is handled. Model A adds minimal comments only for IPv6 split. |
| Ready for review / merge             | `-3 (Model B)`      | Model B is near-merge with a small follow-up test. Model A still has behavior differences against prompt goal, so not PR-ready yet. |

## Final judgment

Model B is better.

Why: the prompt asks to fix inconsistent parse paths between `origin` and `netloc`. Model B actually aligns parsing and failure behavior in `furl.origin` (`B/furl/furl.py:1551`) and adds tests to prove it. Model A improves IPv6 split, but still leaves important inconsistency (`origin` can keep stale old port) and rollback weakness, so it does not fully solve the issue.

## Verification notes

- I ran focused tests in both models: `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py -k test_origin`.
- Result: both passed that focused test, but runtime behavior probes still show the stale-port and partial-update gap in Model A.
