# Shader Functions Restrictions CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,functions,restrictions:*`

Model: Claude Sonnet 4.5

**Overall Status:** Estimated 81% pass rate (based on Shader Validation Summary)

## Test Suite Overview

The `restrictions.spec.ts` file contains 26 test cases that validate various function-related restrictions in WGSL:

1. **vertex_returns_position** - Vertex shaders must return a position builtin
2. **entry_point_call_target** - Entry points cannot be called as regular functions
3. **function_return_types** - Function return types must be constructible
4. **function_parameter_types** - Function parameter type validation
5. **function_parameter_matching** - Argument types must match parameter types
6. **no_direct_recursion** - Direct recursion is forbidden
7. **no_indirect_recursion** - Indirect recursion is forbidden
8. **param_names_must_differ** - Parameter names must be unique
9. **param_scope_is_function_body** - Parameters only in scope in function body
10. **param_number_matches_call** - Argument count must match parameter count
11. **call_arg_types_match_1_param** - Type matching for 1-parameter calls
12. **call_arg_types_match_2_params** - Type matching for 2-parameter calls
13. **call_arg_types_match_3_params** - Type matching for 3-parameter calls
14. **param_name_can_shadow_function_name** - Parameter shadowing allowed
15. **param_name_can_shadow_alias** - Parameter can shadow alias
16. **param_name_can_shadow_global** - Parameter can shadow global
17. **param_comma_placement** - Comma placement validation
18. **param_type_can_be_alias** - Alias as parameter type
19. **function_name_required** - Function must have a name
20. **param_type_required** - Parameter must have a type
21. **body_required** - Function must have a body
22. **parens_required** - Parentheses are required
23. **non_module_scoped_function** - Functions must be module-scoped
24. **function_attributes** - Attribute validation (@diagnostic, @must_use, etc.)
25. **must_use_requires_return** - @must_use requires return value
26. **overload** - Overloading is not allowed

## Remaining Issues

### Primary Blocker: texture_external Not Implemented

**Impact:** Estimated 58+ test failures

Several tests in this suite declare `texture_external` variables in their test shaders:
- `function_return_types` - Uses `texture_external` in test code
- `function_parameter_matching` - Declares `t_external` variable

Since wgpu/Naga does not implement `texture_external` support, any shader that attempts to use this type will fail to compile, causing the test to fail regardless of whether the actual validation being tested is correct.

**Tests affected:**
- Any test case in `function_return_types` or `function_parameter_matching` that includes a shader with `texture_external` declarations
- The `function_parameter_matching` test combines all valid parameter types with all parameter value cases, creating a large matrix of test combinations

### Secondary Issues

#### 1. Attribute Validation

**Test:** `function_attributes`

**What it tests:** Validation of various attributes on functions, parameters, and return types

**Potential issues:**
- Tests validation of `@id`, `@workgroup_size`, and other attributes in contexts where they should not be allowed
- May have minor failures related to error reporting for invalid attribute placement

**Impact:** Estimated 2 failures (per Shader Validation Summary)

#### 2. Unrestricted Pointer Parameters

**Tests affected:** `function_parameter_types`, `function_parameter_matching`

**What it tests:** Validation of pointer parameters with various address spaces

**Potential issues:**
- Tests include cases that are only valid with the `unrestricted_pointer_parameters` language feature
- The tests check for `t.hasLanguageFeature('unrestricted_pointer_parameters')` to conditionally validate
- This relates to issue #8884 where `wgslLanguageFeatures` is not exposed

**Impact:** Unknown - depends on whether the feature detection works correctly in the test environment

## Analysis

### Likely Passing Tests

Based on the test structure and wgpu's known capabilities, these tests should pass:

- **no_direct_recursion** - Naga validates recursion
- **no_indirect_recursion** - Naga validates recursion
- **param_names_must_differ** - Basic name uniqueness
- **param_number_matches_call** - Basic arity checking
- **call_arg_types_match_1_param** - Type matching (for supported types)
- **call_arg_types_match_2_params** - Type matching (for supported types)
- **call_arg_types_match_3_params** - Type matching (for supported types)
- **param_name_can_shadow_function_name** - Scoping rules
- **param_name_can_shadow_alias** - Scoping rules
- **param_name_can_shadow_global** - Scoping rules
- **param_comma_placement** - Parser validation
- **param_type_can_be_alias** - Alias support
- **function_name_required** - Parser validation
- **param_type_required** - Parser validation
- **body_required** - Parser validation
- **parens_required** - Parser validation
- **non_module_scoped_function** - Scope validation
- **overload** - No overloading support
- **param_scope_is_function_body** - Scope validation
- **vertex_returns_position** - Position validation
- **entry_point_call_target** - Entry point call restriction
- **must_use_requires_return** - @must_use validation (mostly)

### Likely Failing Tests

- **function_return_types** - Includes texture_external in test cases
- **function_parameter_types** - May include texture_external or have minor issues
- **function_parameter_matching** - Large test matrix including texture_external cases
- **function_attributes** - Minor failures for @id and @workgroup_size validation

### Test Characteristics

#### function_parameter_matching Complexity

This test is particularly complex because it:
1. Combines ALL valid parameter type declarations from `kFunctionParamTypeCases` (60+ types)
2. With ALL parameter value cases from `kFunctionParamValueCases` (90+ values)
3. Creates a cartesian product of test cases

This means even if only a few parameter value cases reference `texture_external`, they get combined with every valid parameter type, multiplying the failure count.

For example, if `texture_external` appears in 1 parameter value case, and there are 60 valid parameter types, that creates 60 test failures just from that one value case.

## Root Cause Summary

The 81% pass rate (approximately 19% failure rate) for function restrictions tests is primarily caused by:

1. **texture_external not implemented** (~15-17% of failures) - Causes any shader using this type to fail compilation
2. **Minor attribute validation gaps** (~2% of failures) - Small issues with @id and @workgroup_size attribute validation
3. **Possible unrestricted pointer parameter feature detection issues** (unknown impact)

## Recommendations

1. **Do not attempt to fix texture_external issues** - This is a known missing feature that requires substantial implementation work beyond shader validation
2. **Investigate attribute validation failures** - These are likely small fixes that could improve the pass rate
3. **Run tests to confirm analysis** - This triage is based on code analysis; actual test runs would confirm the specific failure patterns
4. **Consider adding texture_external to fail.lst** - If not already there, document these as known failures

## Notes

This triage was conducted through code analysis of the test file due to git lock issues preventing direct test execution. The failure patterns and pass rate estimates are based on:
- Analysis of test structure in `restrictions.spec.ts`
- Cross-reference with the Shader Validation Summary in `triage.md`
- Understanding of wgpu's known limitations with texture_external
- Examination of test case parameter combinations
