# Resource creation and allocation

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

When you're using [CreatePlacedResource](resource-creation.md#create-placed-resource), your application must use **GetResourceAllocationInfo** in order to understand the size and alignment characteristics of texture resources. The results of this method vary depending on the particular adapter, and must be treated as unique to this adapter and driver version.

Your application can't use the output of **GetResourceAllocationInfo** to understand packed mip properties of textures. To understand packed mip properties of textures, your application must use [GetResourceTiling](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-getresourcetiling).

Texture resource sizes significantly differ from the information returned by **GetResourceTiling**, because some adapter architectures allocate extra memory for textures to reduce the effective bandwidth during common rendering scenarios. This even includes textures that have constraints on their texture layouts, or have standardized texture layouts. That extra memory can't be sparsely mapped nor remapped by an application using [CreateReservedResource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12device-createreservedresource) and [UpdateTileMappings](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-updatetilemappings), so it isn't reported by **GetResourceTiling**.

Your application can forgo using **GetResourceAllocationInfo** for buffer resources ([`D3D12_RESOURCE_DIMENSION_BUFFER`](resources.md#resource-dimensions)) when not using [Tight Alignment](resource-creation.md#tight-alignment). Buffers that are not tightly aligned have the same size on all adapters, which is merely the smallest multiple of 64KB that's greater or equal to [`D3D12_RESOURCE_DESC::Width`](resource-properties.md#resource-description).

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

When you create a resource together with a [`D3D12_HEAP_TYPE_UPLOAD`](resource-properties.md#heap-types) heap, you must set *InitialResourceState* to [`D3D12_RESOURCE_STATE_GENERIC_READ`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states).

When you create a resource together with a [`D3D12_HEAP_TYPE_READBACK`](resource-properties.md#heap-types) heap, you must set *InitialResourceState* to [`D3D12_RESOURCE_STATE_COPY_DEST`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states).

#### `pOptimizedClearValue`

Specifies a [`D3D12_CLEAR_VALUE`](resource-properties.md#optimized-clear-value) structure that describes the default value for a clear color.

*pOptimizedClearValue* specifies a value for which clear operations are most optimal. When the created resource is a texture with either the [`D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET`](resource-properties.md#resource-flags) or `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL` flags, you should choose the value with which the clear operation will most commonly be called. You can call the clear operation with other values, but those operations won't be as efficient as when the value matches the one passed in to resource creation.

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

The offset, in bytes, to the resource. The *HeapOffset* must be a multiple of the resource's alignment, and *HeapOffset* plus the resource size must be smaller than or equal to the heap size. [**GetResourceAllocationInfo**](resource-creation.md#allocation-info) must be used to understand the sizes of texture resources.

#### `pDesc`

A pointer to a `D3D12_RESOURCE_DESC` structure that describes the resource.

#### `InitialState`

The initial state of the resource, as a bitwise-OR'd combination of `D3D12_RESOURCE_STATES` enumeration constants.

When a resource is created together with a `D3D12_HEAP_TYPE_UPLOAD` heap, *InitialState* must be `D3D12_RESOURCE_STATE_GENERIC_READ`. When a resource is created together with a `D3D12_HEAP_TYPE_READBACK` heap, *InitialState* must be `D3D12_RESOURCE_STATE_COPY_DEST`.

#### `pOptimizedClearValue`

Specifies a [`D3D12_CLEAR_VALUE`](resource-properties.md#optimized-clear-value) that describes the default value for a clear color.

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

A placed resource holds a reference to its heap. The resource's placement does not clear its contents or replace the [metadata initialization and aliasing requirements](../D3D12EnhancedBarriers.md#placed-resource-metadata-initialization). These obligations are distinct from the optimized clear value supplied at creation and from heap zeroing.

<a id="versioned-creation"></a>

## Versioned creation contributions

The following methods retain the committed/placed/explicit-heap storage models while adding protected sessions, extended descriptions, initial layouts or castable-format lists. Shared resource and heap properties are defined in [Resource descriptions and heap properties](resource-properties.md).

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

Uses [`D3D12_RESOURCE_DESC1`](resource-properties.md#extended-description) and retains the protected-session parameter and legacy initial resource state. The heap, clear-value, interface and output-parameter contracts are those of [`CreateCommittedResource`](#create-committed-resource); protection is specified by `pProtectedSession` as in [`CreateCommittedResource1`](#create-committed-resource1).

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

Uses [`D3D12_RESOURCE_DESC1`](resource-properties.md#extended-description) and retains `InitialState`. Placed resources inherit protection from their heap. The heap, offset, clear-value, interface and output-parameter contracts are those of [`CreatePlacedResource`](#create-placed-resource).

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

Uses [`D3D12_RESOURCE_DESC1`](resource-properties.md#extended-description), retains `pProtectedSession`, and replaces the legacy initial state with `InitialLayout`. Adds `NumCastableFormats` and `pCastableFormats`. The heap, clear-value, interface and output-parameter contracts are those of [`CreateCommittedResource`](#create-committed-resource); `pProtectedSession` specifies protection as in [`CreateCommittedResource1`](#create-committed-resource1). The initial-layout and format-list constraints are defined [below](#initial-layout-and-format-lists).

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

Uses [`D3D12_RESOURCE_DESC1`](resource-properties.md#extended-description) and replaces the legacy initial state with `InitialLayout`. Adds `NumCastableFormats` and `pCastableFormats`. Protection is inherited from the heap. The heap, offset, clear-value, interface and output-parameter contracts are those of [`CreatePlacedResource`](#create-placed-resource). The initial-layout and format-list constraints are defined [below](#initial-layout-and-format-lists).

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

`NumCastableFormats` is the element count of `pCastableFormats`. The [format-list casting contract](../VulkanOn12.md#format-list-casting) specifies eligible formats and corresponding view restrictions; this is separate from [fully typed format-family casting](../RelaxedCasting.md#casting-rules-for-rs2-drivers).

With enhanced-layout creation, a texture with `D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS` uses `D3D12_BARRIER_LAYOUT_COMMON` as its initial layout. The absence of a buffer layout does not remove the upload/readback usage restrictions.

A zero-count, null castable-format list retains ordinary format-casting rules. With an explicit list, other view formats must satisfy that list's contract; the resource's own format need not be repeated in the list. Format-list casting requires `D3D12_FEATURE_DATA_D3D12_OPTIONS12::RelaxedFormatCastingSupported`.

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

## Driver allocation responsibilities

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
