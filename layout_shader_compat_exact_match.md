I see! At line 446, it calls `t.expectValidationError(..., !_success)`. So:
- If `success = true`, it expects NO validation error
- If `success = false`, it expects a validation error

So for `readwriteStorageTex` layout with `readonlyStorageTex` shader:
- `success = false` (line 248, they don't match, and not covered by lines 251 or 254-256)
- Test expects a validation error
- But wgpu is NOT producing an error (test passes means wgpu's behavior matched expectations)

Wait, that doesn't make sense. If the test "passes", it means wgpu DID produce an error when expected. Let me trace through this more carefully by running it with more verbose output.

Actually, wait. Let me reconsider. When I ran the test earlier:
```
[pass] webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:bindingInPipelineLayout="readwriteStorageTex";bindingInShader="readonlyStorageTex"
```

The test PASSED. This means wgpu's behavior matched what the CTS expected. Since `success = false` for this combination (not explicitly allowed in lines 254-256), the CTS expects wgpu to REJECT it, and wgpu does indeed reject it (produces a validation error). So that's correct behavior.

Now for the FAILING test:
```
bindingInPipelineLayout="readwriteStorageTex";bindingInShader="writeonlyStorageTex"
```

According to lines 254-256, `success = true`, so the CTS expects wgpu to ACCEPT this (no validation error). But wgpu is producing a validation error, which is why the test fails with "Unexpected validation error occurred".

So the bug is clear:
- wgpu should ACCEPT `read-write` layout with `write` shader → currently REJECTS (BUG)
- wgpu should REJECT `read-write` layout with `read` shader → currently REJECTS (correct)

Now I have a complete understanding. Let me compile the final triage report.

## CTS Triage Summary

**Test Selector:** `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*`

**Overall Status:** 109P/1F/0S (99.09%/0.91%/0%)

### Root Cause

There is 1 failing test out of 110 total tests:

**Failing Test:**
```
webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:bindingInPipelineLayout="readwriteStorageTex";bindingInShader="writeonlyStorageTex"
```

**Error Message:**
```
Unexpected validation error occurred: Error matching ShaderStages(FRAGMENT) shader requirements against the pipeline: Shader global ResourceBinding { group: 0, binding: 0 } is not available in the pipeline layout: Texture class Storage { format: R32Float, access: StorageAccess(LOAD | STORE) } doesn't match the shader Storage { format: R32Float, access: StorageAccess(STORE) }
```

**Root Cause:**
The validation code in `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs:703-708` performs an exact match comparison between the pipeline layout's storage texture access mode and the shader's access mode:

```rust
if class != expected_class {
    return Err(BindingError::WrongTextureClass {
        binding: expected_class,
        shader: class,
    });
}
```

According to the WebGPU spec (section 8163-8171 of `webgpu-spec.bs`), when a pipeline layout binding has `read-write` access mode, the shader can use EITHER `read_write` OR `write` access mode. The spec states:

```
: {{GPUStorageTextureAccess/"read-write"}}
:: The access mode `A` is `read_write` or `write`.
```

This allows the pipeline layout to provide more capabilities than the shader actually uses - a form of "promoting" the binding. However, wgpu currently requires an exact match, rejecting the valid case where the layout has `read-write` (LOAD | STORE) and the shader has `write` (STORE only).

### Suggested Bug Reference for fail.lst

```
Storage texture read-write bindings not compatible with write-only shaders
```

or

```
Storage texture access mode validation too strict (read-write should match write)
```

### Summary for triage.md

```markdown
## Layout Shader Compatibility - Pipeline Layout Shader Exact Match (99% pass)

**Test selector:** `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*`

**Model:** Claude Sonnet 4.5

**Overall Status:** 109P/1F/0S (99.09%/0.91%/0%)

### Issue: Storage Texture Access Mode Validation Too Strict

**Failing test:**
```
webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:bindingInPipelineLayout="readwriteStorageTex";bindingInShader="writeonlyStorageTex"
```

**Error:**
```
Unexpected validation error occurred: Texture class Storage { format: R32Float, access: StorageAccess(LOAD | STORE) } doesn't match the shader Storage { format: R32Float, access: StorageAccess(STORE) }
```

**Root cause:**
The validation code at `/Users/Andy/Development/wgpu2/wgpu-core/src/validation.rs:703-708` requires an exact match between pipeline layout and shader storage texture access modes. According to WebGPU spec section 8163-8171, when a pipeline layout has `read-write` access, the shader is allowed to use either `read_write` OR `write` access mode. This is a form of capability "promotion" where the layout provides more access than the shader needs.

**Fix needed:**
Modify the `check_binding_use` function in `wgpu-core/src/validation.rs` to allow:
- Pipeline layout `read-write` (LOAD | STORE) to match shader `write` (STORE)
- Pipeline layout `read-write` (LOAD | STORE) to match shader `read_write` (LOAD | STORE) [already works]

The validation should check if the shader's requested access is a subset of the layout's provided access, rather than requiring an exact match.

**Related failures:**
This is the same root cause as failures in:
- `webgpu:api,validation,compute_pipeline:resource_compatibility:*` (12 failures mentioned in triage.md line 236)
- `webgpu:api,validation,render_pipeline,resource_compatibility:*` (mentioned in fail.lst line 73)
```
