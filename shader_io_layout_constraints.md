# Shader IO Layout Constraints CTS Tests - Triage Report

CTS selector: `webgpu:shader,validation,shader_io,layout_constraints:*`

Model: Claude Sonnet 4.5

**Overall Status:** 100P/1F/0S (99.01%/0.99%/0.00%)

## Passing Tests ✅

All 100 passing tests validate address space layout constraints correctly, including:
- Scalar types (u32, i32, f32, f16, bool)
- Vector types (vec2, vec3, vec4 of various scalar types)
- Matrix types
- Array types with various strides and alignments
- Struct types with size and alignment attributes (except one specific case - see below)
- Atomic types in appropriate address spaces

## Failing Test ⚠️

### 1. Struct with @align attribute affecting struct type alignment

**Test selector:** `webgpu:shader,validation,shader_io,layout_constraints:layout_constraints:case="struct_size_5_align16"`

**What it tests:** Validates that when a struct member has an `@align` attribute, the containing struct's alignment is computed correctly as the maximum of all its members' alignments (including overridden alignments), not just the intrinsic type alignments.

**Test case:**
```wgsl
struct T { @align(16) @size(5) x : u32 }
struct S { x : u32, y : T }

@group(0) @binding(0) var<uniform> v : S;
```

**Expected behavior:** Should be valid in all address spaces (including uniform)

**Actual behavior:** Fails validation for uniform address space

**Error message:**
```
EXCEPTION: Error: Unexpected validation error occurred:
Shader validation error: Global variable [0] 'v' is invalid
  ┌─ :6:23
  │
6 │ @group(0) @binding(0) var<uniform> v : S;
  │                       ^^^^^^^^^^^^^^^^^^^ naga::ir::GlobalVariable [0]
  │
  = Alignment requirements for address space Uniform are not met by [2]
  = The struct member[1] offset 4 is not a multiple of the required alignment 16
```

**Root cause:**

There is a mismatch between how the WGSL frontend, the layouter, and the validator compute struct alignment when `@align` attributes are present.

**When lowering `struct T { @align(16) @size(5) x : u32 }`:**

1. The WGSL frontend (`naga/src/front/wgsl/lower/mod.rs`, lines 4254-4275) correctly:
   - Computes `member_alignment = 16` for member `x` (from `@align(16)`)
   - Updates `struct_alignment = 16` (line 4275)
   - Places member `x` at offset 0
   - Computes `span = 16` (rounded up from offset + size = 0 + 5 = 5)

2. The layouter (`naga/src/proc/layouter.rs`, lines 251-268) computes struct `T`'s layout:
   - Iterates through members and computes alignment as `alignment.max(self[member.ty].alignment)`
   - For member `x` of type `u32`, this gives alignment 4
   - **BUG:** The layouter doesn't account for the `@align` override, so struct `T` gets alignment 4 instead of 16
   - Uses `span = 16` from the IR

**When lowering `struct S { x : u32, y : T }`:**

1. The WGSL frontend places members:
   - Member `x` (u32) at offset 0
   - Member `y` (struct T): queries `ctx.layouter[T].alignment`, which returns 4
   - **BUG:** Places member `y` at offset 4 (next multiple of 4), when it should be at offset 16

**During validation (`naga/src/valid/type.rs`):**

1. When validating struct `T`:
   - Starts with `uniform_layout = Ok(Alignment::MIN_UNIFORM)` = 16 (line 654)
   - For member `x` (u32), `base_info.uniform_layout = Ok(Alignment::FOUR)`
   - Calls `check_member_layout`, which computes `Ok(Alignment::MIN_UNIFORM.max(Alignment::FOUR)) = Ok(Alignment::SIXTEEN)`
   - Struct `T` ends up with `uniform_layout = Ok(Alignment::SIXTEEN)`

2. When validating struct `S`:
   - Member `x` (u32) at offset 0: alignment 4, checks OK
   - Member `y` (struct T) at offset 4: has `uniform_layout = Ok(Alignment::SIXTEEN)`
   - **VALIDATION FAILS:** `check_member_layout` verifies that offset 4 is aligned to 16, which fails

**Summary:**

The core issue is that the layouter computes struct alignment based solely on member **types**, ignoring `@align` attribute overrides. This causes the frontend to place subsequent members at incorrectly aligned offsets, which the validator then correctly rejects.

**Fix needed:**

The layouter needs to infer struct alignment from the actual member offsets in the IR, not just from member types.

One approach: In `layouter.rs` line 251-268, when computing struct alignment, the layouter could:
1. For each member at offset `o` with type having alignment `a`:
2. If `o > 0` and `o` is a multiple of some alignment `a' > a`, infer that the struct needs at least alignment `a'`
3. Alternatively, compute the GCD of all non-zero member offsets to infer the struct's alignment

However, the cleanest fix would be to store the struct's intended alignment in the IR. This could be:
- A new `alignment` field in `TypeInner::Struct`
- Or store per-member alignment in `StructMember`

The fix should be in `naga/src/front/wgsl/lower/mod.rs` to pass the computed `struct_alignment` to the IR, and `naga/src/proc/layouter.rs` to use it when available.
