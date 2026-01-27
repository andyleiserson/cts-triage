I see - there's no TypeError option in ErrorType. That's because TypeError is a different kind of error in WebGPU - it's a JavaScript exception that should be thrown synchronously, not a GPU error that goes into an error scope.

This means the fix needs to be in deno_webgpu to detect MissingFeatures errors and throw TypeError instead of treating them as validation errors.

Now I have enough information to provide a comprehensive triage report. Let me create a summary based on my investigation.

Based on my analysis of the CTS tests and wgpu codebase, here are the findings for the `webgpu:api,validation,capability_checks,features,*` selector:

## Summary

**Root Cause**: wgpu-core returns `MissingFeatures` errors as `ErrorType::Validation`, but the WebGPU specification requires that using optional features without enabling them should throw **TypeError** exceptions synchronously, not validation errors that get pushed to error scopes.

The tests in this suite verify that WebGPU correctly throws TypeError when:
1. Creating textures with optional formats without the required feature enabled
2. Creating texture views with optional formats without the required feature enabled  
3. Using query types (timestamp) without the required feature enabled
4. Creating render pipelines with clip_distances without the required feature enabled
5. Using texture component swizzle without the required feature enabled

**Current behavior**: wgpu-core correctly detects when a feature is missing (via `Device::require_features()` and `TextureFormat::required_features()`), but the error is classified as a validation error. In deno_webgpu, these validation errors are pushed to the error handler rather than being thrown as TypeError exceptions.

**Expected behavior**: Per WebGPU spec, these should throw TypeError synchronously when the API call is made, not generate validation errors.

## Specific Test Categories

### 1. **clip_distances**
- **Tests**: Render pipeline creation with `@builtin(clip_distances)` in shaders
- **Status**: wgpu has the CLIP_DISTANCES feature implemented
- **Issue**: Tests expect TypeError when feature not enabled, but likely getting validation error instead

### 2. **query_types**  
- **Tests**: Creating timestamp query sets and using timestampWrites without the feature enabled
- **Status**: wgpu correctly checks for TIMESTAMP_QUERY feature (line 4775-4777 in device/resource.rs)
- **Issue**: MissingFeatures error returned as validation error instead of TypeError

### 3. **texture_component_swizzle**
- **Tests**: Using non-identity swizzle values in texture views
- **Status**: wgpu does NOT have TEXTURE_COMPONENT_SWIZZLE feature implemented
- **Issue**: Feature not implemented at all in wgpu-types/src/features.rs

### 4. **texture_formats** / **texture_formats_tier1** / **texture_formats_tier2**
- **Tests**: Creating textures/views with optional formats (BC, ETC2, ASTC, 16-bit norm, etc.)
- **Status**: wgpu correctly checks format requirements via `TextureFormat::required_features()` (wgpu-types/src/texture/format.rs:690)
- **Issue**: MissingFeatures error returned as validation error instead of TypeError

## Bug Reference for fail.lst

```
webgpu:api,validation,capability_checks,features,* // TypeError not thrown for missing features #<ISSUE_NUMBER>
```

## Suggested Fix

The fix should be in deno_webgpu to intercept `MissingFeatures` errors and convert them to TypeError exceptions. Specifically:

1. In `/Users/Andy/Development/wgpu2/wgpu-core/src/device/mod.rs` line 365-367, consider adding a new ErrorType::TypeError variant
2. Or in deno_webgpu layer, check if errors are `MissingFeatures` and throw TypeError instead of pushing to error handler
3. Additionally, implement TEXTURE_COMPONENT_SWIZZLE feature in wgpu-types/features.rs

## Files Examined

- `/Users/Andy/Development/cts/src/webgpu/api/validation/capability_checks/features/clip_distances.spec.ts`
- `/Users/Andy/Development/cts/src/webgpu/api/validation/capability_checks/features/query_types.spec.ts`
- `/Users/Andy/Development/cts/src/webgpu/api/validation/capability_checks/features/texture_component_swizzle.spec.ts`
- `/Users/Andy/Development/cts/src/webgpu/api/validation/capability_checks/features/texture_formats.spec.ts`
- `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` (lines 1313-1461, 4719-4740, 4765-4792)
- `/Users/Andy/Development/wgpu2/wgpu-core/src/device/mod.rs` (lines 362-368)
- `/Users/Andy/Development/wgpu2/wgpu-types/src/features.rs` (lines 1301-1560)
- `/Users/Andy/Development/wgpu2/wgpu-types/src/texture/format.rs` (lines 690-780)
- `/Users/Andy/Development/wgpu2/deno_webgpu/device.rs` (lines 214-253)
- `/Users/Andy/Development/wgpu2/deno_webgpu/texture.rs`
