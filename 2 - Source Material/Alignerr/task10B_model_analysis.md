# Model B: Code Context Analysis

## Overview
Model B implements code context functionality through a single configuration option:
- `showContext` (boolean flag)

This approach provides a simpler, more unified interface where displaying source code context is treated as a single feature. The implementation uses a class attribute `contextLines` to control granularity, making it clean and cohesive.

---

## What Was Done

### 1. Core Implementation
**Added to `IceCreamDebugger.__init__`:**
```python
showContext: bool = False
```

**New class attribute:**
- `contextLines = 3` (class-level constant, controls number of lines shown)

### 2. New Methods

**`_formatSnippet(self, callFrame: FrameType) -> str`**
- Uses `inspect.getframeinfo(callFrame, context=2*n+1)` to retrieve source context
- Leverages built-in context parameter of `getframeinfo()` instead of manually calling `linecache`
- Returns formatted lines with markers and line numbers
- Format: `'{indent}{marker} {lineNo:4d} | {srcLine}'`
- Returns empty string if code_context is unavailable (REPL, frozen apps, etc.)

### 3. Integration Points

**Modified `_format()` method:**
```python
if self.showContext:
    out += self._formatSnippet(callFrame)
```
- Appends snippet to output if feature is enabled
- Uses `+=` concatenation (more Pythonic than building strings)

**Modified `configureOutput()`:**
- Added parameter: `showContext`
- Removed separate `numCodeContextLines` parameter
- Uses `Sentinel.absent` pattern for optional parameters

### 4. New Imports
- `linecache` is NOT imported
  - Uses `inspect.getframeinfo(context=2*n+1)` instead
  - Simpler, more integrated approach

### 5. Test Coverage
Added comprehensive test suite in `test_icecream.py`:

| Test | Purpose |
|------|---------|
| `test_show_context_disabled_by_default` | Verifies feature is off by default (no snippet lines) |
| `test_show_context_enabled_via_configure` | Tests enabling via configureOutput() |
| `test_show_context_enabled_via_constructor` | Tests enabling via __init__ parameter |
| `test_show_context_custom_context_lines` | Validates `contextLines` attribute behavior |
| `test_show_context_unmarked_lines_have_spaces` | Verifies marker alignment (>>> vs spaces) |
| `test_show_context_line_numbers_are_sequential` | Validates line number accuracy |
| `test_show_context_no_args` | Tests with no-argument ic() calls |

**Helper method:** `_snippet_lines(output: str)` - extracts snippet lines (those with ` | `) from output

---

## How It Works

### Execution Flow
```
ic(variable)
    ↓
_format() called
    ↓
showContext == True?
    ├─ Yes → _formatSnippet() → append snippet
    └─ No  → skip snippet
    ↓
Output format:
ic| variable: value
    >>> 42 | ic(variable)
        41 | prev_line
        43 | next_line
```

### Key Algorithm Details

**Frame Info Retrieval:**
```python
frameInfo = inspect.getframeinfo(callFrame, context=2*n+1)
# Returns frame info WITH code_context list of source lines
```

**Line Extraction:**
```python
icLine = frameInfo.lineno  # 1-based line number of ic() call
startLine = icLine - n      # Calculate starting line
# Iterate through frameInfo.code_context which is pre-fetched by getframeinfo()
for i, srcLine in enumerate(frameInfo.code_context):
    lineNo = startLine + i
```

**Output Formatting:**
- Uses same indentation approach as Model A
- Marker column calculation for alignment verification in tests
- Pythonic iteration over pre-fetched code_context

---

## What's Good About This Approach

### ✅ Strengths

1. **Simpler API**
   - Single boolean parameter: `showContext`
   - Users don't need to know about two separate controls
   - Easier cognitive load: "show context" or "don't show context"

2. **Unified Configuration**
   - `showContext=True` is clear and self-documenting
   - No confusion between on/off and granularity controls
   - More intuitive naming

3. **Class Attribute for Granularity**
   - `contextLines` as class attribute allows per-instance customization
   - Users who want flexibility: `ic.contextLines = 5`
   - Simpler than method parameters

4. **Built-in Frame Info**
   - Uses `inspect.getframeinfo(context=...)` parameter
   - No need to import additional modules (no `linecache`)
   - Fewer dependencies, cleaner imports
   - Inspect module already imported for other functionality

5. **Cleaner Code**
   - Fewer parameters in constructor
   - Fewer parameters in configureOutput()
   - Simpler logic flow

6. **Excellent Test Coverage**
   - Tests verify default behavior
   - Tests validate contextLines customization
   - Tests check marker alignment and line number sequencing
   - Helper method `_snippet_lines()` makes test logic reusable

7. **Edge Case Handling**
   - Early return for missing code_context: `if not frameInfo.code_context: return ''`
   - Graceful degradation (no snippet if unavailable)

8. **Configuration Flexibility**
   - Can set via constructor: `IceCreamDebugger(showContext=True)`
   - Can set via configureOutput(): `ic.configureOutput(showContext=True)`
   - Can customize granularity: `ic.contextLines = 1` (runtime adjustment)

---

## What's Bad About This Approach

### ❌ Weaknesses

1. **Less Flexible Disable Logic**
   - To turn off: must call `configureOutput(showContext=False)`
   - Cannot simply set numCodeContextLines to 0
   - Missing ability to disable via parameter if toggling is needed

2. **Class Attribute Instead of Instance Attribute**
   - `contextLines = 3` defined at class level
   - If modified on instance, affects only that instance (fine)
   - If modified on class, affects ALL instances (potential bug)
   - Example bug:
     ```python
     ic.contextLines = 1  # Oops, affects default ic instance globally
     dbg = IceCreamDebugger()
     dbg(x)  # Will use contextLines=1 (unexpected!)
     ```

3. **No Parameter Validation**
   - `contextLines` can be set to any value (negative, very large)
   - No validation in configureOutput() for showContext parameter
   - Could lead to confusing behavior or performance issues

4. **Integration with Existing Features**
   - Doesn't interact well with `includeContext` feature
   - Output format doesn't distinguish between:
     - Context from includeContext (file.py:42 in func())
     - Context from showContext (code snippet)
   - They appear as separate blocks rather than integrated

5. **Missing Constructor Parameter Validation**
   - `showContext: bool = False` accepts any type (not enforced at runtime)
   - Type hints are present but not checked

6. **Frame Info Behavior Not Documented**
   - `inspect.getframeinfo(context=2*n+1)` behavior with context parameter is less obvious
   - Different from explicit line-by-line retrieval (harder to debug if issues arise)

7. **Naming Collision Risk**
   - `contextLines` is similar to but distinct from `numCodeContextLines`
   - Could confuse users familiar with Model A
   - Not immediately obvious what "contextLines" means (is it context lines, or lines of context?)

---

## Important Design Decisions & Rationale

| Decision | Reasoning |
|----------|-----------|
| Single `showContext` boolean | Simpler API, clearer intent |
| Class attribute `contextLines` | Clean way to control granularity without extra parameters |
| Use `inspect.getframeinfo(context=...)` | Built-in functionality, fewer dependencies, already available |
| Append to output with `+=` | Pythonic, concise |
| Early return for missing context | Graceful degradation |
| Default OFF | Non-intrusive; existing users unaffected |

---

## New Functions and Imports Summary

| Item | Type | Purpose |
|------|------|---------|
| `_formatSnippet()` | Method | Formats source code snippet around ic() call |
| `showContext` | Attribute | Boolean toggle for feature |
| `contextLines` | Class Attribute | Number of lines above/below call to show (default 3) |

**Key Difference from Model A:**
- No `linecache` import
- No additional constant definitions for defaults
- Fewer total additions to the codebase

---

## Code Quality Assessment

### Comments
- ✅ Good: Variable names in `_formatSnippet()` are clear (`icLine`, `startLine`, `marker`)
- ❌ Missing: `_formatSnippet()` method lacks docstring explaining parameters and behavior
- ⚠️ Moderate: Test names are descriptive but brief docstrings would help

### Error Handling
- ✅ Good: Graceful handling of missing code_context
- ✅ Good: Early return for unavailable source
- ❌ Missing: No validation that `contextLines` is positive integer
- ❌ Missing: No bounds checking on `contextLines` (could be 10000)

### Performance
- ✅ Good: Uses inspect module's built-in context retrieval (optimized)
- ✅ Good: Conditional feature (off by default)
- ✅ Good: No external dependencies
- ⚠️ Concern: `inspect.getframeinfo(context=2*n+1)` fetches all context even if `showContext=False` (minor inefficiency)

### PR Readiness
- ✅ Good: Comprehensive test suite with edge cases
- ✅ Good: Backward compatible
- ✅ Good: Cleaner code surface
- ❌ Needs: Documentation (README examples, API docs)
- ❌ Needs: Docstring for `_formatSnippet()`
- ❌ Needs: Input validation for `contextLines`

---

## Potential Issues

### Instance vs Class Attributes
The biggest risk is the class-level `contextLines = 3`:
- If user modifies it globally, ALL instances are affected
- Expected pattern: `ic.contextLines = 1` might be to modify the global `ic` instance only
- But could be mistaken for a global class setting

**Example problematic scenario:**
```python
ic.contextLines = 1  # User thinks they're modifying 'ic' instance
db = IceCreamDebugger()  # New instance
db(x)  # Unexpectedly uses contextLines=1 because it reads from class
```

**Solution would be:** Initialize as instance attribute in `__init__`:
```python
def __init__(self, ..., showContext: bool = False):
    self.contextLines = 3  # Instance attribute, not class
```
But this wasn't done in the implementation.
