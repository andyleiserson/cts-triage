# getBindGroupLayout CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,getBindGroupLayout:*`

**Overall Status:** 4P/10F/0S (28.57%/71.43%/0%)

## Passing Sub-suites

- `unique_js_object,explicit_layout` - Tests that getBindGroupLayout returns unique JS objects (passes)
- `unique_js_object,auto_layout` - Tests that getBindGroupLayout returns unique JS objects (passes)
- `index_range,explicit_layout:index=0` - Getting bind group layout at index 0 (passes)
- `index_range,auto_layout:index=0` - Getting bind group layout at index 0 (passes)

## Remaining Issues

- `index_range,explicit_layout:index=1..5` - All fail with "Invalid group index N"
- `index_range,auto_layout:index=1..5` - All fail with "Invalid group index N"

## Issue Detail

### 1. getBindGroupLayout rejects valid indices beyond defined layouts

**Test selectors:**
- `webgpu:api,validation,getBindGroupLayout:index_range,explicit_layout:index=*`
- `webgpu:api,validation,getBindGroupLayout:index_range,auto_layout:index=*`

**What it tests:**
The test creates a pipeline with either:
- An explicit layout containing only 1 bind group layout (at index 0)
- An auto layout where the shader uses only `@group(0)`

Then it calls `getBindGroupLayout(index)` for indices 0-5 and expects:
- Indices 0, 1, 2, 3: No validation error (since `index < maxBindGroups` where `maxBindGroups=4`)
- Indices 4, 5: Validation error (since `index >= maxBindGroups`)

**Example failing test:**
```
webgpu:api,validation,getBindGroupLayout:index_range,explicit_layout:index=1
```

**Error:**
```
Unexpected validation error occurred: Invalid group index 1
```

**Root cause:**
The wgpu implementation at `/Users/Andy/Development/wgpu2/wgpu-core/src/device/global.rs` lines 1563-1567 validates that the requested index exists within the pipeline's actual bind group layouts array:

```rust
let id = match pipeline.layout.bind_group_layouts.get(index as usize) {
    Some(bg) => fid.assign(Fallible::Valid(bg.clone())),
    None => {
        break 'error binding_model::GetBindGroupLayoutError::InvalidGroupIndex(index)
    }
};
```

However, the WebGPU specification states:

1. Validation should only fail if `index >= device.limits.maxBindGroups`
2. The pipeline layout's `[[bindGroupLayouts]]` internal slot should have `maxBindGroups` entries (with `null` for unused indices), not just the user-specified ones

Per the spec at createPipelineLayout algorithm:
- `[[bindGroupLayouts]]` is initialized as a list of `null` GPUBindGroupLayouts with size equal to `maxBindGroups`
- Only non-empty user-specified layouts are filled in at their respective indices

This means wgpu's `PipelineLayout.bind_group_layouts` field stores only the explicitly defined layouts (1 entry when user provides 1 layout), while the spec expects it to conceptually have `maxBindGroups` entries.

**Fix needed:**
The `render_pipeline_get_bind_group_layout` and `compute_pipeline_get_bind_group_layout` functions need to be modified to:

1. Check if `index >= device.limits.max_bind_groups` and return `InvalidGroupIndex` error if so
2. For indices within bounds but beyond the defined layouts, return an empty bind group layout (or handle the `null` case per spec)

Alternatively, the `PipelineLayout.bind_group_layouts` field could be extended to always contain `max_bind_groups` entries with empty layouts for undefined indices, matching the spec's internal model more closely.

## Key Files Referenced

- **wgpu Implementation:** `/Users/Andy/Development/wgpu2/wgpu-core/src/device/global.rs` (lines 1545-1574 for render pipeline, 1680-1710 for compute pipeline)
- **Error Definition:** `/Users/Andy/Development/wgpu2/wgpu-core/src/binding_model.rs` (lines 1204-1218)
- **PipelineLayout Structure:** `/Users/Andy/Development/wgpu2/wgpu-core/src/binding_model.rs` (lines 863-870)
