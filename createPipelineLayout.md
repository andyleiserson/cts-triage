# createPipelineLayout CTS Tests - Triage Report

**Overall Status:** 2P/4F (33%/67%)

## Passing Sub-suites ✅
- `number_of_bind_group_layouts_exceeds_the_maximum_value` - Properly validates max bind group layouts
- `bind_group_layouts,device_mismatch` - Properly validates device mismatch

## Remaining Issues ⚠️
- `number_of_dynamic_buffers_exceeds_the_maximum_value` - Not validating dynamic buffer count limits for storage buffers in fragment stage (4 failures)
- `bind_group_layouts,null_bind_group_layouts` - TypeError when passing null/undefined bind group layouts
- `bind_group_layouts,create_pipeline_with_null_bind_group_layouts` - Same TypeError issue
- `bind_group_layouts,set_pipeline_with_null_bind_group_layouts` - Same TypeError issue

## Issue Detail

### 1. Dynamic Buffer Count Validation Gap
**Test selector:** `webgpu:api,validation,createPipelineLayout:number_of_dynamic_buffers_exceeds_the_maximum_value:*`
**What it tests:** Validates that creating a pipeline layout exceeding the maximum number of dynamic buffers fails

**Example failures:**
```
visibility=2;type="storage" (FRAGMENT stage, storage buffer)
visibility=2;type="read-only-storage" (FRAGMENT stage, read-only storage)
visibility=6;type="storage" (VERTEX|FRAGMENT, storage buffer)
visibility=6;type="read-only-storage" (VERTEX|FRAGMENT, read-only storage)
```

**Error:**
```
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Root cause:**
wgpu is not validating dynamic buffer count limits for storage buffers when the FRAGMENT shader stage is involved (visibility=2 or visibility=6). The test creates bind group layouts that exceed the per-stage dynamic buffer limits, but wgpu accepts them.

**Investigation findings:**
The validation code in `wgpu-core/src/binding_model.rs` (lines 535-542) appears correct. It validates that:
- `dynamic_uniform_buffers` doesn't exceed `max_dynamic_uniform_buffers_per_pipeline_layout`
- `dynamic_storage_buffers` doesn't exceed `max_dynamic_storage_buffers_per_pipeline_layout`

The counters are accumulated correctly when merging bind group layouts (line 512).

**Possible causes:**
1. Limits mismatch: The device may be reporting different limits than the test expects
2. Downlevel capabilities: Even with "AllFeaturesMaxLimits", the device may use downlevel limits for certain features
3. Backend-specific behavior: Metal/D3D11/etc. may have platform-specific limit handling
4. Test configuration issue: The test fixture may not be properly requesting max limits

**Fix needed:**
Further investigation with debug logging to determine:
- What limits the device is actually reporting (check `device.limits.maxDynamicStorageBuffersPerPipelineLayout`)
- What count is being validated (add logging to the validation code)
- Whether the device is using downlevel vs. full limits
- Whether there's a backend-specific limit override happening

This requires adding debug output to both deno_webgpu (to see reported limits) and wgpu-core validation (to see actual counts being validated).

### 2. Null/Undefined Bind Group Layouts - WebIDL Issue
**Test selectors:**
- `webgpu:api,validation,createPipelineLayout:bind_group_layouts,null_bind_group_layouts:*`
- `webgpu:api,validation,createPipelineLayout:bind_group_layouts,create_pipeline_with_null_bind_group_layouts:*`
- `webgpu:api,validation,createPipelineLayout:bind_group_layouts,set_pipeline_with_null_bind_group_layouts:*`

**What they test:** According to the WebGPU spec, it's valid to pass null or undefined bind group layouts in the bindGroupLayouts array when creating a pipeline layout. These represent empty/unused bind group slots.

**Example failure:**
```
bindGroupLayouts=["NonEmpty","Null","NonEmpty"]
bindGroupLayouts=["NonEmpty","Undefined","NonEmpty"]
```

**Error:**
```
EXCEPTION: TypeError: Failed to execute 'call' on 'GPUDevice': 'bindGroupLayouts' of
'GPUPipelineLayoutDescriptor' (Argument 0), index X can not be converted to a GPUBindGroupLayout
```

**Root cause:**
The deno_webgpu WebIDL bindings don't allow null/undefined entries in the bindGroupLayouts sequence. The WebIDL converter is throwing a TypeError during conversion, before wgpu-core validation is reached. The test expects `kShouldError = false`, meaning these should be accepted as valid.

**Fix needed:**
Update the WebIDL definition in `deno_webgpu` to allow nullable bind group layout entries. The `bindGroupLayouts` field in the pipeline layout descriptor should accept `(GPUBindGroupLayout | null | undefined)[]`. This likely requires changes to how the sequence is defined and converted in the deno bindings.

## Summary

Two distinct issues:
1. **Validation gap in wgpu-core:** Dynamic buffer count limits not enforced for storage buffers in fragment stage
2. **deno_webgpu WebIDL issue:** Nullable bind group layouts not supported in JavaScript bindings

The second issue affects 3 test suites but is a single root cause (WebIDL conversion). The first issue is a genuine validation gap in wgpu-core.
