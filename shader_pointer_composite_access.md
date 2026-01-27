# Shader Pointer Composite Access CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,extension,pointer_composite_access:*`

**Overall Status:** 14P/14F/0S (50%/50%/0%)

## Passing Sub-suites

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `deref` | 14/14 (100%) | Tests pointer dereference syntax (`(*p)[0]`, `(*p).x`) |

## Remaining Issues

| Subcategory | Pass Rate | Description |
|-------------|-----------|-------------|
| `pointer` | 0/14 (0%) | Tests direct pointer access syntax (`p[0]`, `p.x`) |

## Issue Detail

### 1. Missing wgslLanguageFeatures API causes false negatives

**Test selector:** `webgpu:shader,validation,extension,pointer_composite_access:pointer:*`

**What it tests:** Validates that when the `pointer_composite_access` language feature is supported, shaders can use direct pointer access syntax (`p[0]`, `p.x`) instead of requiring explicit dereference (`(*p)[0]`, `(*p).x`).

**Error:**
```
EXPECTATION FAILED: Expected validation error
```

**Root cause:**

The deno_webgpu backend does not expose the `wgslLanguageFeatures` API on the `GPU` object.

The test logic:
1. CTS test queries `gpu.wgslLanguageFeatures.has('pointer_composite_access')`
2. If the feature IS advertised, the test expects shaders using `p[0]` syntax to compile successfully
3. If the feature is NOT advertised, the test expects shaders using `p[0]` syntax to fail with a compilation error

The problem:
- `deno_webgpu/lib.rs` - The `GPU` struct only has `request_adapter()` and `getPreferredCanvasFormat()` methods, no `wgslLanguageFeatures` property
- CTS sees the feature as NOT supported (undefined/empty)
- CTS expects shader compilation to FAIL
- But Naga actually implements `pointer_composite_access` unconditionally (see `naga/src/front/wgsl/lower/mod.rs` lines 2365-2395)
- Shader compilation SUCCEEDS
- Test fails with "Expected validation error"

The wgpu-core backend (`wgpu/src/backend/wgpu_core.rs` lines 880-898) DOES properly report language features including `PointerCompositeAccess`, but this is not exposed through the deno_webgpu integration.

**Fix needed:**

The deno_webgpu backend needs to expose `wgslLanguageFeatures` on the `GPU` object:

1. Add a `wgslLanguageFeatures` getter to the `GPU` struct in `deno_webgpu/lib.rs`
2. Create a `GPUWgslLanguageFeatures` set-like object (similar to `GPUSupportedFeatures`)
3. Populate it with implemented language features from `ImplementedLanguageExtension::all()`

Once the feature is properly advertised, all 14 failing tests should pass because:
- CTS will see `pointer_composite_access` IS supported
- CTS will expect shaders to compile successfully
- Shaders DO compile successfully (as they already do)
