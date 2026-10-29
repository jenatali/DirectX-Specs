# Pipeline state and shader linkage

<a id="ordinary-pipeline-state"></a>

## Ordinary pipeline state

A pipeline state object (PSO) groups shader programs and the fixed-function state needed to prepare their execution. Its creation description is immutable; selecting the object changes command-list pipeline state, not the description. Graphics and compute creation have distinct descriptions, while pipeline-state streams provide a versioned tagged representation.

Grouping this large amount of state into a single PSO enables drivers to
perform as much pre-processing of hardware commands as possible, no
matter what the particular state-mapping dependencies are for a
particular hardware. The intended usage model is for applications to
create Pipeline State Objects up-front (on the order of hundreds) where
driver pre-processing can happen, and then use those PSOs to change the
rendering state during the course of rendering.

The API imposes no fixed limit on the number of live pipeline state objects; creation remains subject to allocation and device constraints.

[Initial/reset and bundle pipeline selection](command-list-state.md#pipeline-selection) and [dynamic state overrides](command-list-state.md#dynamic-state) are canonical command-list contracts. Root-signature selection and compatibility are defined by [Binding](binding.md#pipeline-compatibility); selecting a PSO does not bind root arguments or replace descriptor lifetime/synchronization requirements.

[State objects](state-objects.md#object-model) define collections, libraries, subobject associations, linking and named executable programs. [Generic programs](state-objects.md#generic-programs) reuse the graphics/compute pipeline definitions here; their missing-subobject defaults are defined [with generic-program construction](state-objects.md#defaults-for-subobjects-missing-from-a-generic-program), not as command-list Reset defaults.

<a id="descriptions"></a>

## Aggregate descriptions

<a id="shader-data"></a>

### Shader bytecode and cached data

#### D3D12_SHADER_BYTECODE

```cpp
typedef struct D3D12_SHADER_BYTECODE
{
    const void* pShaderBytecode;
    SIZE_T BytecodeLength;
} D3D12_SHADER_BYTECODE;
```

`pShaderBytecode`

A pointer to a memory block that contains the shader data.

`BytecodeLength`

The size, in bytes, of the shader data that the **pShaderBytecode** member points to.

#### D3D12_CACHED_PIPELINE_STATE

```cpp
typedef struct D3D12_CACHED_PIPELINE_STATE
{
    const void* pCachedBlob;
    SIZE_T CachedBlobSizeInBytes;
} D3D12_CACHED_PIPELINE_STATE;
```

`pCachedBlob`

Specifies pointer that references the memory location of the cache.

`CachedBlobSizeInBytes`

Specifies the size of the cache in bytes.

`pShaderBytecode` and `BytecodeLength` describe a compiled shader blob. The program's stage must match its slot in the aggregate or stream, and its shader-model requirements must be supported by the device. Shader resource declarations and root-signature layout must satisfy [binding compatibility](binding.md#pipeline-compatibility).

<a id="graphics-description"></a>

### Graphics pipeline description

#### D3D12_GRAPHICS_PIPELINE_STATE_DESC

```cpp
typedef struct D3D12_GRAPHICS_PIPELINE_STATE_DESC
{
    ID3D12RootSignature*          pRootSignature;
    D3D12_SHADER_BYTECODE         VS;
    D3D12_SHADER_BYTECODE         PS;
    D3D12_SHADER_BYTECODE         DS;
    D3D12_SHADER_BYTECODE         HS;
    D3D12_SHADER_BYTECODE         GS;
    D3D12_STREAM_OUTPUT_DESC      StreamOutput;
    D3D12_BLEND_DESC              BlendState;
    UINT                          SampleMask;
    D3D12_RASTERIZER_DESC         RasterizerState;
    D3D12_DEPTH_STENCIL_DESC      DepthStencilState;
    D3D12_INPUT_LAYOUT_DESC       InputLayout;
    D3D12_INDEX_BUFFER_STRIP_CUT_VALUE IBStripCutValue;
    D3D12_PRIMITIVE_TOPOLOGY_TYPE PrimitiveTopologyType;
    UINT                          NumRenderTargets;
    DXGI_FORMAT                   RTVFormats[ D3D12_SIMULTANEOUS_RENDER_TARGET_COUNT ];
    DXGI_FORMAT                   DSVFormat;
    DXGI_SAMPLE_DESC              SampleDesc;
    UINT                          NodeMask;
    D3D12_CACHED_PIPELINE_STATE   CachedPSO;
    D3D12_PIPELINE_STATE_FLAGS    Flags;
} D3D12_GRAPHICS_PIPELINE_STATE_DESC;
```

`pRootSignature`

A pointer to the [ID3D12RootSignature](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12rootsignature) object.

`VS`

A [`D3D12_SHADER_BYTECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_shader_bytecode) structure that describes the vertex shader.

`PS`

A [`D3D12_SHADER_BYTECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_shader_bytecode) structure that describes the pixel shader.

`DS`

A [`D3D12_SHADER_BYTECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_shader_bytecode) structure that describes the domain shader.

`HS`

A [`D3D12_SHADER_BYTECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_shader_bytecode) structure that describes the hull shader.

`GS`

A [`D3D12_SHADER_BYTECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_shader_bytecode) structure that describes the geometry shader.

`StreamOutput`

A [`D3D12_STREAM_OUTPUT_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_stream_output_desc) structure that describes stream-output declarations and buffer strides.

`BlendState`

A [`D3D12_BLEND_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_blend_desc) structure that describes the blend state.

`SampleMask`

The sample mask for the blend state.

`RasterizerState`

A [`D3D12_RASTERIZER_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_rasterizer_desc) structure that describes the rasterizer state.

`DepthStencilState`

A [`D3D12_DEPTH_STENCIL_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_depth_stencil_desc) structure that describes the depth-stencil state.

`InputLayout`

A [`D3D12_INPUT_LAYOUT_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_input_layout_desc) structure that describes the input-buffer data for the input-assembler stage.

`IBStripCutValue`

Specifies the properties of the index buffer in a [`D3D12_INDEX_BUFFER_STRIP_CUT_VALUE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_index_buffer_strip_cut_value) enumeration.

`PrimitiveTopologyType`

A [`D3D12_PRIMITIVE_TOPOLOGY_TYPE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_primitive_topology_type)-typed value for the type of primitive, and ordering of the primitive data.

`NumRenderTargets`

The number of render target formats in the **RTVFormats** member.

`RTVFormats[8]`

An array of [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format)-typed values for the render target formats.

`DSVFormat`

A [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format)-typed value for the depth-stencil format.

`SampleDesc`

A [DXGI_SAMPLE_DESC](https://learn.microsoft.com/en-us/windows/win32/api/dxgicommon/ns-dxgicommon-dxgi_sample_desc) structure that specifies multisampling parameters.

`NodeMask`

For single GPU operation, set this to zero. If there are multiple GPU nodes, set bits to identify the nodes (the device's physical adapters) for which the graphics pipeline state is to apply. Each bit in the mask corresponds to a single node. Refer to [Multi-adapter systems](https://learn.microsoft.com/en-us/windows/win32/direct3d12/multi-engine).

`CachedPSO`

A cached pipeline state object, as a [`D3D12_CACHED_PIPELINE_STATE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_cached_pipeline_state) structure. pCachedBlob and CachedBlobSizeInBytes may be set to NULL and 0 respectively.

`Flags`

A [`D3D12_PIPELINE_STATE_FLAGS`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_pipeline_state_flags) enumeration constant such as for "tool debug".

<a id="compute-description"></a>

### Compute pipeline description

#### D3D12_COMPUTE_PIPELINE_STATE_DESC

```cpp
typedef struct D3D12_COMPUTE_PIPELINE_STATE_DESC
{
    ID3D12RootSignature*   pRootSignature;
    D3D12_SHADER_BYTECODE  CS;
    UINT                   NodeMask;
    D3D12_CACHED_PIPELINE_STATE   CachedPSO;
    D3D12_PIPELINE_STATE_FLAGS    Flags;
} D3D12_COMPUTE_PIPELINE_STATE_DESC;
```

`pRootSignature`

A pointer to the [ID3D12RootSignature](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12rootsignature) object.

`CS`

A [`D3D12_SHADER_BYTECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_shader_bytecode) structure that describes the compute shader.

`NodeMask`

For single GPU operation, set this to zero. If there are multiple GPU nodes, set bits to identify the nodes (the device's physical adapters) for which the compute pipeline state is to apply. Each bit in the mask corresponds to a single node. Refer to [Multi-adapter systems](https://learn.microsoft.com/en-us/windows/win32/direct3d12/multi-engine).

`CachedPSO`

A cached pipeline state object, as a [`D3D12_CACHED_PIPELINE_STATE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_cached_pipeline_state) structure. pCachedBlob and CachedBlobSizeInBytes may be set to NULL and 0 respectively.

`Flags`

A [`D3D12_PIPELINE_STATE_FLAGS`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_pipeline_state_flags) enumeration constant such as for "tool debug".

<a id="streams"></a>

## Pipeline-state streams

For recorded offline-compilation inputs, the [database stream and root-signature substitutions](state-object-databases-and-identity.md#storage-only-forms) define storage/reconstruction independently of actual pipeline creation. A successful database store/find does not create a PSO or validate the input against driver capabilities; the complete stream and creation requirements here still apply.

[ID3D12Compiler::CompilePipelineState](compiler-invocation-and-cache-output.md#id3d12compiler-compilepipelinestate) defines offline compilation and output storage. Its [input forms and target validation](compiler-invocation-and-cache-output.md#input-ownership) use serialized roots rather than live device roots and do not produce a device PSO. The ordinary creation and stream requirements here retain their own consuming-API scope.

Subobjects are the constituent components of a pipeline state
object. A pipeline state stream is constructed from a grouping (within a
struct) of these subobjects, which from a technical perspective are
structs that contain a pair of subobject type enum followed by the
respective subobject desc. The D3DX12 stream elements
represent these pairs, and allow short-form construction of custom
streams. The runtime assumes the stream is
constructed from structs of enum+desc pairs, which defines the padding
expected by the runtime.

#### D3D12_PIPELINE_STATE_STREAM_DESC

```cpp
typedef struct D3D12_PIPELINE_STATE_STREAM_DESC
{
    SIZE_T SizeInBytes;
    void* pPipelineStateSubobjectStream;
} D3D12_PIPELINE_STATE_STREAM_DESC;
```

`SizeInBytes`

Specifies the size of the opaque data structure pointed to by the pPipelineStateSubobjectStream member, in bytes.

`pPipelineStateSubobjectStream`

Specifies the address of a data structure that describes as a bytestream an arbitrary pipeline state subobject.

The format of the provided stream should consist of an alternating set of **[`D3D12_PIPELINE_STATE_SUBOBJECT_TYPE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_pipeline_state_subobject_type)**, and the corresponding subobject types for them (for example, **D3D12\_PIPELINE\_STATE\_SUBOBJECT\_TYPE\_RASTERIZER** pairs with **[`D3D12_RASTERIZER_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_rasterizer_desc)**). In terms of alignment, the D3D12 runtime expects subobjects to be individual struct pairs of enum-struct, rather than a continuous set of fields. It also expects them to be aligned to the natural word alignment of the system. This can be achieved either using `alignas(void*)`, or making a `union` of the enum + subobject and a `void*`.

It isn't sufficient to simply union the **D3D12\_PIPELINE\_STATE\_SUBOBJECT\_TYPE** with a **void\***, because this will result in certain subobjects being misaligned. For example, **D3D12\_PIPELINE\_STATE\_SUBOBJECT\_TYPE\_PRIMITIVE\_TOPOLOGY** is followed by a **[`D3D12_PRIMITIVE_TOPOLOGY_TYPE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_primitive_topology_type)** enum. If the subobject type is unioned with a **void\***, then there will be additional padding between these 2 members, resulting in corruption of the stream. The union therefore contains the complete enum/payload record, not just its tag.

An example of a suitable subobject for use with **[`D3D12_RASTERIZER_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_rasterizer_desc)** is shown here:

```cpp
struct alignas(void*) StreamingRasterizerDesc
{
private:
  D3D12_PIPELINE_STATE_SUBOBJECT_TYPE Type = D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_RASTERIZER;
public:
  D3D12_RASTERIZER_DESC Desc;
};
```

The runtime will determine the type of a pipeline stream (valid types being **COMPUTE**, **GRAPHICS**, and **MESH**), by which subobject type, out of **VS** (vertex shader), **CS** (compute shader), and **MS** (mesh shader), is found. If the runtime finds none of these shaders, it will fail pipeline creation. If it finds multiple of these shaders which are not null, it will also fail. This means it is legal to have both, for example, a **CS** and **VS** in your stream object, provided only one has a non-null pointer for the shader bytecode for any given call to **[ID3D12Device2::CreatePipelineState](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device2-createpipelinestate)**. Empty shader slots and other irrelevant subobjects do not add stages to the selected program; they do not waive tag/layout validity or other applicable creation checks. If a subobject is not provided (excluding the above required subobjects), the runtime will provide a default value for it.

<a id="stream-tags"></a>

### Stream tags and payloads

#### D3D12_PIPELINE_STATE_SUBOBJECT_TYPE

```cpp
typedef enum D3D12_PIPELINE_STATE_SUBOBJECT_TYPE
{
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_ROOT_SIGNATURE,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_VS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_PS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_DS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_HS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_GS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_CS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_STREAM_OUTPUT,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_BLEND,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_SAMPLE_MASK,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_RASTERIZER,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_DEPTH_STENCIL,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_INPUT_LAYOUT,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_IB_STRIP_CUT_VALUE,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_PRIMITIVE_TOPOLOGY,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_RENDER_TARGET_FORMATS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_DEPTH_STENCIL_FORMAT,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_SAMPLE_DESC,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_NODE_MASK,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_CACHED_PSO,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_FLAGS,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_DEPTH_STENCIL1,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_VIEW_INSTANCING,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_AS = 24,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_MS = 25,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_DEPTH_STENCIL2 = 26,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_RASTERIZER1 = 27,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_RASTERIZER2 = 28,
    D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_MAX_VALID = 30,
} D3D12_PIPELINE_STATE_SUBOBJECT_TYPE;
```

The following payload types accompany their `D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_*` tags:

| Tag suffix | Payload |
|---|---|
| `ROOT_SIGNATURE` | `ID3D12RootSignature*` |
| `VS` | `D3D12_SHADER_BYTECODE` |
| `PS` | `D3D12_SHADER_BYTECODE` |
| `DS` | `D3D12_SHADER_BYTECODE` |
| `HS` | `D3D12_SHADER_BYTECODE` |
| `GS` | `D3D12_SHADER_BYTECODE` |
| `CS` | `D3D12_SHADER_BYTECODE` |
| `STREAM_OUTPUT` | `D3D12_STREAM_OUTPUT_DESC` |
| `BLEND` | `D3D12_BLEND_DESC` |
| `SAMPLE_MASK` | `UINT` |
| `RASTERIZER` | `D3D12_RASTERIZER_DESC` |
| `DEPTH_STENCIL` | `D3D12_DEPTH_STENCIL_DESC` |
| `INPUT_LAYOUT` | `D3D12_INPUT_LAYOUT_DESC` |
| `IB_STRIP_CUT_VALUE` | `D3D12_INDEX_BUFFER_STRIP_CUT_VALUE` |
| `PRIMITIVE_TOPOLOGY` | `D3D12_PRIMITIVE_TOPOLOGY_TYPE` |
| `RENDER_TARGET_FORMATS` | `D3D12_RT_FORMAT_ARRAY` |
| `DEPTH_STENCIL_FORMAT` | `DXGI_FORMAT` |
| `SAMPLE_DESC` | `DXGI_SAMPLE_DESC` |
| `NODE_MASK` | `UINT` |
| `CACHED_PSO` | `D3D12_CACHED_PIPELINE_STATE` |
| `FLAGS` | `D3D12_PIPELINE_STATE_FLAGS` |
| `DEPTH_STENCIL1` | `D3D12_DEPTH_STENCIL_DESC1` |
| `VIEW_INSTANCING` | `D3D12_VIEW_INSTANCING_DESC` |
| `AS` | `D3D12_SHADER_BYTECODE` |
| `MS` | `D3D12_SHADER_BYTECODE` |
| `DEPTH_STENCIL2` | `D3D12_DEPTH_STENCIL_DESC2` |
| `RASTERIZER1` | `D3D12_RASTERIZER_DESC1` |
| `RASTERIZER2` | `D3D12_RASTERIZER_DESC2` |

A versioned payload retains its own declared size; fields introduced by a newer payload version use their defined defaults when the older version is selected. The applicable pipeline-creation validation still applies.

A stream must have nonempty, non-null storage containing complete supported records; an unknown tag or repeated subobject is invalid. Different versions of the same logical state, such as DEPTH_STENCIL/DEPTH_STENCIL1/DEPTH_STENCIL2 or the rasterizer versions, are alternatives, not additive independent records. Record order does not select precedence or authorize duplicates.

Omitted common stream members include an all-bits sample mask, sample description `{1, 0}`, no input-layout/stream-output entries, and disabled view instancing. Ordinary graphics topology defaults to triangle. Mesh topology omission is instead inferred from the mesh program; an explicit concrete topology must agree with that program. When depth/stencil state is omitted, depth is disabled if no DSV format is supplied, and enabled by the default state when a DSV format is supplied. An explicit depth/stencil record is not replaced by this omission rule.

A zero-initialized graphics description is not the default-state description: enums, blend factors and masks must have valid specified values. Defaults for omitted stream records do not turn zero bytes supplied as an explicit record into an omitted record.

<a id="supporting-state"></a>

## Immediate supporting state

<a id="input-and-stream-output"></a>

### Input layout and stream output

#### D3D12_INPUT_CLASSIFICATION

```cpp
typedef enum D3D12_INPUT_CLASSIFICATION
{
    D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA = 0,
    D3D12_INPUT_CLASSIFICATION_PER_INSTANCE_DATA = 1
} D3D12_INPUT_CLASSIFICATION;
```

#### D3D12_INPUT_ELEMENT_DESC

```cpp
typedef struct D3D12_INPUT_ELEMENT_DESC
{
    LPCSTR SemanticName;
    UINT SemanticIndex;
    DXGI_FORMAT Format;
    UINT InputSlot;
    UINT AlignedByteOffset;
    D3D12_INPUT_CLASSIFICATION InputSlotClass;
    UINT InstanceDataStepRate;
} D3D12_INPUT_ELEMENT_DESC;
```

#### D3D12_INPUT_LAYOUT_DESC

```cpp
typedef struct D3D12_INPUT_LAYOUT_DESC
{
    const D3D12_INPUT_ELEMENT_DESC* pInputElementDescs;
    UINT NumElements;
} D3D12_INPUT_LAYOUT_DESC;
```

#### D3D12_SO_DECLARATION_ENTRY

Defined in [D3D12_SO_DECLARATION_ENTRY](geometry-shaders-and-stream-output.md#so-declaration-entry).

#### D3D12_STREAM_OUTPUT_DESC

Defined in [D3D12_STREAM_OUTPUT_DESC](geometry-shaders-and-stream-output.md#stream-output-desc).

#### D3D12_INDEX_BUFFER_STRIP_CUT_VALUE

```cpp
typedef enum D3D12_INDEX_BUFFER_STRIP_CUT_VALUE
{
    D3D12_INDEX_BUFFER_STRIP_CUT_VALUE_DISABLED = 0,
    D3D12_INDEX_BUFFER_STRIP_CUT_VALUE_0xFFFF = 1,
    D3D12_INDEX_BUFFER_STRIP_CUT_VALUE_0xFFFFFFFF = 2
} D3D12_INDEX_BUFFER_STRIP_CUT_VALUE;
```

#### D3D12_PRIMITIVE_TOPOLOGY_TYPE

```cpp
typedef enum D3D12_PRIMITIVE_TOPOLOGY_TYPE
{
    D3D12_PRIMITIVE_TOPOLOGY_TYPE_UNDEFINED = 0,
    D3D12_PRIMITIVE_TOPOLOGY_TYPE_POINT     = 1,
    D3D12_PRIMITIVE_TOPOLOGY_TYPE_LINE      = 2,
    D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE  = 3,
    D3D12_PRIMITIVE_TOPOLOGY_TYPE_PATCH     = 4
} D3D12_PRIMITIVE_TOPOLOGY_TYPE;
```

#### D3D12_INPUT_ELEMENT_DESC members

`SemanticName`

The HLSL semantic associated with this element in a shader input-signature. See [HLSL Semantics](https://learn.microsoft.com/en-us/windows/win32/direct3dhlsl/dx-graphics-hlsl-semantics) for more info.

`SemanticIndex`

The semantic index for the element. A semantic index modifies a semantic, with an integer index number. A semantic index is only needed in a case where there is more than one element with the same semantic. For example, a 4x4 matrix would have four components each with the semantic name **matrix**, however each of the four components would have different semantic indices (0, 1, 2, and 3).

`Format`

A [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format)-typed value that specifies the format of the element data.

`InputSlot`

An integer value that identifies the input-assembler slot. For more info, see [Input Slots](https://learn.microsoft.com/en-us/windows/desktop/direct3d11/d3d10-graphics-programming-guide-input-assembler-stage-getting-started). Valid values are from 0 through 31 (`D3D12_IA_VERTEX_INPUT_RESOURCE_SLOT_COUNT` slots).

`AlignedByteOffset`

Optional. Offset, in bytes, to this element from the start of the vertex. Use D3D12\_APPEND\_ALIGNED\_ELEMENT (0xffffffff) for convenience to define the current element directly after the previous one, including any packing if necessary.

`InputSlotClass`

A value that identifies the input data class for a single input slot.

`InstanceDataStepRate`

The number of instances to draw using the same per-instance data before advancing in the buffer by one element. This value must be 0 for an element that contains per-vertex data (the slot class is set to the D3D12\_INPUT\_CLASSIFICATION\_PER\_VERTEX\_DATA member of [`D3D12_INPUT_CLASSIFICATION`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_input_classification)).

#### D3D12_INPUT_LAYOUT_DESC members

`pInputElementDescs`

An array of [`D3D12_INPUT_ELEMENT_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_input_element_desc) structures that describe the data types of the input-assembler stage.

`NumElements`

The number of input-data types in the array of input elements that the **pInputElementDescs** member points to.

#### D3D12_SO_DECLARATION_ENTRY members

Defined in [D3D12_SO_DECLARATION_ENTRY](geometry-shaders-and-stream-output.md#so-declaration-entry).

#### D3D12_STREAM_OUTPUT_DESC members

Defined in [D3D12_STREAM_OUTPUT_DESC](geometry-shaders-and-stream-output.md#stream-output-desc).

<a id="input-layout-constraints"></a>

#### Input-layout and stream-output constraints

`SemanticName` and `SemanticIndex` identify shader input/output signature entries, not the buffer byte offset. Element formats must support vertex input and be compatible with the vertex-shader input signature. Stream output declarations describe shader output components and gaps; an empty declaration does not make an otherwise invalid shader-stage combination legal. `PrimitiveTopologyType` is the PSO's point/line/triangle/patch class; the command-list [primitive topology](vertex-processing.md#topology-enumeration) supplies the corresponding detailed list/strip/adjacency/patch selection. [Input assembly and direct draws](vertex-processing.md#input-assembly) define vertex/index fetching, primitive assembly and generated identifiers.

A nonempty input-layout or stream-output count requires the corresponding valid array storage. Input-layout elements cannot exceed `D3D12_IA_VERTEX_INPUT_STRUCTURE_ELEMENT_COUNT` (32). `D3D12_APPEND_ALIGNED_ELEMENT` requests appended element placement rather than making `AlignedByteOffset` an optional field. Stream output may use `D3D12_SO_NO_RASTERIZED_STREAM` to select no rasterized stream. Elements sharing an `InputSlot` must have the same `InputSlotClass` and `InstanceDataStepRate`. For per-instance data, a step rate of zero means that the instance data is never advanced. `D3D12_APPEND_ALIGNED_ELEMENT` places an element after the preceding element in the same input slot, rounded up to the new element format's required alignment. If it is the first element in that slot, its appended offset is zero.

A nonempty input layout requires the root signature's `D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT` permission; enabled stream output requires `D3D12_ROOT_SIGNATURE_FLAG_ALLOW_STREAM_OUTPUT`. These creation conditions are distinct from actually binding vertex or stream-output buffers on the command list.

<a id="blend-state"></a>

### Blend and output formats

These descriptions select the [output-merger blend/logic behavior](output-merger.md#blending), including [dual-source requirements](output-merger.md#dual-source-blending), [precision](output-merger.md#blending-precision), [write masks](output-merger.md#write-masks) and [alpha-to-coverage](output-merger.md#alpha-to-coverage). Description/PSO creation and compatibility remain defined here.

#### D3D12_BLEND

```cpp
typedef enum D3D12_BLEND
{
    D3D12_BLEND_ZERO = 1,
    D3D12_BLEND_ONE = 2,
    D3D12_BLEND_SRC_COLOR = 3,
    D3D12_BLEND_INV_SRC_COLOR = 4,
    D3D12_BLEND_SRC_ALPHA = 5,
    D3D12_BLEND_INV_SRC_ALPHA = 6,
    D3D12_BLEND_DEST_ALPHA = 7,
    D3D12_BLEND_INV_DEST_ALPHA = 8,
    D3D12_BLEND_DEST_COLOR = 9,
    D3D12_BLEND_INV_DEST_COLOR = 10,
    D3D12_BLEND_SRC_ALPHA_SAT = 11,
    D3D12_BLEND_BLEND_FACTOR = 14,
    D3D12_BLEND_INV_BLEND_FACTOR = 15,
    D3D12_BLEND_SRC1_COLOR = 16,
    D3D12_BLEND_INV_SRC1_COLOR = 17,
    D3D12_BLEND_SRC1_ALPHA = 18,
    D3D12_BLEND_INV_SRC1_ALPHA = 19,
    D3D12_BLEND_ALPHA_FACTOR = 20,
    D3D12_BLEND_INV_ALPHA_FACTOR = 21
} D3D12_BLEND;
```

#### D3D12_BLEND_OP

```cpp
typedef enum D3D12_BLEND_OP
{
    D3D12_BLEND_OP_ADD = 1,
    D3D12_BLEND_OP_SUBTRACT = 2,
    D3D12_BLEND_OP_REV_SUBTRACT = 3,
    D3D12_BLEND_OP_MIN = 4,
    D3D12_BLEND_OP_MAX = 5,
} D3D12_BLEND_OP;
```

#### D3D12_LOGIC_OP

```cpp
typedef enum D3D12_LOGIC_OP
{
    D3D12_LOGIC_OP_CLEAR = 0,
    D3D12_LOGIC_OP_SET,
    D3D12_LOGIC_OP_COPY,
    D3D12_LOGIC_OP_COPY_INVERTED,
    D3D12_LOGIC_OP_NOOP,
    D3D12_LOGIC_OP_INVERT,
    D3D12_LOGIC_OP_AND,
    D3D12_LOGIC_OP_NAND,
    D3D12_LOGIC_OP_OR,
    D3D12_LOGIC_OP_NOR,
    D3D12_LOGIC_OP_XOR,
    D3D12_LOGIC_OP_EQUIV,
    D3D12_LOGIC_OP_AND_REVERSE,
    D3D12_LOGIC_OP_AND_INVERTED,
    D3D12_LOGIC_OP_OR_REVERSE,
    D3D12_LOGIC_OP_OR_INVERTED,
} D3D12_LOGIC_OP;
```

#### D3D12_COLOR_WRITE_ENABLE

```cpp
typedef enum D3D12_COLOR_WRITE_ENABLE
{
    D3D12_COLOR_WRITE_ENABLE_RED = 1,
    D3D12_COLOR_WRITE_ENABLE_GREEN = 2,
    D3D12_COLOR_WRITE_ENABLE_BLUE = 4,
    D3D12_COLOR_WRITE_ENABLE_ALPHA = 8,
    D3D12_COLOR_WRITE_ENABLE_ALL = (D3D12_COLOR_WRITE_ENABLE_RED|D3D12_COLOR_WRITE_ENABLE_GREEN|
        D3D12_COLOR_WRITE_ENABLE_BLUE|D3D12_COLOR_WRITE_ENABLE_ALPHA),
} D3D12_COLOR_WRITE_ENABLE;
```

#### D3D12_RENDER_TARGET_BLEND_DESC

```cpp
typedef struct D3D12_RENDER_TARGET_BLEND_DESC
{
    BOOL BlendEnable;
    BOOL LogicOpEnable;
    D3D12_BLEND SrcBlend;
    D3D12_BLEND DestBlend;
    D3D12_BLEND_OP BlendOp;
    D3D12_BLEND SrcBlendAlpha;
    D3D12_BLEND DestBlendAlpha;
    D3D12_BLEND_OP BlendOpAlpha;
    D3D12_LOGIC_OP LogicOp;
    UINT8 RenderTargetWriteMask;
} D3D12_RENDER_TARGET_BLEND_DESC;
```

#### D3D12_BLEND_DESC

```cpp
typedef struct D3D12_BLEND_DESC
{
    BOOL AlphaToCoverageEnable;
    BOOL IndependentBlendEnable;
    D3D12_RENDER_TARGET_BLEND_DESC RenderTarget[D3D12_SIMULTANEOUS_RENDER_TARGET_COUNT];
} D3D12_BLEND_DESC;
```

#### D3D12_RT_FORMAT_ARRAY

```cpp
typedef struct D3D12_RT_FORMAT_ARRAY
{
    DXGI_FORMAT RTFormats[D3D12_SIMULTANEOUS_RENDER_TARGET_COUNT];
    UINT NumRenderTargets;
} D3D12_RT_FORMAT_ARRAY;
```

#### D3D12_BLEND_DESC members

`AlphaToCoverageEnable`

Specifies whether to use alpha-to-coverage as a multisampling technique when setting a pixel to a render target. For more info about using alpha-to-coverage, see [Alpha-To-Coverage](https://learn.microsoft.com/en-us/windows/desktop/direct3d11/d3d10-graphics-programming-guide-blend-state).

`IndependentBlendEnable`

Specifies whether to enable independent blending in simultaneous render targets. Set to **TRUE** to enable independent blending. If set to **FALSE**, only the **RenderTarget**[0] members are used; **RenderTarget**[1..7] are ignored.

Logic operations cannot be combined with blending; the shared restrictions are stated below.

`RenderTarget[8]`

An array of [`D3D12_RENDER_TARGET_BLEND_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_render_target_blend_desc) structures that describe the blend states for render targets; these correspond to the eight render targets that can be bound to the [output-merger stage](https://learn.microsoft.com/en-us/windows/desktop/direct3d11/d3d10-graphics-programming-guide-output-merger-stage) at one time.

#### D3D12_RENDER_TARGET_BLEND_DESC members

`BlendEnable`

Specifies whether to enable (or disable) blending. Set to **TRUE** to enable blending.

Note

It's not valid for *LogicOpEnable* and *BlendEnable* to both be **TRUE**.

`LogicOpEnable`

Specifies whether to enable (or disable) a logical operation. Set to **TRUE** to enable a logical operation.

Note

It's not valid for *LogicOpEnable* and *BlendEnable* to both be **TRUE**.

`SrcBlend`

A [`D3D12_BLEND`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_blend)-typed value that specifies the operation to perform on the RGB value that the pixel shader outputs. The **BlendOp** member defines how to combine the **SrcBlend** and **DestBlend** operations.

`DestBlend`

A [`D3D12_BLEND`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_blend)-typed value that specifies the operation to perform on the current RGB value in the render target. The **BlendOp** member defines how to combine the **SrcBlend** and **DestBlend** operations.

`BlendOp`

A [`D3D12_BLEND_OP`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_blend_op)-typed value that defines how to combine the **SrcBlend** and **DestBlend** operations.

`SrcBlendAlpha`

A [`D3D12_BLEND`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_blend)-typed value that specifies the operation to perform on the alpha value that the pixel shader outputs. Blend options that end in \_COLOR are not allowed. The **BlendOpAlpha** member defines how to combine the **SrcBlendAlpha** and **DestBlendAlpha** operations.

`DestBlendAlpha`

A [`D3D12_BLEND`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_blend)-typed value that specifies the operation to perform on the current alpha value in the render target. Blend options that end in \_COLOR are not allowed. The **BlendOpAlpha** member defines how to combine the **SrcBlendAlpha** and **DestBlendAlpha** operations.

`BlendOpAlpha`

A [`D3D12_BLEND_OP`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_blend_op)-typed value that defines how to combine the **SrcBlendAlpha** and **DestBlendAlpha** operations.

`LogicOp`

A [`D3D12_LOGIC_OP`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_logic_op)-typed value that specifies the logical operation to configure for the render target.

`RenderTargetWriteMask`

A combination of [`D3D12_COLOR_WRITE_ENABLE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_color_write_enable)-typed values that are combined by using a bitwise OR operation. The resulting value specifies a write mask.

When you set the **LogicOpEnable** member of the first element of the **RenderTarget** array (**RenderTarget**[0]) to **TRUE**, you must also set the **BlendEnable** member of **RenderTarget**[0] to **FALSE**, and the **IndependentBlendEnable** member of `D3D12_BLEND_DESC` to **FALSE**. This reflects the limitation in hardware that you can't mix logic operations with blending across multiple render targets, and that when you use a logic operation, you must apply the same logic operation to all render targets.

`D3D12_BLEND_ALPHA_FACTOR` and `D3D12_BLEND_INV_ALPHA_FACTOR` use the alpha component of the command-list blend factor and require `D3D12_FEATURE_DATA_D3D12_OPTIONS13.AlphaBlendFactorSupported`. Logic-operation use has its own `D3D12_FEATURE_DATA_D3D12_OPTIONS.OutputMergerLogicOp` capability; the two feature conditions are independent.

`NumRenderTargets` selects the active `RTVFormats` entries and cannot exceed eight. `DSVFormat`, active render-target formats and `SampleDesc` describe the attachments expected by the pipeline, not bindings of resource views. [Formats and views](formats.md#support) defines per-format and sample support. The [R9G9B9E5 write-mask restriction](../D3D12R9G9B9E5Format.md#interaction-with-rendertargetwritemask) is an additional named format constraint.

<a id="rasterizer"></a>

### Rasterizer descriptions

#### D3D12_FILL_MODE

```cpp
typedef enum D3D12_FILL_MODE
{
    D3D12_FILL_MODE_WIREFRAME = 2,
    D3D12_FILL_MODE_SOLID = 3
} D3D12_FILL_MODE;
```

#### D3D12_CULL_MODE

```cpp
typedef enum D3D12_CULL_MODE
{
    D3D12_CULL_MODE_NONE = 1,
    D3D12_CULL_MODE_FRONT = 2,
    D3D12_CULL_MODE_BACK = 3
} D3D12_CULL_MODE;
```

#### D3D12_CONSERVATIVE_RASTERIZATION_MODE

```cpp
typedef enum D3D12_CONSERVATIVE_RASTERIZATION_MODE
{
    D3D12_CONSERVATIVE_RASTERIZATION_MODE_OFF       = 0,
    D3D12_CONSERVATIVE_RASTERIZATION_MODE_ON        = 1,
} D3D12_CONSERVATIVE_RASTERIZATION_MODE;
```

#### D3D12_LINE_RASTERIZATION_MODE

```cpp
typedef enum D3D12_LINE_RASTERIZATION_MODE
{
    D3D12_LINE_RASTERIZATION_MODE_ALIASED,
    D3D12_LINE_RASTERIZATION_MODE_ALPHA_ANTIALIASED,
    D3D12_LINE_RASTERIZATION_MODE_QUADRILATERAL_WIDE,
    D3D12_LINE_RASTERIZATION_MODE_QUADRILATERAL_NARROW,
} D3D12_LINE_RASTERIZATION_MODE;
```

#### D3D12_RASTERIZER_DESC

```cpp
typedef struct D3D12_RASTERIZER_DESC
{
    D3D12_FILL_MODE FillMode;
    D3D12_CULL_MODE CullMode;
    BOOL FrontCounterClockwise;
    INT DepthBias;
    FLOAT DepthBiasClamp;
    FLOAT SlopeScaledDepthBias;
    BOOL DepthClipEnable;
    BOOL MultisampleEnable;
    BOOL AntialiasedLineEnable;
    UINT ForcedSampleCount;
    D3D12_CONSERVATIVE_RASTERIZATION_MODE ConservativeRaster;
} D3D12_RASTERIZER_DESC;
```

#### D3D12_RASTERIZER_DESC1

```cpp
typedef struct D3D12_RASTERIZER_DESC1
{
    D3D12_FILL_MODE FillMode;
    D3D12_CULL_MODE CullMode;
    BOOL FrontCounterClockwise;
    FLOAT DepthBias;
    FLOAT DepthBiasClamp;
    FLOAT SlopeScaledDepthBias;
    BOOL DepthClipEnable;
    BOOL MultisampleEnable;
    BOOL AntialiasedLineEnable;
    UINT ForcedSampleCount;
    D3D12_CONSERVATIVE_RASTERIZATION_MODE ConservativeRaster;
} D3D12_RASTERIZER_DESC1;
```

#### D3D12_RASTERIZER_DESC2

```cpp
typedef struct D3D12_RASTERIZER_DESC2
{
    D3D12_FILL_MODE FillMode;
    D3D12_CULL_MODE CullMode;
    BOOL FrontCounterClockwise;
    FLOAT DepthBias;
    FLOAT DepthBiasClamp;
    FLOAT SlopeScaledDepthBias;
    BOOL DepthClipEnable;
    D3D12_LINE_RASTERIZATION_MODE LineRasterizationMode;
    UINT ForcedSampleCount;
    D3D12_CONSERVATIVE_RASTERIZATION_MODE ConservativeRaster;
} D3D12_RASTERIZER_DESC2;
```

`FillMode`

A [`D3D12_FILL_MODE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_fill_mode)-typed value that specifies the fill mode to use when rendering.

`CullMode`

A [`D3D12_CULL_MODE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_cull_mode)-typed value that specifies that triangles facing the specified direction are not drawn.

`FrontCounterClockwise`

Determines if a triangle is front- or back-facing. If this member is **TRUE**, a triangle will be considered front-facing if its vertices are counter-clockwise on the render target and considered back-facing if they are clockwise. If this parameter is **FALSE**, the opposite is true.

`DepthBias`

Depth value added to a given pixel. For info about depth bias, see [Depth Bias](rasterization.md#depth-bias).

`DepthBiasClamp`

Maximum depth bias of a pixel. For info about depth bias, see [Depth Bias](rasterization.md#depth-bias).

`SlopeScaledDepthBias`

Scalar on a given pixel's slope. For info about depth bias, see [Depth Bias](rasterization.md#depth-bias).

`DepthClipEnable`

Specifies whether to enable clipping based on distance.

The depth-clipping selector does not disable the other required clipping stages; detailed clipping and line algorithms are defined by [rasterization](rasterization.md#clipping).

`MultisampleEnable`

Selects the line algorithm together with AntialiasedLineEnable; the [line-mode description](#versioned-rasterizer) links the full interpretation. This field does not enable or disable point/triangle multisampling.

`AntialiasedLineEnable`

Specifies whether to enable line antialiasing; only applies if doing line drawing and **MultisampleEnable** is **FALSE**. See the [line-mode description](#versioned-rasterizer).

`ForcedSampleCount`

Type: **[UINT](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types)**

The sample count that is forced while UAV rendering or rasterizing. Valid values are 0, 1, 4, 8, and optionally 16. 0 indicates that the sample count is not forced.
**Note** If you want to render with **ForcedSampleCount** set to 1 or greater, you must follow these guidelines:
- Don't bind depth-stencil views.
- Disable depth testing.
- Ensure the shader doesn't output depth.
- If you have any render-target views bound ([`D3D12_DESCRIPTOR_HEAP_TYPE_RTV`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_descriptor_heap_type)) and **ForcedSampleCount** is greater than 1, ensure that every render target has only a single sample.
- Don't operate the shader at sample frequency. Therefore, [ID3D12ShaderReflection::IsSampleFrequencyShader](https://learn.microsoft.com/en-us/windows/win32/api/d3d12shader/nf-d3d12shader-id3d12shaderreflection-issamplefrequencyshader) returns **FALSE**.

Otherwise, rendering behavior is undefined.
`ConservativeRaster`

A [`D3D12_CONSERVATIVE_RASTERIZATION_MODE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_conservative_rasterization_mode)-typed value that identifies whether conservative rasterization is on or off.

<a id="versioned-rasterizer"></a>

#### Versioned rasterizer fields

To set the initial dynamic depth bias state, `D3D12_RASTERIZER_DESC1` has a `FLOAT` declared `DepthBias`. The value is part of the rasterizer payload supplied during pipeline creation; the command-list setter selects that object. Providing a non-integral value on unsupported hardware will result in a debug layer warning, as those values will be cast to `INT` before being submitted to the driver.

`D3D12_RASTERIZER_DESC2` replaces the two legacy line-selection BOOLs with `LineRasterizationMode`. `D3D12_FEATURE_DATA_D3D12_OPTIONS19` reports `RasterizerDesc2Supported` indicating that the struct itself is supported, and `NarrowQuadrilateralLinesSupported` indicating that `D3D12_LINE_RASTERIZATION_MODE_QUADRILATERAL_NARROW` can be used.

The [rasterization line-mode table](rasterization.md#line-mode-selection) defines the legacy Boolean combinations and explicit enum selection, including wide/narrow coverage. These are interpretation rules for the selected rasterizer description; PSO availability and capability gates remain here.

Enabling conservative rasterization requires a supported `D3D12_FEATURE_DATA_D3D12_OPTIONS.ConservativeRasterizationTier`. Its [fill-mode restriction](rasterization.md#conservative-fill-modes-interaction), [topology interaction](rasterization.md#conservative-ia-primitive-topology-interaction) and [coverage rules](rasterization.md#conservative-rasterization-rules-interaction) remain part of that feature contract, not a consequence of the ON enumerant merely existing.

<a id="depth-stencil"></a>

### Depth/stencil descriptions

The selected fields control the [output-merger depth/stencil algorithms](output-merger.md#depth-stencil), [per-face stencil/reference interaction](output-merger.md#independent-stencil), [depth bounds](output-merger.md#depth-bounds) and [read-only-view write gating](output-merger.md#read-only-depth-stencil). Their dynamic setter contracts are in [Output merger](output-merger.md#output-apis); canonical reset/bundle values remain in Command-list state.

#### D3D12_DEPTH_WRITE_MASK

```cpp
typedef enum D3D12_DEPTH_WRITE_MASK
{
    D3D12_DEPTH_WRITE_MASK_ZERO = 0,
    D3D12_DEPTH_WRITE_MASK_ALL = 1
} D3D12_DEPTH_WRITE_MASK;
```

#### D3D12_COMPARISON_FUNC

```cpp
typedef enum D3D12_COMPARISON_FUNC
{
    D3D12_COMPARISON_FUNC_NONE = 0,
    D3D12_COMPARISON_FUNC_NEVER = 1,
    D3D12_COMPARISON_FUNC_LESS = 2,
    D3D12_COMPARISON_FUNC_EQUAL = 3,
    D3D12_COMPARISON_FUNC_LESS_EQUAL = 4,
    D3D12_COMPARISON_FUNC_GREATER = 5,
    D3D12_COMPARISON_FUNC_NOT_EQUAL = 6,
    D3D12_COMPARISON_FUNC_GREATER_EQUAL = 7,
    D3D12_COMPARISON_FUNC_ALWAYS = 8
} D3D12_COMPARISON_FUNC;
```

#### D3D12_STENCIL_OP

```cpp
typedef enum D3D12_STENCIL_OP
{
    D3D12_STENCIL_OP_KEEP = 1,
    D3D12_STENCIL_OP_ZERO = 2,
    D3D12_STENCIL_OP_REPLACE = 3,
    D3D12_STENCIL_OP_INCR_SAT = 4,
    D3D12_STENCIL_OP_DECR_SAT = 5,
    D3D12_STENCIL_OP_INVERT = 6,
    D3D12_STENCIL_OP_INCR = 7,
    D3D12_STENCIL_OP_DECR = 8
} D3D12_STENCIL_OP;
```

#### D3D12_DEPTH_STENCILOP_DESC

```cpp
typedef struct D3D12_DEPTH_STENCILOP_DESC
{
    D3D12_STENCIL_OP StencilFailOp;
    D3D12_STENCIL_OP StencilDepthFailOp;
    D3D12_STENCIL_OP StencilPassOp;
    D3D12_COMPARISON_FUNC StencilFunc;
} D3D12_DEPTH_STENCILOP_DESC;
```

#### D3D12_DEPTH_STENCILOP_DESC1

```cpp
typedef struct D3D12_DEPTH_STENCILOP_DESC1
{
    D3D12_STENCIL_OP StencilFailOp;
    D3D12_STENCIL_OP StencilDepthFailOp;
    D3D12_STENCIL_OP StencilPassOp;
    D3D12_COMPARISON_FUNC StencilFunc;
    UINT8 StencilReadMask;
    UINT8 StencilWriteMask;
} D3D12_DEPTH_STENCILOP_DESC1;
```

#### D3D12_DEPTH_STENCIL_DESC

```cpp
typedef struct D3D12_DEPTH_STENCIL_DESC
{
    BOOL DepthEnable;
    D3D12_DEPTH_WRITE_MASK DepthWriteMask;
    D3D12_COMPARISON_FUNC DepthFunc;
    BOOL StencilEnable;
    UINT8 StencilReadMask;
    UINT8 StencilWriteMask;
    D3D12_DEPTH_STENCILOP_DESC FrontFace;
    D3D12_DEPTH_STENCILOP_DESC BackFace;
} D3D12_DEPTH_STENCIL_DESC;
```

#### D3D12_DEPTH_STENCIL_DESC1

```cpp
typedef struct D3D12_DEPTH_STENCIL_DESC1
{
    BOOL DepthEnable;
    D3D12_DEPTH_WRITE_MASK DepthWriteMask;
    D3D12_COMPARISON_FUNC DepthFunc;
    BOOL StencilEnable;
    UINT8 StencilReadMask;
    UINT8 StencilWriteMask;
    D3D12_DEPTH_STENCILOP_DESC FrontFace;
    D3D12_DEPTH_STENCILOP_DESC BackFace;
    BOOL DepthBoundsTestEnable;
} D3D12_DEPTH_STENCIL_DESC1;
```

#### D3D12_DEPTH_STENCIL_DESC2

```cpp
typedef struct D3D12_DEPTH_STENCIL_DESC2
{
    BOOL DepthEnable;
    D3D12_DEPTH_WRITE_MASK DepthWriteMask;
    D3D12_COMPARISON_FUNC DepthFunc;
    BOOL StencilEnable;
    D3D12_DEPTH_STENCILOP_DESC1 FrontFace;
    D3D12_DEPTH_STENCILOP_DESC1 BackFace;
    BOOL DepthBoundsTestEnable;
} D3D12_DEPTH_STENCIL_DESC2;
```

#### D3D12_DEPTH_STENCILOP_DESC members

`StencilFailOp`

A [`D3D12_STENCIL_OP`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_stencil_op)-typed value that identifies the stencil operation to perform when stencil testing fails.

`StencilDepthFailOp`

A [`D3D12_STENCIL_OP`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_stencil_op)-typed value that identifies the stencil operation to perform when stencil testing passes and depth testing fails.

`StencilPassOp`

A [`D3D12_STENCIL_OP`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_stencil_op)-typed value that identifies the stencil operation to perform when stencil testing and depth testing both pass.

`StencilFunc`

A [`D3D12_COMPARISON_FUNC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_comparison_func)-typed value that identifies the function that compares stencil data against existing stencil data.

#### D3D12_DEPTH_STENCIL_DESC members

`DepthEnable`

Specifies whether to enable depth testing. Set this member to **TRUE** to enable depth testing.

`DepthWriteMask`

A [`D3D12_DEPTH_WRITE_MASK`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_depth_write_mask)-typed value that identifies a portion of the depth-stencil buffer that can be modified by depth data.

`DepthFunc`

A [`D3D12_COMPARISON_FUNC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_comparison_func)-typed value that identifies a function that compares depth data against existing depth data.

`StencilEnable`

Specifies whether to enable stencil testing. Set this member to **TRUE** to enable stencil testing.

`StencilReadMask`

Identify a portion of the depth-stencil buffer for reading stencil data.

`StencilWriteMask`

Identify a portion of the depth-stencil buffer for writing stencil data.

`FrontFace`

A [`D3D12_DEPTH_STENCILOP_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_depth_stencilop_desc) structure that describes how to use the results of the depth test and the stencil test for front-facing primitives as defined by rasterizer winding state.

`BackFace`

A [`D3D12_DEPTH_STENCILOP_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_depth_stencilop_desc) structure that describes how to use the results of the depth test and the stencil test for back-facing primitives as defined by rasterizer winding state.

`D3D12_DEPTH_STENCIL_DESC1` adds `DepthBoundsTestEnable`; support is reported by `D3D12_FEATURE_DATA_D3D12_OPTIONS2.DepthBoundsTestSupported`. `D3D12_DEPTH_STENCIL_DESC2` moves the read/write masks into its per-face `D3D12_DEPTH_STENCILOP_DESC1` members. Independent front/back support is reported by `D3D12_FEATURE_DATA_D3D12_OPTIONS14.IndependentFrontAndBackStencilRefMaskSupported`. Actual bound stencil references and depth bounds remain [command-list state](command-list-state.md#stencil-references), not mutable fields in an already created object.

When independent front/back stencil masks are unsupported, the front and back read masks must match each other, and the front and back write masks must match each other. This does not prohibit the DESC2 representation when those masks agree.

<a id="state-defaults"></a>

### State-description defaults

These tables describe standard state defaults. Aggregate callers supply their inline state descriptions explicitly; stream omission and version-upgrade rules determine when a default is supplied by the runtime.

Selecting the original `D3D12_DEPTH_STENCIL_DESC` supplies `DepthBoundsTestEnable == FALSE` when it is expanded to the newer description. The common stencil read/write masks in DESC and DESC1 are propagated to both faces of DESC2. Their standard 0xff read/write defaults therefore apply to each face. The standard legacy rasterizer defaults have both line-selection BOOLs false, corresponding to `D3D12_LINE_RASTERIZATION_MODE_ALIASED`; conversion to DESC1/DESC2 preserves the selected bias/line state rather than accumulating separate records.

#### D3D12_BLEND_DESC defaults

Here are the default values for blend state.

| State | Default Value |
| --- | --- |
| AlphaToCoverageEnable | **FALSE** |
| IndependentBlendEnable | **FALSE** |
| RenderTarget[0].BlendEnable | **FALSE** |
| RenderTarget[0].LogicOpEnable | **FALSE** |
| RenderTarget[0].SrcBlend | D3D12\_BLEND\_ONE |
| RenderTarget[0].DestBlend | D3D12\_BLEND\_ZERO |
| RenderTarget[0].BlendOp | D3D12\_BLEND\_OP\_ADD |
| RenderTarget[0].SrcBlendAlpha | D3D12\_BLEND\_ONE |
| RenderTarget[0].DestBlendAlpha | D3D12\_BLEND\_ZERO |
| RenderTarget[0].BlendOpAlpha | D3D12\_BLEND\_OP\_ADD |
| RenderTarget[0].LogicOp | D3D12\_LOGIC\_OP\_NOOP |
| RenderTarget[0].RenderTargetWriteMask | D3D12\_COLOR\_WRITE\_ENABLE\_ALL |

#### D3D12_RASTERIZER_DESC defaults

| State | Default Value |
| --- | --- |
| **FillMode** | D3D12\_FILL\_MODE\_SOLID |
| **CullMode** | D3D12\_CULL\_MODE\_BACK |
| **FrontCounterClockwise** | **FALSE** |
| **DepthBias** | 0 |
| **DepthBiasClamp** | 0.0f |
| **SlopeScaledDepthBias** | 0.0f |
| **DepthClipEnable** | **TRUE** |
| **MultisampleEnable** | **FALSE** |
| **AntialiasedLineEnable** | **FALSE** |
| **ForcedSampleCount** | 0 |
| **ConservativeRaster** | **D3D12\_CONSERVATIVE\_RASTERIZATION\_MODE\_OFF** |

#### D3D12_DEPTH_STENCIL_DESC defaults

This table shows the default values of depth-stencil states.

| State | Default Value |
| --- | --- |
| DepthEnable | TRUE |
| DepthWriteMask | D3D12\_DEPTH\_WRITE\_MASK\_ALL |
| DepthFunc | D3D12\_COMPARISON\_FUNC\_LESS |
| StencilEnable | FALSE |
| StencilReadMask | D3D12\_DEFAULT\_STENCIL\_READ\_MASK |
| StencilWriteMask | D3D12\_DEFAULT\_STENCIL\_WRITE\_MASK |
| FrontFace.StencilFailOp <br>and<br><br>BackFace.StencilFailOp | D3D12\_STENCIL\_OP\_KEEP |
| FrontFace.StencilDepthFailOp <br>and<br><br>BackFace.StencilDepthFailOp | D3D12\_STENCIL\_OP\_KEEP |
| FrontFace.StencilPassOp <br>and<br><br>BackFace.StencilPassOp | D3D12\_STENCIL\_OP\_KEEP |
| FrontFace.StencilFunc <br>and<br><br>BackFace.StencilFunc | D3D12\_COMPARISON\_FUNC\_ALWAYS |

<a id="contributions"></a>

## Feature contributions and dynamic state

#### D3D12_PIPELINE_STATE_FLAGS

```cpp
typedef enum D3D12_PIPELINE_STATE_FLAGS
{
    D3D12_PIPELINE_STATE_FLAG_NONE = 0x0,
    D3D12_PIPELINE_STATE_FLAG_TOOL_DEBUG = 0x1,
    D3D12_PIPELINE_STATE_FLAG_DYNAMIC_DEPTH_BIAS = 0x4,
    D3D12_PIPELINE_STATE_FLAG_DYNAMIC_INDEX_BUFFER_STRIP_CUT = 0x8,
} D3D12_PIPELINE_STATE_FLAGS;
```

`D3D12_PIPELINE_STATE_FLAG_TOOL_DEBUG` requests additional compilation information for debugging. The public flag contract restricts this application flag to WARP devices.

`D3D12_PIPELINE_STATE_FLAG_DYNAMIC_DEPTH_BIAS` and `D3D12_PIPELINE_STATE_FLAG_DYNAMIC_INDEX_BUFFER_STRIP_CUT` permit their respective dynamic commands only with the corresponding device support. Creation flags do not make the PSO mutable. [Override legality, reselection and bundle effects](command-list-state.md#dynamic-application-state) and [driver state application](command-list-state.md#dynamic-driver-state) remain canonical there.

<a id="mesh-pipelines"></a>

### Mesh and amplification pipelines

`ID3D12Device2::CreatePipelineState`, along with a pipeline state stream descriptor that has a mesh shader
and (optionally) an amplification shader attached,
creates a graphics pipeline state object.

The descriptor:

1.  must have IA and streamout disabled

2.  must have a mesh shader attached (`MS`)

3.  may optionally have an amplification shader attached (`AS`)

4.  must **not** have any other shader types attached (`VS`, `GS`, `HS`, `DS`)

5.  must use DXIL bytecode for all attached shaders

Mesh/amplification creation requires `D3D12_FEATURE_DATA_D3D12_OPTIONS7.MeshShaderTier` other than `D3D12_MESH_SHADER_TIER_NOT_SUPPORTED`. The stream uses the MS and optional AS tags; a mesh pipeline can also include its pixel shader. Mesh [signature linkage](amplification-and-mesh-shaders.md#signature-linkage), [payloads](amplification-and-mesh-shaders.md#mesh-payload) and [dispatch](amplification-and-mesh-shaders.md#dispatchmesh) retain their owning definitions.

PSO creation validates the AS-produced and MS-consumed payload sizes for agreement and checks the [combined payload/output storage limits](amplification-and-mesh-shaders.md#amplification-and-mesh-shader). View instancing also imposes storage requirements: output attributes and shared storage that depend on `SV_ViewID` are expanded for the declared views under the applicable tier, and must fit the [mesh output-space limit](amplification-and-mesh-shaders.md#mesh-shader-output-size-limits) and [per-view shared-storage rules](amplification-and-mesh-shaders.md#sv_viewid). This is not satisfied merely by each shader validating independently with one view. Automatic validation coverage for ViewID-dependent groupshared storage is incomplete; the [mesh validation-coverage limit](amplification-and-mesh-shaders.md#mesh-view-validation) distinguishes required storage limits from achieved detection.

<a id="view-instancing"></a>

### View instancing

#### D3D12_VIEW_INSTANCING_FLAGS

```cpp
typedef enum D3D12_VIEW_INSTANCING_FLAGS
{
    D3D12_VIEW_INSTANCING_FLAG_NONE = 0x0,
    D3D12_VIEW_INSTANCING_FLAG_ENABLE_VIEW_INSTANCE_MASKING = 0x1,
} D3D12_VIEW_INSTANCING_FLAGS;
```

#### D3D12_VIEW_INSTANCE_LOCATION

```cpp
typedef struct D3D12_VIEW_INSTANCE_LOCATION
{
    UINT ViewportArrayIndex;
    UINT RenderTargetArrayIndex;
} D3D12_VIEW_INSTANCE_LOCATION;
```

#### D3D12_VIEW_INSTANCING_DESC

```cpp
typedef struct D3D12_VIEW_INSTANCING_DESC
{
    UINT ViewInstanceCount;
    const D3D12_VIEW_INSTANCE_LOCATION* pViewInstanceLocations;
    D3D12_VIEW_INSTANCING_FLAGS Flags;
} D3D12_VIEW_INSTANCING_DESC;
```

The reason the view instance count is fixed in the PSO is to allow whole
pipeline optimization based on the desired view count.

The absence of a view instancing declaration or ViewInstanceCount set to
0 in a PSO means view instancing is disabled. What disabled really means
is rendering behaves the same as having ViewInstanceCount set to 1.
Shaders that input SV_ViewID will only see the value 0 and whether or
not they input SV_ViewID, only one view instance is produced. In this
way an application can author shaders that are view instancing aware
that can function even in PSOs that have view instancing disabled.
Drivers that support shader model 6.1 (which exposes SV_ViewID ) but
which do not expose support for view instancing must still support
running shaders that input SV_ViewID in PSOs that declare
ViewInstanceCount as 0 (disabled) or 1 (the same).

---

`ViewportArrayIndex`

The index of the viewport in the viewports array to be used by the view instance associated with this location.

`RenderTargetArrayIndex`

The render-target array-slice index used by this view instance; this is not an output-merger render-target slot number.

`ViewInstanceCount`

Specifies the number of views to be used, up to D3D12\_MAX\_VIEW\_INSTANCE\_COUNT.

`pViewInstanceLocations`

The address of a memory location that contains **ViewInstanceCount** view instance location structures that specify the location of viewport/scissor and render target details of each view instance.

`Flags`

Configures view instancing with additional options.

`ViewInstanceCount` cannot exceed `D3D12_MAX_VIEW_INSTANCE_COUNT` (4). `D3D12_FEATURE_DATA_D3D12_OPTIONS3.ViewInstancingTier` reports the supported tier. View locations and masking are defined by [view instancing](../ViewInstancing.md#view-instance-locations); the command-list [view mask](command-list-state.md#view-mask) is not stored as a mutable PSO field.

A view count greater than one or `D3D12_VIEW_INSTANCING_FLAG_ENABLE_VIEW_INSTANCE_MASKING` requires `ViewInstancingTier` other than `D3D12_VIEW_INSTANCING_TIER_NOT_SUPPORTED`. A count of one additionally requires device support for Shader Model 6.1 or higher, even if the shaders do not consume `SV_ViewID`. A count of zero retains the disabled behavior without imposing that creation requirement; shaders that use `SV_ViewID` still require their shader-model support. Supported tiers do not introduce an additional prohibition on shader output of viewport/render-target array indices. [View-dependent stage storage and work ordering](vertex-processing.md#geometry-view-instancing) define the interstage execution contract.

<a id="apis"></a>

## Creation and selection APIs

<a id="creation-validation"></a>

### Common creation validation and results

Creation descriptions, their nested arrays/strings, shader byte buffers and cache input must be valid while the creation call consumes them. The created PSO maintains its internal shader/state/root-signature dependencies; shader bytecode and inline state descriptions are input data, not separately bound state objects. Repeated creation is not a guarantee of object identity or absence of driver work. [Recording and execution lifetime](execution.md#lifetime) remains a separate application contract.

For successful validation-only creation, a null `ppPipelineState` returns `S_FALSE` rather than a pipeline object. This is not an allocation-free or driver-call-free probe: runtime/shader/state validation can create internal subobjects, and it does not exercise final driver PSO creation.

Successful object creation returns `S_OK`. Failed creation clears a supplied output pointer. Invalid descriptions/program combinations can return `E_INVALIDARG`; allocation failure can return `E_OUTOFMEMORY`, and driver failures can propagate. Cached-data compatibility has the additional [named error conditions](#cached-data).

<a id="id3d12device-creategraphicspipelinestate"></a>

### ID3D12Device::CreateGraphicsPipelineState

#### Declaration

```cpp
HRESULT CreateGraphicsPipelineState(
    const D3D12_GRAPHICS_PIPELINE_STATE_DESC* pDesc,
    REFIID riid,
    void** ppPipelineState
    );
```

#### Behavior

Creates a graphics pipeline state object.

Uses the [graphics description](#graphics-description) and [compatibility rules](#compatibility). The resulting object is an `ID3D12PipelineState`.

The driver can use [runtime compiled-value lookup](runtime-cache-discovery.md#compiled-value-lookup) while processing this ordinary creation description.

#### Parameters

`[in] pDesc`

A pointer to a [`D3D12_GRAPHICS_PIPELINE_STATE_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_graphics_pipeline_state_desc) structure that describes graphics pipeline state.

`riid`

The globally unique identifier (**GUID**) for the pipeline state interface ([ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate)). The **REFIID**, or **GUID**, of the interface to the pipeline state can be obtained by using the \_\_uuidof() macro. For example, \_\_uuidof(ID3D12PipelineState) will get the **GUID** of the interface to a pipeline state.

`[out] ppPipelineState`

A pointer to a memory block that receives a pointer to the [ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) interface for the pipeline state object. The pipeline state object is an immutable state object.

#### Return values

This method returns **E\_OUTOFMEMORY** if there is insufficient memory to create the pipeline state object. See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

<a id="id3d12device-createcomputepipelinestate"></a>

### ID3D12Device::CreateComputePipelineState

#### Declaration

```cpp
HRESULT CreateComputePipelineState(
    const D3D12_COMPUTE_PIPELINE_STATE_DESC* pDesc,
    REFIID riid,
    void** ppPipelineState
    );
```

#### Behavior

Creates a compute pipeline state object.

Uses the [compute description](#compute-description). The compute shader and root signature must be compatible.

The driver can use [runtime compiled-value lookup](runtime-cache-discovery.md#compiled-value-lookup) while processing this ordinary creation description.

#### Parameters

`[in] pDesc`

A pointer to a [`D3D12_COMPUTE_PIPELINE_STATE_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_compute_pipeline_state_desc) structure that describes compute pipeline state.

`riid`

The globally unique identifier (**GUID**) for the pipeline state interface ([ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate)). The **REFIID**, or **GUID**, of the interface to the pipeline state can be obtained by using the \_\_uuidof() macro. For example, \_\_uuidof(ID3D12PipelineState) will get the **GUID** of the interface to a pipeline state.

`[out] ppPipelineState`

A pointer to a memory block that receives a pointer to the [ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) interface for the pipeline state object. The pipeline state object is an immutable state object.

#### Return values

This method returns **E\_OUTOFMEMORY** if there is insufficient memory to create the pipeline state object. See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

<a id="id3d12device2-createpipelinestate"></a>

### ID3D12Device2::CreatePipelineState

#### Declaration

```cpp
HRESULT CreatePipelineState(
    const D3D12_PIPELINE_STATE_STREAM_DESC* pDesc,
    REFIID riid,
    void** ppPipelineState
    );
```

#### Behavior

Creates a pipeline state object from a pipeline state stream description.

Uses the [tagged stream layout and pipeline selection rules](#streams). Payload representation, padding and relevant creation conditions remain part of this contract.

The driver can use [runtime compiled-value lookup](runtime-cache-discovery.md#compiled-value-lookup) while processing this ordinary creation description.

#### Parameters

`pDesc`

The address of a [`D3D12_PIPELINE_STATE_STREAM_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc) structure that describes the pipeline state.

`riid`

The globally unique identifier (**GUID**) for the pipeline state interface ([ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12pipelinestate)).

The **REFIID**, or **GUID**, of the interface to the pipeline state can be obtained by using the \_\_uuidof() macro. For example, \_\_uuidof(ID3D12PipelineState) will get the **GUID** of the interface to a pipeline state.

`[out] ppPipelineState`

A pointer to a memory block that receives a pointer to the [ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12pipelinestate) interface for the pipeline state object.

The pipeline state object is an immutable state object.

#### Return values

This method returns **E\_OUTOFMEMORY** if there is insufficient memory to create the pipeline state object. See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

<a id="id3d12pipelinestate-getcachedblob"></a>

### Legacy cached-blob retrieval

The full [ID3D12PipelineState::GetCachedBlob contract](shader-delivery-and-caching.md#id3d12pipelinestate-getcachedblob) defines the supported legacy retrieval method, output ownership and results. The [cached-data contract](#cached-data) below retains cached-PSO input and creation requirements.

<a id="id3d12graphicscommandlist-setpipelinestate"></a>

### ID3D12GraphicsCommandList::SetPipelineState

#### Declaration

```cpp
void SetPipelineState(
    ID3D12PipelineState* pPipelineState
    );
```

#### Behavior

Sets all shaders and programs most of the fixed-function state of the graphics processing unit (GPU) pipeline.

Selects an ordinary graphics or compute PSO. [Initial/reset and bundle effects](command-list-state.md#pipeline-selection), [root-signature independence](binding.md#pipeline-compatibility) and [dynamic override reselection](command-list-state.md#dynamic-application-state) apply.

#### Parameters

`[in] pPipelineState`

Pointer to the [ID3D12PipelineState](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) containing the pipeline state data.

#### Return values

None

#### Recording preconditions and lifetime

`pPipelineState` must be a non-null compatible PSO for the device and command-list node; this method has no null-unbind operation. It is valid on DIRECT, COMPUTE and BUNDLE recordings, not COPY. Selection itself is not a compute-only PSO-kind restriction on COMPUTE recordings. The subsequently issued work still requires a compatible selected program and a recording type supporting that work. The [recording/execution lifetime rules](execution.md#lifetime) apply to this API's recorded PSO reference; debug tracking is not a promise that the command list owns a COM reference.

<a id="compatibility"></a>

## Compatibility and validation

The runtime validates:

- Whether the linkage between the shader stages is correct.
- If the **HS** and **DS** members are specified, the **PrimitiveTopologyType** member for topology type must be set to [`D3D12_PRIMITIVE_TOPOLOGY_TYPE_PATCH`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_primitive_topology_type).
- Whether sample frequency execution isn't allowed with the center multi-sample anti-aliasing (MSAA) pattern.
- Whether anti-aliasing lines aren't allowed with the center MSAA pattern.
- If the **ForcedSampleCount** member of [`D3D12_RASTERIZER_DESC`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_rasterizer_desc) that **RasterizerState** specifies isn't zero:
    - Depth/stencil must be disabled.
    - Pixel shader can't output depth.
    - Pixel shader can't run at sample frequency.
    - For ForcedSampleCount greater than one, render target sample count must be 1.
- Whether blend state is compatible with render target formats.
- Whether pixel shader output type is compatible with render target format.
- Whether the sample count and quality are supported for the render target/depth stencil formats.

Traditional graphics creation requires a vertex shader; compute creation requires a compute shader. Hull and domain shaders occur together and use patch topology. The hull shader's output control-point count and tessellator domain must match the domain shader's declarations; both their control-point and patch-constant signatures must match. A geometry shader's input primitive must agree with the preceding topology or tessellator output. Other stage linkages check consumed inputs and their compatible semantics/types/components, not identical complete signatures; that subset rule does not relax the HS/DS interfaces. [Vertex processing and tessellation](vertex-processing.md#vertex-processing) define the stage execution, factor and patch-identity contracts.

Active shader stages within one graphics PSO must not mix DXBC and DXIL bytecode. This is a consistency requirement on the bytecode family, not a requirement that every stage use the identical shader-model number. Mixing those families is invalid and creation returns `E_INVALIDARG`.

<a id="no-output-samples"></a>

### Sample configuration without output attachments

When `NumRenderTargets == 0` and `DSVFormat == DXGI_FORMAT_UNKNOWN`, the valid multisample counts are reported by `D3D12_FEATURE_DATA_D3D12_OPTIONS19.SupportedSampleCountsWithNoOutputs`, not a format-specific quality query. Bit n represents sample count `2^n`; a value of 1 reports single-sample-only support. Counts greater than one require their reported bit. Rasterization defines the [no-output sample-frequency path and mandatory capability floor](rasterization.md#no-output-samples).

D3D12 allows `DXGI_SAMPLE_DESC::Count` to be > 1 when no render targets or depth stencil are bound. In this case, rasterization behaves as with setting `ForcedSampleCount` > 1, except that sample-frequency pixel shader execution is allowed and well-defined. `DXGI_SAMPLE_DESC::Quality` must be 0 or `D3D12_STANDARD_MULTISAMPLE_PATTERN`. When programmable sample positions are set on the command list, those sample positions are used by this type of rendering as well.

Existing interactions with `ForcedSampleCount` remain:
* If `DXGI_SAMPLE_DESC::Count` > 1, `ForcedSampleCount` must be 0 or 1. 0 indicates multisample rasterization, while 1 indicates single-sample rasterization. If there are render targets or depth stencil bound, the results of rasterization and pixel shading are broadcast to all samples for the pixel.
* If `DXGI_SAMPLE_DESC::Count` == 1, `ForcedSampleCount` may be larger, but the existing target-independent rasterization restrictions remain.

<a id="execution-compatibility"></a>

### Execution compatibility

Graphics, compute and mesh shader sets are not additive pipeline bindings. The selected program's root signature must satisfy [shader/root-signature compatibility](binding.md#pipeline-compatibility), and the selected PSO and command-list root signature must match when work is executed. Attachment descriptions require [format/sample support](formats.md#support) and compatible active views; passing one creation check does not waive descriptor, resource-lifetime or synchronization requirements.

[Indirect work types](indirect-execution.md#work-types) select the graphics or compute root-argument bind point and retain their workload's pipeline requirements. [Command-signature root matching](indirect-execution.md#execute-debug) is separate from the pipeline/root compatibility rules here.

<a id="cached-data"></a>

## Cached pipeline data

Per-PSO cached blobs and pipeline libraries are supported legacy caching mechanisms. Their complete [retrieval, storage and serialization contracts](shader-delivery-and-caching.md) remain distinct from the PSO creation inputs described here. Explicit offline driver/pipeline compilation and driver-cache control are the forward-facing directions; this classification does not remove or change the legacy callable contracts.

`D3D12_CACHED_PIPELINE_STATE` is filled with the data retrieved from [ID3D12PipelineState::GetCachedBlob](shader-delivery-and-caching.md#id3d12pipelinestate-getcachedblob). This cached PSO contains data specific to the hardware, driver, and machine that it was retrieved from. The cached bytes are an optimization input; the complete creation description is still required. The rest of the data in the PSO needs to still be valid, and needs to match the cached PSO, otherwise E\_INVALIDARG might be returned.

Driver/runtime version incompatibility can be reported as `D3D12_ERROR_DRIVER_VERSION_MISMATCH`; a different adapter can be reported as `D3D12_ERROR_ADAPTER_NOT_FOUND`.

`CachedPSO.pCachedBlob == nullptr` with `CachedBlobSizeInBytes == 0` supplies no cache input. A cached blob is not a stand-alone serialized replacement for the shader/state description or a resource/descriptor binding. [Shader cache and pipeline-library management](shader-delivery-and-caching.md) has a separate callable/lifetime contract.

<a id="driver"></a>

## Driver responsibilities

Creation resolves shader bytecode, root signatures and fixed-function descriptions into the driver objects and aggregate required by the negotiated DDI. Runtime validation and shader compilation are distinct from debug diagnostics. A driver-facing aggregate can carry both compute and graphics handle fields even though the application creates one compatible pipeline kind at a time.

The 0099 pipeline creation payload and its private-size/create callbacks are:
```c++
typedef struct D3D12DDIARG_CREATE_PIPELINE_STATE_0099
{
    D3D12DDI_HSHADER hComputeShader;
    D3D12DDI_HSHADER hVertexShader;
    D3D12DDI_HSHADER hPixelShader;
    D3D12DDI_HSHADER hDomainShader;
    D3D12DDI_HSHADER hHullShader;
    D3D12DDI_HSHADER hGeometryShader;
    D3D12DDI_HROOTSIGNATURE hRootSignature;
    D3D12DDI_HBLENDSTATE hBlendState;
    UINT SampleMask;
    D3D12DDI_HRASTERIZERSTATE hRasterizerState;
    D3D12DDI_HDEPTHSTENCILSTATE hDepthStencilState;
    D3D12DDI_HELEMENTLAYOUT hElementLayout;
    D3D12DDI_INDEX_BUFFER_STRIP_CUT_VALUE IBStripCutValue;
    D3D12DDI_PRIMITIVE_TOPOLOGY_TYPE PrimitiveTopologyType;
    UINT NumRenderTargets;
    DXGI_FORMAT RTVFormats[8];
    DXGI_FORMAT DSVFormat;
    DXGI_SAMPLE_DESC SampleDesc;
    UINT NodeMask;
    D3D12DDI_LIBRARY_REFERENCE_0010 LibraryReference;
    D3D12DDI_VIEW_INSTANCING_DESC ViewInstancingDesc;
    D3D12DDI_HSHADER hMeshShader;
    D3D12DDI_HSHADER hAmplificationShader;

    /// Pipeline state flags
    D3D12DDI_PIPELINE_STATE_FLAGS Flags;

} D3D12DDIARG_CREATE_PIPELINE_STATE_0099;

typedef HRESULT(APIENTRY* PFND3D12DDI_CREATE_PIPELINE_STATE_0099) (D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_PIPELINE_STATE_0099*, D3D12DDI_HPIPELINESTATE, D3D12DDI_HRTPIPELINESTATE);
typedef SIZE_T(APIENTRY* PFND3D12DDI_CALC_PRIVATE_PIPELINE_STATE_SIZE_0099)(D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_PIPELINE_STATE_0099*);

```

For drivers, the dynamic depth-bias and strip-cut creation flags are only _hints_ about whether or not state is allowed to be changed; it does not alter the behavior of calls to `pfnSetPipelineState`. When setting the pipeline state, the depth bias state and IB strip cut must still be set according to their descriptions in the pipeline state object, regardless of the presence of their corresponding dynamic flags.

Support for dynamic depth bias and index buffer strip cut is implied by, and must be supported on, drivers that report their DDI version to be 0099 or greater.
