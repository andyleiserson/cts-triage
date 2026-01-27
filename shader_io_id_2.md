# Shader IO ID Attribute CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,shader_io,id:*`

Model: Claude Sonnet 4.5

**Overall Status:** 30P/2F/0S (93.75%/6.25%/0%)

## Passing Sub-suites ✅

- `id:attr=*` (21/22 passing) - Most syntax variations of @id attribute work correctly
- `id_fp16:*` (2/2 passing) - Correctly validates that @id cannot use fp16 type suffix
- `id_struct_member:*` (3/3 passing) - Correctly rejects @id on struct members
- `id_non_override:type="var"` and `type="override"` (2/3 passing) - Correctly handles @id on var (rejected) and override (accepted)
- `id_in_function:*` (2/2 passing) - Correctly rejects @id on local variables in functions

## Remaining Issues ⚠️

**2 failing tests across 2 distinct issues:**

1. **Trailing comma syntax** (1 failure) - Parser limitation, part of broader issue #6394
2. **@id accepted on const declarations** (1 failure) - Validation gap, @id should only be valid on override declarations

## Issue Detail

### 1. Trailing Comma in @id() Not Accepted

**Test selector:** `webgpu:shader,validation,shader_io,id:id:attr="trailing_comma"`

**What it tests:** Whether the WGSL parser accepts a trailing comma in the @id attribute argument, which is valid according to the WGSL spec.

**Example failure:**
```
webgpu:shader,validation,shader_io,id:id:attr="trailing_comma"
```

**Shader code:**
```wgsl
@id(1,)
override a = 4;

@workgroup_size(1, 1, 1)
@compute fn main() {}
```

**Error:**
```
Unexpected validation error occurred:
Shader '' parsing error: expected `)`, found ","
  ┌─ wgsl:2:6
  │
2 │ @id(1,)
  │      ^ expected `)`
```

**Root cause:**
Naga's WGSL parser does not accept trailing commas in the @id attribute. The parser code at `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs:3015-3018` parses the @id attribute as:

```rust
"id" => {
    lexer.expect(Token::Paren('('))?;
    id.set(self.general_expression(lexer, &mut ctx)?, name_span)?;
    lexer.expect(Token::Paren(')'))?;
}
```

After parsing the expression, it immediately expects a closing paren without allowing for an optional trailing comma. This is a parser limitation that affects multiple WGSL contexts.

**Fix needed:**
Update the @id attribute parsing to optionally skip a trailing comma before expecting the closing paren, similar to how other attributes handle trailing commas. This is part of a broader effort tracked in https://github.com/gfx-rs/wgpu/issues/6394 to support trailing commas throughout Naga's WGSL parser.

---

### 2. @id Attribute Accepted on Const Declarations

**Test selector:** `webgpu:shader,validation,shader_io,id:id_non_override:type="const"`

**What it tests:** Whether the @id attribute is correctly rejected when applied to const declarations. According to the WGSL spec, @id is only valid on override declarations.

**Example failure:**
```
webgpu:shader,validation,shader_io,id:id_non_override:type="const"
```

**Shader code:**
```wgsl
@id(1) const a = 4;

@workgroup_size(1, 1, 1)
@compute fn main() {}
```

**Error:**
```
EXPECTATION FAILED: Expected validation error

VALIDATION FAILED: Missing expected compilationInfo 'error' message.
```

**Root cause:**
Naga's parser collects the @id attribute value but only uses it when creating override declarations. However, it does not validate that if an @id attribute was specified, the declaration must be an override.

In `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs`:
- Line 2967: `id` attribute is collected as a `ParsedAttribute`
- Lines 3188-3212: When parsing an `override` declaration, the `id.value` is used
- Lines 3165-3186: When parsing a `const` declaration, the `id` attribute is simply ignored

There is no check that validates: "If `id.value` is `Some(_)` and we're not parsing an override, emit an error."

**Fix needed:**
Add validation after determining the declaration type to ensure that if an @id attribute was specified, the declaration must be an override. This could be done by:

1. After parsing each declaration type (const, var, override), check if `id.value.is_some()`
2. For const and var declarations, if `id.value` is set, return an error indicating "@id attribute is only valid on override declarations"
3. Only override declarations should be allowed to proceed when `id.value` is set

The fix should be implemented in the `global_decl` function after each declaration type is determined, similar to how other attribute validations are performed in that function.

---

## Related Issues

- Issue #6394: Broader tracking issue for trailing comma support in Naga's WGSL parser
- The trailing comma issue affects multiple WGSL contexts including attributes, template argument lists, and function arguments

## Test Distribution

The test suite comprehensively covers:
- Valid @id syntax variations (numeric literals, hex, different suffixes)
- Invalid @id syntax (wrong parameter types, out of range values, malformed syntax)
- @id placement validation (only on overrides, not on const/var/struct members/local variables)
- Edge cases (trailing commas, fp16 suffixes, duplicate attributes)
