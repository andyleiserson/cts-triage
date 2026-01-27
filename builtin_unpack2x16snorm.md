Excellent! Now I have a complete picture. Let me create the triage report:

## CTS Triage Report: `webgpu:shader,validation,expression,call,builtin,unpack2x16snorm:*`

### Overall Status
**13P/3F/0S (81.25%/18.75%/0%)**

### Test Breakdown

#### Passing Tests (13 tests - 81.25%)
1. **`values:stage="constant";type="abstract-int"`** - Constant evaluation with abstract-int (passes because abstract-int values are evaluated at compile time before the builtin is called)
2. **`arguments:args="bad_*"`** (11 tests) - All argument validation tests pass correctly:
   - `bad_no_args` - Correctly rejects missing arguments
   - `bad_more_args` - Correctly rejects too many arguments  
   - `bad_i32`, `bad_f32`, `bad_f16`, `bad_bool` - Correctly reject wrong scalar types
   - `bad_vec2u`, `bad_vec3u`, `bad_vec4u` - Correctly reject vector arguments
   - `bad_array` - Correctly rejects array arguments
3. **`must_use:use=true`** and **`must_use:use=false`** - @must_use validation works correctly

#### Failing Tests (3 tests - 18.75%)

All three failures have the **same root cause**: Naga does not support `unpack2x16snorm` in constant evaluation.

##### 1. `values:stage="constant";type="u32"`
**What it tests:** Validates that `unpack2x16snorm` works with concrete u32 values in constant expressions.

**Error:**
```
Shader '' parsing error: Not implemented as constant expression: Unpack2x16snorm built-in function
  ┌─ wgsl:2:12
  │
2 │ const v  = unpack2x16snorm(0u);
  │            ^^^^^^^^^^^^^^^ see msg
```

**Root cause:** In `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` lines 1873-1879, `MathFunction::Unpack2x16snorm` is explicitly listed as `NotImplemented` for constant evaluation.

##### 2. `values:stage="override";type="u32"`
**What it tests:** Validates that `unpack2x16snorm` works with override (pipeline constant) values.

**Error:**
```
Pipeline constant error: MSL: ConstantEvaluatorError(NotImplemented("Unpack2x16snorm built-in function"))
```

**Root cause:** Same as above - override values require constant evaluation at pipeline creation time, which fails because the constant evaluator doesn't support this function.

##### 3. `arguments:args="good"`
**What it tests:** Validates that `unpack2x16snorm(1u)` with explicit `vec2<f32>` return type works in constant expressions.

**Error:**
```
Shader '' parsing error: Not implemented as constant expression: Unpack2x16snorm built-in function
  ┌─ wgsl:2:23
  │
2 │ const v : vec2<f32> = unpack2x16snorm(1u);
  │                       ^^^^^^^^^^^^^^^ see msg
```

**Root cause:** Same as above.

### Root Cause Analysis

The issue is in `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs` at lines 1862-1879. All pack/unpack functions are listed as not implemented:

```rust
| crate::MathFunction::Pack4x8snorm
| crate::MathFunction::Pack4x8unorm
| crate::MathFunction::Pack2x16snorm
| crate::MathFunction::Pack2x16unorm
| crate::MathFunction::Pack2x16float
| crate::MathFunction::Pack4xI8
| crate::MathFunction::Pack4xU8
| crate::MathFunction::Pack4xI8Clamp
| crate::MathFunction::Pack4xU8Clamp
| crate::MathFunction::Unpack4x8snorm
| crate::MathFunction::Unpack4x8unorm
| crate::MathFunction::Unpack2x16snorm  // <- THIS ONE
| crate::MathFunction::Unpack2x16unorm
| crate::MathFunction::Unpack2x16float
| crate::MathFunction::Unpack4xI8
| crate::MathFunction::Unpack4xU8 => Err(ConstantEvaluatorError::NotImplemented(
    format!("{fun:?} built-in function"),
)),
```

### Suggested Bug Reference for fail.lst
The current entry in `/Users/Andy/Development/wgpu2/cts_runner/fail.lst` line 162 is:
```
webgpu:shader,validation,expression,call,builtin,unpack2x16snorm:* // 81%
```

**Suggested update:**
```
webgpu:shader,validation,expression,call,builtin,unpack2x16snorm:* // 81% - #4507 missing constant evaluation
```

### Summary for triage.md

This test suite is already documented in `/Users/Andy/Development/wgpu2/docs/cts-triage/triage.md` under the "Missing Constant Evaluation Support" section (line 120):

```markdown
data unpacking

- [ ] `unpack4x8snorm`
- [ ] `unpack4x8unorm`
- [ ] `unpack2x16snorm`  <- Already documented here
- [ ] `unpack2x16unorm`
- [ ] `unpack2x16float`
- [ ] `unpack4xI8`
- [ ] `unpack4xI8`
```

**No additional section needed** - this is part of the known limitation that all pack/unpack functions lack constant evaluation support. The issue is tracked as #4507 (Functions Missing Constant Evaluation Support).

### Implementation Notes

To fix this issue, someone would need to:

1. Implement constant evaluation for `Unpack2x16snorm` in the `math_function` method of `Evaluator` in `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs`
2. The implementation would need to:
   - Extract a u32 literal from the argument
   - Unpack the two 16-bit signed normalized values
   - Convert each to f32 by dividing by 32767.0 and clamping to [-1.0, 1.0]
   - Return a `vec2<f32>` constant expression

This is a medium-complexity fix that follows the pattern of other constant evaluation implementations in the same file.

---

**Related Issue:** https://github.com/gfx-rs/wgpu/issues/4507
