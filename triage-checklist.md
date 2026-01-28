# CTS Triage Checklist

This document tracks which selectors in `cts_runner/fail.lst` have been properly triaged.

Entries in this file **MUST** report only the triage yes/no indicator symbol and test
selectors. **DO NOT** annotate failure percentages, bug numbers, or any other comment
describing failures in this file.

## Triage Requirements

A selector is considered "triaged" if:
1. It appears in `docs/cts-triage/README.md` with a dedicated section OR is covered in a comprehensive summary section
2. It has a bug reference or descriptive note in `fail.lst` (percentage alone doesn't count)
3. The root cause is documented

## Status Legend

- ✅ **TRIAGED** - Has bug reference/note in fail.lst AND appears in triage README
- ❌ **UNTRIAGED** - Missing from fail.lst, triage README, or both, or manually marked for re-triage

---

# API Validation Tests

## Buffer Mapping
- ✅ `webgpu:api,validation,buffer,mapping:*`

## Capability Checks
- ✅ `webgpu:api,validation,capability_checks,features,*`
- ✅ `webgpu:api,validation,capability_checks,limits,*`

## Compute Pipeline
- ✅ `webgpu:api,validation,compute_pipeline:limits,workgroup_storage_size:*`
- ✅ `webgpu:api,validation,compute_pipeline:overrides,identifier:*`
- ✅ `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits,workgroup_storage_size:*`
- ✅ `webgpu:api,validation,compute_pipeline:overrides,workgroup_size,limits:*`
- ✅ `webgpu:api,validation,compute_pipeline:resource_compatibility:*`
- ✅ `webgpu:api,validation,compute_pipeline:storage_texture,format:*`

## CreateBindGroup
- ✅ `webgpu:api,validation,createBindGroup:buffer,resource_state:*`
- ✅ `webgpu:api,validation,createBindGroup:external_texture,*`
- ✅ `webgpu:api,validation,createBindGroup:texture,resource_state:*`

## CreateBindGroupLayout
- ✅ `webgpu:api,validation,createBindGroupLayout:max_resources_per_stage,in_pipeline_layout:*`
- ✅ `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_buffer_type:*`
- ✅ `webgpu:api,validation,createBindGroupLayout:visibility,VERTEX_shader_stage_storage_texture_access:*`
- ✅ `webgpu:api,validation,createBindGroupLayout:visibility:*`

## CreatePipelineLayout
- ✅ `webgpu:api,validation,createPipelineLayout:*`

## CreateView
- ✅ `webgpu:api,validation,createView:texture_state:*`

## Encoding Commands
- ✅ `webgpu:api,validation,encoding,cmds,copyTextureToTexture:copy_ranges:*`
- ✅ `webgpu:api,validation,encoding,cmds,debug:*`
- ✅ `webgpu:api,validation,encoding,cmds,render,*`
- ✅ `webgpu:api,validation,encoding,cmds,setImmediates:*`
- ✅ `webgpu:api,validation,encoding,cmds,setBindGroup:*`

## Encoding Other
- ✅ `webgpu:api,validation,encoding,createRenderBundleEncoder:*`
- ✅ `webgpu:api,validation,encoding,encoder_open_state:*`
- ✅ `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:default_bind_group_layouts_never_match,*`
- ✅ `webgpu:api,validation,encoding,programmable,pipeline_bind_group_compat:empty_bind_group_layouts_never_requires_empty_bind_groups,*`
- ✅ `webgpu:api,validation,encoding,queries,resolveQuerySet:*`
- ✅ `webgpu:api,validation,encoding,render_bundle:*`

## Error Scope
- ✅ `webgpu:api,validation,error_scope:*`

## GetBindGroupLayout
- ✅ `webgpu:api,validation,getBindGroupLayout:*`

## Image Copy
- ✅ `webgpu:api,validation,image_copy,buffer_texture_copies:*`
- ✅ `webgpu:api,validation,image_copy,layout_related:offset_alignment:*`

## Layout Shader Compat
- ✅ `webgpu:api,validation,layout_shader_compat:pipeline_layout_shader_exact_match:*`

## Non-Filterable Texture
- ✅ `webgpu:api,validation,non_filterable_texture:non_filterable_texture_with_filtering_sampler:*`

## Query Set
- ✅ `webgpu:api,validation,query_set,create:count:*`

## Queue
- ✅ `webgpu:api,validation,queue,buffer_mapped:*`
- ✅ `webgpu:api,validation,queue,destroyed,*`
- ✅ `webgpu:api,validation,queue,writeBuffer:ranges:*`

## Render Pass
- ✅ `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,depth_clear_value:*`
- ✅ `webgpu:api,validation,render_pass,render_pass_descriptor:depth_stencil_attachment,loadOp_storeOp_match_depthReadOnly_stencilReadOnly:*`
- ✅ `webgpu:api,validation,render_pass,render_pass_descriptor:occlusionQuerySet,query_set_type:*`

## Render Pipeline Depth/Stencil
- ✅ `webgpu:api,validation,render_pipeline,depth_stencil_state:depth_write,frag_depth:*`
- ✅ `webgpu:api,validation,render_pipeline,depth_stencil_state:depthCompare_optional:*`

## Render Pipeline Fragment State
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,color_target_count:*`
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:dual_source_blending,use_blend_src:*`
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets,blend:*`
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:pipeline_output_targets:*`
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:targets_blend:*`
- ✅ `webgpu:api,validation,render_pipeline,fragment_state:targets_write_mask:*`

## Render Pipeline Other
- ✅ `webgpu:api,validation,render_pipeline,inter_stage:*`
- ✅ `webgpu:api,validation,render_pipeline,misc:external_texture:*`
- ✅ `webgpu:api,validation,render_pipeline,misc:storage_texture,format:*`
- ✅ `webgpu:api,validation,render_pipeline,multisample_state:*`
- ✅ `webgpu:api,validation,render_pipeline,overrides:*`
- ✅ `webgpu:api,validation,render_pipeline,resource_compatibility:*`
- ✅ `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_attribute_limit:*`
- ✅ `webgpu:api,validation,render_pipeline,vertex_state:max_vertex_buffer_limit:*`

## Resource Usages
- ✅ `webgpu:api,validation,resource_usages,texture,in_pass_encoder:subresources_and_binding_types_combination_for_color:*`
- ✅ `webgpu:api,validation,resource_usages,texture,in_render_common:subresources,depth_stencil_attachment_and_bind_group:*`

## State
- ✅ `webgpu:api,validation,state,device_lost,destroy:*`

## Texture
- ✅ `webgpu:api,validation,texture,destroy:submit_a_destroyed_texture_as_attachment:*`

---

# Shader Validation Tests

## Declarations
- ✅ `webgpu:shader,validation,decl,context_dependent_resolution:*`
- ✅ `webgpu:shader,validation,decl,override:*`
- ✅ `webgpu:shader,validation,decl,var:*`

## Expression Access
- ✅ `webgpu:shader,validation,expression,access,array:*`
- ✅ `webgpu:shader,validation,expression,access,matrix:*`
- ✅ `webgpu:shader,validation,expression,access,vector:*`

## Expression Binary
- ✅ `webgpu:shader,validation,expression,binary,add_sub_mul:*`
- ✅ `webgpu:shader,validation,expression,binary,and_or_xor:*`
- ✅ `webgpu:shader,validation,expression,binary,bitwise_shift:*`
- ✅ `webgpu:shader,validation,expression,binary,comparison:*`
- ✅ `webgpu:shader,validation,expression,binary,div_rem:*`
- ✅ `webgpu:shader,validation,expression,binary,short_circuiting_and_or:*`

## Expression Call Builtin (Lines 93-168)

- `webgpu:shader,validation,expression,binary,*`

### Math Functions
- ✅ `abs:*`
- ✅ `acos:*`
- ✅ `acosh:*`
- ✅ `asin:*`
- ✅ `asinh:*`
- ✅ `atanh:*`
- ✅ `cosh:*`
- ✅ `cross:*`
- ✅ `degrees:*`
- ✅ `derivatives:*`
- ✅ `determinant:*`
- ✅ `distance:*`
- ✅ `dot:*`
- ✅ `exp:*`
- ✅ `exp2:*`
- ✅ `extractBits:*`
- ✅ `faceForward:*`
- ✅ `firstLeadingBit:*`
- ✅ `firstTrailingBit:*`
- ✅ `fma:*`
- ✅ `frexp:*`
- ✅ `insertBits:*`
- ✅ `inverseSqrt:*`
- ✅ `ldexp:*`
- ✅ `length:*`
- ✅ `log:*`
- ✅ `log2:*`
- ✅ `mix:*`
- ✅ `modf:*`
- ✅ `log:*`
- ✅ `log2:*`
- ✅ `mix:*`
- ✅ `modf:*`
- ✅ `normalize:*`
- ✅ `pow:*`
- ✅ `quantizeToF16:*`
- ✅ `reflect:*`
- ✅ `refract:*`
- ✅ `sinh:*`
- ✅ `smoothstep:*`
- ✅ `sqrt:*`
- ✅ `transpose:*`

### Bit/Integer Operations
- ✅ `atomics:*`
- ✅ `bitcast:*`
- ✅ `clamp:*`
- ✅ `countLeadingZeros:*`
- ✅ `countOneBits:*`
- ✅ `countTrailingZeros:*`
- ✅ `extractBits:*`
- ✅ `firstLeadingBit:*`
- ✅ `firstTrailingBit:*`
- ✅ `insertBits:*`
- ✅ `reverseBits:*`
- ✅ `select:*`

### Geometry/Vector Operations
- ✅ `cross:*`

### Derivatives
- ✅ `derivatives:*`

### Dot Products (Packed)
- ✅ `dot4I8Packed:*`
- ✅ `dot4U8Packed:*`

### Pack Functions
- ✅ `pack2x16float:*`
- ✅ `pack2x16snorm:*`
- ✅ `pack2x16unorm:*`
- ✅ `pack4x8snorm:*`
- ✅ `pack4x8unorm:*`
- ✅ `pack4xI8:*`
- ✅ `pack4xI8Clamp:*`
- ✅ `pack4xU8:*`
- ✅ `pack4xU8Clamp:*`

### Unpack Functions
- ✅ `unpack2x16float:*`
- ✅ `unpack2x16snorm:*`
- ✅ `unpack2x16unorm:*`
- ✅ `unpack4x8snorm:*`
- ✅ `unpack4x8unorm:*`
- ✅ `unpack4xI8:*`
- ✅ `unpack4xU8:*`

### Texture Functions
- ✅ `textureDimensions:*`
- ✅ `textureGather:*`
- ✅ `textureGatherCompare:*`
- ✅ `textureLoad:*`
- ✅ `textureSample:*`
- ✅ `textureSampleBaseClampToEdge:*`
- ✅ `textureSampleBias:*`
- ✅ `textureSampleCompare:*`
- ✅ `textureSampleCompareLevel:*`
- ✅ `textureSampleGrad:*`
- ✅ `textureSampleLevel:*`

### Constructor
- ✅ `value_constructor:*`

## Expression Other
- ✅ `webgpu:shader,validation,expression,early_evaluation:*`
- ✅ `webgpu:shader,validation,expression,matrix,*`
- ✅ `webgpu:shader,validation,expression,precedence:*`
- ✅ `webgpu:shader,validation,expression,unary,*`

## Extension
- ✅ `webgpu:shader,validation,extension,dual_source_blending:blend_src_usage:*`
- ✅ `webgpu:shader,validation,extension,pointer_composite_access:*`
- ✅ `webgpu:shader,validation,extension,readonly_and_readwrite_storage_textures:*`

## Functions
- ✅ `webgpu:shader,validation,functions,alias_analysis:*`
- ✅ `webgpu:shader,validation,functions,restrictions:*`

## Parse
- ✅ `webgpu:shader,validation,parse,attribute:*`
- ✅ `webgpu:shader,validation,parse,blankspace:*`
- ✅ `webgpu:shader,validation,parse,comments:*`
- ✅ `webgpu:shader,validation,parse,diagnostic:*`
- ✅ `webgpu:shader,validation,parse,literal:*`
- ✅ `webgpu:shader,validation,parse,must_use:*`
- ✅ `webgpu:shader,validation,parse,requires:*`
- ❌ `webgpu:shader,validation,parse,shadow_builtins:*`

## Shader IO
- ✅ `webgpu:shader,validation,shader_io,align:*`
- ✅ `webgpu:shader,validation,shader_io,binding:*`
- ✅ `webgpu:shader,validation,shader_io,builtins:*`
- ✅ `webgpu:shader,validation,shader_io,group_and_binding:*`
- ✅ `webgpu:shader,validation,shader_io,group:*`
- ✅ `webgpu:shader,validation,shader_io,id:*`
- ✅ `webgpu:shader,validation,shader_io,interpolate:*`
- ✅ `webgpu:shader,validation,shader_io,layout_constraints:*`
- ✅ `webgpu:shader,validation,shader_io,locations:*`
- ✅ `webgpu:shader,validation,shader_io,pipeline_stage:*`
- ✅ `webgpu:shader,validation,shader_io,size:*`
- ✅ `webgpu:shader,validation,shader_io,workgroup_size:*`

## Statement
- ✅ `webgpu:shader,validation,statement,continue:*`
- ✅ `webgpu:shader,validation,statement,for:*`
- ✅ `webgpu:shader,validation,statement,increment_decrement:*`
- ✅ `webgpu:shader,validation,statement,loop:*`
- ✅ `webgpu:shader,validation,statement,phony:*`
- ✅ `webgpu:shader,validation,statement,statement_behavior:*`
- ✅ `webgpu:shader,validation,statement,switch:*`

## Types
- ✅ `webgpu:shader,validation,types,*`

## Uniformity
- ✅ `webgpu:shader,validation,uniformity,*`
