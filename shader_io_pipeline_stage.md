# Pipeline Stage CTS Tests - Triage Report

**Overall Status:** 67P/6F (91.78% pass, 8.22% fail)

## Passing Sub-suites ✅

All of these sub-suites pass with 100% pass rate:

- `vertex_parsing` - 8/8 pass (100%)
- `fragment_parsing` - 8/8 pass (100%)
- `compute_parsing` - 8/8 pass (100%)
- `multiple_entry_points` - 1/1 pass (100%)
- `extra_on_compute_function` - 8/8 pass (100%)
- `extra_on_fragment_function` - 8/8 pass (100%)
- `extra_on_vertex_function` - 8/8 pass (100%)

## Remaining Issues ⚠️

- `placement` - 18/24 pass (75% pass, 6 failures)
  - Stage attributes (`@vertex`, `@fragment`, `@compute`) are incorrectly accepted on `var<private>` and `var<storage>` declarations

## Issue Detail

### 1. Stage Attributes on Variable Declarations

**Test selector:** `webgpu:shader,validation,shader_io,pipeline_stage:placement:*`

**What it tests:** Validates that stage attributes (`@vertex`, `@fragment`, `@compute`) are only allowed on function declarations, not on variables, struct members, function parameters, function variables, function return types, or statements.

**Failing cases:**
- `scope="private-var";attr="@vertex"`
- `scope="private-var";attr="@fragment"`
- `scope="private-var";attr="@compute"`
- `scope="storage-var";attr="@vertex"`
- `scope="storage-var";attr="@fragment"`
- `scope="storage-var";attr="@compute"`

**Example failure:**
```
webgpu:shader,validation,shader_io,pipeline_stage:placement:scope="private-var";attr="%40vertex"
```

**Error:**
```
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----

      @vertex
      var<private> priv_var : i32;

      ...
```

The test expects this to be rejected (stage attribute on a variable), but Naga accepts it without error.

**Root cause:**

In `naga/src/front/wgsl/parse/mod.rs`, the `global_decl()` function:

1. Parses all attributes including `@vertex`, `@fragment`, `@compute` and stores them in the `stage` variable (lines 2977-2986)
2. When it encounters a `var` declaration (line 3156), it creates a `GlobalDeclKind::Var` (line 3162)
3. The `stage` attribute is **only validated and used** for function declarations (lines 3179-3193)
4. For `var`, `struct`, `alias`, `const`, `override`, and `const_assert` declarations, the `stage` attribute is silently ignored if present

**Passing cases (correctly rejected):**
- `fn-param` - stage attribute on function parameter (correctly rejected)
- `fn-var` - stage attribute on function-local variable (correctly rejected)
- `fn-return` - stage attribute on function return type (correctly rejected)
- `struct-member` - stage attribute on struct member (correctly rejected)
- `while-stmt` - stage attribute on while statement (correctly rejected)

**Fix needed:**

After parsing attributes in `global_decl()`, before processing each declaration type, add validation that `stage` is `None` for non-function declarations. The fix should:

1. After line 3067 (after attribute parsing is complete), check if `stage.value.is_some()`
2. When parsing `var`, `struct`, `alias`, `const`, `override`, or `const_assert`, ensure `stage` is `None`
3. Return an appropriate error if a stage attribute is present on a non-function declaration
4. The error should indicate that stage attributes are only valid on function declarations

Example locations where validation should occur:
- Line 3084: `struct` declaration
- Line 3097: `alias` declaration
- Line 3107: `const` declaration
- Line 3130: `override` declaration
- Line 3156: `var` declaration
- Line 3198: `const_assert` declaration
