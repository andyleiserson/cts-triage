# Error Scope CTS Tests - Triage Report

CTS selector: `webgpu:api,validation,error_scope:*`

**Overall Status:** 36P/12F/0S (75%/25%/0%)

## Passing Sub-suites

All tests that do NOT generate OOM errors pass:

1. `empty:` - 1/1 pass
2. `simple:errorType="validation";*` - 3/3 pass
3. `parent_scope:errorFilter="validation";*` - 4/4 pass
4. `current_scope:errorFilter="validation";*` - 5/5 pass
5. `balanced_siblings:*` - 12/12 pass (all filter types)
6. `balanced_nesting:*` - 12/12 pass (all filter types)

## Remaining Issues

All 12 failures are caused by a single bug - a crash in the Metal backend when handling OOM errors:

- `simple:errorType="out-of-memory";*` - 3 tests crash
- `parent_scope:errorFilter="out-of-memory";*` - 4 tests crash
- `current_scope:errorFilter="out-of-memory";*` - 5 tests crash

## Issue Detail

### 1. Metal backend crashes on OOM texture allocation

**Test selector:** All tests with `errorType="out-of-memory"` or `errorFilter="out-of-memory"`

**What it tests:** Tests that error scopes properly capture out-of-memory errors when intentionally exhausting GPU memory.

**Error:**
```
thread 'main' panicked at metal-0.33.0/src/lib.rs:621:5:
null pointer dereference occurred
```

**Root cause:**

The bug is in `/Users/Andy/Development/wgpu2/wgpu-hal/src/metal/device.rs` lines 468-471:

When Metal's `new_texture` fails to allocate a texture (returns null due to OOM), the code at line 470 correctly returns an `OutOfMemory` error. However, the `raw` variable (type `metal::Texture`) still holds the null pointer. When the `autoreleasepool` closure exits, this null texture is dropped, and the `metal` crate's Drop implementation panics on null pointer dereference.

**Fix needed:**

```rust
let raw = self.shared.device.new_texture(&descriptor);
if raw.as_ptr().is_null() {
    std::mem::forget(raw);  // Prevent drop of null pointer
    return Err(crate::DeviceError::OutOfMemory);
}
```

## Summary

The error scope functionality in wgpu works correctly - all tests that don't require generating an OOM error pass (75%). The 12 failing tests all crash due to a bug in the Metal backend's texture creation error handling, not due to actual error scope implementation issues. This is a Metal-specific issue that should be tracked as a separate bug.
