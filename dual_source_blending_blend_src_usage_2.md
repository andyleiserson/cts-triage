Perfect! Now I can see we have 23 unique tests: 14 passing and 9 failing. Let me create a detailed analysis comparing what should pass vs what actually passes.

## Investigation Summary

### Discrepancy Found

**fail.lst claim:** 61% pass rate  
**triage.md claim (lines 594-614):** "blend_src_usage: 23P/0F/0S (100% pass)"  
**Actual current results:** 14P/9F/0S (60.87% pass)

The **fail.lst is accurate** - the pass rate is indeed ~61%. The **triage.md is outdated and incorrect**.

### Detailed Analysis

Out of 23 tests total:

#### Tests That Should PASS (3 tests expected):
All 3 are currently **PASSING** ✅:
1. `struct_member_no_location_no_blend_src` - ✅ PASS
2. `struct_member_blend_src_and_builtin` - ✅ PASS  
3. `struct_member_location_0_blend_src_0_blend_src_1` - ✅ PASS

#### Tests That Should FAIL (20 tests expected to reject invalid shaders):

**Currently PASSING but SHOULD FAIL** (11 tests - validation gaps) ❌:
1. `const` - ✅ PASS (should reject `@blend_src` on const)
2. `override` - ✅ PASS (should reject `@blend_src` on override)
3. `let` - ✅ PASS (should reject `@blend_src` on let)
4. `var_private` - ✅ PASS (should reject `@blend_src` on private var)
5. `var_function` - ✅ PASS (should reject `@blend_src` on function var)
6. `function_declaration` - ✅ PASS (should reject `@blend_src` on function)
7. `non_entrypoint_function_input_non_struct` - ✅ PASS (should reject)
8. `non_entrypoint_function_output_non_struct` - ✅ PASS (should reject)
9. `entrypoint_input_non_struct` - ✅ PASS (should reject `@blend_src` on entry point input)
10. `struct_member_no_location_blend_src_0` - ✅ PASS (should reject - missing location on one member)
11. `struct_member_no_location_blend_src_1` - ✅ PASS (should reject - missing location on one member)

**Currently FAILING as expected** (9 tests) ✅:
1. `entrypoint_output_non_struct` - ❌ FAIL (correctly rejects)
2. `struct_member_only_blend_src_0` - ❌ FAIL (correctly rejects - only one blend_src)
3. `struct_member_only_blend_src_1` - ❌ FAIL (correctly rejects - only one blend_src)
4. `struct_member_duplicate_blend_src_0` - ❌ FAIL (correctly rejects duplicate)
5. `struct_member_duplicate_blend_src_1` - ❌ FAIL (correctly rejects duplicate)
6. `struct_member_has_non_zero_location_blend_src_0` - ❌ FAIL (correctly rejects non-zero location)
7. `struct_member_has_non_zero_location_blend_src_1` - ❌ FAIL (correctly rejects non-zero location)
8. `struct_member_non_zero_location_blend_src_0_blend_src_1` - ❌ FAIL (correctly rejects non-zero location)
9. `struct_member_has_non_zero_location_no_blend_src` - ❌ FAIL (correctly rejects mixed locations)

### Root Cause

The issue is that **Naga (wgpu's shader validator) is missing validation for many invalid uses of `@blend_src`**. The attribute is being accepted in contexts where it should be rejected according to the WebGPU spec.

According to the test description, `@blend_src`:
- Can ONLY be used on a member of a structure
- MUST be used together with the `@location` attribute
- If used, there must be EXACTLY 2 members with location attributes: one `@location(0) @blend_src(0)` and another `@location(0) @blend_src(1)`

Currently, Naga is not validating these restrictions properly, resulting in 11 validation gaps where invalid shaders are incorrectly accepted.

### Recommendation

The **triage.md should be updated** to reflect the current state:
- Change from "23P/0F/0S (100% pass)" to "14P/9F/0S (61% pass)"
- Document the 11 validation gaps that need to be fixed in Naga
- The fail.lst entry at line 173 is correct and should remain as-is
