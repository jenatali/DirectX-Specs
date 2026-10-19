# Resources and memory

<a id="resource-storage"></a>

## Resource identity and storage

Resources are the D3D concept which abstracts the usage of GPU physical memory. Resources require GPU virtual address space to access physical memory.

A resource describes GPU-accessible data; a heap supplies its backing storage. Resource descriptions, heap properties, and the relationship between them are distinct from command-list state and resource synchronization.

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

Placement requirements are defined by [`CreatePlacedResource`](resource-creation.md#create-placed-resource). Overlapping allocations do not by themselves establish synchronization or initialize texture metadata; the [legacy and enhanced metadata-initialization requirements](../D3D12EnhancedBarriers.md#placed-resource-metadata-initialization) apply.

<a id="reserved-resources"></a>

### Reserved resources

On their creation, only a virtual address range is allocated and not mapped to any heap. The application will map such resources to heaps later. Such resources can be mapped to a heap at a 64KB tile granularity with `UpdateTileMappings`.

<a id="resource-contracts"></a>

## Descriptions, creation and synchronization

[Resource descriptions and heap properties](resource-properties.md) define dimensions, storage layouts, usage flags, clear-value optimization, CPU page properties and node visibility. [Resource creation and allocation](resource-creation.md) defines allocation information and the creation methods.

A resource's storage layout is distinct from its current legacy state or enhanced barrier layout. [Synchronization and barriers](synchronization.md) owns resource-state promotion/decay; enhanced access/layout requirements are defined in [Enhanced Barriers](../D3D12EnhancedBarriers.md#layout-transitions).
