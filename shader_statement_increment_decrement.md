Perfect! The final results are confirmed: 218/222 passing (98.2%), with only the 4 atomic-related failures remaining, which is a known issue tracked in #5474.

Let me create a summary of the work completed:

## Triage Summary

I have successfully triaged the CTS selector `webgpu:shader,validation,statement,increment_decrement:*` (97%) from line 201 of triage-checklist.md.

### Results

**Overall Status:** 218/222 passing (98.2%)

### Test Breakdown
- **`component:*`** - 60/60 passing (100%)
  - Tests increment/decrement on component accesses
  - All passing
  
- **`parse:*`** - 128/128 passing (100%)
  - Tests parsing of increment/decrement statements
  - All passing after fixing for-loop initializer support
  
- **`var_init_type:*`** - 30/34 passing (88.2%)
  - Tests increment/decrement on different variable types
  - 4 failures related to atomic types (known issue #5474)

### Issues Found

1. **Atomic Type Validation Gap (4 failures)** - Known issue [#5474](https://github.com/gfx-rs/wgpu/issues/5474)
   - wgpu incorrectly accepts increment/decrement on atomic types
   - Should reject because atomics are not concrete integer scalars
   - Part of broader atomic validation issue

2. **For-Loop Initializer Parsing (FIXED ✅)**
   - Naga was rejecting increment/decrement in for-loop initializers
   - Fixed by adding `StatementKind::Increment` and `StatementKind::Decrement` to allowed statement types
   - Updated error message for clarity

### Files Modified

1. **`/Users/Andy/Development/wgpu2/naga/src/front/wgsl/parse/mod.rs`**
   - Added support for increment/decrement in for-loop initializers

2. **`/Users/Andy/Development/wgpu2/naga/src/front/wgsl/error.rs`**
   - Updated error message to accurately describe allowed for-loop initializer statements

3. **`/Users/Andy/Development/wgpu2/cts_runner/fail.lst`**
   - Updated pass rate from 97% to 98%
   - Added reference to issue #5474

4. **`/Users/Andy/Development/wgpu2/docs/cts-triage/increment_decrement_triage.md`** (created)
   - Comprehensive triage report documenting all findings

### Documentation

Created a detailed triage report at `/Users/Andy/Development/wgpu2/docs/cts-triage/increment_decrement_triage.md` that includes:
- Overall test statistics
- Breakdown by subcategory
- Detailed analysis of both issues
- Root cause analysis with WGSL spec references
- Fix implementation details

The triage is complete. The selector now has 98.2% pass rate with only 4 failures remaining, all related to the known atomic validation gap tracked in issue #5474.
