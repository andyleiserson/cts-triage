Perfect! Now I have all the information needed for a comprehensive triage report.

## CTS Triage Complete

I've completed the triage for **webgpu:shader,validation,expression,call,builtin,frexp:*** and identified the root cause of all failures.

---

### 1. Specific Root Cause

**Naga does not support constant or override evaluation of the `frexp()` builtin function.**

The `frexp()` function is explicitly listed as unimplemented in Naga's constant evaluator at `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` (lines 1847-1849):

```rust
// unimplemented
crate::MathFunction::Modf
| crate::MathFunction::Frexp
| crate::MathFunction::Ldexp
...
=> Err(ConstantEvaluatorError::NotImplemented(
    format!("{fun:?} built-in function"),
)),
```

When the CTS tests try to evaluate `frexp()` at constant or override time, Naga returns the error: "Not implemented as constant expression: Frexp built-in function".

---

### 2. Suggested Bug Reference for fail.lst

The current entry in `/Users/Andy/Development/wgpu2/cts_runner/fail.lst` (line 121) is:
```
webgpu:shader,validation,expression,call,builtin,frexp:* // 39%
```

**Suggested update:**
```
webgpu:shader,validation,expression,call,builtin,frexp:* // Naga: frexp() not implemented for constant/override evaluation
```

---

### 3. Summary for triage.md

```markdown
# frexp CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,expression,call,builtin,frexp:*`

Model: Claude Sonnet 4.5

**Overall Status:** 16P/25F/0S (39%/61%/0%)

## Passing Sub-suites ✅
- `frexp:must_use:*` - All tests pass (2/2)
- `frexp:args:*` - Most argument validation tests pass (14/15)

## Remaining Issues ⚠️
- `frexp:values:*` - All constant/override evaluation tests fail (0/24 passing)
- `frexp:args:arg="good"` - Single test failure (1/15 failing)

## Issue Detail

### 1. Constant and Override Evaluation Not Implemented

**Test selectors:** 
- `webgpu:shader,validation,expression,call,builtin,frexp:values:stage="constant";*`
- `webgpu:shader,validation,expression,call,builtin,frexp:values:stage="override";*`
- `webgpu:shader,validation,expression,call,builtin,frexp:args:arg="good"`

**What it tests:** 
These tests verify that `frexp()` can be evaluated at shader creation time (constant stage) and pipeline creation time (override stage) with various floating-point types including abstract-int, abstract-float, f32, f16, and vectors thereof.

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,frexp:values:stage="constant";type="f32"
```

**Error:**
```
Shader '' parsing error: Not implemented as constant expression: Frexp built-in function
  ┌─ wgsl:2:12
  │
2 │ const v  = frexp(3.4028234663852886e+38f);
  │            ^^^^^ see msg
```

For override stage:
```
Pipeline constant error: MSL: ConstantEvaluatorError(NotImplemented("Frexp built-in function"))
```

**Root cause:**
Naga's constant evaluator (`/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs`, lines 1847-1879) explicitly lists `frexp()` as unimplemented. The function is in a group with other unimplemented math functions like `modf`, `ldexp`, `mix`, `smoothStep`, etc.

The `frexp()` builtin splits a floating-point value into a significand (fractional part) and an exponent, returning a struct with two members: `fract` (float/vec) and `exp` (int/vec). Implementing constant evaluation would require:
1. Splitting float values into mantissa and exponent components
2. Constructing the predeclared `FrexpResult` struct type at compile time
3. Handling all scalar and vector variants (f32, f16, abstract-float, and their vec2/vec3/vec4 forms)

**Fix needed:**
Implement constant evaluation support for `frexp()` in Naga's `constant_evaluator.rs`. This would involve:
1. Removing `MathFunction::Frexp` from the unimplemented list
2. Adding a match arm that:
   - Extracts the mantissa and exponent from the input float(s)
   - Creates a struct value with `fract` and `exp` members
   - Handles scalar and vector cases appropriately
3. Similar to how other math functions are implemented (see examples in the same file)

Note: The `frexp()` function itself works correctly at runtime - this is purely a constant evaluation limitation that affects tests checking compile-time and pipeline-creation-time evaluation.
```

---

### Test Breakdown

**Test Categories:**
- **must_use** (2 tests): 2P/0F - ✅ All pass
- **args** (15 tests): 14P/1F - ⚠️ arg="good" fails due to constant evaluation
- **values** (24 tests): 0P/24F - ❌ All fail
  - constant stage (16 tests): all types (abstract-int, abstract-float, f32, f16, vectors)
  - override stage (8 tests): concrete types only (f32, f16, vectors)
