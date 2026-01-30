# Capability Checks: Limits CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,capability_checks,limits,*`

Model: Claude Sonnet 4.5

**Overall Status:** 4,994P / 2,050F / 2,204S (54.7% / 22.5% / 24.2%)

Note: Testing all limits at once causes a panic in the Metal backend (`wgpu-hal/src/metal/device.rs:412:18`) when attempting to create very large buffers. This appears to be an unwrap on None when Metal fails to allocate the buffer. Individual limit categories were tested separately to work around this crash.

## Summary by Limit Category

### Passing Limits (100% pass rate)
- `maxBindingsPerBindGroup` - 71/71 (100%)
- `maxComputeInvocationsPerWorkgroup` - 20/20 (100%)
- `maxComputeWorkgroupSizeX` - 21/21 (100%)
- `maxComputeWorkgroupSizeY` - 21/21 (100%)
- `maxComputeWorkgroupSizeZ` - 21/21 (100%)
- `maxTextureArrayLayers` - 10/10 (100%)
- `maxTextureDimension1D` - 10/10 (100%)
- `maxVertexAttributes` - 20/20 (100%)
- `maxVertexBufferArrayStride` - 21/21 (100%)
- `minStorageBufferOffsetAlignment` - 22/22 (100%)
- `minUniformBufferOffsetAlignment` - 22/22 (100%)

### Limits with Only Skipped Tests (no failures)
- `maxColorAttachments` - 26P/0F/16S (61.9% / 0% / 38.1%)
- `maxComputeWorkgroupsPerDimension` - 37P/0F/24S (60.7% / 0% / 39.3%)
- `maxDynamicStorageBuffersPerPipelineLayout` - 72P/0F/48S (60% / 0% / 40%)
- `maxDynamicUniformBuffersPerPipelineLayout` - 48P/0F/32S (60% / 0% / 40%)
- `maxSamplersPerShaderStage` - 564P/48F/408S (55.3% / 4.7% / 40%)
- `maxStorageBuffersPerShaderStage` - 374P/46F/280S (53.4% / 6.6% / 40%)
- `maxTextureDimension2D` - 18P/0F/32S (36% / 0% / 64%)
- `maxTextureDimension3D` - 6P/0F/4S (60% / 0% / 40%)

### Limits with Failures

#### Missing Limit Implementation (100% failure - limit not exposed)
- `maxBindGroupsPlusVertexBuffers` - 0P/40F/0S (0% / 100% / 0%)
- `maxStorageBuffersInFragmentStage` - 8P/134F/0S (5.6% / 94.4% / 0%)
- `maxStorageBuffersInVertexStage` - 5P/77F/0S (6.1% / 93.9% / 0%)
- `maxStorageTexturesInFragmentStage` - 11P/191F/0S (5.4% / 94.6% / 0%)
- `maxStorageTexturesInVertexStage` - 5P/77F/0S (6.1% / 93.9% / 0%)

#### Validation and Limit Issues
- `maxBindGroups` - 76P/25F/0S (75.2% / 24.8% / 0%)
- `maxColorAttachmentBytesPerSample` - 346P/54F/0S (86.5% / 13.5% / 0%)
- `maxComputeWorkgroupStorageSize` - 518P/222F/0S (70% / 30% / 0%)
- `maxInterStageShaderVariables` - 312P/208F/1360S (16.6% / 11.1% / 72.3%)
- `maxSampledTexturesPerShaderStage` - 843P/177F/0S (82.6% / 17.4% / 0%)
- `maxStorageBufferBindingSize` - 18P/4F/0S (81.8% / 18.2% / 0%)
- `maxStorageTexturesPerShaderStage` - 741P/219F/0S (77.2% / 22.8% / 0%)
- `maxUniformBufferBindingSize` - 17P/4F/0S (81% / 19% / 0%)
- `maxUniformBuffersPerShaderStage` - 768P/252F/0S (75.3% / 24.7% / 0%)
- `maxVertexBuffers` - 32P/9F/0S (78% / 22% / 0%)

## Issue Detail

### 1. Missing Limit: maxBindGroupsPlusVertexBuffers
**Test selector:** `webgpu:api,validation,capability_checks,limits,maxBindGroupsPlusVertexBuffers:*`

**What it tests:** Validates that the combined count of bind groups and vertex buffers does not exceed the limit.

**Example failure:**
```
webgpu:api,validation,capability_checks,limits,maxBindGroupsPlusVertexBuffers:createRenderPipeline,at_over:limitTest="atDefault";testValueName="atLimit";async=false
```

**Error:**
```
EXPECTATION FAILED: expected actual actualLimit: undefined to equal defaultLimit: 24
```

**Root cause:**
The `maxBindGroupsPlusVertexBuffers` limit is not exposed by wgpu. There is a TODO comment in `/Users/Andy/Development/wgpu2/wgpu/src/backend/webgpu.rs:898`:
```rust
// TODO: (maxBindGroupsPlusVertexBuffers, max_bind_groups_plus_vertex_buffers),
```

The limit is defined in the WebGPU sys bindings but not exposed through the wgpu API.

**Fix needed:**
Expose the `maxBindGroupsPlusVertexBuffers` limit in wgpu's limits structure and populate it from the underlying backend.

### 2. Missing Limits: Per-Stage Resource Limits
**Test selectors:**
- `webgpu:api,validation,capability_checks,limits,maxStorageBuffersInFragmentStage:*`
- `webgpu:api,validation,capability_checks,limits,maxStorageBuffersInVertexStage:*`
- `webgpu:api,validation,capability_checks,limits,maxStorageTexturesInFragmentStage:*`
- `webgpu:api,validation,capability_checks,limits,maxStorageTexturesInVertexStage:*`

**What they test:** Validate per-shader-stage limits for storage buffers and storage textures.

**Example failure:**
```
webgpu:api,validation,capability_checks,limits,maxStorageBuffersInFragmentStage:createBindGroupLayout,at_over:limitTest="atDefault";testValueName="atLimit";type="storage";order="forward"
```

**Error:**
```
EXPECTATION FAILED: expected actual actualLimit: undefined to equal defaultLimit: 8
```

**Root cause:**
These per-stage limits are not implemented in wgpu at all. Searching the codebase for these limit names returns no results in the wgpu source code.

**Fix needed:**
Add these four per-stage limits to wgpu's limits structure:
- `maxStorageBuffersInFragmentStage`
- `maxStorageBuffersInVertexStage`
- `maxStorageTexturesInFragmentStage`
- `maxStorageTexturesInVertexStage`

### 3. Device Creation Validation Gap: Over-Limit Requests
**Test selector:** `webgpu:api,validation,capability_checks,limits,maxBindGroups:createPipeline,at_over:*`

**What it tests:** When requesting a device with a limit value that exceeds the adapter's maximum, device creation should fail or the limit should be clamped. The test then verifies that operations respecting the limit succeed.

**Example failure:**
```
webgpu:api,validation,capability_checks,limits,maxBindGroups:createPipeline,at_over:limitTest="atDefault";testValueName="overLimit";createPipelineType="createRenderPipeline";async=false
```

**Error:**
```
EXPECTATION FAILED: unexpected validation error: Shader global ResourceBinding { group: 4, binding: 0 } uses a group index 4 that exceeds the max_bind_groups limit of 4.
```

**Root cause:**
When the test requests a device with `maxBindGroups` set to 5 (over the default limit of 4), wgpu appears to accept the device creation but then later rejects pipeline creation when it sees `@group(4)`. The test expects that when `testValueName="overLimit"`, the device should not be created with an impossible limit value, or the limit should be silently clamped to the maximum supported value.

The test creates a shader with `@group(4)` (lastIndex = 5-1 = 4) and expects this to succeed because it requested a limit of 5. However, wgpu is validating against the actual limit (4) and rejecting the shader.

**Fix needed:**
During device creation, validate that requested limits do not exceed the adapter's supported maximum limits. Either:
1. Reject device creation with a validation error if any requested limit exceeds the maximum, OR
2. Clamp requested limits to the maximum supported values

This pattern affects multiple limit categories including:
- `maxBindGroups` (25 failures)
- `maxUniformBuffersPerShaderStage` (252 failures)
- `maxSampledTexturesPerShaderStage` (177 failures)
- `maxStorageTexturesPerShaderStage` (219 failures)
- And others

### 4. Incorrect Format Validation: maxColorAttachmentBytesPerSample
**Test selector:** `webgpu:api,validation,capability_checks,limits,maxColorAttachmentBytesPerSample:createRenderPipeline,at_over:*`

**What it tests:** Validates that the total bytes per sample across all color attachments does not exceed the limit when multisampling is enabled.

**Example failure:**
```
webgpu:api,validation,capability_checks,limits,maxColorAttachmentBytesPerSample:createRenderPipeline,at_over:limitTest="atDefault";testValueName="atLimit";async=false;sampleCount=4;interleaveFormat="rgba16uint"
```

**Error:**
```
EXPECTATION FAILED: Color state [1] is invalid: Sample count 4 is not supported by format Rgba32Uint on this device. The WebGPU spec guarantees [1] samples are supported by this format. With the TEXTURE_ADAPTER_SPECIFIC_FORMAT_FEATURES feature your device supports [1, 2, 4].
```

**Root cause:**
The error message is contradictory - it says sample count 4 is not supported, but then states that the device supports [1, 2, 4]. This indicates a bug in wgpu's validation logic where it's incorrectly rejecting a valid configuration.

The test is trying to create a render pipeline with multiple color attachments (including Rgba32Uint) with sample count 4, which should be valid according to both the spec and the device's capabilities.

**Fix needed:**
Fix the validation logic for multisampled render targets to correctly check format support. The validation appears to be incorrectly rejecting formats that are actually supported for multisampling.

### 5. Workgroup Storage Size Validation Gap
**Test selector:** `webgpu:api,validation,capability_checks,limits,maxComputeWorkgroupStorageSize:createComputePipeline,at_over:*`

**What it tests:** Validates that compute shaders with workgroup-shared variables exceeding the size limit are rejected.

**Example failure:**
```
webgpu:api,validation,capability_checks,limits,maxComputeWorkgroupStorageSize:createComputePipeline,at_over:limitTest="atDefault";testValueName="overLimit";async=false;wgslType="f16"
```

**Error:**
```
EXPECTATION FAILED: no error when one was expected: size: 16400, limit: 16384
```

**Root cause:**
wgpu is not validating that the total size of workgroup-shared variables in a compute shader stays within `maxComputeWorkgroupStorageSize`. The test creates a shader with 16400 bytes of workgroup storage (exceeding the 16384 limit), but wgpu accepts it.

**Fix needed:**
Add validation during compute pipeline creation to calculate the total workgroup storage size used by the shader and reject pipelines that exceed `maxComputeWorkgroupStorageSize`.

### 6. Incorrect Limit Value: maxInterStageShaderVariables
**Test selector:** `webgpu:api,validation,capability_checks,limits,maxInterStageShaderVariables:createRenderPipeline,at_over:*`

**What it tests:** Validates that the number of inter-stage shader variables (vertex outputs / fragment inputs) does not exceed the limit.

**Example failure:**
```
webgpu:api,validation,capability_checks,limits,maxInterStageShaderVariables:createRenderPipeline,at_over:limitTest="underDefault";testValueName="atLimit";async=false;items=[]
```

**Error:**
```
EXPECTATION FAILED: expected actual actualLimit: 15 to equal defaultLimit: 16
```

**Root cause:**
wgpu is reporting `maxInterStageShaderVariables` as 15, but the WebGPU specification requires the default to be 16. This is an off-by-one error in the limit value.

**Fix needed:**
Update the default value for `maxInterStageShaderVariables` from 15 to 16 to match the WebGPU specification.

### 7. Feature-Dependent Test Skips
Many tests in `maxInterStageShaderVariables` are skipped because they require unimplemented features:
- `primitive-index` feature (for `@builtin(primitive_index)`)
- Subgroup features (for `@builtin(subgroup_invocation_id)`, `@builtin(subgroup_size)`)

These skips are expected and do not represent bugs in the limits implementation.

### 8. Backend Crash: Large Buffer Allocation
**Issue:** When running all limit tests together, wgpu panics in the Metal backend at `wgpu-hal/src/metal/device.rs:412:18` with:
```
called `Option::unwrap()` on a `None` value
```

**Root cause:**
The code calls `newBufferWithLength_options().unwrap()` which panics when Metal returns `None` (likely because the requested buffer size is too large to allocate).

**Impact:**
This prevents running the full limit test suite in a single pass. Tests must be run per-limit to avoid hitting this crash.

**Fix needed:**
Replace the `.unwrap()` with proper error handling that returns a wgpu error instead of panicking. This is particularly important for limit tests which intentionally try to allocate resources at the maximum supported sizes.

## Testing Notes

1. Individual limit categories were tested separately due to the Metal backend crash when testing all limits together.
2. The crash appears to occur when tests for buffer-size-related limits (`maxBufferSize`) attempt to allocate very large buffers.
3. Many tests have skip conditions for features not yet implemented (subgroup operations, primitive-index, etc.).
4. The overall pattern shows that basic limit functionality works, but there are gaps in:
   - Missing limit implementations (per-stage limits, maxBindGroupsPlusVertexBuffers)
   - Validation of over-limit resource usage
   - Device creation validation when requested limits exceed maximums
   - Correct default limit values

## Recommendations

**Priority 1 (High Impact):**
1. Fix the Metal backend crash by adding proper error handling for failed buffer allocations
2. Implement missing limits: `maxBindGroupsPlusVertexBuffers` and the four per-stage storage limits
3. Add device creation validation to reject or clamp over-limit requests

**Priority 2 (Validation Gaps):**
1. Add validation for workgroup storage size limits
2. Fix the multisampling format validation bug in `maxColorAttachmentBytesPerSample`
3. Correct the `maxInterStageShaderVariables` default value from 15 to 16

**Priority 3 (Lower Impact):**
1. Investigate and fix remaining failures in per-shader-stage limits with similar patterns
2. Add validation for other resource limits that currently have gaps

## Overall Assessment

The limit checking implementation in wgpu is partially complete:
- ✅ Core limits are exposed and functional
- ✅ Basic limit validation works for many common cases
- ❌ Several important limits are not yet implemented
- ❌ Device creation doesn't properly validate requested limits against maximums
- ❌ Some validation checks are missing or incorrect

The 54.7% pass rate is reasonable given the missing implementations, but could be significantly improved by addressing the missing limits and validation gaps identified above.
