# Model A: Code Context Analysis

## Overview
Model A implements code context functionality through two distinct configuration options:
- `includeCodeContext` (boolean flag)
- `numCodeContextLines` (integer to control the number of lines shown)

This approach provides granular control over when and how much source code context is displayed.

---

## What Was Done

### 1. Core Implementation
**Added to `IceCreamDebugger.__init__`:**
```python
includeCodeContext: bool = DEFAULT_INCLUDE_CODE_CONTEXT
numCodeContextLines: int = DEFAULT_NUM_CODE_CONTEXT_LINES
```

**New class attributes (defaults):**
- `DEFAULT_INCLUDE_CODE_CONTEXT = False`
- `DEFAULT_NUM_CODE_CONTEXT_LINES = 3`

### 2. New Methods

**`_formatCodeContext(self, callFrame: FrameType) -> str`**
- Uses `inspect.getframeinfo(callFrame)` to get the call site information
- Iterates through `range(callLineNo - n, callLineNo + n + 1)` to capture lines
- Uses `linecache.getline()` for safe file line retrieval
- Marks the actual call line with `'>>>'` marker
- Other context lines marked with `'   '` (3 spaces)
- Format: `'    {marker} {lineNo:4d} | {line}'`

### 3. Integration Points

**Modified `_format()` method:**
```python
if self.includeCodeContext:
    codeContext = self._formatCodeContext(callFrame)
    if codeContext:
        out = out + '\n' + codeContext
```
- Appends code context to existing output after all other formatting

**Modified `configureOutput()`:**
- Added parameters: `includeCodeContext`, `numCodeContextLines`
- Uses `Sentinel.absent` pattern for optional parameters

### 4. New Imports
- `linecache` module added (line 15)
  - Used to safely retrieve source lines by filename and line number
  - Caches file contents to avoid repeated disk reads

### 5. Test Coverage
Added comprehensive test suite in `test_icecream.py`:

| Test | Purpose |
|------|---------|
| `test_code_context_disabled_by_default` | Verifies feature is off by default (no `>>>` in output) |
| `test_code_context_with_args` | Tests context display when ic() has arguments |
| `test_code_context_without_args` | Tests context display with no-arg ic() calls |
| `test_code_context_num_lines` | Validates `numCodeContextLines` parameter limits output |
| `test_code_context_line_numbers` | Ensures line numbers are correct and sequential |
| `test_code_context_configure_output` | Tests toggling via configureOutput() |

---

## How It Works

### Execution Flow
```
ic(variable)
    ↓
_format() called
    ↓
includeCodeContext == True?
    ├─ Yes → _formatCodeContext() → append context snippet
    └─ No  → skip context
    ↓
Output format:
ic| variable: value
    >>> 42 | ic(variable)
        41 | prev_line
        43 | next_line
```

### Key Algorithm Details

**Line Retrieval:**
```python
for lineNo in range(callLineNo - n, callLineNo + n + 1):
    if lineNo < 1:
        continue  # Skip invalid line numbers
    line = linecache.getline(filename, lineNo)
    if not line:
        continue  # Skip empty/unavailable lines
```

**Output Formatting:**
- Indentation: 4 spaces base + marker (>>> or 3 spaces) + line number (right-aligned 4 chars) + `|` delimiter
- String lines are `.rstrip()` to remove trailing whitespace

---

## What's Good About This Approach

### ✅ Strengths

1. **Fine-Grained Control**
   - Two independent parameters: toggle feature ON/OFF AND control snippet size
   - Users can adjust context size without disabling the feature entirely

2. **Backward Compatible**
   - Defaults to OFF (`DEFAULT_INCLUDE_CODE_CONTEXT = False`)
   - No breaking changes to existing API

3. **Flexible Configuration**
   - Can be set in constructor: `IceCreamDebugger(includeCodeContext=True, numCodeContextLines=5)`
   - Can be toggled via `configureOutput()`

4. **Efficient Line Retrieval**
   - Uses `linecache` module which:
     - Caches file contents (doesn't re-read same file repeatedly)
     - Handles missing/invalid files gracefully
     - Standard library solution (no external dependencies)

5. **Clear Visual Markers**
   - `>>>` clearly marks the actual call line
   - Consistent indentation makes output readable

6. **Comprehensive Testing**
   - Tests cover default behavior, parameter configuration, edge cases
   - Validates line number accuracy
   - Tests both with and without arguments

---

## What's Bad About This Approach

### ❌ Weaknesses

1. **Parameter Explosion**
   - Two parameters for what could be one: `contextSize` (0 = off, >0 = lines to show)
   - More surface area in configureOutput() method signature
   - Slightly harder to understand: users must know BOTH parameters exist

2. **Awkward Semantics**
   - `includeCodeContext=True, numCodeContextLines=0` is confusing
   - Why allow both parameters if one turns it on and the other controls it?
   - Not intuitive for new users

3. **Integration Decision**
   - Context appended AFTER main output as separate lines
   - Could be confusing in logs—context appears divorced from the actual variable output
   - Example:
     ```
     ic| x: 5
         >>> 42 | ic(x)
     ```
     The separation makes it harder to visually connect them

4. **No Contextual Integration**
   - If `includeContext=True` (file/line info), and `includeCodeContext=True`, they're separate:
     ```
     ic| file.py:42 in func()
         >>> 42 | ic(x)
     ```
   - Could be more cohesively formatted

5. **Fixed Formatting**
   - Line format is hardcoded: `'    >>> {lineNo:4d} | {line}'`
   - No way to customize prefix, alignment, delimiter
   - Output might not fit user's preferred style

6. **Naming Ambiguity**
   - `includeCodeContext` vs `includeContext` - easy to confuse
   - `numCodeContextLines` is long; could be `contextLineCount` or similar

7. **Edge Case Handling**
   - If `numCodeContextLines=0`, still shows lines? Behavior unclear from parameter name
   - What if file is deleted after program starts? `linecache.getline()` returns empty string, lines silently skipped

---

## Important Design Decisions & Rationale

| Decision | Reasoning |
|----------|-----------|
| Use `linecache` | Standard library, caches results, handles missing files |
| Append after output | Simple, doesn't complicate main output formatting logic |
| Default OFF | Non-intrusive; doesn't surprise existing users |
| Range: `callLineNo - n` to `callLineNo + n + 1` | Shows n lines above and below the call (symmetric context) |
| Use `>>>` marker | Clear visual distinction, easy to spot in output |
| Separate boolean + int parameters | Allows independent control (on/off vs size) |

---

## New Functions and Imports Summary

| Item | Type | Purpose |
|------|------|---------|
| `linecache` | Import | Standard library for safe source code line retrieval |
| `_formatCodeContext()` | Method | Retrieves and formats code snippet around ic() call |
| `includeCodeContext` | Attribute | Boolean toggle for feature |
| `numCodeContextLines` | Attribute | Number of context lines (above and below call) |
| `DEFAULT_INCLUDE_CODE_CONTEXT` | Constant | Default value (False) |
| `DEFAULT_NUM_CODE_CONTEXT_LINES` | Constant | Default lines count (3) |

---

## Code Quality Assessment

### Comments
- ✅ Good: Constants have docstrings explaining their purpose
- ❌ Missing: `_formatCodeContext()` method lacks docstring
- ⚠️ Moderate: Test names are descriptive but could use docstrings

### Error Handling
- ✅ Good: Gracefully skips invalid line numbers (`if lineNo < 1`)
- ✅ Good: Handles missing files (`if not line: continue`)
- ⚠️ Moderate: No explicit error handling for file permission issues (linecache handles silently)

### Performance
- ✅ Good: Uses linecache for efficient caching
- ✅ Good: Conditional feature (off by default)
- ⚠️ Concern: No limit on `numCodeContextLines` - user could set to 1000+ (though unlikely in practice)

### PR Readiness
- ✅ Good: Tests are comprehensive
- ✅ Good: Backward compatible
- ❌ Needs: Documentation updates (README examples, API docs)
- ⚠️ Needs: Docstring for `_formatCodeContext()`
