# Review Update (JR Engineer Style): Model A vs Model B (Prompt 1 + Prompt 2 + Prompt 3)

Prompts reviewed:

- Prompt 1: fix inconsistent parsing between `origin` and `netloc`.
- Prompt 2: avoid risky direct `_port = None` style usage, modularize shared logic, add tests.
- Prompt 3: make `origin.setter` and `netloc.setter` safe from partial private-state updates, add tests for `parse_host_port()`, and test invalid-host + valid-port paths.

Note:

- Backward compatibility was not required by prompt, so I gave credit for good implementation choices even if API shape changed internally.

## Model A

### Full flow (how it works)

1. Entry from constructor:
- `furl.__init__()` in `A/furl/furl.py:1367` calls `self.load(url)` and then `self.set(...)` in `A/furl/furl.py:1382`.
- `self.set(...)` in `A/furl/furl.py:1664` applies `origin` via `self.origin = origin` and `netloc` via `self.netloc = netloc`.

2. Entry from direct property assignment:
- `f.origin = ...` goes to `origin.setter` in `A/furl/furl.py:1575`.
- `f.netloc = ...` goes to `netloc.setter` in `A/furl/furl.py:1533`.

3. Shared parse logic:
- Both setters call `_parse_host_port(host_port, context_name, context_value)` in `A/furl/furl.py:1479`.
- This function splits `host` and `port` and handles IPv6 bracket format.

4. `netloc.setter` apply order:
- In `A/furl/furl.py:1559`, code sets `self.port = port` first.
- Then in `A/furl/furl.py:1560`, code sets `self.host = host`.

5. `origin.setter` apply order:
- In `A/furl/furl.py:1596`, code sets `self.port` (or `_clear_port()`) first.
- Then in `A/furl/furl.py:1603`, code sets `self.host`, and after that `self.scheme`.

6. Current issue still present:
- If `port` is valid but `host` is invalid, direct setter can fail after `port` already changed.
- So partial update can still happen in direct `origin.setter` and direct `netloc.setter`.

### New or modified functions/parameters (2 lines each)

- `_parse_host_port(host_port, context_name, context_value)` in `A/furl/furl.py:1479` (new).
  Shared parser for both setters. Good step to keep parsing behavior same between `origin` and `netloc`.

- `_clear_port(self)` in `A/furl/furl.py:1505` (new).
  Clears explicit `_port` when no port is provided in `origin.setter`. Helps keep default-port behavior cleaner.

- `netloc.setter` in `A/furl/furl.py:1533` (modified).
  Now uses shared parse helper. But still applies `port` before `host`, so direct partial update risk remains.

- `origin.setter` in `A/furl/furl.py:1575` (modified).
  Uses shared parse helper and `_clear_port()` path. But still applies `port` before `host`, so same risk remains.

- Public parameters: unchanged.
  Inputs like `origin=` and `netloc=` are same at API level, but behavior changed internally.

### Tests added and what they prove

Added in `A/tests/test_furl.py` (inside existing `test_netloc` and `test_origin`):

- `A/tests/test_furl.py:1710` host-only `netloc`.
  Proves default port is used when explicit port is removed.

- `A/tests/test_furl.py:1717` IPv6 host-only `netloc`.
  Proves IPv6 host without explicit port works.

- `A/tests/test_furl.py:1722` `netloc=None`.
  Proves host/username/password are cleared.

- `A/tests/test_furl.py:1749` and `A/tests/test_furl.py:1755` IPv6 `origin` success.
  Proves `[::1]:8080` and `[::1]` are parsed correctly in `origin.setter`.

- `A/tests/test_furl.py:1772` URL unchanged on invalid origin port.
  Proves rollback for invalid-port path.

- `A/tests/test_furl.py:1779` to `A/tests/test_furl.py:1818` extra origin edge checks.
  Proves some error paths and `origin=None` behavior.

Missing tests for prompt 3:

- No dedicated unit tests for `_parse_host_port()` itself.
- No explicit direct-setter test for invalid-host + valid-port (the partial-update case).

### Pros

- Good shared parsing via `_parse_host_port()`.
- Good coverage additions for IPv6 and default-port behavior.
- Good readability around `origin.setter`/`netloc.setter` changes.

### Cons

- Main prompt-3 issue is not fully fixed: direct `origin.setter` and `netloc.setter` can still partially update `port`.
- Missing direct unit tests for `_parse_host_port()`.
- Missing explicit invalid-host + valid-port test in direct setter path.

### PR readiness (JR style)

- Not ready yet.
- Needs one more fix turn for direct setter atomic behavior.

### Concrete next-turn fixes

1. In `origin.setter` and `netloc.setter`, validate host and port first, then assign once.
2. Add direct tests for invalid-host + valid-port in both direct setter paths.
3. Add dedicated tests for `_parse_host_port()`.

## Model B

### Full flow (how it works)

1. Entry from constructor:
- `furl.__init__()` in `B/furl/furl.py:1367` calls `self.load(url)` then `self.set(...)` in `B/furl/furl.py:1382`.
- `self.set(...)` in `B/furl/furl.py:1690` applies `origin` and `netloc` through property setters.

2. Entry from direct property assignment:
- `f.origin = ...` goes to `origin.setter` in `B/furl/furl.py:1596`.
- `f.netloc = ...` goes to `netloc.setter` in `B/furl/furl.py:1549`.

3. Shared parse and validation steps:
- `_parse_host_port(...)` in `B/furl/furl.py:1458` parses host+port.
- `_validate_port(...)` in `B/furl/furl.py:1495` validates port.
- `_validate_and_normalize_host(...)` in `B/furl/furl.py:1505` validates/normalizes host.

4. `netloc.setter` apply flow:
- Parse first, validate `port_val` and `host_val` first in `B/furl/furl.py:1574` and `B/furl/furl.py:1577`.
- Assign only after all validation in `B/furl/furl.py:1580` and `B/furl/furl.py:1581`.

5. `origin.setter` apply flow:
- Parse scheme + host_port, then parse host/port.
- Validate all first in `B/furl/furl.py:1614` to `B/furl/furl.py:1625`.
- Assign only after validation in `B/furl/furl.py:1628` to `B/furl/furl.py:1631`.

6. Prompt-3 core outcome:
- Direct setter partial update behavior is fixed for invalid-host + valid-port path.
- On failure, state stays unchanged.

### New or modified functions/parameters (2 lines each)

- `_parse_host_port(host_port, context_name, context_value)` in `B/furl/furl.py:1458` (new).
  Shared parse logic with context-aware error messages. Keeps `origin` and `netloc` parsing aligned.

- `_validate_port(self, port)` in `B/furl/furl.py:1495` (new).
  Single place to validate non-`None` ports. Returns normalized integer.

- `_validate_and_normalize_host(self, host)` in `B/furl/furl.py:1505` (new).
  Single place for host validation and normalization. Used by setter paths.

- `host.setter` in `B/furl/furl.py:1432` (modified).
  Now delegates to `_validate_and_normalize_host()`.

- `port.setter` in `B/furl/furl.py:1443` (modified).
  Uses `_validate_port()` for non-`None` values and keeps `None` handling.

- `origin.setter` and `netloc.setter` in `B/furl/furl.py:1596` and `B/furl/furl.py:1549` (modified).
  Validate-first, assign-after flow reduces partial-update risk.

- Public parameters: unchanged.
  Same public inputs (`origin`, `netloc`) with safer internal flow.

### Tests added and what they prove

Added in `B/tests/test_furl.py`:

- `B/tests/test_furl.py:1710` host-only `netloc` case.
  Proves port resets to scheme default.

- `B/tests/test_furl.py:1717` IPv6 host-only `netloc` case.
  Proves IPv6 path works without explicit port.

- `B/tests/test_furl.py:1722` `netloc=None` case.
  Proves host/user/password clear behavior.

- `B/tests/test_furl.py:1749` and `B/tests/test_furl.py:1755` IPv6 `origin` success cases.
  Proves consistent parse in `origin.setter`.

- `B/tests/test_furl.py:1772` and onward rollback checks.
  Proves invalid-port style failures do not corrupt URL state.

Missing tests for prompt 3:

- No dedicated unit tests for `_parse_host_port()` itself.
- No explicit direct invalid-host + valid-port test, even though runtime behavior is now correct.

### Pros

- Fixes the key prompt-3 bug in direct setters (validate first, assign after).
- Clear internal helper structure for parse, host validation, and port validation.
- Keeps public interface stable while improving internal safety.

### Cons

- Prompt-3 asked for parse helper tests, but direct `_parse_host_port()` tests are missing.
- Missing explicit test for invalid-host + valid-port direct setter case.
- `_clear_port()` exists but current `origin.setter` path does not use it, which can confuse future readers.

### PR readiness (JR style)

- Ready for merge with small follow-up tests.
- Core requested bug fix is done, but test completeness can be improved.

### Concrete next-turn fixes

1. Add direct unit tests for `_parse_host_port()`.
2. Add explicit direct-setter tests for invalid-host + valid-port in both `origin.setter` and `netloc.setter`.
3. Either use `_clear_port()` in current flow or remove it for clarity.

## Comparison Table

| **Question**                         | **Which is better** | **Reasoning / Justification** |
| ------------------------------------ | ------------------- | ----------------------------- |
| Better logic and correctness         | `-4 (Model B)`      | In `B/origin.setter` and `B/netloc.setter`, validation happens before assignment. In `A/origin.setter` and `A/netloc.setter`, invalid-host + valid-port still leaves partial `port` update. |
| Better Naming and Clarity            | `-2 (Model B)`      | B helper names like `_validate_port()` and `_validate_and_normalize_host()` are easier to understand in code review than A’s simpler split-only helper set. |
| Better Organization and Clarity      | `-3 (Model B)`      | B has cleaner modular path: parse helper + host validator + port validator reused in both setters. A reuses parse only, but setter update flow is still fragile. |
| Better Interface Design              | `-1 (Model B)`      | Both keep public parameter interface stable. B’s internal design supports safer behavior with minimal external change. |
| Better error handling and robustness | `-4 (Model B)`      | Runtime check shows B keeps state unchanged for invalid-host + valid-port in direct setter, while A changes `port` before failing host validation. |
| Better comments and documentation    | `0 (Tie)`           | Both added useful comments around IPv6 and port behavior. Both can still improve by documenting parse-helper tests directly. |
| Ready for review / merge             | `-3 (Model B)`      | A still has core correctness issue, so not ready. B fixed core issue and is merge-ready with small test follow-up. |
| Overall Better Solution              | `-3 (Model B)`      | B solves the main prompt-3 requirement (no partial state update in direct setters). A still misses this key behavior. |

## Final justification

Model B is better.

Reason: prompt 3 asks to stop partial private-state updates in direct setter failure paths. B’s `origin.setter` and `netloc.setter` now validate first and assign later, so this issue is fixed. A still changes `port` before host validation and can leave partial state.

## Validation run

- `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py` in `A/` -> `73 passed`.
- `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python3 -m pytest -q tests/test_furl.py` in `B/` -> `73 passed`.
- Direct runtime check:
- In A: invalid-host + valid-port direct setter changed `port` after exception.
- In B: invalid-host + valid-port direct setter kept full state unchanged.
