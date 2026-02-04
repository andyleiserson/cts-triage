# CTS Triage Checklist

This document tracks which selectors in `cts_runner/fail.lst` have been properly triaged.

Entries in this file **MUST** report only the triage yes/no status and test
selectors. **DO NOT** annotate failure percentages, bug numbers, or any other
comment describing failures in this file.

Do not use emoji. Use only markdown todo lists, i.e.:
- [x] `triaged selector`
- [ ] `untriaged selector`

## Triage Requirements

A selector is considered "triaged" if:
1. It appears in `docs/cts-triage/README.md` with a dedicated section OR is covered in a comprehensive summary section
2. It has a bug reference or descriptive note in `fail.lst` (percentage alone doesn't count)
3. The root cause is documented

In some cases a test has been marked for re-triage in this file even though it
previously satisfied the criteria above.

## Status Legend

- [x] **TRIAGED** - Has bug reference/note in fail.lst AND appears in triage README
- [ ] **UNTRIAGED** - Missing from fail.lst, triage README, or both, or manually marked for re-triage

---

# API Validation Tests

## Buffer Mapping
- [x] `webgpu:api,validation,buffer,mapping:*`

## Capability Checks
- [x] `webgpu:api,validation,capability_checks,features,*`
- [x] `webgpu:api,validation,capability_checks,limits,*`

## Compute Pipeline
- [x] `webgpu:api,validation,compute_pipeline:limits,workgroup_storage_size:*`
- [x] `webgpu:api,validation,compute_pipeline:overrides,identifier:*`
- [x] `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*`
- [x] `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*`
- [x] `webgpu:api,validation,compute_pipeline:resource_compatibility:*`
- [x] `webgpu:api,validation,compute_pipeline:storage_texture,format:*`

## CreateBindGroup
- [x] `webgpu:api,validation,createBindGroup:buffer,resource_state:*`
- [x] `webgpu:api,validation,createBindGroup:external_texture,*`
- [x] `webgpu:api,validation,createBindGroup:texture,resource_state:*`

## CreateBindGroupLayout
- [x] `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_pipeline_layout:*`
- [x] `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_buffer_type:*`
- [x] `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_storage_texture_access:*`
- [x] `webgpu:api,validation,createBindGroupLayout:visibility:*`

## CreatePipelineLayout
- [x] `webgpu:api,validation,createPipelineLayout:*`

## CreateView
- [x] `webgpu:api,validation,createView:texture_state:*`

## Encoding Commands
- [x] `webgpu:api,validation,encoding,cmds,copyTextureToTexture:copy_ranges:*`
- [x] `webgpu:api,validation,encoding,cmds,debug:*`
- [x] `webgpu:api,validation,encoding,cmds,render,*`
- [x] `webgpu:api,validation,encoding,cmds,setImmediates:*`
- [x] `webgpu:api,validation,encoding,cmds,setBindGroup:*`

## Encoding Other
- [x] `webgpu:api,validation,encoding,createRenderBundleEncoder:*`
- [x] `webgpu:api,validation,encoding,encoder_open_state:*`
- [x] `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:default_bind_group_layouts_never_match,*`
- [x] `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:empty_bind_group_layouts_never_requires_empty_bind_groups,*`
- [x] `webgpu:api,validation,encoding,queries,resolveQuerySet:*`
- [x] `webgpu:api,validation,encoding,render_bundle:*`

## Error Scope
- [x] `webgpu:api,validation,error_scope:*`

## GetBindGroupLayout
- [x] `webgpu:api,validation,getBindGroupLayout:*`

## Image Copy
- [x] `webgpu:api,validation,image_copy,buffer_texture_copies:*`
- [x] `webgpu:api,validation,image_copy,layout_related:offset_alignment:*`

## Layout Shader Compat
- [x] `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*`

## Non-Filterable Texture
- [x] `webgpu:api,validation,non_filterable_texture:non_filterable_texture_with_filtering_sampler:*`

## Query Set
- [x] `webgpu:api,validation,query_set,create:count:*`

## Queue
- [x] `webgpu:api,validation,queue,buffer_mapped:*`
- [x] `webgpu:api,validation,queue,destroyed,*`
- [x] `webgpu:api,validation,queue,writeBuffer:ranges:*`

## Render Pass
- [x] `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:*`
- [x] `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,loadOp_storeOp_match_depthReadOnly_stencilReadOnly:*`
- [x] `webgpu:api,validation,render_pass,render_pass_descriptor:occlusionQuerySet,query_set_type:*`

## Render Pipeline Depth/Stencil
- [x] `webgpu:api,validation,render_pipeline,depth_stencil_state:depth_write,frag_depth:*`
- [x] `webgpu:api,validation,render_pipeline,depth_stencil_state:depthCompare_optional:*`

## Render Pipeline Fragment State
- [x] `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,color_target_count:*`
- [x] `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,use_blend_src:*`
- [x] `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets,blend:*`
- [x] `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets:*`
- [x] `webgpu:api,validation,render_pipeline,fragment_state:targets_blend:*`
- [x] `webgpu:api,validation,render_pipeline,fragment_state:targets_write_mask:*`

## Render Pipeline Other
- [x] `webgpu:api,validation,render_pipeline,inter_stage:*`
- [x] `webgpu:api,validation,render_pipeline,misc:external_texture:*`
- [x] `webgpu:api,validation,render_pipeline,misc:storage_texture,format:*`
- [x] `webgpu:api,validation,render_pipeline,multisample_state:*`
- [x] `webgpu:api,validation,render_pipeline,overrides:*`
- [x] `webgpu:api,validation,render_pipeline,resource_compatibility:*`
- [x] `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_attribute_limit:*`
- [x] `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_buffer_limit:*`

## Resource Usages
- [x] `webgpu:api,validation,resource_usages,texture,in_pass_encoder:subresources_and_binding_types_combination_for_color:*`
- [x] `webgpu:api,validation,resource_usages,texture,in_render_common:subresources,depth_stencil_attachment_and_bind_group:*`

## State
- [x] `webgpu:api,validation,state,device_lost,destroy:*`

## Texture
- [x] `webgpu:api,validation,texture,destroy:submit_a_destroyed_texture_as_attachment:*`

---

# Shader Validation Tests

## Declarations
- [x] `webgpu:shader,validation,decl,context_dependent_resolution:*`
- [x] `webgpu:shader,validation,decl,override:*`
- [x] `webgpu:shader,validation,decl,var:*`

## Expression Access
- [x] `webgpu:shader,validation,expression,access,array:*`
- [x] `webgpu:shader,validation,expression,access,matrix:*`
- [x] `webgpu:shader,validation,expression,access,vector:*`

## Expression Binary
- [x] `webgpu:shader,validation,expression,binary,add_sub_mul:*`
- [x] `webgpu:shader,validation,expression,binary,and_or_xor:*`
- [x] `webgpu:shader,validation,expression,binary,bitwise_shift:*`
- [x] `webgpu:shader,validation,expression,binary,comparison:*`
- [x] `webgpu:shader,validation,expression,binary,div_rem:*`
- [x] `webgpu:shader,validation,expression,binary,short_circuiting_and_or:*`

## Expression Call Builtin (Lines 93-168)

- `webgpu:shader,validation,expression,binary,*`

### Math Functions
- [x] `abs:*`
- [x] `acos:*`
- [x] `acosh:*`
- [x] `asin:*`
- [x] `asinh:*`
- [x] `atanh:*`
- [x] `cosh:*`
- [x] `cross:*`
- [x] `degrees:*`
- [x] `derivatives:*`
- [x] `determinant:*`
- [x] `distance:*`
- [x] `dot:*`
- [x] `exp:*`
- [x] `exp2:*`
- [x] `extractBits:*`
- [x] `faceForward:*`
- [x] `firstLeadingBit:*`
- [x] `firstTrailingBit:*`
- [x] `fma:*`
- [x] `frexp:*`
- [x] `insertBits:*`
- [x] `inverseSqrt:*`
- [x] `ldexp:*`
- [x] `length:*`
- [x] `log:*`
- [x] `log2:*`
- [x] `mix:*`
- [x] `modf:*`
- [x] `log:*`
- [x] `log2:*`
- [x] `mix:*`
- [x] `modf:*`
- [x] `normalize:*`
- [x] `pow:*`
- [x] `quantizeToF16:*`
- [x] `reflect:*`
- [x] `refract:*`
- [x] `sinh:*`
- [x] `smoothstep:*`
- [x] `sqrt:*`
- [x] `transpose:*`

### Bit/Integer Operations
- [x] `atomics:*`
- [x] `bitcast:*`
- [x] `clamp:*`
- [x] `countLeadingZeros:*`
- [x] `countOneBits:*`
- [x] `countTrailingZeros:*`
- [x] `extractBits:*`
- [x] `firstLeadingBit:*`
- [x] `firstTrailingBit:*`
- [x] `insertBits:*`
- [x] `reverseBits:*`
- [x] `select:*`

### Geometry/Vector Operations
- [x] `cross:*`

### Derivatives
- [x] `derivatives:*`

### Dot Products (Packed)
- [x] `dot4I8Packed:*`
- [x] `dot4U8Packed:*`

### Pack Functions
- [x] `pack2x16float:*`
- [x] `pack2x16snorm:*`
- [x] `pack2x16unorm:*`
- [x] `pack4x8snorm:*`
- [x] `pack4x8unorm:*`
- [x] `pack4xI8:*`
- [x] `pack4xI8Clamp:*`
- [x] `pack4xU8:*`
- [x] `pack4xU8Clamp:*`

### Unpack Functions
- [x] `unpack2x16float:*`
- [x] `unpack2x16snorm:*`
- [x] `unpack2x16unorm:*`
- [x] `unpack4x8snorm:*`
- [x] `unpack4x8unorm:*`
- [x] `unpack4xI8:*`
- [x] `unpack4xU8:*`

### Texture Functions
- [x] `textureDimensions:*`
- [x] `textureGather:*`
- [x] `textureGatherCompare:*`
- [x] `textureLoad:*`
- [x] `textureSample:*`
- [x] `textureSampleBaseClampToEdge:*`
- [x] `textureSampleBias:*`
- [x] `textureSampleCompare:*`
- [x] `textureSampleCompareLevel:*`
- [x] `textureSampleGrad:*`
- [x] `textureSampleLevel:*`

### Constructor
- [x] `value_constructor:*`

## Expression Other
- [x] `webgpu:shader,validation,expression,early_evaluation:*`
- [x] `webgpu:shader,validation,expression,matrix,*`
- [x] `webgpu:shader,validation,expression,precedence:*`
- [x] `webgpu:shader,validation,expression,unary,*`

## Extension
- [x] `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*`
- [x] `webgpu:shader,validation,extension,pointer_composite_access:*`
- [x] `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*`

## Functions
- [x] `webgpu:shader,validation,functions,alias_analysis:*`
- [x] `webgpu:shader,validation,functions,restrictions:*`

## Parse
- [x] `webgpu:shader,validation,parse,attribute:*`
- [x] `webgpu:shader,validation,parse,blankspace:*`
- [x] `webgpu:shader,validation,parse,comments:*`
- [x] `webgpu:shader,validation,parse,diagnostic:*`
- [x] `webgpu:shader,validation,parse,literal:*`
- [x] `webgpu:shader,validation,parse,must_use:*`
- [x] `webgpu:shader,validation,parse,requires:*`
- [x] `webgpu:shader,validation,parse,shadow_builtins:*`

## Shader IO
- [x] `webgpu:shader,validation,shader_io,align:*`
- [x] `webgpu:shader,validation,shader_io,binding:*`
- [x] `webgpu:shader,validation,shader_io,builtins:*`
- [x] `webgpu:shader,validation,shader_io,group_and_binding:*`
- [x] `webgpu:shader,validation,shader_io,group:*`
- [x] `webgpu:shader,validation,shader_io,id:*`
- [x] `webgpu:shader,validation,shader_io,interpolate:*`
- [x] `webgpu:shader,validation,shader_io,layout_constraints:*`
- [x] `webgpu:shader,validation,shader_io,locations:*`
- [x] `webgpu:shader,validation,shader_io,pipeline_stage:*`
- [x] `webgpu:shader,validation,shader_io,size:*`
- [x] `webgpu:shader,validation,shader_io,workgroup_size:*`

## Statement
- [x] `webgpu:shader,validation,statement,continue:*`
- [x] `webgpu:shader,validation,statement,for:*`
- [x] `webgpu:shader,validation,statement,increment_decrement:*`
- [x] `webgpu:shader,validation,statement,loop:*`
- [x] `webgpu:shader,validation,statement,phony:*`
- [x] `webgpu:shader,validation,statement,statement_behavior:*`
- [x] `webgpu:shader,validation,statement,switch:*`

## Types
- [x] `webgpu:shader,validation,types,*`

## Uniformity
- [x] `webgpu:shader,validation,uniformity,*`
