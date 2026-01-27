Perfect! Now let me create a summary of my findings for the user.

## Summary

I've successfully investigated the CTS test selector `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*` and documented my findings.

### Test Results

**Overall Status:** 0P/2F/0S (0% pass rate, 100% failure rate)

Both test variants fail:
- `isAsync=true` - Async pipeline creation fails with "DID NOT REJECT"
- `isAsync=false` - Sync pipeline creation fails with "Validation succeeded unexpectedly"

### Root Cause

**wgpu is completely missing validation for workgroup storage size limits.** The test validates that when pipeline constants (overrides) are used to size workgroup storage arrays, wgpu should reject pipelines that exceed `device.limits.maxComputeWorkgroupStorageSize`.

The test creates shaders like:
```wgsl
override a: u32;
override b: u32;
var<workgroup> vec4_data: array<vec4<f32>, a>;
var<workgroup> mat4_data: array<mat4x4<f32>, b>;
```

Then tries to create pipelines with array sizes that exceed the limit. wgpu currently accepts these invalid configurations without validation.

### Why This Is Complex

This is a **HIGH COMPLEXITY** validation gap because:

1. **Timing Issue**: Workgroup storage size depends on pipeline constants that are only provided at pipeline creation time, not shader module creation time.

2. **Requires Constant Evaluation**: We need to evaluate override expressions with the provided constant values to determine final array sizes.

3. **Type Size Calculation**: Must calculate sizes for complex WGSL types including arrays, structs, vectors, matrices with proper alignment and padding.

4. **No Existing Infrastructure**: There's no current mechanism in wgpu-core to calculate workgroup storage size, with or without overrides.

### Documentation

I've created a comprehensive triage document at:
**`/Users/Andy/Development/wgpu2/docs/cts-triage/compute_pipeline_overrides_workgroup_storage_size_triage.md`**

This document includes:
- Detailed explanation of what the test validates
- Complete root cause analysis
- Technical details on why this is complex to fix
- Proposed fix implementation approach
- References to all relevant source files and line numbers

The fix would require adding new validation logic in `create_compute_pipeline` to calculate total workgroup storage usage after applying pipeline constants, then checking against the device limit.
