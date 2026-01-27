Now I have enough information to create a comprehensive triage report. Let me create the document.

## Triage Report: `webgpu:shader,validation,parse,comments:*`

Based on my investigation, here's what I found:

### Overall Results
- **Test Suite:** `webgpu:shader,validation,parse,comments:*`
- **Overall Status:** 13 passed / 1 failed / 0 skipped (92.86% pass rate)
- **Model:** Claude Sonnet 4.5

### Test Breakdown

**Passing Tests (13):**
1. `comments` - General comment handling including nesting and various comment styles
2. `line_comment_eof` - Line comments at end of file
3. `line_comment_terminators` (10 subcases) - Tests that line comments are properly terminated by various Unicode whitespace characters
4. `unterminated_block_comment:terminated=true` - Block comment that is properly closed

**Failing Test (1):**
- `unterminated_block_comment:terminated=false` - Block comment that is NOT closed

### Issue Detail

#### Unterminated Block Comments Not Rejected

**Test selector:** `webgpu:shader,validation,parse,comments:unterminated_block_comment:terminated=false`

**What it tests:** Validates that an unterminated block comment (missing closing `*/`) is properly rejected as a parse error.

**Error:**
```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----

/**
 * Unterminated block comment.
 *
```

**Root cause:**
The issue is in Naga's lexer at `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/lexer.rs`, lines 106-154. When parsing a block comment starting with `/*`, the lexer:

1. Iterates through characters tracking the nesting depth
2. Looks for `*/` to close the comment
3. **Bug:** When the loop ends without finding a closing `*/` (line 154), it returns `(Token::End, "")` instead of returning an error

This makes Naga treat an unterminated block comment as if it successfully reached the end of the file, rather than reporting a parse error as required by the WGSL specification.

**Fix needed:**
The lexer should detect when a block comment is not properly terminated and return an error instead of `Token::End`. A new error variant would need to be added to the `Error` enum in `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/error.rs` (something like `UnterminatedBlockComment(Span)`), and the lexer should return this error when the comment loop completes without the depth reaching zero.

This is a **validation gap** - wgpu is accepting invalid WGSL code that should be rejected.
