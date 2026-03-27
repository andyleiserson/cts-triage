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

## Destroyed Resource Validation Error Timing ([#7881](https://github.com/gfx-rs/wgpu/issues/7881))

Tests that check for validation errors when a destroyed resource is used. `wgpu` often reports these errors later than WebGPU requires, causing the tests to fail.

Selectors:
- `webgpu:api,validation,createBindGroup:texture,resource_state:*`
- `webgpu:api,validation,encoding,cmds,setBindGroup:*` (state="destroyed" subcases)

## Behavior Analysis Not Implemented ([#7650](https://github.com/gfx-rs/wgpu/issues/7650))

Naga does not implement the behavior analysis algorithm required by WebGPU.

Selectors:
- `webgpu:shader,validation,statement,statement_behavior:*`
- `webgpu:shader,validation,statement,loop:*`
- `webgpu:shader,validation,statement,switch:*`
- `webgpu:shader,validation,statement,continue:*`

## Constant Evaluation Missing Domain/Range/Overflow Validation ([#8900](https://github.com/gfx-rs/wgpu/issues/8900))

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

## Functions Missing Constant Evaluation Support ([#4507](https://github.com/gfx-rs/wgpu/issues/4507))

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

**Current behavior**: wgpu-core correctly detects when a feature is missing (via `Device::require_features()` and `TextureFormat::required_features()`), but the error is classified as a validation error. In deno\_webgpu, these validation errors are pushed to the error handler rather than being thrown as TypeError exceptions.

**Expected behavior**: Per WebGPU spec, these should throw TypeError synchronously when the API call is made, not generate validation errors.

**Fix needed:** Modify deno\_webgpu to intercept `MissingFeatures` errors and throw TypeError. Implement TEXTURE_COMPONENT_SWIZZLE feature in wgpu-types/features.rs.

See: `docs/cts-triage/capability_checks_features.md` for detailed analysis.

**Related issue:** <https://bugzilla.mozilla.org/show_bug.cgi?id=1917253>

## Limits (65% pass)

Selector: `webgpu:api,validation,capability_checks,limits,*`

**Overall Status:** 5100P/2453F/312S (65%/31%/4%)

**Root causes:**
1. **Missing Limit: maxBindGroupsPlusVertexBuffers** (~40 failures) - Limit not exposed by wgpu
2. **Missing Limits: Per-Stage Resource Limits** (~487 failures) - Four per-stage limits not implemented: maxStorageBuffersInFragmentStage, maxStorageBuffersInVertexStage, maxStorageTexturesInFragmentStage, maxStorageTexturesInVertexStage
3. **Device Creation Validation Gap: Over-Limit Requests** (~700+ failures) - No validation when requested limits exceed adapter maximum
4. **Early Validation: maxColorAttachmentBytesPerSample** (~54 failures) - wgpu validates multisampling format support at pipeline creation, but spec only requires it at texture creation
5. **Workgroup Storage Size Validation Gap** (~222 failures) - No validation of maxComputeWorkgroupStorageSize at pipeline creation
6. **Incorrect Limit Value: maxInterStageShaderVariables** (~208 failures) - wgpu reports 15 instead of spec-required 16

See: `docs/cts-triage/capability_checks_limits.md` for detailed analysis.

**Related issues:**
- [ ] https://github.com/gfx-rs/wgpu/issues/8947
- [ ] https://github.com/gfx-rs/wgpu/issues/8832
- [ ] https://github.com/gfx-rs/wgpu/issues/8748
- [ ] https://github.com/gfx-rs/wgpu/issues/8983
- [ ] https://github.com/gfx-rs/wgpu/issues/8986
- [ ] https://github.com/gfx-rs/wgpu/issues/8946
- [ ] https://github.com/gfx-rs/wgpu/issues/8945

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8946

## Override Workgroup Size Limits (0% pass)

Selector: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*`

Missing re-validation of workgroup size (dimensions, not storage) limits after pipeline constants applied (2 failures).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8946

---

# CreateBindGroup Validation

## Buffer Resource State (0% pass)

Selector: `webgpu:api,validation,createBindGroup:buffer,resource_state:*`

Destroyed buffer validation timing - known issue #7881 (1 failure).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7881

---

# CreateBindGroupLayout Validation

## Max Resources Per Stage (0% pass)

Selectors:
- `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_bind_group_layout:*`
- `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_pipeline_layout:*`

### 1. Incorrect calculation of binding counts

Logic bug: using `max()` instead of `+=` when merging binding counts (11 failures).

**Fix:** Change `merge()` method in `binding_model.rs` from using `max()` to `+=`.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8993

### 2. `GPUSupportedLimits.maxStorage{Buffers,Textures}In{Vertex,Fragment}Stage` are unrecognized

These limits have not been implemented.

One signature of failures due to the missing limits is: `TypeError: Failed to
execute 'call' on 'GPUDevice': 'binding' of 'GPUBindGroupLayoutEntry'
('entries' of 'GPUBindGroupLayoutDescriptor' (Argument 0), index 0) is not a
finite number`.

**Related issue:** : https://github.com/gfx-rs/wgpu/issues/8748

### 3. `Binding index 1000 is greater than the maximum number 1000`

All of the `in_pipeline_layout` tests fail on Vulkan with this symptom, and all
but the sampler `in_pipeline_layout` tests fail on dx12 with this symptom. This
may or may not be due to the missing limit support.

**Related PR:** https://github.com/gfx-rs/wgpu/pull/9118

### 4. Claimed buffer limits are not supported on Metal

We report limits on Metal that we do not actually support.

The symptom of this is `[ERROR wgpu_hal::metal::device] Resource limit
exceeded: StageInfo { stage: Compute, counters: ResourceData { buffers: 32,
textures: 0, samplers: 0 }, ...`.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8173

---

# CreateView Validation (0% pass)

Selector: `webgpu:api,validation,createView:texture_state:*`

Destroyed texture validation timing - known issue #7881 (1 failure).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7881

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7881

## Render Bundle (81% pass)

Selector: `webgpu:api,validation,encoding,render_bundle:*`

**Overall Status:** 81% pass

### 1. Readonly flag normalization mismatch

**What it tests:** Validates render bundle compatibility with render passes, including depth/stencil readonly flags.

**Root cause:** When a render bundle is created with certain readonly depth/stencil configurations, the flags are not being normalized consistently between the bundle encoder and the render pass. This causes compatibility checks to fail when they should pass, or vice versa.

**Likely related issue:** https://github.com/gfx-rs/wgpu/issues/8030

---

# Non-Filterable Textures

Selector: `webgpu:api,validation,non_filterable_texture:*`

**Overall Status:** 80% pass

Missing validation: depth textures with filtering samplers should be rejected (32 failures).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8513

---

# Queue Destroyed

Selector: `webgpu:api,validation,queue,destroyed,*`

**Overall Status:** 71% pass

## 1. writeBuffer/writeTexture return value issue

Tests expect `writeBuffer` and `writeTexture` to return `undefined`, but the implementation may be returning something else or throwing when it shouldn't.

**Related PR:** <https://github.com/denoland/deno_core/pull/1307> (This is a Deno PR, which has landed in Deno, but is not present in the version of deno used by deno\_webgpu in the wgpu tree.)

## 2. Destroyed query set validation missing

**Root cause:** wgpu reports errors related to destroyed query sets at different times than the WebGPU spec requires.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7881

---

# Queue writeBuffer Ranges

Selector: `webgpu:api,validation,queue,writeBuffer:ranges:*`

**Overall Status:** 0% pass

## 1. Missing OperationError for invalid data ranges

**What it tests:** When `writeBuffer` is called with invalid data ranges (e.g., data offset + size exceeds ArrayBuffer bounds), it should throw an `OperationError`.

**Root cause:** deno\_webgpu is not validating the data source bounds properly or not returning the correct error type when validation fails. `wgpu` also fails to raise a validation error for zero-size transfers beyond the end of the buffer.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8920 (wgpu), none filed for deno

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/2944 and/or https://github.com/gfx-rs/wgpu/issues/8030

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9143

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9143

---

# Render Pipeline Output Targets (3% pass)

Selectors:
- `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets:*`
- `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets,blend:*`

Root causes:
- Color target without shader output requires writeMask=0 (66 failures).
- Blend factors reading source alpha require vec4 output (100 failures).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9147

---

# Render Pipeline Vertex State (0% pass)

Selector: `webgpu:api,validation,render_pipeline:vertex_state:*`

**Overall Status:** 0% pass (max_vertex_attribute_limit and max_vertex_buffer_limit tests)

**Root cause:** Validation bug in wgpu-core where empty vertex buffers (buffers with no attributes) are not counted toward the `maxVertexBuffers` limit. The validation at `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs:4009` checks `vertex_buffers.len()` which excludes empty buffers (skipped at lines 3968-3970), but should check `vertex.buffers.len()` instead.

**Impact:** Tests with `maxVertexBuffers + 1` where some buffers are empty show "Validation succeeded unexpectedly" because empty buffers aren't counted.

**Fix needed:** Change line 4009 to check `vertex.buffers.len()` instead of `vertex_buffers.len()`.

**Secondary issue:** Internal errors have empty messages in deno\_webgpu (`error.rs:186`), causing "undefined" error messages in tests.

See: `docs/cts-triage/render_pipeline_vertex_state.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/7912 (That issue is for nullable vertex buffers, so may not inherently fix this issue, but is probably a good time to address it, and it doesn't seem worth a dedicated bug when that one already exists.)

---

# Shader Validation Variable Declaration

Selector: `webgpu:shader,validation,decl,var:*`

**Overall Status:** 783P/10F/0S (98.74% pass rate)

**Root causes:**

1. **Trailing comma in address space template (1 failure)** - Parser rejects valid WGSL syntax `var<function,>` with trailing comma. Affects `address_space_access_mode:address_space="function";trailing_comma=true`.

2. **Shader stage restrictions (5 failures)** - wgpu accepts variables in shader stages where they're prohibited:
   - Write-only storage textures in vertex shaders (`handle_wo`)
   - Read-write storage textures in vertex shaders (`handle_rw`)
   - Read-write storage buffers in vertex shaders (`storage_rw`)
   - Workgroup variables in vertex shaders
   - Workgroup variables in fragment shaders

See: `docs/cts-triage/shader_validation_decl_var.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8925, https://github.com/gfx-rs/wgpu/issues/9148

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

**Related PR:** https://github.com/gpuweb/cts/pull/4610

## Issue 2: Missing Validation for Override-Sized Arrays in Private Address Space

**Affected tests:** private_ptr_param with stage="override"

**What's happening:** Naga accepts override-sized arrays in private address space. WGSL spec says override-sized arrays only valid in workgroup address space.

**Fix needed:** Add Naga validation to reject override-sized arrays in non-workgroup address spaces.

See: `docs/cts-triage/shader_decl_override.md` for details.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9148

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/4390

---

# Shader Expression Access Vector

Selector: `webgpu:shader,validation,expression,access,vector:*`

**Overall Status:** 44P/40F/0S (52.38% pass)

## 1. Overly Strict Runtime Indexing Validation

**Problem:** Naga performs compile-time bounds checking on runtime indices. Doesn't distinguish const-expressions (reject OOB) from runtime (allow, clamping applies).

**Impact:** Tests with `let`/`var` and OOB indices fail when they should pass.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/4390

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9002

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9150

---

# Shader Builtin Functions

## Atomic Builtin Validation

Selector: `webgpu:shader,validation,expression,call,builtin,atomics:*`

**Overall Status:** 132P/22F/0S (85.71% pass rate)

**Root causes:** Two validation gaps in Naga:

1. **Vertex shader stage validation (11 failures)** - Atomic operations incorrectly accepted in vertex shaders. Per WGSL spec, atomic builtin functions must not be used in vertex shader stages, but wgpu accepts them.

**Note:** This requirement is not cited explicitly in #9148, but it follows from the requirements that atomics be in read-write storage and that read-write storage is not supported in the vertex stage.

2. **Address space and access mode validation (11 failures)** - Atomics incorrectly accepted in invalid configurations:
   - Storage atomics with `read` access (should require `read_write`)
   - Atomics in `private` address space (only `storage` and `workgroup` are valid)
   - Atomics in `function` address space (only `storage` and `workgroup` are valid)

**Note:** These failures are distinct from issue #5474 (direct atomic references). These are about missing validation when atomic builtin functions are used in invalid contexts.

See: `docs/cts-triage/builtin_atomics.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/9148

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

**Related issues:** https://github.com/gfx-rs/wgpu/issues/7700, https://github.com/gfx-rs/wgpu/issues/8896

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8900

---

## `clamp()`: Constraint Validation

Selector: `webgpu:shader,validation,expression,call,builtin,clamp:*`

**Overall Status:** 161P/65F/0S (71.24% pass rate)

**Root cause:** Naga only validates the `low <= high` constraint when the entire clamp expression can be fully constant-evaluated (all three arguments are constants). The WebGPU spec requires validation when only `low` and `high` are const-expressions or override-expressions, even if the first argument `e` is a runtime value.

**Failing pattern:** Tests where both `lowStage` and `highStage` are `"constant"` or `"override"` (64 failures in `low_high:*` subcategory). Tests where at least one is `"runtime"` pass.

**Fix needed:** Add partial evaluation logic for builtin parameter constraints, or a separate validation pass that checks clamp calls with const/override low/high parameters independently of full constant evaluation.

See: `docs/cts-triage/builtin_clamp.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9152

---

## `insertBits()`: Override Expression Validation

Selector: `webgpu:shader,validation,expression,call,builtin,insertBits:*`

**Overall Status:** 85P/32F/0S (72.65% pass rate)

**Root causes:**

1. **Missing constant evaluation support** (~20 failures) - Already tracked in Known Issues Reference #4507. The `insertBits` builtin is not implemented in Naga's constant evaluator (`naga/src/proc/constant_evaluator.rs:1861`).

2. **Missing override expression validation** (4 failures in `count_offset:*`) - When `offset + count > 32` and either offset or count are override expressions (not runtime variables), wgpu should produce a pipeline creation error but doesn't. The WebGPU spec requires validation at pipeline creation time when override values can be evaluated. Affects cases where offset=33, count=33, or offset+count>32.

**Fix needed:** Add pipeline creation validation for `insertBits` to check that `offset + count <= 32` when using override expressions.

See: `docs/cts-triage/builtin_insertBits.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/4507, https://github.com/gfx-rs/wgpu/issues/9152

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9087

---

## textureGather()

Selector: `webgpu:shader,validation,expression,call,builtin,textureGather:*`

**Overall Status:** ~99% pass rate

**Root cause:** Small number of failures due to lack of expected validation errors. Appears to be due to missing validation of `offset` argument and some invalid usages involving depth textures.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9087

---

## textureSampleGrad()

Selector: `webgpu:shader,validation,expression,call,builtin,textureSampleGrad:*`

**Overall Status:** 839P/12F/0S (98.59% pass rate)

**Root causes:**

1. **Missing offset range validation** (6 failures) - Naga validates that offset is a const-expression with correct type but doesn't check values are in required range [-8, 7]. Tests with offset values -9 or 8 are incorrectly accepted. Location: `naga/src/valid/expression.rs:509-525`.

2. **Depth textures incorrectly accepted** (6 failures) - Naga accepts depth textures with textureSampleGrad when only sampled textures should be allowed. The spec restricts textureSampleGrad to `texture_2d<f32>`, `texture_2d_array<f32>`, `texture_3d<f32>`, `texture_cube<f32>`, and `texture_cube_array<f32>`. Location: `naga/src/valid/expression.rs:660-692`.

**Fix needed:** Add offset value range validation and image class check to reject depth textures in the Gradient sample level validation.

See: `docs/cts-triage/builtin_textureSampleGrad.md`

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9087

---

# Shader Functions Restrictions

Selector: `webgpu:shader,validation,functions,restrictions:*`

**Overall Status:** 98% pass (if external texture is enabled)

**What it tests:** Validates function restrictions including attribute placement,
return types, and parameter matching.

## Invalid function attributes accepted (2? failures)

**Root cause:** Naga accepts `@id` and `@workgroup_size` attributes on regular functions when they should only be valid on specific contexts:
- `@id` should only be valid on override declarations (not functions at all)
- `@workgroup_size` should only be valid on compute entry point functions (functions with `@compute`)

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8898

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

# Shader Parse `@group` attribute

Selectors:
- `webgpu:shader,validation,parse,attribute:expressions:value="val";attribute="group"`
- `webgpu:shader,validation,parse,attribute:expressions:value="expr";attribute="group"`
- `webgpu:shader,validation,parse,attribute:expressions:value="const";attribute="group"`

**What it tests:** Validates that WGSL attributes can accept various expression types in their parameters.

**Root cause:** wgpu validates bind group indices against device limits (`max_bind_groups = 8`) during shader module creation at wgpu-core/src/device/resource.rs:2265-2276. The test generates shaders with `@group(32)` or `@group(30 + 2)` or `@group(8)` which are syntactically valid WGSL.

**Why this is incorrect:** Per WebGPU spec, `createShaderModule` should only perform WGSL language validation (syntax and semantic analysis), NOT validate against device limits. Device limit validation should occur during pipeline/bind group creation when the shader is actually used.

**Fix needed:** Remove the group index validation loop (lines 2265-2276) from `create_shader_module`. Move validation to pipeline creation functions where shader requirements are checked against available resources.

See: `docs/cts-triage/shader_parse_group.md` for detailed analysis.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9156

---

# Shader Parse Shadow Builtins

Selector: `webgpu:shader,validation,parse,shadow_builtins:*`

**Overall Status:** 25P/5F/0S (83.33% pass rate)

**What it tests:** Validates that WGSL allows user-defined identifiers to shadow built-in functions, types, and keywords in appropriate scoping contexts.

**Note:** Some tests in this suite require external texture support and must be run with `--enable-external-texture` on a platform that supports external textures.

**Selector:** `shadow_hides_builtin:inject="none"` and `inject="sibling"` (determinant subcase)

**Root cause:** Naga fails to materialize abstract numeric literals to concrete `f32` types when calling `determinant()`. The literals `1, 2, 3, 4` create abstract integers/floats that should be materialized to `f32` when constructing a `mat2x2`, but the evaluation pipeline fails.

**Error:**
```
failed to convert expression to a concrete type: Subexpression(s) are not constant
```

**Fix needed:** Investigate Naga's abstract numeric type materialization for the `determinant` builtin. This is a broader const evaluation issue, not specific to shadowing.

**Related issues:** https://github.com/gfx-rs/wgpu/issues/4507, https://github.com/gfx-rs/wgpu/issues/7405, possibly more

---

# Phony Assignment Statement Validation

Selector: `webgpu:shader,validation,statement,phony:*`

**Overall Status:** 56P/6F/62S (90.3%/9.7%/100%)

## 1. Phony assignment in for-loops with semicolons

**What it tests:** WGSL phony assignments (`_ = expr`) are special statements that discard values. The spec defines specific syntax rules for where they can appear, including within for-loops.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9150

---

# Shader IO Align

Selector: `webgpu:shader,validation,shader_io,align:*`

**Overall Status:** 292P/1F/4S (98.32% pass rate)

**Root cause:** `@align(2147483648)` (2^31) is accepted by naga when the WGSL
spec requires rejection. The alignment value is parsed as `u32` via
`const_u32()`, and 2^31 fits in `u32` and is a power of 2, so it passes. The
WGSL spec requires the `@align` argument to be expressible as `i32`, and 2^31
exceeds `i32::MAX` (2147483647).

**Fix needed:** Add a range check in `naga/src/front/wgsl/lower/mod.rs` (around
line 4616) to reject alignment values greater than `i32::MAX`.

---

# Shader IO ID

Selector: `webgpu:shader,validation,shader_io,id:*`

**Overall Status:** 30P/2F/0S (93.75% pass rate)

**@id accepted on const declarations (1 failure)** - Naga incorrectly accepts `@id(1) const a = 4;` when @id should only be valid on `override` declarations. Parser collects @id but doesn't validate it's only used with override.

See: `docs/cts-triage/shader_io_id.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8898

---

# Shader IO Interpolate

Selector: `webgpu:shader,validation,shader_io,interpolate:*`

**Overall Status:** 640P/13F/49S (91.2% pass rate)

## 1. Integer IO accepted without explicit `@interpolate(flat)` (12 failures)

The WGSL spec requires integer-typed user IO to have an explicit
`@interpolate(flat)` attribute. Naga silently defaults integer types to `Flat`
interpolation in `naga/src/front/interpolator.rs` (`apply_default_interpolation`,
line 41), which runs before validation. The validator has an
`InvalidIntegerInterpolation` error variant in `naga/src/valid/interface.rs:152`
but it is never emitted anywhere in the codebase.

**Fix needed:** The WGSL front end needs to distinguish "no `@interpolate`
specified" from "defaulted" for integer types and emit an error in the former
case.

## 2. Trailing comma with one argument not accepted (1 failure)

`@interpolate(flat,)` is valid WGSL but Naga rejects it. In
`naga/src/front/wgsl/parse/mod.rs` at line 204, after consuming the comma, the
parser unconditionally tries to parse a second identifier, failing when it finds
`)`.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/6394

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

1. **Large size value rejected (1 failure)** - `@size(2147483647)` (2GB) rejected because it exceeds wgpu's MAX_TYPE_SIZE limit of 1GB. Requires investigation if spec allows 2GB sizes.

2. **Missing validation for runtime-sized arrays (1 failure)** - `@size` incorrectly accepted on runtime-sized arrays (`array<f32>` without size). Should be rejected since runtime-sized arrays don't have creation-fixed footprint.

See: `docs/cts-triage/shader_io_size.md`

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8898

---

# Shader Validation Types

Selector: `webgpu:shader,validation,types,*`

**Overall Status:** 95% pass rate (1450P/47F/29S out of 1526 tests covering 10 type areas: alias, array, atomics, enumerant, matrix, pointer, ref, struct, textures, vector)

**Note:** Some tests in this suite require external texture support and must be run with `--enable-external-texture` on a platform that supports external textures.

## Failure Patterns (47 failures)

### 1. Atomic Validation Gaps (7 failures)

**Root cause:** Multiple validation issues with atomic types:
- Atomics accepted in read-only storage (should require read_write) - 1 failure
- Atomics accepted in pointer types with invalid address spaces (private, function, uniform) - 3 failures

### 2. Pointer with write-only access mode to storage should be invalid (2 failures)

### 3. 16-bit Normalized Storage Texture Formats (36 failures)

**Root cause:** Naga rejects storage textures with 16-bit normalized formats (r16unorm, r16snorm, rg16unorm, rg16snorm, rgba16unorm, rgba16snorm) even though these are valid with the `texture-formats-tier1` feature. This is an architectural issue where shader validation happens before device feature checking.

**Impact:** Highest number of failures in this category.

See: `docs/cts-triage/shader_validation_types.md` for detailed analysis of all 6 failure patterns.

**Related issues:** https://github.com/gfx-rs/wgpu/issues/8122, https://github.com/gfx-rs/wgpu/issues/9148

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

**Related issue:** https://github.com/gfx-rs/wgpu/issues/9001

---
