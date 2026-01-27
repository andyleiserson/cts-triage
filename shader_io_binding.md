# Shader IO Binding Attribute CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,shader_io,binding:*`

Model: Claude Sonnet 4.5

**Overall Status:** 20P/1F/0S (95.24%/4.76%/0%)

## Passing Sub-suites ✅

- `binding:attr=*` (20/21 passing) - Most syntax variations of @binding attribute work correctly
- `binding_f16:*` (1/1 passing) - Correctly validates that @binding cannot use fp16 type suffix

## Remaining Issues ⚠️

**1 failing test:**

1. **Trailing comma syntax** (1 failure) - Parser limitation, part of broader issue #6394

## Issue Detail

### 1. Trailing Comma in @binding() Not Accepted

**Test selector:** `webgpu:shader,validation,shader_io,binding:binding:attr="trailing_comma"`

**What it tests:** Whether the WGSL parser accepts a trailing comma in the @binding attribute argument, which is valid according to the WGSL spec.

**Example failure:**
```
webgpu:shader,validation,shader_io,binding:binding:attr="trailing_comma"
```

**Shader code:**
```wgsl
@binding(1,) @group(1)
var<storage> a: i32;

@workgroup_size(1, 1, 1)
@compute fn main() {
  _ = a;
}
```

**Error:**
```
Unexpected validation error occurred:
Shader '' parsing error: expected `)`, found ","
  ┌─ wgsl:2:11
  │
2 │ @binding(1,) @group(1)
  │           ^ expected `)`
```

**Root cause:**
Naga's WGSL parser does not accept trailing commas in the @binding attribute. The parser expects exactly one expression followed immediately by a closing paren, without allowing for an optional trailing comma. This is a parser limitation that affects multiple WGSL attribute contexts.

**Fix needed:**
Update the @binding attribute parsing to optionally skip a trailing comma before expecting the closing paren, similar to how other attributes should handle trailing commas. This is part of a broader effort tracked in https://github.com/gfx-rs/wgpu/issues/6394 to support trailing commas throughout Naga's WGSL parser.

---

## Related Issues

- Issue #6394: Broader tracking issue for trailing comma support in Naga's WGSL parser
- The trailing comma issue affects multiple WGSL contexts including:
  - Attributes (@binding, @group, @location, @id, @workgroup_size, @interpolate, @builtin)
  - Template argument lists (vec, mat, array, ptr, atomic)
  - Function arguments

## Test Distribution

The test suite comprehensively covers:
- **Valid @binding syntax variations:**
  - Constant expressions: `@binding(z + y)` where z and y are const
  - Integer literals: `@binding(0)`, `@binding(1)`
  - Typed integer literals: `@binding(1i)`, `@binding(1u)`
  - Hexadecimal literals: `@binding(0x1)`
  - Comments: `@/* comment */binding(1)`
  - Line breaks: `@ \n binding(1)`
  - Trailing comma: `@binding(1,)` (currently fails)

- **Invalid @binding syntax (correctly rejected):**
  - Override expressions: `@binding(z)` where z is an override
  - Negative values: `@binding(-1)`
  - Missing value: `@binding()`
  - Missing parentheses: `@binding 1)`, `@binding(1`
  - Multiple values: `@binding(1,2)`
  - Float literals: `@binding(1.0)`, `@binding(1f)`
  - FP16 type suffix: `@binding(1h)`
  - No parameters: `@binding`
  - Misspelling: `@abinding(1)`
  - Duplicate attributes: `@binding(1) @binding(1)`

The test suite provides excellent coverage of both valid and invalid syntax patterns. Only the trailing comma case fails, which is a known parser limitation affecting many WGSL contexts.
