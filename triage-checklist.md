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
- ✅ `webgpu:api,validation,buffer,mapping:*` - crash

## Capability Checks
- ✅ `webgpu:api,validation,capability_checks,features,*` - 21%, TypeError not thrown for missing features (needs deno_webgpu fix)
- ✅ `webgpu:api,validation,capability_checks,limits,*` - 60%, wgslLanguageFeatures missing (#8884); workgroup storage size validation unimplemented; interstage vars; dx12 bind group timing

## Compute Pipeline
- ✅ `webgpu:api,validation,compute_pipeline:limits,workgroup_storage_size:*` - 0%
- ✅ `webgpu:api,validation,compute_pipeline:overrides,identifier:*` - 46%
- ✅ `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*` - 0%
- ✅ `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*` - 0%
- ✅ `webgpu:api,validation,compute_pipeline:resource_compatibility:*` - 84%
- ✅ `webgpu:api,validation,compute_pipeline:storage_texture,format:*` - 85%

## CreateBindGroup
- ✅ `webgpu:api,validation,createBindGroup:buffer,resource_state:*` - #7881
- ✅ `webgpu:api,validation,createBindGroup:external_texture,*` - 0%, no external texture in deno
- ✅ `webgpu:api,validation,createBindGroup:texture,resource_state:*` - crash, #7881

## CreateBindGroupLayout
- ✅ `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_pipeline_layout:*` - 0%, using max() instead of +=
- ✅ `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_buffer_type:*` - 25%, writable storage buffers not allowed in VERTEX
- ✅ `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_storage_texture_access:*` - 25%, write-access storage textures not allowed in VERTEX
- ✅ `webgpu:api,validation,createBindGroupLayout:visibility:*` - 25%, missing per-stage storage limits

## CreatePipelineLayout
- ✅ `webgpu:api,validation,createPipelineLayout:*` - #4738 and more

## CreateView
- ✅ `webgpu:api,validation,createView:texture_state:*` - 0%, #7881

## Encoding Commands
- ✅ `webgpu:api,validation,encoding,cmds,copyTextureToTexture:copy_ranges:*` - ***, #8118
- ✅ `webgpu:api,validation,encoding,cmds,debug:*` - 92%, #8039
- ✅ `webgpu:api,validation,encoding,cmds,render,*` - #7912
- ✅ `webgpu:api,validation,encoding,cmds,setImmediates:*` - feature not implemented
- ✅ Lines 36-38: `webgpu:api,validation,encoding,cmds,setBindGroup:*` - #7881

## Encoding Other
- ✅ `webgpu:api,validation,encoding,createRenderBundleEncoder:*` - 26%, empty attachments, format compatibility
- ✅ `webgpu:api,validation,encoding,encoder_open_state:*` - #7857
- ✅ Lines 41-42: `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:default_bind_group_layouts_never_match,*` - 75%, empty bind group layouts compatibility
- ✅ Lines 43-44: `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:empty_bind_group_layouts_never_requires_empty_bind_groups,*` - 17%, null bind groups, https://github.com/gfx-rs/wgpu/issues/4738
- ✅ `webgpu:api,validation,encoding,queries,resolveQuerySet:*` - 93%, #7881
- ✅ `webgpu:api,validation,encoding,render_bundle:*` - 81%, readonly flag normalization mismatch

## Error Scope
- ✅ `webgpu:api,validation,error_scope:*` - 75%, Metal backend crashes on OOM texture allocation (null pointer dereference)

## GetBindGroupLayout
- ✅ `webgpu:api,validation,getBindGroupLayout:*` - 29%, rejects valid indices beyond defined layouts

## Image Copy
- ✅ `webgpu:api,validation,image_copy,buffer_texture_copies:*` - #7946
- ✅ `webgpu:api,validation,image_copy,layout_related:offset_alignment:*` - fails on DX12

## Layout Shader Compat
- ✅ `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*` - 99%, Storage texture access mode validation too strict (read-write should match write)

## Non-Filterable Texture
- ✅ `webgpu:api,validation,non_filterable_texture:non_filterable_texture_with_filtering_sampler:*` - 80%, depth textures with filtering samplers

## Query Set
- ✅ `webgpu:api,validation,query_set,create:count:*` - 0%, wgpu incorrectly rejects zero-count query sets

## Queue
- ✅ `webgpu:api,validation,queue,buffer_mapped:*` - ***, vulkan
- ✅ `webgpu:api,validation,queue,destroyed,*` - 71%, writeBuffer/writeTexture return value, destroyed query set
- ✅ `webgpu:api,validation,queue,writeBuffer:ranges:*` - 0%, missing OperationError for invalid ranges

## Render Pass
- ✅ `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:*` - 94%, TypeError instead of validation error
- ✅ `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,loadOp_storeOp_match_depthReadOnly_stencilReadOnly:*` - 33%, missing validation: depth/stencil load/store ops provided for missing texture format aspects
- ✅ `webgpu:api,validation,render_pass,render_pass_descriptor:occlusionQuerySet,query_set_type:*` - 50%, occlusionQuerySet type validation

## Render Pipeline Depth/Stencil
- ✅ `webgpu:api,validation,render_pipeline,depth_stencil_state:depth_write,frag_depth:*` - 86%, #8856
- ✅ `webgpu:api,validation,render_pipeline,depth_stencil_state:depthCompare_optional:*` - #8830

## Render Pipeline Fragment State
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,color_target_count:*` - #8856
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,use_blend_src:*` - 62%, #8856
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets,blend:*` - 95%, blend factors reading source alpha require vec4
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets:*` - 3%, color target without shader output requires writeMask=0
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:targets_blend:*` - 0%, missing min/max blend factor validation
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:targets_write_mask:*` - 50%, TypeError instead of validation error

## Render Pipeline Other
- ✅ `webgpu:api,validation,render_pipeline,inter_stage:*` - 15%, inter-stage type validation gaps and interpolation default handling
- ✅ `webgpu:api,validation,render_pipeline,misc:external_texture:*` - 0%, no external texture in deno
- ✅ `webgpu:api,validation,render_pipeline,misc:storage_texture,format:*` - 8%, similar to compute pipeline issue
- ✅ `webgpu:api,validation,render_pipeline,multisample_state:*` - 60%, #8779
- ✅ `webgpu:api,validation,render_pipeline,overrides:*` - 17%, missing validation: invalid pipeline constant identifiers silently ignored
- ✅ `webgpu:api,validation,render_pipeline,resource_compatibility:*` - 90%
  - Has dedicated triage report
  - 12 failures: read-write storage textures not compatible with write-only shaders in fragment stage
- ✅ `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_attribute_limit:*` - 0%
  - Has dedicated triage report
  - Empty vertex buffers not counted in validation, GPUInternalError has empty message
- ✅ `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_buffer_limit:*` - 0%
  - Has dedicated triage report
  - Empty vertex buffers not counted toward limit

## Resource Usages
- ✅ `webgpu:api,validation,resource_usages,texture,in_pass_encoder:subresources_and_binding_types_combination_for_color:*` - 92%, #3126
- ✅ `webgpu:api,validation,resource_usages,texture,in_render_common:subresources,depth_stencil_attachment_and_bind_group:*` - 69%, #8705

## State
- ✅ `webgpu:api,validation,state,device_lost,destroy:*` - crash

## Texture
- ✅ `webgpu:api,validation,texture,destroy:submit_a_destroyed_texture_as_attachment:*` - 44%, #8714

---

# Shader Validation Tests

## Declarations
- ✅ `webgpu:shader,validation,decl,context_dependent_resolution:*` - 92%
  - Has dedicated triage report
  - 1 failure: f16 treated as reserved keyword even when enabled, 4 skipped (wgslLanguageFeatures not exposed)
- ✅ `webgpu:shader,validation,decl,override:*` - 93%, #5158
  - Has dedicated triage report
  - 3 failures: unrestricted_pointer_parameters not implemented
- ✅ `webgpu:shader,validation,decl,var:*` - 97%
  - Has dedicated triage report
  - 17 failures: template arg trailing comma (9), atomics in read-only storage (4), storage textures in vertex shaders (4)

## Expression Access
- ✅ `webgpu:shader,validation,expression,access,array:*` - 97%, runtime indexing with compile-time-known values, #4390
- ✅ `webgpu:shader,validation,expression,access,matrix:*` - 93%, runtime OOB matrix access with literal indices incorrectly rejected
- ✅ `webgpu:shader,validation,expression,access,vector:*` - 52%, runtime indexing and swizzle

## Expression Binary
- ✅ `webgpu:shader,validation,expression,binary,add_sub_mul:*` - 95%, u32 const-eval overflow, f16 overflow not rejected, atomics #5474
- ✅ `webgpu:shader,validation,expression,binary,and_or_xor:*` - 96%, #5474
- ✅ `webgpu:shader,validation,expression,binary,bitwise_shift:*` - 97%, atomics #5474, partial eval errors
- ✅ `webgpu:shader,validation,expression,binary,comparison:*` - 74%, #5474
- ✅ `webgpu:shader,validation,expression,binary,div_rem:*` - 86%, #5474
- ✅ `webgpu:shader,validation,expression,binary,short_circuiting_and_or:*` - 92%, #8440

## Expression Call Builtin (Lines 93-168)

- `webgpu:shader,validation,expression,binary,*`

### Math Functions
- ✅ `abs:*` - 98%, atomic type validation gap, #5474
- ✅ `acos:*` - 71%, missing domain validation
- ✅ `acosh:*` - 56%, missing domain validation (agent hit rate limit, skipped detailed triage)
- ✅ `asin:*` - 71%, missing NaN/Inf validation for AbstractFloat/F16
- ✅ `asinh:*` - 85%, asinh(large f32) produces infinity
- ✅ `atanh:*` - 72%, domain (-1,1) not validated
- ✅ `cosh:*` - 61%, overflow detection missing for abstract types, f16
- ✅ `cross:*` - 86%, abstract types const eval overflow
- ✅ `degrees:*` - 73%, const eval overflow for abstract/f16
- ✅ `derivatives:*` - 83%, f16 support validation
- ✅ `determinant:*` - 71%, abstract types const eval
- ✅ `distance:*` - 66%, scalar uses wrong formula (sqrt vs abs)
- ✅ `dot:*` - 79%, const eval overflow for abstract/f16
- ✅ `exp:*` - 61%, const eval overflow for abstract/f16
- ✅ `exp2:*` - 80%, f16 overflow detection missing
- ✅ `extractBits:*` - 55%, const eval issues
- ✅ `faceForward:*` - 68%, const eval overflow
- ✅ `firstLeadingBit:*` - 98%, atomic type validation gap, #5474
- ✅ `firstTrailingBit:*` - 98%, atomic type validation gap, #5474
- ✅ `fma:*` - 85%, const eval issues
- ✅ `frexp:*` - 64%, const/override eval not supported
- ✅ `insertBits:*` - 73%, const eval not supported
- ✅ `inverseSqrt:*` - 61%, const eval overflow for abstract/f16
- ✅ `ldexp:*` - 43%, const eval not implemented
- ✅ `length:*` - 74%, const eval overflow for abstract/f16
- ✅ `log:*` - 64%, domain validation missing
- ✅ `log2:*` - 64%, domain validation missing
- ✅ `mix:*` - 66%, const eval issues
- ✅ `modf:*` - 52%, const eval not fully implemented
- ✅ `log:*` - 64%, domain validation (x > 0) missing for abstract/f16
- ✅ `log2:*` - 64%, domain validation (x > 0) missing for abstract/f16
- ✅ `mix:*` - 66%, const eval issues
- ✅ `modf:*` - 52%, const eval not fully implemented
- ✅ `normalize:*` - 63%, missing const/override eval overflow validation (intermediate values overflow to infinity)
- ✅ `pow:*` - 63%, missing const/override eval validation (negative base, zero^non-positive, overflow), http://github.com/gpuweb/issues/4527
- ✅ `quantizeToF16:*` - 77%, overflow validation issues (const eval not implemented, #4507)
- ✅ `reflect:*` - 39%, const eval not implemented
- ✅ `refract:*` - 44%, const eval not implemented
- ✅ `sinh:*` - 61%, const eval overflow for abstract/f16
- ✅ `smoothstep:*` - 69%, const eval issues
- ✅ `sqrt:*` - 64%, domain validation (x >= 0) missing for abstract/f16
- ✅ `transpose:*` - 86%, missing const eval

### Bit/Integer Operations
- ✅ `atomics:*` - 86%, atomics in vertex shaders, invalid address spaces/access modes
- ✅ `bitcast:*` - 57%, bitcast const-eval unimplemented; size validation missing; f16 vector validation incorrect
- ✅ `clamp:*` - 71%, clamp low<=high constraint not checked for const/override parameters
- ✅ `countLeadingZeros:*` - 98%, atomic type validation gap, #5474
- ✅ `countOneBits:*` - 98%, atomic type validation gap, #5474
- ✅ `countTrailingZeros:*` - 98%, atomic type validation gap, #5474
- ✅ `extractBits:*` - 55%, const eval issues
- ✅ `firstLeadingBit:*` - 98%, atomic type validation gap, #5474
- ✅ `firstTrailingBit:*` - 98%, atomic type validation gap, #5474
- ✅ `insertBits:*` - 73%, missing const eval support
- ✅ `reverseBits:*` - 98%, atomic type validation gap (#5474)
- ✅ `select:*` - >99%, #5474

### Geometry/Vector Operations
- ✅ `cross:*` - 86%, abstract int/float overflow issues in const eval

### Derivatives
- ✅ `derivatives:*` - 83%, f16 support not properly validated

### Dot Products (Packed)
- ✅ `dot4I8Packed:*` - 0%, #8884
- ✅ `dot4U8Packed:*` - 0%, #8884

### Pack Functions
- ✅ `pack2x16float:*` - 80%, missing const eval (#4507)
- ✅ `pack2x16snorm:*` - 88%, missing const eval (#4507)
- ✅ `pack2x16unorm:*` - 88%, missing const eval (#4507)
- ✅ `pack4x8snorm:*` - 88%, missing const eval (#4507)
- ✅ `pack4x8unorm:*` - 88%, missing const eval (#4507)
- ✅ `pack4xI8:*` - 21%, #8884
- ✅ `pack4xI8Clamp:*` - 21%, #8884
- ✅ `pack4xU8:*` - 21%, #8884
- ✅ `pack4xU8Clamp:*` - 21%, #8884

### Unpack Functions
- ✅ `unpack2x16float:*` - 81%, missing const eval (#4507)
- ✅ `unpack2x16snorm:*` - 81%, missing const eval (#4507)
- ✅ `unpack2x16unorm:*` - 81%, missing const eval (#4507)
- ✅ `unpack4x8snorm:*` - 81%, missing const eval (#4507)
- ✅ `unpack4x8unorm:*` - 81%, missing const eval (#4507)
- ✅ `unpack4xI8:*` - 75%, wgslLanguageFeatures not implemented (#8884)
- ✅ `unpack4xU8:*` - 75%, wgslLanguageFeatures not implemented (#8884)

### Texture Functions
- ✅ `textureDimensions:*` - >99%, no external texture in deno
- ✅ `textureGather:*` - 99%, #8876
- ✅ `textureGatherCompare:*` - 99%, edge case in offset or external texture validation
- ✅ `textureLoad:*` - 99%, texture_external not implemented
- ✅ `textureSample:*` - 99%, texture_external not implemented
- ✅ `textureSampleBaseClampToEdge:*` - 96%, texture_external not exposed in deno_webgpu
- ✅ `textureSampleBias:*` - 99%, missing offset validation (range & cube texture)
- ✅ `textureSampleCompare:*` - 99%, missing offset range validation (-8 to +7)
- ✅ `textureSampleCompareLevel:*` - 99%, missing offset range validation (-8 to +7)
- ✅ `textureSampleGrad:*` - 99%, missing offset range validation + depth textures incorrectly accepted
- ✅ `textureSampleLevel:*` - 99%, missing offset validation (cube texture & value range)

### Constructor
- ✅ `value_constructor:*` - 92%, value constructors lack must-use validation + abstract-float matrix validation issues

## Expression Other
- ✅ `webgpu:shader,validation,expression,early_evaluation:*` - 67%
  - Has dedicated triage report
  - 6 failures: early evaluation of mixed override/runtime composites causes infinity errors
- ✅ `webgpu:shader,validation,expression,matrix,*` - 83%, #5474, #8790, #8868
  - Has dedicated section in triage.md with multiple issues documented
- ✅ `webgpu:shader,validation,expression,precedence:*` - 76%, #4536
- ✅ `webgpu:shader,validation,expression,unary,*` - 74%, #8884, #5474
  - Has dedicated triage report
  - 76 failures: pointer_composite_access feature not advertised (#8884)
  - 3 failures: atomic direct reference (#5474), matrix negation validation gap

## Extension
- ✅ `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*` - 61%, @blend_src validation gaps
- ✅ `webgpu:shader,validation,extension,pointer_composite_access:*` - 50%, #8884
  - Has dedicated triage report
  - All 14 failures: wgslLanguageFeatures API not exposed in deno_webgpu
- ✅ `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*` - 0%, #8884

## Functions
- ✅ `webgpu:shader,validation,functions,alias_analysis:*` - 2%, #7650
- ✅ `webgpu:shader,validation,functions,restrictions:*` - 81%, texture_external

## Parse
- ✅ `webgpu:shader,validation,parse,attribute:*` - 93%
  - Has dedicated triage report
  - 3 failures: group index validation during shader module creation (should only validate at pipeline creation)
- ✅ `webgpu:shader,validation,parse,blankspace:*` - 95%
  - Has dedicated triage report
  - 1 failure: null characters in comments not rejected
- ✅ `webgpu:shader,validation,parse,comments:*` - 93%
  - Has dedicated triage report
  - 1 failure: unterminated block comments not rejected
- ✅ `webgpu:shader,validation,parse,diagnostic:*` - 50%, #6458
- ✅ `webgpu:shader,validation,parse,literal:*` - 96%, #7046
- ✅ `webgpu:shader,validation,parse,must_use:*` - 97%, #8876
- ✅ `webgpu:shader,validation,parse,requires:*` - 14%, #8884
- ❌ `webgpu:shader,validation,parse,shadow_builtins:*` - 60%, #7405

## Shader IO
- ✅ `webgpu:shader,validation,shader_io,align:*` - 98%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 1 failure: trailing comma in @align attribute
- ✅ `webgpu:shader,validation,shader_io,binding:*` - 95%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 1 failure: trailing comma in @binding attribute
- ✅ `webgpu:shader,validation,shader_io,builtins:*` - 50%, trailing comma not accepted
- ✅ `webgpu:shader,validation,shader_io,group_and_binding:*` - 87%
  - Has dedicated triage report
  - All 67 failures: texture_external not implemented
- ✅ `webgpu:shader,validation,shader_io,group:*` - 95%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 1 failure: trailing comma not accepted in @group attribute
- ✅ `webgpu:shader,validation,shader_io,id:*` - 94%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage doc (shader_io_id_triage.md)
- ✅ `webgpu:shader,validation,shader_io,interpolate:*` - 91%, https://github.com/gfx-rs/wgpu/issues/8892
  - 2 failures: trailing comma not accepted, @id accepted on const declarations (validation gap)
  - Has dedicated section in triage.md and dedicated triage doc (shader_io_interpolate_triage.md)
- ✅ `webgpu:shader,validation,shader_io,layout_constraints:*` - 99%
  - Has dedicated triage report
  - 1 failure: struct alignment not inferred from @align on members (layouter issue)
- ✅ `webgpu:shader,validation,shader_io,locations:*` - 98%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage section in triage.md
  - 1 failure: trailing comma not accepted in @location attribute
- ✅ `webgpu:shader,validation,shader_io,pipeline_stage:*` - 92%, stage attributes incorrectly accepted on var<private> and var<storage>
  - Has dedicated triage report: docs/cts-triage/shader_io_pipeline_stage.md
  - 6 failures: stage attributes (@vertex, @fragment, @compute) silently ignored on variable declarations
- ✅ `webgpu:shader,validation,shader_io,size:*` - 92%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 3 failures: trailing comma (2 tests), large size validation, runtime-sized array validation
- ✅ `webgpu:shader,validation,shader_io,workgroup_size:*` - 83%, https://github.com/gfx-rs/wgpu/issues/8892
  - Has dedicated triage report
  - 10 failures: trailing comma syntax, type mixing validation, attribute placement validation

## Statement
- ✅ `webgpu:shader,validation,statement,continue:*` - 90%, #7650
  - Tests continue statement placement and validation
  - Known issue: Missing validation that continue must not bypass declarations used in continuing block
  - Related to statement behavior validation issue #7650
- ✅ `webgpu:shader,validation,statement,for:*` - 93%
  - Tests for statement parsing and validation
  - Known issues: phony assignment and increment/decrement in for-loop initializer/continuation, empty loop behavior analysis
  - Dedicated triage report: docs/cts-triage/statement_for_triage.md
- ✅ `webgpu:shader,validation,statement,increment_decrement:*` - 98%, #5474
  - Tests increment/decrement statement validation
  - Known issue: atomic type validation gap (4 failures)
  - Dedicated triage report: docs/cts-triage/increment_decrement_triage.md
- ✅ `webgpu:shader,validation,statement,loop:*` - 92%, #7650
  - Tests loop statement parsing and validation, including behavior analysis
  - Known issue: infinite loop detection (empty loop, loop with only continue/discard)
  - Related to statement behavior validation issue #7650
- ✅ `webgpu:shader,validation,statement,phony:*` - 90%
  - Tests phony assignment statement validation (syntax and type checking)
  - Known issues: for-loop semicolon parsing, atomic/unsized array validation
  - Has dedicated section "Phony Assignment Statement Validation" in triage.md
- ✅ `webgpu:shader,validation,statement,statement_behavior:*` - #7650
- ✅ `webgpu:shader,validation,statement,switch:*` - #7650
  - Tests switch statement validation including condition types, case type matching, and parsing
  - Related to statement behavior validation issue #7650

## Types
- ✅ `webgpu:shader,validation,types,*` - 86%
  - Tests type validation across 10 categories: alias, array, atomics, enumerant, matrix, pointer, ref, struct, textures, vector
  - Main issues: trailing commas in template args (~100 tests), keyword shadowing (10 tests), f16 constructors without enable (9 tests)
  - Has dedicated section "Shader Validation Types" in triage.md with comprehensive analysis

## Uniformity
- ✅ `webgpu:shader,validation,uniformity,*` - 21%, #4369
