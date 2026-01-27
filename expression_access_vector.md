# Vector Access CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,expression,access,vector:*`

Model: Claude Sonnet 4.5

**Overall Status:** 44P/40F/0S (52.38%/47.62%/0%)

## Passing Sub-suites ✅

- `webgpu:shader,validation,expression,access,vector:abstract:*` - 24/24 passing (100%)

## Remaining Issues ⚠️

- `webgpu:shader,validation,expression,access,vector:concrete:*` - 20/60 passing (33.33%), 40 failures

All failures are in the `concrete` subcategory. The issue affects vector widths 2 and 3, but not 4.

## Issue Detail

### 1. Runtime vector indexing incorrectly rejected for out-of-bounds compile-time-known indices

**Test selector:** `webgpu:shader,validation,expression,access,vector:concrete:vector_decl="let";*` and `vector_decl="var";*`

**Affected tests:** All tests with `vector_width=2` or `vector_width=3` that use `let` or `var` variables with indices 2 or 3.

### 2. Swizzle validation missing for out-of-bounds component access

**Test selector:** `webgpu:shader,validation,expression,access,vector:concrete:*` (swizzle cases)

**Affected tests:** All tests with `vector_width=2` or `vector_width=3` that use swizzles accessing z or w components.

## Detailed Analysis

### Issue 1: Overly strict bounds checking for runtime vector indexing

**Test selector:** `webgpu:shader,validation,expression,access,vector:concrete:*`

**What it tests:** Vector indexing and swizzling operations using various index types (literals, `const`, `let`, `var` variables) across different vector widths (2, 3, 4) and element types (i32, u32, f32, f16, bool).

**Failure pattern:**
- All tests with `vector_width=2` fail (20 tests)
- All tests with `vector_width=3` fail (20 tests)
- All tests with `vector_width=4` pass (20 tests)

**Example failures:**

1. `const` variable indexing:
```
webgpu:shader,validation,expression,access,vector:concrete:vector_decl="const";vector_width=2;element_type="i32"
```

Generated code:
```wgsl
@compute @workgroup_size(1)
fn main() {
  const v = vec2<i32>();
  const i = 2; let r : i32 = v[i];
}
```

**Error:**
```
Shader validation error: Entry point main at Compute is invalid
   ┌─ :13:26
   │
13 │   const i = 2; let r : T = v[i];
   │                          ^^^^ naga::ir::Expression [2]
   │
   = Expression [2] is invalid
   = Accessing index 2 is out of [0] bounds
```

2. `let` variable indexing:
```
webgpu:shader,validation,expression,access,vector:concrete:vector_decl="let";vector_width=2;element_type="i32"
```

Generated code:
```wgsl
@compute @workgroup_size(1)
fn main() {
  let v = vec2<i32>();
  let i = 2; let r : i32 = v[i];
}
```

**Error:**
```
Shader validation error: Entry point main at Compute is invalid
   ┌─ :13:26
   │
13 │   let i = 2; let r : T = v[i];
   │                          ^^^^ naga::ir::Expression [2]
   │
   = Expression [2] is invalid
   = Accessing index 2 is out of [0] bounds
```

**Root cause:**

Naga's expression validator (`naga/src/valid/expression.rs`, lines 293-318) performs constant value extraction on `Access` expressions to check if the index is compile-time known. When it is, it performs bounds checking during shader module validation.

The issue is in how Naga handles different variable types:

1. **For `const` variables with out-of-bounds indices:** According to the WGSL spec and WebGPU CTS expectations, `const i = 2; v[i]` where `v` is a `vec2` SHOULD fail validation because the index is a constant expression that's provably out of bounds. **Naga is correctly rejecting these cases.**

2. **For `let` and `var` variables:** The expression `let i = 2; v[i]` uses runtime indexing semantics. Even though the value is known at compile-time, WGSL semantics require this to be treated as a runtime index. Runtime out-of-bounds vector access should result in **clamping** behavior (accessing the last valid element), not a validation error. **Naga is incorrectly rejecting these cases.**

From the test source (`/Users/Andy/Development/cts/src/webgpu/shader/validation/expression/access/vector.spec.ts`):
- Lines 28-39: `const` variable indices with out-of-bounds values should fail (these ARE correctly failing)
- Lines 42-53: `let` variable indices should ALWAYS pass (`ok: true`), even with out-of-bounds values (these are INCORRECTLY failing)
- Lines 56-67: `var` variable indices should ALWAYS pass (`ok: true`), even with out-of-bounds values (these are INCORRECTLY failing)

The validator is treating `let i = 2` as a constant expression and performing bounds checking, when it should recognize this as runtime indexing.

**Why vector width 4 passes:**
For `vec4`, indices 0-3 are all valid, so accessing index 2 or 3 doesn't trigger the bounds check error. The test cases only test indices up to 3, so `vec4` naturally passes all tests.

**Comparison with arrays:**
The array access tests (`webgpu:shader,validation,expression,access,array:*`) have a 97.37% pass rate with only 2/76 failures. This suggests the same issue may affect arrays but to a much lesser extent, likely because the test cases for arrays use different index values or array sizes that don't trigger the problem as often.

**Fix needed:**

The fix requires distinguishing between:
1. **Constant expression indexing** (`const i = 2; v[i]`) - should perform bounds checking
2. **Runtime indexing with constant values** (`let i = 2; v[i]`) - should NOT perform bounds checking at compile time

The key is in `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs` around line 293-318. The validator needs to check not just whether the index value is constant, but whether the index is in a **constant expression context**.

Specifically, `let` and `var` variables should never be treated as constant expressions for the purpose of compile-time bounds checking, even if their values are known. Only `const` variables, literals, and other const-expressions should trigger compile-time bounds checks.

One potential approach: Check the expression type before calling `get_const_val_from()`. If the index expression is a `LocalVariable` reference where the variable is declared with `let` or `var`, skip the bounds check. Only perform bounds checking for literals, `Constant` expressions, and references to `const` variables.

This would align Naga's behavior with the WGSL specification's distinction between constant expressions and runtime expressions.

### Issue 2: Missing swizzle validation for out-of-bounds component access

**What it tests:** Swizzle operations on vectors that access components beyond the vector's width.

**Example failure:**
```
webgpu:shader,validation,expression,access,vector:concrete:vector_decl="const";vector_width=2;element_type="i32"
```
With subcase `xyz`:

Generated code:
```wgsl
@compute @workgroup_size(1)
fn main() {
  const v = vec2<i32>();
  let r : vec3<i32> = v.xyz;
}
```

**Expected:** Compilation should fail because `vec2` only has `x` and `y` components, but the swizzle attempts to access `z`.

**Actual:** Naga accepts this shader without error.

**Similar failures:**
- `xyz`, `zyx` - access z component on vec2 (should fail)
- `xyxz` - access z component on vec2 (should fail)
- `xyzw`, `yxwz`, `wxyz_bga_xy` - access w component on vec2 or vec3 (should fail)
- `rgb`, `grb`, `rgba`, `gbra`, `rgbr`, `rbrg_xyzw` - color swizzles that access b (blue/z) or a (alpha/w) on vec2 (should fail)

**Root cause:**

Naga's swizzle validation is not checking whether the swizzle components (x, y, z, w or r, g, b, a) are valid for the vector's width. It appears to accept any syntactically valid swizzle pattern without verifying that all components are within bounds.

For a `vec2`, only components with indices 0-1 (x/y or r/g) should be accessible.
For a `vec3`, only components with indices 0-2 (x/y/z or r/g/b) should be accessible.
For a `vec4`, all components with indices 0-3 (x/y/z/w or r/g/b/a) are accessible.

**Fix needed:**

Add validation in Naga's swizzle handling to ensure that all components in a swizzle pattern are valid for the source vector's width. This likely needs to be done during WGSL parsing or during the expression validation phase.

The swizzle validation should:
1. Determine the width of the source vector
2. Check each component in the swizzle pattern
3. Reject swizzles that access components beyond the vector's width

**Test failure breakdown:**

Out of 40 total failures for `concrete` tests:
- Approximately 12 failures are from Issue 1 (runtime indexing with `let`/`var`)
- Approximately 28 failures are from Issue 2 (swizzle validation)

Both issues need to be fixed to achieve 100% pass rate on vector access tests.
