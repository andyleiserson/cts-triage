This is very clear - these are NOT the same as issue #5474. These failures are about:

1. **Stage validation**: Atomic builtin functions (atomicLoad, atomicStore, etc.) should NOT be allowed in vertex shaders, but wgpu is accepting them.

2. **Address space validation**: Atomics should only be allowed in `storage` (with `read_write` access) and `workgroup` address spaces, but wgpu is accepting them in:
   - `storage` with `read` access (should require `read_write`)
   - `private` address space (should be rejected)
   - `function` address space (should be rejected)

These are distinct validation gaps from issue #5474, which is about direct references to atomics vs. requiring atomic builtin functions.

Now I have all the information I need. Let me prepare the final triage report.

## CTS Triage Report: Atomic Builtin Validation

### Summary

**CTS Selector:** `webgpu:shader,validation,expression,call,builtin,atomics:*`

**Model:** Claude Sonnet 4.5

**Overall Status:** 132P/22F/0S (85.71% pass rate)

### Test Breakdown

- **Total tests:** 154
- **Passing:** 132 (85.71%)
- **Failing:** 22 (14.29%)

### Passing Sub-suites ✅

- `data_parameters:*` - 11P/0F (100%) - Validates that atomic operation parameters match atomic type
- `return_types:*` - 11P/0F (100%) - Validates return types of atomic operations
- `non_atomic:*` - 88P/0F (100%) - Validates that non-atomic integers are rejected by atomic functions
- `stage:stage="compute";*` - 11P/0F (100%) - Atomic operations in compute shaders
- `stage:stage="fragment";*` - 11P/0F (100%) - Atomic operations in fragment shaders

### Remaining Issues ⚠️

- `stage:stage="vertex";*` - 0P/11F (0%) - Atomic operations incorrectly accepted in vertex shaders
- `atomic_parameterization:*` - 0P/11F (0%) - Atomics incorrectly accepted in invalid address spaces/access modes

---

## Issue Detail

### 1. Atomic Operations in Vertex Shaders (11 failures)

**Test selector:** `webgpu:shader,validation,expression,call,builtin,atomics:stage:stage="vertex";*`

**What it tests:** Per the WGSL spec, atomic built-in functions must not be used in vertex shader stages. The test validates that all atomic operations (atomicAdd, atomicSub, atomicLoad, atomicStore, etc.) are rejected when used in a vertex shader.

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,atomics:stage:stage="vertex";atomicOp="load"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----

@group(0) @binding(0) var<storage, read_write> a: atomic<i32>;

@vertex fn vmain() -> @builtin(position) vec4<f32> {
  atomicLoad(&a);
  return vec4<f32>();
}
```

**Root cause:**
wgpu/Naga is not validating that atomic builtin functions are prohibited in vertex shader entry points. The shader compiler accepts atomic operations in vertex shaders when it should reject them with a compilation error.

**Spec reference:** 
According to the test source (line 52-55), the WGSL spec at https://www.w3.org/TR/WGSL/#atomic-rmw states: "Atomic built-in functions must not be used in a vertex shader stage."

**Fix needed:**
Add validation in Naga's shader validator to check the shader stage when processing atomic builtin function calls. If the atomic operation is called from a vertex shader entry point (or a function called by a vertex entry point), emit a validation error.

**Affected operations:** All 11 atomic operations fail this test:
- atomicAdd, atomicSub, atomicMax, atomicMin
- atomicAnd, atomicOr, atomicXor
- atomicLoad, atomicStore
- atomicExchange, atomicCompareExchangeWeak

---

### 2. Atomic Address Space and Access Mode Validation (11 failures)

**Test selector:** `webgpu:shader,validation,expression,call,builtin,atomics:atomic_parameterization:*`

**What it tests:** Validates that atomic operations are only accepted for atomics in valid address spaces with correct access modes. Per the test source (lines 177-180), atomics should only be valid in:
- `storage` address space with `read_write` access
- `workgroup` address space (always read_write)

**Example failures:**

**Storage with read-only access:**
```
(in subcase: aspace="storage";access="read";type="i32";style="var")

---- shader ----

@group(0) @binding(0) var<storage, read> a : atomic<i32>;

fn foo() {
  atomicAdd(&a,1);
}
```

**Private address space:**
```
(in subcase: aspace="private";access="read_write";type="i32";style="param")

---- shader ----

fn foo(p : ptr<private, atomic<i32>>) {
  atomicAdd(p,1);
}
```

**Function address space:**
```
(in subcase: aspace="function";access="read_write";type="i32";style="param")

---- shader ----

fn foo(p : ptr<function, atomic<i32>>) {
  atomicAdd(p,1);
}
```

**Root cause:**
wgpu/Naga is not validating the address space and access mode of atomic types when atomic builtin functions are called. It accepts:

1. **Storage atomics with `read` access** - Should require `read_write` access (2 subcases per operation × 11 operations = 22 subcases)
2. **Private address space atomics** - Should be rejected entirely (2 subcases per operation × 11 operations = 22 subcases)  
3. **Function address space atomics** - Should be rejected entirely (2 subcases per operation × 11 operations = 22 subcases)

Each of the 11 atomic operations has exactly 6 failing subcases (2 + 2 + 2), but they're grouped into a single test per operation, resulting in 11 test failures.

**Fix needed:**
Add validation in Naga when resolving atomic builtin function calls:
1. Check that the atomic pointer's address space is either `storage` or `workgroup`
2. If the address space is `storage`, verify the access mode is `read_write` (not just `read`)
3. Reject atomics in `private`, `function`, and `uniform` address spaces

**Affected operations:** All 11 atomic operations have this validation gap:
- atomicAdd, atomicSub, atomicMax, atomicMin
- atomicAnd, atomicOr, atomicXor
- atomicLoad, atomicStore
- atomicExchange, atomicCompareExchangeWeak

---

## Relationship to Known Issue #5474

These failures are **NOT** related to the known issue #5474. Issue #5474 is about Naga allowing direct references to atomic values in expressions (e.g., `let x = a;` where `a` is atomic) instead of requiring atomic builtin functions (e.g., `let x = atomicLoad(&a);`).

The failures in this test suite are different - they are about **missing validation** when atomic builtin functions ARE being used correctly, but in invalid contexts:
- Using atomic operations in vertex shaders (not allowed)
- Using atomics in invalid address spaces (private, function)
- Using atomics with invalid access modes (storage + read instead of read_write)

---

## Suggested Bug Reference for fail.lst

Current entry in `/Users/Andy/Development/wgpu2/cts_runner/fail.lst` line 99:
```
webgpu:shader,validation,expression,call,builtin,atomics:* // 86%
```

**Suggested update:**
```
webgpu:shader,validation,expression,call,builtin,atomics:* // 86%, atomics in vertex shaders, invalid address spaces/access modes
```

---

## Summary for triage.md

```markdown
## Atomic Builtin Validation

Selector: `webgpu:shader,validation,expression,call,builtin,atomics:*`

**Overall Status:** 132P/22F/0S (85.71% pass rate)

### Passing Tests (132 tests)
- Atomic parameter type validation (data_parameters)
- Atomic return type validation (return_types)
- Non-atomic type rejection (non_atomic)
- Atomics in compute shaders (stage:compute)
- Atomics in fragment shaders (stage:fragment)

### Remaining Issues (22 failures)

1. **Vertex shader stage validation gap (11 failures)** - wgpu accepts atomic operations in vertex shaders when they should be rejected per WGSL spec.

2. **Address space and access mode validation gap (11 failures)** - wgpu accepts atomics in invalid configurations:
   - Storage atomics with `read` access (should require `read_write`)
   - Atomics in `private` address space (only `storage` and `workgroup` are valid)
   - Atomics in `function` address space (only `storage` and `workgroup` are valid)

**Note:** These failures are distinct from issue #5474 (direct atomic references). These are about missing validation when atomic builtin functions are used in invalid contexts.
```
