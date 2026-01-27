# Shader Validation - Parse Blankspace - Triage Report

CTS selector: `webgpu:shader,validation,parse,blankspace:*`

Model: Claude Sonnet 4.5

**Overall Status:** 18P/1F/0S (94.74%/5.26%/0%)

## Summary

The blankspace parse validation tests verify that WGSL handles whitespace characters, null characters, and byte order marks (BOMs) according to the specification. Almost all tests pass (18/19), with only 1 failure related to null character validation inside comments.

## Passing Tests ✅

The following categories of tests all pass:

- **Blankspace characters as delimiters** (11 tests): All Unicode Pattern_White_Space characters work correctly:
  - Space (U+0020)
  - Horizontal tab (U+0009)
  - Line feed (U+000A)
  - Vertical tab (U+000B)
  - Form feed (U+000C)
  - Carriage return (U+000D)
  - Next line (U+0085)
  - Left-to-right mark (U+200E)
  - Right-to-left mark (U+200F)
  - Line separator (U+2028)
  - Paragraph separator (U+2029)

- **Byte Order Mark (BOM) validation** (2 tests): Correctly rejects BOMs and accepts code without BOMs

- **Null character validation in non-comment contexts** (4 tests):
  - Null in delimiter position: Correctly rejected
  - Null at end of line: Correctly rejected
  - Valid code without null: Correctly accepted in all positions

## Remaining Issues ⚠️

**1 failure (5.26%)** - Null character inside comment not being rejected

## Issue Detail

### 1. Null Character in Comment Not Rejected

**Test selector:** `webgpu:shader,validation,parse,blankspace:null_characters:contains_null=true;placement="comment"`

**What it tests:** Validates that WGSL source containing a null character is rejected, even when the null character appears inside a comment.

**Example failure:**

```
webgpu:shader,validation,parse,blankspace:null_characters:contains_null=true;placement="comment"
```

**Error:**

```
EXPECTATION FAILED: Expected validation error
VALIDATION FAILED: Missing expected compilationInfo 'error' message.

---- shader ----
// Here is a   character
```

(The null character is present in the comment but not visible in the error message)

**Root cause:**

Naga's WGSL lexer does not validate that null characters are absent from the source code when they appear inside comments. The lexer in `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/lexer.rs` handles comments as follows:

1. Lines 87-104: When encountering `//`, the lexer finds the comment end using `is_comment_end()` and returns the entire comment as `Token::Trivia`
2. Lines 215-219: The `is_comment_end()` function only checks for line break characters, not null characters
3. Lines 398-405: The `next_impl()` function skips all `Token::Trivia` tokens without validating their contents

This means that null characters inside comments are consumed as part of the comment text and never validated. In contrast:

- **Null in delimiter position** (`const\0name : i32 = 0;`): The null appears between tokens and is processed as `Token::Unknown('\0')`, which triggers an error when the parser encounters it
- **Null at end of line** (`const name : i32 = 0;\0`): Same behavior - processed as an unknown token and rejected
- **Null in comment** (`// Here is a \0 character`): Consumed as part of the comment trivia and never validated

**WGSL Specification Requirement:**

According to the WGSL spec, null characters (U+0000) must be rejected anywhere in the source code, including inside comments. The spec does not make an exception for comments.

**Fix needed:**

The fix requires adding validation to detect null characters during comment processing. Possible approaches:

1. **Validate during comment consumption**: In the `consume_token()` function (lines 87-104 for line comments, 106-155 for block comments), check for null characters while consuming the comment text and return an error if found.

2. **Add null character check to lexer**: Create a validation pass that scans for null characters before or during lexing, rejecting any source that contains them.

3. **Pre-validate source**: Check the entire source string for null characters before starting the lexing process.

The most appropriate fix would be option 1, validating during comment consumption, as it would catch null characters at the same time they're being processed and provide accurate error location information.

**File locations:**

- Lexer: `/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/lexer.rs`
- Comment handling: Lines 87-155 in lexer.rs
- Comment end detection: Lines 215-219 in lexer.rs

**GitHub Issue:**

This issue is tracked in the fail.lst as: https://github.com/gfx-rs/wgpu/issues/8877

## Test Categories

### null_characters (6 tests)
- 5 passing, 1 failing
- Tests that null characters are rejected in various positions
- Only failure is null character inside comments

### blankspace (11 tests)
- All passing
- Tests that all valid Unicode blankspace characters work as delimiters

### bom (2 tests)
- All passing
- Tests that BOM (U+FEFF) is correctly rejected

## Recommendations

This is a low-priority validation gap since:
1. Null characters in source code are extremely rare in practice
2. The validation gap is limited to comments only - null characters in actual code are correctly rejected
3. The impact is minimal - accepting null characters in comments doesn't affect program behavior

However, fixing this would improve spec compliance and is relatively straightforward to implement during comment tokenization.
