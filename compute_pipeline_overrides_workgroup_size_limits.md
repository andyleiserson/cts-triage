# Compute Pipeline Overrides Workgroup Size Limits - Triage Report

CTS selector: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*`

Model: Claude Sonnet 4.5

**Overall Status:** 0P/4F/0S (0%/100%/0%)

## Test Status

All 4 test cases are failing with device initialization errors, preventing validation testing:
- `isAsync=true;type="u32"` - Device initialization failed
- `isAsync=true;type="i32"` - Device initialization failed
- `isAsync=false;type="u32"` - Device initialization failed
- `isAsync=false;type="i32"` - Device initialization failed

## Issue Detail

### 1. Device Initialization Failure Blocks Testing

**Test selector:** `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*`

**What it tests:** These tests validate that when pipeline constants (overrides) are used in `@workgroup_size()` attributes, the resulting workgroup dimensions are validated against device limits at pipeline creation time.

The test shader uses override constants for workgroup dimensions:
```wgsl
override x: u32 = 1u;
override y: u32 = 1u;
override z: u32 = 1u;
@compute @workgroup_size(x, y, z) fn main() {
  _ = 0u;
}
```

The test then tries to create pipelines with constants that:
- Should pass: `x = maxComputeWorkgroupSizeX, y = 1, z = 1`
- Should fail: `x = maxComputeWorkgroupSizeX + 1, y = 1, z = 1`
- Should pass: `x = 1, y = maxComputeWorkgroupSizeY, z = 1`
- Should fail: `x = 1, y = maxComputeWorkgroupSizeY + 1, z = 1`
- And similar for Z dimension and total invocations

**Error:**
```
EXCEPTION: Error: WebGPU device failed to initialize with Error "requestAdapter returned null"; not retrying
  at assert (file:///Users/Andy/Development/wgpu2/cts/out/common/util/util.js:37:11)
  at DevicePool.acquire (file:///Users/Andy/Development/wgpu2/cts/out/webgpu/util/device_pool.js:80:5)
```

**Root cause:**

The tests extend `AllFeaturesMaxLimitsGPUTest` which attempts to request a device with all features enabled and all limits set to their maximum values (from `adapter.limits`). However, wgpu's `requestAdapter` is returning `null`, indicating that:

1. Either wgpu cannot provide an adapter that supports all the requested limits
2. Or there's an incompatibility between what the test requests and what wgpu can provide on the current backend (Metal on macOS in this case)

This is a pre-test infrastructure issue, not the actual validation gap being tested.

**Expected validation gap (once device initialization works):**

Per the WebGPU spec and the brief note in `triage.md`, wgpu is missing validation of workgroup size dimensions after pipeline constants are applied. Currently:

1. When a shader module is created with `@workgroup_size(x, y, z)` where x/y/z are override constants with default values (e.g., `override x: u32 = 1u`), the validation in `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs` (lines 1358-1414) checks the default workgroup size against limits
2. However, when `createComputePipeline()` is called with pipeline constants that override these values, there is no re-validation of the resulting workgroup size
3. This allows creating pipelines with workgroup sizes that exceed device limits

The validation code exists in `wgpu-core/src/validation.rs` in the `check_stage()` function, which validates:
- `max_compute_workgroup_size_x/y/z` for individual dimensions
- `max_compute_invocations_per_workgroup` for total invocations (x * y * z)
- Workgroup size is not zero

But this validation runs at shader module creation time, before pipeline constants are applied.

**Fix needed:**

1. **Immediate:** Debug and resolve the device initialization issue affecting all `compute_pipeline` validation tests using `AllFeaturesMaxLimitsGPUTest`
   - Investigate what limits are being requested
   - Determine which limit wgpu cannot provide
   - Check if this is backend-specific (Metal on macOS)
   - Consider whether the test infrastructure needs updates

2. **After device initialization is fixed:** Add workgroup size re-validation in `create_compute_pipeline()`
   - After pipeline constants are applied by the HAL backend, retrieve the final workgroup size
   - Re-run the workgroup size validation logic (currently in `validation.rs:check_stage()`)
   - Return appropriate error if limits are exceeded
   - Add new error variant to `CreateComputePipelineError` for workgroup size limit violations

## Related Files

- **Test source:** `/Users/Andy/Development/cts/src/webgpu/api/validation/compute_pipeline.spec.ts` (lines 632-682)
- **Pipeline creation:** `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` (`create_compute_pipeline()` function, line 3721+)
- **Existing validation:** `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs` (lines 1358-1414, `check_stage()` function)
- **Test infrastructure:** `/Users/Andy/Development/cts/src/webgpu/gpu_test.ts` (AllFeaturesMaxLimitsGPUTest class)

## Investigation Notes

The device initialization failure affects ALL `compute_pipeline` validation tests, not just this specific selector. For example:
- `webgpu:api,validation,compute_pipeline:basic:*` - Same failure
- `webgpu:api,validation,compute_pipeline:overrides,workgroup_size:*` - Same failure (20/20 tests fail with device init)

This suggests a systemic issue with how the CTS test harness requests maximum limits vs. what wgpu can provide. This needs to be resolved before the actual validation gap can be tested and fixed.
