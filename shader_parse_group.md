Good! Now I have all the information I need to create the triage report. Let me create it now.

## Summary

I've completed the triage for the selector `webgpu:shader,validation,parse,attribute:*` (93% pass rate). Here are my findings:

### Triage Report

**CTS selector:** `webgpu:shader,validation,parse,attribute:*`

**Model:** Claude Sonnet 4.5

**Overall Status:** 39P/3F (92.86%)

---

### Passing Sub-suites ✅

The following test combinations pass successfully:
- All combinations of `attribute` types (align, binding, group, id, location, size, workgroup_size) with value types: `user_func`, `const_func`, and `override`
- Most combinations with `val`, `expr`, and `const` value types

**Total passing:** 39/42 tests

---

### Remaining Issues ⚠️

**3 failing tests** - All related to `@group` attribute with large group indices:
- `expressions:value="val";attribute="group"` 
- `expressions:value="expr";attribute="group"`
- `expressions:value="const";attribute="group"`

---

### Issue Detail

#### 1. Group Index Validation During Shader Module Creation

**Test selector:** `webgpu:shader,validation,parse,attribute:expressions:value="val";attribute="group"`

**What it tests:** Validates that WGSL attributes can accept various expression types (literals, expressions, constants, const functions, overrides, user functions) in their parameters.

**Example failure:**
```
webgpu:shader,validation,parse,attribute:expressions:value="val";attribute="group"
```

**Error:**
```
Unexpected validation error occurred: Shader global ResourceBinding { group: 32, binding: 1 } uses a group index 32 that exceeds the max_bind_groups limit of 8.
```

**Root cause:**

The test generates a shader with `@group(32)` (for `value="val"`) or `@group(30 + 2)` (for `value="expr"`) or `@group(8)` (for `value="const"`, since `a_const = -2 + 10 = 8`). These are syntactically valid WGSL and should be accepted by `createShaderModule`.

However, wgpu is validating bind group indices against device limits (`max_bind_groups = 8`) **during shader module creation** at `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs:2265-2276`:

```rust
for (_, var) in module.global_variables.iter() {
    match var.binding {
        Some(br) if br.group >= self.limits.max_bind_groups => {
            return Err(pipeline::CreateShaderModuleError::InvalidGroupIndex {
                bind: br,
                group: br.group,
                limit: self.limits.max_bind_groups,
            });
        }
        _ => continue,
    };
}
```

**Why this is incorrect:**

According to the WebGPU specification, `createShaderModule` should only perform WGSL language validation (syntax and semantic analysis). It should NOT validate against device limits. Device limit validation should occur later during:
1. Pipeline creation (when the shader is actually used)
2. Bind group creation (when bindings are allocated)

The shader itself may declare bindings at any group index - the validation should only fail if those bindings are actually used in a way that exceeds device limits.

**Fix needed:**

Remove the group index validation loop (lines 2265-2276) from `create_shader_module` in `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs`. This validation should instead be performed during pipeline creation where it's checked that the shader's requirements match the available resources.

The validation logic should be moved to pipeline creation functions where the shader module is actually bound and used, ensuring that only pipelines that would exceed device limits are rejected, not the shader modules themselves.

---

### Files Involved

- **Test failures:** `/Users/Andy/Development/wgpu2/cts_runner/fail.lst` line 178
- **Incorrect validation:** `/Users/Andy/Development/wgpu2/wgpu-core/src/device/resource.rs` lines 2265-2276
- **Error definition:** `/Users/Andy/Development/wgpu2/wgpu-core/src/pipeline.rs` lines 125-132 (`InvalidGroupIndex` error variant)
- **CTS test source:** `/Users/Andy/Development/cts/src/webgpu/shader/validation/parse/attribute.spec.ts`
