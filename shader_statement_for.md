Good! The tests are in fail.lst but not in test.lst. Now let me provide a summary of my findings.

## Triage Summary

I've completed the triage of the CTS selector `webgpu:shader,validation,statement,for:*` (93% pass rate, line 200 in fail.lst). Here's what I found:

### Overall Status
- **Pass rate:** 55/59 tests passing (93.22%)
- **Failures:** 4 tests failing (6.78%)
- **Location:** Test results documented in `/Users/Andy/Development/wgpu2/docs/cts-triage/statement_for_triage.md`

### Failing Tests

#### 1. **Phony Assignment in For Loop Initializer** 
- **Test:** `webgpu:shader,validation,statement,for:parse:test="init_phony"`
- **Issue:** `for (_ = v;;) { break; }` is rejected but should be valid
- **Root cause:** Naga's for loop initializer validation only allows `Call`, `Assign`, and `LocalDecl` statement kinds, but needs to also allow `Phony`
- **Fix:** Add `ast::StatementKind::Phony(_)` to the allowed statement kinds at `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs:2558-2566`

#### 2. **Increment Operation in For Loop Initializer**
- **Test:** `webgpu:shader,validation,statement,for:parse:test="init_increment"`
- **Issue:** `for (v++;;) { break; }` is rejected but should be valid
- **Root cause:** Same as above - needs to allow `Increment` and `Decrement` statement kinds
- **Fix:** Add `ast::StatementKind::Increment(_)` and `ast::StatementKind::Decrement(_)` to the allowed statement kinds

#### 3. **Phony Assignment in For Loop Continuation**
- **Test:** `webgpu:shader,validation,statement,for:parse:test="cont_phony"`
- **Issue:** `for (;;_ = v) { break; }` fails with "unknown identifier: `_`"
- **Root cause:** The continuation parsing doesn't recognize `_` as a phony assignment token; it tries to resolve it as a regular identifier
- **Fix:** Update `function_call_or_assignment_statement` to check for `_` token and handle phony assignments before calling `assignment_statement`

#### 4. **Empty For Loop (Behavior Analysis)**
- **Test:** `webgpu:shader,validation,statement,for:parse:test="empty"`
- **Issue:** `for (;;) {}` should be rejected as an obviously infinite loop but is accepted
- **Root cause:** Naga doesn't implement WGSL behavior analysis to detect and reject infinite loops with empty behavior sets
- **Fix:** Requires implementing full WGSL behavior analysis - a significant architectural change beyond a simple parser fix

### Priority Assessment

**High Priority (Easy fixes):**
- Issues #1 and #2: Simple additions to the pattern match in the parser (one-line changes)

**Medium Priority (Moderate complexity):**
- Issue #3: Requires refactoring the continuation parsing to handle phony assignments

**Low Priority (Major feature):**
- Issue #4: Requires implementing WGSL behavior analysis - should be tracked as a separate feature request

### Recommendations

1. **Quick win:** Fix issues #1 and #2 by updating the statement kind validation in the for loop initializer
2. **Follow-up:** Address issue #3 by enhancing phony assignment handling in continuation expressions  
3. **Long-term:** File a separate issue for implementing WGSL behavior analysis to handle issue #4 and other spec compliance gaps

The triage document has been saved to `/Users/Andy/Development/wgpu2/docs/cts-triage/statement_for_triage.md` with detailed analysis of each failure.
