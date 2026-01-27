# Shader Validation pack4x Packed Integer CTS Tests - Triage Report

CTS selectors:
- `webgpu:shader,validation,expression,call,builtin,pack4xI8:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xI8Clamp:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xU8:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xU8Clamp:*`

Model: Claude Sonnet 4.5

**Overall Status:** 4P/0F/15S per test suite (21.05% pass rate per suite)

## Summary

All four test suites have identical 21% pass rates and are currently listed in `cts_runner/fail.lst`. These tests validate the `pack4xI8`, `pack4xI8Clamp`, `pack4xU8`, and `pack4xU8Clamp` WGSL builtin functions, which are part of the `packed_4x8_integer_dot_product` language extension.

## Test Structure

Each test suite contains 19 tests divided into 4 categories:

### 1. `unsupported` tests (2 tests) - PASSING
Tests that verify the builtin is rejected when the `packed_4x8_integer_dot_product` feature is NOT supported.
- `requires=false`: Shader without `requires` directive should fail
- `requires=true`: Shader with `requires` directive should fail

**Status:** Both tests pass (2/2 = 100%)

These tests use `skipIfLanguageFeatureSupported()` to only run when the feature is not available. Since `wgslLanguageFeatures` is not implemented in deno_webgpu, the CTS framework treats the feature as unsupported, which is the correct state for these tests. The tests verify that Naga correctly rejects shaders using these builtins without the language extension enabled.

### 2. `supported` tests (2 tests) - SKIPPED
Tests that verify the builtin works correctly when the `packed_4x8_integer_dot_product` feature IS supported.
- `requires=false`: Shader without `requires` directive should succeed
- `requires=true`: Shader with `requires` directive should succeed

**Status:** Both tests skipped (0/2 tested)

These tests use `skipIfLanguageFeatureNotSupported()` to only run when the feature is available. Since `wgslLanguageFeatures` is not implemented, the tests are skipped.

### 3. `args` tests (13 tests) - SKIPPED
Tests that verify argument type validation when the feature is supported:
- `good`: Valid argument type (vec4i for I8 variants, vec4u for U8 variants)
- `bad_*`: Various invalid argument types and counts

**Status:** All tests skipped (0/13 tested)

These tests also use `skipIfLanguageFeatureNotSupported()` and are skipped for the same reason.

### 4. `must_use` tests (2 tests) - PASSING
Tests that verify the builtin result must be used (cannot be called as a statement):
- `use=true`: Result is assigned, should succeed
- `use=false`: Result is discarded, should fail

**Status:** Both tests pass (2/2 = 100%)

These tests do NOT have the language feature check, so they run regardless of feature support. They pass because Naga correctly enforces the `@must_use` attribute on these builtins.

## Root Cause

This is a **wgslLanguageFeatures implementation issue**. The problem is identical to the issues documented in:
- `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_parse_requires_triage.md`
- `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_validation_dot4_packed_triage.md`

**The `deno_webgpu` backend does not implement the `gpu.wgslLanguageFeatures` property.**

When `wgslLanguageFeatures` is `undefined`:
- The CTS framework's `hasLanguageFeature()` returns `false`
- Tests using `skipIfLanguageFeatureNotSupported()` are skipped
- Only tests that run without the feature (like `unsupported` and `must_use`) execute

## Implementation Status in wgpu/Naga

The `packed_4x8_integer_dot_product` language extension is **FULLY IMPLEMENTED** in Naga:

### Language Extension
- **Definition**: `ImplementedLanguageExtension::Packed4x8IntegerDotProduct`
- **File**: `naga/src/front/wgsl/parse/directive/language_extension.rs:59`

### Builtin Functions
The extension provides four packing functions and corresponding unpacking functions:

**Pack functions (relevant to these tests):**
- `pack4xI8(vec4<i32>) -> u32`: Packs four signed 8-bit integers (wrapping behavior)
- `pack4xI8Clamp(vec4<i32>) -> u32`: Packs four signed 8-bit integers (clamping to [-128, 127])
- `pack4xU8(vec4<u32>) -> u32`: Packs four unsigned 8-bit integers (wrapping behavior)
- `pack4xU8Clamp(vec4<u32>) -> u32`: Packs four unsigned 8-bit integers (clamping to [0, 255])

**Implementation locations:**
- **IR representation**: `MathFunction::Pack4xI8`, `Pack4xI8Clamp`, `Pack4xU8`, `Pack4xU8Clamp` in `naga/src/ir/mod.rs`
- **Constant evaluation**: Implemented in `naga/src/proc/constant_evaluator.rs`
- **Backend support**: All backends (SPIR-V, HLSL, MSL, GLSL) have code generation
- **Test coverage**: Test cases in `naga/tests/in/wgsl/functions.wgsl`

## What These Functions Do

These functions pack four 8-bit integer components into a single 32-bit unsigned integer:

```wgsl
requires packed_4x8_integer_dot_product;

// pack4xI8: Wrapping behavior for signed integers
const a = pack4xI8(vec4i(-128, 0, 127, 255));
// Result: 0x7F007F80 (byte layout: 0x80, 0x00, 0x7F, 0xFF after wrapping)

// pack4xI8Clamp: Clamping behavior for signed integers
const b = pack4xI8Clamp(vec4i(-200, 0, 200, 255));
// Result: 0x7F007F80 (clamped: -128, 0, 127, 127)

// pack4xU8: Wrapping behavior for unsigned integers
const c = pack4xU8(vec4u(0, 64, 128, 255));
// Result: 0xFF804000

// pack4xU8Clamp: Clamping behavior for unsigned integers
const d = pack4xU8Clamp(vec4u(0, 64, 128, 300));
// Result: 0xFF804000 (clamped: 0, 64, 128, 255)
```

The byte order follows the WebGPU specification's definition for these functions.

## Fix Needed

Add the `wgslLanguageFeatures` property to `deno_webgpu`'s `GPU` class in `/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs`. The property should return a Set-like object containing the supported language extensions:

- `readonly_and_readwrite_storage_textures`
- `packed_4x8_integer_dot_product`
- `pointer_composite_access`

This is the same fix needed for all language extension tests. Once implemented:
- The `supported` tests (2 per suite) should pass
- The `args` tests (13 per suite) should pass
- Total expected pass rate: 17/19 = 89.5% per suite

The `unsupported` and `must_use` tests will continue to pass, maintaining the current 4 passing tests.

## Expected Behavior After Fix

Once `wgslLanguageFeatures` is implemented:

### Passing Tests (17/19 per suite = 89.5%)
1. `unsupported:*` (2 tests) - Already passing
2. `supported:*` (2 tests) - Will pass after fix
3. `args:arg="good"` (1 test) - Will pass after fix
4. `args:arg="bad_*"` (12 tests) - Will pass after fix
5. `must_use:*` (2 tests) - Already passing

### Tests That Should Skip (0)
None - all tests should execute once the feature is properly advertised.

## Related Issues

- **GitHub PR**: #8884 (wgslLanguageFeatures implementation)
- **Related test suites** with the same root cause:
  - `webgpu:shader,validation,expression,call,builtin,dot4I8Packed:*` (0% pass)
  - `webgpu:shader,validation,expression,call,builtin,dot4U8Packed:*` (0% pass)
  - `webgpu:shader,validation,expression,call,builtin,unpack4xI8:*` (75% pass)
  - `webgpu:shader,validation,expression,call,builtin,unpack4xU8:*` (75% pass)
  - `webgpu:shader,validation,parse,requires:wgsl_matches_api:feature="packed_4x8_integer_dot_product"`
  - `webgpu:shader,validation,extension,pointer_composite_access:*`
  - `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*`

## Use Cases

These pack/unpack functions are useful for:
- **Quantized neural networks**: 8-bit integer weights and activations
- **Compressed data storage**: Efficient storage of small integers in buffers
- **Color packing**: RGBA8 color values in compute shaders
- **Data serialization**: Efficient bit-packing for GPU-to-CPU communication

They complement the `dot4I8Packed` and `dot4U8Packed` functions (used for dot products on packed integers) by providing the packing/unpacking primitives needed to work with the packed format.
