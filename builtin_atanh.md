Now let me create a comprehensive triage document summarizing my findings:

## CTS Triage Summary for atanh

Based on my analysis, here are the findings:

### Overall Statistics
- **Test selector**: `webgpu:shader,validation,expression,call,builtin,atanh:*`
- **Total tests**: 58 (out of 70 listed, some are skipped due to f16 support)
- **Passed**: 42 / 58 (72.41%)
- **Failed**: 16 / 58 (27.59%)

### Passing Sub-suites
- `webgpu:shader,validation,expression,call,builtin,atanh:integer_argument:*` - 9/9 passing (100%)
- `webgpu:shader,validation,expression,call,builtin,atanh:parameters:*` - 25/25 passing (100%)
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="f32"` - Passing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="vec*<f32>"` - All passing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="override";type="f32"` - Passing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="override";type="vec*<f32>"` - All passing

### Remaining Issues
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="abstract-int"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="vec*<abstract-int>"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="abstract-float"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="vec*<abstract-float>"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="f16"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="constant";type="vec*<f16>"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="override";type="f16"` - Failing
- `webgpu:shader,validation,expression,call,builtin,atanh:values:stage="override";type="vec*<f16>"` - Failing

### Root Cause

**Naga does not validate the domain of `atanh()` during constant evaluation for abstract-float, abstract-int (implicitly converted to abstract-float), and f16 types.**

The mathematical function `atanh(x)` is only defined for values in the open interval `(-1, 1)`. According to the WebGPU WGSL specification, constant evaluation of `atanh()` must produce a shader-creation error if the input value is outside this domain (i.e., if `abs(x) >= 1`).

**Current behavior:**
- Naga accepts values like `atanh(-2)`, `atanh(1.5)`, `atanh(65504.0h)` etc. during constant evaluation
- These should produce shader-creation errors but are being accepted

**Location in code:**
- `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs`, line 1592-1594
- The `Atanh` case simply calls `component_wise_float!(self, span, [arg], |e| { Ok([e.atanh()]) })`
- No validation is performed to check if `abs(e) < 1.0`

**Similar issues:**
This appears to be part of a broader issue affecting multiple math builtins with domain restrictions:
- `acos` and `asin` - require domain `[-1, 1]` - also failing 16/56 tests (28.57%)
- `acosh` - requires domain `[1, ∞)` - failing 24/54 tests (44.44%)

### Suggested Bug Reference for fail.lst

```
webgpu:shader,validation,expression,call,builtin,atanh:values:* // Naga: missing domain validation for atanh constant eval (|x| < 1)
```

Or more concisely:
```
webgpu:shader,validation,expression,call,builtin,atanh:values:* // Naga: atanh domain not validated in const eval
```

### Summary for triage.md

**Issue**: Missing domain validation for `atanh()` in constant evaluation

**Affected tests**: 16 failures in `webgpu:shader,validation,expression,call,builtin,atanh:values:*`

**What's tested**: The CTS validates that constant/override evaluation of `atanh()` rejects values outside its mathematical domain `(-1, 1)`.

**Root cause**: Naga's constant evaluator (`naga/src/proc/constant_evaluator.rs:1592`) directly calls the Rust `atanh()` method without validating that the input satisfies `abs(x) < 1`. The same issue affects abstract-int (which gets implicitly converted to abstract-float), abstract-float, and f16 types.

**Fix needed**: Add domain validation in the constant evaluator for `atanh()`, similar to how `clamp()` validates that `low <= high`. Should return a new error variant (e.g., `InvalidAtanhDomain`) when `abs(value) >= 1`. This fix should also be applied to other domain-restricted functions: `acos`, `asin` (domain `[-1, 1]`), and `acosh` (domain `[1, ∞)`).
