# Compute Pipeline Overrides Workgroup Storage Size - Triage Report

CTS selector: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*`

Model: Claude Sonnet 4.5

**Overall Status:** 0P/2F/0S (0%/100%/0%)

## Test Overview

This test validates that wgpu correctly enforces the `maxComputeWorkgroupStorageSize` limit when pipeline constants (overrides) are used to determine the size of workgroup storage arrays.

## Failing Tests

Both test cases fail with 100% failure rate:
- `isAsync=true` - Testing async pipeline creation
- `isAsync=false` - Testing sync pipeline creation

## Issue Detail

### Missing validation of workgroup storage size with overrides

**Test selector:** `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*`

**What it tests:**

The test creates compute pipelines with workgroup storage arrays whose sizes are determined by overridable constants. It validates that when the total workgroup storage exceeds `device.limits.maxComputeWorkgroupStorageSize`, the pipeline creation should fail with a validation error.

The test shader template is:
```wgsl
override a: u32;
override b: u32;
var<workgroup> vec4_data: array<vec4<f32>, a>;
var<workgroup> mat4_data: array<mat4x4<f32>, b>;
```

The test runs three scenarios:
1. `testFn(1, 1, true)` - Valid case with small arrays (should pass)
2. `testFn(maxVec4Count + 1, 0, false)` - Exceeds limit with vec4 array (should fail validation)
3. `testFn(0, maxMat4Count + 1, false)` - Exceeds limit with mat4x4 array (should fail validation)

Where:
- `maxVec4Count = maxComputeWorkgroupStorageSize / 16` (vec4<f32> is 16 bytes)
- `maxMat4Count = maxComputeWorkgroupStorageSize / 64` (mat4x4<f32> is 64 bytes)

**Example failing test:**
```
webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:isAsync=false
```

**Error (async case):**
```
EXPECTATION FAILED: DID NOT REJECT
```

**Error (sync case):**
```
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Root cause:**

wgpu-core does not validate workgroup storage size at all, neither with nor without pipeline constants. When creating a compute pipeline in `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` (function `create_compute_pipeline` starting at line 3721), the code:

1. Validates the shader module and entry point exist
2. Validates binding compatibility
3. Passes the pipeline constants directly to the HAL layer (line 3801: `constants: &desc.stage.constants`)
4. Creates the pipeline via HAL without any workgroup storage size validation

The validation is completely missing. The HAL backends also do not perform this validation - they typically pass shader compilation to the driver, which may or may not enforce this limit.

**Why this is complex:**

The validation is non-trivial because:

1. **Timing Issue**: Workgroup storage size depends on pipeline constants, which are only known at pipeline creation time, not at shader module creation time. The `Interface::new()` function in `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs` (line 1083) runs at shader module creation and cannot know the final array sizes.

2. **Requires Constant Evaluation**: To calculate the workgroup storage size, we need to:
   - Identify all global variables with `AddressSpace::WorkGroup`
   - For each variable, get its type size using `module.types[var.ty].inner.size(module.to_ctx())`
   - If the type is an array with an overridable constant for its size, evaluate that constant with the provided pipeline constant values
   - Sum up the total workgroup storage usage
   - Check against `device.limits.max_compute_workgroup_storage_size`

3. **Array Size Evaluation**: Arrays can be sized by:
   - Literal constants: `array<f32, 100>` - size known at shader module creation
   - Overridable constants: `array<f32, N>` where N is an override - size only known at pipeline creation after constants are provided
   - Override expressions: More complex cases where array sizes depend on expressions involving overrides

4. **Data Availability**: The naga `Module` contains:
   - `global_variables` arena with each variable's type and address space
   - `types` arena with type information including size calculation
   - `entry_points` with workgroup size information

   However, the current `EntryPoint` struct in wgpu-core validation (line 172 of validation.rs) only tracks `workgroup_size: [u32; 3]` for workgroup dimensions, not workgroup storage size.

**Fix needed:**

A complete fix requires:

1. **Add workgroup storage calculation function** in wgpu-core that:
   - Takes a naga Module, entry point, and pipeline constants map
   - Iterates through global variables used by the entry point
   - For WorkGroup address space variables, calculates their size
   - For arrays with override-sized dimensions, evaluates the overrides with provided constants
   - Returns total workgroup storage size in bytes

2. **Add validation in `create_compute_pipeline`** (around line 3805, before calling HAL):
   ```rust
   // Validate workgroup storage size
   let workgroup_storage_size = calculate_workgroup_storage_size(
       shader_module.module(),
       &final_entry_point_name,
       &desc.stage.constants,
   )?;

   if workgroup_storage_size > self.limits.max_compute_workgroup_storage_size as u64 {
       return Err(pipeline::CreateComputePipelineError::WorkgroupStorageSizeExceeded {
           used: workgroup_storage_size,
           limit: self.limits.max_compute_workgroup_storage_size,
       });
   }
   ```

3. **Add new error variant** in `CreateComputePipelineError`:
   ```rust
   #[error("Workgroup storage size {used} bytes exceeds limit {limit}")]
   WorkgroupStorageSizeExceeded { used: u64, limit: u32 },
   ```

4. **Consider leveraging naga's pipeline constant processing**: The function `naga::back::pipeline_constants::process_overrides` already handles evaluating constants, but would need to be extended or its results used to calculate workgroup storage sizes.

**Complexity Assessment:**

This is a **HIGH COMPLEXITY** fix that requires:
- Deep understanding of naga's type system and constant evaluation
- Access to or calculation of array sizes with overrides applied
- Integration point between shader module analysis and pipeline creation
- Careful handling of edge cases (nested types, struct padding, alignment)

The implementation would need to handle the full complexity of WGSL types including:
- Primitive types (scalars, vectors, matrices)
- Arrays (fixed-size and override-sized)
- Structs with proper alignment and padding
- Nested combinations of the above

**Related Tests:**

There's also a similar selector mentioned in triage.md:
- `webgpu:api,validation,compute_pipeline:limits,workgroup_storage_size:*` - Tests without overrides (4 failures)

This suggests that even the basic workgroup storage size validation (without overrides) is missing.

## Key Files Referenced

- **Test Source:** `/Users/Andy/Development/cts/src/webgpu/api/validation/compute_pipeline.spec.ts` (lines 689-733)
- **Pipeline Creation:** `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` (lines 3721-3851)
- **Validation Interface:** `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs` (lines 186-190, 1083-1193)
- **Type Size Calculation:** `/Users/Andy/Development/wgpu2/naga/src/valid/interface.rs` (example at line 1171)
- **Global Variables:** `/Users/Andy/Development/wgpu2/naga/src/ir/mod.rs` (lines 1100-1113)
