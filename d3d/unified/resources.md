# Resources and memory

| Subject | Sections |
|---|---|
| Resource and heap model | [Identity](#resource-storage), [subresources](#subresources), [storage and ownership](#storage-models) |
| Descriptions and properties | [Resources](#resource-description), [heaps](#heap-properties), [heap flags](#heap-flags) |
| Explicit memory management | [Residency and budgets](#residency), [CPU mapping](#cpu-access) |
| Allocation and creation | [Allocation information](#allocation-info), [tight alignment](#tight-alignment), [creation methods](#creation-contracts) |
| Driver and hardware contract | [Allocation and residency](#allocation-driver), [memory access](#hardware-memory-access) |

<a id="resource-storage"></a>

## Resource identity and storage

Resources are the D3D concept which abstracts the usage of GPU physical memory. Resources require GPU virtual address (VA) space to access physical memory.

Resources are associated with virtual memory functionality, while heaps are associated with physical memory functionality. These associations are honored more at the DDI, to encourage driver simplification and ease DDI evolution. But, the associations are more loosely honored at the API, to simplify application concepts and avoid leveraging the available orthogonality implied by the DDI design.

Physical-address (PA) control does not mean that a game can reference actual physical addresses directly. For the most part, it refers to the fact that the game has control over how much physical memory is available (assuming it is under budget) and how it has the ability to explicitly map GPU VA (in the form of DX12 resources) to it.

The concept of "Heaps" in "Resource Heaps" is not to be confused with the other DX12 concepts of "Heaps" in the "Descriptor Heaps" or "Query Heaps" features.

Resources are aligned closer to the notion of a GPU virtual address range than physical video memory; but there is some flexibility for the driver. It can choose whether each resource typically reuses both virtual address range and physical memory, or if each resource has its own virtual address range mapped to the same physical memory.

<a id="resource-dimensions"></a>

## Resource dimensions

```cpp
typedef enum D3D12_RESOURCE_DIMENSION
{
    D3D12_RESOURCE_DIMENSION_UNKNOWN = 0,
    D3D12_RESOURCE_DIMENSION_BUFFER = 1,
    D3D12_RESOURCE_DIMENSION_TEXTURE1D = 2,
    D3D12_RESOURCE_DIMENSION_TEXTURE2D = 3,
    D3D12_RESOURCE_DIMENSION_TEXTURE3D = 4,
} D3D12_RESOURCE_DIMENSION;
```

### Buffers

Buffers are a contiguous memory region. Exhaustion of GPU virtual address space, memory residency budget (see [IDXGIAdapter3::QueryVideoMemoryInfo](https://learn.microsoft.com/en-us/windows/desktop/api/dxgi1_4/nf-dxgi1_4-idxgiadapter3-queryvideomemoryinfo)), and/or system memory may occur.

For buffers, `D3D12_RESOURCE_DESC::Width` is a nonzero size in bytes. For textures it is a size in texels; block-compressed texture dimensions are logical texel dimensions rather than compressed-block counts. GPU virtual-address capability fields report address-bit counts, not byte sizes or guarantees that an allocation will succeed.

### Texture1D

A Texture1D is a homogeneous array of 1D Textures. The array is homogeneous
in the sense that each Texture has the same data format and dimensions
(including miplevels). The entire array of Textures are created
atomically. The memory for the entire Resource need not be contiguous.

### Texture2D

A Texture2D is a homogeneous array of 2D Textures. The array is homogeneous
in the sense that each Texture has the same data format and dimensions
(including miplevels). The entire array of Textures are created
atomically. The memory for the entire Resource need not be contiguous.

### Texture3D

A Texture3D is a 3D grid data layout, supporting mipmaps; and is also
known as a Volume Texture. The entire Resource is created
atomically. The memory for the entire Resource need not be contiguous.

<a id="subresources"></a>

## Subresources

If a resource contains a buffer, then it simply contains one subresource with an index of 0. If the resource contains a texture (or texture array), then referencing the subresources is more complex.

A 3D texture mip level contains all depth slices of that mip; the depth slices are not array slices and do not have separate subresource indices.

Mip level 0 is the most detailed level. Each subsequent level halves each spatial dimension, rounding down, with a minimum of one texel in each dimension. Array-element counts do not shrink with mip level. The resource can contain fewer levels than the full chain.

### Mip slices

A mip slice includes one mipmap level for every texture in an array, as shown in the following image.

![subresource mip slices](https://learn.microsoft.com/en-us/windows/win32/direct3d12/images/subresource-mip-slice.png)

### Array slices

Given an array of textures, each texture with mipmaps, an array slice includes one texture and all of its mip levels, as shown in the following image.

![subresource array slices](https://learn.microsoft.com/en-us/windows/win32/direct3d12/images/subresource-array-slices.png)

### Plane slices

Typically planar formats are not used to store RGBA data, but in the cases where it is (perhaps 24bpp RGB data), one plane could represent the red image, one the green, and one the blue image. One plane though is not necessarily one color, two or more colors could be combined into one plane. More typically planar data is used for sub-sampled YCbCr and Depth-Stencil data. Depth-stencil formats can have separate depth and stencil planes.

### Subresource indexing

```cpp
UINT D3D12CalcSubresource(
    UINT MipSlice, UINT ArraySlice, UINT PlaneSlice, UINT MipLevels, UINT ArraySize
    )
{ return MipSlice + ArraySlice * MipLevels + PlaneSlice * MipLevels * ArraySize; }
```

`MipSlice`, `ArraySlice`, and `PlaneSlice` are zero-based indices. `MipLevels` and `ArraySize` are the resource's mip-level and array-element counts; `ArraySize` is 1 for a 3D texture. Mip indices vary fastest, followed by array indices and then plane indices. This is a logical indexing order, not a portable byte-offset formula for opaque texture storage.

<a id="storage-models"></a>

## Storage models

<a id="committed-resources"></a>

### Committed resources

Creating such a resource allocates virtual address range, an implicit heap large enough to fit the whole resource, and commits the virtual address range to the physical memory encapsulated by the heap.

<a id="placed-resources"></a>

### Placed resources

Applications may create heaps separate from resources. Afterward, the application may locate multiple resources within a single heap. This can be done without creating tiled or reserved resources, enabling the capabilities for all resource types able to be created directly by applications. Multiple resources may overlap; their reuse is subject to the applicable aliasing and initialization requirements.

Placement requirements are defined by [`CreatePlacedResource`](resources.md#create-placed-resource). Overlapping allocations do not by themselves establish synchronization or initialize texture metadata; the [legacy and enhanced metadata-initialization requirements](synchronization.md#enhanced-aliasing) apply.

<a id="reserved-resources"></a>

### Reserved resources

Shader [mapping-status feedback](common-shader-internals.md#mapping-feedback) and [sampler-feedback usage recording](common-shader-internals.md#sampler-feedback) have distinct meanings. Neither maps tiles, establishes residency nor synchronizes a mapping change.

On their creation, only a virtual address range is allocated and not mapped to any heap. The application will map such resources to heaps later. Such resources can be mapped to a heap at a 64KB tile granularity with `UpdateTileMappings`.

<a id="arrayed-packed-mips"></a>

#### Arrayed packed mips and Tier 4

Tiers 2 and 3 support arrayed, mipped tiled resources, but not resources combining more than one array slice with any mipmap having a dimension smaller than the corresponding standard tile extent for its format. Such small levels require packed mips.

Tiled resources Tier 4 includes Tier 3 behavior and permits `64KB_UNDEFINED_SWIZZLE` texture arrays with a full mip chain, including packed levels. This contribution does not apply to `64KB_STANDARD_SWIZZLE`.

Existing resource creation, `GetResourceTiling` and tile-mapping APIs operate on these resources without new method signatures. `D3D12_TILED_RESOURCES_TIER_4` reports support. Drivers report the tier through `D3D12DDI_TILED_RESOURCES_TIER_0117_4` starting at DDI version 0117; reporting the tier requires supporting and testing this resource shape.

The creation/runtime/debug-layer restrictions on arrayed packed-mip resources are relaxed only when the applicable Tier 4 support is exposed. The ordinary layout, resource description and mapping/synchronization requirements still apply.

<a id="resource-heap-allocation"></a>

### Resource and heap allocation

For resource backing in D3D12, the way to allocate physical video memory is by creating a heap. This can be done in one of two ways: implicitly by using `CreateCommittedResource`, or explicitly using `CreateHeap`.

`CreateHeap` creates backing storage without creating a valid D3D resource. In order to use that newly created heap memory, an application is then expected to create a placed resource within that heap or map a reserved resource to that heap.

For typical scenarios, in order to make a resource available to the GPU for rendering, both a resource and heap memory backing that resource need to exist; the resource for binding to the GPU and the heap to contain the actual data.

| Method | Creates resource | Creates heap |
| --- | --- | --- |
| `CreateHeap` | No | Yes |
| `CreatePlacedResource` | Yes | No |
| `CreateReservedResource`\* | Yes | No |
| `CreateCommittedResource` | Yes | Yes |

Table: Basic resource/heap creation method comparison.
\*While reserved resources are similar to placed resources, they can be created stand-alone without a pre-created heap and do not automatically get mapped to heap memory upon creation.

A heap can contain multiple placed resources. However, they do not provide the flexibility for the converse. Unlike reserved resources, placed resources cannot be used to access data in multiple heaps. Meaning, placed resources represent a one-heap-to-many-resources relationship, unlike the reserved resource's many-to-many relationship.

A reserved resource describes a region of VA — for example, a texture mip chain — while heaps describe regions of PA. Multiple heaps can back different regions of the same reserved resource, and any region of the reserved resource may be unmapped (i.e. not backed by any heap) at any given time. This many-to-many relationship between resources (VA) and heaps (PA) is the basis for fine-grained, runtime memory management.

<a id="resource-heap-ownership"></a>

### Ownership, destruction and reuse

As an example, if a game were to create a heap (PA) and then create a placed resource (VA) within that heap, simply destroying the placed resource will not lower a game's overall memory footprint; the heap itself must be destroyed to return memory back to the system.

A heap backing a resource (or sub-resource) that is no longer needed doesn't necessarily need to be destroyed; it could, for example, be reused for a higher-priority allocation instead, to remove the cost of recreating the heap later on.

Compression metadata is used by some hardware to provide performance benefits when accessing certain resources (e.g. fast clears, etc.). This typically comes at the cost of using slightly more memory. The amount of memory required is dependent on the hardware and implementation of specific optimizations.

Without going into too much detail, when using committed and placed resources, the amount of compression metadata required is known when the resource is created. A committed resource will create a heap behind the app's back with the requested size plus the amount of space required for the metadata. For placed resources, the game has the ability to determine how much space it needs including the compression metadata before the game needs to place it into an actual heap.

In the case of reserved resources however, since there is no heap space actually associated with the resource during resource creation, a driver may decide to allocate space elsewhere for the compression metadata. Although the space required for this metadata is not directly known by the game, the developer can use the budget querying APIs to check their before and after memory footprint.

It's also important to note that currently, in DX12, creating a new heap (even if it does not contain a resource) can allocate VA. For example, creating a reserved resource of 32MiB and a heap of 32MiB can consume 64MiB of virtual address space.

<a id="resource-binding-declarations"></a>

### Resource creation and pipeline bindings

`VERTEX_BUFFER`, `INDEX_BUFFER`, `STREAM_OUTPUT`, and `CONSTANT_BUFFER` bind flags are no longer declared at resource creation in D3D12. Instead, D3D12 buffers can be used by the GPU for all these purposes without declaration.

`DRAWINDIRECT_ARGS` is no longer required to be declared during resource creation in D3D12. Instead, D3D12 buffers can be used by the GPU for this purpose without declaration.

`ALLOW_RAW_VIEWS` and `BUFFER_STRUCTURED` are no longer required to be declared in D3D12. Instead, D3D12 buffers can be used as RAW Buffer SRVs without such up-front declaration. The structured buffer stride and the flags related to these features are specified by the view descriptor.

<a id="resource-contracts"></a>

## Descriptions, creation and synchronization

[Resource descriptions](#resource-description), [heap properties](#heap-properties), [CPU mapping](#cpu-access), [residency](#residency), and [resource creation](#creation-contracts) are defined below.

A resource's storage layout is distinct from its current legacy state or enhanced barrier layout. [Synchronization and barriers](synchronization.md) owns promotion/decay; enhanced access/layout requirements are defined in [Enhanced Barriers](synchronization.md#layout-transitions).

<a id="resource-description"></a>

## D3D12_RESOURCE_DESC

```cpp
typedef struct D3D12_RESOURCE_DESC
{
    D3D12_RESOURCE_DIMENSION Dimension;
    UINT64 Alignment;
    UINT64 Width;
    UINT Height;
    UINT16 DepthOrArraySize;
    UINT16 MipLevels;
    DXGI_FORMAT Format;
    DXGI_SAMPLE_DESC SampleDesc;
    D3D12_TEXTURE_LAYOUT Layout;
    D3D12_RESOURCE_FLAGS Flags;
} D3D12_RESOURCE_DESC;
```

### `Dimension`

One member of [`D3D12_RESOURCE_DIMENSION`](resources.md#resource-dimensions), specifying the dimensions of the resource (for example, `D3D12_RESOURCE_DIMENSION_TEXTURE1D`), or whether it is a buffer (`D3D12_RESOURCE_DIMENSION_BUFFER`).

### `Alignment`

Specifies the alignment.

### `Width`

Specifies the buffer size in bytes or the texture width in texels. The value must be nonzero.

### `Height`

Specifies the height of the resource.

### `DepthOrArraySize`

Specifies the depth of the resource, if it is 3D, or the array size if it is an array of 1D or 2D resources.

### `MipLevels`

Specifies the number of MIP levels.

### `Format`

Specifies one member of [`DXGI_FORMAT`](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format).

### `SampleDesc`

Specifies a [`DXGI_SAMPLE_DESC`](https://learn.microsoft.com/en-us/windows/desktop/api/dxgicommon/ns-dxgicommon-dxgi_sample_desc) structure.

### `Layout`

Specifies one member of [`D3D12_TEXTURE_LAYOUT`](resources.md#texture-layout).

### `Flags`

Bitwise-OR'd flags, as [`D3D12_RESOURCE_FLAGS`](resources.md#resource-flags) enumeration constants.

<a id="buffer-description"></a>

### Buffer descriptions

*Height*, *DepthOrArraySize*, and *MipLevels* must be 1.

*Format* must be `DXGI_FORMAT_UNKNOWN`.

*SampleDesc.Count* must be 1 and *Quality* must be 0.

*Layout* must be `D3D12_TEXTURE_LAYOUT_ROW_MAJOR`, as buffer memory layouts are understood by applications and row-major texture data is commonly marshaled through buffers.

*Flags* must still be accurately filled out by applications for buffers, with minor exceptions. However, applications can use the most amount of capability support without concern about the efficiency impact on buffers. The flags field is meant to control properties related to textures.

Without `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT`, *Alignment* must be 64KB (`D3D12_DEFAULT_RESOURCE_PLACEMENT_ALIGNMENT`) or 0, which is effectively 64KB.

<a id="texture-description"></a>

### Texture descriptions

Textures are a multi-dimensional arrangement of texels in a contiguous region of memory, heavily optimized to maximize bandwidth for rendering and sampling. Texture sizes are hard to predict and vary from adapter to adapter. Applications must use [ID3D12Device::GetResourceAllocationInfo](resources.md#allocation-info) to accurately understand their size.
TEXTURE1D, TEXTURE2D, and TEXTURE3D are not supported orthogonally on every format. See the use of `D3D12_FORMAT_SUPPORT1_TEXTURE1D`, `D3D12_FORMAT_SUPPORT1_TEXTURE2D`, and `D3D12_FORMAT_SUPPORT1_TEXTURE3D` in [`D3D12_FORMAT_SUPPORT1`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_format_support1).

*Width*, *Height*, and *DepthOrArraySize* must be between 1 and the maximum dimension supported for the particular feature level and texture dimension. However, exhaustion of GPU virtual address space, memory residency budget (see [IDXGIAdapter3::QueryVideoMemoryInfo](https://learn.microsoft.com/en-us/windows/desktop/api/dxgi1_4/nf-dxgi1_4-idxgiadapter3-queryvideomemoryinfo)), and/or system memory may easily occur first. For compressed formats, these dimensions are logical. For example:

- For TEXTURE1D:
    - *Width* must be less than or equal to `D3D12_REQ_TEXTURE1D_U_DIMENSION`.
    - *Height* must be 1.
    - *DepthOrArraySize* is interpreted as array size and must be less than or equal to `D3D12_REQ_TEXTURE1D_ARRAY_AXIS_DIMENSION`.
- For TEXTURE2D:
    - *Width* and *Height* must be less than or equal to `D3D12_REQ_TEXTURE2D_U_OR_V_DIMENSION`.
    - *DepthOrArraySize* is interpreted as array size and must be less than or equal to `D3D12_REQ_TEXTURE2D_ARRAY_AXIS_DIMENSION`.
- For TEXTURE3D:
    - *Width* and *Height* and *DepthOrArraySize* must be less than or equal to `D3D12_REQ_TEXTURE3D_U_V_OR_W_DIMENSION`.
    - *DepthOrArraySize* is interpreted as depth.

### Texture dimension limits

```cpp
const UINT D3D12_REQ_TEXTURE1D_ARRAY_AXIS_DIMENSION = 2048;
const UINT D3D12_REQ_TEXTURE1D_U_DIMENSION = 16384;
const UINT D3D12_REQ_TEXTURE2D_ARRAY_AXIS_DIMENSION = 2048;
const UINT D3D12_REQ_TEXTURE2D_U_OR_V_DIMENSION = 16384;
const UINT D3D12_REQ_TEXTURE3D_U_V_OR_W_DIMENSION = 2048;
```

### Texture alignment

Without `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT`, *Alignment* may be one of 0, 4KB, 64KB or 4MB.
If *Alignment* is set to 0, the runtime will use 4MB for MSAA textures and 64KB for everything else. The application may choose smaller alignments than these defaults for a couple of texture types when the texture is small. Textures with UNKNOWN layout and MSAA may be created with 64KB alignment (if they pass the small size restriction detailed below).

Textures with UNKNOWN layout without MSAA and without render-target nor depth-stencil flags may be created with 4KB Alignment (again, passing the small size restriction).

Applications can create smaller aligned resources when the estimated size of the most-detailed mip level is a total of the larger alignment restriction or less. The runtime will use an architecture-independent mechanism of size-estimation, that mimics the way standard swizzle and D3D12 tiled resources are sized. However, the tile sizes will be of the smaller alignment restriction for such calculations. Using the non-render-target and non-depth-stencil texture as an example, the runtime will assume near-equilateral tile shapes of 4KB, and calculate the number of tiles needed for the most-detailed mip level. If the number of tiles is equal to or less than 16, then the application can create a 4KB aligned resource. So, a mipped tex2d array of any array size and any number of mip levels can be 4KB, as long as the width and height are small enough for the particular format and MSAA.

Tight-alignment allocation rules and alignment floors are defined in [Tight resource alignment](resources.md#tight-alignment).

### Mip levels

*MipLevels* may be 0, or 1 to the maximum mip levels supported by the spatial dimensions (*Width*, *Height*, and, for 3D textures, *DepthOrArraySize*). When 0 is used, the API will automatically calculate the maximum mip levels supported and use that. But, some resource and heap properties preclude mip levels, so the app must specify the value as 1.
Refer to the `D3D12_FORMAT_SUPPORT1_MIP` field of [`D3D12_FORMAT_SUPPORT1`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_format_support1) for per-format restrictions. MSAA resources, textures with `D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER`, and heaps with `D3D12_HEAP_FLAG_ALLOW_DISPLAY` all require *MipLevels* to be 1.

### Format and multisampling

*Format* must be a valid format supported at the feature level of the device.

A *SampleDesc.Count* greater than 1 and/ or non-zero *Quality* are only supported for TEXTURE2D and when either `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` are set.
For these multisampled resources, the following are unsupported:

- `D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`,
- `D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS` without [writable-MSAA texture support](#writable-msaa),
- `D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS`,
- `D3D12_HEAP_FLAG_ALLOW_DISPLAY`

See [`D3D12_FEATURE_DATA_MULTISAMPLE_QUALITY_LEVELS`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_feature_data_multisample_quality_levels) for determining valid *Count* and *Quality* values.

<a id="extended-description"></a>

## D3D12_RESOURCE_DESC1

```cpp
typedef struct D3D12_MIP_REGION
{
    UINT Width;
    UINT Height;
    UINT Depth;
} D3D12_MIP_REGION;
```

```cpp
typedef struct D3D12_RESOURCE_DESC1
{
    D3D12_RESOURCE_DIMENSION Dimension;
    UINT64 Alignment;
    UINT64 Width;
    UINT Height;
    UINT16 DepthOrArraySize;
    UINT16 MipLevels;
    DXGI_FORMAT Format;
    DXGI_SAMPLE_DESC SampleDesc;
    D3D12_TEXTURE_LAYOUT Layout;
    D3D12_RESOURCE_FLAGS Flags;
    D3D12_MIP_REGION SamplerFeedbackMipRegion;
} D3D12_RESOURCE_DESC1;
```

`D3D12_RESOURCE_DESC1` retains the fields and constraints of [`D3D12_RESOURCE_DESC`](#resource-description) and adds `SamplerFeedbackMipRegion`. The `Width`, `Height`, and `Depth` of `D3D12_MIP_REGION` describe the sampler-feedback mip region, not the resource's dimensions.

Each of the Width and Height fields of a 2D sampler-feedback mip region is
* a power-of-two number
* greater than or equal to 4
* less than or equal to half of the dimension of the most detailed mip of the paired texture.

For 2D sampler-feedback resources, applications always set `Depth` to 0, or equivalently to 1 when sampler feedback is used.

For a non-feedback texture, `SamplerFeedbackMipRegion` is zero. Creating opaque sampler-feedback maps requires `D3D12_FEATURE_DATA_D3D12_OPTIONS7::SamplerFeedbackTier` other than `D3D12_SAMPLER_FEEDBACK_TIER_NOT_SUPPORTED`.

Feedback maps can be committed or placed resources.

Feedback maps cannot themselves be tiled (reserved) resources.

Their dimension is `D3D12_RESOURCE_DIMENSION_TEXTURE2D`.

Their width and height come from the paired resource.

They have a mip count which matches the resource with which they are paired.

They have an array size which matches the resource with which they are paired.

Their format is `DXGI_FORMAT_SAMPLER_FEEDBACK_MIN_MIP_OPAQUE` or `DXGI_FORMAT_SAMPLER_FEEDBACK_MIP_REGION_USED_OPAQUE`.

They are single-sampled; that is to say, they have sample count 1. They have sample quality 0.

Their layout is `D3D12_TEXTURE_LAYOUT_UNKNOWN`. The hardware implementation decides their layout.

They are created with the flag `D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS`.

The [mip-region constraints](../SamplerFeedback.md#mip-region-constraints) and [feedback-map creation contract](../SamplerFeedback.md#resource-properties) define the feature-specific dimensions and format restrictions. These requirements apply to opaque sampler-feedback maps, not ordinary textures.

<a id="resource-flags"></a>

## Resource usage flags

```cpp
typedef enum D3D12_RESOURCE_FLAGS
{
    D3D12_RESOURCE_FLAG_NONE = 0x0,
    D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET = 0x1,
    D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL = 0x2,
    D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS = 0x4,
    D3D12_RESOURCE_FLAG_DENY_SHADER_RESOURCE = 0x8,
    D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER = 0x10,
    D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS = 0x20,
    D3D12_RESOURCE_FLAG_VIDEO_DECODE_REFERENCE_ONLY = 0x40,
    D3D12_RESOURCE_FLAG_VIDEO_ENCODE_REFERENCE_ONLY = 0x80,
    D3D12_RESOURCE_FLAG_RAYTRACING_ACCELERATION_STRUCTURE = 0x100,
    D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT = 0x400,

} D3D12_RESOURCE_FLAGS;
```

### D3D12_RESOURCE_FLAG_NONE

No options are specified.

### D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET

Allows a render target view to be created for the resource; and also enables the resource to transition into the state of [`D3D12_RESOURCE_STATE_RENDER_TARGET`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states). Some adapter architectures allocate extra memory for textures with this flag to reduce the effective bandwidth during common rendering. This characteristic may not be beneficial for textures that are never rendered to, nor is it available for textures compressed with BC formats. Your application should avoid setting this flag when rendering will never occur.

The following restrictions and interactions apply:
- Either the texture format must support render target capabilities at the current feature level. Or, when the format is a typeless format, a format within the same typeless group must support render target capabilities at the current feature level.
- Can't be set in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_ROW_MAJOR`](resources.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::CrossAdapterRowMajorTextureSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`, nor in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`](resources.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::StandardSwizzle64KBSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`.
- Can't be used with 4KB alignment without `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`, nor usage with heaps that have [`D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES`](resources.md#heap-flags).

### D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL

Allows a depth stencil view to be created for the resource, as well as enables the resource to transition into the state of [`D3D12_RESOURCE_STATE_DEPTH_WRITE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) and/or `D3D12_RESOURCE_STATE_DEPTH_READ`. Most adapter architectures allocate extra memory for textures with this flag to reduce the effective bandwidth, and maximize optimizations for early depth-test. Your application should avoid setting this flag when depth operations will never occur.

The following restrictions and interactions apply:
- Either the texture format must support depth stencil capabilities at the current feature level. Or, when the format is a typeless format, a format within the same typeless group must support depth stencil capabilities at the current feature level.
- Can't be used with [`D3D12_RESOURCE_DIMENSION_BUFFER`](resources.md#resource-dimensions), 4KB alignment without `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS`, [`D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`](resources.md#texture-layout), `D3D12_TEXTURE_LAYOUT_ROW_MAJOR`, nor used with heaps that have [`D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES`](resources.md#heap-flags) or `D3D12_HEAP_FLAG_ALLOW_DISPLAY`.
- Precludes usage of [WriteToSubresource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) and [ReadFromSubresource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource).
- Precludes GPU copying of a subregion. [CopyTextureRegion](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) must copy a whole subresource to or from resources with this flag.

### D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS

Allows an unordered access view to be created for the resource, as well as enables the resource to transition into the state of [`D3D12_RESOURCE_STATE_UNORDERED_ACCESS`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states). Some adapter architectures must resort to less efficient texture layouts in order to provide this functionality. If a texture is rarely used for unordered access, then it might be worth having two textures around and copying between them. One texture would have this flag, while the other wouldn't. Your application should avoid setting this flag when unordered access operations will never occur.

The following restrictions and interactions apply:
- Either the texture format must support unordered access capabilities at the current feature level. Or, when the format is a typeless format, a format within the same typeless group must support unordered access capabilities at the current feature level.
- Can't be set in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_ROW_MAJOR`](resources.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::CrossAdapterRowMajorTextureSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`, nor in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`](resources.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::StandardSwizzle64KBSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`, nor when the feature level is less than 11.0.
- Use with MSAA textures requires [writable-MSAA texture support](#writable-msaa).

### D3D12_RESOURCE_FLAG_DENY_SHADER_RESOURCE

Disallows a shader resource view from being created for the resource, as well as disables the resource from transitioning into the state of [`D3D12_RESOURCE_STATE_NON_PIXEL_SHADER_RESOURCE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) or `D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE`. Some adapter architectures gain bandwidth capacity for depth stencil textures when shader resource views are precluded. If a texture is rarely used for shader resources, then it might be worth having two textures around and copying between them. One texture would have this flag, while the other wouldn't. Your application should set this flag when depth stencil textures will never be used from shader resource views.

The following restrictions and interactions apply:
- Must be used with `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`.

### D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER

Allows the resource to be used for cross-adapter data, as well as those features enabled by `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS`. Cross-adapter resources commonly preclude techniques that reduce effective texture bandwidth during usage, and some adapter architectures might require different caching behavior. Your application should avoid setting this flag when the resource data will never be used with another adapter.

The following restrictions and interactions apply:
- Must be used with heaps that have [`D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER`](resources.md#heap-flags).
- Can't be used with heaps that have [`D3D12_HEAP_FLAG_ALLOW_DISPLAY`](resources.md#heap-flags).

### D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS

Allows a resource to be simultaneously accessed by multiple different queues, devices, or processes (for example, allows a resource to be used with [ResourceBarrier](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) transitions performed in more than one command list executing at the same time). Simultaneous access allows multiple readers and one writer, as long as the writer doesn't concurrently modify the texels that other readers are accessing. Some adapter architectures can't leverage techniques to reduce effective texture bandwidth during usage.

However, your application should avoid setting this flag when multiple readers are not required during frequent, non-overlapping writes to textures. Use of this flag can compromise resource fences to perform waits, and prevent any compression being used with a resource.

The following restrictions and interactions apply:
- Can't be used with [`D3D12_RESOURCE_DIMENSION_BUFFER`](resources.md#resource-dimensions); but buffers always have the properties represented by this flag.
- Can't be used with MSAA textures.
- Can't be used with `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`.

### D3D12_RESOURCE_FLAG_VIDEO_DECODE_REFERENCE_ONLY

Specifies that this resource may be used only as a decode reference frame. It may be written to or read only by the video decode operation.

[`D3D12_VIDEO_DECODE_TIER_1`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/ne-d3d12video-d3d12_video_decode_tier) and [`D3D12_VIDEO_DECODE_TIER_2`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/ne-d3d12video-d3d12_video_decode_tier) may report [`D3D12_VIDEO_DECODE_CONFIGURATION_FLAG_REFERENCE_ONLY_ALLOCATIONS_REQUIRED`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/ne-d3d12video-d3d12_video_decode_configuration_flags) in the [`D3D12_FEATURE_DATA_VIDEO_DECODE_SUPPORT`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/ns-d3d12video-d3d12_feature_data_video_decode_support) structure configuration flag. If that happens, then your application must allocate reference frames with the `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_VIDEO_DECODE_REFERENCE_ONLY` resource flag.

[`D3D12_VIDEO_DECODE_TIER_3`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/ne-d3d12video-d3d12_video_decode_tier) must not set the [`D3D12_VIDEO_DECODE_CONFIGURATION_FLAG_REFERENCE_ONLY_ALLOCATIONS_REQUIRED`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/ne-d3d12video-d3d12_video_decode_configuration_flags) configuration flag, and must not require the use of this resource flag.

### D3D12_RESOURCE_FLAG_VIDEO_ENCODE_REFERENCE_ONLY

Specifies that this resource may be used only as an encode reference frame. It may be written to or read only by the video encode operation.

### D3D12_RESOURCE_FLAG_RAYTRACING_ACCELERATION_STRUCTURE

Requires the DirectX 12 Agility SDK 1.608.0 or later. Indicates that a buffer is to be used as a raytracing acceleration structure. When using D3D12 Enhanced Barriers, this flag serves as a replacement for `D3D12_RESOURCE_STATE_RAYTRACING_ACCELERATION_STRUCTURE`, since buffers have no layout in the enhanced barrier model. Requires `D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS` to be set.

### D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT

`D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT` selects the [tight-alignment allocation contract](resources.md#tight-alignment); the device must report support before the flag can be used.

<a id="writable-msaa"></a>

### Writable multisampled textures

Support for writable MSAA textures is determined by
the `WritableMSAATexturesSupported` field of `D3D12_FEATURE_DATA_D3D12_OPTIONS14`.

When `WritableMSAATexturesSupported` is true, multisampled resources can be created with `D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS`. This does not permit combining unordered access with `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`.

<a id="texture-layout"></a>

## Texture storage layouts

```cpp
typedef enum D3D12_TEXTURE_LAYOUT
{
    D3D12_TEXTURE_LAYOUT_UNKNOWN = 0,
    D3D12_TEXTURE_LAYOUT_ROW_MAJOR = 1, // Must be used with buffers.  Can be used with cross-adapter shared textures.
    D3D12_TEXTURE_LAYOUT_64KB_UNDEFINED_SWIZZLE = 2,
    D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE = 3,
} D3D12_TEXTURE_LAYOUT;
```

### D3D12_TEXTURE_LAYOUT_UNKNOWN

Indicates that the layout is unknown, and is likely adapter-dependent.

During creation, the driver chooses the most efficient layout based on other resource properties, especially resource size and flags.

Prefer this choice unless certain functionality is required from another texture layout.

Zero-copy texture upload optimizations exist for UMA architectures; see [ID3D12Resource::WriteToSubresource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12resource-writetosubresource).

### D3D12_TEXTURE_LAYOUT_ROW_MAJOR

Indicates that data for the texture is stored in row-major order (sometimes called "pitch-linear order").

This texture layout locates consecutive texels of a row contiguously in memory, before the texels of the next row.

Similarly, consecutive texels of a particular depth or array slice are contiguous in memory before the texels of the next depth or array slice.

Padding may exist between rows and between depth or array slices to align collections of data.

A stride is the distance in memory between rows, depth, or array slices; and it includes any padding.

This texture layout enables sharing of the texture data between multiple adapters, when other layouts aren't available.

Many restrictions apply, because this layout is generally not efficient for extensive usage:
- The locality of nearby texels is not rotationally invariant.
- Only the following texture properties are supported:
    - [D3D12_RESOURCE_DIMENSION_TEXTURE2D](resources.md#resource-dimensions).
    - A single mip level.
    - A single array slice.
    - 64KB alignment.
    - Non-MSAA.
    - No [`D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`](resources.md#resource-flags).
    - The format cannot be a YUV format.
- The texture must be created on a heap with [`D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER`](resources.md#heap-flags).

Buffers are created with [`D3D12_TEXTURE_LAYOUT_ROW_MAJOR`](resources.md#texture-layout), because row-major texture data can be located in them without creating a texture object.

This is commonly used for uploading or reading back texture data, especially for discrete/NUMA adapters.

However, `D3D12_TEXTURE_LAYOUT_ROW_MAJOR` can also be used when marshaling texture data between GPUs or adapters.



### D3D12_TEXTURE_LAYOUT_64KB_UNDEFINED_SWIZZLE

Indicates that the layout within 64KB tiles and tail mip packing is up to the driver.

There is no standard swizzle pattern.

This texture layout is arranged into contiguous 64KB regions, also known as tiles, containing near equilateral amount of consecutive number of texels along each dimension.

Tiles are arranged in row-major order.

While there is no padding between tiles, there are typically unused texels within the last tile in each dimension.

The layout of texels within the tile is undefined.

Each subresource immediately follows where the previous subresource end, and the subresource order follows the same sequence as subresource ordinals.

However, tail mip packing is adapter-specific.

For more details, see tiled resource tier and [ID3D12Device::GetResourceTiling](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-getresourcetiling).

This texture layout enables partially resident or sparse texture scenarios when used together with virtual memory page mapping functionality.

This texture layout must be used together with [ID3D12Device::CreateReservedResource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-createreservedresource) to enable the usage of [ID3D12CommandQueue::UpdateTileMappings](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings).

Some restrictions apply to textures with this layout:
- The adapter must support [`D3D12_TILED_RESOURCES_TIER`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_tiled_resources_tier) 1 or greater.
- 64KB alignment must be used.
- [`D3D12_RESOURCE_DIMENSION_TEXTURE1D`](resources.md#resource-dimensions) is not supported, nor are all formats.
- The tiled resource tier indicates whether textures with [`D3D12_RESOURCE_DIMENSION_TEXTURE3D`](resources.md#resource-dimensions) is supported.

### D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE

Indicates that a default texture uses the standardized swizzle pattern.

This texture layout is arranged the same way that `D3D12_TEXTURE_LAYOUT_64KB_UNDEFINED_SWIZZLE` is, except that the layout of texels within the tile is defined. Tail mip packing is adapter-specific.

This texture layout enables optimizations when marshaling data between multiple adapters or between the CPU and GPU.

The amount of copying can be reduced when multiple components understand the texture memory layout.

This layout is generally more efficient for extensive usage than row-major layout, due to the rotationally invariant locality of neighboring texels.

This layout can typically only be used with adapters that support standard swizzle, but exceptions exist for cross-adapter shared heaps.

The restrictions for this layout are that the following aren't supported:
- [`D3D12_RESOURCE_DIMENSION_TEXTURE1D`](resources.md#resource-dimensions)
- Multi-sample anti-aliasing (MSAA)
- [`D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`](resources.md#resource-flags)
- Formats within the [`DXGI_FORMAT_R32G32B32_TYPELESS`](https://learn.microsoft.com/en-us/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format) group

<a id="optimized-clear-value"></a>

## Optimized clear value

```cpp
typedef struct D3D12_DEPTH_STENCIL_VALUE
{
    FLOAT Depth;
    UINT8 Stencil;
} D3D12_DEPTH_STENCIL_VALUE;
```

```cpp
typedef struct D3D12_CLEAR_VALUE
{
    DXGI_FORMAT Format;
    union
    {
        FLOAT Color[4];
        D3D12_DEPTH_STENCIL_VALUE DepthStencil;
    };
} D3D12_CLEAR_VALUE;
```

### `Format`

Specifies one member of the [`DXGI_FORMAT`](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) enum.

The format of the commonly cleared color follows the same validation rules as a view/ descriptor creation. In general, the format of the clear color can be any format in the same typeless group that the resource format belongs to.

This *Format* must match the format of the view used during the clear operation. It indicates whether the *Color* or the *DepthStencil* member is valid and how to convert the values for usage with the resource.

### `Color[4]`

Specifies a 4-entry array of float values, determining the RGBA value. The order of RGBA matches the order used with [ClearRenderTargetView](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-clearrendertargetview).

### `DepthStencil`

Specifies one member of [`D3D12_DEPTH_STENCIL_VALUE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ns-d3d12-d3d12_depth_stencil_value). These values match the semantics of *Depth* and *Stencil* in [ClearDepthStencilView](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12graphicscommandlist-cleardepthstencilview).

`pOptimizedClearValue` must be null for buffers and for textures without either `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`. For a non-null value, `Format` must be a fully typed, compatible render-target or depth-stencil view format, as appropriate for the resource. For a depth-stencil clear value, `DepthStencil.Depth` must lie in the inclusive range [0, 1].

An optimized clear value describes an optimization for subsequent clear commands. It is not a command to clear the resource during creation, and it does not replace required initialization of aliased texture metadata.

<a id="heap-properties"></a>

## Heap properties

```cpp
typedef struct D3D12_HEAP_PROPERTIES
{
    D3D12_HEAP_TYPE Type;
    D3D12_CPU_PAGE_PROPERTY CPUPageProperty;
    D3D12_MEMORY_POOL MemoryPoolPreference;
    UINT CreationNodeMask;
    UINT VisibleNodeMask;
} D3D12_HEAP_PROPERTIES;
```

### `Type`

A [`D3D12_HEAP_TYPE`](resources.md#heap-types)-typed value that specifies the type of heap.

### `CPUPageProperty`

A [`D3D12_CPU_PAGE_PROPERTY`](resources.md#cpu-page-property)-typed value that specifies the CPU-page properties for the heap.

### `MemoryPoolPreference`

A [`D3D12_MEMORY_POOL`](resources.md#memory-pools)-typed value that specifies the memory pool for the heap.

### `CreationNodeMask`

For multi-adapter operation, this indicates the node where the resource should be created.

Exactly one bit of this UINT must be set. See [Multi-adapter systems](https://learn.microsoft.com/en-us/windows/win32/direct3d12/multi-engine).

Passing zero is equivalent to passing one, in order to simplify the usage of single-GPU adapters.

### `VisibleNodeMask`

For multi-adapter operation, this indicates the set of nodes where the resource is visible.

*VisibleNodeMask* must have the same bit set that is set in *CreationNodeMask*. *VisibleNodeMask* can *also* have additional bits set for cross-node resources, but doing so can potentially reduce performance for resource accesses, so you should do so only when needed.

Passing zero is equivalent to passing one, in order to simplify the usage of single-GPU adapters.

Valid combinations of struct member values:
- When **Type** is [`D3D12_HEAP_TYPE_CUSTOM`](resources.md#heap-types), **CPUPageProperty** and **MemoryPoolPreference** must not be `D3D12_CPU_PAGE_PROPERTY_UNKNOWN` and `D3D12_MEMORY_POOL_UNKNOWN`, respectively.
- When **Type** is not `D3D12_HEAP_TYPE_CUSTOM`, **CPUPageProperty** and **MemoryPoolPreference** must be `D3D12_CPU_PAGE_PROPERTY_UNKNOWN` and `D3D12_MEMORY_POOL_UNKNOWN`, respectively.

<a id="cpu-page-property"></a>

### CPU page properties

```cpp
typedef enum D3D12_CPU_PAGE_PROPERTY
{
    D3D12_CPU_PAGE_PROPERTY_UNKNOWN = 0,
    D3D12_CPU_PAGE_PROPERTY_NOT_AVAILABLE = 1,
    D3D12_CPU_PAGE_PROPERTY_WRITE_COMBINE = 2,
    D3D12_CPU_PAGE_PROPERTY_WRITE_BACK = 3,
} D3D12_CPU_PAGE_PROPERTY;
```

#### D3D12_CPU_PAGE_PROPERTY_UNKNOWN

The CPU-page property is unknown.

#### D3D12_CPU_PAGE_PROPERTY_NOT_AVAILABLE

The CPU cannot access the heap, therefore no page properties are available.

#### D3D12_CPU_PAGE_PROPERTY_WRITE_COMBINE

The CPU-page property is write-combined.

#### D3D12_CPU_PAGE_PROPERTY_WRITE_BACK

The CPU-page property is write-back.

<a id="memory-pools"></a>

### Memory pools

```cpp
typedef enum D3D12_MEMORY_POOL
{
    D3D12_MEMORY_POOL_UNKNOWN = 0,
    D3D12_MEMORY_POOL_L0 = 1, // Maximum bandwidth for CPU
    D3D12_MEMORY_POOL_L1 = 2, // More bandwidth for GPU, less for CPU
} D3D12_MEMORY_POOL;
```

#### D3D12_MEMORY_POOL_UNKNOWN

The memory pool is unknown.

#### D3D12_MEMORY_POOL_L0

The memory pool is L0. L0 is the physical system memory pool. When the adapter is discrete/NUMA, this pool has greater bandwidth for the CPU and less bandwidth for the GPU. When the adapter is UMA, this pool is the only one which is valid.

#### D3D12_MEMORY_POOL_L1

The memory pool is L1. L1 is typically known as the physical video memory pool. L1 is only available when the adapter is discrete/NUMA, and has greater bandwidth for the GPU. When the adapter is UMA, this pool is not available.

<a id="heap-types"></a>

## Heap types

```cpp
typedef enum D3D12_HEAP_TYPE
{
    D3D12_HEAP_TYPE_DEFAULT   = 1,
    D3D12_HEAP_TYPE_UPLOAD    = 2,
    D3D12_HEAP_TYPE_READBACK  = 3,
    D3D12_HEAP_TYPE_CUSTOM    = 4,
    D3D12_HEAP_TYPE_GPU_UPLOAD = 5,
} D3D12_HEAP_TYPE;
```

### D3D12_HEAP_TYPE_DEFAULT

Specifies the default heap. This heap type experiences the most bandwidth for the GPU, but cannot provide CPU access. The GPU can read and write to the memory from this pool, and resource transition barriers may be changed. The majority of heaps and resources are expected to be located here, and are typically populated through resources in upload heaps.

### D3D12_HEAP_TYPE_UPLOAD

Specifies a heap used for uploading. This heap type has CPU access optimized for uploading to the GPU, but does not experience the maximum amount of bandwidth for the GPU. This heap type is best for CPU-write-once, GPU-read-once data; but GPU-read-once is stricter than necessary. GPU-read-once-or-from-cache is an acceptable use-case for the data; but such usages are hard to judge due to differing GPU cache designs and sizes.

With legacy resource-state creation, resources in this heap must be created with [`D3D12_RESOURCE_STATE_GENERIC_READ`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) and cannot be changed away from this. The CPU address for such heaps is commonly not efficient for CPU reads.

The following are typical usages for \_UPLOAD heaps:
- Initializing resources in a \_DEFAULT heap with data from the CPU.
- Uploading dynamic data in a constant buffer that is read, repeatedly, by each vertex or pixel.

The following are likely not good usages for \_UPLOAD heaps:
- Re-initializing the contents of a resource every frame.
- Uploading constant data which is only used every other Draw call, where each Draw uses a non-trivial amount of other data.

### D3D12_HEAP_TYPE_READBACK

Specifies a heap used for reading back. This heap type has CPU access optimized for reading data back from the GPU, but does not experience the maximum amount of bandwidth for the GPU. This heap type is best for GPU-write-once, CPU-readable data. The CPU cache behavior is write-back, which is conducive for multiple sub-cache-line CPU reads.

With legacy resource-state creation, resources in this heap must be created with [`D3D12_RESOURCE_STATE_COPY_DEST`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states), and cannot be changed away from this.

### D3D12_HEAP_TYPE_CUSTOM

Specifies a custom heap. The application may specify the memory pool and CPU cache properties directly, which can be useful for UMA optimizations, multi-engine, multi-adapter, or other special cases. To do so, the application is expected to understand the adapter architecture to make the right choice. For more details, see [`D3D12_FEATURE_ARCHITECTURE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_feature), [`D3D12_FEATURE_DATA_ARCHITECTURE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_architecture), and [GetCustomHeapProperties](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-getcustomheapproperties%28uint_d3d12_heap_type%29).

Note that textures (unlike buffers) can't be heap type UPLOAD or READBACK.

<a id="gpu-upload"></a>

### GPU-upload heaps

Like `D3D12_HEAP_TYPE_DEFAULT`, `D3D12_HEAP_TYPE_UPLOAD`, and `D3D12_HEAP_TYPE_READBACK`, when heap type is set to `D3D12_HEAP_TYPE_GPU_UPLOAD`, `CPUPageProperty` and `MemoryPoolPreference` must be `D3D12_CPU_PAGE_PROPERTY_UNKNOWN` and `D3D12_MEMORY_POOL_UNKNOWN`, respectively.

On discrete GPUs, Upload heaps have `D3D12_MEMORY_POOL_L0` as their memory pool while GPU upload heaps have `D3D12_MEMORY_POOL_L1` as their memory pool. And the page property for GPU upload heaps is the same as upload heaps, which is `D3D12_CPU_PAGE_PROPERTY_WRITE_COMBINE`. CPU reads of this memory are inefficient.

On integrated GPUs, the memory pool is always `D3D12_MEMORY_POOL_L0`.

On discrete GPUs, when the heap type is `D3D12_HEAP_TYPE_CUSTOM`, the application can specify the `D3D12_CPU_PAGE_PROPERTY` to be `D3D12_CPU_PAGE_PROPERTY_WRITE_COMBINE` and the MemoryPoolPreference to be `D3D12_MEMORY_POOL_L1`. Custom heaps created like this are equivalent to GPU upload heaps.

On integrated GPUs, MemoryPoolPreference needs to be `D3D12_MEMORY_POOL_L0` and `D3D12_CPU_PAGE_PROPERTY` can be `D3D12_CPU_PAGE_PROPERTY_WRITE_COMBINE`/`D3D12_CPU_PAGE_PROPERTY_WRITE_BACK` depending on the adapter architecture.

GetCustomHeapProperties supports `D3D12_HEAP_TYPE_GPU_UPLOAD` and it can always be used to get the corresponding CPUPageProperty and MemoryPoolPreference for current adapter architecture.

GPU upload heaps can be used only if `D3D12_FEATURE_DATA_D3D12_OPTIONS16::GPUUploadHeapSupported` is true. GPU upload heaps creation will fail if they are not supported.

Since GPU upload heaps are CPU-accessible heaps, they do not support shared heap flags such as:
```c++
D3D12_HEAP_FLAG_SHARED
D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER
D3D12_HEAP_FLAG_ALLOW_DISPLAY
```

<a id="heap-description"></a>

## Heap description

```cpp
typedef struct D3D12_HEAP_DESC
{
    UINT64 SizeInBytes;
    D3D12_HEAP_PROPERTIES Properties;
    UINT64 Alignment;
    D3D12_HEAP_FLAGS Flags;
} D3D12_HEAP_DESC;
```

### `SizeInBytes`

The size, in bytes, of the heap. To avoid wasting memory, applications should pass *SizeInBytes* values which are multiples of the effective *Alignment*; but non-aligned *SizeInBytes* is also supported, for convenience. To find out how large a heap must be to support textures with undefined layouts and adapter-specific sizes, call [ID3D12Device::GetResourceAllocationInfo](resources.md#allocation-info).

### `Properties`

A [`D3D12_HEAP_PROPERTIES`](resources.md#heap-properties) structure that describes the heap properties.

### `Alignment`

The alignment value for the heap. Valid values:

| Value | Description |
| --- | --- |
| 0 | An alias for 64KB. |
| `D3D12_DEFAULT_RESOURCE_PLACEMENT_ALIGNMENT` | Defined as 64KB. |
| `D3D12_DEFAULT_MSAA_RESOURCE_PLACEMENT_ALIGNMENT` | Defined as 4MB. This alignment is required when a resource's allocation alignment exceeds 64KB. Eligible small or tightly aligned MSAA resources do not by themselves require a 4MB-aligned heap. |

### `Flags`

A combination of [`D3D12_HEAP_FLAGS`](resources.md#heap-flags)-typed values that are combined by using a bitwise-OR operation. The resulting value identifies heap options. When creating heaps to support adapters with resource heap tier 1, an application must choose some flags.

<a id="heap-flags"></a>

## Heap flags and resource categories

```cpp
typedef enum D3D12_HEAP_FLAGS
{
    D3D12_HEAP_FLAG_NONE = 0x0,
    D3D12_HEAP_FLAG_SHARED = 0x1,
    D3D12_HEAP_FLAG_DENY_BUFFERS = 0x4,
    D3D12_HEAP_FLAG_ALLOW_DISPLAY = 0x8,
    D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER = 0x20,
    D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES = 0x40,
    D3D12_HEAP_FLAG_DENY_NON_RT_DS_TEXTURES = 0x80,
    D3D12_HEAP_FLAG_HARDWARE_PROTECTED = 0x100,
    D3D12_HEAP_FLAG_ALLOW_WRITE_WATCH = 0x200,
    D3D12_HEAP_FLAG_ALLOW_SHADER_ATOMICS = 0x400,
    D3D12_HEAP_FLAG_CREATE_NOT_RESIDENT = 0x800,
    D3D12_HEAP_FLAG_CREATE_NOT_ZEROED = 0x1000,
    D3D12_HEAP_FLAG_TOOLS_USE_MANUAL_WRITE_TRACKING = 0x2000,

    // These are convenience aliases to manage resource heap tier restrictions. They cannot be bitwise OR'ed together cleanly.
    D3D12_HEAP_FLAG_ALLOW_ALL_BUFFERS_AND_TEXTURES = 0x0,
    D3D12_HEAP_FLAG_ALLOW_ONLY_BUFFERS = 0xC0,
    D3D12_HEAP_FLAG_ALLOW_ONLY_NON_RT_DS_TEXTURES = 0x44,
    D3D12_HEAP_FLAG_ALLOW_ONLY_RT_DS_TEXTURES = 0x84,


} D3D12_HEAP_FLAGS;
```

### D3D12_HEAP_FLAG_NONE

No options are specified.

### D3D12_HEAP_FLAG_SHARED

The heap is shared. Refer to [Shared Heaps](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/shared-heaps).

### D3D12_HEAP_FLAG_DENY_BUFFERS

The heap isn't allowed to contain buffers.

### D3D12_HEAP_FLAG_ALLOW_DISPLAY

The heap is allowed to contain swap-chain surfaces.

### D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER

The heap is allowed to share resources across adapters. Refer to [Shared Heaps](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/shared-heaps). A protected session cannot be mixed with resources that are shared across adapters.

### D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES

The heap is not allowed to store Render Target (RT) and/or Depth-Stencil (DS) textures.

### D3D12_HEAP_FLAG_DENY_NON_RT_DS_TEXTURES

The heap is not allowed to contain resources with `D3D12_RESOURCE_DIMENSION_TEXTURE1D`, `D3D12_RESOURCE_DIMENSION_TEXTURE2D`, or `D3D12_RESOURCE_DIMENSION_TEXTURE3D` unless either `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` are present. Refer to [`D3D12_RESOURCE_DIMENSION`](resources.md#resource-dimensions) and [`D3D12_RESOURCE_FLAGS`](resources.md#resource-flags).

### D3D12_HEAP_FLAG_HARDWARE_PROTECTED

Unsupported.

### D3D12_HEAP_FLAG_ALLOW_WRITE_WATCH

The heap supports MEM\_WRITE\_WATCH functionality, which causes the system to track the pages that are written to in the committed memory region. This flag can't be combined with the `D3D12_HEAP_TYPE_DEFAULT` or `D3D12_CPU_PAGE_PROPERTY_UNKNOWN` flags. Applications are discouraged from using this flag themselves because it prevents tools from using this functionality.

### D3D12_HEAP_FLAG_ALLOW_SHADER_ATOMICS

Ensures that atomic operations will be atomic on this heap's memory, according to components able to see the memory.

Creating a heap with this flag will fail under either of these conditions:

- The heap type is `D3D12_HEAP_TYPE_DEFAULT`, and the heap can be visible on multiple nodes, but the device does *not* support [`D3D12_CROSS_NODE_SHARING_TIER_3`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_cross_node_sharing_tier).
- The heap is CPU-visible, but the heap type is *not* `D3D12_HEAP_TYPE_CUSTOM`.

Note that heaps with this flag might be a limited resource on some systems.

### D3D12_HEAP_FLAG_CREATE_NOT_RESIDENT

The heap is created in a non-resident state and must be made resident using [ID3D12Device::MakeResident](#make-resident) or [ID3D12Device3::EnqueueMakeResident](#enqueue-make-resident).

By default, the final step of heap creation is to make the heap resident, so this flag skips this step and allows the application to decide when to do so.

### D3D12_HEAP_FLAG_CREATE_NOT_ZEROED

Allows the OS to not zero the heap created. By default, committed resources and heaps are almost always zeroed upon creation. This flag allows this to be elided in some scenarios. However, it doesn't guarantee it. For example, memory coming from other processes still needs to be zeroed for data protection and process isolation. This can lower the overhead of creating the heap.

### D3D12_HEAP_FLAG_ALLOW_ALL_BUFFERS_AND_TEXTURES

The heap is allowed to store all types of buffers and/or textures. This is an alias; for more details, see [Aliases](#aliases).

### D3D12_HEAP_FLAG_ALLOW_ONLY_BUFFERS

The heap is only allowed to store buffers. This is an alias; for more details, see [Aliases](#aliases).

### D3D12_HEAP_FLAG_ALLOW_ONLY_NON_RT_DS_TEXTURES

The heap is only allowed to store non-RT, non-DS textures. This is an alias; for more details, see [Aliases](#aliases).

### D3D12_HEAP_FLAG_ALLOW_ONLY_RT_DS_TEXTURES

The heap is only allowed to store RT and/or DS textures. This is an alias; for more details, see [Aliases](#aliases).

### Resource heap tiers

The following heap flags must be used with [ID3D12Device::CreateHeap](resources.md#create-heap), but will be set automatically for implicit heaps created by [ID3D12Device::CreateCommittedResource](resources.md#create-committed-resource). For ordinary non-cross-adapter heaps on [heap tier 1](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_heap_tier), two out of the three following flags must be set.

| Value | Description |
| --- | --- |
| `D3D12_HEAP_FLAG_DENY_BUFFERS` | The heap isn't allowed to contain resources with `D3D12_RESOURCE_DIMENSION_BUFFER` (which is a [`D3D12_RESOURCE_DIMENSION`](resources.md#resource-dimensions) enumeration constant). |
| `D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES` | The heap isn't allowed to contain resources with `D3D12_RESOURCE_DIMENSION_TEXTURE1D`, `D3D12_RESOURCE_DIMENSION_TEXTURE2D`, or `D3D12_RESOURCE_DIMENSION_TEXTURE3D` together with either `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`. (The latter two items are [`D3D12_RESOURCE_FLAGS`](resources.md#resource-flags) enumeration constants.) |
| `D3D12_HEAP_FLAG_DENY_NON_RT_DS_TEXTURES` | The heap isn't allowed to contain resources with `D3D12_RESOURCE_DIMENSION_TEXTURE1D`, `D3D12_RESOURCE_DIMENSION_TEXTURE2D`, or `D3D12_RESOURCE_DIMENSION_TEXTURE3D` unless `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` is present. |

### Aliases
 Adapters that support [heap tier 2](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_heap_tier) or greater are additionally allowed to set none of the above flags. Aliases for these flags are available for applications that prefer thinking only of which resources are supported.
The following aliases exist, so be careful when doing bit-manipulations:

- `D3D12_HEAP_FLAG_ALLOW_ALL_BUFFERS_AND_TEXTURES` = 0 and is supported for ordinary heaps on [heap tier 2](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_heap_tier) and greater; shared-cross-adapter heaps also use this zero category mask.
- `D3D12_HEAP_FLAG_ALLOW_ONLY_BUFFERS` = `D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES` | `D3D12_HEAP_FLAG_DENY_NON_RT_DS_TEXTURES`
- `D3D12_HEAP_FLAG_ALLOW_ONLY_NON_RT_DS_TEXTURES` = `D3D12_HEAP_FLAG_DENY_BUFFERS` | `D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES`
- `D3D12_HEAP_FLAG_ALLOW_ONLY_RT_DS_TEXTURES` = `D3D12_HEAP_FLAG_DENY_BUFFERS` | `D3D12_HEAP_FLAG_DENY_NON_RT_DS_TEXTURES`

Heaps with `D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER` must leave `D3D12_HEAP_FLAG_DENY_BUFFERS`, `D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES` and `D3D12_HEAP_FLAG_DENY_NON_RT_DS_TEXTURES` clear. This category-mask exception does not remove the [cross-adapter resource restrictions](https://learn.microsoft.com/en-us/windows/win32/direct3d12/shared-heaps).

### Displayable heaps

Displayable heaps are most commonly created by the swapchain for presentation, to enable scanning out to a monitor.
Displayable heaps are specified with the `D3D12_HEAP_FLAG_ALLOW_DISPLAY` member of the `D3D12_HEAP_FLAGS` enum.

Applications may create displayable heaps outside of a swapchain; but cannot actually present with them. This flag is not supported by [CreateHeap](resources.md#create-heap) and can only be used with [CreateCommittedResource](resources.md#create-committed-resource) with `D3D12_HEAP_TYPE_DEFAULT`.

Additional restrictions to the [`D3D12_RESOURCE_DESC`](resources.md#resource-description) apply to the resource created with displayable heaps.

- The format must not only be supported by the device, but must be supported for scan-out. Refer to the use of the `D3D12_FORMAT_SUPPORT1_DISPLAY` member of [`D3D12_FORMAT_SUPPORT1`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_format_support1).
- *Dimension* must be `D3D12_RESOURCE_DIMENSION_TEXTURE2D`.
- *Alignment* must be 0.
- *DepthOrArraySize* may be either 1 or 2.
- *MipLevels* must be 1.
- *SampleDesc* must have *Count* set to 1 and *Quality* set to 0.
- *Layout* must be `D3D12_TEXTURE_LAYOUT_UNKNOWN`.
- `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` and `D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER` are invalid flags.

<a id="manual-write-tracking"></a>

### D3D12_HEAP_FLAG_TOOLS_USE_MANUAL_WRITE_TRACKING

D3D12 sets ManualWriteTrackingResourceSupported to false by default, and tools like Pix will override this value. If this value is false, using `D3D12_HEAP_FLAG_TOOLS_USE_MANUAL_WRITE_TRACKING` is treated as an error by D3D12.

If an ID3D12Heap* is created with the `D3D12_HEAP_FLAG_TOOLS_USE_MANUAL_WRITE_TRACKING` flag, the application is expected to use ID3D12ManualWriteTrackingResource to notify tools about any CPU resource writes that occur on any placed or committed resources in the heap. Nothing needs to be done for reserved/tiled resources, because D3D12 does not allow Map() to be used on reserved/tiled resources.

ID3D12ManualWriteTrackingResource can be used with any type of CPU writeable resource, not just GPU upload heaps.

`ManualWriteTrackingResourceSupported` is reported by `D3D12_FEATURE_DATA_D3D12_OPTIONS17`. The [manual-write notification contract](../D3D12GPUUploadHeaps.md#detect-modifications-to-the-gpu-upload-heaps) defines the tool-provided `ID3D12ManualWriteTrackingResource` interface and `TrackWrite` ranges.

<a id="residency"></a>

## Residency and memory budgets

To prevent applications from using too much memory which can cause the system to thrash, the graphics kernel provides a per-process budget, representing what it thinks is a fair amount of video memory for that application to use. Under low memory pressure, this value is generally large for all processes. Once memory starts becoming constrained, the graphics kernel will start restricting the budget of various processes. The graphics kernel will prioritize memory to DWM (the desktop compositor that composites rendertargets from multiple apps among other things) and the foreground application above applications running in the background. This helps ensure that the application the user is currently using remains smooth and responsive.

Windows' DirectX Graphics Infrastructure (DXGI) provides two means of allowing applications to monitor their memory usage: `QueryVideoMemoryInfo` and `RegisterVideoMemoryBudgetChangeNotificationEvent`.

`QueryVideoMemoryInfo` is a way for the application to poll the graphics kernel for information about how much memory they are using, how much video memory their process is budgeted, and information about video memory reservation.

In contrast to the polling method, `RegisterVideoMemoryBudgetChangeNotificationEvent` allows the application to map an event object to budget changes, allowing the graphics kernel to inform the application when their budget changes. This notification should be combined with the use of `QueryVideoMemoryInfo` (after the event is signaled) to refresh the application's view of video memory.

```cpp
typedef enum DXGI_MEMORY_SEGMENT_GROUP
{
    // Local to the video adapter; fastest available memory to the GPU.
    // Applications should target this group as their working-set size.
    DXGI_MEMORY_SEGMENT_GROUP_LOCAL     = 0,
    // Non-local to the video adapter; may have slower performance than LOCAL.
    DXGI_MEMORY_SEGMENT_GROUP_NON_LOCAL = 1
} DXGI_MEMORY_SEGMENT_GROUP;

typedef struct DXGI_QUERY_VIDEO_MEMORY_INFO
{
    // OS-provided video-memory budget, in bytes, that the application
    // should target. Exceeding Budget may incur stuttering or performance
    // penalties as the OS gives other applications fair use of video memory.
    UINT64 Budget;
    // Application's current video-memory usage, in bytes.
    UINT64 CurrentUsage;
    // Bytes the application has available to reserve via
    // IDXGIAdapter3::SetVideoMemoryReservation.
    UINT64 AvailableForReservation;
    // Bytes reserved by the application. The OS uses this as a hint for
    // the application's minimum working set; the application should ensure
    // its usage can be trimmed to this amount.
    UINT64 CurrentReservation;
} DXGI_QUERY_VIDEO_MEMORY_INFO;
```

The residency budget is not a guarantee that an allocation will succeed, nor is it an unconditional prohibition on allocations above the budget. `D3D12_RESIDENCY_FLAG_DENY_OVERBUDGET` selects the stricter queued-residency policy.

<a id="resident-objects"></a>

### Residency ownership

The following object types support application-controlled residency:

| Object | Memory owner |
|---|---|
| `ID3D12Heap` | Explicit heap backing |
| Committed `ID3D12Resource` | Its implicit heap |
| `ID3D12DescriptorHeap` | Descriptor storage |
| `ID3D12QueryHeap` | Query storage |
| `ID3D12VideoDecoder` | Decoder backing allocations |
| `ID3D12VideoProcessor` | Processor backing allocations |
| `ID3D12VideoDecoderHeap` | Decoder-heap backing allocations |
| `ID3D12VideoMotionEstimator` | Motion-estimator backing allocations |
| `ID3D12VideoMotionVectorHeap` | Motion-vector backing allocations |
| `ID3D12VideoExtensionCommand` | Extension-command backing allocations |
| `ID3D12VideoEncoder` | Encoder backing allocations |
| `ID3D12VideoEncoderHeap` | Encoder-heap backing allocations |

A placed resource borrows its heap's memory; a reserved resource's tile mappings refer to heaps. Residency of that backing memory is managed through its owning heap rather than by treating each mapping as an independent allocation. Inheritance from `ID3D12Pageable` is not by itself permission to call application residency methods: command allocators, command queues, pipeline state objects, meta-commands and state objects are not direct application-residency targets.

D3D12's heap eviction interface gives applications the ability to reduce their memory footprint by evicting without actually destroying the heap. The application can then "make the heap resident" again to make it available for rendering.

`MakeResident` acquires a residency reference and `Evict` releases one; eviction becomes possible when the residency-reference count reaches zero. Objects created resident have an initial residency reference; `D3D12_HEAP_FLAG_CREATE_NOT_RESIDENT` omits that initial residency.

`EnqueueMakeResident` acquires the same residency reference as `MakeResident`. Its asynchronous completion fence does not replace the matching `Evict` operation.

Applications must use fences to ensure the GPU doesn't use non-resident objects. **MakeResident** must return before the GPU accesses the object. [Evict](#evict) must be called after the GPU finishes the accesses that require the object to remain resident.

Evicted objects still consume the same GPU virtual address and same amount of GPU virtual address space. Therefore, resource descriptors and other GPU virtual address references are not invalidated after [Evict](#evict).

In D3D12, applications must tightly understand all GPU-accessible memory lifespans, including residency. The resource binding model enables applications to build large descriptor heaps for resources. Some entries can refer to resources/ heaps that are no longer alive or resident. The application must ensure such entries are not referenced to keep the application running normally.

<a id="make-resident"></a>

### ID3D12Device::MakeResident

#### Declaration

```cpp
HRESULT MakeResident(
    UINT NumObjects,
    ID3D12Pageable* const* ppObjects);
```

#### Behavior

Makes objects resident for the device.

#### Parameters

##### `NumObjects`

The number of objects in the *ppObjects* array to make resident for the device.

##### `ppObjects`

A pointer to a memory block that contains an array of [ID3D12Pageable](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pageable) interface pointers for the objects.

Eligible objects are defined in [Residency ownership](#resident-objects).

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues).

#### Allocation and paging

**MakeResident** and [Evict](#evict) can help applications manage the residency budget on many adapters. **MakeResident** explicitly pages-in data and, then, precludes page-out so the GPU can access the data. **Evict** enables page-out.

Applications must handle **MakeResident** failures, even if there appears to be enough residency budget available. Physical memory fragmentation and adapter architecture quirks can preclude the utilization of large contiguous ranges. Applications should free up more residency budget before trying again.

<a id="evict"></a>

### ID3D12Device::Evict

#### Declaration

```cpp
HRESULT Evict(
    UINT NumObjects,
    ID3D12Pageable* const* ppObjects);
```

#### Behavior

Enables the page-out of data, which precludes GPU access of that data.

#### Parameters

##### `NumObjects`

The number of objects in the *ppObjects* array to evict from the device.

##### `ppObjects`

A pointer to a memory block that contains an array of [ID3D12Pageable](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pageable) interface pointers for the objects.

Eligible objects are defined in [Residency ownership](#resident-objects).

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues).

#### Backing-storage ownership

This method should be called on the object which owns the physical memory: either a committed resource (which owns both virtual and physical memory assignments) or a heap - noting that reserved resources do not have physical memory, and placed resources are borrowing memory from a heap.

<a id="enqueue-make-resident"></a>

### ID3D12Device3::EnqueueMakeResident

#### Declaration

```cpp
HRESULT EnqueueMakeResident(
    D3D12_RESIDENCY_FLAGS Flags,
    UINT NumObjects,
    ID3D12Pageable* const* ppObjects,
    ID3D12Fence* pFenceToSignal,
    UINT64 FenceValueToSignal);
```

#### Behavior

Asynchronously makes objects resident for the device.

#### Parameters

##### `Flags`

Controls whether the objects should be made resident if the application is over its memory budget.

##### `NumObjects`

The number of objects in the *ppObjects* array to make resident for the device.

##### `ppObjects`

A pointer to a memory block; contains an array of [ID3D12Pageable](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pageable) interface pointers for the objects.

Eligible objects are defined in [Residency ownership](#resident-objects).

##### `pFenceToSignal`

A pointer to the fence used to signal when the work is done.

##### `FenceValueToSignal`

An unsigned 64-bit value signaled to the fence when the work is done.

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues).

#### Completion and ordering

**EnqueueMakeResident** performs the same actions as [MakeResident](#make-resident), but does not wait for the resources to be made resident. Instead, **EnqueueMakeResident** signals a fence when the work is done.

A successful return does not itself establish completion of residency work. The application must order GPU access after the residency-completion fence, using the [queue wait contract](execution.md#queue-waits).

Calls to this API are guaranteed to signal their corresponding fence in order, so the same fence can be used from call to call.

#### Completion fence eligibility

`pFenceToSignal` must be a writable fence that supports queue/CPU synchronization. Ordinary fences and supported [non-monitored fences](execution.md#fence-flags) are eligible. [Command-list-scoped fences](execution.md#command-list-scoped-fences) and [read-only protected-session status fences](execution.md#status-fences) are not eligible. Non-monitored fences retain their CPU-observation restrictions; the residency signal does not turn them into monitored fences.

### Residency flags

```cpp
typedef enum D3D12_RESIDENCY_FLAGS
{
    D3D12_RESIDENCY_FLAG_NONE = 0x0,
    D3D12_RESIDENCY_FLAG_DENY_OVERBUDGET = 0x1,
} D3D12_RESIDENCY_FLAGS;
```

`D3D12_RESIDENCY_FLAG_NONE` permits the normal over-budget behavior. `D3D12_RESIDENCY_FLAG_DENY_OVERBUDGET` causes an over-budget residency request to fail with `E_OUTOFMEMORY` rather than proceeding with that policy.

<a id="cpu-access"></a>

## CPU access and mapping

The runtime ref-counts the driver's `pfnMapHeap` / `pfnUnmapHeap` (DDI) calls internally and serializes them across multiple application threads. This avoids redundant calls into the driver. (No equivalent `Map` exists on `ID3D12Heap` at the API level — only on `ID3D12Resource`.)

1. `Map` has no discard mode in D3D12. Applications must implement resource renaming themselves.
2. Map calls are implicitly `NO_OVERWRITE` and multi-threaded. There is no `DO_NOT_WAIT` Map flag.
    1. The driver does not implicitly flush any command buffers when map is called. It is the application's responsibility to ensure that any relevant work contained in command lists is submitted to the appropriate command queue before relying on completion of that work.
    2. Map does not block waiting for the GPU to finish work. It is the application's responsibility to ensure that the GPU has finished any relevant work before conflicting CPU access to that data.

CPU-accessible heaps can be mapped at the DDI level, even while the GPU is busy reading or writing to the resource. The runtime will understand how to handle the lack of I/O coherence.

<a id="cpu-texture-state"></a>

### CPU texture access and state

With the legacy state model, CPU-accessible *textures* must be in the `D3D12_RESOURCE_STATE_COMMON` state in order to be accessed for read or write from the CPU.

The legacy COMMON-state restriction for CPU texture access does not impose a COMMON state on buffers. Buffer access still requires synchronization of conflicting CPU/GPU accesses. For texture access with enhanced barriers, the corresponding common-layout and CPU-visibility requirements apply.

<a id="cpu-range"></a>

### CPU read/write ranges

```cpp
typedef struct D3D12_RANGE
{
    SIZE_T Begin;
    SIZE_T End; // One past end, so (End - Begin) = Size
} D3D12_RANGE;
```

#### `Begin`

The offset, in bytes, denoting the beginning of a memory range.

#### `End`

The offset, in bytes, denoting the end of a memory range. **End** is one-past-the-end.

### Mapping, ranges and synchronization

Read/write ranges describe CPU access for cache management and tooling; they are not permissions or synchronization barriers. Mapping or unmapping does not authorize races with GPU accesses. Resource [state and barrier requirements](synchronization.md) and [completion dependencies](execution.md#completion) remain distinct.

<a id="resource-map"></a>

### ID3D12Resource::Map

#### Declaration

```cpp
HRESULT Map(
    UINT Subresource,
    const D3D12_RANGE* pReadRange,
    void** ppData);
```

#### Behavior

Gets a CPU pointer to the specified subresource in the resource, but may not disclose the pointer value to applications. **Map** also invalidates the CPU cache, when necessary, so that CPU reads to this address reflect any modifications made by the GPU.

#### Parameters

##### `Subresource`

Specifies the index number of the subresource.

##### `pReadRange`

A pointer to a [`D3D12_RANGE`](#cpu-range) structure that describes the range of memory to access.

This indicates the region the CPU might read, and the coordinates are subresource-relative. A null pointer indicates the entire subresource might be read by the CPU. It is valid to specify the CPU won't read any data by passing a range where **End** is less than or equal to **Begin**.

##### `ppData`

A pointer to a memory block that receives a pointer to the resource data.

A null pointer is valid and is useful to cache a CPU virtual address range for methods like [WriteToSubresource](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource). When *ppData* is not NULL, the pointer returned is never offset by any values in *pReadRange*.

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues).

#### Nested mapping and address lifetime

**Map** and [Unmap](#resource-unmap) can be called by multiple threads safely. Nested **Map** calls are supported and are ref-counted. The first call to **Map** allocates a CPU virtual address range for the resource. The last call to **Unmap** deallocates the CPU virtual address range. The CPU virtual address is commonly returned to the application; but manipulating the contents of textures with unknown layouts precludes disclosing the CPU virtual address. See [WriteToSubresource](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) for more details. Applications cannot rely on the address being consistent, unless **Map** is persistently nested.

Pointers returned by **Map** are not guaranteed to have all the capabilities of normal pointers, but most applications won't notice a difference in normal usage. For example, pointers with WRITE\_COMBINE behavior have weaker CPU memory ordering guarantees than WRITE\_BACK behavior. Memory accessible by both CPU and GPU are not guaranteed to share the same atomic memory guarantees that the CPU has, due to PCIe limitations. Use fences for synchronization.

#### Persistent mapping

Resources on CPU-accessible heaps can be persistently mapped, meaning **Map** can be called once, immediately after resource creation. [Unmap](#resource-unmap) never needs to be called, but the address returned from **Map** must no longer be used after the last reference to the resource is released. When using persistent map, the application must ensure the CPU finishes writing data into memory before the GPU reads or writes that memory. In common scenarios, the application merely must write to memory before calling [ExecuteCommandLists](execution.md#execute-command-lists); but using a fence to delay command list execution works as well.
All CPU-accessible memory types support persistent mapping usage, where the resource is mapped but then never unmapped, provided the application does not access the pointer after the resource has been disposed.

#### Opaque texture layouts

`Map` will only succeed on opaque resource layouts, if `ppData` is NULL. This is useful to cache information for the [CPU subresource copy operations](../ResourceHeaps.md#strip-mine-cpu-copy-operations) across multiple copy operations. The following restrictions apply to CPU subresource copy operations on opaque resources:

- No MSAA resources
- No depth-stencil resources
- Formats must have a power-of-two element size
- Volume textures must have only a single mip-level

Ranges must be NULL or empty when the layout is opaque.

<a id="resource-unmap"></a>

### ID3D12Resource::Unmap

#### Declaration

```cpp
void Unmap(
    UINT Subresource,
    const D3D12_RANGE* pWrittenRange);
```

#### Behavior

Releases a mapping reference for the specified subresource. The CPU address lifetime follows the nested-mapping rules of `Map`.

#### Parameters

##### `Subresource`

Specifies the index of the subresource.

##### `pWrittenRange`

A pointer to a [`D3D12_RANGE`](#cpu-range) structure that describes the range of memory to unmap.

This indicates the region the CPU might have modified, and the coordinates are subresource-relative. A null pointer indicates the entire subresource might have been modified by the CPU. It is valid to specify the CPU didn't write any data by passing a range where **End** is less than or equal to **Begin**.

This parameter is only used by tooling, and not for correctness of the actual unmap operation.

#### Return values

None

<a id="allocation-info"></a>

## Allocation size and alignment

```cpp
typedef struct D3D12_RESOURCE_ALLOCATION_INFO
{
    UINT64 SizeInBytes;
    UINT64 Alignment;
} D3D12_RESOURCE_ALLOCATION_INFO;
```

<a id="get-resource-allocation-info"></a>

### ID3D12Device::GetResourceAllocationInfo

#### Declaration

```cpp
D3D12_RESOURCE_ALLOCATION_INFO GetResourceAllocationInfo(
    UINT visibleMask,
    UINT numResourceDescs,
    const D3D12_RESOURCE_DESC* pResourceDescs);
```

#### Behavior

Gets the size and alignment of memory required for a collection of resources on this adapter.

#### Parameters

##### `visibleMask`

For single-GPU operation, set this to zero. If there are multiple GPU nodes, then set bits to identify the nodes (the device's physical adapters). Each bit in the mask corresponds to a single node. Also see [Multi-adapter systems](https://learn.microsoft.com/en-us/windows/win32/direct3d12/multi-engine).

##### `numResourceDescs`

The number of resource descriptors in the *pResourceDescs* array.

##### `pResourceDescs`

An array of `D3D12_RESOURCE_DESC` structures that describe the resources to get info about.

#### Return value

A [`D3D12_RESOURCE_ALLOCATION_INFO`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_resource_allocation_info) structure that provides info about video memory allocated for the specified array of resources.

If an error occurs, then **`D3D12_RESOURCE_ALLOCATION_INFO::SizeInBytes`** equals `UINT64_MAX`.

#### Allocation rules

When you're using [CreatePlacedResource](resources.md#create-placed-resource), your application must use **GetResourceAllocationInfo** in order to understand the size and alignment characteristics of texture resources. The results of this method vary depending on the particular adapter, and must be treated as unique to this adapter and driver version.

Your application can't use the output of **GetResourceAllocationInfo** to understand packed mip properties of textures. To understand packed mip properties of textures, your application must use [GetResourceTiling](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-getresourcetiling).

Texture resource sizes significantly differ from the information returned by **GetResourceTiling**, because some adapter architectures allocate extra memory for textures to reduce the effective bandwidth during common rendering scenarios. This even includes textures that have constraints on their texture layouts, or have standardized texture layouts. That extra memory can't be sparsely mapped nor remapped by an application using [CreateReservedResource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-createreservedresource) and [UpdateTileMappings](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings), so it isn't reported by **GetResourceTiling**.

Your application can forgo using **GetResourceAllocationInfo** for buffer resources ([`D3D12_RESOURCE_DIMENSION_BUFFER`](resources.md#resource-dimensions)) when not using [Tight Alignment](resources.md#tight-alignment). Buffers that are not tightly aligned have the same size on all adapters, which is merely the smallest multiple of 64KB that's greater or equal to [`D3D12_RESOURCE_DESC::Width`](resources.md#resource-description).

When multiple resource descriptions are passed in, the C++ algorithm for calculating a structure size and alignment are used. For example, a three-element array with two tiny 64KB-aligned resources and a tiny 4MB-aligned resource, reports differing sizes based on the order of the array. If the 4MB aligned resource is in the middle, then the resulting **SizeInBytes** is 12MB. Otherwise, the resulting **SizeInBytes** is 8MB. The **Alignment** returned would always be 4MB, because it's the superset of all alignments in the resource array.

<a id="tight-alignment"></a>

## Tight resource alignment

For alignment selection, a "Small resource" is defined as:
1. MUST have UNKNOWN layout.
1. MUST NOT be RENDER_TARGET or DEPTH_STENCIL.
1. The estimated size of the most-detailed mip level MUST be a total of the larger alignment restriction or less.

The runtime will use an architecture-independent mechanism of size-estimation, that mimics the way standard swizzle and D3D11 tiled resources are sized.
However, the tile sizes will be of the smaller alignment restriction for such calculations. Additional data associated with resources, which is typically associated with compression, will not be added into this size.
For a non-MSAA texture, when this calculated size is <= 64 KiB, the alignment of 4 KiB is permitted.
For an MSAA texture, when this calculated size is <= 4 MiB, the alignment of 64 KiB is permitted.

`D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT` is valid only when `D3D12_FEATURE_DATA_TIGHT_ALIGNMENT::SupportTier` reports `D3D12_TIGHT_ALIGNMENT_TIER_1`. The query uses `D3D12_FEATURE_D3D12_TIGHT_ALIGNMENT`.

```cpp
typedef enum D3D12_TIGHT_ALIGNMENT_TIER
{
    D3D12_TIGHT_ALIGNMENT_TIER_NOT_SUPPORTED,
    D3D12_TIGHT_ALIGNMENT_TIER_1                // Tight alignment of buffers supported
} D3D12_TIGHT_ALIGNMENT_TIER;
```

```cpp
typedef struct D3D12_FEATURE_DATA_TIGHT_ALIGNMENT
{
    D3D12_TIGHT_ALIGNMENT_TIER SupportTier;
} D3D12_FEATURE_DATA_TIGHT_ALIGNMENT;
```

The placed and committed tables below describe tight driver-selected alignment ranges without an application-requested higher floor. [Nonzero alignment floors](#tight-input-floors) have separate input validation.

### Placed resources

|       Type        | MIN | MAX                                                            |
|-------------------|-----|----------------------------------------------------------------|
|      Buffers      | 8B  | 256B                                                           |
|      Textures     | 8B  | 64KiB (4KiB when it meets the definition of a Small Resource)  |
|        MSAA       | 8B  | 4MiB (64KiB when it meets the definition of a Small Resource)  |

### Committed resources

Committed resources can also benefit from having their minimum alignment reduced, specifically committed buffers.
There is some nuance here though due to the fact that a heap is implicitly created for each committed resource, and the Windows video memory manager's (VidMM's) minimum alignment and size granularity for managing memory is 4KiB.
Each committed resource creation call results in 1 allocation.
Drivers do not need to manage suballocations; the minimum alignment for committed resources is 4KiB or larger.

**Note: For drivers that report LargePageSupport for VidMM allocations, it is acceptable and optimal for allocations that are a multiple of the LargePage size to be aligned to the large page size instead of using the 4KiB alignment.**



|      Type        | MIN  | MAX                                                            |
|------------------|------|----------------------------------------------------------------|
|     Buffers      | 4KiB | 4KiB                                                           |
|     Textures     | 4KiB | 64KiB (4KiB when it meets the definition of a Small Resource)  |
|       MSAA       | 4KiB | 4MiB (64KiB when it meets the definition of a Small Resource)  |

### Validation

* Cases where a warning will be issued via debug layer when `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT` is set:
  * Used along-side `D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER`, as alignment MUST be 64 KiB, or 4MiB for MSAA
* Cases where `E_INVALIDARG` will be returned when `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT` is set:
  * Used with `D3D12_TEXTURE_LAYOUT_64KB_UNDEFINED_SWIZZLE` or `D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE` resource formats, as alignment must be 64KiB. **Note that this means that Reserved Resources are not supported**.
  * Used when `D3D12_FEATURE_D3D12_TIGHT_ALIGNMENT` is reported as `D3D12_TIGHT_ALIGNMENT_TIER_NOT_SUPPORTED`
  * The app-supplied `Alignment` is non-zero and not a power of two
  * The app-supplied `Alignment` exceeds the [maximum input floor](#tight-input-floors) for its resource class

### Nonzero alignment floors

When `USE_TIGHT_ALIGNMENT` is set and `Alignment` in the resource desc is non-zero, the runtime treats the value as a *target floor* rather than a hard requirement.
The runtime honors the app-supplied floor when it is a valid value (i.e. a power of two and within the per-type spec maximum) and clamps up to its own required minimum when the floor is too small.

When the runtime has to clamp up, a debug-layer INFO message `D3D12_MESSAGE_ID_CREATERESOURCE_ALIGNMENT_OVERRIDDEN` is emitted so the override is discoverable without the app having to compare `Alignment` against `GetDesc()` afterwards.

Because a non-zero `Alignment` is now legal, applications can round-trip a desc through `ID3D12Resource::GetDesc` and pass it back into `CreateCommittedResource*`, `CreatePlacedResource*`, `GetResourceAllocationInfo`, or `GetCopyableFootprints` without needing to reset `Alignment` to `0` first.

<a id="tight-input-floors"></a>

#### Maximum input floors

For non-cross-adapter resources, a nonzero `Alignment` floor must be a power of two and must not exceed the following input limits:

| Resource class | Maximum input floor |
|---|---|
| Placed buffer | 256 B |
| Committed buffer | 4 KiB |
| Non-MSAA texture | 64 KiB |
| MSAA texture | 4 MiB |

These are input limits, not the driver-selected output ranges in the tables above. Small-resource qualifications do not reduce the texture input limits. A valid requested floor can raise the selected alignment above the no-floor small-resource range.

Resource allocation alignment does not relax the alignment required by a particular use of a buffer. The [published buffer-use alignment requirements](../D3D12TightPlacedResourceAlignment.md#choosing-an-alignment-floor-for-buffer-use-cases) remain applicable; explicit heap alignment is a separate requirement from a resource's placement alignment.

<a id="creation-contracts"></a>

## Creation contracts

Resource creation is free-threaded. The initial resource state must be valid for the resource type, or `E_INVALIDARG` occurs.

The legacy creation methods specify an initial state valid for the resource description. The [legacy state model](synchronization.md#legacy-resource-states) and [common-state promotion/decay](synchronization.md#legacy-promotion-and-decay) define subsequent state behavior. The [enhanced-layout creation alternatives](#versioned-creation) specify `InitialLayout` instead.

Writable multisampled UAV resources also require `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET`; `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` cannot be combined with unordered access.

For these creation methods, a null output pointer selects parameter validation without object creation. `S_FALSE` does not establish successful backing allocation or support for the requested `riid`/`riidResource`. `CreatePlacedResource` has the same validation-only contract.

<a id="create-heap"></a>

## ID3D12Device::CreateHeap

### Declaration

```cpp
HRESULT CreateHeap(
    const D3D12_HEAP_DESC* pDesc,
    REFIID riid,
    void** ppvHeap);
```

### Behavior

Creates a heap that can be used with placed resources and reserved resources.

### Parameters

#### `pDesc`

A pointer to a constant `D3D12_HEAP_DESC` structure that describes the heap.

#### `riid`

A reference to the globally unique identifier (**GUID**) of the heap interface to return in *ppvHeap*.

While *riid* is most commonly the **GUID** of [ID3D12Heap](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12heap), it may be the **GUID** of any interface. If the heap object doesn't support the interface for this **GUID**, then creation fails with `E_NOINTERFACE`.

#### `ppvHeap`

An optional pointer to a memory block that receives the requested interface pointer to the created heap object.

*ppvHeap* can be `nullptr`, for parameter validation. When *ppvHeap* is `nullptr`, no object is created, and `S_FALSE` is returned when the parameters are valid.

### Return values

If an object is created successfully, the method returns `S_OK`; successful validation with a null output pointer returns `S_FALSE`. Otherwise, it returns an [**HRESULT**](https://learn.microsoft.com/en-us/windows/desktop/com/structure-of-com-error-codes)[error code](https://learn.microsoft.com/en-us/windows/win32/com/com-error-codes-10).

| Return value | Description |
| --- | --- |
| `E_OUTOFMEMORY` | There is insufficient memory to create the heap. |

See [Direct3D 12 return codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

### Lifetime

Before releasing the final reference on the heap, your application must ensure that the GPU will no longer read or write to this heap.

A placed resource object holds a reference on the heap it is created on; but a reserved resource doesn't hold a reference for each mapping made to a heap.

<a id="create-committed-resource"></a>

## ID3D12Device::CreateCommittedResource

### Declaration

```cpp
HRESULT CreateCommittedResource(
    const D3D12_HEAP_PROPERTIES* pHeapProperties,
    D3D12_HEAP_FLAGS HeapFlags,
    const D3D12_RESOURCE_DESC* pDesc,
    D3D12_RESOURCE_STATES InitialResourceState,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    REFIID riidResource,
    void** ppvResource);
```

### Behavior

Creates both a resource and an implicit heap, such that the heap is big enough to contain the entire resource, and the resource is mapped to the heap.

### Parameters

#### `pHeapProperties`

A pointer to a `D3D12_HEAP_PROPERTIES` structure that provides properties for the resource's heap.

#### `HeapFlags`

Heap options, as a bitwise-OR'd combination of `D3D12_HEAP_FLAGS` enumeration constants.

#### `pDesc`

A pointer to a `D3D12_RESOURCE_DESC` structure that describes the resource.

#### `InitialResourceState`

The initial state of the resource, as a bitwise-OR'd combination of `D3D12_RESOURCE_STATES` enumeration constants.

When you create a resource together with a [`D3D12_HEAP_TYPE_UPLOAD`](resources.md#heap-types) heap, you must set *InitialResourceState* to [`D3D12_RESOURCE_STATE_GENERIC_READ`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states).

When you create a resource together with a [`D3D12_HEAP_TYPE_READBACK`](resources.md#heap-types) heap, you must set *InitialResourceState* to [`D3D12_RESOURCE_STATE_COPY_DEST`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states).

#### `pOptimizedClearValue`

Specifies a [`D3D12_CLEAR_VALUE`](resources.md#optimized-clear-value) structure that describes the default value for a clear color.

*pOptimizedClearValue* specifies a value for which clear operations are most optimal. When the created resource is a texture with either the [`D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET`](resources.md#resource-flags) or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` flags, you should choose the value with which the clear operation will most commonly be called. You can call the clear operation with other values, but those operations won't be as efficient as when the value matches the one passed in to resource creation.

When you use [`D3D12_RESOURCE_DIMENSION_BUFFER`](resources.md#resource-dimensions), you must set *pOptimizedClearValue* to `nullptr`.

#### `riidResource`

A reference to the globally unique identifier (**GUID**) of the resource interface to return in *ppvResource*.

While *riidResource* is most commonly the **GUID** of [ID3D12Resource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12resource), it may be the **GUID** of any interface. If the resource object doesn't support the interface for this **GUID**, then creation fails with `E_NOINTERFACE`.

#### `ppvResource`

An optional pointer to a memory block that receives the requested interface pointer to the created resource object.

*ppvResource* can be `nullptr`, for parameter validation. When *ppvResource* is `nullptr`, no object is created, and `S_FALSE` is returned when the parameters are valid.

### Return values

If an object is created successfully, the method returns `S_OK`; successful validation with a null output pointer returns `S_FALSE`. Otherwise, it returns an [**HRESULT**](https://learn.microsoft.com/en-us/windows/desktop/com/structure-of-com-error-codes)[error code](https://learn.microsoft.com/en-us/windows/win32/com/com-error-codes-10).

| Return value | Description |
| --- | --- |
| `E_OUTOFMEMORY` | There is insufficient memory to create the resource. |

See [Direct3D 12 return codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

### Storage, residency and lifetime

This method creates both a resource and a heap, such that the heap is big enough to contain the entire resource, and the resource is mapped to the heap. The created heap is known as an implicit heap, because the heap object can't be obtained by the application. Before releasing the final reference on the resource, your application must ensure that the GPU will no longer read nor write to this resource.

Unless `D3D12_HEAP_FLAG_CREATE_NOT_RESIDENT` is specified, the implicit heap is made resident for GPU access before the method returns control to your application. Also see [Residency](https://learn.microsoft.com/en-us/windows/win32/direct3d12/residency).

The resource GPU VA mapping can't be changed. See [ID3D12CommandQueue::UpdateTileMappings](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings) and [Volume tiled resources](https://learn.microsoft.com/en-us/windows/win32/direct3d12/volume-tiled-resources).

This method may be called by multiple threads concurrently.

<a id="create-placed-resource"></a>

## ID3D12Device::CreatePlacedResource

### Declaration

```cpp
HRESULT CreatePlacedResource(
    ID3D12Heap* pHeap,
    UINT64 HeapOffset,
    const D3D12_RESOURCE_DESC* pDesc,
    D3D12_RESOURCE_STATES InitialState,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    REFIID riid,
    void** ppvResource);
```

### Behavior

Creates a resource that is placed in a specific heap.

### Parameters

#### `pHeap`

A pointer to the **ID3D12Heap** interface that represents the heap in which the resource is placed.

#### `HeapOffset`

The offset, in bytes, to the resource. The *HeapOffset* must be a multiple of the resource's alignment, and *HeapOffset* plus the resource size must be smaller than or equal to the heap size. [**GetResourceAllocationInfo**](resources.md#allocation-info) must be used to understand the sizes of texture resources.

#### `pDesc`

A pointer to a `D3D12_RESOURCE_DESC` structure that describes the resource.

#### `InitialState`

The initial state of the resource, as a bitwise-OR'd combination of `D3D12_RESOURCE_STATES` enumeration constants.

When a resource is created together with a `D3D12_HEAP_TYPE_UPLOAD` heap, *InitialState* must be `D3D12_RESOURCE_STATE_GENERIC_READ`. When a resource is created together with a `D3D12_HEAP_TYPE_READBACK` heap, *InitialState* must be `D3D12_RESOURCE_STATE_COPY_DEST`.

#### `pOptimizedClearValue`

Specifies a [`D3D12_CLEAR_VALUE`](resources.md#optimized-clear-value) that describes the default value for a clear color.

*pOptimizedClearValue* specifies a value for which clear operations are most optimal. When the created resource is a texture with either the `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` flags, your application should choose the value that the clear operation will most commonly be called with.

Clear operations can be called with other values, but those operations will not be as efficient as when the value matches the one passed into resource creation.

*pOptimizedClearValue* must be NULL when used with `D3D12_RESOURCE_DIMENSION_BUFFER`.

#### `riid`

The globally unique identifier (**GUID**) for the resource interface. This is an input parameter.

The **REFIID**, or **GUID**, of the interface to the resource can be obtained by using the `__uuidof` macro. For example, `__uuidof(ID3D12Resource)` gets the **GUID** of the interface to a resource. Although **riid** is, most commonly, the GUID for [**ID3D12Resource**](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12resource), it may be any **GUID** for any interface. If the resource object doesn't support the interface for this **GUID**, then creation fails with `E_NOINTERFACE`.

#### `ppvResource`

A pointer to a memory block that receives a pointer to the resource. *ppvResource* can be NULL, for parameter validation. When *ppvResource* is NULL, no object will be created and `S_FALSE` will be returned when *pDesc* and other parameters are valid.

### Return values

This method returns `E_OUTOFMEMORY` if there is insufficient memory to create the resource. See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

### Storage and initialization

**CreatePlacedResource** is similar to fully mapping a reserved resource to an offset within a heap; but the virtual address space associated with a heap may be reused as well.

A placed resource holds a reference to its heap. The resource's placement does not clear its contents or replace the [metadata initialization and aliasing requirements](synchronization.md#enhanced-aliasing). These obligations are distinct from the optimized clear value supplied at creation and from heap zeroing.

<a id="versioned-creation"></a>

## Versioned creation contributions

The following methods retain the committed/placed/explicit-heap storage models while adding protected sessions, extended descriptions, initial layouts or castable-format lists. Shared resource and heap properties are defined in [Resource descriptions and heap properties](resources.md#resource-description).

<a id="create-heap1"></a>

### ID3D12Device4::CreateHeap1

#### Declaration

```cpp
HRESULT CreateHeap1(
    const D3D12_HEAP_DESC* pDesc,
    ID3D12ProtectedResourceSession *pProtectedSession,
    REFIID riid,
    void** ppvHeap);
```

#### Behavior

Adds `pProtectedSession` to [`CreateHeap`](#create-heap). A null session creates an unprotected heap.

<a id="create-committed-resource1"></a>

### ID3D12Device4::CreateCommittedResource1

#### Declaration

```cpp
HRESULT CreateCommittedResource1(
    const D3D12_HEAP_PROPERTIES* pHeapProperties,
    D3D12_HEAP_FLAGS HeapFlags,
    const D3D12_RESOURCE_DESC* pDesc,
    D3D12_RESOURCE_STATES InitialResourceState,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    ID3D12ProtectedResourceSession *pProtectedSession,
    REFIID riidResource,
    void** ppvResource);
```

#### Behavior

Adds `pProtectedSession` to [`CreateCommittedResource`](#create-committed-resource). A null session creates an unprotected resource.

<a id="create-committed-resource2"></a>

### ID3D12Device8::CreateCommittedResource2

#### Declaration

```cpp
HRESULT CreateCommittedResource2(
    const D3D12_HEAP_PROPERTIES* pHeapProperties,
    D3D12_HEAP_FLAGS HeapFlags,
    const D3D12_RESOURCE_DESC1* pDesc,
    D3D12_RESOURCE_STATES InitialResourceState,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    ID3D12ProtectedResourceSession* pProtectedSession,
    REFIID riidResource,
    void** ppvResource);
```

#### Behavior

Uses [`D3D12_RESOURCE_DESC1`](resources.md#extended-description) and retains the protected-session parameter and legacy initial resource state. The heap, clear-value, interface and output-parameter contracts are those of [`CreateCommittedResource`](#create-committed-resource); protection is specified by `pProtectedSession` as in [`CreateCommittedResource1`](#create-committed-resource1).

<a id="create-placed-resource1"></a>

### ID3D12Device8::CreatePlacedResource1

#### Declaration

```cpp
HRESULT CreatePlacedResource1(
    ID3D12Heap* pHeap,
    UINT64 HeapOffset,
    const D3D12_RESOURCE_DESC1* pDesc,
    D3D12_RESOURCE_STATES InitialState,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    REFIID riid,
    void** ppvResource);
```

#### Behavior

Uses [`D3D12_RESOURCE_DESC1`](resources.md#extended-description) and retains `InitialState`. Placed resources inherit protection from their heap. The heap, offset, clear-value, interface and output-parameter contracts are those of [`CreatePlacedResource`](#create-placed-resource).

<a id="create-committed-resource3"></a>

### ID3D12Device10::CreateCommittedResource3

#### Declaration

```cpp
HRESULT CreateCommittedResource3(
    const D3D12_HEAP_PROPERTIES* pHeapProperties,
    D3D12_HEAP_FLAGS HeapFlags,
    const D3D12_RESOURCE_DESC1* pDesc,
    D3D12_BARRIER_LAYOUT InitialLayout,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    ID3D12ProtectedResourceSession* pProtectedSession,
    UINT32 NumCastableFormats,
    const DXGI_FORMAT *pCastableFormats,
    REFIID riidResource,
    void** ppvResource);
```

#### Behavior

Uses [`D3D12_RESOURCE_DESC1`](resources.md#extended-description), retains `pProtectedSession`, and replaces the legacy initial state with `InitialLayout`. Adds `NumCastableFormats` and `pCastableFormats`. The heap, clear-value, interface and output-parameter contracts are those of [`CreateCommittedResource`](#create-committed-resource); `pProtectedSession` specifies protection as in [`CreateCommittedResource1`](#create-committed-resource1). The initial-layout and format-list constraints are defined [below](#initial-layout-and-format-lists).

<a id="create-placed-resource2"></a>

### ID3D12Device10::CreatePlacedResource2

#### Declaration

```cpp
HRESULT CreatePlacedResource2(
    ID3D12Heap* pHeap,
    UINT64 HeapOffset,
    const D3D12_RESOURCE_DESC1* pDesc,
    D3D12_BARRIER_LAYOUT InitialLayout,
    const D3D12_CLEAR_VALUE* pOptimizedClearValue,
    UINT32 NumCastableFormats,
    const DXGI_FORMAT *pCastableFormats,
    REFIID riid,
    void** ppvResource);
```

#### Behavior

Uses [`D3D12_RESOURCE_DESC1`](resources.md#extended-description) and replaces the legacy initial state with `InitialLayout`. Adds `NumCastableFormats` and `pCastableFormats`. Protection is inherited from the heap. The heap, offset, clear-value, interface and output-parameter contracts are those of [`CreatePlacedResource`](#create-placed-resource). The initial-layout and format-list constraints are defined [below](#initial-layout-and-format-lists).

<a id="protected-creation"></a>

### Protected creation

Protected creation methods for heaps, committed resources, and reserved resources receive a protected session.  Placed resources infer their protection from the heap they are placed on.

<a id="initial-layout-and-format-lists"></a>

### Initial layout and format-list parameters

| Parameter            |                                                                                  |
|----------------------|----------------------------------------------------------------------------------|
| `InitialLayout`      | Initial layout of texture resource, `D3D12_BARRIER_LAYOUT_UNDEFINED` for buffers |
| `NumCastableFormats` | The number of elements in `pCastableFormats`                                     |
| `pCastableFormats`   | A contiguous array of `DXGI_FORMAT`s that this resource may be cast to           |

Creation methods that accept `InitialLayout` require `D3D12_FEATURE_DATA_D3D12_OPTIONS12::EnhancedBarriersSupported`. Buffer resources use `D3D12_BARRIER_LAYOUT_UNDEFINED`; texture resources use a layout compatible with their resource description.

`NumCastableFormats` is the element count of `pCastableFormats`. The [format-list casting contract](formats.md#cast-lists) specifies eligible formats and corresponding view restrictions; this is separate from [fully typed format-family casting](formats.md#casting).

With enhanced-layout creation, a texture with `D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS` uses `D3D12_BARRIER_LAYOUT_COMMON` as its initial layout. The absence of a buffer layout does not remove the upload/readback usage restrictions.

Zero-list interpretation and the resource-own-format exception are defined by the [format-list casting contract](formats.md#cast-lists). Format-list casting requires `D3D12_FEATURE_DATA_D3D12_OPTIONS12::RelaxedFormatCastingSupported`.

<a id="resource-queries"></a>

## Resource description and address queries

<a id="resource-get-desc"></a>

### ID3D12Resource::GetDesc

#### Declaration

```cpp
D3D12_RESOURCE_DESC GetDesc();
```

#### Behavior

Gets the resource description.

#### Return value

A Direct3D 12 resource description structure.

<a id="resource-gpu-address"></a>

### ID3D12Resource::GetGPUVirtualAddress

#### Declaration

```cpp
D3D12_GPU_VIRTUAL_ADDRESS GetGPUVirtualAddress();
```

#### Behavior

This method returns the GPU virtual address of a buffer resource.

This method is only useful for buffer resources, it will return zero for all texture resources.

#### Return value

This method returns the GPU virtual address. `D3D12_GPU_VIRTUAL_ADDRESS` is a typedef'd synonym of UINT64.

A GPU virtual address is not a CPU mapping or a completion indication. Resource and backing-storage lifetime must cover the GPU accesses that use the address; the [execution lifetime rules](execution.md#lifetime) apply.

<a id="allocation-driver"></a>

## Runtime, driver and hardware requirements

<a id="heap-resource-driver"></a>

### Heap/resource driver creation model

The [published `PFND3D12DDI_CREATEHEAPANDRESOURCE_0088` callback](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/d3d12umddi/nc-d3d12umddi-pfnd3d12ddi_createheapandresource_0088) accepts an optional `D3D12DDIARG_CREATEHEAP_0001` and an optional `D3D12DDIARG_CREATERESOURCE_0088`. This supports heap-only, resource-only and combined creation. Its handles distinguish the driver heap, the runtime resource used for callbacks, the protected session, and the driver resource. The combined surface does not imply that every application call maps to one identical driver-call sequence.

The resource and heap creation & destruction methods are able to be called by multiple threads concurrently.

The runtime chooses CPU access characteristics and location preference of the memory segment. The driver must honor the page & segment request exactly. The runtime understands when a UMA design is available, so will not request `_L1` on UMA designs. The runtime also understands when a cache-coherent UMA design is available, so requests for write-combine will be rare. The runtime will never request `WRITE_BACK` with `_L1`. The runtime owns any policy related to the functional disparity between integrated and discrete GPU designs.

`_L0` is required to be in system memory segments.

`_L1` is required to be in video memory segments for discrete GPUs.

Creating a heap is equivalent to creating backing store, just like a section does. The driver is required to call `AllocateCB` during heap creation, supplying `HRTRESOURCE` when associating the allocations with a runtime resource.

The driver is required to call `DeallocateCB` during heap destruction, identifying the resource or allocation handles as appropriate.

<a id="allocation-reflection-driver"></a>

### Allocation reflection and resolved layouts

Creation of a resource still goes through two steps in D3D12. These steps are most obviously needed for textures, but buffers must be supported for orthogonality.

1. First, the sizes and alignments of the resource data, additional data header, and additional data are determined along with the parameterized swizzle patterns, using [`CheckResourceAllocationInfo`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/d3d12umddi/nc-d3d12umddi-pfnd3d12ddi_checkresourceallocationinfo_0088). When the resource description is passed into `CheckResourceAllocationInfo`, the `Layout` of the resource description may be set to `_UNDEFINED`. When the `Layout` of the resource description is `STANDARD_SWIZZLE` or `ROW_MAJOR`, the driver must return the corresponding layout.
2. Second, the resource is created using the properties derived from this method. During resource creation, the `Layout` will never be set to `_UNDEFINED`, since the driver will have returned a resolved swizzle pattern.

This DDI is also used by applications to calculate the size of resources to create appropriately sized heaps, before actually creating any resources.

The driver's texture storage `Layout` here is distinct from the enhanced barrier `InitialLayout` parameter. In particular, the no-layout value required for buffer creation with enhanced barriers does not request an unresolved driver texture layout.

This method enables applications to calculate the size of resources and understand alignment requirements. The size of a resource needs to include any additional data necessary that the application may not expect. For example, depth stencil textures may need additional data for early depth fail optimizations.

This method can be called from multiple threads and must return the same result when the same input resource and flag parameters are passed.

Sizes must be nondecreasing with respect to dimensions. A resource descriptor with the same parameters but greater or equal dimensions must return a size that is greater or equal to the other resource descriptor.

The alignment fields must be a power of two, and the alignment requirements must not be larger than 64KiB, unless the number of MSAA samples is greater than 1.

Resource-optimization flags are intentionally separated from resource creation flags to highlight how they are intended to be used by the driver. They should only influence the driver's choice of device-dependent swizzle pattern. Resource-optimization flags cannot choose a resource layout which would preclude orthogonally supporting such other resource properties, like resource misc flags, in general.

### Tight-alignment DDI requirements

The `AlignmentRestriction` parameter of the DDI is set to the minimum acceptable alignment value based on the [tight-alignment tables](#tight-alignment) during resource creation (varies based on placed vs committed resource requirements, which can't be inferred in the `CheckResourceAllocationInfo` call the way it can in `CreateHeapAndResource`).
Drivers must return an alignment between the `AlignmentRestriction` and the max alignment value in the [tight-alignment tables](#tight-alignment), subject to a valid requested alignment floor.

IMPORTANT: Drivers should size buffers appropriately (`desc.width`) rather than assume 64KiB size when `D3D12DDI_RESOURCE_FLAG_0111_USE_TIGHT_ALIGNMENT` is set.
`CheckExistingResourceAllocationInfo_*` should also return appropriate values for resources that were created with tight alignment.

Starting in DDI 0115, the runtime sets the new `D3D12DDI_HEAP_FLAG_0115_IMPLICIT` bit to true in the `D3D12DDIARG_CREATEHEAP_0001::Flags` parameter of `CreateHeapAndResource` calls for committed resources.

```cpp
typedef enum D3D12DDI_HEAP_FLAGS
{
    // existing heap flags
    D3D12DDI_HEAP_FLAG_0115_IMPLICIT = 0x80,
} D3D12DDI_HEAP_FLAGS;
```

<a id="residency-driver"></a>

### Driver residency responsibilities

The driver `MakeResident` and `Evict` entry points support multiple threads calling into them simultaneously, so the driver must use fine-grained synchronization when appropriate. The driver must forward the flags passed in to both `MakeResident` and `Evict` to the callback routines, and copy the resulting fence value from the callback as an output to these calls.

For a driver `MakeResident` residency batch, drivers must obey the "all or nothing" policy. That is, a call to `MakeResident` must fully succeed, or fully fail, and cannot return to the caller with a partially resident state. The runtime manages return values from kernel in the callback routine, as well as cleanup involved with batching during the runtime's `MakeResident` call. When the driver translates the driver objects to the appropriate `D3DKMT_HANDLE` array for the callback routine, it may process resources that contain more than one allocation per resource. Therefore the driver is required to do one of two things:

- Ensure that the array used to hold the `D3DKMT_HANDLE`s is large enough to accommodate all allocations in one batch. All failure cases will then be handled in the callback routine in the runtime.
- The other option is to batch calls internally to the callback routine. While this is discouraged if the previous method can be done, it may be unavoidable to achieve optimal performance requirements. In this case it is absolutely critical that the driver cleanup all failure cases. This means that if the array is translated to two batches, and the callback routine returns an error on the second batch, the driver must manually call `EvictCb` on the first batch.

The driver batching rule applies to residency work and rollback of its allocation batches. It does not imply that a later failure while submitting an asynchronous completion signal rolls back all earlier successful residency operations.

<a id="resident-destruction-driver"></a>

### Resident-object destruction

The driver must set the `AssumeNotInUse` and `SynchronousDestroy` flags when calling `DeallocateCB` for any DXGK allocations associated with the DDI handles that support residency operations. Heaps and resources are the DDI handle types most commonly associated with residency operations.

The driver should be able to set these flags for all DXGK allocations; but it's possible a few internal UMD allocations may still require different flag settings.

The application is responsible to synchronize the CPU and GPU for many aspects of D3D12: cpu-manipulation of persistently mapped resources, residency management, and all residency object destruction. These flags will destroy resident objects in the most efficient way on D3D12.

The application expects the residency budget to be immediately affected by the destruction of heaps & resources, and keeps tighter control of CPU work with these flags always set. If the flags are not used, a two-second compute task can cause large heap destructions to be wastefully deferred when that compute task is known to not use those heaps. And, DXGK asynchronous processing for deallocation can cause glitches on the render thread.

<a id="hardware-memory-access"></a>

### Hardware memory-access requirements

These hardware requirements describe memory access through supported resource configurations. They do not waive the resource-description, queue, synchronization or CPU-access restrictions defined above.

1. The GPU must read and write data to and from memory pages. When such pages are accessible by the CPU, they must be able to be marked as write-back or write-combine for the CPU.
    1. The GPU must be able to read and write single-dimensional buffer data & multi-dimensional texture data to and from such pages for all GPU operations, like copying, constant reads, vertex reads, index reads, indirect draw reads, buffer & texture loading and sampling, buffer & texture rendering, UAV processing, and even depth-stencil. Resource restrictions and undefined behavior still apply.

The following GPU write operations are well-defined, such that the CPU and/ or other GPU engines can guarantee writes to heaps without losing memory write operations. Bytes within row padding must not be modified.
1. Copy, resolve, stream output, decoder, encoder, and capture GPU-write operations modify only the bytes in the destination resource where each copied texel data resides, as specified by the operation.
2. UAV write operations modify only the bytes in the destination resource in which the modified UAV element resides.

GPUs capable of I/O coherence must use it when write-back is required or chosen for pages that reside in physical system memory.

D3D12 allows metadata on all textures, not just those with `RENDER_TARGET` or `DEPTH_STENCIL` flags. However, any metadata on non-RT/DS textures must not require full subresource copies for initialization. Instead, the metadata must be robust enough to honor the app initializing each texel, discretely, from all possible initial physical memory values.
