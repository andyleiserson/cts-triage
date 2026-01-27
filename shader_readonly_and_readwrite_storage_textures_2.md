Perfect! Now let me create a summary of the investigation for the user.

## Investigation Complete

I've successfully investigated the `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*` selector and created a comprehensive triage document.

### Key Findings

**Test Status:** 0% pass rate (0P/2F/0S) - Both tests are failing

**Root Cause:** Missing `wgslLanguageFeatures` API in deno_webgpu

The tests are failing because:

1. **The CTS test logic expects** the feature to be advertised via `gpu.wgslLanguageFeatures.has('readonly_and_readwrite_storage_textures')`
2. **deno_webgpu doesn't expose** the `wgslLanguageFeatures` property on the GPU object
3. **CTS sees the feature as NOT supported**, so it expects shader compilation to FAIL
4. **But Naga actually implements the feature** fully:
   - Storage texture access modes `read` and `read_write` are supported
   - The `textureBarrier()` builtin is implemented
   - The feature is listed in `ImplementedLanguageExtension::ReadOnlyAndReadWriteStorageTextures`
5. **Shader compilation SUCCEEDS** when CTS expects it to fail → Test fails

### Two Failing Tests

1. **`var_decl`** - Tests that `read` and `read_write` access modes on storage textures should fail without the language feature
2. **`textureBarrier`** - Tests that the `textureBarrier()` builtin should fail without the language feature

### Additional Issue Found

The `var_decl` test also encounters a secondary problem with 16-bit normalized formats (r16unorm, r16snorm, etc.). These fail because wgpu requires the `STORAGE_TEXTURE_16BIT_NORM_FORMATS` capability, which isn't being exposed during CTS runs.

### Solution

This is the **same root cause** as documented in two other triage documents:
- `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_pointer_composite_access_triage.md` (14 tests failing)
- `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_parse_requires_triage.md` (3 tests failing + 15 skipping)

The fix is to implement `wgslLanguageFeatures` in deno_webgpu to properly advertise which WGSL language extensions are supported. This is tracked in https://github.com/gfx-rs/wgpu/pull/8884.

### Documentation

I've created a detailed triage document at:
**`/Users/Andy/Development/wgpu2/docs/cts-triage/shader_readonly_and_readwrite_storage_textures_triage.md`**

This document includes test details, error messages, root cause analysis, and the needed fix.
