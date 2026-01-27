# Render Pipeline Overrides CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,render_pipeline,overrides:*`

Model: Claude Sonnet 4.5

**Overall Status:** 140P/28F/0S (83.33%/16.67%/0%)

## Passing Sub-suites ✅

- `webgpu:api,validation,render_pipeline,overrides:uninitialized,vertex:*` - 8P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:uninitialized,fragment:*` - 8P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:value,type_error,vertex:*` - 8P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:value,type_error,fragment:*` - 8P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:value,validation_error,vertex:*` - 28P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:value,validation_error,fragment:*` - 28P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:value,validation_error,f16,vertex:*` - 16P/0F/0S (100%)
- `webgpu:api,validation,render_pipeline,overrides:value,validation_error,f16,fragment:*` - 16P/0F/0S (100%)

## Remaining Issues ⚠️

- `webgpu:api,validation,render_pipeline,overrides:identifier,vertex:*` - 10P/14F/0S (41.67%)
- `webgpu:api,validation,render_pipeline,overrides:identifier,fragment:*` - 10P/14F/0S (41.67%)

## Issue Detail

### 1. Unknown pipeline constant identifiers silently accepted

**Test selector:** `webgpu:api,validation,render_pipeline,overrides:identifier,vertex:*` and `webgpu:api,validation,render_pipeline,overrides:identifier,fragment:*`

**What it tests:** When creating a render pipeline with pipeline constants, the provided constant identifiers must match valid overrides in the shader module. Invalid identifiers should cause a validation error.

**Example failure (vertex):**
```
webgpu:api,validation,render_pipeline,overrides:identifier,vertex:isAsync=false;vertexConstants={"xxx":1}
```

**Error:**
```
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Shader context (vertex):**
The test shader defines these overrides:
```wgsl
override x: f32 = 0.0;
override y: f32 = 0.0;
override 数: f32 = 0.0;
override séquençage: f32 = 0.0;
@id(1) override z: f32 = 0.0;
@id(1000) override w: f32 = 1.0;
```

**Failing test cases (vertex):**

| Test Constants | Why it should fail | isAsync=true | isAsync=false |
|----------------|-------------------|--------------|---------------|
| `{"x\u0000":1,"y":1}` | Name contains null character | ✗ | ✗ |
| `{"xxx":1}` | No override named "xxx" exists | ✗ | ✗ |
| `{"2":1}` | No override has @id(2) | ✗ | ✗ |
| `{"z":1}` | z has @id(1), must use ID not name | ✗ | ✗ |
| `{"w":1}` | w has @id(1000), must use ID not name | ✗ | ✗ |
| `{"1":1,"z":1}` | Can't use both ID and name for z | ✗ | ✗ |
| `{"séquençage":0}` (NFD) | Unicode not normalized; NFD ≠ NFC | ✗ | ✗ |

**Passing test cases (vertex):**

| Test Constants | Why it should pass |
|----------------|-------------------|
| `{}` | Empty constants valid |
| `{"x":1,"y":1}` | Valid overrides by name |
| `{"1":1,"1000":1,"x":1,"y":1}` | Valid mix of IDs and names |
| `{"1":1}` | Valid override by ID (z) |
| `{"数":1}` | Valid Unicode override name |

**Shader context (fragment):**
The test shader defines these overrides:
```wgsl
override r: f32 = 0.0;
override g: f32 = 0.0;
override 数: f32 = 0.0;
override sequencage: f32 = 0.0;  // Note: NFC form in shader
@id(1) override b: f32 = 0.0;
@id(1000) override a: f32 = 0.0;
```

**Failing test cases (fragment):**

| Test Constants | Why it should fail | isAsync=true | isAsync=false |
|----------------|-------------------|--------------|---------------|
| `{"r\u0000":1}` | Name contains null character | ✗ | ✗ |
| `{"xxx":1}` | No override named "xxx" exists | ✗ | ✗ |
| `{"2":1}` | No override has @id(2) | ✗ | ✗ |
| `{"b":1}` | b has @id(1), must use ID not name | ✗ | ✗ |
| `{"a":1}` | a has @id(1000), must use ID not name | ✗ | ✗ |
| `{"1":1,"b":1}` | Can't use both ID and name for b | ✗ | ✗ |
| `{"séquençage":0}` (NFD) | Unicode not normalized; NFD ≠ NFC | ✗ | ✗ |

**Root cause:**

The issue is in `/Users/Andy/Development/wgpu2/naga/src/back/pipeline_constants.rs` in the `process_overrides` function (lines 328-376). The code only:
1. Determines the lookup key for each override (either numeric ID or name)
2. Checks if a value was provided for required overrides (those without initializers)

It does **NOT** validate that user-provided keys in `pipeline_constants` actually correspond to valid overrides in the shader. Unknown keys are silently ignored.

Per the WebGPU spec, providing a constant identifier that doesn't match any pipeline-overridable constant should be a validation error. Additionally:
- If an override has an `@id()` attribute, it can ONLY be referenced by that numeric ID
- If an override does NOT have an `@id()` attribute, it can ONLY be referenced by its name
- Names with null characters should be rejected
- Unicode names must match exactly (no normalization)

**Fix needed:**

Add validation in `process_overrides` to ensure all keys in `pipeline_constants` correspond to valid overrides. This requires:

1. Build a set of valid identifiers from the shader's overrides:
   - For overrides with `@id(N)`: add string `"N"` to valid set
   - For overrides without `@id()`: add the override's name to valid set

2. After processing all overrides, check that every key in `pipeline_constants` was in the valid set

3. Add a new error variant in `PipelineConstantError`:
```rust
#[error("Unknown pipeline constant identifier: '{0}'")]
UnknownIdentifier(String),
```

This is the same issue documented in `compute_pipeline_overrides_triage.md` and should be fixed together.

## Key Files Referenced

- **Test source:** `/Users/Andy/Development/cts/src/webgpu/api/validation/render_pipeline/overrides.spec.ts`
- **Naga implementation:** `/Users/Andy/Development/wgpu2/naga/src/back/pipeline_constants.rs` (process_overrides function, lines 328-376)
- **Related issue:** See `compute_pipeline_overrides_triage.md` for the same issue in compute pipelines
