Good, they're already in fail.lst. Now let me create my summary.

## CTS Triage Summary

Based on my analysis of the test selector `webgpu:shader,validation,expression,call,builtin,exp:*`, I've identified the root cause of the failures.

### Overall Statistics
- **Total tests**: 41 (not including the 11 skipped due to no f16 support in wgpu)
- **Passed**: 25 / 41 = 60.98%
- **Failed**: 16 / 41 = 39.02%
- **Skipped**: 0 / 41 = 0%

### Failure Pattern

The failures follow a clear pattern based on the numeric type being tested:

**Passing tests:**
- `stage="constant";type="f32"` and all f32 vector variants
- `stage="override";type="f32"` and all f32 vector variants
- All `args:*` tests (argument validation)
- All `must_use:*` tests

**Failing tests:**
- `stage="constant";type="abstract-int"` 
- `stage="constant";type="abstract-float"` and all abstract-float vector variants
- `stage="constant";type="f16"` and all f16 vector variants
- `stage="override";type="f16"` and all f16 vector variants

### Root Cause

The root cause is in **`/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs`**, specifically in the `check_literal_value()` function at lines 1321-1341.

This function validates literal values after constant evaluation, checking for NaN and infinity values. However, it only checks `Literal::F64` and `Literal::F32`:

```rust
let is_nan = match literal {
    crate::Literal::F64(v) => v.is_nan(),
    crate::Literal::F32(v) => v.is_nan(),
    _ => false,  // AbstractFloat and F16 not checked!
};

let is_infinite = match literal {
    crate::Literal::F64(v) => v.is_infinite(),
    crate::Literal::F32(v) => v.is_infinite(),
    _ => false,  // AbstractFloat and F16 not checked!
};
```

The wildcard pattern `_ => false` means that `Literal::AbstractFloat` and `Literal::F16` are **not** checked for NaN or infinity. When `exp()` is evaluated at constant-evaluation time with values that cause overflow (e.g., `exp(1000)` for abstract types, `exp(16.1)` for f16), Rust's `exp()` method returns infinity, but this infinity value is not caught and rejected for AbstractFloat and F16 types.

According to the WGSL spec and CTS expectations, constant evaluation that produces non-representable values (infinity) should result in a compilation error. The test validates this by calling `isRepresentable(Math.exp(value), targetType)` and expecting a validation error when it returns false.

### Suggested Bug Reference

For `/Users/Andy/Development/wgpu2/cts_runner/fail.lst`:

```
webgpu:shader,validation,expression,call,builtin,exp:* // Missing overflow validation for AbstractFloat and F16 in constant evaluation
```

### Fix Needed

The fix is to add checks for `Literal::AbstractFloat` and `Literal::F16` in the `check_literal_value()` function in `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs`. The matches should be:

```rust
let is_nan = match literal {
    crate::Literal::F64(v) => v.is_nan(),
    crate::Literal::F32(v) => v.is_nan(),
    crate::Literal::AbstractFloat(v) => v.is_nan(),
    crate::Literal::F16(v) => f32::from(v).is_nan(),
    _ => false,
};

let is_infinite = match literal {
    crate::Literal::F64(v) => v.is_infinite(),
    crate::Literal::F32(v) => v.is_infinite(),
    crate::Literal::AbstractFloat(v) => v.is_infinite(),
    crate::Literal::F16(v) => f32::from(v).is_infinite(),
    _ => false,
};
```

This same fix will likely apply to other math functions that can overflow (exp2, pow, sinh, cosh, etc.).

### Files Involved

- **Bug location**: `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs` (lines 1321-1341)
- **Constant evaluator**: `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` (line 1653 for exp())
- **Test source**: `/Users/Andy/Development/cts/src/webgpu/shader/validation/expression/call/builtin/exp.spec.ts`
