Good, so degrees is on the triage checklist. Now let me document my findings. Let me create a comprehensive summary of the root cause and suggested fixes.

## Summary

I've completed the CTS triage for `webgpu:shader,validation,expression,call,builtin,degrees:*`. Here are my findings:

### Overall Status
**12 failures out of 45 tests (26.67% failure rate)**

All failures are in the `values` subtest, which validates that constant and override evaluation of `degrees()` properly rejects values that would overflow.

### Root Cause

The root cause is in Naga's `check_literal_value` function located at `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs` (lines 1321-1341). This function validates that float literals are not NaN or infinity, but it has **incomplete coverage**:

**Current implementation:**
```rust
pub fn check_literal_value(literal: crate::Literal) -> Result<(), LiteralError> {
    let is_nan = match literal {
        crate::Literal::F64(v) => v.is_nan(),
        crate::Literal::F32(v) => v.is_nan(),
        _ => false,  // ❌ Missing AbstractFloat and F16
    };
    if is_nan {
        return Err(LiteralError::NaN);
    }

    let is_infinite = match literal {
        crate::Literal::F64(v) => v.is_infinite(),
        crate::Literal::F32(v) => v.is_infinite(),
        _ => false,  // ❌ Missing AbstractFloat and F16
    };
    if is_infinite {
        return Err(LiteralError::Infinity);
    }

    Ok(())
}
```

**Missing checks:**
1. `Literal::AbstractFloat` - not checked for NaN or infinity
2. `Literal::F16` - not checked for NaN or infinity

**Why this causes degrees() to fail:**
- `degrees(x) = x * 180 / π ≈ x * 57.3`
- For large f16 values (2584+), the result exceeds f16_MAX (65504) → infinity
- For max abstract-float (1.7976931348623157e+308), the result overflows to infinity
- Naga's constant evaluator calls `.to_degrees()` but doesn't detect the resulting infinity

**Note:** The `radians()` builtin doesn't fail because it divides (making values smaller), so no overflow occurs.

### Failing Test Patterns

All 12 failures follow this pattern:

**Abstract-float (constant stage only):**
- `values:stage="constant";type="abstract-float"`
- `values:stage="constant";type="vec2<abstract-float>"`
- `values:stage="constant";type="vec3<abstract-float>"`
- `values:stage="constant";type="vec4<abstract-float>"`

**F16 (constant and override stages):**
- `values:stage="constant";type="f16"`
- `values:stage="constant";type="vec2<f16>"`
- `values:stage="constant";type="vec3<f16>"`
- `values:stage="constant";type="vec4<f16>"`
- `values:stage="override";type="f16"`
- `values:stage="override";type="vec2<f16>"`
- `values:stage="override";type="vec3<f16>"`
- `values:stage="override";type="vec4<f16>"`

### Suggested Bug Reference for fail.lst

```
# Missing infinity/NaN checks for AbstractFloat and F16 in constant evaluation
# https://github.com/gfx-rs/wgpu/issues/XXXX
webgpu:shader,validation,expression,call,builtin,degrees:values:stage="constant";type="abstract-float"
webgpu:shader,validation,expression,call,builtin,degrees:values:stage="constant";type="f16"
webgpu:shader,validation,expression,call,builtin,degrees:values:stage="constant";type="vec*<abstract-float>"
webgpu:shader,validation,expression,call,builtin,degrees:values:stage="constant";type="vec*<f16>"
webgpu:shader,validation,expression,call,builtin,degrees:values:stage="override";type="f16"
webgpu:shader,validation,expression,call,builtin,degrees:values:stage="override";type="vec*<f16>"
```

### Triage Summary for triage.md

```markdown
## webgpu:shader,validation,expression,call,builtin,degrees:*

**Status:** 33P/12F (73% pass rate)

**Failing tests:** All failures are in the `values` subtest with abstract-float and f16 types.

**Root cause:** Naga's `check_literal_value()` function in `/naga/src/valid/expression.rs` only checks `Literal::F64` and `Literal::F32` for infinity/NaN, but does not check `Literal::AbstractFloat` or `Literal::F16`. When `degrees()` is evaluated at compile time for large values, it produces infinity which should be rejected but isn't detected.

**Specific failure examples:**
- `degrees(1.7976931348623157e+308)` (max f64 as abstract-float) → infinity (not detected)
- `degrees(2584h)` (f16) → infinity (not detected)
- `degrees(65504h)` (max f16) → infinity (not detected)

**Fix needed:** Update `check_literal_value()` to add:
- `crate::Literal::AbstractFloat(v) => v.is_nan()` / `v.is_infinite()`
- `crate::Literal::F16(v) => v.is_nan()` / `v.is_infinite()`

**Impact:** This affects any builtin that can overflow during constant evaluation with AbstractFloat or F16 types, not just `degrees()`.
```
