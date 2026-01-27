# For Statement Validation CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,statement,for:*`

Model: Claude Sonnet 4.5

**Overall Status:** 55P/4F/0S (93%/7%/0%)

## Passing Sub-suites ✅

- `webgpu:shader,validation,statement,for:condition_type:*` - All tests pass (bool type validation)
- Most `webgpu:shader,validation,statement,for:parse:*` tests pass (55 out of 59)

## Remaining Issues ⚠️

- `webgpu:shader,validation,statement,for:parse:*` - 4 failures related to:
  1. Phony assignment in initializer (`init_phony`)
  2. Increment operation in initializer (`init_increment`)
  3. Phony assignment in continuation (`cont_phony`)
  4. Empty for loop without break (`empty`)

## Issue Detail

### 1. Phony Assignment and Increment/Decrement Not Allowed in For Loop Initializer

**Test selectors:**
- `webgpu:shader,validation,statement,for:parse:test="init_phony"`
- `webgpu:shader,validation,statement,for:parse:test="init_increment"`

**What it tests:**
These tests verify that phony assignments (`_ = expr`) and increment/decrement operations (`v++`, `v--`) are valid in the initializer part of a for loop.

**Example failures:**

```
webgpu:shader,validation,statement,for:parse:test="init_phony"
```

**Error:**
```
Unexpected validation error occurred:
Shader '' parsing error: for(;;) initializer is not an assignment or a function call: `_ = v;`
  ┌─ wgsl:4:8
  │
4 │   for (_ = v;;) { break; }
  │        ^^^^^^ not an assignment or function call
```

```
webgpu:shader,validation,statement,for:parse:test="init_increment"
```

**Error:**
```
Unexpected validation error occurred:
Shader '' parsing error: for(;;) initializer is not an assignment or a function call: `v++;`
  ┌─ wgsl:4:8
  │
4 │   for (v++;;) { break; }
  │        ^^^^ not an assignment or function call
```

**Root cause:**

Naga's for loop initializer validation in `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs` (lines 2557-2568) only allows three statement kinds:
- `ast::StatementKind::Call`
- `ast::StatementKind::Assign`
- `ast::StatementKind::LocalDecl`

However, the WGSL spec and CTS tests expect these statement kinds to also be valid:
- `ast::StatementKind::Phony` - for phony assignments like `_ = expr`
- `ast::StatementKind::Increment` - for increment operations like `v++`
- `ast::StatementKind::Decrement` - for decrement operations like `v--`

The test shows that compound assignments (`v += 3`) work because they create an `Assign` statement kind. The increment/decrement and phony assignments create different statement kinds that are currently rejected.

**Fix needed:**

Update the for loop initializer validation in `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs` at lines 2558-2566 to also accept `Phony`, `Increment`, and `Decrement` statement kinds:

```rust
match block.stmts.last().unwrap().kind {
    ast::StatementKind::Call { .. }
    | ast::StatementKind::Assign { .. }
    | ast::StatementKind::LocalDecl(_)
    | ast::StatementKind::Phony(_)           // Add this
    | ast::StatementKind::Increment(_)       // Add this
    | ast::StatementKind::Decrement(_) => {} // Add this
    _ => {
        return Err(Box::new(Error::InvalidForInitializer(
            span,
        )))
    }
}
```

### 2. Phony Assignment in For Loop Continuation

**Test selector:**
`webgpu:shader,validation,statement,for:parse:test="cont_phony"`

**What it tests:**
Verifies that phony assignments are valid in the continuation part of a for loop.

**Example failure:**

```
webgpu:shader,validation,statement,for:parse:test="cont_phony"
```

**Error:**
```
Unexpected validation error occurred:
Shader '' parsing error: no definition in scope for identifier: `_`
  ┌─ wgsl:4:10
  │
4 │   for (;;_ = v) { break; }
  │          ^ unknown identifier
```

**Root cause:**

The for loop continuation parsing uses `function_call_or_assignment_statement` (line 2596-2600), which delegates to `assignment_statement`. The `assignment_statement` function calls `lhs_expression`, which tries to resolve `_` as an identifier rather than recognizing it as a phony assignment target.

The issue is different from the initializer case. In the continuation part, the parser doesn't recognize `_` as a special token for phony assignments. The phony assignment handling in the `statement` function (lines 2293-2299) explicitly checks for the `_` token, but this code path isn't used for continuation expressions.

**Fix needed:**

The `function_call_or_assignment_statement` function needs to be updated to handle phony assignments by checking for the `_` token before calling `assignment_statement`. This would require adding special case handling similar to what's done in the main `statement` function.

### 3. Empty For Loop (Behavior Analysis)

**Test selector:**
`webgpu:shader,validation,statement,for:parse:test="empty"`

**What it tests:**
Verifies that an empty infinite loop `for (;;) {}` is rejected due to WGSL behavior analysis rules.

**Example failure:**

```
webgpu:shader,validation,statement,for:parse:test="empty"
```

**Error:**
```
EXPECTATION FAILED: Expected validation error

VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----

fn f() {
  var v = 1;
  for (;;) {}
}

fn x() {}
```

**Root cause:**

The CTS test expects this code to fail validation (note the `pass: false` in the test source at line 64), with a comment explaining it "fails due to behavior-analysis". According to the WGSL specification section on "Statements Behavior Analysis", obviously infinite loops with empty behavior sets must be rejected.

Naga currently does not implement WGSL behavior analysis to detect and reject obviously infinite loops. The spec states (section 8162): "Some obviously infinite loops have an empty behavior set, and are therefore invalid."

The spec gives examples of invalid infinite loops:
```wgsl
fn invalid_infinite_loop() {
  loop { }     // Behavior: { }.  Invalid because it's empty.
}

fn invalid_infinite_loop() {
  loop {
    discard; // Behavior { Next }.
  }          // Invalid, behavior of the whole loop is { }.
}
```

The `for (;;) {}` desugars to a similar infinite loop structure and should be rejected.

**Fix needed:**

This requires implementing WGSL behavior analysis in Naga's validator. This is a significant feature that would need to:
1. Track statement behaviors (Next, Return, Break, Continue, etc.)
2. Compute combined behaviors for compound statements
3. Detect loops with empty behavior sets
4. Reject shaders containing obviously infinite loops

This is a larger architectural change beyond the scope of a simple parser fix. It may be worth filing a separate issue for tracking WGSL behavior analysis implementation.
