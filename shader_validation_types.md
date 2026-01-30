# Shader Validation Types - CTS Triage Report

CTS selector: `webgpu:shader,validation,types,*`

Model: Claude Sonnet 4.5

**Overall Status:** 1450P/47F/29S (95.02%/3.08%/1.90%) out of 1526 tests

## Passing Sub-suites

- **array** - 58P/0F/5S (100% pass) - Tests for array type validation
- **enumerant** - 32P/0F/0S (100% pass) - Tests for enumerant type validation
- **matrix** - 99P/0F/0S (100% pass) - Tests for matrix type validation
- **ref** - 56P/0F/0S (100% pass) - Tests for ref type validation
- **struct** - 53P/0F/0S (100% pass) - Tests for struct type validation
- **vector** - 43P/0F/0S (100% pass) - Tests for vector type validation

## Remaining Issues

- **alias** - 87P/1F/0S (98.86% pass) - One failure related to texture_external capability
- **atomics** - 40P/4F/0S (90.91% pass) - Failures related to atomic operation validation
- **pointer** - 185P/5F/0S (97.37% pass) - Failures related to pointer type validation
- **textures** - 797P/37F/24S (95.55% pass) - Failures primarily related to storage texture formats

## Issue Detail

### 1. texture_external capability not supported

**Test selector:** `webgpu:shader,validation,types,alias:any_type:type="texture_external"`

**What it tests:** Tests that texture_external can be used as a type in an alias declaration.

**Example failure:**
```
webgpu:shader,validation,types,alias:any_type:type="texture_external"
```

**Error:**
```
Shader validation error: Type [12] 'myType' is invalid
 = Capability Capabilities(TEXTURE_EXTERNAL) is required
```

**Root cause:**
The shader uses `texture_external` type but Naga reports that the `TEXTURE_EXTERNAL` capability is not supported. This is because `texture_external` is not yet implemented in wgpu/Naga.

**Related failure:**
- `webgpu:shader,validation,types,textures:external_sampled_texture_types:` - Same TEXTURE_EXTERNAL capability error

**Fix needed:**
This is a known limitation. The `texture_external` type is part of WebGPU spec but not yet implemented in wgpu. This would require significant work to support external textures (typically used for video frames).

### 2. Atomics - Invalid operations not rejected

**Test selector:** `webgpu:shader,validation,types,atomics:invalid_operations:*`

**What it tests:** Tests that certain operations on atomic types should be rejected (e.g., using atomics directly in arithmetic expressions without atomic built-in functions).

**Example failures:**
```
webgpu:shader,validation,types,atomics:invalid_operations:op="add"
webgpu:shader,validation,types,atomics:invalid_operations:op="load"
webgpu:shader,validation,types,atomics:invalid_operations:op="abs"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
var<workgroup> a1 : atomic<u32>;
var<workgroup> a2 : atomic<u32>;

fn foo() {
  let x : u32 = a1 + a2;  // Should be rejected but isn't
}
```

**Root cause:**
Naga allows referencing an atomic directly in an expression (e.g., `a1 + a2`). According to the WGSL spec, atomics should only be accessed via atomic built-in functions like `atomicLoad`, `atomicStore`, `atomicAdd`, etc. Direct usage in expressions should be a validation error.

**Fix needed:**
This is a known issue tracked at https://github.com/gfx-rs/wgpu/issues/5474. Naga's validator needs to reject direct references to atomic variables in expressions.

### 3. Atomics - Read-only storage address space incorrectly accepted

**Test selector:** `webgpu:shader,validation,types,atomics:address_space:aspace="storage-ro"`

**What it tests:** Tests that atomics are not allowed in read-only storage address space (only in read_write storage and workgroup).

**Example failure:**
```
webgpu:shader,validation,types,atomics:address_space:aspace="storage-ro"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
@group(0) @binding(0) var<storage> x : atomic<u32>;  // Should fail

fn foo() {
}
```

**Root cause:**
The test declares an atomic in storage address space without explicitly specifying `read_write`. The default access mode for storage is `read`, which should make this invalid (atomics require `read_write` or `workgroup` address spaces). However, Naga is not catching this validation error.

**Fix needed:**
Naga's validator should check that atomics in storage address space have explicit `read_write` access mode, not the default `read` access mode.

### 4. Pointer - Write-only access mode incorrectly accepted for storage

**Test selector:** `webgpu:shader,validation,types,pointer:access_mode:aspace="storage";access="write";*`

**What it tests:** Tests that pointer types with `write` access mode are not allowed for storage address space. Only `read` and `read_write` are valid for storage pointers.

**Example failures:**
```
webgpu:shader,validation,types,pointer:access_mode:aspace="storage";access="write";comma=""
webgpu:shader,validation,types,pointer:access_mode:aspace="storage";access="write";comma=","
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
alias T = ptr<storage, u32, write>;  // Should be rejected
```

**Root cause:**
The WGSL spec does not allow `write` as an access mode for storage address space pointer types. Only `read` and `read_write` are valid. Naga is incorrectly accepting this.

**Fix needed:**
Naga's validator should reject pointer types with `write` access mode when the address space is `storage`.

### 5. Pointer - Atomics in non-storage/non-workgroup address spaces

**Test selector:** `webgpu:shader,validation,types,pointer:ptr_not_instantiable:case="*Atomic"`

**What it tests:** Tests that pointer types pointing to atomics are only valid for storage (read_write) and workgroup address spaces.

**Example failures:**
```
webgpu:shader,validation,types,pointer:ptr_not_instantiable:case="privateAtomic"
webgpu:shader,validation,types,pointer:ptr_not_instantiable:case="functionAtomic"
webgpu:shader,validation,types,pointer:ptr_not_instantiable:case="uniformAtomic"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
alias p = ptr<private,atomic<u32>>;  // Should be rejected
```

**Root cause:**
Atomics are only allowed in `storage` (with `read_write` access) and `workgroup` address spaces. Pointer types pointing to atomics in other address spaces (private, function, uniform) should be rejected as they reference types that cannot be instantiated in those address spaces.

**Fix needed:**
Naga's validator should check that pointer types referencing atomics only use valid address spaces (storage with read_write, or workgroup).

### 6. Textures - 16-bit normalized storage texture formats

**Test selector:** `webgpu:shader,validation,types,textures:storage_texture_types:*;format="r16*norm";*` and related

**What it tests:** Tests that storage textures with 16-bit normalized formats (r16unorm, r16snorm, rg16unorm, rg16snorm, rgba16unorm, rgba16snorm) are accepted by the shader compiler when the device has the `texture-formats-tier1` feature.

**Example failures:** (36 failures total)
```
webgpu:shader,validation,types,textures:storage_texture_types:access="read";format="r16unorm";comma=""
webgpu:shader,validation,types,textures:storage_texture_types:access="read";format="r16snorm";comma=""
webgpu:shader,validation,types,textures:storage_texture_types:access="write";format="rg16unorm";comma=""
webgpu:shader,validation,types,textures:storage_texture_types:access="read_write";format="rgba16unorm";comma=""
... (and many more)
```

**Error:**
```
Shader validation error: Global variable [0] 'tex' is invalid
  ┌─ :1:23
  │
1 │ @group(0) @binding(0) var tex: texture_storage_2d<r16unorm, read>;
  │                       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ naga::ir::GlobalVariable [0]
  │
  = Capability Capabilities(STORAGE_TEXTURE_16BIT_NORM_FORMATS) is not supported

---- shader ----
@group(0) @binding(0) var tex: texture_storage_2d<r16unorm, read>;
```

**Root cause:**
The CTS tests run with all features enabled, including `texture-formats-tier1` which enables 16-bit normalized formats as storage textures. According to the WebGPU spec and CTS format info, these formats should be usable in shaders when the feature is available. However, Naga is rejecting these shaders saying the `STORAGE_TEXTURE_16BIT_NORM_FORMATS` capability is not supported.

The issue is that Naga doesn't have a way to know what device features are enabled when validating the shader. The shader compilation happens independently of device feature checking, and the comment in the CTS test says "the shader compilation should always pass regardless of whether the format supports the usage indicated by the access or not."

**Fix needed:**
This appears to be a fundamental architectural issue. According to the WebGPU spec and CTS expectations, shader compilation should succeed for these formats, and the validation should happen later when binding the shader to a pipeline (checking that the device actually supports the required features). Currently, Naga is doing format validation too early in the pipeline.

This would require changes to:
1. Allow these formats to pass shader validation in Naga
2. Move the feature checking to the pipeline/bind group layout validation stage in wgpu-core

## Summary of Required Fixes

1. **texture_external** (2 failures) - Known limitation, requires implementing external texture support
2. **Atomic operation validation** (3 failures) - Tracked in issue #5474, needs Naga validator changes
3. **Atomic address space validation** (1 failure) - Needs Naga validator changes
4. **Pointer access mode validation** (2 failures) - Needs Naga validator changes
5. **Pointer to atomic validation** (3 failures) - Needs Naga validator changes
6. **16-bit norm storage formats** (36 failures) - Architectural issue, shader validation should not reject these formats

Most of these issues (10 out of 47 failures) are related to atomic type validation in Naga. The largest category (36 failures) is the 16-bit normalized format issue which appears to be an architectural problem with when/where format validation occurs.
