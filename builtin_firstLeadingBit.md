Good! `firstTrailingBit` has the same issue. This makes sense since they're related builtins. Now let me prepare the comprehensive triage report.

## CTS Triage Complete

Based on my analysis of the `webgpu:shader,validation,expression,call,builtin,firstLeadingBit:*` test selector, here are the findings:

### 1. Specific Root Cause of the Failure

The single failing test (`webgpu:shader,validation,expression,call,builtin,firstLeadingBit:arguments:test="atomic"`) fails because **Naga allows referencing an atomic variable directly in an expression** (e.g., `firstLeadingBit(a)` where `a` is `var<workgroup> a: atomic<u32>`).

According to the WebGPU specification, atomic variables should only be accessed through atomic built-in functions like `atomicLoad()`, `atomicStore()`, etc. The `firstLeadingBit()` function expects a concrete integer type (`i32`, `u32`, or vectors of these), not an atomic type.

**Error pattern:** "EXPECTATION FAILED: Expected validation error" followed by "VALIDATION FAILED: Missing expected compilationInfo 'error' message."

This indicates wgpu is accepting invalid shader code that should be rejected.

### 2. Suggested Bug Reference for fail.lst

The current entry in `/Users/Andy/Development/wgpu2/cts_runner/fail.lst` (line 118) is:
```
webgpu:shader,validation,expression,call,builtin,firstLeadingBit:* // 98%
```

This should be updated to:
```
webgpu:shader,validation,expression,call,builtin,firstLeadingBit:arguments:test="atomic" // Naga #5474: atomic direct reference
```

The remaining 47 passing tests can be removed from fail.lst by excluding this specific failing test.

### 3. Summary for triage.md

**Test Suite:** `webgpu:shader,validation,expression,call,builtin,firstLeadingBit:*`

**Overall Status:** 47P/1F/0S (97.92%/2.08%/0.00%)

**Passing Sub-suites:**
- ✅ `values:*` - 16/16 pass (100%) - Constant evaluation tests
- ✅ `float_argument:*` - 13/13 pass (100%) - Type validation for float arguments
- ✅ `must_use:*` - 2/2 pass (100%) - Must-use attribute validation
- ✅ `arguments:*` - 16/17 pass (94.1%) - Most argument validation tests

**Remaining Issues:**

**1. Atomic Direct Reference in firstLeadingBit (1 failure)**

**Selector:** `webgpu:shader,validation,expression,call,builtin,firstLeadingBit:arguments:test="atomic"`

**What it tests:** Validates that passing an atomic variable directly to `firstLeadingBit()` should fail, since the builtin expects concrete integer types (i32/u32 or vectors), not atomic types.

**Example failure:**
```
webgpu:shader,validation,expression,call,builtin,firstLeadingBit:arguments:test="atomic"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
var<workgroup> a: atomic<u32>;
@vertex
fn main() -> @builtin(position) vec4<f32> {
  _ = firstLeadingBit(a);
  return vec4<f32>(.4, .2, .3, .1);
}
```

**Root cause:** Known Naga issue #5474. Naga allows referencing atomic variables directly in expressions instead of requiring atomic built-in functions (`atomicLoad`, `atomicStore`, etc.).

**Related issue:** https://github.com/gfx-rs/wgpu/issues/5474

**Status:** Known limitation - DO NOT FIX without coordinating on the Naga issue.

**Note:** This test suite should also be added to the "Affected test suites" list under the "Atomics Direct Reference (#5474)" section in `/Users/Andy/Development/wgpu2/docs/cts-triage/triage.md`, as it follows the same pattern as other affected suites like `expression,binary,and_or_xor`, `expression,binary,comparison`, etc.

The `firstTrailingBit` builtin (line 119 in fail.lst) has the identical issue and should receive the same treatment.
