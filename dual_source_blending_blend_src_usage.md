# Dual Source Blending `blend_src_usage` CTS Tests - Triage Report

**Test Selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*`

**Overall Status:** 14P/9F (60.87% pass rate, 39.13% failure rate)

## Summary

The failing tests expose missing WGSL validation for the `@blend_src` attribute. Currently, wgpu only validates `@blend_src` usage when a struct is used as a fragment shader output. However, the WGSL specification requires validation at struct definition time, regardless of whether the struct is actually used.

## Passing Tests ✅

The following test cases pass correctly:
- `const`, `override`, `let`, `var_private`, `var_function`, `function_declaration` - Correctly reject `@blend_src` on non-struct-member declarations
- `non_entrypoint_function_input_non_struct`, `non_entrypoint_function_output_non_struct` - Correctly reject `@blend_src` on non-struct function parameters
- `entrypoint_input_non_struct` - Correctly rejects `@blend_src` on fragment input
- `struct_member_no_location_no_blend_src` - Correctly accepts struct with proper blend_src and additional member without location
- `struct_member_blend_src_and_builtin` - Correctly accepts struct with blend_src and builtin member
- `struct_member_location_0_blend_src_0_blend_src_1` - Correctly accepts valid blend_src usage (the canonical valid case)

## Failing Tests ⚠️

### 1. entrypoint_output_non_struct
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="entrypoint_output_non_struct"`

**What it tests:** `@blend_src` cannot be used on non-struct entry point return values

**Example failure:**
```wgsl
enable dual_source_blending;

@fragment fn main() -> @location(0) @blend_src(0) vec4f {
  return vec4f();
}
```

**Error:** Missing expected compilation error

**Root cause:** The validation in naga/src/valid/interface.rs only checks that `@blend_src` is used on output (not input), but doesn't verify that the output is a struct type. The check at interface.rs:487-492 validates output vs input, but doesn't enforce that it must be used on a struct member.

**Fix needed:** Add validation that `@blend_src` can only appear on struct members, not on direct entry point parameters or return values.

### 2. struct_member_only_blend_src_0
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_only_blend_src_0"`

**What it tests:** A struct with only `@blend_src(0)` but not `@blend_src(1)` is invalid

**Example failure:**
```wgsl
enable dual_source_blending;

struct BlendSrcStruct {
  @location(0) @blend_src(0) color : vec4f,
}

@fragment fn main() -> @location(0) vec4f {
  return vec4f();
}
```

**Error:** Missing expected compilation error

**Root cause:** Validation in interface.rs:608-625 only runs when the struct is actually used as an entry point output. In this test, the struct is defined but never used (main() returns a non-struct), so the validation never runs.

**Fix needed:** Add struct-level validation in naga/src/valid/type.rs in the `validate_type` function for `TypeInner::Struct`. When any struct member has `@blend_src`, validate that:
- Exactly 2 members have `@location` attributes
- One is `@location(0) @blend_src(0)`
- One is `@location(0) @blend_src(1)`

### 3. struct_member_only_blend_src_1
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_only_blend_src_1"`

**What it tests:** A struct with only `@blend_src(1)` but not `@blend_src(0)` is invalid

**Root cause:** Same as #2 - missing struct-level validation

### 4. struct_member_duplicate_blend_src_0
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_duplicate_blend_src_0"`

**What it tests:** A struct cannot have two members with the same `@blend_src` index

**Example failure:**
```wgsl
struct BlendSrcStruct {
  @location(0) @blend_src(0) color : vec4f,
  @location(0) @blend_src(0) blend : vec4f,
}
```

**Error:** Missing expected compilation error

**Root cause:** Same as #2 - validation only runs when struct is used as entry point output. The validation at interface.rs:499-501 does check for duplicates, but only for used structs.

**Fix needed:** Add duplicate check in struct-level validation.

### 5. struct_member_duplicate_blend_src_1
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_duplicate_blend_src_1"`

**What it tests:** Same as #4 but for `@blend_src(1)`

**Root cause:** Same as #4

### 6. struct_member_has_non_zero_location_blend_src_0
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_has_non_zero_location_blend_src_0"`

**What it tests:** `@blend_src` can only be used with `@location(0)`, not other locations

**Example failure:**
```wgsl
struct BlendSrcStruct {
  @location(0) @blend_src(0) color1 : vec4f,
  @location(1) @blend_src(0) color2 : vec4f,
  @location(0) @blend_src(1) blend : vec4f,
}
```

**Error:** Missing expected compilation error

**Root cause:** Same as #2 - missing struct-level validation. The check at interface.rs:493-498 validates location is 0, but only for used structs.

**Fix needed:** Add location validation in struct-level validation.

### 7. struct_member_has_non_zero_location_blend_src_1
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_has_non_zero_location_blend_src_1"`

**What it tests:** Same as #6 but for `@blend_src(1)`

**Root cause:** Same as #6

### 8. struct_member_non_zero_location_blend_src_0_blend_src_1
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_non_zero_location_blend_src_0_blend_src_1"`

**What it tests:** Both blend_src attributes must be on location 0

**Example failure:**
```wgsl
struct BlendSrcStruct {
  @location(1) @blend_src(0) color : vec4f,
  @location(1) @blend_src(1) blend : vec4f,
}
```

**Root cause:** Same as #6

### 9. struct_member_has_non_zero_location_no_blend_src
**Test selector:** `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:attr="struct_member_has_non_zero_location_no_blend_src"`

**What it tests:** If a struct uses `@blend_src`, ALL members with `@location` must have `@blend_src` and be at location 0

**Example failure:**
```wgsl
struct BlendSrcStruct {
  @location(0) @blend_src(0) color : vec4f,
  @location(0) @blend_src(1) blend : vec4f,
  @location(1) color2 : vec4f,
}
```

**Error:** Missing expected compilation error

**Root cause:** Same as #2 - this rule is not validated at struct definition time

**Fix needed:** When struct has any `@blend_src`, validate that all members with `@location` are at location 0 and have `@blend_src`.

## Required Changes

### 1. Add Struct-Level Validation (Primary Fix)

**Location:** `naga/src/valid/type.rs` in the `validate_type` function

**What to add:** In the `Ti::Struct` case (around line 603), after the existing member validation loop, add validation for `@blend_src` usage:

1. Iterate through all struct members and collect those with `@blend_src` attributes
2. If any member has `@blend_src`, validate:
   - Exactly 2 members have `@location` attributes
   - Both must be at `@location(0)`
   - One must have `@blend_src(0)` and the other `@blend_src(1)`
   - No duplicate `@blend_src` indices
   - All members with `@location` must have `@blend_src`

**Error types needed:** May need to add new error variants to the `TypeError` enum in naga/src/valid/type.rs for:
- Incomplete blend_src usage (only one of the two required)
- Invalid location for blend_src (not location 0)
- Duplicate blend_src index
- Location member without blend_src when struct uses blend_src

### 2. Add Non-Struct Binding Validation

**Location:** `naga/src/valid/interface.rs` in the `VaryingContext::validate` method

**What to add:** Around line 470 where blend_src validation starts, add a check to ensure the binding is part of a struct member validation, not a direct entry point parameter/return. This requires tracking whether we're validating a struct member or a direct binding.

### 3. Update Entry Point Validation

**Location:** `naga/src/valid/interface.rs`

**What to verify:** The existing validation at lines 608-625 should remain as a redundant check when structs are actually used, but the primary validation should happen at struct definition time. This provides defense-in-depth.

## Implementation Complexity

**Estimated complexity:** Medium

**Key challenges:**
1. The struct validation in type.rs runs early, before we know how the struct will be used, which is correct for this spec requirement
2. Need to carefully handle the case where a struct has members with no IO binding (like fields with no attributes, or `@builtin` fields)
3. The validation logic already exists in interface.rs but needs to be adapted for struct definition time
4. Need to ensure the validation doesn't break valid uses like `struct_member_no_location_no_blend_src` (struct with blend_src and additional non-location member)

## Testing Strategy

After implementing fixes:

1. Run the full test suite to verify all failures are resolved:
   ```bash
   cargo xtask cts 'webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*'
   ```

2. Verify all passing tests still pass (don't break existing functionality)

3. Run related dual source blending tests to ensure no regressions:
   ```bash
   cargo xtask cts 'webgpu:shader,validation,extension,dual_source_blending:*'
   ```

4. Consider adding unit tests in naga/src/valid/type.rs to test struct validation directly

## WebGPU Spec Reference

The validation rules being tested come from the WGSL specification for the `dual_source_blending` extension:
https://www.w3.org/TR/WGSL/#extension-dual_source_blending

Key spec requirements:
- `@blend_src` can only appear on members of structures
- If used, both `@blend_src(0)` and `@blend_src(1)` must be present
- Both must be at `@location(0)`
- Both must have the same type
- Can only be used in fragment shader outputs
