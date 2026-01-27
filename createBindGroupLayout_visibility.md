# CreateBindGroupLayout Visibility CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,createBindGroupLayout:visibility:*`

**Overall Status:** 2P/6F/0S (25%/75%/0%)

## Passing Sub-suites

- `visibility=0` - No shader stage visibility (all subcases pass)

## Remaining Issues

All 6 tests with non-zero visibility fail:
- `visibility=1` (VERTEX)
- `visibility=2` (FRAGMENT)
- `visibility=3` (VERTEX + FRAGMENT)
- `visibility=4` (COMPUTE)
- `visibility=5` (VERTEX + COMPUTE)
- `visibility=6` (FRAGMENT + COMPUTE)
- `visibility=7` (all three stages)

## Issue Detail

### 1. Missing per-stage storage limits validation

**Test selector:** `webgpu:api,validation,createBindGroupLayout:visibility:*`

**What it tests:** Validates that creating bind group layout entries respects the per-stage storage limits. WebGPU spec requires these four limits:
1. `maxStorageBuffersInVertexStage`
2. `maxStorageBuffersInFragmentStage`
3. `maxStorageTexturesInVertexStage`
4. `maxStorageTexturesInFragmentStage`

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Root cause:**

wgpu does not implement the four per-stage storage limits that WebGPU requires. Instead, wgpu uses binary feature flags (`DownlevelFlags`):
- `VERTEX_STORAGE` - controls storage buffers in vertex stage
- `FRAGMENT_STORAGE` - controls storage buffers in fragment stage
- `FRAGMENT_WRITABLE_STORAGE` - controls writable storage in fragment stage

When the per-stage limits are 0 (the default in CTS), creating a bind group layout entry with storage buffers/textures visible in those stages should fail validation. But wgpu accepts them.

**Validation gaps in `wgpu-core/src/device/resource.rs`:**
1. Read-only storage buffers in fragment stage are not checked
2. Storage textures in vertex stage are not checked at all
3. Read-only storage textures in fragment stage are not checked

**Fix needed:**

This requires a significant architectural change:
1. Add the four per-stage limit fields to the `Limits` struct:
   - `max_storage_buffers_in_vertex_stage`
   - `max_storage_buffers_in_fragment_stage`
   - `max_storage_textures_in_vertex_stage`
   - `max_storage_textures_in_fragment_stage`
2. Add validation in `create_bind_group_layout_internal` to check these limits
3. Map device capabilities to these numeric limits appropriately

This is a non-trivial change that affects the limits API surface.
