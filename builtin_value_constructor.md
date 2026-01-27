So the failing subcases are when the dimensions don't match (c1≠c2 or r1≠r2). Looking at the test expectation (line 375-380 of the test source), these should fail when dimensions don't match. But wgpu is accepting them.

This is a validation gap in Naga - it's not properly checking that matrix constructors have matching dimensions when using abstract-float vectors.

Now let me create a summary document for these findings.

## Summary

Based on my triage of `webgpu:shader,validation,expression,call,builtin,value_constructor:*`, I've identified the following:

**Overall Status:** 476P/7F (98.55% pass rate)

### Root Causes

There are **two distinct issues** causing the 7 failures:

#### 1. **Value constructors lack must-use validation** (5 failures)
   
**Affected tests:**
- `must_use:ctor="S_0";use=false` - struct zero-value constructor without use
- `must_use:ctor="S";use=false` - struct value constructor without use
- `must_use:ctor="mat4x2";use=true` - matrix constructor with abstract vectors (false positive)
- `must_use:ctor="mat4x3";use=true` - matrix constructor with abstract vectors (false positive)
- `must_use:ctor="mat4x4";use=true` - matrix constructor with abstract vectors (false positive)

**Root cause:** In `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/lower/mod.rs`, the `call()` function (lines 3565-3573) handles type constructors but does not enforce must-use semantics when they are used as statements. When a type is used as a constructor, it returns `Ok(Some(handle))` without checking if `is_statement` is true. The must-use check only happens for actual function calls (lines 3615-3616), not for type constructors.

According to WGSL spec, value constructors (like struct constructors, matrix constructors, etc.) must have their results used - they cannot be standalone statements. The tests expect these to fail when `use=false`, but Naga is accepting them.

Note: The mat4x2, mat4x3, mat4x4 failures with `use=true` are actually a separate issue - Naga is incorrectly rejecting valid abstract vector types in matrix constructors (see issue #2 below), which causes an unexpected validation error before the must-use check can occur.

#### 2. **Abstract-float matrix constructor validation is incorrect** (2 failures, plus 3 false positives in must_use)

**Affected tests:**
- `matrix_column:type1="abstract-float";type2="abstract-float"` - Missing dimension validation
- `matrix_elementwise:type1="abstract-float";type2="abstract-float"` - Missing dimension validation

**Root cause:** When constructing matrices with abstract-float vectors or scalars, Naga is:
1. Not properly validating that dimensions match (validation gap - accepts invalid code)
2. Not properly resolving abstract vector types in matrix constructor context (rejects valid code with "Matrix elements must always be floating-point types")

The second issue manifests in the must_use tests for mat4x2, mat4x3, mat4x4, where abstract vectors like `vec2()`, `vec3()`, `vec4()` are used. Naga fails to infer that these abstract vectors should be floating-point types in the matrix constructor context, leading to the error "Matrix elements must always be floating-point types" from `/Users/Andy/Development/wgpu2/naga/src/valid/type.rs:407`.

### Suggested bug references for fail.lst

```
webgpu:shader,validation,expression,call,builtin,value_constructor:must_use:* // Value constructors lack must-use validation; abstract vectors in matrix constructors not properly resolved
webgpu:shader,validation,expression,call,builtin,value_constructor:matrix_column:type1="abstract-float";type2="abstract-float" // Abstract-float matrix constructor dimension validation missing
webgpu:shader,validation,expression,call,builtin,value_constructor:matrix_elementwise:type1="abstract-float";type2="abstract-float" // Abstract-float matrix constructor dimension validation missing
```

### Summary for triage.md

```markdown
## webgpu:shader,validation,expression,call,builtin,value_constructor

**Status:** 476P/7F (98.55%)

**Failures:**
- `must_use:*` (5/78 failing) - Value constructors lack must-use validation. When type constructors (struct, matrix, etc.) are used as standalone statements without their result being used, Naga should reject them but currently accepts them. Additionally, abstract vectors in matrix constructors are not properly type-resolved, causing valid code to be rejected.
- `matrix_column:type1="abstract-float";type2="abstract-float"` (1/18 failing) - Naga accepts matrix constructors with mismatched dimensions when using abstract-float types (validation gap).
- `matrix_elementwise:type1="abstract-float";type2="abstract-float"` (1/18 failing) - Naga accepts matrix constructors with wrong element count when using abstract-float types (validation gap).

**Required fixes:**
1. Add must-use validation for type constructors in statement context (mod.rs:3565-3573)
2. Fix abstract vector type resolution in matrix constructor context
3. Add dimension validation for abstract-float matrix constructors
```
