Perfect! I can see in the triage.md that `unpack4x8snorm` is already listed in the "Missing Constant Evaluation Support" section. Now let me look at the specific test failures to better understand what's happening.

Based on my analysis, I now have a comprehensive understanding of the triage. Let me summarize the findings.

## CTS Triage Report: `webgpu:shader,validation,expression,call,builtin,unpack4x8snorm:*`

### Overall Statistics
**Test Results:** 13P/3F/0S (81.25% pass, 18.75% fail, 0% skip)

### Root Cause Analysis

All 3 failures are caused by the same underlying issue: **Naga does not implement constant evaluation support for the `unpack4x8snorm` builtin function**.

The `unpack4x8snorm` function is explicitly listed as not implemented in Naga's constant evaluator at `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs:1871-1879`, where it returns `ConstantEvaluatorError::NotImplemented`.

### Detailed Failure Breakdown

#### 1. **Test: `values:stage="constant";type="u32"`**
- **What it tests:** Validates that `unpack4x8snorm` can be evaluated in constant expressions with various u32 input values
- **Error:** `"Not implemented as constant expression: Unpack4x8snorm built-in function"`
- **Impact:** Multiple subcases fail when trying to use `unpack4x8snorm` in a `const` declaration
- **Example failing shader:**
  ```wgsl
  const v = unpack4x8snorm(0u);
  ```

#### 2. **Test: `values:stage="override";type="u32"`**
- **What it tests:** Validates that `unpack4x8snorm` can be evaluated at pipeline creation time with override values
- **Error:** `"Pipeline constant error: MSL: ConstantEvaluatorError(NotImplemented("Unpack4x8snorm built-in function"))"`
- **Impact:** Fails when the function is used in an override-evaluated context
- **Example context:** Using pipeline constants with `unpack4x8snorm` in variable initialization

#### 3. **Test: `arguments:args="good"`**
- **What it tests:** Validates that `unpack4x8snorm` can be called with correct arguments and explicit return type annotation
- **Error:** `"Not implemented as constant expression: Unpack4x8snorm built-in function"`
- **Impact:** Only fails when testing with the correct return type `vec4<f32>` in constant context
- **Example failing shader:**
  ```wgsl
  const v : vec4<f32> = unpack4x8snorm(1u);
  ```

### Passing Tests
The following test categories pass successfully:
- **`values:stage="constant";type="abstract-int"`** - Passes (likely because abstract-int is out of u32 range and validation correctly rejects it for other reasons)
- **`arguments:args="bad_*"`** - All pass (correctly reject invalid argument types like i32, f32, vectors, etc.)
- **`must_use:*`** - Both pass (correctly enforce that the return value must be used)

### Related Issue
This is tracked as a **Known Issue (#4507)** - "Functions Missing Constant Evaluation Support"

The issue is already documented in `/Users/Andy/Development/wgpu2/docs/cts-triage/triage.md` under the "Missing Constant Evaluation Support" section, which lists `unpack4x8snorm` along with other unpack functions that need constant evaluation implementation.

### Suggested Updates

**For fail.lst:**
The current entry is appropriate:
```
webgpu:shader,validation,expression,call,builtin,unpack4x8snorm:* // 81%
```

However, it could be more descriptive:
```
webgpu:shader,validation,expression,call,builtin,unpack4x8snorm:* // 81%, const eval not implemented (#4507)
```

**For triage.md:**
No changes needed - already documented in the "Missing Constant Evaluation Support" section at lines 118-124.

### Summary for User

1. **Root cause:** Naga's constant evaluator does not implement the `unpack4x8snorm` builtin function. The function is explicitly listed as `NotImplemented` in the constant evaluator code.

2. **Bug reference:** Known issue #4507 - "Functions Missing Constant Evaluation Support"

3. **Triage summary:** 
   - 81.25% pass rate (13/16 tests)
   - 3 failures all stem from missing constant evaluation support
   - Runtime usage of `unpack4x8snorm` works correctly
   - Tests for argument validation and must_use semantics all pass
   - This is part of a broader gap where multiple pack/unpack functions lack constant evaluation support
