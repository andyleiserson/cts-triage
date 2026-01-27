# Readonly and Readwrite Storage Textures Extension CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*`

Model: Sonnet 4.5

**Overall Status:** 0P/2F/0S (0%/100%/0%)

## Passing Sub-suites
None.

## Remaining Issues

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `var_decl` | 0/1 (0%) | Tests that `read` and `read_write` access modes require the language feature |
| `textureBarrier` | 0/1 (0%) | Tests that `textureBarrier()` builtin requires the language feature |

## Issue Detail

### 1. Missing wgslLanguageFeatures API causes false negatives

**Test selector:** `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*`

**What it tests:** Validates that when the `readonly_and_readwrite_storage_textures` language feature is NOT advertised as supported, shaders using:
1. Storage texture access modes `read` or `read_write` (instead of the default `write`)
2. The `textureBarrier()` builtin function

should fail to compile with a validation error.

**Example failures:**

`var_decl` test:
```
webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:var_decl:type="texture_storage_1d";format="rgba8unorm";access="read"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
@group(0) @binding(0) var t : texture_storage_1d<rgba8unorm, read>;
```

`textureBarrier` test:
```
webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:textureBarrier:
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
@workgroup_size(1) @compute fn main() {
    textureBarrier();
}
```

**Root cause:**

The deno_webgpu backend does not expose the `wgslLanguageFeatures` API on the `GPU` object (see `/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs`).

The test logic:
1. CTS test queries `gpu.wgslLanguageFeatures.has('readonly_and_readwrite_storage_textures')`
2. Since `wgslLanguageFeatures` is undefined, the test sees the feature as NOT supported
3. CTS expects shader compilation to FAIL (because the feature is "not supported")
4. But Naga actually implements this feature unconditionally:
   - Storage texture access modes `read` and `read_write` are parsed and accepted (see `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/directive/language_extension.rs:26`)
   - The `textureBarrier()` builtin is implemented (see `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/lower/mod.rs:2970`)
   - The feature is listed in `ImplementedLanguageExtension::ReadOnlyAndReadWriteStorageTextures` (see `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/directive/language_extension.rs:58`)
5. Shader compilation SUCCEEDS
6. Test fails with "Expected validation error"

**Additional complication - 16-bit norm formats:**

The `var_decl` test also encounters a separate issue with 16-bit normalized formats (r16unorm, r16snorm, rg16unorm, rg16snorm, rgba16unorm, rgba16snorm). These formats fail with:

```
VALIDATION FAILED: Unexpected compilationInfo 'error' message.
Shader validation error: Capability Capabilities(STORAGE_TEXTURE_16BIT_NORM_FORMATS) is not supported
```

This is because:
1. The CTS expects these formats to work when `access="write"` (the default, which doesn't require the language feature)
2. But wgpu/Naga requires the `STORAGE_TEXTURE_16BIT_NORM_FORMATS` capability for these formats
3. This capability is not being exposed/enabled during CTS runs

This is a separate issue from the missing `wgslLanguageFeatures` API, but causes additional test failures within the `var_decl` suite.

**Fix needed:**

The primary fix needed is to expose `wgslLanguageFeatures` on the `GPU` object in deno_webgpu:

1. Add a `wgslLanguageFeatures` getter to the `GPU` struct in `/Users/Andy/Development/wgpu2/deno_webgpu/lib.rs`
2. Create a `GPUWgslLanguageFeatures` set-like object (similar to `GPUSupportedFeatures`)
3. Populate it with implemented language features from `ImplementedLanguageExtension::all()`
4. Include `readonly_and_readwrite_storage_textures` in the set

Once the feature is properly advertised:
- CTS will see `readonly_and_readwrite_storage_textures` IS supported
- CTS will expect shaders using `read`/`read_write` access and `textureBarrier()` to compile successfully
- Shaders DO compile successfully (as they already do)
- Both tests should pass

This is the same root cause and fix as documented in:
- `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_pointer_composite_access_triage.md`
- `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_parse_requires_triage.md`

The 16-bit norm format issue is a secondary problem that may need separate investigation if those formats are supposed to be supported by default.

**Related issue:** https://github.com/gfx-rs/wgpu/pull/8884 (PR for implementing wgslLanguageFeatures)
