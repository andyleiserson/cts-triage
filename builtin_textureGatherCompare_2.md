# textureGatherCompare CTS Tests - Triage Report

**Test Selector:** `webgpu:shader,validation,expression,call,builtin,textureGatherCompare:*`

**Overall Status:** 619P/5F/0S (99.20% pass rate)

## Summary

The textureGatherCompare suite has a 99.20% pass rate with only 5 failures out of 624 tests. This is slightly better than the related textureGather suite (98.89% pass rate).

### Key Differences from textureGather

**textureGatherCompare has BETTER offset validation:**
- textureGather: Missing offset range validation entirely (accepts both -9 and 8)
- textureGatherCompare: Partial offset range validation (correctly rejects -9, but incorrectly accepts 8)

**textureGatherCompare does NOT have texture_type failures:**
- textureGather has 6 failures where component argument is incorrectly accepted with depth textures
- textureGatherCompare only works with depth textures (no component parameter), so this issue doesn't apply

## Failing Tests

### 1. Offset Upper Bound Validation Missing (4 failures)

**Test selector:** `webgpu:shader,validation,expression,call,builtin,textureGatherCompare:offset_argument:*`

**Failures:**
- `textureType="texture_depth_2d";offsetType="vec2<abstract-int>"`
- `textureType="texture_depth_2d";offsetType="vec2<i32>"`
- `textureType="texture_depth_2d_array";offsetType="vec2<abstract-int>"`
- `textureType="texture_depth_2d_array";offsetType="vec2<i32>"`

**What it tests:** Validates that the offset parameter values are within the required range [-8, 7].

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,textureGatherCompare:offset_argument:textureType="texture_depth_2d";offsetType="vec2<i32>"
```

**Test behavior:**
- Subcase value=-9: ✅ Correctly rejected (validation error)
- Subcase value=-8: ✅ Correctly accepted
- Subcase value=0: ✅ Correctly accepted
- Subcase value=7: ✅ Correctly accepted
- Subcase value=8: ❌ Incorrectly accepted (should fail validation)

**Error for value=8:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
@group(0) @binding(0) var s: sampler_comparison;
@group(0) @binding(1) var t: texture_depth_2d;
@fragment fn fs() -> @location(0) vec4f {
  let v = textureGatherCompare(t, s, vec2(0.0f, 0.0f), 0, vec2(i32(8), i32(8)));
  return vec4f(0);
}
```

**Root cause:**
Naga implements partial offset range validation for textureGatherCompare:
- Lower bound check exists and works correctly (rejects offset < -8)
- Upper bound check is missing (does not reject offset > 7)

This is BETTER than textureGather, which has no offset validation at all (both -9 and 8 are accepted).

**Fix needed:**
Add upper bound validation in Naga's textureGatherCompare implementation to reject offset values > 7.

**Code location:** Likely in `naga/src/valid/expression.rs` where texture builtin validation occurs.

### 2. Must-use Validation Missing (1 failure)

**Test selector:** `webgpu:shader,validation,expression,call,builtin,textureGatherCompare:must_use:use=false`

**What it tests:** WGSL requires that builtin function results marked with `@must_use` must have their return values used.

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,textureGatherCompare:must_use:use=false
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
@group(0) @binding(0) var t : texture_depth_2d;
@group(0) @binding(1) var s : sampler_comparison;
fn foo() {
   textureGatherCompare(t, s, vec2(0,0), 0);
}
```

**Root cause:**
This is the broader @must_use validation infrastructure issue tracked in https://github.com/gfx-rs/wgpu/issues/8876. The function is called as a statement without using its result, which should fail validation.

**Related issue:** #8876

## Comparison with textureGather

| Issue | textureGather | textureGatherCompare |
|-------|---------------|---------------------|
| Offset validation | ❌ Completely missing (both bounds) | ⚠️ Partial (lower bound only) |
| Component arg with depth textures | ❌ Incorrectly accepts (6 failures) | ✅ N/A (no component param) |
| Must-use validation | ❌ Missing (#8876) | ❌ Missing (#8876) |
| Overall pass rate | 98.89% | 99.20% |

## Suggested Bug Reference for fail.lst

```
webgpu:shader,validation,expression,call,builtin,textureGatherCompare:* // 99%, offset upper bound validation missing
```

Note: The existing reference attributes all failures to #8876, but the primary issue is actually the missing offset upper bound validation (4/5 failures).
