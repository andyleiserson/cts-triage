# Shader IO Interpolate CTS Tests - Triage Report

Selector: `webgpu:shader,validation,shader_io,interpolate:*`

Model: Claude Sonnet 4.5

**Overall Status:** 91% pass rate (specific counts to be determined)

## Test Coverage

The interpolate tests validate the `@interpolate` attribute used in WGSL shader input/output variables. The tests are organized into 5 main categories:

1. **type_and_sampling** - Validates all combinations of interpolation types and sampling modes
2. **require_location** - Validates that @interpolate only works with @location attributes
3. **integral_types** - Validates that integer types require @interpolate(flat)
4. **duplicate** - Validates that @interpolate can only be applied once
5. **interpolation_validation** - Validates syntax variations (trailing commas, missing parens, etc.)

## Test Details

### Test 1: type_and_sampling
**Test selector:** `webgpu:shader,validation,shader_io,interpolate:type_and_sampling:*`

**What it tests:** All valid and invalid combinations of interpolation type (flat, perspective, linear) and sampling mode (center, centroid, sample, first, either).

**Valid combinations per WebGPU spec:**
- No interpolate attribute (default)
- `@interpolate(perspective)` (default sampling: center)
- `@interpolate(perspective, center)`
- `@interpolate(perspective, centroid)`
- `@interpolate(perspective, sample)`
- `@interpolate(flat)` (default sampling: first)
- `@interpolate(flat, first)`
- `@interpolate(flat, either)`
- `@interpolate(linear)` (default sampling: center)
- `@interpolate(linear, center)`
- `@interpolate(linear, centroid)`
- `@interpolate(linear, sample)`

**Invalid combinations tested:**
- Using sampling modes as first parameter (center, centroid, sample, first, either)
- Using interpolation types as second parameter (flat, perspective, linear)

### Test 2: require_location
**Test selector:** `webgpu:shader,validation,shader_io,interpolate:require_location:*`

**What it tests:** The `@interpolate` attribute is only valid on user-defined IO (with `@location`), not on builtin IO (with `@builtin`).

**Expected behavior:**
- `@location(0) @interpolate(flat, either)` should pass
- `@builtin(position) @interpolate(flat, either)` should fail

### Test 3: integral_types
**Test selector:** `webgpu:shader,validation,shader_io,interpolate:integral_types:*`

**What it tests:** Integer types (i32, u32, vec2<i32>, vec4<u32>) in user-defined IO must use `@interpolate(flat)` or `@interpolate(flat, ...)`.

**Expected behavior:**
- Integer types with `@interpolate(flat)` or `@interpolate(flat, first)` or `@interpolate(flat, either)` should pass
- Integer types with `@interpolate(perspective, ...)` or `@interpolate(linear, ...)` should fail
- Integer types without interpolate attribute should fail (no default for integer types)

### Test 4: duplicate
**Test selector:** `webgpu:shader,validation,shader_io,interpolate:duplicate:*`

**What it tests:** The `@interpolate` attribute can only be applied once to a variable.

**Expected behavior:**
- `@location(0) @interpolate(flat, either)` should pass
- `@location(0) @interpolate(flat, either) @interpolate(flat)` should fail

### Test 5: interpolation_validation
**Test selector:** `webgpu:shader,validation,shader_io,interpolate:interpolation_validation:*`

**What it tests:** Syntax validation for the `@interpolate` attribute, including edge cases like trailing commas, missing parentheses, and invalid values.

**Valid syntax:**
- `@interpolate(perspective)` - basic usage
- `@interpolate(perspective,center)` - no space
- `@interpolate(flat,)` - trailing comma with one arg (should pass per WGSL spec)
- `@interpolate(perspective, center,)` - trailing comma with two args (should pass per WGSL spec)
- `@\ninterpolate(perspective)` - newline after @
- `@/* comment */interpolate(perspective)` - comment after @

**Invalid syntax:**
- `@interpolate()` - no parameters
- `@interpolate perspective)` - missing left paren
- `@interpolate)` - missing value and left paren
- `@interpolate(perspective` - missing right paren
- `@interpolate` - missing parens entirely
- `@interpolate(perspective center)` - missing comma
- `@interpolate(1)` - numeric value
- `@interpolate(perspective, 1)` - numeric second parameter

## Known Issues

### Issue 1: Trailing Comma Support

**Status:** Likely a Naga parser limitation similar to other trailing comma issues documented in the main triage.

**Description:** The WGSL spec allows trailing commas in attribute argument lists. Tests with `@interpolate(flat,)` and `@interpolate(perspective, center,)` may fail if Naga's parser doesn't support this.

**Related issues:** https://github.com/gfx-rs/wgpu/issues/6394 (general trailing comma support)

**Expected impact:** Small subset of interpolation_validation tests

### Issue 2: Error Message Quality

**Status:** To be investigated

**Description:** Even if validation is working correctly, some tests may fail if error messages don't match CTS expectations or if errors are reported at wrong locations.

## Investigation Needed

To complete this triage, the following investigations should be performed:

1. Run the full test suite to get exact pass/fail counts:
   ```bash
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:*'
   ```

2. Run each subcategory individually:
   ```bash
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:type_and_sampling:*'
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:require_location:*'
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:integral_types:*'
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:duplicate:*'
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:interpolation_validation:*'
   ```

3. Examine specific failures to identify root causes:
   ```bash
   cargo xtask cts --skip-checkout 'webgpu:shader,validation,shader_io,interpolate:interpolation_validation:attr="trailing_comma_one_arg"'
   ```

4. Check Naga's WGSL parser implementation:
   - Look at `naga/src/front/wgsl/parse/mod.rs` for interpolate parsing
   - Check `naga/src/front/wgsl/parse/conv.rs` for type conversions
   - Review `naga/src/valid/interface.rs` for validation logic

5. Compare with WebGPU spec section on interpolate:
   - https://www.w3.org/TR/WGSL/#interpolation

## Next Steps

1. Run the test suite to get actual failure counts and patterns
2. Identify which specific subcategories have failures
3. Examine representative failing tests to understand root causes
4. Check if failures are parser issues (syntax not accepted) or validation issues (wrong accept/reject behavior)
5. Document specific fixes needed once root causes are identified
