# Shader IO Workgroup Size CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,shader_io,workgroup_size:*`

Model: Claude Sonnet 4.5

**Overall Status:** 48P/10F/0S (82.76%/17.24%/0%)

## Passing Sub-suites ✅

The following test categories have no failures:
- `workgroup_size_fragment_shader` - Correctly rejects @workgroup_size on fragment shaders
- `workgroup_size_vertex_shader` - Correctly rejects @workgroup_size on vertex shaders
- `workgroup_size_fp16` - Correctly rejects f16 types in workgroup_size
- Most `workgroup_size:attr=*` tests (48 out of 58 pass)

Correctly passing tests include:
- Valid workgroup size declarations with abstract int, signed (i32), unsigned (u32), hex literals
- Const expressions in workgroup_size
- Override expressions with defaults and in pipelines
- Mixed abstract and concrete types (e.g., `@workgroup_size(8, 8i)` is valid)
- Rejection of float types, zero values, negative values
- Rejection of empty parameters, missing parentheses, misspellings
- Rejection of duplicate @workgroup_size attributes
- Multi-line attributes and attributes with comments

## Remaining Issues ⚠️

**10 failing tests** across 3 categories:
1. **Trailing comma syntax** (3 failures) - Parser issue
2. **Type mixing validation** (4 failures) - Validation gap for mixing signed/unsigned
3. **Attribute placement validation** (3 failures) - @workgroup_size accepted on non-compute-entry-points

## Issue Detail

### 1. Trailing Comma Syntax Not Supported

**Test selector:** `webgpu:shader,validation,shader_io,workgroup_size:workgroup_size:attr="trailing_comma_*"`

**What it tests:** Whether the parser accepts trailing commas in @workgroup_size attribute arguments, which is valid according to WGSL spec.

**Failing tests:**
- `attr="trailing_comma_x"` - `@workgroup_size(8, )`
- `attr="trailing_comma_y"` - `@workgroup_size(8, 8,)`
- `attr="trailing_comma_z"` - `@workgroup_size(8, 8, 8,)`

**Example failure:**
```
webgpu:shader,validation,shader_io,workgroup_size:workgroup_size:attr="trailing_comma_x"
```

**Error:**
```
Shader '' parsing error: expected expression, found ")"
  ┌─ wgsl:1:21
  │
1 │  @workgroup_size(8, )
  │                     ^ expected expression
```

**Root cause:**
Naga's parser does not support trailing commas in @workgroup_size arguments. The WGSL spec allows trailing commas in many contexts, but Naga's parser implementation for attribute arguments requires expressions at each position and doesn't handle the trailing comma case.

**Fix needed:**
Update Naga's parser to accept trailing commas in attribute argument lists, consistent with other comma-separated lists in WGSL. This is part of a known issue pattern affecting multiple contexts (template argument lists, attributes, etc.). See https://github.com/gfx-rs/wgpu/issues/6394 for the general trailing comma support issue.

---

### 2. Type Mixing Validation Gap (Signed/Unsigned)

**Test selector:** `webgpu:shader,validation,shader_io,workgroup_size:workgroup_size:attr="mixed_signed_unsigned"` and `attr="mix_u*"`

**What it tests:** Validation that all arguments to @workgroup_size must have the same concrete type. Mixing i32 and u32 should be rejected, even though mixing abstract int with either is allowed.

**Failing tests:**
- `attr="mixed_signed_unsigned"` - `@workgroup_size(8i, 8i, 8u)`
- `attr="mix_ux"` - `@workgroup_size(1u, 1i, 1i)`
- `attr="mix_uy"` - `@workgroup_size(1i, 1u, 1i)`
- `attr="mix_uz"` - `@workgroup_size(1i, 1i, 1u)`

**Example failure:**
```
webgpu:shader,validation,shader_io,workgroup_size:workgroup_size:attr="mixed_signed_unsigned"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
 @workgroup_size(8i, 8i, 8u)
      @compute fn main() {}
```

**Root cause:**
Naga is accepting workgroup_size declarations with mixed concrete integer types (i32 and u32). According to WGSL spec section on workgroup_size, all arguments must resolve to the same concrete type. While abstract int can mix with either i32 or u32 (causing the abstract int to convert), two different concrete types should not be allowed.

**Fix needed:**
Add validation in Naga to ensure all @workgroup_size arguments have the same concrete type after abstract-to-concrete conversion. This validation should:
1. Allow abstract int to mix with i32 or u32 (abstract converts to the concrete type)
2. Allow all abstract ints (they all convert to either i32 or u32 consistently)
3. Reject mixing i32 and u32 concrete types

This is likely a validation check needed during attribute processing in Naga's front-end validation.

---

### 3. Attribute Placement Validation Gap

**Test selector:** Tests for @workgroup_size on non-compute-entry-points

**What it tests:** Validation that @workgroup_size can only be applied to compute shader entry points, not regular functions, const declarations, or var declarations.

**Failing tests:**
- `workgroup_size_function` - @workgroup_size on a regular function
- `workgroup_size_const` - @workgroup_size on a const declaration
- `workgroup_size_var` - @workgroup_size on a var declaration

**Example failure:**
```
webgpu:shader,validation,shader_io,workgroup_size:workgroup_size_function:
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----

@workgroup_size(1)
fn my_func() {}
```

**Root cause:**
Naga is accepting @workgroup_size attribute on declarations where it should not be allowed. The attribute is only valid on functions with the @compute attribute. The validation that checks attribute applicability is not catching these misuses.

**Detailed failure examples:**

1. **On regular function:**
   ```wgsl
   @workgroup_size(1)
   fn my_func() {}
   ```
   Should be rejected, but Naga accepts it.

2. **On const declaration:**
   ```wgsl
   @workgroup_size(1)
   const a : i32 = 4;
   ```
   Should be rejected, but Naga accepts it.

3. **On var declaration:**
   ```wgsl
   @workgroup_size(1)
   @group(1) @binding(1)
   var<storage> a: i32;
   ```
   Should be rejected, but Naga accepts it.

**Fix needed:**
Add validation in Naga to ensure @workgroup_size is only applied to functions with @compute stage attribute. This should be part of attribute validation that checks:
1. The attribute is applied to a function declaration
2. The function has the @compute stage attribute
3. Reject the attribute on any other declaration type (const, var, override, struct, etc.)

The validation likely needs to be added in Naga's front-end during attribute processing and entry point validation.

---

## Notes on Passing Tests

The implementation correctly handles many edge cases:
- **Fragment/vertex shader rejection:** Tests `workgroup_size_fragment_shader` and `workgroup_size_vertex_shader` correctly fail when @workgroup_size is applied to @fragment or @vertex entry points
- **Type validation:** Correctly rejects float types, float literals
- **Value validation:** Correctly rejects zero and negative values
- **Syntax validation:** Correctly rejects empty parameters, missing parentheses, misspellings
- **Semantic validation:** Correctly rejects duplicate @workgroup_size attributes
- **Override handling:** Correctly handles pipeline overrides with and without defaults
- **Expression support:** Correctly supports const expressions

## Related Issues

The trailing comma issue is part of a broader pattern affecting multiple WGSL syntax elements. See general triage documentation about trailing comma support in Naga parser.
