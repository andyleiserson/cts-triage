# Increment/Decrement Statement CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,statement,increment_decrement:*`

Model: Claude Sonnet 4.5

**Overall Status:** 218P/4F/0S (98.2%/1.8%/0%)

**Note:** This triage was performed after fixing the for-loop initializer issue. Original status before fix was 216P/6F/0S (97.3%/2.7%/0%).

## Passing Sub-suites ✅

- `component:*` - 60/60 passing (100%)
  - Tests increment/decrement on various component accesses (vector components, array indices, struct fields)
  - All passing - wgpu correctly validates that only i32/u32 scalar components can be incremented/decremented

- `parse:*` - 128/128 passing (100%)
  - Tests parsing of increment/decrement in various contexts
  - All passing after fixing for-loop initializer support

- `var_init_type:*` - 30/34 passing (88.2%)
  - Tests increment/decrement on variables of different types
  - Only 4 failures related to atomic types (see Issue #1 below)

## Remaining Issues ⚠️

One issue causing the 4 remaining test failures:

1. **Atomic type validation gap** (4 failures) - wgpu accepts increment/decrement on atomics when it should reject them

## Issue Detail

### 1. Atomic Type Increment/Decrement Validation Gap

**Test selector:** `webgpu:shader,validation,statement,increment_decrement:var_init_type:type="atomic_*";direction=*`

**What it tests:** Whether increment/decrement statements are correctly rejected when applied to atomic types.

**Example failures:**
```
webgpu:shader,validation,statement,increment_decrement:var_init_type:type="atomic_u32";direction="up"
webgpu:shader,validation,statement,increment_decrement:var_init_type:type="atomic_u32";direction="down"
webgpu:shader,validation,statement,increment_decrement:var_init_type:type="atomic_i32";direction="up"
webgpu:shader,validation,statement,increment_decrement:var_init_type:type="atomic_i32";direction="down"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error

---- shader ----

var<workgroup> xu: atomic<u32>;
fn f() {
  var a = xu;
  a++;
}
```

The test expects this shader to fail validation, but wgpu accepts it.

**Root cause:**

According to the WGSL spec section [Increment and Decrement Statements](https://gpuweb.github.io/gpuweb/wgsl/#increment-decrement), the expression in an increment/decrement statement:

> must evaluate to a reference with a concrete integer scalar store type and read_write access mode.

The issue is that when `var a = xu` is executed where `xu` is `atomic<u32>`, the variable `a` gets the atomic type, not the underlying integer type. Naga is allowing `a++` even though `a` has type `ref<function, atomic<u32>, read_write>` rather than `ref<function, u32, read_write>`.

This is a known pattern documented in the triage skill under "Pattern: Atomics accepted incorrectly":
- Naga allows referencing an atomic directly in an expression
- Should only allow accessing via `atomicLoad`, `atomicStore`, etc.
- Related to issue [#5474](https://github.com/gfx-rs/wgpu/issues/5474)

**Fix needed:**

This is part of the broader atomic validation issue tracked in [#5474](https://github.com/gfx-rs/wgpu/issues/5474). The fix would require Naga's type system to properly distinguish atomic types from their underlying integer types and reject operations like increment/decrement that require concrete integer scalars.

### 2. For-Loop Initializer Does Not Accept Increment/Decrement (FIXED ✅)

**Test selector:** `webgpu:shader,validation,statement,increment_decrement:parse:test="in_for_init";direction=*`

**What it tests:** Whether increment/decrement statements are allowed in the initializer part of a for-loop.

**Status:** Fixed - all tests now passing.

**Root cause:**

The WGSL spec section [For Statement](https://gpuweb.github.io/gpuweb/wgsl/#for-statement) allows increment/decrement statements in the for-loop initializer, but Naga's parser was only accepting Call, Assign, and LocalDecl statement types.

**Fix applied:**

Modified `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs` to accept `ast::StatementKind::Increment(_)` and `ast::StatementKind::Decrement(_)` in for-loop initializers. Also updated the error message in `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/error.rs` to be more accurate.

## Summary

Out of 222 tests:
- **218 passing** (98.2%) - Core increment/decrement validation works correctly
- **4 failing** (1.8%) - All related to atomic type validation gap (issue #5474)

The atomic type issue is part of a broader validation gap tracked in issue #5474, where Naga allows direct references to atomics when it should only allow access via atomic builtin functions.
