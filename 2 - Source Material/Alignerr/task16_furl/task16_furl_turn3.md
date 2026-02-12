# Review (Prompt 1 + Prompt 2 + Prompt 3): Model A vs Model B

Scoring rule used: `+4` means Model A fully wins, `-4` means Model B fully wins.

Backward compatibility was **not** scored up or down, based on your instruction.

## Model A

### Full flow (how the fix is applied)
1. User calls `f.set(origin=...)` in `furl.set()` at `A/furl/furl.py:1632`.
2. `furl.set()` calls `self.origin = origin` at `A/furl/furl.py:1731`.
3. In `origin.setter` (`A/furl/furl.py:1568`), origin is split with `origin.split('://', 1)`.
4. `origin.setter` calls `_parse_host_port(host_port)` at `A/furl/furl.py:1582`.
5. `_parse_host_port()` (`A/furl/furl.py:1337`) parses:
   - `[ipv6]`
   - `[ipv6]:port`
   - `host:port`
   - `host`
   and raises `ValueError` for malformed separator after `]`.
6. If parsed `port` exists, `origin.setter` validates it with `is_valid_port(port)` (`A/furl/furl.py:1586`).
7. After validation, it applies changes directly to private state:
   - `self.scheme = scheme` (if provided)
   - `self._port = port`
   - `self._host = ...` (lowercased)
   at `A/furl/furl.py:1591` to `A/furl/furl.py:1594`.
8. URL output later uses `url -> tostr() -> netloc` (`A/furl/furl.py:1597`, `A/furl/furl.py:1831`, `A/furl/furl.py:1510`).

Netloc path:
- `netloc.setter` (`A/furl/furl.py:1527`) also calls `_parse_host_port(netloc)` (`A/furl/furl.py:1547`) and then applies `self.port` then `self.host` (safe order).

### New/modified functions/parameters (2 lines each)
- `_parse_host_port(host_port)` (new) in `A/furl/furl.py:1337`.
  - Shared internal parser for IPv6 and non-IPv6 host/port split.
  - Removes duplicate split logic in both `origin.setter` and `netloc.setter`.

- `furl.netloc(self, netloc)` setter (modified) in `A/furl/furl.py:1527`.
  - Uses `_parse_host_port()` instead of inline IPv6 parsing.
  - Keeps safer assignment order (`self.port` then `self.host`).

- `furl.origin(self, origin)` setter (modified) in `A/furl/furl.py:1568`.
  - Parses/validates first to avoid side effects on parse/port failure.
  - Applies to `_host`/`_port` directly, not through `host.setter`/`port.setter`.

Parameters:
- No new public parameter.
- Behavior changes are in existing `origin` and `netloc` setter flows.

### Tests added and what each test proves
File: `A/tests/test_furl.py`.

`TestFurl.test_netloc()` updates (`A/tests/test_furl.py:1679`):
- Adds malformed netloc checks:
  - missing `]`
  - missing `[`
  - junk between `]` and `:`
  - empty port (`[::1]:`)
- Adds invalid port checks:
  - non-numeric
  - too large (`99999`, long value)
  - zero (`0`)
  - negative (`-1`)
- Proves `netloc.setter` raises `ValueError` and keeps previous `host`/`port` unchanged.

`TestFurl.test_origin()` updates (`A/tests/test_furl.py:1723`):
- Positive IPv6 coverage:
  - `http://[::1]:8081`
  - `http://[::1]`
  - `http://[2001:db8::1]:9000`
- No-side-effect checks on `ValueError`:
  - invalid port string (`notaport`) and malformed bracket format
  - asserts scheme/host/port unchanged
- Port reset coverage:
  - old custom port should reset when new origin has no explicit port
- Proves core prompt behavior and prompt-3 behavior are covered for common cases.

### Pros
- Good refactor: duplicated host/port parsing moved into `_parse_host_port()` (`A/furl/furl.py:1337`).
- `netloc.setter` path is robust and keeps safe assignment order (`A/furl/furl.py:1552`, `A/furl/furl.py:1553`).
- Test coverage is broad in `test_netloc()` and `test_origin()`, including many edge cases (`A/tests/test_furl.py:1693`, `A/tests/test_furl.py:1751`, `A/tests/test_furl.py:1761`).
- Port-leak case is tested and addressed for origin without explicit port (`A/tests/test_furl.py:1761`).

### Cons
- `origin.setter` bypasses `host.setter` by assigning `self._host` directly (`A/furl/furl.py:1594`).
  - This skips existing host validation and normalization rules in `host.setter` (`A/furl/furl.py:1464`).
- `origin.setter` bypasses `port.setter` by assigning `self._port` directly (`A/furl/furl.py:1593`).
  - This can drift from central setter behavior over time.
- Real regression example: `f.origin = 'http://a/b'` is accepted in Model A, while host validation should reject it.
- IDNA decode path from `host.setter` can be skipped in `origin.setter` due direct `_host` assignment.

### PR readiness (Model A)
- **Not ready for merge**.
- Main blocker: `origin.setter` in `A/furl/furl.py:1568` bypasses `host.setter`/`port.setter`, causing behavior inconsistency and validation gap.

### Concrete next-turn fixes (Model A)
1. In `origin.setter`, use validated setter flow with rollback (same pattern as Model B): assign through `self.port` and `self.host`, not `_port`/`_host`.
2. Add test: `f.origin = 'http://a/b'` should raise `ValueError` and keep state unchanged.
3. Add test for IDNA normalization parity in `origin.setter` vs `host.setter`.

---

## Model B

### Full flow (how the fix is applied)
1. User calls `f.set(origin=...)` in `furl.set()` at `B/furl/furl.py:1632`.
2. `furl.set()` calls `self.origin = origin` at `B/furl/furl.py:1731`.
3. `origin.setter` (`B/furl/furl.py:1568`) splits origin with `origin.split('://', 1)`.
4. It calls `_parse_host_port(host_port)` at `B/furl/furl.py:1582`.
5. `_parse_host_port()` (`B/furl/furl.py:1337`) does shared host/port parsing for IPv6 and non-IPv6.
6. `origin.setter` saves old state (`_scheme`, `_host`, `_port`) at `B/furl/furl.py:1585`.
7. In `try` block:
   - set `self.scheme` (if provided)
   - if port exists: `self.port = port`
   - if no port: reset to scheme default via `self._port = DEFAULT_PORTS.get(self.scheme)`
   - set `self.host = host`
8. If any `ValueError`, rollback old state in `except` (`B/furl/furl.py:1596` to `B/furl/furl.py:1598`).
9. URL rebuild remains through `url -> tostr() -> netloc`.

Netloc path:
- `netloc.setter` (`B/furl/furl.py:1527`) also uses `_parse_host_port(netloc)` (`B/furl/furl.py:1547`) with safe assignment order.

### New/modified functions/parameters (2 lines each)
- `_parse_host_port(host_port)` (new) in `B/furl/furl.py:1337`.
  - Single internal helper for host/port parsing across setters.
  - Removes duplication and keeps IPv6 parsing logic centralized.

- `furl.netloc(self, netloc)` setter (modified) in `B/furl/furl.py:1527`.
  - Uses shared helper instead of duplicated branch logic.
  - Keeps old “port first, host second” safety behavior.

- `furl.origin(self, origin)` setter (modified) in `B/furl/furl.py:1568`.
  - Implements rollback-based atomic update for parse/port/host failures.
  - Resets port when origin has no explicit port to stop old-port leakage.

Parameters:
- No new public parameter.
- Behavior is improved for existing `origin`/`netloc` setters.

### Tests added and what each test proves
File: `B/tests/test_furl.py`.

`TestFurl.test_netloc()` updates (`B/tests/test_furl.py:1679`):
- Adds malformed netloc tests:
  - missing bracket
  - junk between `]` and `:`
- Adds invalid port tests:
  - non-numeric
  - zero
  - out of range
- Confirms no-side-effect after ValueError for netloc path.

`TestFurl.test_origin()` updates (`B/tests/test_furl.py:1721`):
- Positive IPv6 tests for with/without port.
- Port reset tests when new origin omits port (http/https/custom scheme cases).
- Atomic no-side-effect tests for failures (scheme/host/port unchanged).
- Confirms prompt-1 + prompt-2 + prompt-3 goals in the origin path.

### Pros
- Stronger atomic update in `origin.setter` with rollback (`B/furl/furl.py:1585` to `B/furl/furl.py:1598`).
- Uses `self.host` and `self.port` setters, so central validation stays active (`B/furl/furl.py:1590`, `B/furl/furl.py:1595`).
- Explicit fix for old-port leak when no explicit port is given (`B/furl/furl.py:1592` to `B/furl/furl.py:1594`).
- Tests cover positives, negatives, no-side-effects, and scheme-default port resets (`B/tests/test_furl.py:1749`, `B/tests/test_furl.py:1764`).

### Cons
- `_parse_host_port()` error text is generic (`Invalid host:port`) and could be more specific for debugging.
- `test_netloc()` coverage is good, but slightly less broad than A (A also checks empty port and negative `-1`).

### PR readiness (Model B)
- **Ready for review/merge**.
- Remaining items are minor quality improvements, not correctness blockers for this prompt sequence.

### Concrete next-turn fixes (Model B)
1. Improve helper error messages in `_parse_host_port()` with clearer reason (`missing bracket`, `junk after ]`, etc.).
2. Add extra malformed netloc tests for parity with A (`[::1]:`, `example.com:-1`).
3. Optionally replace direct `_port = DEFAULT_PORTS.get(self.scheme)` with `self.port = None` for setter consistency.

---

## Comparison Table (rating: +4 A wins, -4 B wins)

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Overall Better Solution              | Model B (`-3`)      | Both handle IPv6 and shared parser refactor, but `B/origin.setter` keeps setter validation and rollback atomicity better. |
| Better logic and correctness         | Model B (`-4`)      | `A/origin.setter` directly sets `_host`/`_port` and can accept invalid host forms (example: `'http://a/b'`), while B rejects via `host.setter`. |
| Better Naming and Clarity            | Tie (`0`)           | Both use the same clear helper name `_parse_host_port` and similar variable names. |
| Better Organization and Clarity      | Model B (`-2`)      | B keeps one consistent validation path (`self.port`, `self.host`) inside rollback; A splits logic between helper validation and direct private assignment. |
| Better Interface Design              | Model B (`-2`)      | B preserves existing setter-driven behavior for `origin` changes; A bypasses setter interface internally. |
| Better error handling and robustness | Model B (`-4`)      | B rollback in `origin.setter` protects full state and keeps host validation active; A has a validation gap due direct `_host` assignment. |
| Better comments and documentation    | Model A (`+1`)      | A test comments are slightly more detailed for malformed netloc variants and examples. |
| Ready for review / merge             | Model B (`-3`)      | B is functionally solid for prompt 1/2/3; A still needs one correction in `origin.setter` path before safe merge. |

## Final judgment
**Model B is better after prompt 3.**

Why: Model B solves the requested issues while keeping validation flow through `host.setter`/`port.setter` and adding rollback for no-side-effects. Model A added many good tests, but `A/furl/furl.py:1593` and `A/furl/furl.py:1594` bypass the setter checks, which creates a correctness risk in `origin.setter`.

## Validation run notes
I ran these targeted tests in both models:
- `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py::TestFurl::test_origin tests/test_furl.py::TestFurl::test_netloc`
- Result: both A and B `2 passed`.

I also ran manual behavior checks:
- In A, `f.origin = 'http://a/b'` was accepted (invalid host should be rejected).
- In B, same input raised `ValueError` and old state stayed unchanged.
