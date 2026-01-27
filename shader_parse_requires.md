# Shader Parse Requires CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,parse,requires:*`

Model: Claude Sonnet 4.5

**Overall Status:** 5P/3F/15S (21.74%/13.04%/65.22%)

## Passing Sub-suites ✅

- `wgsl_matches_api:feature="unrestricted_pointer_parameters"` - Passes because this feature is unimplemented in Naga, so both the API reports it as unsupported AND Naga rejects it
- `wgsl_matches_api:feature="uniform_buffer_standard_layout"` - Passes (feature not in Naga's known features list)
- `wgsl_matches_api:feature="texture_and_sampler_let"` - Passes (feature not in Naga's known features list)
- `wgsl_matches_api:feature="subgroup_id"` - Passes (feature not in Naga's known features list)
- `wgsl_matches_api:feature="subgroup_uniformity"` - Passes (feature not in Naga's known features list)

## Remaining Issues ⚠️

- `requires:requires:*` (15 tests) - All **skipped** because `wgslLanguageFeatures` is not implemented in deno_webgpu
- `wgsl_matches_api` - 3 tests fail for features that ARE implemented in Naga but not reported by the API

## Issue Detail

### 1. Missing wgslLanguageFeatures implementation in deno_webgpu

**Test selector:** `webgpu:shader,validation,parse,requires:*`

**What it tests:** The `requires` directive in WGSL allows shaders to declare that they depend on specific language features. The CTS has two test suites:

1. `requires:requires:*` - Tests that the `requires` directive syntax is parsed correctly (placement, comments, duplicates, etc.)
2. `wgsl_matches_api:*` - Tests that shader compilation succeeds if and only if `gpu.wgslLanguageFeatures` reports the feature as supported

**Example failures:**

```
webgpu:shader,validation,parse,requires:wgsl_matches_api:feature="readonly_and_readwrite_storage_textures"
webgpu:shader,validation,parse,requires:wgsl_matches_api:feature="packed_4x8_integer_dot_product"
webgpu:shader,validation,parse,requires:wgsl_matches_api:feature="pointer_composite_access"
```

**Error from wgsl_matches_api failures:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
requires readonly_and_readwrite_storage_textures;
```

**Root cause:**

The `deno_webgpu` crate does not implement the `wgslLanguageFeatures` property on its `GPU` class (`/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs`).

When the CTS JavaScript test calls `gpu.wgslLanguageFeatures`, it returns `undefined`. This causes:

1. **Failures in `wgsl_matches_api` for implemented features:**
   - Test checks `gpu.wgslLanguageFeatures.has('readonly_and_readwrite_storage_textures')` → returns `false` (because property is undefined)
   - Test expects shader compilation to **fail** (since API reports feature as "not supported")
   - But Naga has this feature implemented (see `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/directive/language_extension.rs:57-61`)
   - Naga accepts the `requires` directive without error
   - Result: Test expects error, compilation succeeds → **FAIL**

2. **Skips in `requires:requires:*`:**
   - These 15 tests all call `skipIfLanguageFeatureNotSupported('readonly_and_readwrite_storage_textures')` at the start
   - Since `wgslLanguageFeatures` is undefined, the check returns `false`
   - All 15 tests skip before they can test the actual `requires` syntax parsing

**Features that should be reported:**

Naga currently implements these three language features (as `ImplementedLanguageExtension` in Naga):
- `readonly_and_readwrite_storage_textures`
- `packed_4x8_integer_dot_product`
- `pointer_composite_access`

Naga recognizes but does NOT implement:
- `unrestricted_pointer_parameters` (marked as `UnimplementedLanguageExtension`, tracking issue #5158)

**Fix needed:**

Add the `wgslLanguageFeatures` property to the `GPU` class in deno_webgpu. The implementation should:

1. Add a getter method for `wgslLanguageFeatures` on the `GPU` struct in `/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs`
2. Return a JavaScript Set-like object containing the implemented features
3. Use the same logic as the wgpu_core backend in `/Users/Andy/Development/wgpu2/wgpu/src/backend/wgpu_core.rs:880-898` which queries `ImplementedLanguageExtension::all()`
4. The returned Set should contain the string names of the three implemented features

Reference implementations:
- `/Users/Andy/Development/wgpu2/wgpu/src/backend/wgpu_core.rs:880-898` - Shows how to query Naga's implemented features
- `/Users/Andy/Development/wgpu2/wgpu/src/backend/webgpu.rs:1630-1655` - Shows how the browser WebGPU backend reads from `gpu.wgslLanguageFeatures`
- `/Users/Andy/Development/wgpu2/deno_webgpu/adapter.rs:388-427` - Shows how `GPUSupportedFeatures` is implemented as a similar Set-like object

**Expected outcome after fix:**
- 15 skipped `requires:requires:*` tests should run and pass
- 3 failing `wgsl_matches_api` tests should pass
- Overall: 23P/0F/0S (100% pass rate)
