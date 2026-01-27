# Shader Validation - Continue Statement - Triage Report

CTS selector: `webgpu:shader,validation,statement,continue:*`

Model: Claude Sonnet 4.5

**Overall Status:** 19P/2F/0S (90.48%/9.52%/0%)

## Summary

The continue statement validation tests check that `continue` statements are only used in valid contexts and follow WGSL scoping rules. Most tests pass (19/21), but there are 2 failures related to a specific WGSL validation rule: detecting when a `continue` statement would bypass a declaration that is used in the `continuing` block.

## Passing Tests ✅

The following categories of tests all pass:
- Basic continue placement validation (outside loops, in continuing blocks, etc.)
- Continue in loop, while, and for statements
- Continue in nested contexts (if inside loop, switch inside loop)
- Invalid continue in non-loop contexts (module scope, if, switch, return expression)
- Continue after declarations used in continuing block
- Continue before declarations not used in continuing block
- Invalid continue with expression (continue is a statement, not expression)
- Invalid continue in for-loop initializer, condition, or increment

## Remaining Issues ⚠️

**2 failures (9.52%)** - Both related to continue bypassing declarations used in continuing blocks

## Issue Detail

### 1. Continue Bypassing Declaration Used in Continuing Block

**Test selector:** `webgpu:shader,validation,statement,continue:placement:*`

**What it tests:** Validates the WGSL rule that a `continue` statement must not be placed such that it would transfer control past a declaration used in the targeted `continuing` statement.

**Failing tests:**

1. `webgpu:shader,validation,statement,continue:placement:stmt="loop_continue_before_decl_used_in_continuing"`
2. `webgpu:shader,validation,statement,continue:placement:stmt="loop_nested_continue_before_decl_used_in_continuing"`

**Example failure:**

```
webgpu:shader,validation,statement,continue:placement:stmt="loop_continue_before_decl_used_in_continuing"
```

**Error:**

```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----

@vertex
fn vtx() -> @builtin(position) vec4f {
  loop { continue; let cond = false; continuing { break if cond; } }
  return vec4f(1);
}
```

**Root cause:**

Naga does not implement the WGSL validation rule from the spec (https://gpuweb.github.io/gpuweb/wgsl/#continue-statement):

> A `continue` statement [=shader-creation error|must not=] be placed such that it would transfer control past a declaration used in the targeted [=statement/continuing=] statement.

The spec provides this example of invalid code:

```wgsl
var i: i32 = 0;
loop {
  if i >= 4 { break; }
  if i % 2 == 0 { continue; }  // Invalid!

  let step: i32 = 2;

  continuing {
    i = i + step;  // 'step' is used here
  }
}
```

The `continue` is invalid because it bypasses the declaration of `step`, which is used in the `continuing` block. This means the `continuing` block would reference an uninitialized variable if the continue is taken.

**Current implementation:**

In `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/lower/mod.rs` (around line 1965-1987), the loop statement is lowered without tracking variable declarations or checking if continue statements bypass declarations used in the continuing block:

```rust
ast::StatementKind::Loop {
    ref body,
    ref continuing,
    break_if,
} => {
    let body = self.block(body, true, ctx)?;
    let mut continuing = self.block(continuing, true, ctx)?;
    // ... (break_if handling)
    ir::Statement::Loop {
        body,
        continuing,
        break_if,
    }
}
```

The `Continue` statement at line 1989 is lowered without any additional validation:

```rust
ast::StatementKind::Continue => ir::Statement::Continue,
```

**Fix needed:**

This validation requires control flow analysis to determine:

1. Which declarations exist in the loop body after potential continue statements
2. Which variables are referenced in the continuing block
3. Whether any continue statement would bypass a declaration used in the continuing block

This is a complex validation that would need to be implemented in the WGSL frontend's lowering phase. The validation would need to:

1. Track all `continue` statements in the loop body and their positions
2. Track all variable declarations after each continue statement until the end of the loop body
3. Analyze the continuing block to find all variable references
4. Check if any continue statement would bypass a declaration that is referenced in the continuing block

This is related to the broader "statement behavior" validation issues tracked in issue #7650. The validation requires understanding the control flow through the loop body to determine which declarations would be bypassed.

**Note:** The passing test `loop_continue_before_decl_not_used_in_continuing` shows that it's valid to have a continue before a declaration if that declaration is NOT used in the continuing block. This means the validation must specifically check for usage in the continuing block, not just the presence of declarations after the continue.

The nested variant (`loop_nested_continue_before_decl_used_in_continuing`) shows that this applies even when the continue is inside a nested if statement - the control flow analysis must account for all paths that could execute the continue.

## Related Issues

This validation gap is likely related to:
- Issue #7650: Statement behavior validation issues
- Other statement behavior tests that are failing (statement_behavior, loop, switch)

All of these require sophisticated control flow and data flow analysis to properly validate according to WGSL specification requirements.
