# CTS Triage Summary Report

This document contains triage information for CTS test failures in
`cts_runner/fail.lst`.

It contains this title section and an initial section "Known Issues Reference"
with some overview information. It then contains sections documenting specific
failing test selectors from `fail.lst`.

## Rules When Editing This Document

* Selectors should appear in in alphabetical order. (If the same problem
  applies to multiple selectors, you can add a cross-reference to the primary
  section describing the problem from other sections.)
* Every section except this title section and "Known Issues Reference" should
  list associated selectors.

**IMPORTANT**: When creating or editing the sections documenting specific
failures, always use the label "Selector: " or "Selectors: " to identify the
failing selectors. Do not make it bold or add any other kind of emphasis. Do
not say "CTS selectors" or "Test selectors". Only say "Selector: " or
"Selectors: ", never anything else.

# Known Issues Reference

## Direct Access to Atomics (#5474)

Many shader validation tests fail because Naga allows referencing atomics directly in expressions instead of requiring `atomicLoad`, `atomicStore`, etc.

Selectors:
- `webgpu:shader,validation,expression,binary,add_sub_mul:*`
- `webgpu:shader,validation,expression,binary,and_or_xor:*`
- `webgpu:shader,validation,expression,binary,comparison:*`
- `webgpu:shader,validation,expression,binary,div_rem:*`
- `webgpu:shader,validation,expression,binary,bitwise_shift:*`
- `webgpu:shader,validation,expression,call,builtin,abs:*`
- `webgpu:shader,validation,expression,call,builtin,countLeadingZeros:*`
- `webgpu:shader,validation,expression,call,builtin,countOneBits:*`
- `webgpu:shader,validation,expression,call,builtin,countTrailingZeros:*`
- `webgpu:shader,validation,expression,call,builtin,firstLeadingBit:*`
- `webgpu:shader,validation,expression,call,builtin,firstTrailingBit:*`
- `webgpu:shader,validation,expression,call,builtin,reverseBits:*`
- `webgpu:shader,validation,expression,call,builtin,select:*`
- `webgpu:shader,validation,statement,increment_decrement:*`

## Destroyed Resource Validation Error Timing (#7881)

Tests that check for validation errors when a destroyed resource is used. `wgpu` often reports these errors later than WebGPU requires, causing the tests to fail.

Selectors:
- `webgpu:api,validation,createBindGroup:texture,resource_state:*`
- `webgpu:api,validation,encoding,cmds,setBindGroup:*` (state="destroyed" subcases)

## Behavior Analysis Not Implemented (#7650)

Naga does not implement the behavior analysis algorithm required by WebGPU.

Selectors:
- `webgpu:shader,validation,statement,statement_behavior:*`
- `webgpu:shader,validation,statement,loop:*`
- `webgpu:shader,validation,statement,switch:*`
- `webgpu:shader,validation,statement,continue:*`

## wgslLanguageFeatures Not Implemented in Deno (PR #8884)

Tests that rely on `gpu.wgslLanguageFeatures` to detect support for WGSL language extensions. Since `deno_webgpu` doesn't expose this API, tests fail because they cannot detect that certain features are implemented.

Selectors:
- `webgpu:shader,validation,expression,call,builtin,dot4I8Packed:*`
- `webgpu:shader,validation,expression,call,builtin,dot4U8Packed:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xI8:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xI8Clamp:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xU8:*`
- `webgpu:shader,validation,expression,call,builtin,pack4xU8Clamp:*`
- `webgpu:shader,validation,expression,call,builtin,unpack4xI8:*`
- `webgpu:shader,validation,expression,call,builtin,unpack4xU8:*`
- `webgpu:shader,validation,extension,pointer_composite_access:*`
- `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*`
- `webgpu:shader,validation,parse,requires:*`

Implementation is in [PR #8884](https://github.com/gfx-rs/wgpu/pull/8884).

## Constant Evaluation Missing Domain/Range/Overflow Validation (#8900)

The implementation of many functions for constant evaluation omits required
checks of the domain/range of the function (e.g. detecting illegal input
values or overflow during the computation).

Selectors:
- `webgpu:shader,validation,expression,call,builtin,acos:*`
- `webgpu:shader,validation,expression,call,builtin,asin:*`
- `webgpu:shader,validation,expression,call,builtin,atanh:*`
- `webgpu:shader,validation,expression,call,builtin,cosh:*`
- `webgpu:shader,validation,expression,call,builtin,degrees:*`
- `webgpu:shader,validation,expression,call,builtin,dot:*`
- `webgpu:shader,validation,expression,call,builtin,exp:*`
- `webgpu:shader,validation,expression,call,builtin,exp2:*`
- `webgpu:shader,validation,expression,call,builtin,length:*`
- `webgpu:shader,validation,expression,call,builtin,log:*`
- `webgpu:shader,validation,expression,call,builtin,log2:*`
- `webgpu:shader,validation,expression,call,builtin,normalize:*`
- `webgpu:shader,validation,expression,call,builtin,pow:*`
- `webgpu:shader,validation,expression,call,builtin,sinh:*`
- `webgpu:shader,validation,expression,call,builtin,sqrt:*`

## Functions Missing Constant Evaluation Support (#4507)

Tests that fail because Naga has not implemented support for a particular
function(s) in constant evaluation.

trigonometry

- [ ] `atan2`

decomposition

- [ ] `modf`
- [ ] `frexp`
- [ ] `ldexp`

geometry

- [ ] `dot`
- [ ] `cross`
- [ ] `distance`
- [ ] `length`
- [ ] `normalize`
- [ ] `faceForward`
- [ ] `reflect`
- [ ] `refract`

computational

- [ ] `mix`
- [ ] `smoothstep`
- [ ] `transpose`
- [ ] `determinant`

bits

- [ ] `extractBits`
- [ ] `insertBits`

data packing

- [ ] `pack4x8snorm`
- [ ] `pack4x8unorm`
- [ ] `pack2x16snorm`
- [ ] `pack2x16unorm`
- [ ] `pack2x16float`
- [ ] `pack4xI8`
- [ ] `pack4xU8`
- [ ] `pack4xI8Clamp`
- [ ] `pack4xU8Clamp`

data unpacking

- [ ] `unpack4x8snorm`
- [ ] `unpack4x8unorm`
- [ ] `unpack2x16snorm`
- [ ] `unpack2x16unorm`
- [ ] `unpack2x16float`
- [ ] `unpack4xI8`
- [ ] `unpack4xI8`

misc

- [ ] `bitcast`
- [ ] `select`
- [ ] `all`
- [ ] `any`
- [ ] `quantizeToF16`

---

# Capability Checks

## Features (21% pass)

Selector: `webgpu:api,validation,capability_checks,features,*`

**Overall Status:** 21% pass rate

wgpu-core returns `MissingFeatures` errors as `ErrorType::Validation`, but the WebGPU specification requires that using optional features without enabling them should throw **TypeError** exceptions synchronously, not validation errors that get pushed to error scopes.

**Current behavior**: wgpu-core correctly detects when a feature is missing (via `Device::require_features()` and `TextureFormat::required_features()`), but the error is classified as a validation error. In deno_webgpu, these validation errors are pushed to the error handler rather than being thrown as TypeError exceptions.

**Expected behavior**: Per WebGPU spec, these should throw TypeError synchronously when the API call is made, not generate validation errors.

**Fix needed:** Modify deno_webgpu to intercept `MissingFeatures` errors and throw TypeError. Implement TEXTURE_COMPONENT_SWIZZLE feature in wgpu-types/features.rs.

See: `docs/cts-triage/capability_checks_features.md` for detailed analysis.

## Limits (65% pass)

Selector: `webgpu:api,validation,capability_checks,limits,*`

**Overall Status:** 5100P/2453F/312S (65%/31%/4%)

**Root causes:**
1. **wgslLanguageFeatures Not Implemented** (~789 failures) - deno_webgpu missing gpu.wgslLanguageFeatures API (PR #8884)
2. **Workgroup Storage Size Validation Missing** (~222 failures) - No validation of maxComputeWorkgroupStorageSize at pipeline creation
3. **InterStage Shader Variables Counting** (~256 failures) - Incorrect built-in variable limit counting (CTS issue #4538)
4. **Bind Group Index Validation Timing** (~25 failures, dx12-specific) - Validation at wrong pipeline point

See: `docs/cts-triage/capability_checks_limits.md` for detailed analysis.

---

# Compute Pipeline Validation

## Workgroup Limits (0% pass)

Selector: `webgpu:api,validation,compute_pipeline:limits,workgroup_storage_size:*`

wgpu missing validation of `maxComputeWorkgroupStorageSize` limit (4 failures).

## Override Workgroup Storage Size (0% pass)

Selector: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*`

**Overall Status:** 0P/2F/0S (0% pass, 100% fail)

**What it tests:** Validates that pipeline constants (overrides) determining workgroup storage array sizes are checked against `maxComputeWorkgroupStorageSize` at pipeline creation.

**Root cause:** wgpu-core completely lacks validation for workgroup storage size limits. The code validates shader modules and bindings but never checks workgroup storage size against device limits.

**Why complex:** Requires constant evaluation of override expressions at pipeline creation time, calculating sizes for complex WGSL types with proper alignment, and integrating shader analysis with pipeline validation.

**Fix needed:** Add validation in `create_compute_pipeline` to calculate total workgroup storage size (evaluating overrides) and check against limit.

See: `docs/cts-triage/compute_pipeline_overrides_workgroup_storage_size.md` for full analysis.

## Override Workgroup Size Limits (0% pass)

Selector: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*`

Missing re-validation of workgroup size (dimensions, not storage) limits after pipeline constants applied (2 failures).

## Resource Compatibility (84% pass)

Selector: `webgpu:api,validation,compute_pipeline:resource_compatibility:*`

Storage texture read-write bindings not compatible with write-only shaders (12 failures).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8905

---

# Compute Pipeline Overrides (Identifier)

Selector: `webgpu:api,validation,compute_pipeline:overrides,identifier:*`

**Overall Status:** 12P/14F/0S (46%/54%/0%)

## 1. Unknown pipeline constant identifiers silently accepted

Selector: `webgpu:api,validation,compute_pipeline:overrides,identifier:*`

**What it tests:** Pipeline constant identifiers must match valid shader overrides.

**Root cause:** Naga's `process_overrides` doesn't validate that provided keys correspond to shader overrides. Unknown keys silently ignored.

**Fix needed:** Add validation to track consumed keys and error on unused keys. Requires new `PipelineConstantError::UnknownIdentifier` variant.

See: `docs/cts-triage/compute_pipeline_overrides.md` for details.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8906

---

# CreateBindGroup Validation

## Buffer Resource State (0% pass)

Selector: `webgpu:api,validation,createBindGroup:buffer,resource_state:*`

Destroyed buffer validation timing - known issue #7881 (1 failure).

---

# CreateBindGroupLayout Validation (overall good pass rates)

## Max Resources Per Stage (0% pass)

Selector: `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_pipeline_layout:*`

Logic bug: using `max()` instead of `+=` when merging binding counts (11 failures).

**Fix:** Change `merge()` method in `binding_model.rs` from using `max()` to `+=`.

TODO: file bug

## Vertex Shader Storage (25% pass)

Selector: `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_buffer_type:*`

Missing validation: writable storage buffers not allowed in VERTEX stage (6 failures).

TODO: file bug

## Vertex Shader Storage Textures (25% pass)

Selector: `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_storage_texture_access:*`

Missing validation: write-access storage textures not allowed in VERTEX stage (6 failures).

TODO: file bug

---

# createBindGroupLayout Visibility

Selector: `webgpu:api,validation,createBindGroupLayout:visibility:*`

**Overall Status:** 2P/6F/0S (25%/75%/0%)

## 1. Missing per-stage storage limits validation

Selector: `webgpu:api,validation,createBindGroupLayout:visibility:*` with visibility > 0

**What it tests:** Per-stage storage limits (maxStorageBuffersInVertexStage, maxStorageBuffersInFragmentStage, maxStorageTexturesInVertexStage, maxStorageTexturesInFragmentStage).

**Root cause:** wgpu uses binary feature flags (VERTEX_STORAGE, FRAGMENT_STORAGE, FRAGMENT_WRITABLE_STORAGE) instead of numeric per-stage limits.

**Fix needed:** Add four per-stage limit fields to Limits struct, add validation in create_bind_group_layout_internal, map device capabilities to limits.

See: `docs/cts-triage/createBindGroupLayout_visibility.md` for details.

---

# CreateView Validation (0% pass)

Selector: `webgpu:api,validation,createView:texture_state:*`

Destroyed texture validation timing - known issue #7881 (1 failure).

---

# Encoding Validation

## CreateRenderBundleEncoder (28% pass)

Selector: `webgpu:api,validation,encoding,createRenderBundleEncoder:*`

1. **Empty attachments** - Missing validation for no color + no depth/stencil (1 failure)
2. **Format compatibility** - TODO comment at line 191-192 of bundle.rs (95 failures)

## Pipeline Bind Group Compatibility (75% pass for default, 17% for empty)

Selector: `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:*`

Empty bind group layouts from different sources not treated as compatible (48 failures total).

null/undefined not accepted in bindGroupLayouts array - type conversion issue (30 failures).

## ResolveQuerySet (93% pass)

Destroyed query set validation - known issue #7881 (1 failure).

## Render Bundle (81% pass)

Selector: `webgpu:api,validation,encoding,render_bundle:*`

**Overall Status:** 81% pass

### 1. Readonly flag normalization mismatch

**What it tests:** Validates render bundle compatibility with render passes, including depth/stencil readonly flags.

**Root cause:** When a render bundle is created with certain readonly depth/stencil configurations, the flags are not being normalized consistently between the bundle encoder and the render pass. This causes compatibility checks to fail when they should pass, or vice versa.

---

# Error Scope

Selector: `webgpu:api,validation,error_scope:*`

**Overall Status:** 36P/12F/0S (75%/25%/0%)

## 1. Metal backend crashes on OOM texture allocation

Selector: All tests with `errorType="out-of-memory"` or `errorFilter="out-of-memory"`

**Root cause:** Metal's `new_texture` returns null on OOM. Code returns OutOfMemory error correctly, but null pointer remains. When autoreleasepool exits, metal crate's Drop panics on null pointer.

**Fix needed:** Add `std::mem::forget(raw)` before returning OutOfMemory error to prevent drop of null pointer.

See: `docs/cts-triage/error_scope.md` for details.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7357

---

# getBindGroupLayout

Selector: `webgpu:api,validation,getBindGroupLayout:*`

**Overall Status:** 4P/10F/0S (28.57%/71.43%/0%)

## 1. getBindGroupLayout rejects valid indices beyond defined layouts

Selectors: `webgpu:api,validation,getBindGroupLayout:index_range,*:index=1..5`

**What it tests:** Per WebGPU spec, `getBindGroupLayout(index)` should only fail validation if `index >= device.limits.maxBindGroups`. For indices within bounds but beyond the defined layouts, it should return an empty bind group layout.

**Error:**
```
Unexpected validation error occurred: Invalid group index 1
```

**Root cause:** In `wgpu-core/src/device/global.rs` (lines 1563-1567), the code validates that the requested index exists within the pipeline's actual bind group layouts array. But the spec says `[[bindGroupLayouts]]` should conceptually have `maxBindGroups` entries (with `null` for unused indices).

**Fix needed:** Modify `render_pipeline_get_bind_group_layout` and `compute_pipeline_get_bind_group_layout` to:
1. Check if `index >= device.limits.max_bind_groups` and return error if so
2. For indices within bounds but beyond defined layouts, return an empty bind group layout

---

# Layout Shader Compat

Selector: `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*`

**Overall Status:** 109P/1F/0S (99.09% pass)

### Issue: Storage Texture Access Mode Validation Too Strict

**Failing test:** bindingInPipelineLayout="readwriteStorageTex"; bindingInShader="writeonlyStorageTex"

**Root cause:** Validation requires exact match between layout and shader storage texture access modes. Spec allows shader to use subset of layout's access (e.g., write-only shader with read-write layout).

**Fix needed:** Modify `check_binding_use` to allow shader's requested access to be subset of layout's provided access.

See: `docs/cts-triage/layout_shader_compat_exact_match.md`

---

# Non-Filterable Textures

Selector: `webgpu:api,validation,non_filterable_texture:*`

**Overall Status:** 80% pass

Missing validation: depth textures with filtering samplers should be rejected (32 failures).

---

# Queue Destroyed

Selector: `webgpu:api,validation,queue,destroyed,*`

**Overall Status:** 71% pass

## 1. writeBuffer/writeTexture return value issue

Tests expect `writeBuffer` and `writeTexture` to return `undefined`, but the implementation may be returning something else or throwing when it shouldn't.

## 2. Destroyed query set validation missing

Tests with destroyed query sets are not being properly validated.

---

# Queue writeBuffer Ranges

Selector: `webgpu:api,validation,queue,writeBuffer:ranges:*`

**Overall Status:** 0% pass

## 1. Missing OperationError for invalid data ranges

**What it tests:** When `writeBuffer` is called with invalid data ranges (e.g., data offset + size exceeds ArrayBuffer bounds), it should throw an `OperationError`.

**Root cause:** deno\_webgpu is not validating the data source bounds properly or not returning the correct error type when validation fails. `wgpu` also fails to raise a validation error for zero-size transfers beyond the end of the buffer.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8920 (wgpu), none filed for deno

---

# Render Pass depth_clear_value

Selector: `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:*`

**Overall Status:** 94% pass

## 1. TypeError at deno binding layer instead of validation error

**What it tests:** Validates that `depthClearValue` must be within [0.0, 1.0] range when `depthLoadOp` is "clear".

**Error:**
```
TypeError: depth_clear_value is required when depth_load_op is 'clear'
```

**Root cause:** The error is thrown as a TypeError at the deno binding layer instead of being returned as a GPUValidationError through the WebGPU error handling mechanism. The CTS expects validation errors to be captured through error scopes, not thrown as JavaScript exceptions.

---

# Render Pass Validation

## LoadOp/StoreOp Match (33% pass)

Selector: `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,loadOp_storeOp_match_depthReadOnly_stencilReadOnly:*`

**Overall Status:** 2P/4F/0S (33.33% pass rate)

### 1. Missing validation for load/store operations on non-existent texture format aspects

**Root cause:** wgpu does not validate whether load/store operations are provided for texture format aspects that don't exist.

Per the WebGPU specification, the rules are:
- If format has depth aspect AND `depthReadOnly` is false: `depthLoadOp` and `depthStoreOp` MUST be provided
- Otherwise (no depth aspect OR `depthReadOnly` is true): `depthLoadOp` and `depthStoreOp` MUST NOT be provided
- Same rules apply for stencil aspect with `stencilReadOnly`, `stencilLoadOp`, and `stencilStoreOp`

**Current behavior:** wgpu accepts load/store operations even when the texture format lacks the corresponding aspect:
- `stencil8` (no depth aspect) incorrectly accepts `depthLoadOp`/`depthStoreOp`
- `depth16unorm`, `depth32float`, `depth24plus` (no stencil aspect) incorrectly accept `stencilLoadOp`/`stencilStoreOp`

**Code location:** `/Users/Andy/Development/wgpu2/wgpu-core/src/command/render.rs:1700-1718`

**Fix needed:** Add validation before lines 1700-1718 to check:
1. If format lacks depth aspect but `depthLoadOp` or `depthStoreOp` are provided → validation error
2. If format lacks stencil aspect but `stencilLoadOp` or `stencilStoreOp` are provided → validation error

## Occlusion Query Set Type (50% pass)

Selector: `webgpu:api,validation,render_pass,render_pass_descriptor:occlusionQuerySet,query_set_type:*`

Missing validation: occlusionQuerySet must have type "occlusion" (1 failure).

---

# Render Pipeline Depth/Stencil State

Selector: `webgpu:api,validation,render_pipeline,depth_stencil_state:*`

**Overall Status:** 1218P/4F/12S (98.7%/0.32%/0.97%)

## 1. depthCompare_optional stencil8

Selector: `webgpu:api,validation,render_pipeline,depth_stencil_state:depthCompare_optional:*`

**What it tests:**
- `depthCompare` is optional for stencil-only formats when not used
- `depthCompare` is required for formats with depth aspect when `depthWriteEnabled` is true

**Example failure:**
```
webgpu:api,validation,render_pipeline,depth_stencil_state:depthCompare_optional:isAsync=false;format="stencil8"
```

**Error:**
```
EXCEPTION: Error: Unexpected validation error occurred: Depth/stencil state is invalid:
Format Stencil8 does not have a depth aspect, but depth test/write is enabled
```

**Root cause:**
All 24 individual subcases report "INFO: subcase ran" indicating they passed their validation checks correctly. However, an error occurs during test cleanup at `DeviceHolder.attemptEndTestScope`, suggesting a validation error is queued on the device that wasn't properly handled or cleared during the test execution.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8830

## 2. depth_write,frag_depth undefined format

Selector: `webgpu:api,validation,render_pipeline,depth_stencil_state:depth_write,frag_depth:*`

**What it tests:**
Depth aspect must be contained in the format if `frag_depth` is written in fragment stage.

**Example failure:**
```
webgpu:api,validation,render_pipeline,depth_stencil_state:depth_write,frag_depth:isAsync=false;format="_undef_"
```

**Error:**
```
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Root cause:**
When `format=undefined` (no depth/stencil attachment), but the fragment shader writes to `frag_depth` output, wgpu accepts the pipeline when it should reject it. This is a shader output validation gap.

**Related issue:** https://github.com/gfx-rs/wgpu/pull/8856

---

# Dual Source Blending

Selectors:
- `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,*` (100% pass)
- `webgpu:shader,validation,extension,dual_source_blending:*` (overall)

**Overall API Status:** ALL API TESTS PASSING

## API Validation (100% pass)

- `dual_source_blending,color_target_count`: 16P/0F/0S (100% pass)
- `dual_source_blending,use_blend_src`: 68P/0F/0S (100% pass)

## Shader Extension: blend_src_usage (61% pass)

Selector: `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*`

**Overall Status:** 14P/9F/0S (60.87% pass)

Naga is missing validation for many invalid uses of `@blend_src`. The attribute is being accepted in contexts where it should be rejected.

## Validation Gaps (11 failures)

The following invalid uses are incorrectly accepted:
1. `@blend_src` on const, override, let declarations
2. `@blend_src` on private/function scope variables
3. `@blend_src` on function declarations
4. `@blend_src` on non-entry point function inputs/outputs
5. `@blend_src` on entry point inputs (non-struct)
6. Struct members with `@blend_src` but missing `@location` on some members

## Correctly Validated (9 failures)

Tests that properly fail validation (??? so why are they failures???):
- Entry point outputs (non-struct)
- Duplicate `@blend_src` values
- Non-zero location with `@blend_src`
- Missing one of the required pair

---

# Fragment State targets_blend

Selector: `webgpu:api,validation,render_pipeline,fragment_state:targets_blend:*`

**Overall Status:** ~0% pass (99.65% fail)

## 1. Missing min/max blend factor validation

**What it tests:** Validates that blend factors `min` and `max` cannot be used with certain blend operations.

**Error:**
```
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Root cause:** wgpu does not validate that `min` and `max` blend factors are only valid with specific blend operations. The WebGPU spec restricts their usage but wgpu accepts them unconditionally.

---

# Render Pipeline Inter Stage (85% pass)

Selector: `webgpu:api,validation,render_pipeline,inter_stage:*`

**Overall Status:** 78P/14F/0S (84.78% pass rate)

## 1. Type mismatch validation gap (4 failures)

**Root cause:** wgpu accepts render pipelines where the vertex shader output type and fragment shader input type don't match exactly.

**Examples:**
- `vec2<f32>` output vs `f32` input - should be rejected but is accepted
- `vec3<f32>` output vs `vec2<f32>` input - should be rejected but is accepted

**Expected behavior:** Per WebGPU spec, inter-stage variable types must match exactly. The CTS test shows validation should only succeed when `output === input`.

**Current behavior:** wgpu uses `iv.ty.is_subtype_of(&provided.ty)` which is too permissive.

**Code location:** `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs:1449`

**Fix needed:** Replace subtype checking with exact type equality for inter-stage variables.

TODO: verify and file issue

## 2. Interpolation default handling (8 failures)

**Root cause:** wgpu doesn't properly handle WebGPU's interpolation defaults when matching vertex output and fragment input.

Per WebGPU spec:
- Empty interpolation defaults to `@interpolate(perspective, center)`
- `@interpolate(perspective)` without sampling defaults to `center`
- `@interpolate(linear)` without sampling defaults to `center`

**Examples that should match but don't:**
- `` (empty) vs `@interpolate(perspective)` - should match (both default to `perspective, center`)
- `@interpolate(perspective)` vs `@interpolate(perspective, center)` - should match
- `@interpolate(linear, center)` vs `@interpolate(linear)` - should match

**Current behavior:** wgpu does exact equality checks on interpolation and sampling at lines 1438-1446 in `validation.rs`, treating `None` sampling as different from `Some(Center)`.

**Expected behavior:** When interpolation is `Perspective` or `Linear`, missing (None) sampling should be treated as equivalent to `Some(Center)`.

**Fix needed:** Implement default-aware comparison that treats `None` as `Some(Center)` when interpolation is `Perspective` or `Linear`.

TODO: verify and file issue

## 3. Fragment input variable counting with built-ins (2 failures)

**Root cause:** Fragment input variable counting logic appears incorrect when built-ins are present.

**Example:**
- Test creates pipeline with `maxInterStageShaderVariables - 1` (30) user-defined variables plus 3 built-in inputs
- wgpu reports: "found 30 user-defined fragment shader input variables, which exceeds the max_inter_stage_shader_variables limit (31)"
- Since 30 ≤ 31, this should pass, but wgpu rejects it

**Expected behavior:** The test expects this to succeed when using 30 user-defined variables + 3 built-ins with appropriate deductions.

**Code location:** `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs` (fragment input variable counting)

**Fix needed:** Review the variable counting logic to ensure built-in deductions are properly applied before comparison.

**Impact:** Moderate - affects inter-stage variable validation, a core WebGPU feature.

---

# Render Pipeline Overrides (83% pass)

Selector: `webgpu:api,validation,render_pipeline,overrides:*`

**Overall Status:** 140P/28F/0S (83.33% pass rate)

## 1. Missing validation for invalid pipeline constant identifiers (28 failures)

**Root cause:** wgpu/Naga silently ignores invalid pipeline constant identifiers instead of reporting validation errors.

**Examples:** Non-existent identifiers, using name when @id exists, null bytes in identifier, wrong Unicode normalization.

**Current behavior:** process_overrides looks up shader overrides but doesn't validate that all provided keys correspond to valid overrides.

**Expected:** Pipeline creation should fail with validation error for invalid identifiers.

**Fix needed:** Add validation in Naga's process_overrides to check all provided keys: correspond to actual override, use correct identifier (numeric for @id, name otherwise), no invalid characters, correct Unicode normalization.

See: `docs/cts-triage/render_pipeline_overrides.md` for details.

---

# Render Pipeline Output Targets (3% pass)

Selector: `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets:*`

Color target without shader output requires writeMask=0 (66 failures).

---

# Render Pipeline Output Targets Blend (95% pass)

Selector: `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets,blend:*`

Blend factors reading source alpha require vec4 output (100 failures).

---

# Render Pipeline Resource Compatibility (90% pass)

Selector: `webgpu:api,validation,render_pipeline:resource_compatibility:*`

**Overall Status:** 111P/12F/0S (90.24% pass rate)

**Root cause:** Storage texture access mode validation is too strict. When a pipeline layout binding has `read-write` access, shaders using `write-only` access should be compatible, but wgpu requires exact match.

**Failing tests:** All 12 failures are fragment stage tests with `read-write` storage textures (all dimensions and formats: 1d, 2d, 2d-array, 3d × r32float, r32sint, r32uint).

**Fix needed:** Modify validation at `wgpu-core/src/validation.rs:703` to allow bind group layout access to be a superset of shader access requirements (i.e., `read-write` layout matches `write-only` shader).

**Note:** Same issue as compute pipeline resource compatibility (84% pass rate).

See: `docs/cts-triage/render_pipeline_resource_compatibility.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8905

---

# Render Pipeline Storage Texture Format (8% pass)

Selector: `webgpu:api,validation,render_pipeline:storage_texture,format:*`

Similar to compute pipeline issue.

---

# Render Pipeline Targets Write Mask (50% pass)

Selector: `webgpu:api,validation,render_pipeline,fragment_state:targets_write_mask:*`

Write mask validation at wrong layer - TypeError instead of validation error (4 failures).

---

# Render Pipeline Vertex State (0% pass)

Selector: `webgpu:api,validation,render_pipeline:vertex_state:*`

**Overall Status:** 0% pass (max_vertex_attribute_limit and max_vertex_buffer_limit tests)

**Root cause:** Validation bug in wgpu-core where empty vertex buffers (buffers with no attributes) are not counted toward the `maxVertexBuffers` limit. The validation at `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs:4009` checks `vertex_buffers.len()` which excludes empty buffers (skipped at lines 3968-3970), but should check `vertex.buffers.len()` instead.

**Impact:** Tests with `maxVertexBuffers + 1` where some buffers are empty show "Validation succeeded unexpectedly" because empty buffers aren't counted.

**Fix needed:** Change line 4009 to check `vertex.buffers.len()` instead of `vertex_buffers.len()`.

**Secondary issue:** Internal errors have empty messages in deno_webgpu (`error.rs:186`), causing "undefined" error messages in tests.

See: `docs/cts-triage/render_pipeline_vertex_state.md`

---

# Bitwise Shift Validation

Selector: `webgpu:shader,validation,expression,binary,bitwise_shift:*`

**Overall Status:** 976P/6F/0S (99.39% pass rate)

## 1. Atomic types in shift operations (2 failures)

Selector: `webgpu:shader,validation,expression,binary,bitwise_shift:invalid_types:type="atomic";*`

**Root cause:** Known issue with Naga allowing direct references to atomics in expressions.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/5474

## 2. Partial evaluation errors (4 failures)

Selector: `webgpu:shader,validation,expression,binary,bitwise_shift:partial_eval_errors:*`

**Root cause:** wgpu is not validating shift amounts during constant evaluation when the shift amount comes from an override value or a literal >= bitwidth.

**Related PR:** https://github.com/gfx-rs/wgpu/pull/8907

---

# Builtin Functions dot4I8Packed and dot4U8Packed (0% pass)

Selectors:
- `webgpu:shader,validation,expression,call,builtin,dot4I8Packed:*`
- `webgpu:shader,validation,expression,call,builtin,dot4U8Packed:*`

**Overall Status:** 0% pass

## 1. Missing wgslLanguageFeatures implementation

**What it tests:** Validates the `dot4I8Packed` and `dot4U8Packed` WGSL builtins, which are part of the `packed_4x8_integer_dot_product` language extension.

**Root cause:** Same as "Shader Language Features" section - `deno_webgpu` does not implement `wgslLanguageFeatures`. Tests expect feature to be unsupported (compilation should fail), but Naga has the feature implemented and accepts the builtins.

**Related issue:** PR #8884 (wgslLanguageFeatures implementation)

**See also:** `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_validation_dot4_packed_triage.md` for detailed analysis.

---

# Matrix Expression Validation

Selector: `webgpu:shader,validation,expression,matrix,*`

**Overall Status:** 1676P/205F/0S (89.10% pass rate)

## Remaining Issues (205 failures)

### 1. Matrix Multiplication Constant Evaluation (129 failures in mul)
**Status:** Known limitation - DO NOT FIX

Naga does not support matrix multiplication in constant evaluation.

### 2. Matrix Addition/Subtraction Constant Evaluation (56 failures in add_sub)
**Status:** Known limitation

The constant evaluator's `binary_op` function only handles `Vector + Vector` for Compose types. It does not handle `Matrix + Matrix` constant evaluation.

### 3. Atomic Direct Reference (14 failures total)
**Status:** Known issue #5474 - DO NOT FIX

Naga allows referencing atomics directly instead of requiring atomic functions.

### 4. Incorrect binary operands accepted in constant evaluation

See [#8868](https://github.com/gfx-rs/wgpu/issues/8868).

---

# Shader Language Features (pointer_composite_access, requires)

Selectors:
- `webgpu:shader,validation,extension,pointer_composite_access:pointer:*` (50% pass)
- `webgpu:shader,validation,parse,requires:*` (14% pass: 5P/3F/15S)

**Root Cause:** The deno_webgpu backend does not expose the `wgslLanguageFeatures` API on the `GPU` object.

## 1. Missing wgslLanguageFeatures API in deno_webgpu

**What it tests:** Validates that when language features are supported, shaders can use the corresponding syntax.

**For pointer_composite_access:** ~50% pass (some tests skip, some fail)

**For parse,requires:**
- 3 failures: Tests for `wgsl_matches_api:feature=` with implemented features (`readonly_and_readwrite_storage_textures`, `packed_4x8_integer_dot_product`, `pointer_composite_access`) - CTS expects compilation to fail but Naga accepts the `requires` directive
- 15 skipped: Tests in `requires:requires:*` suite skip because `skipIfLanguageFeatureNotSupported()` sees features as unsupported
- 5 passing: Tests that don't require feature detection

**Error:**
```
EXPECTATION FAILED: Expected validation error
```

**Root cause:** When CTS checks `gpu.wgslLanguageFeatures.has('feature_name')`, it gets `false` (property undefined). Tests expect compilation to fail, but Naga correctly supports these features.

**Fix needed:** Implement `wgslLanguageFeatures` property in `/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs` returning a Set-like object with the three implemented features.

**Related issue:** https://github.com/gfx-rs/wgpu/pull/8884

---

# Shader Context Dependent Resolution

Selector: `webgpu:shader,validation,decl,context_dependent_resolution:*`

**Overall Status:** 60P/1F/4S (92.31% pass rate)

**Root cause:** Naga treats `f16` as a reserved keyword unconditionally. When `enable f16;` is used, the name `f16` should be available as a regular identifier (context-dependent resolution), but Naga's lexer rejects it. The `word_as_ident` function in `naga/src/front/wgsl/parse/lexer.rs:495-500` checks reserved keywords without considering enabled extensions.

**Failing test:** `enable_names:case="f16"` - Tests that `f16` can be used as identifier after `enable f16;`

**Skipped tests:** 4 tests in `language_names` subcategory skip due to missing wgslLanguageFeatures API (#8884).

**Fix needed:** Modify lexer to accept enable extension names as identifiers when those extensions are enabled.

See: `docs/cts-triage/shader_context_dependent_resolution.md`

---

# Shader Override Declarations

Selector: `webgpu:shader,validation,decl,override:*`

**Overall Status:** 64P/3F/0S (95.52% pass rate)

**Affected tests:** `array_size:*` subcategory with pointer parameters

**Failing tests:**
- `case="workgroup_ptr_param";stage="const"`
- `case="workgroup_ptr_param";stage="override"`
- `case="private_ptr_param";stage="override"`

## Issue 1: CTS using unrestricted_pointer_parameters when not available

**Affected tests:** workgroup_ptr_param cases

**What's happening:** Test expects ptr<workgroup> parameters to work, but Naga rejects them as requiring unrestricted_pointer_parameters extension (not implemented, tracked in #5158).

**TODO:** Fix CTS to check for feature support.

## Issue 2: Missing Validation for Override-Sized Arrays in Private Address Space

**Affected tests:** private_ptr_param with stage="override"

**What's happening:** Naga accepts override-sized arrays in private address space. WGSL spec says override-sized arrays only valid in workgroup address space.

**Fix needed:** Add Naga validation to reject override-sized arrays in non-workgroup address spaces.

See: `docs/cts-triage/shader_decl_override.md` for details.

---

# Shader Expression Access Array

Selector: `webgpu:shader,validation,expression,access,array:*`

**Overall Status:** 74P/2F/0S (97.37% pass)

**Runtime indexing incorrectly rejected for compile-time-known out-of-bounds indices**

**Failing tests:**
- `webgpu:shader,validation,expression,access,array:early_eval_errors:case="runtime_oob_neg"`
- `webgpu:shader,validation,expression,access,array:early_eval_errors:case="runtime_oob_pos"`

**Root cause:** Naga's validator in `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs` (lines 292-318) performs constant value extraction on `Access` expressions to check if the index can be evaluated at compile-time. When successful, it performs bounds checking during shader module validation. However, it doesn't distinguish between:
1. **Constant expression contexts** (`const idx = -1; array(1,2,3)[idx]`) - Should reject out-of-bounds at compile-time ✓
2. **Runtime expression contexts** (`let idx = -1; array(1,2,3)[idx]`) - Should allow, as this is runtime indexing ✗

Even though `let idx = -1` has a compile-time-evaluable value, it is **not** a constant expression according to WGSL semantics. The `let` keyword creates a runtime-typed value that happens to be known at compile-time. According to the WebGPU spec, out-of-bounds validation errors should only occur for const-expressions. Runtime out-of-bounds access has undefined behavior but should not cause validation failure.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/4390

---

# Shader Expression Access Matrix

Selector: `webgpu:shader,validation,expression,access,matrix:*`

**Overall Status:** 27P/2F/0S (93.10% pass)

**Runtime out-of-bounds matrix access with literal indices incorrectly rejected**

**Failing tests:** early_eval_errors with runtime_oob_neg and runtime_oob_pos cases

**Root cause:** Naga performs bounds checking on runtime indices with literal values (e.g., `let idx = -1`). Spec requires validation errors only for const-expressions, not runtime expressions with literals. Validator doesn't distinguish const from let.

**Related issue:** #4390

---

# Shader Expression Access Vector

Selector: `webgpu:shader,validation,expression,access,vector:*`

**Overall Status:** 44P/40F/0S (52.38% pass)

## 1. Overly Strict Runtime Indexing Validation

**Problem:** Naga performs compile-time bounds checking on runtime indices. Doesn't distinguish const-expressions (reject OOB) from runtime (allow, clamping applies).

**Impact:** Tests with `let`/`var` and OOB indices fail when they should pass.

**Related issue:** #4390

## 2. Missing Swizzle Validation

**Problem:** Naga doesn't validate swizzle components exist for vector's width. Accepts invalid swizzles (e.g., v.xyz on vec2).

**Pattern:** All 40 failures with vector_width=2 or 3. Tests with vector_width=4 pass.

See: `docs/cts-triage/expression_access_vector.md` for details.

---

# Shader Expression Binary Add/Sub/Mul

Selector: `webgpu:shader,validation,expression,binary,add_sub_mul:*`

**Overall Status:** 859P/45F/0S (95.02% pass)

## 1. u32/i32 Constant Evaluation Incorrectly Rejects Overflow (21 failures)

**Root cause:** Naga uses checked_add/sub/mul and errors on overflow. WGSL spec requires wrapping semantics for integer constant evaluation.

**Related PR:** [#8912](https://github.com/gfx-rs/wgpu/pull/8912)

## 2. f16 Constant Evaluation Doesn't Reject Overflow (21 failures)

**Root cause:** Naga performs f16 arithmetic without overflow checks. Accepts infinity results. WGSL spec requires validation error on float overflow in constant evaluation.

## 3. Atomics Can Be Directly Referenced in Expressions (3 failures)

**Root cause:** Known issue #5474. Naga allows direct atomic references in expressions when should require atomicLoad/atomicStore builtins.

See: `docs/cts-triage/expression_binary_add_sub_mul.md` for details.

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8900, https://github.com/gfx-rs/wgpu/issues/5474

---

# Shader Expression Early Evaluation

Selector: `webgpu:shader,validation,expression,early_evaluation:*`

**Overall Status:** 12P/6F/0S (67% pass rate)

**Root cause:** Incorrect early evaluation of composite expressions that mix override values with runtime values during pipeline creation. When a composite expression (vector, array, struct, matrix) contains BOTH override/const values AND runtime values (like `let` variables), it should NOT be evaluated early. However, Naga is attempting to evaluate these mixed expressions during override processing, causing infinity errors when arithmetic operations overflow.

**Failing pattern:** All 6 failures involve expressions like:
- `vec4(override_1e30) * vec4(vec3(override_1e30), let_1e30)`

Where one operand is pure override (should be evaluated early) and the other mixes override with runtime `let` values (should NOT be evaluated early). The evaluation causes `1e30 * 1e30` which exceeds f32 range, producing infinity errors.

**Location:** The issue is in Naga's override processing (`naga/src/back/pipeline_constants.rs`) and constant evaluator (`naga/src/proc/constant_evaluator.rs`). The expression kind tracking should classify mixed composite expressions as Runtime and skip evaluation, but the current implementation still attempts to evaluate them.

**Fix needed:** Improve expression kind tracking during override processing to properly identify and skip evaluation of composite expressions that mix override with runtime values.

See: `docs/cts-triage/expression_early_evaluation.md`

TODO: bug

---

# Shader Expression Unary

Selector: `webgpu:shader,validation,expression,unary:*`

**Overall Status:** 225P/79F/0S (74% pass rate)

**Root causes:**

1. **Missing wgslLanguageFeatures API (76 failures)** - Tests for `pointer_composite_access` language feature fail because the feature is not advertised through WebGPU API. See Known Issues Reference section.

2. **Atomic Direct Reference (2 failures)** - Naga allows direct atomic references in negation/complement operations instead of requiring atomic builtins. See Known Issues Reference (#5474).

3. **Matrix Negation Accepted (1 failure)** - Naga accepts unary negation on matrix types (e.g., `-m`) but WGSL spec only allows negation on scalar and vector types. TODO: bug

**Failing test:** `arithmetic_negation:expr="unary_minus";type="matrix"`

**Fix needed:** Add validation in Naga's WGSL expression validator to reject unary negation operator on matrix operands.

See: `docs/cts-triage/expression_unary_2.md`

---

# Shader Statement Validation

## For Statement

Selector: `webgpu:shader,validation,statement,for:*`

**Overall Status:** 55P/4F/0S (93% pass rate)

**Root causes:**

1. **Phony assignment not allowed in initializer** (1 failure) - `for (_ = v;;) { break; }` rejected. Naga's initializer validation only allows `Call`, `Assign`, and `LocalDecl`, but needs to also allow `Phony`.

2. **Increment/decrement not allowed in initializer** (1 failure) - `for (v++;;) { break; }` rejected. Needs to allow `Increment` and `Decrement` statement kinds.

3. **Phony assignment in continuation not recognized** (1 failure) - `for (;;_ = v) { break; }` fails with "unknown identifier: `_`". Continuation parsing doesn't recognize `_` as phony assignment token.

4. **Empty for loop not detected as infinite** (1 failure) - `for (;;) {}` should be rejected as obviously infinite loop. Covered by Behavior Analysis Not Implemented (#7650).

**Fix needed:** Update for loop parser in `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs` (lines 2558-2566) to allow `Phony`, `Increment`, and `Decrement` in initializer. Update continuation parsing to handle phony assignments.

See: `docs/cts-triage/shader_statement_for.md`

---

# Shader Builtin Functions

## Atomic Builtin Validation

Selector: `webgpu:shader,validation,expression,call,builtin,atomics:*`

**Overall Status:** 132P/22F/0S (85.71% pass rate)

**Root causes:** Two validation gaps in Naga:

1. **Vertex shader stage validation (11 failures)** - Atomic operations incorrectly accepted in vertex shaders. Per WGSL spec, atomic builtin functions must not be used in vertex shader stages, but wgpu accepts them.

2. **Address space and access mode validation (11 failures)** - Atomics incorrectly accepted in invalid configurations:
   - Storage atomics with `read` access (should require `read_write`)
   - Atomics in `private` address space (only `storage` and `workgroup` are valid)
   - Atomics in `function` address space (only `storage` and `workgroup` are valid)

**Note:** These failures are distinct from issue #5474 (direct atomic references). These are about missing validation when atomic builtin functions are used in invalid contexts.

See: `docs/cts-triage/builtin_atomics.md`

---

## `bitcast()`: Multiple Validation Issues

Selector: `webgpu:shader,validation,expression,call,builtin,bitcast:*`

**Overall Status:** 138P/106F/0S (56.56% pass rate)

**Root causes:**

1. **Bitcast not implemented in constant evaluator (32 failures)** - Tests expect rejection of const bitcasts producing NaN/infinity. Already tracked in Known Issues Reference (#4507). Location: `naga/src/proc/constant_evaluator.rs:1320`.

2. **Missing size validation (66 failures)** - Naga accepts bitcasts between types of different bit widths (e.g., `vec3<f16>` ↔ `u32`). Should reject mismatched sizes. Location: `naga/src/valid/expression.rs:1154-1174`.

3. **Incorrect f16 vector validation (8 failures)** - Naga incorrectly rejects valid f16 vector bitcasts: `vec2<f16>` ↔ 32-bit types, `vec4<f16>` ↔ `vec2<32-bit>`.

**Fix needed:** Implement constant evaluation for bitcast. Add size validation checking total bit width matches. Fix f16 vector size calculations.

See: `docs/cts-triage/builtin_bitcast.md`

---

## `asinh()`: Large f32 values produce infinity

Selector: `webgpu:shader,validation,expression,call,builtin,asinh:*`

**Overall Status:** 46P/8F/0S (85.19% pass rate)

**Failing Tests (8 failures):**
- `values:stage="constant";type="f32"` and vec2/vec3/vec4 variants (4 failures)
- `values:stage="override";type="f32"` and vec2/vec3/vec4 variants (4 failures)

**Root cause:** Naga's constant evaluator produces infinite results when evaluating `asinh()` for large f32 values near f32::MAX (±3.4e38). The mathematical result `asinh(3.4e38) ≈ 88.72` is representable as f32, but the implementation uses Rust's `f32::asinh()` which returns infinity due to intermediate overflow in calculating `sqrt(x² + 1)`.

**Fix needed:** Implement numerically stable `asinh()` using approximation `asinh(x) ≈ sign(x) * ln(2|x|)` for large |x|.

See: `docs/cts-triage/builtin_asinh.md`

---

## `clamp()`: Constraint Validation

Selector: `webgpu:shader,validation,expression,call,builtin,clamp:*`

**Overall Status:** 161P/65F/0S (71.24% pass rate)

**Root cause:** Naga only validates the `low <= high` constraint when the entire clamp expression can be fully constant-evaluated (all three arguments are constants). The WebGPU spec requires validation when only `low` and `high` are const-expressions or override-expressions, even if the first argument `e` is a runtime value.

**Failing pattern:** Tests where both `lowStage` and `highStage` are `"constant"` or `"override"` (64 failures in `low_high:*` subcategory). Tests where at least one is `"runtime"` pass.

**Fix needed:** Add partial evaluation logic for builtin parameter constraints, or a separate validation pass that checks clamp calls with const/override low/high parameters independently of full constant evaluation.

See: `docs/cts-triage/builtin_clamp.md`

TODO: file bug

---

## `derivative()`: f16 Type Validation

Selectors:
- `webgpu:shader,validation,expression,call,builtin,derivatives:invalid_argument_types:type="f16";*`
- `webgpu:shader,validation,expression,call,builtin,derivatives:invalid_argument_types:type="vec2%3Cf16%3E";*`
- `webgpu:shader,validation,expression,call,builtin,derivatives:invalid_argument_types:type="vec3%3Cf16%3E";*`
- `webgpu:shader,validation,expression,call,builtin,derivatives:invalid_argument_types:type="vec4%3Cf16%3E";*`

**Overall Status:** 182P/36F/0S (83.49%/16.51%/0%)

**What these tests validate:** Derivative builtins (dpdx, dpdy, fwidth, etc.) only accept f32 scalar/vector types. Tests verify f16 types rejected.

**Root cause:** Naga's derivative validation checks for float types but doesn't validate width. Accepts any float (f16, f32, f64) when should only accept f32 (width 4).

**Fix needed:** Update validation pattern match at naga/src/valid/expression.rs:1037-1052 to check both kind: Sk::Float AND width: 4.

See: `docs/cts-triage/bulitin_derivative.md` for details.

## `distance()`: 1D distance calculation error

Selector: `webgpu:shader,validation,expression,call,builtin,distance:*`

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,distance:values:stage="constant";type="f32"
Subcase: a=-3.4028234663852886e+38;b=-11728123330560
```

**Error:**
```
Shader '' parsing error: Float literal is infinite
  ┌─ wgsl:2:12
  │
2 │ const v  = distance(-3.4028234663852886e+38f, -11728123330560.0f);
  │            ^^^^^^^^ see msg
```

**Root cause:**
Naga's constant evaluator computes `distance(a, b)` for scalars using the generic formula `sqrt((a-b)²)`, which causes overflow to infinity even when `a-b` is representable. 

For the failing case above:
- `a - b = -3.4028235e38 - (-1.17e13) ≈ -3.4028235e38` (finite, representable in f32)
- `(a-b)² = 1.158e77` (overflows f32 max of 3.4e38 → **infinity**)
- `sqrt(infinity) = infinity` (rejected by Naga's literal validator)

According to the WGSL spec, for scalars, `distance(a, b)` should be equivalent to `abs(a - b)`, which would give a finite result. The test expects this case to succeed since the result is representable.

## `insertBits()`: Override Expression Validation

Selector: `webgpu:shader,validation,expression,call,builtin,insertBits:*`

**Overall Status:** 85P/32F/0S (72.65% pass rate)

**Root causes:**

1. **Missing constant evaluation support** (~20 failures) - Already tracked in Known Issues Reference #4507. The `insertBits` builtin is not implemented in Naga's constant evaluator (`naga/src/proc/constant_evaluator.rs:1861`).

2. **Missing override expression validation** (4 failures in `count_offset:*`) - When `offset + count > 32` and either offset or count are override expressions (not runtime variables), wgpu should produce a pipeline creation error but doesn't. The WebGPU spec requires validation at pipeline creation time when override values can be evaluated. Affects cases where offset=33, count=33, or offset+count>32.

3. **Atomic direct reference** (some `typed_arguments:*` failures) - Already tracked in Known Issues Reference #5474. Naga incorrectly allows atomics to be used directly as builtin arguments.

**Fix needed:** Add pipeline creation validation for `insertBits` to check that `offset + count <= 32` when using override expressions.

See: `docs/cts-triage/builtin_insertBits.md`

---

## textureLoad()

Selector: `webgpu:shader,validation,expression,call,builtin,textureLoad:*`

**Overall Status:** 988P/5F/0S (99.50% pass rate)

**Root cause:** All 5 failures are related to `texture_external` support. Tests using `texture_external` fail because it requires the `TEXTURE_EXTERNAL` capability which is not being properly enabled. The error message indicates "Capability TEXTURE_EXTERNAL is required".

**Failing tests:** All failures involve `texture_external` texture type:
- `return_type,non_storage` with `texture_external` (1 failure)
- `texture_type,non_storage` with `texture_external` (1 failure)
- `texture_type,storage` with `texture_external` (1 failure)
- `coords_argument,non_storage` with `texture_external` (3 failures with different coord types)

**Fix needed:** The tests need to either enable the appropriate capability directive for `texture_external` in WGSL or skip tests when the device does not support the `texture_external` capability. This is related to the broader issue that `EXTERNAL_TEXTURE` is classified as `FeaturesWGPU` (native-only) rather than `FeaturesWebGPU`.

---

## textureSample()

Selectors:
- `webgpu:shader,validation,expression,call,builtin,textureSample:*`
- `webgpu:shader,validation,expression,call,builtin,textureSampleBias:*`
- `webgpu:shader,validation,expression,call,builtin,textureSampleCompare:*`
- `webgpu:shader,validation,expression,call,builtin,textureSampleCompareLevel:*`
- `webgpu:shader,validation,expression,call,builtin,textureSampleLevel:*`

1. Offset validation (10+ failures) - Naga is not properly validating that offset values are within the required range [-8, +7]. This affects `textureSampleBias`, `textureSampleCompare`, `textureSampleCompareLevel`, and `textureSampleLevel`.
2. Texture type mismatches with offsets (2 failures) - Cube textures used with 3D parameters and offset=true
3. must_use validation (1 failure) - Not enforcing that builtin function results must be used

## textureSampleBaseClampToEdge()

Selector: `webgpu:shader,validation,expression,call,builtin,textureSampleBaseClampToEdge:*`

**Overall Status:** 137P/5F/0S (96.48% pass rate)

**Root cause:** All failures are due to `texture_external` not being exposed in deno_webgpu. The `EXTERNAL_TEXTURE` feature is classified as `FeaturesWGPU` (native-only) rather than `FeaturesWebGPU`, so it's filtered out when exposing features to JavaScript. Without the feature enabled, Naga rejects shaders containing `texture_external` with "Capability TEXTURE_EXTERNAL is required".

**Failing tests:** All 5 failures involve `texture_external` texture type. Tests with `texture_2d<f32>` pass (137/137 = 100%).

**Fix needed:** Move `EXTERNAL_TEXTURE` from `FeaturesWGPU` to `FeaturesWebGPU` or add special handling in deno_webgpu to expose this feature.

See: `docs/cts-triage/builtin_textureSampleBaseClampToEdge.md`

---

## textureGather()

Selector: `webgpu:shader,validation,expression,call,builtin,textureGather:*`

**Overall Status:** ~99% pass rate

**Root cause:** Minimal failures (~1%) likely due to must_use validation infrastructure issue (#8876). Naga implements comprehensive validation for textureGather including component parameter validation, dimension restrictions, const-expression requirements, and offset range validation.

See: `docs/cts-triage/builtin_textureGather.md`

---

## textureSampleGrad()

Selector: `webgpu:shader,validation,expression,call,builtin,textureSampleGrad:*`

**Overall Status:** 839P/12F/0S (98.59% pass rate)

**Root causes:**

1. **Missing offset range validation** (6 failures) - Naga validates that offset is a const-expression with correct type but doesn't check values are in required range [-8, 7]. Tests with offset values -9 or 8 are incorrectly accepted. Location: `naga/src/valid/expression.rs:509-525`.

2. **Depth textures incorrectly accepted** (6 failures) - Naga accepts depth textures with textureSampleGrad when only sampled textures should be allowed. The spec restricts textureSampleGrad to `texture_2d<f32>`, `texture_2d_array<f32>`, `texture_3d<f32>`, `texture_cube<f32>`, and `texture_cube_array<f32>`. Location: `naga/src/valid/expression.rs:660-692`.

**Fix needed:** Add offset value range validation and image class check to reject depth textures in the Gradient sample level validation.

See: `docs/cts-triage/builtin_textureSampleGrad.md`

---

## Value Constructors

Selector: `webgpu:shader,validation,expression,call,builtin,value_constructor:*`

**Overall Status:** 476P/7F (98.55% pass rate)

### 1. Value constructors lack must-use validation (5 failures)

**Affected tests:** `must_use:*` subcategory (5/78 failing)

**Root cause:** In `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/lower/mod.rs`, the `call()` function (lines 3565-3573) handles type constructors but does not enforce must-use semantics when they are used as statements. According to WGSL spec, value constructors (like struct constructors, matrix constructors, etc.) must have their results used - they cannot be standalone statements.

**Current behavior:** Naga accepts type constructors as standalone statements without their result being used.

**Expected behavior:** These should fail when `use=false`.

**Additional issue:** Abstract vectors in matrix constructors are not properly type-resolved, causing valid code to be rejected with "Matrix elements must always be floating-point types" error.

### 2. Abstract-float matrix constructor validation gaps (2 failures)

Selectors:
- `webgpu:shader,validation,expression,call,builtin,value_constructor:matrix_column:type1="abstract-float";type2="abstract-float"`
- `webgpu:shader,validation,expression,call,builtin,value_constructor:matrix_elementwise:type1="abstract-float";type2="abstract-float"`

**Root cause:** When constructing matrices with abstract-float vectors or scalars, Naga is not properly validating that dimensions match (validation gap - accepts invalid code).

**Current behavior:** Naga accepts matrix constructors with mismatched dimensions or wrong element count when using abstract-float types.

**Expected behavior:** Matrix constructors should validate that column dimensions match the matrix type and that the correct number of elements are provided.

---

# Shader Parse Blankspace

Selector: `webgpu:shader,validation,parse,blankspace:*`

**Overall Status:** 18P/1F/0S (94.74% pass rate)

**Root cause:** Naga's lexer does not validate null characters inside comments. While null characters are correctly rejected in code and at line boundaries, they are accepted when they appear inside comment text (e.g., `// comment with \0 character`). The lexer consumes comments as `Token::Trivia` without content validation.

**Failing test:** `null_characters:contains_null=true;placement="comment"` - Tests that null characters inside comments are rejected per WGSL spec.

**Fix needed:** Add null character validation during comment tokenization in `naga/src/front/wgsl/parse/lexer.rs:87-155`.

See: `docs/cts-triage/shader_parse_blankspace.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8877

---

# Shader Parse Comments

Selector: `webgpu:shader,validation,parse,comments:*`

**Overall Status:** 13P/1F/0S (92.86% pass rate)

**Root cause:** Naga's lexer does not detect unterminated block comments. When a block comment starting with `/*` is not properly closed with `*/`, the lexer returns `Token::End` instead of an error. This makes Naga accept invalid WGSL that should be rejected.

**Failing test:** `unterminated_block_comment:terminated=false` - Tests that shaders with unclosed `/*` comments are rejected.

**Fix needed:** Update lexer in `naga/src/front/wgsl/parse/lexer.rs:106-154` to detect when block comment depth is non-zero at end of input and return an error (e.g., `UnterminatedBlockComment`).

See: `docs/cts-triage/shader_parse_comments.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8877

---

# Shader Functions Restrictions

Selector: `webgpu:shader,validation,functions,restrictions:*`

**Overall Status:** 98% pass (if external texture is enabled)

**What it tests:** Validates that attributes can only be applied in appropriate contexts (function declaration, parameter, or return value).

## 1. Invalid function attributes accepted

**Root cause:** Naga accepts `@id` and `@workgroup_size` attributes on regular functions when they should only be valid on specific contexts:
- `@id` should only be valid on override declarations (not functions at all)
- `@workgroup_size` should only be valid on compute entry point functions (functions with `@compute`)

## 2. Unrestricted pointer parameters accepted without feature

**What it tests:** Validates that certain pointer expressions (pointers to struct members, array elements within structs) require the `unrestricted_pointer_parameters` WGSL feature to be enabled.

**Root cause:** Without the `unrestricted_pointer_parameters` feature enabled, Naga should reject complex pointer expressions like `&f_constructible.b` or `&f_struct_with_array.a[1].b`, but currently accepts them.

See: `docs/cts-triage/shader_functions_restrictions.md` for details.

---

# `@group` attribute

Selectors:
- `webgpu:shader,validation,parse,attribute:expressions:value="val";attribute="group"`
- `webgpu:shader,validation,parse,attribute:expressions:value="expr";attribute="group"`
- `webgpu:shader,validation,parse,attribute:expressions:value="const";attribute="group"`

**What it tests:** Validates that WGSL attributes can accept various expression types in their parameters.

**Root cause:** wgpu validates bind group indices against device limits (`max_bind_groups = 8`) during shader module creation at wgpu-core/src/device/resource.rs:2265-2276. The test generates shaders with `@group(32)` or `@group(30 + 2)` or `@group(8)` which are syntactically valid WGSL.

**Why this is incorrect:** Per WebGPU spec, `createShaderModule` should only perform WGSL language validation (syntax and semantic analysis), NOT validate against device limits. Device limit validation should occur during pipeline/bind group creation when the shader is actually used.

**Fix needed:** Remove the group index validation loop (lines 2265-2276) from `create_shader_module`. Move validation to pipeline creation functions where shader requirements are checked against available resources.

See: `docs/cts-triage/shader_parse_group.md` for detailed analysis.

---

# `requires` directive

Selector: `webgpu:shader,validation,parse,requires:*`

Missing `wgslLanguageFeatures` support. (#8884)

---

# Phony Assignment Statement Validation

Selector: `webgpu:shader,validation,statement,phony:*`

**Overall Status:** 56P/6F/62S (90.3%/9.7%/100%)

## 1. Phony assignment in for-loops with semicolons

**What it tests:** WGSL phony assignments (`_ = expr`) are special statements that discard values. The spec defines specific syntax rules for where they can appear, including within for-loops.

**Test suite breakdown:**
- `rhs_constructible`: Tests that phony assignment RHS can be constructible types (17 tests)
- `rhs_with_decl`: Tests phony assignment RHS with various declarations (23 tests)
- `parse`: Tests parsing rules for phony assignment syntax (21 tests)
- `module_scope`: Tests that phony assignment is rejected at module scope (1 test)

**Known failures (approximately 6 tests):**
The primary failures are related to phony assignment syntax in for-loops:
- `in_for_init_semi`: `for (_ = v;;false;) {}` - Should fail (extra semicolon), but may be incorrectly accepted
- `in_for_update_semi`: `for (;false; _ = v;) {}` - Should fail (extra semicolon), but may be incorrectly accepted

Additionally, some tests may fail due to:
- Atomic type validation in phony assignments
- Unsized array type validation
- Parse error messages not matching expected patterns

**Root cause:** Naga's WGSL parser may not correctly enforce the syntax rules for phony assignments in for-loop init and update clauses, specifically around semicolon placement.

**Related work:**
- Issue #7524: Fixed phony assignments not being included in bind group layouts (resolved in PR #7540)
- PR #6328: Fixed handling of phony statements so they are actually emitted

**References:**
- WebGPU Spec: https://www.w3.org/TR/WGSL/#phony-assignment-section
- Phony assignments are used to discard @must_use function results or include unused bindings in shader interfaces

---

# Shader IO Align

Selector: `webgpu:shader,validation,shader_io,align:*`

**Overall Status:** 296P/1F/4S (98.32% pass rate)

**Root cause:** Trailing comma not accepted in `@align` attribute. Parser rejects `@align(4,)` syntax even though WGSL spec allows trailing commas in attribute argument lists.

**Fix needed:** Update Naga's attribute parser to accept optional trailing comma before closing parenthesis.

See: `docs/cts-triage/shader_io_align.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8892

---

# Shader IO Binding

Selector: `webgpu:shader,validation,shader_io,binding:*`

**Overall Status:** 20P/1F/0S (95.24% pass rate)

**Root cause:** Trailing comma not accepted in `@binding` attribute. Parser rejects `@binding(1,)` syntax even though WGSL spec allows trailing commas in attribute argument lists. Related to issue #8892.

**Fix needed:** Update Naga's attribute parser to accept optional trailing comma before closing parenthesis.

See: `docs/cts-triage/shader_io_binding.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8892

---

# Shader IO Builtins

Selector: `webgpu:shader,validation,shader_io,builtins:*`

**Overall Status:** 356P/1F/357S (49.9%/0.14%/50%)

## 1. Trailing comma in @builtin() not accepted

**What it tests:** WGSL syntax allows trailing commas in various contexts.

**Error:** A shader with `@builtin(position,)` (trailing comma) is rejected when it should be accepted.

**Root cause:** Naga's WGSL parser does not accept trailing commas inside `@builtin()` attribute.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8892

---

# Shader IO Group

Selector: `webgpu:shader,validation,shader_io,group:*`

**Overall Status:** 20P/1F/0S (95% pass rate)

**Root cause:** Naga's WGSL parser does not accept trailing commas inside `@group()` attribute parameter lists. The WGSL spec allows optional trailing commas in attribute parameter lists.

**Failing test:** `attr="trailing_comma"` - Tests that `@group(1,)` with trailing comma is accepted.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/6394

**Fix needed:** Update Naga's attribute parser to accept optional trailing comma before closing parenthesis in `@group()` attributes.

See: `docs/cts-triage/shader_io_group.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8892

---

# Shader IO Group and Bindings

Selector: `webgpu:shader,validation,shader_io,group_and_binding:*`

External texture related. All pass with external texture support enabled in Deno.

---

# Shader IO ID

Selector: `webgpu:shader,validation,shader_io,id:*`

**Overall Status:** 30P/2F/0S (93.75% pass rate)

**Root causes:** Two validation/parser issues:

1. **Trailing comma not accepted (1 failure)** - Parser rejects `@id(1,)` but WGSL spec allows trailing commas. Part of broader trailing comma issue (#8892).

2. **@id accepted on const declarations (1 failure)** - Naga incorrectly accepts `@id(1) const a = 4;` when @id should only be valid on `override` declarations. Parser collects @id but doesn't validate it's only used with override.

See: `docs/cts-triage/shader_io_id.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8892, https://github.com/gfx-rs/wgpu/issues/8898

---

# Shader IO Interpolate

Selector: `webgpu:shader,validation,shader_io,interpolate:*`

**Overall Status:** 91% pass rate

For detailed triage information, see [shader_io_interpolate_triage.md](shader_io_interpolate_triage.md).

The interpolate tests validate the `@interpolate` attribute for shader input/output variables. The attribute controls how values are interpolated between shader stages (flat, perspective, linear) and where sampling occurs (center, centroid, sample, first, either).

Tests cover:
- Valid/invalid combinations of interpolation type and sampling mode
- Requirement that @interpolate only works with @location (not @builtin)
- Requirement that integer types must use @interpolate(flat)
- Prevention of duplicate @interpolate attributes
- Syntax validation including trailing commas

With a 91% pass rate, most interpolate validation is working correctly. Failures likely include:
- Trailing comma support in attribute arguments
- Possible edge cases in syntax parsing or error reporting

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8892

---

# Shader IO Layout Constraints

Selector: `webgpu:shader,validation,shader_io,layout_constraints:*`

**Overall Status:** 100P/1F/0S (99.01% pass rate)

**Root cause:** Naga's layouter doesn't account for `@align` attributes on struct members when computing struct alignment. When a nested struct member has `@align(16)` but the member type is `u32`, the layouter uses alignment 4 instead of 16, causing incorrect offset calculations in the containing struct.

**Failing test:** `layout_constraints:case="struct_size_5_align16"` - Struct with `@align(16) @size(5) x : u32` member incorrectly gets alignment 4.

**Fix needed:** Update layouter in `naga/src/proc/layouter.rs` to either infer struct alignment from member offsets or store alignment in `TypeInner::Struct`.

See: `docs/cts-triage/shader_io_layout_constraints.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/4377

---

# Shader IO Locations

Selector: `webgpu:shader,validation,shader_io,locations:*`

**Overall Status:** 98% pass rate

**Root cause:** Naga's WGSL parser does not accept trailing commas inside `@location()` attribute parameter lists. The WGSL spec allows optional trailing commas in attribute parameter lists.

**Failing test:** `validation:attr="extra_comma"` - Tests that `@location(1,)` with trailing comma is accepted.

**Fix needed:** Update Naga's attribute parser to accept optional trailing comma before closing parenthesis.

See: `docs/cts-triage/shader_io_locations.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8892

---

# Shader IO Pipeline Stage

Selector: `webgpu:shader,validation,shader_io,pipeline_stage:*`

**Overall Status:** 67P/6F/0S (91.78% pass rate)

**Failing Tests:** `placement:*` - 18/24 pass (6 failures)

**Root cause:** Stage attributes (`@vertex`, `@fragment`, `@compute`) are incorrectly accepted on variable declarations. Naga's parser silently ignores stage attributes on `var<private>` and `var<storage>` declarations instead of rejecting them.

**Affected cases:**
- `scope="private-var";attr="@vertex"/"@fragment"/"@compute"` (3 failures)
- `scope="storage-var";attr="@vertex"/"@fragment"/"@compute"` (3 failures)

**Fix needed:** In `naga/src/front/wgsl/parse/mod.rs`, add validation after attribute parsing to ensure stage attributes are only present on function declarations, not on var/struct/alias/const/override/const_assert.

See: `docs/cts-triage/shader_io_pipeline_stage.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8898

---

# Shader IO Size

Selector: `webgpu:shader,validation,shader_io,size:*`

**Overall Status:** 33P/3F/0S (91.67% pass rate)

**Root causes:** Three validation/parser issues:

1. **Trailing comma not supported (2 failures)** - Parser rejects valid WGSL syntax `@size(4,)` with trailing comma. Fix needed in attribute parser to accept optional trailing comma.

2. **Large size value rejected (1 failure)** - `@size(2147483647)` (2GB) rejected because it exceeds wgpu's MAX_TYPE_SIZE limit of 1GB. Requires investigation if spec allows 2GB sizes.

3. **Missing validation for runtime-sized arrays (1 failure)** - `@size` incorrectly accepted on runtime-sized arrays (`array<f32>` without size). Should be rejected since runtime-sized arrays don't have creation-fixed footprint.

See: `docs/cts-triage/shader_io_size.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8892, https://github.com/gfx-rs/wgpu/issues/8898

---

# Shader Validation Types

Selector: `webgpu:shader,validation,types,*`

**Overall Status:** 86% pass rate (1526 total tests covering 10 type areas: alias, array, atomics, enumerant, matrix, pointer, ref, struct, textures, vector)

## Failure Patterns

### 1. Trailing commas in template argument lists (~100 failures)

**Root cause:** Naga doesn't accept trailing commas in template argument lists (e.g., `vec3<u32,>`, `mat2x2<f32,>`, `array<u32,4,>`). WebGPU spec's `template_arg_comma_list` production explicitly allows optional trailing comma.

### 2. f16 usage without enable directive (9 failures)

**Root cause:** Naga doesn't fully implement `enable` directive checking. Using f16 without `enable f16;` should produce validation error. Related issues: #5476, #4384.

**Expected after fixes:** 94.6% pass rate (~1445/1526 tests passing)

See: `docs/cts-triage/shader_validation_types.md` for details.

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8899, TODO

---

# Shader Validation Variables

Selector: `webgpu:shader,validation,variables:*`

**Overall Status:** 773P/17F/3S (97.48% pass rate)

**Root causes:**

1. **Trailing commas in address space/access mode (9 failures)** - Parser rejects valid WGSL syntax like `var<private,>` with trailing comma in template arguments. Affects `address_space_access_mode:*` (7 failures) and `var_access_mode_bad_other_template_contents:*` (2 failures).

2. **Atomic types in read-only storage (4 failures)** - Naga incorrectly accepts atomic types in `var<storage, read>` declarations. WebGPU spec requires atomics only in read-write storage. Affects `module_scope_types:*` tests with atomic types and read access mode.

3. **Storage textures in vertex shaders (4 failures)** - wgpu accepts write-only and read-write storage textures in vertex shaders, but WebGPU spec prohibits this. Affects `shader_stage:*` tests. Same validation gap as in createBindGroupLayout tests.

**Fix needed:** Add parser support for trailing commas in var template arguments. Add validation to reject atomics in read-only storage. Add validation to reject storage textures in vertex shader stage.

See: `docs/cts-triage/shader_validation_variables.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8899, TODO

---

# Shader Uniformity

Selector: `webgpu:shader,validation,uniformity:*`

**Overall Status:** Most tests fail

## 1. Fragment uniformity analysis intentionally disabled

**Root cause:** Fragment shader uniformity analysis is intentionally disabled in wgpu/Naga.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/4369

---

# createBindGroup Buffer Resource State

Selector: `webgpu:api,validation,createBindGroup:buffer,resource_state:*`

**Overall Status:** 0% pass (state="destroyed" subcases)

## 1. Destroyed buffer validation timing

**Root cause:** wgpu reports destroyed buffer errors at different times than the WebGPU spec requires.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7881

---

# Query Set Creation with Zero Count

Selector: `webgpu:api,validation,query_set,create:count:*`

**Overall Status:** 0% pass (0/2 tests passing)

## 1. wgpu incorrectly rejects zero-count query sets

**Root cause:** wgpu's query set validation incorrectly rejects query sets with `count=0`. The validation check at `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs:4783-4784` returns a `ZeroCount` error when `desc.count == 0`.

```rust
if desc.count == 0 {
    return Err(Error::ZeroCount);
}
```

**Expected behavior:** Per the CTS tests, query sets with zero queries should be valid. The only count constraint is that `count <= kMaxQueryCount` (4096).

**Actual behavior:** wgpu rejects query set creation with error: "QuerySets cannot be made with zero queries"

**Test cases:**
- ❌ `count=0` - should be valid, currently fails with validation error
- ❌ `count=4096` (kMaxQueryCount) - should be valid, currently fails due to test failure on count=0
- Expected to pass: `count=4097` (kMaxQueryCount+1) - should be invalid

**Fix needed:** Remove the zero-count validation check at lines 4783-4785 in `wgpu-core/src/device/resource.rs`. Also update documentation at `wgpu-types/src/lib.rs:470` that states "Must not be zero".

**Error definition location:** `/Users/Andy/Development/wgpu2/wgpu-core/src/resource.rs:2111-2112` (ZeroCount error variant may be removed if no longer used)

**Impact:** Low - affects edge case of creating query sets with zero queries, which is uncommon but should be spec-compliant.

TODO: file bug (but very minor)

---

