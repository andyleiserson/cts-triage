Perfect! Now let me create a summary of the triage findings:

## Summary

I've completed a triage of the CTS test selector `webgpu:shader,validation,shader_io,layout_constraints:*` (99% pass rate). Here are the key findings:

### Test Results
- **Total tests:** 101
- **Passing:** 100 (99.01%)
- **Failing:** 1 (0.99%)
- **Skipped:** 0

### Failing Test

**Selector:** `webgpu:shader,validation,shader_io,layout_constraints:layout_constraints:case="struct_size_5_align16"`

**Issue:** Naga incorrectly rejects valid WGSL code when a struct member has an `@align` attribute that increases the containing struct's alignment requirement.

**Test case:**
```wgsl
struct T { @align(16) @size(5) x : u32 }
struct S { x : u32, y : T }
@group(0) @binding(0) var<uniform> v : S;
```

**Root cause:** The Naga layouter (`/Users/Andy/Development/wgpu2/naga/src/proc/layouter.rs`) computes struct alignment by examining member **types** only, not accounting for `@align` attribute overrides. This causes:

1. Struct `T` gets alignment 4 (from `u32`) instead of alignment 16 (from `@align(16)`)
2. When lowering struct `S`, member `y` is incorrectly placed at offset 4 instead of offset 16
3. Validation correctly detects this misalignment and rejects the shader

**Fix needed:** The layouter needs to either:
- Infer struct alignment from actual member offsets in the IR, or
- Store the struct's intended alignment in the IR (e.g., add an `alignment` field to `TypeInner::Struct`)

**Documentation:** Full triage report written to `/Users/Andy/Development/wgpu2/docs/cts-triage/shader_io_layout_constraints_triage.md`
