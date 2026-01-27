Now I have all the information I need to provide a comprehensive triage report. Let me compile the findings.

## CTS Triage Report: ldexp Builtin Validation Tests

Based on my analysis of the `webgpu:shader,validation,expression,call,builtin,ldexp:*` test selector, here are my findings:

### 1. Root Cause

**The `ldexp` builtin function is not implemented for constant evaluation in Naga.**

The error message clearly states: `"Not implemented as constant expression: Ldexp built-in function"`

In `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` (lines 1846-1879), `ldexp` is explicitly listed among the unimplemented math functions for constant evaluation:

```rust
// unimplemented
crate::MathFunction::Modf
| crate::MathFunction::Frexp
| crate::MathFunction::Ldexp
| crate::MathFunction::Outer
| crate::MathFunction::FaceForward
...
=> Err(ConstantEvaluatorError::NotImplemented(
    format!("{fun:?} built-in function"),
))
```

### 2. Test Breakdown

**Overall Status:** 43P/57F/0S (43% pass rate)

The test suite has 4 subcategories with clear patterns:

#### Passing Tests (43 tests):
- **`partial_values:stage="runtime"`** - All runtime value tests pass because they don't require constant evaluation
- **`args:*`** - All argument validation tests pass (15 tests)
- **`must_use:*`** - Both must-use tests pass (2 tests)

#### Failing Tests (57 tests):
- **`values:stage="constant"`** - ALL constant evaluation tests fail (57 tests)
  - Tests all type combinations: abstract-int, abstract-float, f32, f16, i32
  - Tests scalar and vector types (vec2, vec3, vec4)
  - All fail with the same error: constant evaluation not implemented

The failure is exclusively in constant evaluation. Runtime evaluation works perfectly, as do all other validation checks (argument types, must-use enforcement).

### 3. Suggested Bug Reference for fail.lst

```
webgpu:shader,validation,expression,call,builtin,ldexp:* // 43% - Naga: ldexp constant eval not implemented
```

Or more concisely:
```
webgpu:shader,validation,expression,call,builtin,ldexp:* // 43% - ldexp const eval unimplemented
```

### 4. Summary for triage.md

```markdown
### ldexp

**Test selector:** `webgpu:shader,validation,expression,call,builtin,ldexp:*`

**Overall Status:** 43P/57F/0S (43% pass rate)

**Root cause:** Naga does not implement constant evaluation for the `ldexp` builtin function. The function is marked as unimplemented in `naga/src/proc/constant_evaluator.rs:1849`.

**Failing tests:** All `values:stage="constant"` tests fail (57 tests) - these test constant evaluation with various type combinations (abstract-int, abstract-float, f32, f16, i32) for both scalars and vectors.

**Passing tests:** 
- All `partial_values:stage="runtime"` tests pass (8 tests)
- All `args:*` tests pass (15 tests) 
- All `must_use:*` tests pass (2 tests)
- Remaining passing tests (18 tests) include `partial_values` with constant/override stages

**Fix needed:** Implement constant evaluation for `ldexp(e1, e2)` in Naga's constant evaluator. The function computes `e1 * 2^e2` and needs to handle overflow/underflow according to WGSL spec. This is a sibling builtin to `frexp` (also unimplemented) - they form a pair for floating-point decomposition/composition.

**Related:** `frexp` (39% pass), `modf` (52% pass) - other unimplemented decomposition builtins in constant evaluator.
```

This is a straightforward implementation gap in Naga. The `ldexp` function works correctly at runtime but needs constant evaluation support to pass the CTS validation tests that use it in constant expressions.
