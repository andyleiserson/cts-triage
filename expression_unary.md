# Shader Validation Expression Unary CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,expression,unary,*`

Model: Claude Sonnet 4.5

**Overall Status:** 225P/79F/0S (74%/26%/0%)

## Passing Sub-suites ✅

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `logical_negation` | 48/48 (100%) | Tests logical NOT operator (!) validation |

## Remaining Issues ⚠️

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `address_of_and_indirection` | 96/172 (55.81%) | Tests address-of (&) and dereference (*) operators |
| `arithmetic_negation` | 40/42 (95.24%) | Tests arithmetic negation (-) operator |
| `bitwise_complement` | 41/42 (97.62%) | Tests bitwise complement (~) operator |

## Issue Detail

### 1. Pointer Composite Access Without Language Feature

**Test selector:** `webgpu:shader,validation,expression,unary,address_of_and_indirection:composite:*`

**Pass rate:** 96/172 tests pass (55.81%)

**What it tests:** Validates that pointer composite access syntax (e.g., `p[0].member` where `p` is a pointer) is only accepted when the `pointer_composite_access` language feature is supported. Also validates that dereference-then-access syntax (e.g., `(*p)[0].member`) works without the feature.

**Example failure:**
```
webgpu:shader,validation,expression,unary,address_of_and_indirection:composite:addressSpace="function";compositeType="array";storageType="bool"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.
```

**Root cause:**

The test logic:
1. CTS test queries `gpu.wgslLanguageFeatures.has('pointer_composite_access')`
2. If the feature IS advertised, the test expects shaders using `p[0]` syntax to compile successfully
3. If the feature is NOT advertised, the test expects shaders using `p[0]` syntax to fail

The problem:
- `deno_webgpu` does not expose the `wgslLanguageFeatures` API on the `GPU` object (see #8884)
- CTS sees the feature as NOT supported
- CTS expects shader compilation to FAIL for `derefType="pointer"` and `derefType="address_of_identifier"` (which have `requires_pointer_composite_access: true`)
- But Naga implements `pointer_composite_access` unconditionally
- Shader compilation SUCCEEDS
- Test fails with "Expected validation error"

**Affected tests:** 76 failures (all have `derefType="pointer"` subcases with `accessMode="read_write"`)

**Fix needed:**

This is a known issue tracked in #8884. The fix requires:
1. Exposing `wgslLanguageFeatures` on the `GPU` object in `deno_webgpu`
2. Reporting `pointer_composite_access` as a supported language feature

Once fixed, all 76 failing tests should pass because:
- CTS will see `pointer_composite_access` IS supported
- CTS will expect shaders to compile successfully
- Shaders DO compile successfully (as they already do)

See also: `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_pointer_composite_access_triage.md` for detailed analysis of the related `webgpu:shader,validation,extension,pointer_composite_access:*` selector.

---

### 2. Arithmetic Negation Accepted for Invalid Types

**Test selector:** `webgpu:shader,validation,expression,unary,arithmetic_negation:invalid_types:*`

**Pass rate:** 40/42 tests pass (95.24%)

**What it tests:** Validates that arithmetic negation (-) is rejected for non-numeric types including matrices, atomics, arrays, pointers, textures, samplers, and structs.

**Failures:**

#### 2a. Matrix Negation

**Example failure:**
```
webgpu:shader,validation,expression,unary,arithmetic_negation:invalid_types:type="mat2x2f";control=false
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
```

**Test code:**
```wgsl
var<private> m : mat2x2f;
fn main() {
  let foo = -m;
}
```

**Root cause:**
Naga accepts arithmetic negation on matrix types (`-m`), but the WebGPU spec only allows negation on scalar and vector signed numeric types. Matrices are not in the list of supported types for the negation operator.

**Fix needed:**
Add validation in Naga's WGSL frontend to reject unary negation when the operand is a matrix type.

#### 2b. Atomic Negation

**Example failure:**
```
webgpu:shader,validation,expression,unary,arithmetic_negation:invalid_types:type="atomic";control=false
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
```

**Test code:**
```wgsl
@group(0) @binding(2) var<storage, read_write> a : atomic<i32>;
fn main() {
  let foo = -a;
}
```

**Root cause:**
This is a known issue tracked in #5474. Naga allows referencing atomics directly in expressions instead of requiring `atomicLoad`, `atomicStore`, etc. The negation operator should reject direct references to atomic types.

**Fix needed:**
Part of the broader #5474 fix to require atomic operations go through atomic builtin functions.

---

### 3. Bitwise Complement Accepted for Atomic Types

**Test selector:** `webgpu:shader,validation,expression,unary,bitwise_complement:invalid_types:*`

**Pass rate:** 41/42 tests pass (97.62%)

**What it tests:** Validates that bitwise complement (~) is rejected for non-integer types including matrices, atomics, arrays, pointers, textures, samplers, and structs.

**Example failure:**
```
webgpu:shader,validation,expression,unary,bitwise_complement:invalid_types:type="atomic";control=false
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
```

**Test code:**
```wgsl
@group(0) @binding(2) var<storage, read_write> a : atomic<i32>;
fn main() {
  let foo = ~a;
}
```

**Root cause:**
This is a known issue tracked in #5474. Naga allows referencing atomics directly in expressions instead of requiring atomic builtin functions. The bitwise complement operator should reject direct references to atomic types.

**Fix needed:**
Part of the broader #5474 fix to require atomic operations go through atomic builtin functions.

---

## Summary

The unary expression validation tests have a 74% pass rate with 79 failures broken down as:
- **76 failures (96% of failures):** Missing `wgslLanguageFeatures` API causes pointer composite access tests to fail (#8884)
- **2 failures:** Invalid type validation gaps (1 matrix negation, 1 atomic negation)
- **1 failure:** Atomic type validation gap (bitwise complement)

The atomic-related failures (3 total) are part of the known issue #5474.

The matrix negation failure is a straightforward validation gap in Naga.

The largest impact is the missing `wgslLanguageFeatures` API (#8884), which accounts for 96% of failures.
