# Compute Pipeline Overrides CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,compute_pipeline:overrides,identifier:*`

**Overall Status:** 12P/14F/0S (46%/54%/0%)

## Passing Sub-suites

The following valid constant identifier combinations pass:
- `constants={}` - empty constants
- `constants={"c0":0}` - valid override by name
- `constants={"c0":0,"c1":1}` - valid overrides by name
- `constants={"1":0}` - valid override by numeric ID (references c3 which has @id(1))
- `constants={"1000":0}` - valid override by numeric ID (references c2 which has @id(1000))
- `constants={"%E6%95%B0":0}` - valid override with Unicode name (CJK character)

## Remaining Issues

All 14 failures have the same root cause: wgpu does not validate that pipeline constant keys exist in the shader module.

## Issue Detail

### 1. Unknown pipeline constant identifiers silently accepted

**Test selector:** `webgpu:api,validation,compute_pipeline:overrides,identifier:*`

**What it tests:** When creating a compute pipeline with pipeline constants, the provided constant identifiers must match valid overrides in the shader module. Invalid identifiers should cause a validation error.

**Failing test cases:**

| Constants | Why it should fail |
|-----------|-------------------|
| `{"c0\u0000":0}` | Name contains null character |
| `{"c9":0}` | No override named "c9" exists |
| `{"c3":0}` | c3 has @id(1), so must be referenced by ID "1", not name |
| `{"2":0}` | No override has @id(2) |
| `{"9999":0}` | No override has @id(9999) |
| `{"1000":0,"c2":0}` | "c2" as name is invalid since c2 has @id(1000) |
| `{"sequencage":0}` (NFD) | WGSL does not normalize Unicode; NFD form doesn't match NFC |

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Validation succeeded unexpectedly.
```

**Root cause:**

The issue is in `naga/src/back/pipeline_constants.rs` in the `process_overrides` function. The code at lines 338-363 only:
1. Determines the lookup key for each override (either numeric ID or name)
2. Checks if a value was provided for required overrides

It does **NOT** validate that user-provided keys in `pipeline_constants` actually correspond to valid overrides in the shader. Unknown keys are silently ignored.

Per the WebGPU spec, providing a constant identifier that doesn't match any pipeline-overridable constant should be a validation error.

**Fix needed:**

Add validation in `process_overrides` to track which keys from `pipeline_constants` were consumed, then error if any keys remain unused. This requires:

1. A new error variant in `PipelineConstantError`:
```rust
#[error("Unknown pipeline constant identifier: '{0}'")]
UnknownIdentifier(String),
```

2. Build a set of valid identifiers (both names and numeric IDs) from the shader's overrides, then check that every key in `pipeline_constants` exists in that set.

## Key Files Referenced

- **Naga Implementation:** `naga/src/back/pipeline_constants.rs` (lines 338-363)
