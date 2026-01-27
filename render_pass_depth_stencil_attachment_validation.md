Let me document my findings in a comprehensive report:

## Investigation Summary

I've successfully triaged the CTS test `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,loadOp_storeOp_match_depthReadOnly_stencilReadOnly:*` which has a 33% pass rate (2/6 tests passing).

### Test Overview

This test validates the WebGPU specification requirements for `GPURenderPassDepthStencilAttachment`:

**Spec Requirements:**
1. If the format has a depth aspect AND `depthReadOnly` is `false` (or `undefined`):
   - Both `depthLoadOp` AND `depthStoreOp` must be provided
2. Otherwise (if `depthReadOnly` is `true` OR the format has no depth aspect):
   - Neither `depthLoadOp` nor `depthStoreOp` should be provided
3. Same logic applies for stencil (with `stencilReadOnly`, `stencilLoadOp`, `stencilStoreOp`)

**Additional requirement for `TRANSIENT_ATTACHMENT` usage:**
- If the format has a depth aspect: `depthLoadOp` must be "clear" and `depthStoreOp` must be "discard"
- If the format has a stencil aspect: `stencilLoadOp` must be "clear" and `stencilStoreOp` must be "discard"

### Root Cause Analysis

I identified the validation gap in `/Users/Andy/Development/wgpu2/wgpu-core/src/command/render.rs` at lines 1700-1718.

**The Problem:**

When a format does NOT have a particular aspect (depth or stencil), wgpu automatically sets that channel to `ReadOnly`:

```rust
depth: if format.has_depth_aspect() {
    depth_stencil_attachment.depth.resolve(|clear| /* ... */)?
} else {
    ResolvedPassChannel::ReadOnly  // Line 1712 - PROBLEM!
},
stencil: if format.has_stencil_aspect() {
    depth_stencil_attachment.stencil.resolve(|clear| /* ... */)?
} else {
    ResolvedPassChannel::ReadOnly  // Line 1717 - PROBLEM!
},
```

This bypasses all validation checks. The code should instead validate that when a format lacks an aspect, the corresponding load/store operations must NOT be provided by the user.

**Why This Causes Failures:**

Looking at the CTS test logic (lines 1181-1183 in the test spec):

```javascript
const goodDepthCombo = hasDepth && !depthReadOnly ? hasBothDepthOps : hasNeitherDepthOps;
const goodStencilCombo = hasStencil && !stencilReadOnly ? hasBothStencilOps : hasNeitherStencilOps;
```

The validation should fail when:
- Format lacks depth aspect BUT user provided `depthLoadOp` or `depthStoreOp`
- Format lacks stencil aspect BUT user provided `stencilLoadOp` or `stencilStoreOp`
- Format has aspect, `readOnly` is false/undefined, BUT only one op is provided (not both)

Currently, wgpu silently ignores these cases instead of rejecting them.

### Test Results

**Summary:** 2P/0W/0S/4F (33.33% pass rate)

**Passing Tests:**
- `format="depth16unorm"` (depth-only format)
- `format="depth24plus-stencil8"` (combined depth-stencil format)

**Failing Tests (4):**
- `format="depth24plus"` 
- `format="depth32float"`
- `format="depth32float-stencil8"`
- `format="stencil8"` (stencil-only format)

The failures occur because wgpu accepts invalid configurations where:
1. Only one of `{load,store}Op` is provided when both are required
2. Load/store ops are provided for aspects that don't exist in the format

### Fix Needed

The validation logic in `/Users/Andy/Development/wgpu2/wgpu-core/src/command/render.rs` (around lines 1700-1718) needs to be modified:

**Current behavior:** When format lacks an aspect, silently set channel to `ReadOnly`

**Required behavior:** 
1. Check if user provided load/store ops for an aspect that doesn't exist → return validation error
2. Check if user provided only one op (load OR store) when both are required → return validation error (this already exists via `AttachmentError::NoLoad` and `AttachmentError::NoStore`)
3. Only set to `ReadOnly` automatically if the aspect doesn't exist AND no ops were provided

**Recommended approach:**
1. Before the `if format.has_depth_aspect()` check, validate that if `!format.has_depth_aspect()` then both `depth.load_op` and `depth.store_op` must be `None`
2. Similarly for stencil aspect
3. Add new error variants like `AttachmentError::LoadOpForMissingAspect` or similar

### Files Involved

- **Test source:** `/Users/Andy/Development/cts/src/webgpu/api/validation/render_pass/render_pass_descriptor.spec.ts` (lines 1085-1196)
- **Validation code:** `/Users/Andy/Development/wgpu2/wgpu-core/src/command/render.rs` (lines 1687-1722)
- **Error types:** `/Users/Andy/Development/wgpu2/wgpu-core/src/command/render.rs` (AttachmentError enum, likely around line 40-70)

### Next Steps

To fix this issue:
1. Add new `AttachmentError` variants for ops provided on missing aspects
2. Add validation before lines 1700 and 1714 to check if ops are provided for missing aspects
3. Only default to `ReadOnly` if the aspect is missing AND no ops were explicitly provided
4. Test with the full CTS suite to ensure all 6 tests pass
