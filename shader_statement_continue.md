Excellent! Let me create a summary of the triage.

## Triage Complete: `webgpu:shader,validation,statement,continue:*`

### Summary

I've completed the triage of the continue statement validation tests. Here are the key findings:

**Overall Status:** 19 passing, 2 failing (90.48% pass rate)

### Test Results

**Passing (19/21):**
- Basic continue placement validation (in loops, outside loops)
- Continue in loop, while, and for statements
- Continue in nested contexts (if/switch inside loop)
- Invalid continue in non-loop contexts (module scope, if, switch, return)
- Continue after declarations used in continuing block
- Continue before declarations NOT used in continuing block
- Invalid continue with expression
- Invalid continue in for-loop components

**Failing (2/21):**
1. `webgpu:shader,validation,statement,continue:placement:stmt="loop_continue_before_decl_used_in_continuing"`
2. `webgpu:shader,validation,statement,continue:placement:stmt="loop_nested_continue_before_decl_used_in_continuing"`

### Root Cause

Naga is missing validation for a specific WGSL rule from the spec: **a `continue` statement must not be placed such that it would transfer control past a declaration used in the continuing block**.

Example of invalid code that Naga incorrectly accepts:

```wgsl
loop {
  continue;  // Invalid! Bypasses 'cond' declaration
  let cond = false;
  continuing {
    break if cond;  // 'cond' is used here
  }
}
```

This validation gap requires control flow analysis to track:
1. Continue statement positions in the loop body
2. Variable declarations after each continue
3. Variable usage in the continuing block
4. Whether any continue bypasses a declaration used in continuing

### Documentation Updates

Created/updated the following files:

1. **New triage report:** `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_validation_statement_continue_triage.md`
   - Detailed analysis of all test cases
   - Root cause explanation with code references
   - Links to WGSL spec (https://gpuweb.github.io/gpuweb/wgsl/#continue-statement)

2. **Updated fail.lst:** Added bug reference and description to line 199

3. **Updated triage-checklist.md:** Changed status from ❌ UNTRIAGED to ✅ TRIAGED with issue #7650

4. **Updated triage.md:** Added `statement,continue` to the Statement Behavior (#7650) section

### Related Issue

This is related to **issue #7650** (Statement behavior validation), which covers multiple statement validation gaps including loop, switch, and statement_behavior tests that require sophisticated control flow analysis.
