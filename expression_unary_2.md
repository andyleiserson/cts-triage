Excellent! I've completed the triage analysis. Let me create a summary of my findings:

## Triage Summary: `webgpu:shader,validation,expression,unary,*` (74% pass rate)

I've completed a comprehensive triage of the unary expression validation tests. Here's what I found:

### Overall Statistics
- **Total tests:** 304
- **Pass rate:** 225/304 (74.01%)
- **Failures:** 79 tests

### Breakdown by Subcategory

1. **logical_negation** ✅
   - 48/48 pass (100%)
   - No issues

2. **address_of_and_indirection** ⚠️
   - 96/172 pass (55.81%)
   - 76 failures in the `composite` subtest

3. **arithmetic_negation** ⚠️
   - 40/42 pass (95.24%)
   - 2 failures (matrix and atomic types)

4. **bitwise_complement** ⚠️
   - 41/42 pass (97.62%)
   - 1 failure (atomic type)

### Root Causes

**1. Missing `wgslLanguageFeatures` API (76 failures - 96% of all failures)**
- **Issue:** The `pointer_composite_access` language feature is implemented in Naga but not advertised through the WebGPU API
- **Impact:** Tests expect compilation to fail when the feature isn't advertised, but shaders compile successfully
- **Reference:** Known issue #8884
- **Fix:** Expose `wgslLanguageFeatures` on the `GPU` object in `deno_webgpu`

**2. Atomic Direct Reference (2 failures)**
- **Issue:** Naga allows direct references to atomics in expressions (e.g., `-a`, `~a`) instead of requiring atomic builtin functions
- **Reference:** Known issue #5474
- **Fix:** Requires broader fix to enforce atomic operations go through builtin functions

**3. Matrix Negation Validation Gap (1 failure)**
- **Issue:** Naga accepts `-m` where `m` is a matrix, but WebGPU spec only allows negation on scalar and vector types
- **Fix:** Add validation in Naga's WGSL frontend to reject unary negation on matrix types

### Documentation Created

I've created a detailed triage report at:
`/Users/Andy/Development/wgpu2/docs/cts-triage/expression_unary_triage.md`

The report includes:
- Detailed breakdown of each failure category
- Example failing test selectors
- Error messages
- Root cause analysis
- Fix recommendations
- References to related issues and documentation
