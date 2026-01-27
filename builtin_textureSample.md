Perfect! So `kTestTextureTypes` includes `texture_external` at line 133. This means the `texture_type` test in textureSample will test against texture_external and fail because texture_external is not implemented in wgpu/Naga.

Now I have enough information to provide a comprehensive triage. Let me compile the findings:

## CTS Triage Report: webgpu:shader,validation,expression,call,builtin,textureSample:*

Based on my analysis of the test specification and known issues in the wgpu codebase, here are my findings:

### **Test Coverage**

The `textureSample` validation tests cover:
1. **return_type** - Validates return type is correct (vec4<f32> for color textures, f32 for depth textures)
2. **coords_argument** - Validates coordinate argument type matches texture dimension
3. **array_index_argument** - Validates array index for array textures (must be i32 or u32 convertible)
4. **offset_argument** - Validates offset argument type and value range (-8 to +7)
5. **offset_argument,non_const** - Validates offset must be const-expression
6. **only_in_fragment** - Validates textureSample only works in fragment shaders
7. **texture_type** - Validates textureSample works with compatible texture types
8. **must_use** - Validates the @must_use attribute is enforced

### **Root Cause Analysis**

At 99% pass rate, there are very few failures. Based on the test structure and known wgpu issues:

**Primary failure point: texture_external in texture_type test**

The `texture_type` test (lines 336-378 in the spec) uses `kTestTextureTypes` which includes `texture_external` (line 133 of shader_builtin_utils.ts). This test validates that textureSample rejects incompatible texture types.

When the test tries to compile shaders with `texture_external`:
```wgsl
@group(0) @binding(1) var t: texture_external;
```

The shader fails to compile because `texture_external` is not implemented in Naga, causing test failures before validation can even run.

**Expected failures:**
- Tests where `testTextureType='texture_external'` in the `texture_type` test
- This would be a small subset of the texture_type combinations, explaining the ~1% failure rate

### **Suggested fail.lst Entry**

```
webgpu:shader,validation,expression,call,builtin,textureSample:* // 99%, texture_external not implemented
```

### **Summary for triage.md**

```markdown
### textureSample Validation

**Test selector:** `webgpu:shader,validation,expression,call,builtin,textureSample:*`

**Pass rate:** 99%

**What it tests:** Validation of the `textureSample()` builtin function including:
- Return type correctness
- Coordinate argument type matching
- Array index argument validation (i32/u32 convertible)
- Offset argument type, range (-8 to +7), and const-expression requirement
- Fragment shader stage restriction
- Texture type compatibility
- @must_use attribute enforcement

**Root cause:** The `texture_type` test uses `kTestTextureTypes` which includes `texture_external`. Shaders with `texture_external` fail to compile because this texture type is not implemented in Naga, causing a small number of test failures (~1%).

**Related tests:** Other textureSample* variants (textureSampleBias, textureSampleCompare, textureSampleGrad, textureSampleLevel, textureSampleBaseClampToEdge) likely have similar failures with texture_external.

**Fix needed:** None required for textureSample itself. These failures will resolve when texture_external is implemented in Naga (tracked elsewhere).
```
