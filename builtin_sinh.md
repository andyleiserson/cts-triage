So sinh is on the checklist but hasn't been triaged yet. Now let me check what other builtin validation tests look like to see if there's a pattern. Let me look at one that's similar and see if it has the same issue.

Let me verify my understanding by checking the percentage. 61% pass rate means roughly 39% failure - which matches my finding of 16/41 = 39% failing.

Now I have enough information to write the triage report. Let me create a summary:

## Summary of Findings

**Test Selector:** `webgpu:shader,validation,expression,call,builtin,sinh:*`

**Overall Status:** 25P/16F/0S (60.98% / 39.02% / 0%)

### Passing Sub-suites
- `args:*` - All 15 tests pass (argument type validation)
- `must_use:*` - All 2 tests pass (must-use validation)

### Failing Sub-suite
- `values:*` - 8P/16F (33.33% pass rate)
  - All f32 tests PASS (both constant and override stages)
  - All abstract-int tests FAIL
  - All abstract-float tests FAIL  
  - All f16 tests FAIL (both constant and override stages)

### Root Cause

The `sinh()` builtin function in Naga's constant evaluator does not validate that the result is representable in the target type. According to the WGSL spec (line 2614 of wgsl-spec.bs):

> An evaluation of an expression in one of these types [abstract types] [=shader-creation error|must not=] overflow or produce an infinite or NaN value.

When `sinh()` is called with large values:
- For abstract-int/abstract-float: The result computation produces infinity or overflows, but Naga doesn't reject it
- For f16: Values like `sinh(65504.0h)` overflow f16's range but aren't caught
- For f32: The validation DOES work correctly (likely because f32 has enough range or existing checks catch it)

The issue is in `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` at line 1564:
```rust
crate::MathFunction::Sinh => {
    component_wise_float!(self, span, [arg], |e| { Ok([e.sinh()]) })
}
```

This simply computes `.sinh()` without checking if the result is representable (finite, not NaN) for abstract types.

### Related Issues

This is a systematic issue affecting multiple hyperbolic and other math functions:
- `cosh:*` - 61% pass (same 66.67% failure rate on values tests)
- `asinh:*` - 85% pass
- `acosh:*` - 56% pass
- `atanh:*` - 72% pass

All show similar patterns of abstract type and f16 constant evaluation not validating overflow/infinity results.

---

Based on this analysis:

1. **Root cause**: Naga's constant evaluator doesn't validate that math function results are representable (finite, not NaN) for abstract types and f16
2. **Bug reference**: "Missing overflow/infinity validation for abstract types and f16 in const eval"
3. **Triage summary**: See above
