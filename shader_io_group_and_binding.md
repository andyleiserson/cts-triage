# Shader IO Group and Binding CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,shader_io,group_and_binding:*`

Model: Claude Sonnet 4.5

**Overall Status:** 445P/67F/0S (86.91%/13.09%/0.00%)

## Passing Sub-suites ✅

- `binding_attributes:*` (249/252 passing, 98.8%) - Correctly validates that @group and @binding must both be present
- `function_scope:*` (1/1 passing, 100%) - Correctly rejects @group/@binding on function-scope variables
- `function_scope_texture:*` (1/1 passing, 100%) - Correctly rejects @group/@binding on function-scope texture variables
- `private_function_scope:*` (1/1 passing, 100%) - Correctly rejects @group/@binding on function-scope private variables
- `private_module_scope:*` (1/1 passing, 100%) - Correctly rejects @group/@binding on module-scope private variables
- `different_entry_points:*` (144/192 passing, 75%) - Validates binding reuse between different entry points
- `single_entry_point:*` (48/64 passing, 75%) - Validates binding uniqueness within a single entry point

## Remaining Issues ⚠️

**All 67 failures (100%) are caused by `texture_external` not being implemented in wgpu/Naga.**

The failures break down as:
1. **binding_attributes** - 3 failures (all `texture_external`)
2. **different_entry_points** - 48 failures (all involving `texture_external`)
3. **single_entry_point** - 16 failures (all involving `texture_external`)

## Issue Detail

### 1. texture_external Not Implemented

**Test selectors:**
- `webgpu:shader,validation,shader_io,group_and_binding:binding_attributes:*;resource="texture_external"`
- `webgpu:shader,validation,shader_io,group_and_binding:different_entry_points:*;a_kind="texture_external";*`
- `webgpu:shader,validation,shader_io,group_and_binding:single_entry_point:*;a_kind="texture_external";*`

**What it tests:**
These tests validate @group and @binding attribute behavior with various resource types, including `texture_external`. The tests check:
- That both @group and @binding attributes must be present on resource variables
- That two resources in a single entry point cannot share the same (group, binding) pair
- That different entry points may reuse the same binding points if they don't share resources

**Example failure:**
```
webgpu:shader,validation,shader_io,group_and_binding:binding_attributes:stage="vertex";has_group=true;has_binding=true;resource="texture_external"
```

**Shader code:**
```wgsl
@group(0) @binding(0) var R : texture_external;
```

**Error:**
```
VALIDATION FAILED: Unexpected compilationInfo 'error' message.
0:0: error:
Shader validation error: Type [8] '' is invalid
 = Capability Capabilities(TEXTURE_EXTERNAL) is required


---- shader ----
@group(0) @binding(0) var R : texture_external;
```

**Root cause:**
The `texture_external` type is not implemented in wgpu/Naga. When any shader attempts to use `texture_external`, Naga's validator rejects it with a "Capability Capabilities(TEXTURE_EXTERNAL) is required" error. This prevents the actual validation logic from being tested.

The `texture_external` type is a WebGPU feature for importing external video frames or other platform-specific image sources. Implementing this requires:
- Naga type system support for `texture_external`
- Backend-specific handling for external texture bindings
- Proper capability tracking and validation

**Fix needed:**
This is a known missing feature that requires substantial implementation work beyond shader validation. The issue affects multiple CTS test suites, not just group_and_binding tests.

**Impact on test results:**
- `binding_attributes`: 3/252 failures (1.2%) - All 3 are `texture_external` related
- `different_entry_points`: 48/192 failures (25%) - All involve `texture_external` in parameter combinations
- `single_entry_point`: 16/64 failures (25%) - All involve `texture_external` in parameter combinations

The 25% failure rate in the latter two suites is because `texture_external` is one of 4 resource types tested in `kResourceKindsA`, and it's combined with multiple other parameters (stages, other resource types, usage patterns).

## Related Issues

This issue is already documented in:
- `docs/cts-triage/shader_functions_restrictions_triage.md` - texture_external causes ~15-17% of failures
- `docs/cts-triage/triage.md` - Listed as primary blocker for multiple test suites

## Test Coverage

The test suite provides comprehensive coverage of @group and @binding validation:

### binding_attributes (252 tests)
Tests that both @group and @binding must be present on resource variables:
- Covers all combinations of: 3 stages × 2 has_group states × 2 has_binding states × 21 resource types
- Resource types include: textures (1d, 2d, 3d, cube, arrays, multisampled), depth textures, storage textures, samplers, uniform/storage buffers, and texture_external
- Correctly validates that:
  - Resources with both @group and @binding are accepted
  - Resources missing either attribute are rejected
  - All non-texture_external resource types work correctly

### private_module_scope, private_function_scope, function_scope, function_scope_texture (4 tests)
Tests that @group and @binding cannot be applied to private or function-scope variables:
- All 4 tests pass, correctly rejecting invalid attribute placement

### single_entry_point (64 tests)
Tests that within a single entry point, two resources cannot share the same (group, binding) pair:
- Tests combinations of: 3 stages × 4 resource types A × 3 resource types B × 2 usage patterns × 2 groups × 2 bindings
- Usage patterns: direct (both resources used in entry point) and transitive (resources used via helper functions)
- Correctly validates binding conflicts within a single entry point
- 48/64 tests pass (all non-texture_external combinations work)

### different_entry_points (192 tests)
Tests that different entry points may safely reuse the same binding points:
- Tests combinations of: 3 stages A × 3 stages B × 4 resource types A × 3 resource types B × 2 usage patterns
- Verifies that binding reuse between different entry points is allowed
- 144/192 tests pass (all non-texture_external combinations work)

## Recommendations

1. **Do not attempt to fix texture_external issues** - This is a known missing feature requiring substantial implementation work
2. **Document as known limitation** - The fail.lst entry at line 190 should reference texture_external as the blocker
3. **Consider the test suite otherwise fully passing** - All validation logic for @group and @binding works correctly for implemented resource types
4. **Track texture_external implementation separately** - This affects multiple test suites and should be tracked as a cross-cutting concern

## Summary

The `webgpu:shader,validation,shader_io,group_and_binding:*` test suite has an 87% pass rate. All 67 failures (100%) are caused by the missing `texture_external` implementation. The actual @group and @binding validation logic in wgpu is working correctly for all implemented resource types:

- ✅ Correctly requires both @group and @binding to be present
- ✅ Correctly rejects @group/@binding on private and function-scope variables
- ✅ Correctly validates binding uniqueness within a single entry point
- ✅ Correctly allows binding reuse between different entry points
- ❌ Cannot test any scenario involving `texture_external` (known limitation)

Once `texture_external` is implemented in Naga, this test suite should reach 100% pass rate with no additional validation work needed.
