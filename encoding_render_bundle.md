# Render Bundle Encoding CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,encoding,render_bundle:*`

**Overall Status:** 17P/4F/0S (80.95%/19.05%/0%)

## Passing Sub-suites

- `empty_bundle_list` - Tests that executing an empty bundle list is valid
- `device_mismatch` - Tests that bundles from different devices are rejected
- `color_formats_mismatch` - Tests that bundle color formats must match pass formats
- `depth_stencil_formats_mismatch` - Tests that bundle depth/stencil formats must match pass formats
- `sample_count_mismatch` - Tests that bundle sample counts must match pass sample counts

## Remaining Issues

- `depth_stencil_readonly_mismatch` - 4 test cases fail due to incorrect normalization of readonly flags for formats missing depth or stencil aspects

## Issue Detail

### 1. depth_stencil_readonly_mismatch - Readonly flag normalization mismatch

**Test selector:** `webgpu:api,validation,encoding,render_bundle:depth_stencil_readonly_mismatch:*`

**What it tests:** Validates that render bundles cannot be executed in a render pass if the readonly state of the depth/stencil attachment doesn't match. The spec requires that if the render pass has `depthReadOnly=true`, the bundle must also have `depthReadOnly=true` (and similarly for stencil).

**Example failing tests:**
```
webgpu:api,validation,encoding,render_bundle:depth_stencil_readonly_mismatch:depthStencilFormat="stencil8"
webgpu:api,validation,encoding,render_bundle:depth_stencil_readonly_mismatch:depthStencilFormat="depth24plus"
webgpu:api,validation,encoding,render_bundle:depth_stencil_readonly_mismatch:depthStencilFormat="depth16unorm"
webgpu:api,validation,encoding,render_bundle:depth_stencil_readonly_mismatch:depthStencilFormat="depth32float"
```

**Specific failing subcases (stencil8):**
- `bundleDepthReadOnly=false;bundleStencilReadOnly=false;passDepthReadOnly=true;passStencilReadOnly=false`
- `bundleDepthReadOnly=false;bundleStencilReadOnly=true;passDepthReadOnly=true;passStencilReadOnly=false`

**Specific failing subcases (depth24plus):**
- `bundleDepthReadOnly=false;bundleStencilReadOnly=false;passDepthReadOnly=false;passStencilReadOnly=true`
- `bundleDepthReadOnly=true;bundleStencilReadOnly=false;passDepthReadOnly=false;passStencilReadOnly=true`
- `bundleDepthReadOnly=true;bundleStencilReadOnly=false;passDepthReadOnly=true;passStencilReadOnly=true`

**Error:**
```
EXPECTATION FAILED: Expected validation error
```

The tests expect a validation error to be generated when the bundle's readonly flags don't match the pass's readonly flags, but wgpu is not producing the error.

**Root cause:**

The render bundle encoder normalizes `is_depth_read_only` and `is_stencil_read_only` flags based on format aspects in `/Users/Andy/Development/wgpu2/wgpu-core/src/command/bundle.rs` (lines 174-180):

```rust
let (is_depth_read_only, is_stencil_read_only) = match desc.depth_stencil {
    Some(ds) => {
        let aspects = hal::FormatAspects::from(ds.format);
        (
            !aspects.contains(hal::FormatAspects::DEPTH) || ds.depth_read_only,
            !aspects.contains(hal::FormatAspects::STENCIL) || ds.stencil_read_only,
        )
    }
    None => (true, true),
};
```

For formats without a depth aspect (like `stencil8`), `is_depth_read_only` is always set to `true` regardless of user input. Similarly, for formats without a stencil aspect (like `depth24plus`), `is_stencil_read_only` is always set to `true`.

However, the render pass does not perform the same normalization when setting its `is_depth_read_only` and `is_stencil_read_only` flags in `/Users/Andy/Development/wgpu2/wgpu-core/src/command/render.rs` (lines 1184-1185):

```rust
is_depth_read_only = at.depth.is_readonly();
is_stencil_read_only = at.stencil.is_readonly();
```

The validation check in `execute_bundles` (lines 3082-3083) then compares these values:

```rust
if (state.info.is_depth_read_only && !bundle.is_depth_read_only)
    || (state.info.is_stencil_read_only && !bundle.is_stencil_read_only)
```

**Example trace (stencil8 format with bundleDepthReadOnly=false, passDepthReadOnly=true):**
1. Bundle: `is_depth_read_only = !false || false = true` (no depth aspect, so always true)
2. Pass: `is_depth_read_only = true` (from user input)
3. Check: `true && !true = false` - no error generated
4. Expected: Error because user specified `bundleDepthReadOnly=false` but `passDepthReadOnly=true`

**Fix needed:**

The WebGPU spec requires that the user-specified readonly flags be compared, not the internally normalized values. Either:

1. The render pass should apply the same normalization based on format aspects when setting `is_depth_read_only`/`is_stencil_read_only`, OR
2. The bundle should store both the user-specified values and the normalized values, using the user-specified values for compatibility checks


# Follow-up prompt


Read `docs/cts-triage/encoding_render_bundle_triage.md`.

Study the validation implementation for render bundles and render passes. Does the missing validation that is causing this test to
fail apply to both? If so, is it correct for render passes?

Remember that you can consult /webgpu-specs to understand what validation is needed.


# Follow-up response

Yes, the normalization behavior applies to both render bundles and render passes, but wgpu's implementation is NOT spec-compliant.

What the WebGPU Spec Says

According to the WebGPU specification:

1. Render Pass (https://gpuweb.github.io/gpuweb/#dom-gpucommandencoder-beginrenderpass):
  - Line 10584-10585: Directly copies depthReadOnly and stencilReadOnly from the descriptor without normalization
2. Render Bundle (https://gpuweb.github.io/gpuweb/#render-bundle-creation):
  - Line 13905-13906: Directly copies depthReadOnly and stencilReadOnly from the descriptor without normalization
3. Compatibility Check (https://gpuweb.github.io/gpuweb/#render-bundle-encoding):
  - Lines 13746-13747: Compares the readonly flags directly:
      - "If pass.[[depthReadOnly]] is true, bundle.[[depthReadOnly]] must be true"
    - "If pass.[[stencilReadOnly]] is true, bundle.[[stencilReadOnly]] must be true"
4. Validation (https://gpuweb.github.io/gpuweb/#dom-gpurenderpassdepthstencilattachment-depthreadonly):
  - Lines 12539-12546: If format has depth aspect AND depthReadOnly is false, then load/store ops must be provided
  - Otherwise (no depth aspect OR depthReadOnly is true), load/store ops must NOT be provided

The spec allows setting depthReadOnly=false even when the format lacks a depth aspect - it just means you can't provide load/store ops.
There's no normalization specified.

What wgpu Does

Render Bundle (wgpu-core/src/command/bundle.rs:174-180):
let (is_depth_read_only, is_stencil_read_only) = match desc.depth_stencil {
    Some(ds) => {
        let aspects = hal::FormatAspects::from(ds.format);
        (
            !aspects.contains(hal::FormatAspects::DEPTH) || ds.depth_read_only,
            !aspects.contains(hal::FormatAspects::STENCIL) || ds.stencil_read_only,
        )
    }
    None => (true, true),
};
Normalizes at creation time: if format lacks depth aspect, forces is_depth_read_only = true.

Render Pass (wgpu-core/src/command/render.rs:1700-1718):
depth: if format.has_depth_aspect() {
    depth_stencil_attachment.depth.resolve(...)?
} else {
    ResolvedPassChannel::ReadOnly  // Forced normalization
},
Also normalizes: if format lacks depth aspect, forces depth to ReadOnly.

The Problem

Both sides normalize, which seems consistent BUT it's not spec-compliant. The spec says to use the raw user-provided values for the
compatibility check. wgpu's normalization makes it impossible to detect when a user provides incompatible readonly flags for aspects that
 don't exist in the format.

Is the Normalization Correct?

No, it's not spec-compliant. However, the normalization has some practical benefits:
- It prevents meaningless readonly flags from affecting behavior
- It simplifies internal logic

But it violates the spec's requirement to compare the actual user-provided values. The failing CTS tests expect validation errors when
bundle and pass have different readonly flags, even for aspects that don't exist in the format.

Should This Validation Apply to Render Passes?

The validation for render passes (render.rs:1700-1718) is similarly non-spec-compliant. It should allow depthReadOnly=false when the
format lacks a depth aspect (just rejecting load/store ops), rather than silently forcing it to readonly.
