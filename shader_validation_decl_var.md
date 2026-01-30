# Shader Validation Variable Declaration

Selector: `webgpu:shader,validation,decl,var:*`

**Overall Status:** 783P/10F/0S (98.74% pass rate)

## Failing Tests

### 1. Trailing comma in address space template (1 failure)

**Test:** `address_space_access_mode:address_space="function";access_mode="";trailing_comma=true`

**Error:**
```
Shader '' parsing error: expected template args end, found ","
  ┌─ wgsl:3:19
  │
3 │       var<function,> x : u32;
  │                   ^ expected template args end
```

**Root cause:** Naga's WGSL parser does not accept trailing commas in template argument lists like `var<function,>`.

**Related issue:** https://github.com/gfx-rs/wgpu/issues/8925

### 2. Atomic types in read-only storage (4 failures)

**Tests:**
- `module_scope_types:type="atomic<i32>";kind="storage_ro";via_alias=false`
- `module_scope_types:type="atomic<i32>";kind="storage_ro";via_alias=true`
- `module_scope_types:type="atomic<u32>";kind="storage_ro";via_alias=false`
- `module_scope_types:type="atomic<u32>";kind="storage_ro";via_alias=true`

**Error:**
```
EXPECTATION FAILED: Expected validation error
---- shader ----
@group(0) @binding(0) var<storage, read> foo : atomic<i32>;
```

**Root cause:** wgpu/Naga accepts atomic types in `var<storage, read>` declarations, but the WebGPU spec requires atomics only in read-write storage.

### 3. Shader stage restrictions (5 failures)

**Tests:**
- `shader_stage:stage="vertex";kind="handle_wo"` - write-only storage texture in vertex
- `shader_stage:stage="vertex";kind="handle_rw"` - read-write storage texture in vertex
- `shader_stage:stage="vertex";kind="storage_rw"` - read-write storage buffer in vertex
- `shader_stage:stage="vertex";kind="workgroup"` - workgroup variable in vertex
- `shader_stage:stage="fragment";kind="workgroup"` - workgroup variable in fragment

**Root cause:** wgpu accepts variables in shader stages where they're prohibited per WebGPU spec:
- Storage textures with write access are not allowed in vertex shaders
- Workgroup variables are only allowed in compute shaders
