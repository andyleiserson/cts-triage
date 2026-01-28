# CTS Triage Checklist

This document tracks which selectors in `cts_runner/fail.lst` have been properly triaged.

## Triage Requirements

A selector is considered "triaged" if:
1. It appears in `triage.md` with a dedicated section OR is covered in a comprehensive summary section
2. It has a bug reference or descriptive note in `fail.lst` (percentage alone doesn't count)
3. The root cause is documented

## Status Legend

- ✅ **TRIAGED** - Has bug reference/note in fail.lst AND appears in triage.md
- ❌ **UNTRIAGED** - Missing from fail.lst, triage.md, or both, or manually marked for re-triage

---

# API Validation Tests

## Buffer Mapping
- ✅ Line 14: `webgpu:api,validation,buffer,mapping:*` - crash

## Capability Checks
- ✅ Line 15: `webgpu:api,validation,capability_checks,features,*` - 21%, TypeError not thrown for missing features (needs deno_webgpu fix)
- ✅ Line 16: `webgpu:api,validation,capability_checks,limits,*` - 60%, wgslLanguageFeatures missing (#8884); workgroup storage size validation unimplemented; interstage vars; dx12 bind group timing

## Compute Pipeline
- ✅ Line 17: `webgpu:api,validation,compute_pipeline:limits,workgroup_storage_size:*` - 0%
- ✅ Line 18: `webgpu:api,validation,compute_pipeline:overrides,identifier:*` - 46%
- ✅ Line 19: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*` - 0%
- ✅ Line 20: `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*` - 0%
- ✅ Line 21: `webgpu:api,validation,compute_pipeline:resource_compatibility:*` - 84%
- ✅ Line 22: `webgpu:api,validation,compute_pipeline:storage_texture,format:*` - 85%

## CreateBindGroup
- ✅ Line 23: `webgpu:api,validation,createBindGroup:buffer,resource_state:*` - #7881
- ✅ Line 24: `webgpu:api,validation,createBindGroup:external_texture,*` - 0%, no external texture in deno
- ✅ Line 25: `webgpu:api,validation,createBindGroup:texture,resource_state:*` - crash, #7881

## CreateBindGroupLayout
- ✅ Line 26: `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_pipeline_layout:*` - 0%, using max() instead of +=
- ✅ Line 27: `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_buffer_type:*` - 25%, writable storage buffers not allowed in VERTEX
- ✅ Line 28: `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_storage_texture_access:*` - 25%, write-access storage textures not allowed in VERTEX
- ✅ Line 29: `webgpu:api,validation,createBindGroupLayout:visibility:*` - 25%, missing per-stage storage limits

## CreatePipelineLayout
- ✅ Line 30: `webgpu:api,validation,createPipelineLayout:*` - #4738 and more

## CreateView
- ✅ Line 31: `webgpu:api,validation,createView:texture_state:*` - 0%, #7881

## Encoding Commands
- ✅ Line 32: `webgpu:api,validation,encoding,cmds,copyTextureToTexture:copy_ranges:*` - ***, #8118
- ✅ Line 33: `webgpu:api,validation,encoding,cmds,debug:*` - 92%, #8039
- ✅ Line 34: `webgpu:api,validation,encoding,cmds,render,*` - #7912
- ✅ Line 35: `webgpu:api,validation,encoding,cmds,setImmediates:*` - feature not implemented
- ✅ Lines 36-38: `webgpu:api,validation,encoding,cmds,setBindGroup:*` - #7881

## Encoding Other
- ✅ Line 39: `webgpu:api,validation,encoding,createRenderBundleEncoder:*` - 26%, empty attachments, format compatibility
- ✅ Line 40: `webgpu:api,validation,encoding,encoder_open_state:*` - #7857
- ✅ Lines 41-42: `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:default_bind_group_layouts_never_match,*` - 75%, empty bind group layouts compatibility
- ✅ Lines 43-44: `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:empty_bind_group_layouts_never_requires_empty_bind_groups,*` - 17%, null bind groups, https://github.com/gfx-rs/wgpu/issues/4738
- ✅ Line 45: `webgpu:api,validation,encoding,queries,resolveQuerySet:*` - 93%, #7881
- ✅ Line 46: `webgpu:api,validation,encoding,render_bundle:*` - 81%, readonly flag normalization mismatch

## Error Scope
- ✅ Line 47: `webgpu:api,validation,error_scope:*` - 75%, Metal backend crashes on OOM texture allocation (null pointer dereference)

## GetBindGroupLayout
- ✅ Line 48: `webgpu:api,validation,getBindGroupLayout:*` - 29%, rejects valid indices beyond defined layouts

## Image Copy
- ✅ Line 49: `webgpu:api,validation,image_copy,buffer_texture_copies:*` - #7946
- ✅ Line 50: `webgpu:api,validation,image_copy,layout_related:offset_alignment:*` - fails on DX12

## Layout Shader Compat
- ✅ Line 51: `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*` - 99%, Storage texture access mode validation too strict (read-write should match write)

## Non-Filterable Texture
- ✅ Line 52: `webgpu:api,validation,non_filterable_texture:non_filterable_texture_with_filtering_sampler:*` - 80%, depth textures with filtering samplers

## Query Set
- ✅ Line 53: `webgpu:api,validation,query_set,create:count:*` - 0%, wgpu incorrectly rejects zero-count query sets

## Queue
- ✅ Line 54: `webgpu:api,validation,queue,buffer_mapped:*` - ***, vulkan
- ✅ Line 55: `webgpu:api,validation,queue,destroyed,*` - 71%, writeBuffer/writeTexture return value, destroyed query set
- ✅ Line 56: `webgpu:api,validation,queue,writeBuffer:ranges:*` - 0%, missing OperationError for invalid ranges

## Render Pass
- ✅ Line 57: `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:*` - 94%, TypeError instead of validation error
- ✅ Line 58: `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,loadOp_storeOp_match_depthReadOnly_stencilReadOnly:*` - 33%, missing validation: depth/stencil load/store ops provided for missing texture format aspects
- ✅ Line 59: `webgpu:api,validation,render_pass,render_pass_descriptor:occlusionQuerySet,query_set_type:*` - 50%, occlusionQuerySet type validation

## Render Pipeline Depth/Stencil
- ✅ Line 60: `webgpu:api,validation,render_pipeline,depth_stencil_state:depth_write,frag_depth:*` - 86%, #8856
- ✅ Line 61: `webgpu:api,validation,render_pipeline,depth_stencil_state:depthCompare_optional:*` - #8830

## Render Pipeline Fragment State
- ✅ Line 62: `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,color_target_count:*` - #8856
- ✅ Line 63: `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,use_blend_src:*` - 62%, #8856
- ✅ Line 64: `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets,blend:*` - 95%, blend factors reading source alpha require vec4
- ✅ Line 65: `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets:*` - 3%, color target without shader output requires writeMask=0
- ✅ Line 66: `webgpu:api,validation,render_pipeline,fragment_state:targets_blend:*` - 0%, missing min/max blend factor validation
- ✅ Line 67: `webgpu:api,validation,render_pipeline,fragment_state:targets_write_mask:*` - 50%, TypeError instead of validation error

## Render Pipeline Other
- ✅ Line 68: `webgpu:api,validation,render_pipeline,inter_stage:*` - 15%, inter-stage type validation gaps and interpolation default handling
- ✅ Line 69: `webgpu:api,validation,render_pipeline,misc:external_texture:*` - 0%, no external texture in deno
- ✅ Line 70: `webgpu:api,validation,render_pipeline,misc:storage_texture,format:*` - 8%, similar to compute pipeline issue
- ✅ Line 71: `webgpu:api,validation,render_pipeline,multisample_state:*` - 60%, #8779
- ✅ Line 72: `webgpu:api,validation,render_pipeline,overrides:*` - 17%, missing validation: invalid pipeline constant identifiers silently ignored
- ✅ Line 73: `webgpu:api,validation,render_pipeline,resource_compatibility:*` - 90%
  - Has dedicated triage report
  - 12 failures: read-write storage textures not compatible with write-only shaders in fragment stage
- ✅ Line 74: `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_attribute_limit:*` - 0%
  - Has dedicated triage report
  - Empty vertex buffers not counted in validation, GPUInternalError has empty message
- ✅ Line 75: `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_buffer_limit:*` - 0%
  - Has dedicated triage report
  - Empty vertex buffers not counted toward limit

## Resource Usages
- ✅ Line 76: `webgpu:api,validation,resource_usages,texture,in_pass_encoder:subresources_and_binding_types_combination_for_color:*` - 92%, #3126
- ✅ Line 77: `webgpu:api,validation,resource_usages,texture,in_render_common:subresources,depth_stencil_attachment_and_bind_group:*` - 69%, #8705

## State
- ✅ Line 78: `webgpu:api,validation,state,device_lost,destroy:*` - crash

## Texture
- ✅ Line 79: `webgpu:api,validation,texture,destroy:submit_a_destroyed_texture_as_attachment:*` - 44%, #8714

---

# Shader Validation Tests

## Declarations
- ✅ Line 81: `webgpu:shader,validation,decl,context_dependent_resolution:*` - 92%
  - Has dedicated triage report
  - 1 failure: f16 treated as reserved keyword even when enabled, 4 skipped (wgslLanguageFeatures not exposed)
- ✅ Line 82: `webgpu:shader,validation,decl,override:*` - 93%, #5158
  - Has dedicated triage report
  - 3 failures: unrestricted_pointer_parameters not implemented
- ✅ Line 83: `webgpu:shader,validation,decl,var:*` - 97%
  - Has dedicated triage report
  - 17 failures: template arg trailing comma (9), atomics in read-only storage (4), storage textures in vertex shaders (4)

## Expression Access
- ✅ Line 84: `webgpu:shader,validation,expression,access,array:*` - 97%, runtime indexing with compile-time-known values, #4390
- ✅ Line 85: `webgpu:shader,validation,expression,access,matrix:*` - 93%, runtime OOB matrix access with literal indices incorrectly rejected
- ✅ Line 86: `webgpu:shader,validation,expression,access,vector:*` - 52%, runtime indexing and swizzle

## Expression Binary
- ✅ Line 87: `webgpu:shader,validation,expression,binary,add_sub_mul:*` - 95%, u32 const-eval overflow, f16 overflow not rejected, atomics #5474
- ✅ Line 88: `webgpu:shader,validation,expression,binary,and_or_xor:*` - 96%, #5474
- ✅ Line 89: `webgpu:shader,validation,expression,binary,bitwise_shift:*` - 97%, atomics #5474, partial eval errors
- ✅ Line 90: `webgpu:shader,validation,expression,binary,comparison:*` - 74%, #5474
- ✅ Line 91: `webgpu:shader,validation,expression,binary,div_rem:*` - 86%, #5474
- ✅ Line 92: `webgpu:shader,validation,expression,binary,short_circuiting_and_or:*` - 92%, #8440

## Expression Call Builtin (Lines 93-168)

- `webgpu:shader,validation,expression,binary,*`

### Math Functions
- ✅ Line 93: `abs:*` - 98%, atomic type validation gap, #5474
- ✅ Line 94: `acos:*` - 71%, missing domain validation
- ✅ Line 95: `acosh:*` - 56%, missing domain validation (agent hit rate limit, skipped detailed triage)
- ✅ Line 96: `asin:*` - 71%, missing NaN/Inf validation for AbstractFloat/F16
- ✅ Line 97: `asinh:*` - 85%, asinh(large f32) produces infinity
- ✅ Line 98: `atanh:*` - 72%, domain (-1,1) not validated
- ✅ Line 102: `cosh:*` - 61%, overflow detection missing for abstract types, f16
- ✅ Line 106: `cross:*` - 86%, abstract types const eval overflow
- ✅ Line 107: `degrees:*` - 73%, const eval overflow for abstract/f16
- ✅ Line 108: `derivatives:*` - 83%, f16 support validation
- ✅ Line 109: `determinant:*` - 71%, abstract types const eval
- ✅ Line 110: `distance:*` - 66%, scalar uses wrong formula (sqrt vs abs)
- ✅ Line 111: `dot:*` - 79%, const eval overflow for abstract/f16
- ✅ Line 114: `exp:*` - 61%, const eval overflow for abstract/f16
- ✅ Line 115: `exp2:*` - 80%, f16 overflow detection missing
- ✅ Line 116: `extractBits:*` - 55%, const eval issues
- ✅ Line 117: `faceForward:*` - 68%, const eval overflow
- ✅ Line 118: `firstLeadingBit:*` - 98%, atomic type validation gap, #5474
- ✅ Line 119: `firstTrailingBit:*` - 98%, atomic type validation gap, #5474
- ✅ Line 120: `fma:*` - 85%, const eval issues
- ✅ Line 121: `frexp:*` - 64%, const/override eval not supported
- ✅ Line 122: `insertBits:*` - 73%, const eval not supported
- ✅ Line 123: `inverseSqrt:*` - 61%, const eval overflow for abstract/f16
- ✅ Line 124: `ldexp:*` - 43%, const eval not implemented
- ✅ Line 125: `length:*` - 74%, const eval overflow for abstract/f16
- ✅ Line 126: `log:*` - 64%, domain validation missing
- ✅ Line 127: `log2:*` - 64%, domain validation missing
- ✅ Line 128: `mix:*` - 66%, const eval issues
- ✅ Line 129: `modf:*` - 52%, const eval not fully implemented
- ✅ Line 126: `log:*` - 64%, domain validation (x > 0) missing for abstract/f16
- ✅ Line 127: `log2:*` - 64%, domain validation (x > 0) missing for abstract/f16
- ✅ Line 128: `mix:*` - 66%, const eval issues
- ✅ Line 129: `modf:*` - 52%, const eval not fully implemented
- ✅ Line 130: `normalize:*` - 63%, missing const/override eval overflow validation (intermediate values overflow to infinity)
- ✅ Line 131: `pow:*` - 63%, missing const/override eval validation (negative base, zero^non-positive, overflow), http://github.com/gpuweb/issues/4527
- ✅ Line 132: `quantizeToF16:*` - 77%, overflow validation issues (const eval not implemented, #4507)
- ✅ Line 133: `reflect:*` - 39%, const eval not implemented
- ✅ Line 134: `refract:*` - 44%, const eval not implemented
- ✅ Line 136: `sinh:*` - 61%, const eval overflow for abstract/f16
- ✅ Line 137: `smoothstep:*` - 69%, const eval issues
- ✅ Line 138: `sqrt:*` - 64%, domain validation (x >= 0) missing for abstract/f16
- ✅ Line 160: `transpose:*` - 86%, missing const eval

### Bit/Integer Operations
- ✅ Line 99: `atomics:*` - 86%, atomics in vertex shaders, invalid address spaces/access modes
- ✅ Line 100: `bitcast:*` - 57%, bitcast const-eval unimplemented; size validation missing; f16 vector validation incorrect
- ✅ Line 101: `clamp:*` - 71%, clamp low<=high constraint not checked for const/override parameters
- ✅ Line 103: `countLeadingZeros:*` - 98%, atomic type validation gap, #5474
- ✅ Line 104: `countOneBits:*` - 98%, atomic type validation gap, #5474
- ✅ Line 105: `countTrailingZeros:*` - 98%, atomic type validation gap, #5474
- ✅ Line 116: `extractBits:*` - 55%, const eval issues
- ✅ Line 118: `firstLeadingBit:*` - 98%, atomic type validation gap, #5474
- ✅ Line 119: `firstTrailingBit:*` - 98%, atomic type validation gap, #5474
- ✅ Line 122: `insertBits:*` - 73%, missing const eval support
- ✅ Line 135: `reverseBits:*` - 98%, atomic type validation gap (#5474)
- ✅ Line 145: `select:*` - >99%, #5474

### Geometry/Vector Operations
- ✅ Line 106: `cross:*` - 86%, abstract int/float overflow issues in const eval

### Derivatives
- ✅ Line 108: `derivatives:*` - 83%, f16 support not properly validated

### Dot Products (Packed)
- ✅ Line 112: `dot4I8Packed:*` - 0%, #8884
- ✅ Line 113: `dot4U8Packed:*` - 0%, #8884

### Pack Functions
- ✅ Line 131: `pack2x16float:*` - 80%, missing const eval (#4507)
- ✅ Line 132: `pack2x16snorm:*` - 88%, missing const eval (#4507)
- ✅ Line 133: `pack2x16unorm:*` - 88%, missing const eval (#4507)
- ✅ Line 134: `pack4x8snorm:*` - 88%, missing const eval (#4507)
- ✅ Line 135: `pack4x8unorm:*` - 88%, missing const eval (#4507)
- ✅ Line 136: `pack4xI8:*` - 21%, #8884
- ✅ Line 137: `pack4xI8Clamp:*` - 21%, #8884
- ✅ Line 138: `pack4xU8:*` - 21%, #8884
- ✅ Line 139: `pack4xU8Clamp:*` - 21%, #8884

### Unpack Functions
- ✅ Line 161: `unpack2x16float:*` - 81%, missing const eval (#4507)
- ✅ Line 162: `unpack2x16snorm:*` - 81%, missing const eval (#4507)
- ✅ Line 163: `unpack2x16unorm:*` - 81%, missing const eval (#4507)
- ✅ Line 164: `unpack4x8snorm:*` - 81%, missing const eval (#4507)
- ✅ Line 165: `unpack4x8unorm:*` - 81%, missing const eval (#4507)
- ✅ Line 166: `unpack4xI8:*` - 75%, wgslLanguageFeatures not implemented (#8884)
- ✅ Line 167: `unpack4xU8:*` - 75%, wgslLanguageFeatures not implemented (#8884)

### Texture Functions
- ✅ Line 149: `textureDimensions:*` - >99%, no external texture in deno
- ✅ Line 150: `textureGather:*` - 99%, #8876
- ✅ Line 151: `textureGatherCompare:*` - 99%, edge case in offset or external texture validation
- ✅ Line 152: `textureLoad:*` - 99%, texture_external not implemented
- ✅ Line 153: `textureSample:*` - 99%, texture_external not implemented
- ✅ Line 154: `textureSampleBaseClampToEdge:*` - 96%, texture_external not exposed in deno_webgpu
- ✅ Line 155: `textureSampleBias:*` - 99%, missing offset validation (range & cube texture)
- ✅ Line 156: `textureSampleCompare:*` - 99%, missing offset range validation (-8 to +7)
- ✅ Line 157: `textureSampleCompareLevel:*` - 99%, missing offset range validation (-8 to +7)
- ✅ Line 158: `textureSampleGrad:*` - 99%, missing offset range validation + depth textures incorrectly accepted
- ✅ Line 159: `textureSampleLevel:*` - 99%, missing offset validation (cube texture & value range)

### Constructor
- ✅ Line 168: `value_constructor:*` - 92%, value constructors lack must-use validation + abstract-float matrix validation issues

## Expression Other
- ✅ Line 169: `webgpu:shader,validation,expression,early_evaluation:*` - 67%
  - Has dedicated triage report
  - 6 failures: early evaluation of mixed override/runtime composites causes infinity errors
- ✅ Line 170: `webgpu:shader,validation,expression,matrix,*` - 83%, #5474, #8790, #8868
  - Has dedicated section in triage.md with multiple issues documented
- ✅ Line 171: `webgpu:shader,validation,expression,precedence:*` - 76%, #4536
- ✅ Line 172: `webgpu:shader,validation,expression,unary,*` - 74%, #8884, #5474
  - Has dedicated triage report
  - 76 failures: pointer_composite_access feature not advertised (#8884)
  - 3 failures: atomic direct reference (#5474), matrix negation validation gap

## Extension
- ✅ Line 173: `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*` - 61%, @blend_src validation gaps
- ✅ Line 174: `webgpu:shader,validation,extension,pointer_composite_access:*` - 50%, #8884
  - Has dedicated triage report
  - All 14 failures: wgslLanguageFeatures API not exposed in deno_webgpu
- ✅ Line 175: `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*` - 0%, #8884

## Functions
- ✅ Line 176: `webgpu:shader,validation,functions,alias_analysis:*` - 2%, #7650
- ✅ Line 177: `webgpu:shader,validation,functions,restrictions:*` - 81%, texture_external

## Parse
- ✅ Line 178: `webgpu:shader,validation,parse,attribute:*` - 93%
  - Has dedicated triage report
  - 3 failures: group index validation during shader module creation (should only validate at pipeline creation)
- ✅ Line 179: `webgpu:shader,validation,parse,blankspace:*` - 95%
  - Has dedicated triage report
  - 1 failure: null characters in comments not rejected
- ✅ Line 180: `webgpu:shader,validation,parse,comments:*` - 93%
  - Has dedicated triage report
  - 1 failure: unterminated block comments not rejected
- ✅ Line 181: `webgpu:shader,validation,parse,diagnostic:*` - 50%, #6458
- ✅ Line 183: `webgpu:shader,validation,parse,literal:*` - 96%, #7046
- ✅ Line 184: `webgpu:shader,validation,parse,must_use:*` - 97%, #8876
- ✅ Line 185: `webgpu:shader,validation,parse,requires:*` - 14%, #8884
- ❌ Line 186: `webgpu:shader,validation,parse,shadow_builtins:*` - 60%, #7405

## Shader IO
- ✅ Line 187: `webgpu:shader,validation,shader_io,align:*` - 98%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 1 failure: trailing comma in @align attribute
- ✅ Line 188: `webgpu:shader,validation,shader_io,binding:*` - 95%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 1 failure: trailing comma in @binding attribute
- ✅ Line 189: `webgpu:shader,validation,shader_io,builtins:*` - 50%, trailing comma not accepted
- ✅ Line 190: `webgpu:shader,validation,shader_io,group_and_binding:*` - 87%
  - Has dedicated triage report
  - All 67 failures: texture_external not implemented
- ✅ Line 191: `webgpu:shader,validation,shader_io,group:*` - 95%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 1 failure: trailing comma not accepted in @group attribute
- ✅ Line 192: `webgpu:shader,validation,shader_io,id:*` - 94%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage doc (shader_io_id_triage.md)
- ✅ Line 193: `webgpu:shader,validation,shader_io,interpolate:*` - 91%, https://github.com/gfx-rs/wgpu/issues/8892
  - 2 failures: trailing comma not accepted, @id accepted on const declarations (validation gap)
  - Has dedicated section in triage.md and dedicated triage doc (shader_io_interpolate_triage.md)
- ✅ Line 194: `webgpu:shader,validation,shader_io,layout_constraints:*` - 99%
  - Has dedicated triage report
  - 1 failure: struct alignment not inferred from @align on members (layouter issue)
- ✅ Line 195: `webgpu:shader,validation,shader_io,locations:*` - 98%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage section in triage.md
  - 1 failure: trailing comma not accepted in @location attribute
- ✅ Line 196: `webgpu:shader,validation,shader_io,pipeline_stage:*` - 92%, stage attributes incorrectly accepted on var<private> and var<storage>
  - Has dedicated triage report: docs/cts-triage/shader_io_pipeline_stage.md
  - 6 failures: stage attributes (@vertex, @fragment, @compute) silently ignored on variable declarations
- ✅ Line 197: `webgpu:shader,validation,shader_io,size:*` - 92%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 3 failures: trailing comma (2 tests), large size validation, runtime-sized array validation
- ✅ Line 198: `webgpu:shader,validation,shader_io,workgroup_size:*` - 83%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 10 failures: trailing comma syntax, type mixing validation, attribute placement validation

## Statement
- ✅ Line 199: `webgpu:shader,validation,statement,continue:*` - 90%, #7650
  - Tests continue statement placement and validation
  - Known issue: Missing validation that continue must not bypass declarations used in continuing block
  - Related to statement behavior validation issue #7650
- ✅ Line 200: `webgpu:shader,validation,statement,for:*` - 93%
  - Tests for statement parsing and validation
  - Known issues: phony assignment and increment/decrement in for-loop initializer/continuation, empty loop behavior analysis
  - Dedicated triage report: docs/cts-triage/statement_for_triage.md
- ✅ Line 201: `webgpu:shader,validation,statement,increment_decrement:*` - 98%, #5474
  - Tests increment/decrement statement validation
  - Known issue: atomic type validation gap (4 failures)
  - Dedicated triage report: docs/cts-triage/increment_decrement_triage.md
- ✅ Line 202: `webgpu:shader,validation,statement,loop:*` - 92%, #7650
  - Tests loop statement parsing and validation, including behavior analysis
  - Known issue: infinite loop detection (empty loop, loop with only continue/discard)
  - Related to statement behavior validation issue #7650
- ✅ Line 203: `webgpu:shader,validation,statement,phony:*` - 90%
  - Tests phony assignment statement validation (syntax and type checking)
  - Known issues: for-loop semicolon parsing, atomic/unsized array validation
  - Has dedicated section "Phony Assignment Statement Validation" in triage.md
- ✅ Line 204: `webgpu:shader,validation,statement,statement_behavior:*` - #7650
- ✅ Line 205: `webgpu:shader,validation,statement,switch:*` - #7650
  - Tests switch statement validation including condition types, case type matching, and parsing
  - Related to statement behavior validation issue #7650

## Types
- ✅ Line 206: `webgpu:shader,validation,types,*` - 86%
  - Tests type validation across 10 categories: alias, array, atomics, enumerant, matrix, pointer, ref, struct, textures, vector
  - Main issues: trailing commas in template args (~100 tests), keyword shadowing (10 tests), f16 constructors without enable (9 tests)
  - Has dedicated section "Shader Validation Types" in triage.md with comprehensive analysis

## Uniformity
- ✅ Line 207: `webgpu:shader,validation,uniformity,*` - 21%, #4369
