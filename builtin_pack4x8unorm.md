Good! So the test structure is:
- `args` tests (17 tests): 2 good cases + 15 bad cases
- `must_use` tests (2 tests): use=true and use=false
- `return` tests (5 tests): testing different return types

Total: 24 tests, 21 passing, 3 failing.

The 3 failing tests are:
1. `args:arg="good"` - Should pass, tests valid argument `vec4f()`
2. `args:arg="good_vec4_abstract_float"` - Should pass, tests valid argument `vec4(0.1)`
3. `return:type="u32"` - Should pass, tests correct return type

All three failures are because these tests use the function in a constant expression context, which Naga doesn't support yet.

Now I have all the information I need to provide a comprehensive triage report. Let me summarize my findings:

## Summary of CTS Triage for `webgpu:shader,validation,expression,call,builtin,pack4x8unorm:*`

**Overall Status:** 21P/3F/0S (87.50% pass rate)

### Root Cause

Naga does not implement `pack4x8unorm()` and `unpack4x8unorm()` as constant expressions, even though the WGSL specification marks them with the `@const` attribute. When these functions are used in constant contexts (e.g., `const c = pack4x8unorm(vec4f())`), Naga raises a `ConstantEvaluatorError::NotImplemented`.

**Location in code:** `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs:1863`

The error explicitly lists `Pack4x8unorm` and `Unpack4x8unorm` (along with all other pack/unpack functions) as not implemented for constant evaluation.

### Failing Tests (3/24 = 12.5%)

1. **`args:arg="good"`** - Tests `const c = pack4x8unorm(vec4f());`
   - **Error:** `Not implemented as constant expression: Pack4x8unorm built-in function`
   - **Expected:** Should compile successfully

2. **`args:arg="good_vec4_abstract_float"`** - Tests `const c = pack4x8unorm(vec4(0.1));`
   - **Error:** `Not implemented as constant expression: Pack4x8unorm built-in function`
   - **Expected:** Should compile successfully

3. **`return:type="u32"`** - Tests `const c: u32 = pack4x8unorm(vec4f());`
   - **Error:** `Not implemented as constant expression: Pack4x8unorm built-in function`
   - **Expected:** Should compile successfully

### Passing Tests (21/24 = 87.5%)

All other tests pass, including:
- **`args:arg="bad_*"`** (15 tests) - Invalid argument types correctly rejected
- **`must_use:use=false`** - Correctly rejects when result is not used
- **`must_use:use=true`** - Correctly accepts when result is used
- **`return:type=<non-u32>`** (4 tests) - Correctly rejects invalid return types

These tests pass because they either test validation errors (which work correctly) or don't use the function in constant contexts.

### Affected Functions

This issue affects all pack/unpack builtin functions in WGSL:

**Pack functions:**
- `pack4x8snorm`, `pack4x8unorm` (88% pass rate each)
- `pack2x16snorm`, `pack2x16unorm`, `pack2x16float` (88% pass rate each)
- `pack4xI8`, `pack4xU8`, `pack4xI8Clamp`, `pack4xU8Clamp` (different issue - wgslLanguageFeatures)

**Unpack functions:**
- `unpack4x8snorm`, `unpack4x8unorm` (81% pass rate each)
- `unpack2x16snorm`, `unpack2x16unorm`, `unpack2x16float` (81% pass rate each)
- `unpack4xI8`, `unpack4xU8` (different issue - wgslLanguageFeatures)

### Bug Reference for fail.lst

**Suggested comment:** `pack/unpack not implemented as constant expressions`

### Fix Needed

Implement constant evaluation for `pack4x8unorm` and related pack/unpack functions in `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs`. 

The functions need to be removed from the "NotImplemented" match arm (lines 1862-1879) and actual implementations need to be added to the constant evaluator.

For `pack4x8unorm(vec4<f32>) -> u32`, the implementation should:
1. Clamp each f32 component to [0.0, 1.0]
2. Convert to u8 by multiplying by 255 and rounding
3. Pack the four u8 values into a u32

For `unpack4x8unorm(u32) -> vec4<f32>`, the implementation should:
1. Extract four u8 values from the u32
2. Convert each to f32 by dividing by 255.0

Similar logic applies to the snorm variants (which use signed values and the range [-1.0, 1.0]) and the float16 variants.
