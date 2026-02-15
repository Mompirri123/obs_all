# Review Update: Prompt 1 + Prompt 2 + Prompt 3 (Model A vs Model B)

Context reviewed:

- Prompt 1: fix parse inconsistency between `origin` and `netloc`.
- Prompt 2: remove risky direct `_port = None` style path, modularize shared logic, add tests.
- Prompt 3: ensure `origin.setter` and `netloc.setter` cannot partially update private state on failure, add tests for `parse_host_port()`, and add invalid-host + valid-port tests.

I reviewed only current code in `A/` and `B/` and re-ran tests.

## Model A

### Full flow of how option is applied

1. Constructor path:
- `furl.__init__()` in `A/furl/furl.py:1367` calls `self.load(url)` and then `self.set(...)` in `A/furl/furl.py:1382`.
- `self.set(...)` can call `self.netloc = netloc` and `self.origin = origin` in `A/furl/furl.py:1760` and `A/furl/furl.py:1763`.

2. Direct property path:
- `f.origin = ...` goes to `origin.setter` in `A/furl/furl.py:1575`.
- `f.netloc = ...` goes to `netloc.setter` in `A/furl/furl.py:1533`.

3. Shared parse path:
- Both setters call `_parse_host_port(host_port, context_name, context_value)` in `A/furl/furl.py:1479`.
- This function splits host/port and handles IPv6 bracket patterns.

4. Apply path in `netloc.setter`:
- Parse `userpass` and host/port in `A/furl/furl.py:1547` and `A/furl/furl.py:1554`.
- Then assign in order: `self.port = port` then `self.host = host` in `A/furl/furl.py:1559` and `A/furl/furl.py:1560`.

5. Apply path in `origin.setter`:
- Parse/validate URL and split scheme/host_port in `A/furl/furl.py:1580` and `A/furl/furl.py:1584`.
- Parse host/port via `_parse_host_port` in `A/furl/furl.py:1591`.
- Apply `self.port` (or `_clear_port`) then `self.host`, then `self.scheme` in `A/furl/furl.py:1596` to `A/furl/furl.py:1605`.

6. Important behavior gap:
- If `self.port` succeeds but `self.host` fails (invalid host + valid port), state becomes partial.
- Example in direct `origin.setter`: old `:88` changed to `:55` even though setter raises.

### New or modified functions/parameters (2-line each)

- `_parse_host_port(host_port, context_name, context_value)` in `A/furl/furl.py:1479` (new).
  Adds shared host/port parsing for `origin.setter` and `netloc.setter`. This improves consistency for IPv6 and port parsing.

- `_clear_port(self)` in `A/furl/furl.py:1505` (new).
  Clears stored `_port` so default port can be derived from scheme later. Used in `origin.setter` when new origin has no explicit port.

- `netloc.setter` in `A/furl/furl.py:1533` (modified).
  Uses shared parser but still applies `port` before `host`. This order can still leave partial update when host is invalid.

- `origin.setter` in `A/furl/furl.py:1575` (modified).
  Uses shared parser and `_clear_port()`, but still applies port before host. This still allows partial update in direct assignment failure.

- Public parameters: unchanged.
  Behavior changed for existing `origin` and `netloc` assignment paths, but no new public parameter was added.

### Tests added and what each proves

Added in `A/tests/test_furl.py` inside `test_netloc()` and `test_origin()`:

- `A/tests/test_furl.py:1710` host-only `netloc` case.
  Proves `f.netloc='newhost.com'` resets effective port to scheme default (`80` for `http`).

- `A/tests/test_furl.py:1717` IPv6 host-only `netloc` case.
  Proves IPv6 without explicit port works and default port is used.

- `A/tests/test_furl.py:1722` `netloc=None` case.
  Proves host/user/password are cleared.

- `A/tests/test_furl.py:1749` and `A/tests/test_furl.py:1755` IPv6 `origin` success cases.
  Proves `origin.setter` correctly handles `[::1]:8080` and `[::1]`.

- `A/tests/test_furl.py:1772` invalid-port rollback check in `origin`.
  Proves URL is unchanged when port parsing fails.

- `A/tests/test_furl.py:1779` to `A/tests/test_furl.py:1818` extra origin behavior checks.
  Proves selected error paths and host-only origin cases; also checks `origin=None` clearing.

Missing tests in A (prompt-3 asks for these):

- No dedicated test function for `_parse_host_port()`.
- No invalid-host + valid-port direct setter test (example `bad..host:55`) to prove no partial update.

### Pros

- Shared parser `_parse_host_port()` reduces duplicated parse logic in `origin.setter` and `netloc.setter`.
- `_clear_port()` is explicit and readable for no-port origin behavior.
- Added tests improve coverage for IPv6 and default-port transitions.

### Cons

- Critical: partial-update bug still exists in direct setters (`origin.setter`, `netloc.setter`) when host validation fails after port assignment.
- Prompt-3 requested tests for parse helper were not added.
- Comment in tests says “after any error” but test set does not include invalid-host + valid-port direct setter path.

### PR readiness

- Not ready for merge.
- Must fix direct setter partial-state update and add missing tests requested in prompt 3.

### Concrete next-turn fixes

1. In `origin.setter` and `netloc.setter`, validate `host` and `port` first into local variables, then assign once.
2. Add direct tests for invalid-host + valid-port in both setters.
3. Add dedicated unit tests for `_parse_host_port()` with normal and malformed inputs.

## Model B

### Full flow of how option is applied

1. Constructor path:
- `furl.__init__()` in `B/furl/furl.py:1367` calls `self.load(url)` then `self.set(...)` in `B/furl/furl.py:1382`.
- `self.set(...)` can call `self.netloc = netloc` and `self.origin = origin` in `B/furl/furl.py:1786` and `B/furl/furl.py:1789`.

2. Direct property path:
- `f.origin = ...` uses `origin.setter` in `B/furl/furl.py:1596`.
- `f.netloc = ...` uses `netloc.setter` in `B/furl/furl.py:1549`.

3. Shared parse and validation path:
- `_parse_host_port(...)` in `B/furl/furl.py:1458` parses host/port.
- `_validate_port(port)` in `B/furl/furl.py:1495` validates and normalizes port.
- `_validate_and_normalize_host(host)` in `B/furl/furl.py:1505` validates and normalizes host.

4. Apply path in `netloc.setter`:
- Parse user/pass + host/port first.
- Validate `port_val` and `host_val` first in `B/furl/furl.py:1574` and `B/furl/furl.py:1577`.
- Only after validation, assign `_port` and `_host` in `B/furl/furl.py:1580` and `B/furl/furl.py:1581`.

5. Apply path in `origin.setter`:
- Split scheme/host_port and parse host/port.
- Validate all pieces first (`port_val`, `host_val`, `scheme_val`) in `B/furl/furl.py:1614` to `B/furl/furl.py:1625`.
- Assign private fields only after success in `B/furl/furl.py:1628` to `B/furl/furl.py:1631`.

6. Prompt-3 outcome:
- Direct setter partial-state bug is fixed in runtime behavior for invalid-host + valid-port path.
- Failure now leaves original state unchanged.

### New or modified functions/parameters (2-line each)

- `_parse_host_port(host_port, context_name, context_value)` in `B/furl/furl.py:1458` (new).
  Central parser for host/port and IPv6 bracket checks. Used by both setters to keep parse logic same.

- `_validate_port(self, port)` in `B/furl/furl.py:1495` (new).
  Dedicated validation for non-`None` port values. Returns normalized integer and raises on invalid values.

- `_validate_and_normalize_host(self, host)` in `B/furl/furl.py:1505` (new).
  Moves host validation and normalization into one reusable place. Used by host setter and both URL setters.

- `host.setter` in `B/furl/furl.py:1432` (modified).
  Now calls `_validate_and_normalize_host()` so behavior is centralized.

- `port.setter` in `B/furl/furl.py:1443` (modified).
  Now calls `_validate_port()` for non-`None` values and preserves existing `None` behavior.

- `netloc.setter` and `origin.setter` in `B/furl/furl.py:1549` and `B/furl/furl.py:1596` (modified).
  Now do validate-first then assign, preventing partial updates on errors.

- Public parameters: unchanged.
  Existing `origin` and `netloc` parameter paths now use stronger internals.

### Tests added and what each proves

Added in `B/tests/test_furl.py` (same locations as Model A):

- `B/tests/test_furl.py:1710` host-only netloc default-port check.
  Proves default-port behavior when explicit port is removed.

- `B/tests/test_furl.py:1717` IPv6 host-only netloc check.
  Proves IPv6 without port still works.

- `B/tests/test_furl.py:1722` netloc `None` clearing check.
  Proves host/user/password clear behavior.

- `B/tests/test_furl.py:1749` and `B/tests/test_furl.py:1755` IPv6 origin success checks.
  Proves consistent parsing for `origin.setter` with and without explicit port.

- `B/tests/test_furl.py:1772` and `B/tests/test_furl.py:1779` rollback checks.
  Proves invalid-port style failures do not leave partial state.

Missing tests in B (prompt-3 asks for these):

- No dedicated unit tests for `_parse_host_port()` itself.
- No explicit invalid-host + valid-port direct-setter tests, even though runtime behavior is now correct.

### Pros

- Correctly fixes partial-state update risk in direct `origin.setter` and `netloc.setter`.
- Validation logic is modular and reused (`_parse_host_port`, `_validate_port`, `_validate_and_normalize_host`).
- Design is cleaner for future maintenance than Model A in this round.

### Cons

- Prompt-3 requested direct tests for parse helper were not added.
- Tests still miss the exact invalid-host + valid-port direct setter case.
- `_clear_port()` exists but is not used by current setter flow in `B/furl/furl.py`, so there is small dead-code risk.

### PR readiness

- Almost ready, but not fully ready for strict acceptance of prompt-3 requirements.
- Code behavior is improved, but required direct helper tests and specific edge-case tests are still missing.

### Concrete next-turn fixes

1. Add unit tests for `_parse_host_port()` directly (valid and malformed inputs).
2. Add explicit invalid-host + valid-port tests for direct `origin.setter` and `netloc.setter`.
3. Either use `_clear_port()` in setter flow or remove it to avoid dead internal API.

## Comparison Table

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Better logic and correctness         | `-4 (Model B)`      | `B/origin.setter` and `B/netloc.setter` validate first, then assign (`B/furl/furl.py:1614` and `B/furl/furl.py:1572`). In A, invalid-host + valid-port still partially updates `port` (`A/furl/furl.py:1559`, `A/furl/furl.py:1596`). |
| Better Naming and Clarity            | `-2 (Model B)`      | B adds clear validator names (`_validate_port`, `_validate_and_normalize_host`) and keeps responsibilities explicit. A is simpler but less explicit about validation separation. |
| Better Organization and Clarity      | `-3 (Model B)`      | B modularizes parsing + validation in reusable helpers and uses same flow in both setters. A reuses parser only, but setter apply order is still fragile. |
| Better Interface Design              | `-1 (Model B)`      | Both keep external parameter interface stable. B internal design better supports safe behavior without changing public call style. |
| Better error handling and robustness | `-4 (Model B)`      | Runtime check shows B keeps full state unchanged on invalid-host + valid-port errors, while A changes `port` and leaves partial state. |
| Better comments and documentation    | `0 (Tie)`           | Both have useful comments. Both still miss explicit test comments for `_parse_host_port()` direct behavior. |
| Ready for review / merge             | `-2 (Model B)`      | A is not ready (correctness bug still present). B is close, but still missing prompt-3-required tests (`_parse_host_port()` and explicit invalid-host+valid-port direct-setter tests). |
| Overall Better Solution              | `-3 (Model B)`      | B addresses the core correctness issue from prompt 3 (atomic-like setter behavior). A still fails the main risk case. B still needs one more test-focused turn for full completion. |

## Final justification

Model B is better for this turn.

Why: in `B/origin.setter` and `B/netloc.setter`, validation happens before assignment, so failure does not leave partial state. In Model A, direct setter path can still change `port` when `host` fails, which is exactly the issue prompt 3 asked to fix. Both models still need better tests for `_parse_host_port()` and explicit invalid-host + valid-port direct setter coverage.

## Verification I ran

- `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py` in `A/` -> `73 passed`.
- `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py` in `B/` -> `73 passed`.
- Runtime probe for direct setter invalid-host + valid-port:
- In A: state changed partially (`:88` -> `:55`) after exception.
- In B: state remained unchanged after exception.
