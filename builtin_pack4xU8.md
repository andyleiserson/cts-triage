# pack4xU8 CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,expression,call,builtin,pack4xU8:*`

Model: Claude Sonnet 4.5

**Overall Status:** 14P/3F/2S (73.68% pass rate)

## Summary

This test suite validates the `pack4xU8` WGSL builtin function, which is part of the `packed_4x8_integer_dot_product` language extension. The function packs four unsigned 8-bit integers from a `vec4<u32>` into a single `u32` value with wrapping behavior.

The test suite has a 73.68% pass rate with 3 failures due to missing constant evaluation support in Naga.

## Passing Sub-suites ✅

### 1. `args` - Argument Validation (12/13 tests passing)
Tests that verify argument type validation:
- All `bad_*` tests pass (wrong types, wrong number of args)
- The `good` test fails (see Remaining Issues below)

### 2. `must_use` - Must-Use Attribute (2/2 tests passing)
Tests that verify the builtin result must be used:
- `use=true`: Result is assigned → passes
- `use=false`: Result is discarded → correctly fails validation

### 3. `unsupported` - Feature Not Supported (2/2 tests skipped)
Tests that verify the builtin is rejected when `packed_4x8_integer_dot_product` is not supported:
- Both tests are correctly skipped because wgpu supports the feature

## Remaining Issues ⚠️

### 1. Missing Constant Evaluation Support

**Failing tests:**
- `supported:requires=false` (1 test)
- `supported:requires=true` (1 test)
- `args:arg="good"` (1 test)

All three failures have the **same root cause**.

## Issue Detail

### 1. pack4xU8 Not Implemented as Constant Expression

**Test selector:** `webgpu:shader,validation,expression,call,builtin,pack4xU8:supported:*`

**What it tests:** Tests that `pack4xU8` can be used in constant expressions when the `packed_4x8_integer_dot_product` feature is supported.

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,pack4xU8:supported:requires=false
```

**Error:**
```
Shader '' parsing error: Not implemented as constant expression: Pack4xU8 built-in function
  ┌─ wgsl:1:11
  │
1 │ const c = pack4xU8(vec4u());
  │           ^^^^^^^^ see msg
```

**Root cause:**
Naga does not implement constant evaluation for the `pack4xU8` built-in function. The function is explicitly listed in the "Not Implemented" section of the constant evaluator at `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` lines 1883-1895:

```rust
| crate::MathFunction::Pack4xI8
| crate::MathFunction::Pack4xU8
| crate::MathFunction::Pack4xI8Clamp
| crate::MathFunction::Pack4xU8Clamp
...
=> Err(ConstantEvaluatorError::NotImplemented(
    format!("{fun:?} built-in function"),
)),
```

The function works correctly at runtime (in non-const contexts), but fails when used in:
- `const` declarations
- `override` declarations
- Other constant evaluation contexts

**Fix needed:**
Implement constant evaluation for `pack4xU8` in Naga's constant evaluator. This is tracked as part of GitHub issue #4507 ("Functions Missing Constant Evaluation Support").

The implementation would need to:
1. Take a `vec4<u32>` input
2. Wrap each component to 8-bit unsigned range (cast to u8)
3. Pack the four bytes into a u32
4. Handle the byte order according to WebGPU spec

**Related functions with the same issue:**
- `pack4xI8` - Same issue, signed variant
- `pack4xI8Clamp` - Same issue, signed clamping variant
- `pack4xU8Clamp` - Same issue, unsigned clamping variant
- All other pack/unpack functions (pack2x16float, pack4x8snorm, etc.)

## Test Categories Breakdown

| Category | Passing | Failing | Skipped | Total |
|----------|---------|---------|---------|-------|
| unsupported | 0 | 0 | 2 | 2 |
| supported | 0 | 2 | 0 | 2 |
| args (good) | 0 | 1 | 0 | 1 |
| args (bad_*) | 12 | 0 | 0 | 12 |
| must_use | 2 | 0 | 0 | 2 |
| **Total** | **14** | **3** | **2** | **19** |

## Expected After Fix

Once constant evaluation is implemented for `pack4xU8`:
- Pass rate should increase to 89.47% (17P/0F/2S)
- The 2 `unsupported` tests will remain skipped (correct behavior)
- All other tests should pass

## Related Issues

- **GitHub Issue:** #4507 - Functions Missing Constant Evaluation Support
- **Related test suites:**
  - `webgpu:shader,validation,expression,call,builtin,pack4xI8:*` (73% pass, same issue)
  - `webgpu:shader,validation,expression,call,builtin,pack4xI8Clamp:*` (73% pass, same issue)
  - `webgpu:shader,validation,expression,call,builtin,pack4xU8Clamp:*` (73% pass, same issue)
  - `webgpu:shader,validation,expression,call,builtin,unpack4xI8:*` (75% pass, same issue)
  - `webgpu:shader,validation,expression,call,builtin,unpack4xU8:*` (75% pass, same issue)
  - All other pack/unpack builtins (pack2x16float, pack4x8snorm, etc.)

## What pack4xU8 Does

`pack4xU8` packs four unsigned 32-bit integers into a single 32-bit unsigned integer by:
1. Taking each u32 component from the input vec4
2. Converting to 8-bit unsigned integers with wrapping (values > 255 wrap around)
3. Packing the four bytes into a u32

Example:
```wgsl
requires packed_4x8_integer_dot_product;

// Runtime usage (works)
fn example() {
    let packed = pack4xU8(vec4u(0, 127, 255, 256));
    // Result: bytes packed into u32 (256 wraps to 0)
}

// Constant usage (currently fails)
const c = pack4xU8(vec4u(0, 127, 255, 256)); // Error: Not implemented as constant expression
```

The difference between `pack4xU8` and `pack4xU8Clamp`:
- `pack4xU8`: Wrapping behavior (e.g., 256 wraps to 0)
- `pack4xU8Clamp`: Clamping behavior (e.g., 256 clamps to 255)
