# CreateBindGroupLayout VERTEX Storage CTS Tests - Triage Report

CTS selectors:
- `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_buffer_type:*`
- `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_storage_texture_access:*`

Model: Opus

**Overall Status:** 4P/12F/0S (25%/75%/0%)

## Summary

These tests validate TWO separate WebGPU requirements for the VERTEX shader stage:

1. **Writable storage is prohibited** - `type="storage"` buffers and writable storage textures must not be visible to VERTEX
2. **Read-only storage depends on per-stage limits** - `type="read-only-storage"` buffers and `access="read-only"` storage textures are allowed only if `maxStorageBuffersInVertexStage` / `maxStorageTexturesInVertexStage` > 0

**wgpu correctly enforces requirement #1** - writable storage in VERTEX is rejected.

**wgpu fails requirement #2** - it doesn't implement the per-stage limits and accepts read-only storage regardless.

## Test Results Detail

### Buffer Type Test (`VERTEX_shader_stage_buffer_type`)

For `shaderStage=1` (VERTEX only):
| Subcase | Expected | wgpu Behavior | Result |
|---------|----------|---------------|--------|
| `type="uniform"` | Accept | Accept | ✅ Pass |
| `type="storage"` | Reject (writable prohibited) | Reject | ✅ Pass |
| `type="read-only-storage"` | Reject (limit is 0) | Accept | ❌ Fail |

### Storage Texture Test (`VERTEX_shader_stage_storage_texture_access`)

For `shaderStage=1` (VERTEX only):
| Subcase | Expected | wgpu Behavior | Result |
|---------|----------|---------------|--------|
| `access=undefined` (→write-only) | Reject (writable prohibited) | Reject | ✅ Pass |
| `access="write-only"` | Reject (writable prohibited) | Reject | ✅ Pass |
| `access="read-write"` | Reject (writable prohibited) | Reject | ✅ Pass |
| `access="read-only"` | Reject (limit is 0) | Accept | ❌ Fail |

## Root Cause

wgpu does not implement the WebGPU per-stage storage limits:
- `maxStorageBuffersInVertexStage`
- `maxStorageBuffersInFragmentStage`
- `maxStorageTexturesInVertexStage`
- `maxStorageTexturesInFragmentStage`

These limits do not exist in `wgpu-types/src/limits.rs`. wgpu only has `max_storage_buffers_per_shader_stage` and `max_storage_textures_per_shader_stage` which apply globally, not per-stage.

The CTS test helper `isValidBufferTypeForStages` checks:
```typescript
if (visibility & GPUShaderStage.VERTEX) {
  if (!(device.limits.maxStorageBuffersInVertexStage! > 0)) {
    return false;  // Expect validation to fail
  }
}
```

Since wgpu doesn't expose this limit, it's `undefined` in JavaScript, and `undefined > 0` is `false`, so the test expects validation to fail. But wgpu accepts the configuration.

## What About VERTEX_WRITABLE_STORAGE?

The native feature `VERTEX_WRITABLE_STORAGE` is **not relevant to these test failures**.

wgpu's validation in `wgpu-core/src/device/resource.rs` correctly handles writable storage:
```rust
if entry.visibility.contains(wgt::ShaderStages::VERTEX) {
    if writable_storage == WritableStorage::Yes {
        required_features |= wgt::Features::VERTEX_WRITABLE_STORAGE;
    }
}
```

Since `VERTEX_WRITABLE_STORAGE` is a native-only feature and is NOT being requested by the CTS (the adapter may advertise it but it's apparently not being enabled on the device), wgpu correctly rejects writable storage in VERTEX.

The failing tests are all about **read-only** storage, which has nothing to do with `VERTEX_WRITABLE_STORAGE`.

## Fix Required

Implement the four per-stage storage limits in wgpu:

1. Add to `wgpu-types/src/limits.rs`:
   - `max_storage_buffers_in_vertex_stage: u32`
   - `max_storage_buffers_in_fragment_stage: u32`
   - `max_storage_textures_in_vertex_stage: u32`
   - `max_storage_textures_in_fragment_stage: u32`

2. Expose these limits through the Deno bindings in `deno_webgpu/adapter.rs`

3. Add validation in `wgpu-core/src/device/resource.rs` to check these limits during bind group layout creation

4. Set appropriate default values based on adapter capabilities

This is the same issue documented in `createBindGroupLayout_visibility.md`.

## Related Documentation

- `docs/cts-triage/createBindGroupLayout_visibility.md` - Documents the same per-stage limits issue
- `docs/cts-triage/capability_checks_limits.md` - Documents limit validation failures
