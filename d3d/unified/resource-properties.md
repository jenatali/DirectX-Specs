# Resource descriptions and heap properties

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

Specifies one member of [`D3D12_TEXTURE_LAYOUT`](resource-properties.md#texture-layout).

### `Flags`

Bitwise-OR'd flags, as [`D3D12_RESOURCE_FLAGS`](resource-properties.md#resource-flags) enumeration constants.

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

Textures are a multi-dimensional arrangement of texels in a contiguous region of memory, heavily optimized to maximize bandwidth for rendering and sampling. Texture sizes are hard to predict and vary from adapter to adapter. Applications must use [ID3D12Device::GetResourceAllocationInfo](resource-creation.md#allocation-info) to accurately understand their size.
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

Tight-alignment allocation rules and alignment floors are defined in [Tight resource alignment](resource-creation.md#tight-alignment).

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
- Can't be set in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_ROW_MAJOR`](resource-properties.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::CrossAdapterRowMajorTextureSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`, nor in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`](resource-properties.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::StandardSwizzle64KBSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`.
- Can't be used with 4KB alignment without `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`, nor usage with heaps that have [`D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES`](resource-properties.md#heap-flags).

### D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL

Allows a depth stencil view to be created for the resource, as well as enables the resource to transition into the state of [`D3D12_RESOURCE_STATE_DEPTH_WRITE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) and/or `D3D12_RESOURCE_STATE_DEPTH_READ`. Most adapter architectures allocate extra memory for textures with this flag to reduce the effective bandwidth, and maximize optimizations for early depth-test. Your application should avoid setting this flag when depth operations will never occur.

The following restrictions and interactions apply:
- Either the texture format must support depth stencil capabilities at the current feature level. Or, when the format is a typeless format, a format within the same typeless group must support depth stencil capabilities at the current feature level.
- Can't be used with [`D3D12_RESOURCE_DIMENSION_BUFFER`](resources.md#resource-dimensions), 4KB alignment without `D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS`, `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS`, [`D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`](resource-properties.md#texture-layout), `D3D12_TEXTURE_LAYOUT_ROW_MAJOR`, nor used with heaps that have [`D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES`](resource-properties.md#heap-flags) or `D3D12_HEAP_FLAG_ALLOW_DISPLAY`.
- Precludes usage of [WriteToSubresource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12resource-writetosubresource) and [ReadFromSubresource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12resource-readfromsubresource).
- Precludes GPU copying of a subregion. [CopyTextureRegion](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-copytextureregion) must copy a whole subresource to or from resources with this flag.

### D3D12_RESOURCE_FLAG_ALLOW_UNORDERED_ACCESS

Allows an unordered access view to be created for the resource, as well as enables the resource to transition into the state of [`D3D12_RESOURCE_STATE_UNORDERED_ACCESS`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states). Some adapter architectures must resort to less efficient texture layouts in order to provide this functionality. If a texture is rarely used for unordered access, then it might be worth having two textures around and copying between them. One texture would have this flag, while the other wouldn't. Your application should avoid setting this flag when unordered access operations will never occur.

The following restrictions and interactions apply:
- Either the texture format must support unordered access capabilities at the current feature level. Or, when the format is a typeless format, a format within the same typeless group must support unordered access capabilities at the current feature level.
- Can't be set in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_ROW_MAJOR`](resource-properties.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::CrossAdapterRowMajorTextureSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`, nor in conjunction with textures that have [`D3D12_TEXTURE_LAYOUT_64KB_STANDARD_SWIZZLE`](resource-properties.md#texture-layout) when [`D3D12_FEATURE_DATA_D3D12_OPTIONS::StandardSwizzle64KBSupported`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_feature_data_d3d12_options) is `FALSE`, nor when the feature level is less than 11.0.
- Use with MSAA textures requires [writable-MSAA texture support](#writable-msaa).

### D3D12_RESOURCE_FLAG_DENY_SHADER_RESOURCE

Disallows a shader resource view from being created for the resource, as well as disables the resource from transitioning into the state of [`D3D12_RESOURCE_STATE_NON_PIXEL_SHADER_RESOURCE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) or `D3D12_RESOURCE_STATE_PIXEL_SHADER_RESOURCE`. Some adapter architectures gain bandwidth capacity for depth stencil textures when shader resource views are precluded. If a texture is rarely used for shader resources, then it might be worth having two textures around and copying between them. One texture would have this flag, while the other wouldn't. Your application should set this flag when depth stencil textures will never be used from shader resource views.

The following restrictions and interactions apply:
- Must be used with `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`.

### D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER

Allows the resource to be used for cross-adapter data, as well as those features enabled by `D3D12_RESOURCE_FLAGS::D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS`. Cross-adapter resources commonly preclude techniques that reduce effective texture bandwidth during usage, and some adapter architectures might require different caching behavior. Your application should avoid setting this flag when the resource data will never be used with another adapter.

The following restrictions and interactions apply:
- Must be used with heaps that have [`D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER`](resource-properties.md#heap-flags).
- Can't be used with heaps that have [`D3D12_HEAP_FLAG_ALLOW_DISPLAY`](resource-properties.md#heap-flags).

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

`D3D12_RESOURCE_FLAG_USE_TIGHT_ALIGNMENT` selects the [tight-alignment allocation contract](resource-creation.md#tight-alignment); the device must report support before the flag can be used.

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
    - No [`D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`](resource-properties.md#resource-flags).
    - The format cannot be a YUV format.
- The texture must be created on a heap with [`D3D12_HEAP_FLAG_SHARED_CROSS_ADAPTER`](resource-properties.md#heap-flags).

Buffers are created with [`D3D12_TEXTURE_LAYOUT_ROW_MAJOR`](resource-properties.md#texture-layout), because row-major texture data can be located in them without creating a texture object.

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
- [`D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`](resource-properties.md#resource-flags)
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

A [`D3D12_HEAP_TYPE`](resource-properties.md#heap-types)-typed value that specifies the type of heap.

### `CPUPageProperty`

A [`D3D12_CPU_PAGE_PROPERTY`](resource-properties.md#cpu-page-property)-typed value that specifies the CPU-page properties for the heap.

### `MemoryPoolPreference`

A [`D3D12_MEMORY_POOL`](resource-properties.md#memory-pools)-typed value that specifies the memory pool for the heap.

### `CreationNodeMask`

For multi-adapter operation, this indicates the node where the resource should be created.

Exactly one bit of this UINT must be set. See [Multi-adapter systems](https://learn.microsoft.com/en-us/windows/win32/direct3d12/multi-engine).

Passing zero is equivalent to passing one, in order to simplify the usage of single-GPU adapters.

### `VisibleNodeMask`

For multi-adapter operation, this indicates the set of nodes where the resource is visible.

*VisibleNodeMask* must have the same bit set that is set in *CreationNodeMask*. *VisibleNodeMask* can *also* have additional bits set for cross-node resources, but doing so can potentially reduce performance for resource accesses, so you should do so only when needed.

Passing zero is equivalent to passing one, in order to simplify the usage of single-GPU adapters.

Valid combinations of struct member values:
- When **Type** is [`D3D12_HEAP_TYPE_CUSTOM`](resource-properties.md#heap-types), **CPUPageProperty** and **MemoryPoolPreference** must not be `D3D12_CPU_PAGE_PROPERTY_UNKNOWN` and `D3D12_MEMORY_POOL_UNKNOWN`, respectively.
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

The size, in bytes, of the heap. To avoid wasting memory, applications should pass *SizeInBytes* values which are multiples of the effective *Alignment*; but non-aligned *SizeInBytes* is also supported, for convenience. To find out how large a heap must be to support textures with undefined layouts and adapter-specific sizes, call [ID3D12Device::GetResourceAllocationInfo](resource-creation.md#allocation-info).

### `Properties`

A [`D3D12_HEAP_PROPERTIES`](resource-properties.md#heap-properties) structure that describes the heap properties.

### `Alignment`

The alignment value for the heap. Valid values:

| Value | Description |
| --- | --- |
| 0 | An alias for 64KB. |
| `D3D12_DEFAULT_RESOURCE_PLACEMENT_ALIGNMENT` | #defined as 64KB. |
| `D3D12_DEFAULT_MSAA_RESOURCE_PLACEMENT_ALIGNMENT` | #defined as 4MB. This alignment is required when a resource's allocation alignment exceeds 64KB. Eligible small or tightly aligned MSAA resources do not by themselves require a 4MB-aligned heap. |

### `Flags`

A combination of [`D3D12_HEAP_FLAGS`](resource-properties.md#heap-flags)-typed values that are combined by using a bitwise-OR operation. The resulting value identifies heap options. When creating heaps to support adapters with resource heap tier 1, an application must choose some flags.

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

The heap is not allowed to contain resources with `D3D12_RESOURCE_DIMENSION_TEXTURE1D`, `D3D12_RESOURCE_DIMENSION_TEXTURE2D`, or `D3D12_RESOURCE_DIMENSION_TEXTURE3D` unless either `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` are present. Refer to [`D3D12_RESOURCE_DIMENSION`](resources.md#resource-dimensions) and [`D3D12_RESOURCE_FLAGS`](resource-properties.md#resource-flags).

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

The heap is created in a non-resident state and must be made resident using [ID3D12Device::MakeResident](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-makeresident) or [ID3D12Device3::EnqueueMakeResident](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device3-enqueuemakeresident).

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

The following heap flags must be used with [ID3D12Device::CreateHeap](resource-creation.md#create-heap), but will be set automatically for implicit heaps created by [ID3D12Device::CreateCommittedResource](resource-creation.md#create-committed-resource). For ordinary non-cross-adapter heaps on [heap tier 1](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_heap_tier), two out of the three following flags must be set.

| Value | Description |
| --- | --- |
| `D3D12_HEAP_FLAG_DENY_BUFFERS` | The heap isn't allowed to contain resources with `D3D12_RESOURCE_DIMENSION_BUFFER` (which is a [`D3D12_RESOURCE_DIMENSION`](resources.md#resource-dimensions) enumeration constant). |
| `D3D12_HEAP_FLAG_DENY_RT_DS_TEXTURES` | The heap isn't allowed to contain resources with `D3D12_RESOURCE_DIMENSION_TEXTURE1D`, `D3D12_RESOURCE_DIMENSION_TEXTURE2D`, or `D3D12_RESOURCE_DIMENSION_TEXTURE3D` together with either `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET` or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`. (The latter two items are [`D3D12_RESOURCE_FLAGS`](resource-properties.md#resource-flags) enumeration constants.) |
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

Applications may create displayable heaps outside of a swapchain; but cannot actually present with them. This flag is not supported by [CreateHeap](resource-creation.md#create-heap) and can only be used with [CreateCommittedResource](resource-creation.md#create-committed-resource) with `D3D12_HEAP_TYPE_DEFAULT`.

Additional restrictions to the [`D3D12_RESOURCE_DESC`](resource-properties.md#resource-description) apply to the resource created with displayable heaps.

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
