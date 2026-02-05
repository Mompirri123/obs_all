# Analysis of Model A vs Model B Implementation
## IceCream Type Info and Dataclass Expansion Feature

### Objective
Implement an optional mode to `ic()` that:
1. Includes runtime type information in the output
2. Expands dataclass fields up to a user-controlled depth

---

## MODEL A: SUCCESSFUL IMPLEMENTATION

### Imports Added
- **`dataclasses`** - To work with dataclass inspection
- **`Literal`** - For type annotations in `configureOutput()` method

### New Functions and Helper Code

#### 1. `_format_type(obj: object) -> str` (Line 268-275)
Returns a easy to read type name for any object
- Gets the type of the object
- Checks if it's a builtin type (like int, str, list)
- Returns just the class name for builtins (e.g., "int")
- Returns module-qualified name for custom types (e.g., "mymodule.MyClass")

**Example:**
```python
_format_type(42)        # Returns: "int"
_format_type("hello")   # Returns: "str"
_format_type(MyClass()) # Returns: "__main__.MyClass"
```

#### 2. `_expand_dataclass(obj: object, depth: int, argToStringFunction: Callable) -> str` (Line 278-301)
**What it does:** Pretty-prints a dataclass instance while expanding its fields to a specified depth.

**How it works:**
- Takes a dataclass instance, a depth level, and the normal string function
- If depth < 1, just uses the normal string function (stops expanding)
- Gets all fields of the dataclass using `dataclasses.fields()`
- For each field:
  - If the field value is itself a dataclass AND depth > 1, recursively expands it
  - Otherwise, converts it to a string normally
- Builds output in format: `ClassName(field1=value1, field2=value2)`

**Example:**
```python
@dataclass
class Point:
    x: int
    y: int

@dataclass  
class Box:
    corner: Point
    color: str

# With depth=1:
_expand_dataclass(Box(Point(1,2), "red"), 1, func)
# Returns: "Box(corner=Point(x=1, y=2), color='red')"
# Note: Point is expanded because it's a field value at depth 1

# With depth=2:
_expand_dataclass(Box(Point(1,2), "red"), 2, func)
# Returns: "Box(corner=Point(x=1, y=2), color='red')"
# Note: At depth=2, the inner Point ALSO expands
```

### Changes to Existing Classes/Methods

#### 1. IceCreamDebugger `__init__()` method
Added two new parameters:
- **`includeTypeInfo: bool = False`** - Enable/disable type annotations
- **`dataclassExpansionDepth: int = 0`** - How many levels of dataclass nesting to expand

Added two new instance variables:
- `self.includeTypeInfo`
- `self.dataclassExpansionDepth`

#### 2. IceCreamDebugger `_constructArgumentOutput()` method
Added logic to handle both new features:

**For dataclass expansion:**
```python
def _valToString(val: object) -> str:
    if (self.dataclassExpansionDepth > 0
            and dataclasses.is_dataclass(val)
            and not isinstance(val, type)):
        return _expand_dataclass(
            val, self.dataclassExpansionDepth,
            self.argToStringFunction)
    return self.argToStringFunction(val)
```

**For type info:**
```python
if self.includeTypeInfo:
    pairs = [
        (arg, '%s <type: %s>' % (valStr, _format_type(val)), val)
        for arg, valStr, val in pairs]
```

#### 3. IceCreamDebugger `configureOutput()` method
Added two new optional parameters to allow runtime configuration:
- **`includeTypeInfo: Union[bool, Literal[Sentinel.absent]] = Sentinel.absent`**
- **`dataclassExpansionDepth: Union[int, Literal[Sentinel.absent]] = Sentinel.absent`**

Added handling to update the instance variables if these parameters are provided.

### Test Coverage

**Type Info Tests:**
- `test_include_type_info_int()` - Verifies type annotation for integers
- `test_include_type_info_string()` - Verifies type annotation for strings  
- `test_include_type_info_list()` - Verifies type annotation for lists
- `test_include_type_info_custom_class()` - Tests module-qualified names
- `test_include_type_info_multiple_args()` - Tests multiple arguments get annotations
- `test_include_type_info_disabled_by_default()` - Verifies default is off

**Dataclass Expansion Tests:**
- `test_dataclass_expansion_depth_1()` - Simple dataclass expansion
- `test_dataclass_expansion_nested_depth_1()` - Stops at depth=1 for nested classes
- `test_dataclass_expansion_nested_depth_2()` - Expands nested classes at depth=2
- `test_dataclass_expansion_disabled_by_default()` - Verifies default is off
- `test_dataclass_expansion_with_type_info()` - Tests both features together

### How to Use (Model A)

```python
from icecream import ic

# Enable type information
ic.configureOutput(includeTypeInfo=True)
x = 42
ic(x)  # Output: ic| x: 42 <type: int>

# Enable dataclass expansion
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

ic.configureOutput(dataclassExpansionDepth=1)
p = Point(1, 2)
ic(p)  # Output: ic| p: Point(x=1, y=2)

# Use both together
ic.configureOutput(includeTypeInfo=True, dataclassExpansionDepth=1)
ic(p)  # Output: ic| p: Point(x=1, y=2) <type: __main__.Point>
```

---

## MODEL B: INCOMPLETE IMPLEMENTATION ✗

### What Was Done
Model B added the same basic imports (`dataclasses`, `Literal`) to the file but **did not complete the implementation**.

### What Was NOT Done

#### Missing New Functions
- `_format_type()` function - NOT implemented
- `_expand_dataclass()` function - NOT implemented

#### Missing Class Parameter Changes
The `IceCreamDebugger.__init__()` method was NOT modified to include:
- `includeTypeInfo` parameter
- `dataclassExpansionDepth` parameter

#### Missing Instance Variables
The `__init__()` method doesn't store:
- `self.includeTypeInfo`
- `self.dataclassExpansionDepth`

#### Missing Implementation Logic
The `_constructArgumentOutput()` method was NOT modified to:
- Handle dataclass expansion
- Add type information annotations

#### Missing Configuration Support
The `configureOutput()` method was NOT updated to accept:
- `includeTypeInfo` parameter
- `dataclassExpansionDepth` parameter

#### Missing Tests
- **Zero tests** written for the new features
- The test file remains at 700 lines (original size)
- Model A has 954 lines (includes comprehensive test coverage)

### How Much Was Completed
| Feature | Model A | Model B |
|---------|---------|---------|
| Helper functions | ✓ 100% | ✗ 0% |
| Type info feature | ✓ 100% | ✗ 0% |
| Dataclass expansion | ✓ 100% | ✗ 0% |
| Configuration support | ✓ 100% | ✗ 0% |
| Test coverage | ✓ 100% | ✗ 0% |
| **Overall** | **✓ Complete** | **✗ Incomplete** |

### File Comparison
- **Model A:** 534 lines in icecream.py, 954 lines in test file
- **Model B:** 470 lines in icecream.py, 700 lines in test file
- **Difference:** Model A has 64 additional lines of core functionality + 254 lines of tests

---

## Summary

### Model A
**FUNCTIONAL IMPLEMENTATION**
- Implemented both type info and dataclass expansion features
- Added helper functions with clear purpose
- Modified core methods to support both features
- Added configuration methods for runtime control
- Comprehensive test suite with 10+ test cases
- Ready to use immediately

### Model B  
 **INCOMPLETE - ONLY IMPORTS ADDED**
- Added necessary imports but no actual implementation
- Missing all core functionality

- Has a missing parameter handling
- Low to zero test coverage
- **Would not work if called**, cause the feature parameters don't exist

### Key Differences in Approach
The two models took fundamentally different paths:

1. **Model A:** Systematic, complete approach
   - Added all required helper functions first
   - Then modified class constructor properly
   - updated formatting logic to use newly implemented features
   - Added the configuration hook(s)
   - Tests for generated functions

2. **Model B:** Incomplete approach
   - Added imports but never actually uses them
   - Did not follow through with the implementation (COT only)
   - Features declared in objective but where never built
   - No integration into existing code
   - Abandoned before testing phase

### Which Works?
- **Model A:** Yes, fully functional
- **Model B:** No, non-functional
