# Shader Validation Expression Unary CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,expression,unary,*`

Model: Claude Sonnet 4.5

**Overall Status:** 301P/3F/0S (99%/1%/0%)

## Passing Sub-suites ✅

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `address_of_and_indirection` | 172/172 (100%) | Tests address-of (&) and dereference (*) operators |
| `logical_negation` | 48/48 (100%) | Tests logical NOT operator (!) validation |

## Remaining Issues ⚠️

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `arithmetic_negation` | 40/42 (95.24%) | Tests arithmetic negation (-) operator |
| `bitwise_complement` | 41/42 (97.62%) | Tests bitwise complement (~) operator |

## Issue Detail

### 1. Arithmetic Negation Accepted for Invalid Types

**Test selector:** `webgpu:shader,validation,expression,unary,arithmetic_negation:invalid_types:*`

**Pass rate:** 40/42 tests pass (95.24%)

**What it tests:** Validates that arithmetic negation (-) is rejected for non-numeric types including matrices, atomics, arrays, pointers, textures, samplers, and structs.

**Failures:**

#### 1a. Matrix Negation

**Example failure:**
```
webgpu:shader,validation,expression,unary,arithmetic_negation:invalid_types:type="mat2x2f";control=false
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.
```

**Test code:**
```wgsl
var<private> m : mat2x2f;
fn main() {
  let foo = -m;
}
```

**Root cause:**
Naga accepts arithmetic negation on matrix types (`-m`), but the WebGPU spec only allows negation on scalar and vector signed numeric types. According to the CTS test at line 108 in `arithmetic_negation.spec.ts`, the test expects `control=false` (i.e., the direct expression) to fail compilation, but Naga accepts it.

The WGSL spec defines the negation operator for scalar and vector types only. Matrix negation is not a valid operation.

**Fix needed:**
Add validation in Naga's WGSL frontend to reject unary negation when the operand is a matrix type. This would require checking the type in Naga's expression validation and emitting an error for matrix types.

#### 1b. Atomic Negation

**Example failure:**
```
webgpu:shader,validation,expression,unary,arithmetic_negation:invalid_types:type="atomic";control=false
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.
```

**Test code:**
```wgsl
@group(0) @binding(2) var<storage, read_write> a : atomic<i32>;
fn main() {
  let foo = -a;
}
```

**Root cause:**
This is a known issue tracked in https://github.com/gfx-rs/wgpu/issues/5474. Naga allows referencing atomics directly in expressions instead of requiring `atomicLoad`, `atomicStore`, etc. The negation operator should reject direct references to atomic types.

According to the CTS test source (line 58-59), the control case uses `atomicLoad(&a)` which should be valid, but the test case directly references `a` which should be invalid.

**Fix needed:**
Part of the broader #5474 fix to require atomic operations go through atomic builtin functions. Do not attempt to fix - refer user to https://github.com/gfx-rs/wgpu/issues/5474.

---

### 2. Bitwise Complement Accepted for Atomic Types

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
VALIDATION FAILED: Missing expected compilationInfo 'error' message.
```

**Test code:**
```wgsl
@group(0) @binding(2) var<storage, read_write> a : atomic<i32>;
fn main() {
  let foo = ~a;
}
```

**Root cause:**
This is a known issue tracked in https://github.com/gfx-rs/wgpu/issues/5474. Naga allows referencing atomics directly in expressions instead of requiring atomic builtin functions. The bitwise complement operator should reject direct references to atomic types.

According to the CTS test source (line 58-59 in `bitwise_complement.spec.ts`), the control case uses `atomicLoad(&a)` which should be valid, but the test case directly references `a` which should be invalid.

**Fix needed:**
Part of the broader #5474 fix to require atomic operations go through atomic builtin functions. Do not attempt to fix - refer user to https://github.com/gfx-rs/wgpu/issues/5474.

---

## Summary

The unary expression validation tests now have a 99% pass rate (improved from 74% in previous triage) with only 3 failures:
- **2 failures:** Atomic type validation gaps (1 arithmetic negation, 1 bitwise complement) - part of known issue #5474
- **1 failure:** Matrix negation validation gap - straightforward fix needed in Naga

### Progress Since Previous Triage

The previous triage (225P/79F) showed 76 failures related to missing `wgslLanguageFeatures` API (#8884). These have all been resolved, indicating that:
- The `wgslLanguageFeatures` API has been implemented
- The `pointer_composite_access` feature is now properly advertised
- All 172 `address_of_and_indirection` tests now pass

The remaining 3 failures are:
1. **Matrix negation** - New validation gap that could be fixed in Naga
2. **Atomic negation and complement** - Part of the known #5474 issue

### Recommended Actions

1. **Matrix negation validation (Priority: Medium)** - Add validation in Naga to reject unary negation on matrix types. This is a straightforward spec compliance fix.

2. **Atomic expression validation (Priority: Low)** - Already tracked under #5474. No immediate action needed as this is part of a broader fix.
