Excellent! The triage document has been created. Now let me create a concise summary of my findings.

---

## Investigation Summary

I've completed the triage of `webgpu:shader,validation,expression,access,vector:*` tests and documented my findings in:

**Triage Document:** `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_validation_expression_access_vector_triage.md`

### Key Findings

**Overall Status:** 44/84 tests passing (52.38%), with 40 failures

**Breakdown by subcategory:**
- `abstract:*` - 24/24 passing (100%) ✅
- `concrete:*` - 20/60 passing (33.33%)

### Root Causes

All 40 failures occur in tests with `vector_width=2` or `vector_width=3`. Tests with `vector_width=4` all pass. The failures stem from **two distinct issues in Naga**:

#### Issue 1: Overly Strict Runtime Indexing Validation

**Location:** `/Users/Andy/Development/wgpu2/naga/src/valid/expression.rs`, lines 293-318

**Problem:** Naga performs constant value extraction on all `Access` expressions and does bounds checking when the index value is known at compile time. However, it doesn't distinguish between:
- Constant expression contexts (`const i = 2; v[i]`) - should reject out-of-bounds
- Runtime expression contexts (`let i = 2; v[i]`) - should NOT reject, must use clamping

**Impact:** Tests using `let` or `var` variables with out-of-bounds indices incorrectly fail validation, even though WGSL semantics require runtime clamping behavior.

**Example:**
```wgsl
let i = 2;
let r = vec2<i32>()[i];  // Should pass but Naga rejects with bounds error
```

#### Issue 2: Missing Swizzle Validation

**Problem:** Naga doesn't validate that swizzle components are within the bounds of the source vector's width.

**Impact:** Swizzles that access non-existent components are accepted when they should fail.

**Examples:**
```wgsl
let v = vec2<i32>();
let r = v.xyz;   // Should fail (z doesn't exist) but Naga accepts
let s = v.rgb;   // Should fail (b doesn't exist) but Naga accepts
```

### Comparison with Other Access Tests

- **Array access:** 74/76 passing (97.37%) - Much better because test cases don't trigger the same edge cases
- **Matrix access:** 90% pass rate (from summary)
- **Vector access:** 52.38% pass rate - Significantly worse due to both issues

### Recommended Fixes

1. **For runtime indexing:** Modify expression validator to check if index is a `LocalVariable` reference to a `let`/`var` before performing bounds checking
2. **For swizzles:** Add validation during parsing or expression validation to verify all swizzle components exist for the vector's width

Both issues need to be addressed to achieve 100% pass rate on vector access tests.
