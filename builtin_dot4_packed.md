# Shader Validation dot4 Packed CTS Tests - Triage Report

CTS selectors:
- `webgpu:shader,validation,expression,call,builtin,dot4I8Packed:*`
- `webgpu:shader,validation,expression,call,builtin,dot4U8Packed:*`

Model: Claude Sonnet 4.5

**Overall Status:** 0P/??F/0S (0% pass rate)

## Summary

Both test suites have 0% pass rate and are currently listed in `cts_runner/fail.lst`. These tests validate the `dot4I8Packed` and `dot4U8Packed` WGSL builtin functions, which are part of the `packed_4x8_integer_dot_product` language extension.

## What These Tests Do

The validation tests for these builtins check:

1. **Feature detection (`unsupported` test)**: When `packed_4x8_integer_dot_product` is NOT supported (detected via `gpu.wgslLanguageFeatures.has('packed_4x8_integer_dot_product')`), shaders using `dot4I8Packed` or `dot4U8Packed` should fail to compile, regardless of whether a `requires` directive is present.

2. **Feature availability (`supported` test)**: When the feature IS supported, shaders using these builtins should compile successfully, with or without the `requires` directive.

3. **Argument validation (`args` test)**: When the feature is supported, the builtins should validate argument types correctly:
   - Valid: `dot4I8Packed(1u, 2u)` - both arguments are u32
   - Invalid: wrong number of arguments, wrong types (i32, f32, bool, vectors, arrays, structs)

4. **Must-use validation (`must_use` test)**: When the feature is supported, the result must be used (cannot call `dot4I8Packed(1u, 2u);` without assigning the result).

## Root Cause

This is a **wgslLanguageFeatures implementation issue**. The problem is identical to the issue documented in `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_parse_requires_triage.md`:

**The `deno_webgpu` backend does not implement the `gpu.wgslLanguageFeatures` property.**

When `wgslLanguageFeatures` is `undefined`:
- The CTS test helper `hasLanguageFeature()` returns `false`
- Tests expect the feature to be unsupported, so they expect shader compilation to **fail**
- But Naga has `packed_4x8_integer_dot_product` implemented (see `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/directive/language_extension.rs:59`)
- Naga accepts the `dot4I8Packed` and `dot4U8Packed` builtins and compiles successfully
- Result: Test expects error, compilation succeeds = **FAIL**

## Implementation Status in wgpu/Naga

The `packed_4x8_integer_dot_product` language extension is **FULLY IMPLEMENTED** in Naga:

- **Language extension definition**: `ImplementedLanguageExtension::Packed4x8IntegerDotProduct` (language_extension.rs:59)
- **Builtin functions**: `MathFunction::Dot4I8Packed` and `MathFunction::Dot4U8Packed` (ir/mod.rs:1314-1315)
- **Backend support**: SPIR-V, HLSL, MSL, GLSL all have specialized code generation for these builtins
- **Constant evaluation**: Implemented in proc/constant_evaluator.rs
- **Test coverage**: Naga has test cases in tests/in/wgsl/functions.wgsl

## Fix Needed

Add the `wgslLanguageFeatures` property to `deno_webgpu`'s `GPU` class (in `/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs`). The property should return a Set-like object containing:
- `readonly_and_readwrite_storage_textures`
- `packed_4x8_integer_dot_product`
- `pointer_composite_access`

This is the same fix needed for the tests documented in `shader_parse_requires_triage.md`.

## Related Issues

- GitHub issue for wgslLanguageFeatures: Referenced in PR #8884
- Related test suites with the same root cause:
  - `webgpu:shader,validation,parse,requires:wgsl_matches_api:feature="packed_4x8_integer_dot_product"`
  - `webgpu:shader,validation,extension,pointer_composite_access:*`
  - Any other tests that use `skipIfLanguageFeatureNotSupported()`

## Additional Context

The `dot4I8Packed` and `dot4U8Packed` functions perform dot products on 4-component vectors of 8-bit integers packed into u32 values:

```wgsl
// Signature
@const fn dot4I8Packed(e1: u32, e2: u32) -> i32
@const fn dot4U8Packed(e1: u32, e2: u32) -> u32

// Example usage (requires packed_4x8_integer_dot_product feature)
requires packed_4x8_integer_dot_product;

const result = dot4I8Packed(0x01020304u, 0x05060708u);
// Computes: 1*5 + 2*6 + 3*7 + 4*8 = 70
```

These functions are useful for machine learning workloads using 8-bit quantized weights and are supported by hardware instructions on many GPUs (DP4a on Vulkan/OpenGL, dot4add on D3D12, etc.).
