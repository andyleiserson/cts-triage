AHA! I found the bug! Look at lines 3968-3970:

```rust
if vb_state.attributes.is_empty() {
    continue;
}
vertex_buffers.push(hal::VertexBufferLayout { ... });
```

When a vertex buffer has no attributes, it's **skipped** (not added to `vertex_buffers`)! But it's **not** skipped from the loop count. So when the CTS test creates multiple empty vertex buffers and then adds all attributes to the last buffer, the empty buffers are not counted in `vertex_buffers.len()`.

This means:
1. If the test creates `maxVertexBuffers + 1` buffers, but some of them are empty, then `vertex_buffers.len()` might be less than `maxVertexBuffers + 1`
2. The validation on line 4009 checks `vertex_buffers.len()` not `desc.vertex.buffers.len()`

So the fix should be to check `desc.vertex.buffers.len()` or `vertex.buffers.len()` instead of `vertex_buffers.len()`. Let me verify this theory. Looking at line 3907:

```rust
for (i, vb_state) in vertex.buffers.iter().enumerate() {
```

So `vertex.buffers` is the input from the descriptor. We should be checking `vertex.buffers.len()` not `vertex_buffers.len()`.

Now let me write up my findings in a triage report. Based on my investigation, I've identified:

1. **Root cause for max_vertex_buffer_limit**: The validation checks `vertex_buffers.len()` but this excludes empty vertex buffers (line 3968-3970). It should check `vertex.buffers.len()` instead.

2. **Root cause for max_vertex_attribute_limit**: Same validation issue, plus there's an additional issue where `GPUInternalError` has no message (line 186 in error.rs returns Ok() without writing anything).

Let me document these findings:

## Summary

I've completed the triage of the selector `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_attribute_limit:*`. Here are my findings:

### Test Overview

**Selector:** `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_attribute_limit:*`

**What it tests:** Validates that render pipelines correctly enforce the `maxVertexAttributes` limit, testing with 0, 1, exactly the limit, and limit+1 vertex attributes across different buffer configurations (0, 1, or 4 attributes per buffer).

**Test Status:** 0% pass rate (1 test with 12 subcases, all failing)

### Issues Found

#### Issue 1: Validation Bug - Empty Vertex Buffers Not Counted

**Root Cause:**

In `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` at lines 3968-3970 and 4009:

```rust
for (i, vb_state) in vertex.buffers.iter().enumerate() {
    // ... validation code ...
    if vb_state.attributes.is_empty() {
        continue;  // Skip adding empty buffers to vertex_buffers
    }
    vertex_buffers.push(hal::VertexBufferLayout { ... });
    // ...
}

if vertex_buffers.len() > self.limits.max_vertex_buffers as usize {
    return Err(pipeline::CreateRenderPipelineError::TooManyVertexBuffers { ... });
}
```

The code checks `vertex_buffers.len()` which excludes empty vertex buffers (those with no attributes). According to the WebGPU spec, empty vertex buffers should still count toward the `maxVertexBuffers` limit. The validation should check `vertex.buffers.len()` instead.

**Impact:** The `max_vertex_buffer_limit` test shows "Validation succeeded unexpectedly" when trying to create pipelines with `maxVertexBuffers + 1` buffers where some are empty.

**Fix needed:** Change line 4009 from:
```rust
if vertex_buffers.len() > self.limits.max_vertex_buffers as usize {
```
to:
```rust
if vertex.buffers.len() > self.limits.max_vertex_buffers as usize {
```

#### Issue 2: Internal Error with Empty Message

**Root Cause:**

In `/Users/Andy/Development/wgpu2/deno_webgpu/error.rs` at lines 176-177 and 186:

```rust
#[class("GPUInternalError")]
Internal,

// In Display implementation:
GPUError::Internal => Ok(()),  // Returns Ok() without writing any message!
```

When a `GPUInternalError` is generated, the `Display` implementation writes nothing to the formatter, resulting in an empty/undefined message. This causes the CTS test to report "Unexpected internal error occurred: undefined".

**Impact:** The `max_vertex_attribute_limit` test fails during cleanup with an internal error that has no message, making it difficult to diagnose the underlying issue.

**Fix needed:** Update the Display implementation to provide a meaningful message for internal errors, for example:
```rust
GPUError::Internal => f.write_str("internal error"),
```

### Related Test

The selector `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_buffer_limit:*` (also at 0%) has the same validation bug. It explicitly shows "Validation succeeded unexpectedly" for subcases with `countVariant={"mult":1,"add":1}` (limit+1 buffers).

### Files Requiring Changes

1. `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` - Fix vertex buffer count validation (line 4009)
2. `/Users/Andy/Development/wgpu2/deno_webgpu/error.rs` - Add message for internal errors (line 186)
