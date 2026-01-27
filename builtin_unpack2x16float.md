Now I have a good understanding. Let me summarize my findings:

## Summary of Findings

Based on my investigation of the `webgpu:shader,validation,expression,call,builtin,unpack2x16float:*` test selector:

### 1. Root Cause
The failures are due to **missing constant evaluation support** for pack/unpack functions in Naga's constant evaluator. The constant evaluator explicitly returns `NotImplemented` for these functions (found at `/Users/Andy/Development/wgpu2/naga/src/proc/constant_evaluator.rs:1875-1879`).

### 2. What the Tests Check
The tests validate three aspects:
1. **`values` tests**: Verify that constant/override evaluation of `unpack2x16float` rejects u32 values that would unpack to NaN or Infinity (i.e., the two packed f16 values must both be finite)
2. **`arguments` tests**: Verify argument type validation (expects u32 or abstract-int, rejects other types)
3. **`must_use` tests**: Verify that the function result must be used (cannot be called as a statement)

### 3. Bug Reference
**Missing constant evaluation for pack/unpack builtins**

The issue affects all pack2x16/unpack2x16 and pack4x8/unpack4x8 variants:
- `pack2x16float` (80% pass)
- `pack2x16snorm` (88% pass)  
- `pack2x16unorm` (88% pass)
- `unpack2x16float` (81% pass)
- `unpack2x16snorm` (81% pass)
- `unpack2x16unorm` (81% pass)
- `unpack4x8snorm` (81% pass)
- `unpack4x8unorm` (81% pass)
- `unpack4xI8` (75% pass)
- `unpack4xU8` (75% pass)
