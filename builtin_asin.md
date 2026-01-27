So values -1, 0, 1 pass (as they should, since they're in the valid range), but values outside that range like -2 and 2 should fail but don't. This suggests that abstract-int values are being converted to float for the asin operation, and then the result isn't being validated for NaN.

Now I have enough information to provide a complete triage report. Let me create a summary:

## CTS Triage Summary for `webgpu:shader,validation,expression,call,builtin,asin:*`

### Overall Statistics
- **Total tests**: 56
- **Passing**: 40 (71.43%)
- **Failing**: 16 (28.57%)
- **Skipped**: 0

### Test Breakdown by Subcategory

1. **`integer_argument:*`** - ✅ All passing (9/9 = 100%)
   - Tests that integer arguments are correctly rejected

2. **`parameters:*`** - ✅ All passing (23/23 = 100%)
   - Tests for parameter validation (wrong types, wrong number of params, etc.)

3. **`values:*`** - ❌ Partially failing (8/24 = 33.33% passing, 16/24 = 66.67% failing)
   - Tests that validate value ranges for asin() during constant/override evaluation

### Failing Tests Pattern

All 16 failures are in the `values` subcategory and fall into two groups:

**Group 1: Constant stage with abstract-int, abstract-float, and f16 types** (12 failures)
- `stage="constant";type="abstract-int"`
- `stage="constant";type="abstract-float"`
- `stage="constant";type="f16"`
- `stage="constant";type="vec2<abstract-int>"`
- `stage="constant";type="vec3<abstract-int>"`
- `stage="constant";type="vec4<abstract-int>"`
- `stage="constant";type="vec2<abstract-float>"`
- `stage="constant";type="vec3<abstract-float>"`
- `stage="constant";type="vec4<abstract-float>"`
- `stage="constant";type="vec2<f16>"`
- `stage="constant";type="vec3<f16>"`
- `stage="constant";type="vec4<f16>"`

**Group 2: Override stage with f16 types** (4 failures)
- `stage="override";type="f16"`
- `stage="override";type="vec2<f16>"`
- `stage="override";type="vec3<f16>"`
- `stage="override";type="vec4<f16>"`

### Root Cause

The `asin()` function has a domain restriction: it only accepts values in the range [-1, 1]. When given values outside this range during constant or override evaluation, the shader should fail to compile.

**Current behavior:**
- ✅ **f32 types work correctly**: When `asin(2.0)` is evaluated with f32, it produces NaN, which is caught by Naga's `check_literal_value()` function and rejected.
- ❌ **abstract-int, abstract-float, and f16 types fail**: The `check_literal_value()` function in `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs` (lines 1321-1341) only checks for NaN and Infinity on `F64` and `F32` literals, but not on `AbstractFloat` or `F16` literals.

**Specific issue location:**
```rust
pub fn check_literal_value(literal: crate::Literal) -> Result<(), LiteralError> {
    let is_nan = match literal {
        crate::Literal::F64(v) => v.is_nan(),
        crate::Literal::F32(v) => v.is_nan(),
        _ => false,  // <-- Missing AbstractFloat and F16 checks
    };
    // ... similar for is_infinite
}
```

### Suggested Fix

The `check_literal_value()` function needs to be extended to check for NaN and Infinity on `AbstractFloat` (f64) and `F16` (half::f16) literals as well.

### Bug Reference for fail.lst

**Suggested description**: `Missing NaN/Infinity validation for AbstractFloat and F16 in constant evaluation`

### Triage Summary for triage.md

**Selector**: `webgpu:shader,validation,expression,call,builtin,asin:*`

**Status**: 40/56 passing (71.43%)

**What it tests**: Validates that the `asin()` builtin correctly rejects invalid arguments (wrong types, wrong number of parameters) and values outside the valid domain [-1, 1] during constant and override evaluation.

**Passing suites**:
- `integer_argument:*` - All tests pass (9/9)
- `parameters:*` - All tests pass (23/23)  
- `values:*` with f32 types - All tests pass (8/8)

**Failing tests**: 16 failures in `values:*` subcategory
- All failures involve `abstract-int`, `abstract-float`, or `f16` types at constant or override stages
- Tests expect shader compilation to fail when `asin()` is called with values outside [-1, 1]
- Currently these shaders compile successfully instead of failing

**Root cause**: 
Naga's `check_literal_value()` function (`/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs` lines 1321-1341) only validates NaN and Infinity for `F32` and `F64` literals, but not for `AbstractFloat` or `F16` literals. When `asin()` is evaluated with out-of-range values for these types, it produces NaN, but this NaN is not detected and the invalid literal is accepted.

**Fix needed**:
Extend the NaN and Infinity checks in `check_literal_value()` to cover `AbstractFloat` (f64) and `F16` (half::f16) types. The same issue likely affects `acos()` and potentially other math functions with domain restrictions.
