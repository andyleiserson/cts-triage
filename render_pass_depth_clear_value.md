# Render Pass Depth Clear Value - Triage Report

**CTS selector:** `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:*`

**Overall Status:** 17P/1F/0S (94.44%/5.56%/0%)

## Passing Sub-suites

This test suite has a single test function with parameter combinations. All combinations pass except one:

- `depthLoadOp="load"` with any `depthClearValue` (6 tests) - All pass
- `depthLoadOp="clear"` with defined `depthClearValue` values (-1, 0, 0.5, 1, 1.5) - 5/6 pass
- `depthLoadOp="_undef_"` with any `depthClearValue` (6 tests) - All pass

## Remaining Issues

- `depthLoadOp="clear";depthClearValue="_undef_"`: 1 failure (TypeError exception instead of validation error)

## Issue Detail

### 1. TypeError exception instead of validation error for missing depthClearValue

**Test selector:** `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:depthLoadOp="clear";depthClearValue="_undef_"`

**What it tests:**
The test validates that when `depthLoadOp` is "clear" and `depthClearValue` is undefined/missing, a validation error should occur. The test uses `expectValidationError()` to catch the expected validation failure.

**Error:**
```
EXCEPTION: TypeError: 'depthClearValue' must be specified when 'depthLoadOp' is "clear"
TypeError: 'depthClearValue' must be specified when 'depthLoadOp' is "clear"
    at F.tryRenderPass (file:///Users/Andy/Development/wgpu2/cts/out/webgpu/api/validation/render_pass/render_pass_descriptor.spec.js:102:39)
    at RunCaseSpecific.fn (file:///Users/Andy/Development/wgpu2/cts/out/webgpu/api/validation/render_pass/render_pass_descriptor.spec.js:1233:5)
```

**Root cause:**
The issue is in `/Users/Andy/Development/wgpu2/deno_webgpu/command_encoder.rs` (lines 104-113). When creating a render pass descriptor with `depthLoadOp="clear"` and `depthClearValue` undefined:

```rust
if attachment
    .depth_load_op
    .as_ref()
    .is_some_and(|op| matches!(op, GPULoadOp::Clear))
    && attachment.depth_clear_value.is_none()
{
    return Err(JsErrorBox::type_error(
        r#"'depthClearValue' must be specified when 'depthLoadOp' is "clear""#,
    ));
}
```

This code throws a JavaScript `TypeError` exception immediately at the API binding layer. However, according to WebGPU spec semantics, this should be a **validation error** that gets queued on the device and can be caught via the validation error mechanism (checked during `commandEncoder.finish()`).

The CTS test:
1. Creates a render pass with the invalid descriptor
2. Expects the validation error to be caught via `expectValidationError()` when calling `commandEncoder.finish()`

But wgpu throws an exception during `beginRenderPass()` itself, which bypasses the validation error mechanism entirely.

**Fix needed:**
Instead of throwing a `TypeError` at the deno binding layer, the code should:
1. Allow the operation to proceed with some default/sentinel value for `depthClearValue`
2. Let wgpu-core's validation layer catch this as a proper validation error
3. Return the validation error through the standard WebGPU error reporting mechanism
