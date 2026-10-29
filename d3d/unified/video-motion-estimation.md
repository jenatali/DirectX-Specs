<a id="video-motion-estimation"></a>

# Video: motion estimation reference

This subordinate [Video](video.md) reference describes motion estimation,
opaque motion-vector storage and its resolution, together with the base
VIDEO_ENCODE command-list lifecycle. A motion estimator maintains operation
context; a motion-vector heap stores hardware-dependent output; a resolve
records conversion into an application texture.

This is a **qualified draft**. Published motion-search requirements remain
source-only where algorithm, hardware-quality or implementation evidence is
insufficient. Specific qualifications below do not establish expert or
hardware approval.

<a id="motion-model"></a>

## Motion search and execution

This feature exposes a motion search operation to produce motion vectors for the purpose of extrapolating future frames.

The operation performs a luma-only motion search between the current input
frame and a reference (past) frame. Microsoft does not specify the details
of the search algorithm. Search block sizes are 8×8 or 16×16 pixels, as
reported for the node and input format; the selected vector precision is
quarter-pixel. The block size is not a search-window radius. There is no
published fixed search radius, tie-breaking rule or guaranteed VR quality
in this contract. A prior heap may supply hints, not mandatory search results.

The compact route is [capabilities](#motion-capabilities),
[objects](#motion-objects), [recording](#motion-recording), and the subordinate
[type](#motion-types) and [driver](#motion-driver) references.

Motion estimation uses a list and allocator of
`D3D12_COMMAND_LIST_TYPE_VIDEO_ENCODE`, submitted to a VIDEO_ENCODE queue.
[CPU threading](execution.md#threading),
[submission threading](execution.md#submission-threading),
[queue/list types](execution.md#command-list-types),
[creation](execution.md#creation) and [submission validation](execution.md#validation)
own device/node and list/allocator/queue compatibility.
[Submission and ordering](execution.md#submission) own execution order.
The motion estimator's explicit exclusion of concurrent use on distinct queues
is defined under [driver object lifetime](#motion-driver-objects); it is not
a decoder-wide rule or a restriction to VIDEO_DECODE.

[Recording storage](execution.md#command-storage),
[object/resource lifetime](execution.md#lifetime),
[completion and lifetime](execution.md#completion-lifetime), and
[video resources](video.md#video-resource-contract) own storage, completion
and residency. Both motion objects derive from `ID3D12Pageable`.
The estimator, output and hint heaps, and input, reference and resolved-output
resources must remain alive for their recorded uses that will execute and
until those GPU uses finish. CPU descriptions and argument structures are
recording inputs, not GPU picture storage. Resetting the list does not complete
earlier GPU work or reset its allocator. Caller-owned interface references do
not imply that recording motion commands automatically takes additional
references to those objects.

Input and reference pictures are read in motion estimation. The legacy states
are `D3D12_RESOURCE_STATE_VIDEO_ENCODE_READ` for those pictures and
`D3D12_RESOURCE_STATE_VIDEO_ENCODE_WRITE` for the resolve destination texture.
The opaque heap is not an `ID3D12Resource` to which an application applies
resource barriers. [Enhanced encode barriers](synchronization.md#encode-enhanced-barrier)
and [legacy encode barriers](synchronization.md#encode-resource-barrier)
retain their full declarations; [access](synchronization.md#barrier-access),
[synchronization scopes](synchronization.md#barrier-sync), and
[barrier-model interoperation](synchronization.md#model-interoperation) own
the enhanced path. Neither motion call inserts an application resource
transition or a cross-queue fence wait.

[Encode predication](counters-queries-and-predication.md#videoencode-setpredication)
and the [operation matrix](counters-queries-and-predication.md#predication-operation-matrix)
own conditional execution: both `EstimateMotion` and `ResolveMotionVectorHeap`
are predicated. [Encode queries](counters-queries-and-predication.md#videoencode-beginquery),
[timestamp ending](counters-queries-and-predication.md#videoencode-endquery), and
[query resolution](counters-queries-and-predication.md#videoencode-resolvequerydata)
own the query/list/heap combinations and lifecycle. The encode route supports
timestamp queries, not a motion-estimation statistics or completion query.

<a id="motion-capabilities"></a>

## Capability queries

[`ID3D12VideoDevice::CheckFeatureSupport`](video-decode.md#video-check-feature-support)
owns the common declaring entry and return protocol. Each query below uses
a fixed payload and `FeatureSupportDataSize` equal to `sizeof` that payload.
Initialize input fields and read output fields only on success.

| `D3D12_FEATURE_VIDEO` case | Value | Payload | Inputs | Outputs |
|---|---|---|---|---|
| `FEATURE_AREA_SUPPORT` | 19 | `D3D12_FEATURE_DATA_VIDEO_FEATURE_AREA_SUPPORT` | `NodeIndex` | `VideoDecodeSupport`, `VideoProcessSupport`, `VideoEncodeSupport` |
| `MOTION_ESTIMATOR` | 20 | `D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR` | `NodeIndex`, `InputFormat` | `BlockSizeFlags`, `PrecisionFlags`, `SizeRange` |
| `MOTION_ESTIMATOR_SIZE` | 21 | `D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_SIZE` | `NodeIndex`, `InputFormat`, `BlockSize`, `Precision`, `SizeRange`, `Protected` | Four estimator/heap pool-size fields |
| `MOTION_ESTIMATOR_PROTECTED_RESOURCES` | 30 | `D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES` | `NodeIndex` | `SupportFlags` |

The case names in this table have prefix `D3D12_FEATURE_VIDEO_`.
`NodeIndex` is a zero-based index, not the creation `NodeMask`.
The [shared feature-area driver contract](#video-feature-area-support)
defines the decode, process and encode command-family obligations.
Feature-area encode support makes the queue/list family available; it is
not itself support for every motion-estimation format or block size.
`MOTION_ESTIMATOR` establishes the supported tuple: at least one block-size
bit and precision bit must be set, and the size range must contain the inputs.
For an unsupported format both flags are `NONE` and every size-range field
is zero; a successful query alone is not support.
The selected baseline input/reference format is `DXGI_FORMAT_NV12`.

The size query reports residency allocations for both the context and opaque
output heap using the supplied creation parameters. It does not allocate
either object. The heap and estimator size outputs have distinct names;
their API field order is not the DDI field order.
`Protected` is TRUE for protected input/output operation and FALSE otherwise.
Narrow protected creation additionally requires motion-estimator protected
resource support on that node; a non-null creation session is not an ordinary
unprotected object.

**Scoped runtime validation:** an out-of-range query node returns
`E_INVALIDARG`. The inspected support path returns no-support outputs for
an unavailable encode route or disallowed input format. Invalid driver
capability flags, inconsistent support outputs or malformed size ranges can
produce `DXGI_ERROR_DRIVER_INTERNAL_ERROR`. The size path rejects a driver
report with both motion-vector-heap pool sizes zero. These source-inspected
checks do not prove an adapter's capabilities or search quality.

<a id="motion-objects"></a>

## Object creation and reflection

Both creation descriptions are input-only. Their `NodeMask` selects one
physical node: zero for single-GPU operation, or one bit for the selected
node in multi-node operation. Input format, block size, precision and size
range must be supported for that node. The selected size range may be a
subset of supported sizes to reduce allocation requirements; it must
accommodate each actual input and reference.

On successful creation, `ppVideoMotionEstimator` or
`ppVideoMotionVectorHeap` receives a caller-owned reference to the requested
interface identified by `riid`. The caller releases that reference under the
common [object lifetime rules](execution.md#lifetime). Each creation method
copies its CPU description during the call; the caller need not retain the
description's storage after the call returns.

Each `GetDesc` returns a value copy of the immutable creation description.
Changing that returned copy does not reconfigure the object. Different
creation parameters require a new object.

`pProtectedResourceSession` may be null for unprotected operation.
For protected input, the estimator and vector heap must be created with a
protected resource session and the output must be protected. These are
narrow object-association requirements, not an independent session lifecycle.
On success, each `GetProtectedResourceSession` returns an owned reference to
the requested session interface through `ppProtectedSession`, identified by
`riid`. The caller releases that reference under the same object lifetime
rules; retrieving the interface does not create a new protection session.

**Scoped creation validation:** the inspected creation paths validate the
node, size range, reported block-size and precision bits and protected support.
They propagate validation or creation failures as HRESULTs. A null result
pointer has a validation-only path returning `S_FALSE` on successful validation;
it creates no object. This observed path does not change the required output
pointer annotation in the released declaration.

**Session-getter qualification:** the published getters retrieve the creation
session and return HRESULT. The precise no-session failure and null-output
postcondition are not established by the selected owning references; no
specific success/failure code for that case is asserted here.

<a id="motion-create-estimator"></a>

### ID3D12VideoDevice1::CreateVideoMotionEstimator

#### Declaration

```cpp
HRESULT CreateVideoMotionEstimator(
  const D3D12_VIDEO_MOTION_ESTIMATOR_DESC *pDesc,
  ID3D12ProtectedResourceSession          *pProtectedResourceSession,
  REFIID                                  riid,
  void                                    **ppVideoMotionEstimator
);
```

#### Behavior

Creates an [ID3D12VideoMotionEstimator](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionestimator), which maintains context for video motion estimation operations.

#### Parameters

`pDesc`

A [D3D12_VIDEO_MOTION_ESTIMATOR_DESC](#d3d12-video-motion-estimator-desc) describing the parameters used for motion estimation.

`pProtectedResourceSession`

A [ID3D12ProtectedResourceSession](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12protectedresourcesession) for managing access to protected resources.

`riid`

The globally unique identifier (GUID) for the **ID3D12VideoMotionEstimator** interface.

`ppVideoMotionEstimator`

A pointer to a memory block that receives a pointer to the **ID3D12VideoMotionEstimator** interface.

#### Return values

This method returns HRESULT.

<a id="motion-create-heap"></a>

### ID3D12VideoDevice1::CreateVideoMotionVectorHeap

#### Declaration

```cpp
HRESULT CreateVideoMotionVectorHeap(
  const D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC *pDesc,
  ID3D12ProtectedResourceSession            *pProtectedResourceSession,
  REFIID                                    riid,
  void                                      **ppVideoMotionVectorHeap
);
```

#### Behavior

Allocates a heap that contains motion vectors for video motion estimation.

#### Parameters

`pDesc`

A pointer to a [D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC](#d3d12-video-motion-vector-heap-desc) describing the format of the heap.

`pProtectedResourceSession`

A [ID3D12ProtectedResourceSession](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12protectedresourcesession) for managing access to protected resources.

`riid`

The globally unique identifier (GUID) for the **ID3D12VideoMotionVectorHeap** interface.

`ppVideoMotionVectorHeap`

A pointer to a memory block that receives a pointer to the **ID3D12VideoMotionVectorHeap** interface.

#### Return values

This method returns HRESULT.

<a id="motion-estimator-get-desc"></a>

### ID3D12VideoMotionEstimator::GetDesc

#### Declaration

```cpp
D3D12_VIDEO_MOTION_ESTIMATOR_DESC GetDesc();
```

#### Behavior

Gets the [D3D12_VIDEO_MOTION_ESTIMATOR_DESC](#d3d12-video-motion-estimator-desc) structure that was passed into [ID3D12VideoDevice1::CreateVideoMotionEstimator](#motion-create-estimator) when the [ID3D12VideoMotionEstimator](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionestimator) was created.

#### Return values

This method returns a **D3D12\_VIDEO\_MOTION\_ESTIMATOR\_DESC** structure.

<a id="motion-estimator-get-session"></a>

### ID3D12VideoMotionEstimator::GetProtectedResourceSession

#### Declaration

```cpp
HRESULT GetProtectedResourceSession(
  REFIID riid,
  void   **ppProtectedSession
);
```

#### Behavior

Gets the [ID3D12ProtectedResourceSession](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12protectedresourcesession) that was passed into [ID3D12VideoDevice1::CreateVideoMotionEstimator](#motion-create-estimator) when the [ID3D12VideoMotionEstimator](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionestimator) was created.

#### Parameters

`riid`

The globally unique identifier (GUID) for the **ID3D12ProtectedResourceSession** interface.

`ppProtectedSession`

Receives a void pointer representing the **ID3D12ProtectedResourceSession** interface.

#### Return values

This method returns HRESULT.

<a id="motion-heap-get-desc"></a>

### ID3D12VideoMotionVectorHeap::GetDesc

#### Declaration

```cpp
D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC GetDesc();
```

#### Behavior

Gets the [D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC](#d3d12-video-motion-vector-heap-desc) structure that was passed into [ID3D12VideoDevice1::CreateVideoMotionVectorHeap](#motion-create-heap) when the [ID3D12VideoMotionVectorHeap](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionvectorheap) was created.

#### Return values

This method returns a **D3D12\_VIDEO\_MOTION\_VECTOR\_HEAP\_DESC** structure.

<a id="motion-heap-get-session"></a>

### ID3D12VideoMotionVectorHeap::GetProtectedResourceSession

#### Declaration

```cpp
HRESULT GetProtectedResourceSession(
  REFIID riid,
  void   **ppProtectedSession
);
```

#### Behavior

Gets the [ID3D12ProtectedResourceSession](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12protectedresourcesession) that was passed into [ID3D12VideoDevice1::CreateVideoMotionVectorHeap](#motion-create-heap) when the [ID3D12VideoMotionVectorHeap](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionvectorheap) was created.

#### Parameters

`riid`

The globally unique identifier (GUID) for the **ID3D12ProtectedResourceSession** interface.

`ppProtectedSession`

Receives a void pointer representing the **ID3D12ProtectedResourceSession** interface.

#### Return values

This method returns HRESULT.

<a id="motion-recording"></a>

## Encode-list lifecycle and motion recording

The base encode list has no graphics pipeline-state argument.
`ClearState` resets video recording state without beginning a new recording;
`Reset` starts recording with the supplied encode allocator after a successful
close. `Close` ends recording, not GPU execution. Submission and allocator
reuse follow [recording and storage](execution.md#command-storage) and
[completion](execution.md#completion-lifetime).

**Scoped reset-state evidence:** shared video-list reset and clear-state code
disable predication (null predicate, zero offset, `EQUAL_ZERO`) and clear the
protected-session binding on a protection-capable node. They do not clear
motion-vector heap contents or recreate a motion estimator.
There is no graphics PSO or decoder binding implied by this state reset.

<a id="encode-clear-state"></a>

### ID3D12VideoEncodeCommandList::ClearState

#### Declaration

```cpp
void ClearState();
```

#### Behavior

Resets the state of a video encode command list back to the state it was in when the command list was created.

#### Return values

None

<a id="encode-close"></a>

### ID3D12VideoEncodeCommandList::Close

#### Declaration

```cpp
HRESULT Close();
```

#### Behavior

Indicates that recording to the command list has finished.

#### Return values

Type: **[HRESULT](https://learn.microsoft.com/en-us/windows/win32/com/structure-of-com-error-codes)**

Returns **S\_OK** if successful; otherwise, returns one of the following values:

- **E\_FAIL** if the command list has already been closed, or an invalid API was called during command list recording.
- **E\_OUTOFMEMORY** if the operating system ran out of memory during recording.
- **E\_INVALIDARG** if an invalid argument was passed to the command list API during recording.

See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

#### Runtime validation

The runtime will validate that the command list has not previously been closed. If an error was encountered during recording, the error code is returned here. The runtime won't call the close device driver interface (DDI) in this case.

<a id="encode-reset"></a>

### ID3D12VideoEncodeCommandList::Reset

#### Declaration

```cpp
HRESULT Reset(
  ID3D12CommandAllocator *pAllocator
);
```

#### Behavior

Resets a command list back to its initial state as if a new command list was just created.

#### Parameters

`pAllocator`

Type: **ID3D12CommandAllocator\***

A pointer to the [ID3D12CommandAllocator](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12commandallocator) object that the device creates command lists from.

#### Return values

Type: **[HRESULT](https://learn.microsoft.com/en-us/windows/win32/com/structure-of-com-error-codes)**

Returns **S\_OK** if successful; otherwise, returns one of the following values:

- **E\_FAIL** if the command list was not in the "closed" state when the **Reset** call was made, or the per-device limit would have been exceeded.
- **E\_OUTOFMEMORY** if the operating system ran out of memory.
- **E\_INVALIDARG** if the allocator is currently being used with another command list in the "recording" state or if the specified allocator was created with the wrong type.

 See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

#### Recording storage

The allocator must have type `D3D12_COMMAND_LIST_TYPE_VIDEO_ENCODE`.
The list can be reset while an earlier execution is in flight, but the
allocator's memory cannot be reclaimed until its GPU uses finish. A recording
error returned by `Close` is not repaired by `Reset`; the shared
[recording lifecycle](execution.md#recording-lifecycle) owns failed-recording handling.

<a id="motion-estimate"></a>

### ID3D12VideoEncodeCommandList::EstimateMotion

#### Declaration

```cpp
void EstimateMotion(
  ID3D12VideoMotionEstimator                *pMotionEstimator,
  const D3D12_VIDEO_MOTION_ESTIMATOR_OUTPUT *pOutputArguments,
  const D3D12_VIDEO_MOTION_ESTIMATOR_INPUT  *pInputArguments
);
```

#### Behavior

Performs the motion estimation operation.

#### Parameters

`pMotionEstimator`

An [ID3D12VideoMotionEstimator](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionestimator) representing the video motion estimator context object.

`pOutputArguments`

A [D3D12_VIDEO_MOTION_ESTIMATOR_OUTPUT](#d3d12-video-motion-estimator-output) structure representing the video motion estimation output arguments.

`pInputArguments`

A [D3D12_VIDEO_MOTION_ESTIMATOR_INPUT](#d3d12-video-motion-estimator-input) structure representing the video motion estimation input arguments.

#### Return values

None

#### Operation and runtime validation

`pMotionEstimator` supplies the context; `pOutputArguments->pMotionVectorHeap`
receives opaque output. `pInputArguments` selects the whole current frame,
the reference frame and an optional previous heap for hints. Input and
reference subresource indices select the base plane of the chosen array slice.
The estimator and output heap must agree in input format, block size and
precision. Input/reference dimensions must agree and fit the objects'
creation ranges.

**Scoped recording validation:** the inspected path requires non-reserved 2D
input/reference textures, one mip level, valid subresources selecting plane
zero, supported video heap properties, matching input format and matching
input/reference dimensions. It checks actual input size against the output
heap's range. This does not independently establish a complete estimator-range,
reference-format, hint-compatibility or aliasing validation contract.
Estimator/output-heap node mismatches emit validation diagnostics, but the
selected function does not set a failure HRESULT in those branches.
The caller still must satisfy the node associations and creation contracts.
Protected-session compatibility is checked for the estimator, output heap,
both pictures and a non-null hint heap.

Invalid motion arguments can invalidate recording; these void methods do not
return an immediate HRESULT. The inspected path records an error for later
`Close`. Diagnostic presence or absence is not a hardware-validity guarantee.

<a id="motion-resolve"></a>

### ID3D12VideoEncodeCommandList::ResolveMotionVectorHeap

#### Declaration

```cpp
void ResolveMotionVectorHeap(
  const D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT *pOutputArguments,
  const D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT  *pInputArguments
);
```

#### Behavior

Translates the motion vector output of the [EstimateMotion](#motion-estimate) method from hardware-dependent formats into a consistent format defined by the video motion estimation APIs.

#### Parameters

`pOutputArguments`

A [D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT](#d3d12-resolve-video-motion-vector-heap-output) structure containing the translated motion vectors.

`pInputArguments`

A [D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT](#d3d12-resolve-video-motion-vector-heap-input) structure containing the motion vectors to translate.

#### Return values

None

#### Operation and runtime validation

This records conversion, not a CPU readback. The input heap must contain
the motion search result for `PixelWidth` and `PixelHeight`; those are the
dimensions of the last estimated pictures, not the heap's maximum dimensions.
The output is the application-provided 2D vector texture at
`MotionVectorCoordinate`. Its remaining region must hold one vector per
selected search block.

**Scoped recording validation:** the selected path checks the heap size range,
a non-reserved 2D destination, one mip and a valid destination subresource;
it checks protected-session compatibility of the heap and texture.
It diagnoses nonzero Z, an out-of-bounds destination region and an
incompatible heap node, but the selected branches do not themselves set a
failure HRESULT. The region check uses integer `PixelWidth / blockSize` and
`PixelHeight / blockSize`; it is not evidence that partial edge blocks may
be truncated. Non-block-aligned picture rounding and the treatment of such
edges remain unestablished; no floor/ceil rule is asserted.

**Format qualification:** the published resolve contract specifies
`DXGI_FORMAT_R16G16_SINT`. The inspected validator also accepts
`DXGI_FORMAT_R16G16_SNORM`, but that does not establish normalized-vector
scaling or universal driver support. No SNORM output contract is asserted.

<a id="motion-types"></a>

## API types and layout

The declarations below are API layouts, not driver layouts.
[`D3D12_VIDEO_SIZE_RANGE`](video-decode.md#d3d12-video-size-range) retains its
shared definition: maximum width/height and minimum width/height constrain
both input and reference pictures. Query direction annotations identify
inputs and outputs; creation descriptions contain only inputs.

<a id="d3d12-video-motion-estimator-search-block-size"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE

```cpp
typedef enum D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE
{
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_8X8       = 0,
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_16X16     = 1,
} D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE;
```

| Constant | Meaning |
|---|---|
| `D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_8X8` | Value: *0*. The search block size is 8x8 pixels. |
| `D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_16X16` | Value: *1*. The search block size is 16x16 pixels. |

<a id="d3d12-video-motion-estimator-search-block-size-flags"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS

```cpp
typedef enum D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS
{
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_NONE  = 0,
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_8X8        = (1 << D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_8X8),
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_16X16      = (1 << D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_16X16),

} D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS;
```

| Constant | Meaning |
|---|---|
| `D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_NONE` | Value: *0*. Search block size is not supported by the encoder. |
| `D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_8X8` | The encoder supports a search block size of 8x8 pixels. |
| `D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_16X16` | The encoder supports a search block size of 16x16 pixels. |

<a id="d3d12-video-motion-estimator-vector-precision"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION

```cpp
typedef enum D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION
{
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_QUARTER_PEL       = 0,
} D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION;
```

| Constant | Meaning |
|---|---|
| `D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_QUARTER_PEL` | Value: *0*. The vector precision is quarter-pixel motion. |

<a id="d3d12-video-motion-estimator-vector-precision-flags"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS

```cpp
typedef enum D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS
{
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAG_NONE           = 0,
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAG_QUARTER_PEL    = (1 << D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_QUARTER_PEL),

} D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS;
```

| Constant | Meaning |
|---|---|
| `D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAG_NONE` | Value: *0*. Vector precision is not supported by the encoder. |
| `D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAG_QUARTER_PEL` | The vector precision is quarter-pixel motion. |

<a id="d3d12-feature-data-video-feature-area-support"></a>

### D3D12_FEATURE_DATA_VIDEO_FEATURE_AREA_SUPPORT

```cpp
typedef struct D3D12_FEATURE_DATA_VIDEO_FEATURE_AREA_SUPPORT
{
    UINT NodeIndex;                                                     // input
    BOOL VideoDecodeSupport;                                            // output
    BOOL VideoProcessSupport;                                           // output
    BOOL VideoEncodeSupport;                                            // output
} D3D12_FEATURE_DATA_VIDEO_FEATURE_AREA_SUPPORT;
```

`NodeIndex` selects the physical adapter. The three BOOL outputs identify
availability of the decode, process and encode command families separately;
this payload does not replace operation-specific motion support.

<a id="d3d12-feature-data-video-motion-estimator"></a>

### D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR

```cpp
typedef struct D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR
{
    UINT NodeIndex;                                                                 // input
    DXGI_FORMAT InputFormat;                                                        // input
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS BlockSizeFlags;            // output
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS PrecisionFlags;             // output
    D3D12_VIDEO_SIZE_RANGE SizeRange;                                               // output
} D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR;
```

`NodeIndex`

In multi-adapter operation, identifies the physical adapter of the device this operation applies to.

`InputFormat`

A [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) structure specifying the format of the input resources.

`BlockSizeFlags`

A bitwise OR combination of values from the [D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS](#d3d12-video-motion-estimator-search-block-size-flags) enumeration specifying the encoder's supported search block sizes for motion estimation.

`PrecisionFlags`

A bitwise OR combination of values from the [D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS](#d3d12-video-motion-estimator-vector-precision-flags) enumeration specifying the encoder's supported vector precision for motion estimation.

`SizeRange`

A [D3D12_VIDEO_SIZE_RANGE](video-decode.md#d3d12-video-size-range) structure representing the minimum and maximum input size supported by the driver. The driver sets the fields of this structure to zero if motion estimation is unsupported.

<a id="d3d12-feature-data-video-motion-estimator-size"></a>

### D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_SIZE

```cpp
typedef struct D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_SIZE
{
    UINT NodeIndex;                                                     // input
    DXGI_FORMAT InputFormat;                                            // input
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE BlockSize;           // input
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION Precision;            // input
    D3D12_VIDEO_SIZE_RANGE SizeRange;                                   // input
    BOOL Protected;                                                     // input
    UINT64 MotionVectorHeapMemoryPoolL0Size;                            // output
    UINT64 MotionVectorHeapMemoryPoolL1Size;                            // output
    UINT64 MotionEstimatorMemoryPoolL0Size;                             // output
    UINT64 MotionEstimatorMemoryPoolL1Size;                             // output
} D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_SIZE;
```

`NodeIndex`

In multi-adapter operation, identifies the physical adapter of the device this operation applies to.

`InputFormat`

A [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) structure specifying the format of the input and reference resources.

`BlockSize`

A value from the [D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE](#d3d12-video-motion-estimator-search-block-size) specifying the search block size for motion estimation.

`Precision`

A value from the [D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION](#d3d12-video-motion-estimator-vector-precision) specifying the vector precision for motion estimation.

`SizeRange`

A [D3D12_VIDEO_SIZE_RANGE](video-decode.md#d3d12-video-size-range) structure representing the minimum and maximum input and reference frame size, in pixels, used by the motion estimator.

`Protected`

TRUE if the motion estimator operates on protected resource input and produces protected output; otherwise, FALSE.

`MotionVectorHeapMemoryPoolL0Size`

The allocation size of the motion vector heap in the L0 memory pool. L0 is the physical system memory pool. When the adapter is discrete/NUMA, this pool has greater bandwidth for the CPU and less bandwidth for the GPU. When the adapter is UMA, this pool is the only one which is valid. For more information, see [Residency](https://learn.microsoft.com/en-us/windows/win32/direct3d12/residency).

`MotionVectorHeapMemoryPoolL1Size`

The allocation size of the motion vector heap in the L1 memory pool. L1 is typically known as the physical video memory pool. L1 is only available when the adapter is discrete/NUMA, and has greater bandwidth for the GPU and cannot even be accessed by the CPU. When the adapter is UMA, this pool is not available. For more information, see [Residency](https://learn.microsoft.com/en-us/windows/win32/direct3d12/residency).

`MotionEstimatorMemoryPoolL0Size`

The allocation size of the motion estimator in the L0 memory pool. L0 is the physical system memory pool. When the adapter is discrete/NUMA, this pool has greater bandwidth for the CPU and less bandwidth for the GPU. When the adapter is UMA, this pool is the only one which is valid. For more information, see [Residency](https://learn.microsoft.com/en-us/windows/win32/direct3d12/residency).

`MotionEstimatorMemoryPoolL1Size`

The allocation size of the motion estimator in the L1 memory pool. L1 is typically known as the physical video memory pool. L1 is only available when the adapter is discrete/NUMA, and has greater bandwidth for the GPU and cannot even be accessed by the CPU. When the adapter is UMA, this pool is not available. For more information, see [Residency](https://learn.microsoft.com/en-us/windows/win32/direct3d12/residency).

<a id="d3d12-video-protected-resource-support-flags"></a>

### D3D12_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS

```cpp
typedef enum D3D12_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS
{
    D3D12_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAG_NONE                  = 0x0,
    D3D12_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAG_SUPPORTED             = 0x1,

} D3D12_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS;
```

`NONE` is zero; `SUPPORTED` (`0x1`) reports protected-resource operation support. These flags describe capability, not a session or encryption policy.

<a id="d3d12-feature-data-video-motion-estimator-protected-resources"></a>

### D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES

```cpp
typedef struct D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES
{
    UINT NodeIndex;                                                                 // input
    D3D12_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS SupportFlags;                      // output
} D3D12_FEATURE_DATA_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES;
```

`NodeIndex` selects the physical adapter. `SupportFlags` reports motion-estimator protected-resource support; non-null protected-session creation requires `SUPPORTED`.

<a id="d3d12-video-motion-estimator-desc"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_DESC

```cpp
typedef struct D3D12_VIDEO_MOTION_ESTIMATOR_DESC
{
    UINT NodeMask;
    DXGI_FORMAT InputFormat;
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE BlockSize;
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION Precision;
    D3D12_VIDEO_SIZE_RANGE SizeRange;

} D3D12_VIDEO_MOTION_ESTIMATOR_DESC;
```

`NodeMask`

The node mask specifying the physical adapter on which the motion object will be used. For single GPU operation, set this to zero. If there are multiple GPU nodes, set a bit to identify the node, i.e. the device's physical adapter, on which the motion object will be used. Each bit in the mask corresponds to a single node. Only 1 bit may be set.

`InputFormat`

A value from the [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) enumeration specifying the format of the input and reference frames.

`BlockSize`

A value from the [D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE](#d3d12-video-motion-estimator-search-block-size) enumeration specifying the search block size the video motion estimator will use.

`Precision`

A value from the [D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION](#d3d12-video-motion-estimator-vector-precision) enumeration specifying the vector precision the video motion estimator will use.

`SizeRange`

A [D3D12_VIDEO_SIZE_RANGE](video-decode.md#d3d12-video-size-range) structure representing the minimum and maximum input and reference frame size, in pixels, that the motion estimator will accept.

<a id="d3d12-video-motion-vector-heap-desc"></a>

### D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC

```cpp
typedef struct D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC
{
    UINT NodeMask;
    DXGI_FORMAT InputFormat;
    D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE BlockSize;
    D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION Precision;
    D3D12_VIDEO_SIZE_RANGE SizeRange;

} D3D12_VIDEO_MOTION_VECTOR_HEAP_DESC;
```

`NodeMask`

The node mask specifying the physical adapter on which the motion object will be used. For single GPU operation, set this to zero. If there are multiple GPU nodes, set a bit to identify the node, i.e. the device's physical adapter, on which the motion object will be used. Each bit in the mask corresponds to a single node. Only 1 bit may be set.

`InputFormat`

A value from the [DXGI_FORMAT](https://learn.microsoft.com/en-us/windows/desktop/api/dxgiformat/ne-dxgiformat-dxgi_format) enumeration specifying the format of the input and reference frames.

`BlockSize`

A value from the [D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE](#d3d12-video-motion-estimator-search-block-size) enumeration specifying the search block size the video motion estimator will use.

`Precision`

A value from the [D3D12_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION](#d3d12-video-motion-estimator-vector-precision) enumeration specifying the vector precision the video motion estimator will use.

`SizeRange`

A [D3D12_VIDEO_SIZE_RANGE](video-decode.md#d3d12-video-size-range) structure representing the minimum and maximum input and reference frame size, in pixels, that the motion estimator will accept.

<a id="d3d12-resource-coordinate"></a>

### D3D12_RESOURCE_COORDINATE

```cpp
typedef struct D3D12_RESOURCE_COORDINATE
{
    // Coordinate values below index pixels.
    UINT64 X; // Used for buffer, 1D, 2D, 3D
    UINT Y; // Used for 2D, 3D
    UINT Z; // Used for 3D
    UINT SubresourceIndex; // indexes into mips, arrays, and planes. Used for 1D, 2D, 3D
} D3D12_RESOURCE_COORDINATE;
```

`X`

The x-coordinate of the resource.

`Y`

The y-coordinate of the resource.

`Z`

The z-coordinate of the resource.

`SubresourceIndex`

The index of the subresource for the resource.

<a id="d3d12-video-motion-estimator-output"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_OUTPUT

```cpp
typedef struct D3D12_VIDEO_MOTION_ESTIMATOR_OUTPUT
{
    ID3D12VideoMotionVectorHeap* pMotionVectorHeap;
} D3D12_VIDEO_MOTION_ESTIMATOR_OUTPUT;
```

`pMotionVectorHeap`

An [ID3D12VideoMotionVectorHeap](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionvectorheap) containing the hardware-dependent motion estimation output.

The resolved texture representation is defined by [resolve output](#d3d12-resolve-video-motion-vector-heap-output); it is not the opaque heap's layout.

<a id="d3d12-video-motion-estimator-input"></a>

### D3D12_VIDEO_MOTION_ESTIMATOR_INPUT

```cpp
typedef struct D3D12_VIDEO_MOTION_ESTIMATOR_INPUT
{
    ID3D12Resource*                     pInputTexture2D;
    UINT                                InputSubresourceIndex;
    ID3D12Resource*                     pReferenceTexture2D;
    UINT                                ReferenceSubresourceIndex;
    ID3D12VideoMotionVectorHeap*        pHintMotionVectorHeap;
} D3D12_VIDEO_MOTION_ESTIMATOR_INPUT;
```

`pInputTexture2D`

An [ID3D12Resource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12resource) representing the current frame. The motion estimation operation applies to the entire frame.

`InputSubresourceIndex`

The base plane of the MIP and array slice to use for the input.

`pReferenceTexture2D`

An [ID3D12Resource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12resource) representing the reference frame, or past frame, used for motion estimation.

`ReferenceSubresourceIndex`

The base plane of the MIP and array slice to use for the reference.

`pHintMotionVectorHeap`

An [ID3D12VideoMotionVectorHeap](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionvectorheap) representing the buffer containing the hardware-dependent output of the previous motion estimator operation which may be used for hinting the current operation. This parameter may be NULL, indicating that previous motion estimator output should not be considered for the current operation.

<a id="d3d12-resolve-video-motion-vector-heap-output"></a>

### D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT

```cpp
typedef struct D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT
{
    ID3D12Resource*             pMotionVectorTexture2D;
    D3D12_RESOURCE_COORDINATE   MotionVectorCoordinate;
} D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT;
```

`pMotionVectorTexture2D`

An [ID3D12Resource](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12resource) representing the output resource for resolved motion vectors. Motion vectors are resolved to [DXGI_FORMAT_R16G16_SINT](https://learn.microsoft.com/en-us/windows/win32/api/dxgiformat/ne-dxgiformat-dxgi_format) 2D textures. The resolved data is a pair of signed 16-bit integers with quarter PEL units with the X vector component stored in the R component and the Y vector component stored in the G component. Motion vectors are stored in a 2D layout that corresponds to the pixel layout of the original input textures.

`MotionVectorCoordinate`

A [D3D12_RESOURCE_COORDINATE](#d3d12-resource-coordinate) structure specifying the output origin of the motion vectors. The remaining sub-region must be large enough to store all motion vectors per block specified by the input pixel width and pixel height and the specified [D3D12_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE](#d3d12-video-motion-estimator-search-block-size).

The signed 16-bit representation can encode integer
components −32768 through 32767, corresponding arithmetically to −8192
through 8191.75 pixels at that scale. This is a representation range, not a
guaranteed search displacement, clipping rule or algorithmic search window.
The block grid corresponds spatially to the original input pictures.
The sign convention for displacement direction and partial-block edge handling
are not established by the selected source; they are not inferred from
component width.

<a id="d3d12-resolve-video-motion-vector-heap-input"></a>

### D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT

```cpp
typedef struct D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT
{
    ID3D12VideoMotionVectorHeap*                    pMotionVectorHeap;
    UINT                                            PixelWidth;
    UINT                                            PixelHeight;
} D3D12_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT;
```

`pMotionVectorHeap`

The [ID3D12VideoMotionVectorHeap](https://learn.microsoft.com/en-us/windows/win32/api/d3d12video/nn-d3d12video-id3d12videomotionvectorheap) containing the hardware-dependent data layout of the motion search.

`PixelWidth`

The pixel width of the texture that the motion estimation operation was performed on. The motion vector heap may be allocated to support a size range, this parameter informs the size of the last motion estimation operation.

`PixelHeight`

The pixel height of the texture that the motion estimation operation was performed on. The motion vector heap may be allocated to support a size range, this parameter informs the size of the last motion estimation operation.

<a id="motion-driver"></a>

## Driver reference

The following driver contract uses published WDK declarations. API pointers
are translated to driver handles; API and DDI payloads are not interchangeable.
In particular, the API size query uses `NodeIndex` and heap-size fields first;
`D3D12DDI_VIDEO_MOTION_ESTIMATOR_SIZE_DATA_0060` uses `NodeMask` and
estimator-size fields first. The protected-session creation parameter becomes
`hDrvProtectedResourceSession`. `pHintMotionVectorHeap` becomes
`hDrvPreviousMotionVectorHeap`. The _0060 support payload and resolve output
do not include the earlier _0053 differences-metric members.

The selected _0060 device-table slots are
`pfnCalcPrivateVideoMotionEstimatorSize`, `pfnCreateVideoMotionEstimator`,
`pfnDestroyVideoMotionEstimator`, `pfnCalcPrivateVideoMotionVectorHeapSize`,
`pfnCreateVideoMotionVectorHeap`, and `pfnDestroyVideoMotionVectorHeap`.
Their function declarations appear with their objects below.
The encode-list _0060 table maps `pfnEstimateMotion` to
`PFND3D12DDI_ESTIMATE_MOTION_0053` and
`pfnResolveVideoMotionVectorHeap` to
`PFND3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_0060`.
`pfnCloseCommandList` and `pfnResetCommandList` use the common
`PFND3D12DDI_CLOSECOMMANDLIST` and `PFND3D12DDI_RESETCOMMANDLIST_0040`;
`ClearState` is not a dedicated slot in this table.

The public feature version `D3D12DDI_FEATURE_VERSION_VIDEO_0060_0`
is 14; the published header identifies minimum core DDI
`D3D12DDI_SUPPORTED_0043`. The video encode queue table uses
`D3D12DDI_COMMAND_QUEUE_FUNCS_VIDEO_0020` under table type
`D3D12DDI_TABLE_TYPE_0053_COMMAND_QUEUE_VIDEO_ENCODE`.
These DDI version facts are not a claim that every optional video operation
is supported.

<a id="motion-driver-types"></a>

### Driver block-size, precision and coordinate types

```cpp
typedef enum D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053
{
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053_8X8       = 0,
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053_16X16     = 1,
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053;
```

```cpp
typedef enum D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS_0053
{
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_0053_NONE  = 0,
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_0053_8X8        = (1 << D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053_8X8),
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAG_0053_16X16      = (1 << D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053_16X16),

} D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS_0053;
```

```cpp
typedef enum D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053
{
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053_QUARTER_PEL       = 0,
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053;
```

```cpp
typedef enum D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS_0053
{
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAG_0053_NONE           = 0,
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAG_0053_QUARTER_PEL    = (1 << D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053_QUARTER_PEL),

} D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS_0053;
```

```cpp
typedef struct D3D12DDI_RESOURCE_COORDINATE_0053
{
    // Coordinate values below index pixels.
    UINT64 X; // Used for buffer, 1D, 2D, 3D
    UINT Y; // Used for 2D, 3D
    UINT Z; // Used for 3D
    UINT SubresourceIndex; // indexes into mips, arrays. Used for 1D, 2D, 3D
} D3D12DDI_RESOURCE_COORDINATE_0053;
```

The shared DDI size-range contract remains under
[driver capability payloads](video-decode.md#ddi-capability-payloads):
`D3D12DDI_VIDEO_SIZE_RANGE_0032` is the driver type used below, not an alias
for the API `D3D12_VIDEO_SIZE_RANGE`.

The narrow protected-motion capability case is
`D3D12DDICAPS_TYPE_VIDEO_0072_MOTION_ESTIMATOR_PROTECTED_RESOURCES` (30).
It takes the following driver payload, with node input and support-flag output.
`SUPPORTED` enables the protected-object association; this query does not
create a protected session.

```cpp
typedef enum D3D12DDI_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS_0072
{
    D3D12DDI_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAG_0072_NONE                    = 0,
    D3D12DDI_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAG_0072_SUPPORTED               = 0x1,

} D3D12DDI_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS_0072;
```

```cpp
typedef struct D3D12DDI_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES_DATA_0072
{
    UINT NodeIndex;                                                                           // input
    D3D12DDI_VIDEO_PROTECTED_RESOURCE_SUPPORT_FLAGS_0072 SupportFlags;                        // output
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES_DATA_0072;
```

<a id="motion-driver-capabilities"></a>

### Driver capability payloads

The shared
[driver capability protocol](video-decode.md#ddi-capability-payloads) defines
`D3D12DDIARG_VIDEO_GETCAPS_0020` and the
[video driver entry](video-decode.md#decode-driver) defines
`PFND3D12DDI_VIDEO_GETCAPS`. For the motion cases, `pInfo` is null,
`pData` addresses the named payload and `DataSize` is `sizeof` that payload:

* `D3D12DDICAPS_TYPE_VIDEO_0053_FEATURE_AREA_SUPPORT` (19) uses
  `D3D12DDI_VIDEO_FEATURE_AREA_SUPPORT_DATA_0053`.
* `D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR` (20) uses
  `D3D12DDI_VIDEO_MOTION_ESTIMATOR_DATA_0060`.
* `D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR_SIZE` (21) uses
  `D3D12DDI_VIDEO_MOTION_ESTIMATOR_SIZE_DATA_0060`.
* `D3D12DDICAPS_TYPE_VIDEO_0072_MOTION_ESTIMATOR_PROTECTED_RESOURCES` (30)
  uses `D3D12DDI_VIDEO_MOTION_ESTIMATOR_PROTECTED_RESOURCES_DATA_0072`.

The negotiated video DDI version selects the payload version; sharing a
capability case number does not make the older _0053 payload an _0060
layout. These cases are capability requests, not encode-list timestamp
queries.

The `D3D12DDICAPS_TYPE_VIDEO_0020` enum identifies capability checks for Motion Estimation.

`D3D12DDICAPS_TYPE_VIDEO_0053_FEATURE_AREA_SUPPORT`

Video Decode, Video Processing, and Video Encoding are all versioned under the D3D12 Video extended feature. However, these DDIs mostly describe optional features. This cap helps shortcut understanding of driver support for any decode, video processing, or encode support making it easier to block queue/commandlist/command recorder/command pool objects. Each of these queue types has required functionality that we don't want to require for encode if motion estimation or encode is not supported. In all cases, function tables for a supported queue type must be fully populated; functions for an unsupported queue type may be nullptr.

By reporting any video feature version, driver must support the `pfnGetCaps` member of `D3D12DDI_DEVICE_FUNCS_VIDEO`.

`D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR`

Checks for motion estimator support.  See D3D12DDI_VIDEO_MOTION_ESTIMATOR_DATA_0060.

`D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR_SIZE`

Checks for Motion Estimator object sizes.  D3D12 applications are given a budget.  This helps applications account for these objects.

<a id="video-feature-area-support"></a>

#### D3D12DDI_VIDEO_FEATURE_AREA_SUPPORT_DATA_0053

```cpp
typedef struct D3D12DDI_VIDEO_FEATURE_AREA_SUPPORT_DATA_0053
{
    UINT NodeIndex;                                                     // input
    BOOL VideoDecodeSupport;                                            // output
    BOOL VideoProcessSupport;                                           // output
    BOOL VideoEncodeSupport;                                            // output
} D3D12DDI_VIDEO_FEATURE_AREA_SUPPORT_DATA_0053;
```

This payload describes the command types supported by the driver. Driver must report support for at least one video command type if the video feature version is supported.

`NodeIndex`

In multi-adapter operation, this indicates which physical adapter of the device this operation applies to.

The [decode driver associations](video-decode.md#ddi-table-members)
define the decoding operations named by the following device slots.

If `VideoDecodeSupport` is TRUE:

- Predication, timestamp queries, and history buffers must be supported with the decode queue type.

- The table corresponding to the following table types for the reported video feature must be fully populated:
  - `D3D12DDI_TABLE_TYPE_0022_COMMAND_LIST_VIDEO_DECODE`
  - `D3D12DDI_TABLE_TYPE_0022_COMMAND_QUEUE_VIDEO_DECODE`

- Also, the following members of the table type corresponding to `D3D12DDI_TABLE_TYPE_0020_DEVICE_VIDEO` and the reported video feature version must be populated:
  - `pfnCalcPrivateVideoDecoderSize`
  - `pfnCreateVideoDecoder`
  - `pfnDestroyVideoDecoder`
  - `pfnCalcPrivateVideoDecoderHeapSize`
  - `pfnCreateVideoDecoderHeap`
  - `pfnDestroyVideoDecoderHeap`

When the flag is not set, all of these functions may be set to nullptr.

The [processing driver associations](video-processing.md#process-ddi-associations)
define the processing operations named by the following device slots.

If `VideoProcessSupport` is TRUE:

- Predication, timestamp queries, and history buffers must be supported with the video process queue type.

- The table corresponding to the following table types for the reported video feature must be fully populated:
  - `D3D12DDI_TABLE_TYPE_0022_COMMAND_LIST_VIDEO_PROCESS`
  - `D3D12DDI_TABLE_TYPE_0022_COMMAND_QUEUE_VIDEO_PROCESS`

- Also, the following members of the table type corresponding to `D3D12DDI_TABLE_TYPE_0020_DEVICE_VIDEO` and the reported video feature version must be populated:
  - `pfnCalcPrivateVideoProcessorSize`
  - `pfnCreateVideoProcessor`
  - `pfnDestroyVideoProcessor`

When the flag is not set, all of these functions may be set to nullptr.

If `VideoEncodeSupport` is TRUE:

- Predication, timestamp queries, and history buffers must be supported with the video encode queue type.

- The table corresponding to the following table types for the reported video feature must be fully populated:
  - `D3D12DDI_TABLE_TYPE_0053_COMMAND_LIST_VIDEO_ENCODE`
  - `D3D12DDI_TABLE_TYPE_0053_COMMAND_QUEUE_VIDEO_ENCODE`

- Also, the following members of the table type corresponding to `D3D12DDI_TABLE_TYPE_0020_DEVICE_VIDEO` and the reported video feature version must be populated:
  - `pfnCalcPrivateVideoMotionEstimatorSize`
  - `pfnCreateVideoMotionEstimator`
  - `pfnDestroyVideoMotionEstimator`

When the flag is not set, all of these functions may be set to nullptr.

#### D3D12DDI_VIDEO_MOTION_ESTIMATOR_DATA_0060

```cpp
typedef struct D3D12DDI_VIDEO_MOTION_ESTIMATOR_DATA_0060
{
    UINT NodeIndex;                                                                           // input
    DXGI_FORMAT InputFormat;                                                                  // input
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS_0053 BlockSizeFlags;              // output
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS_0053 PrecisionFlags;               // output
    D3D12DDI_VIDEO_SIZE_RANGE_0032 SizeRange;                                                 // output
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_DATA_0060;
```

`NodeIndex`

In multi-adapter operation, this indicates which physical adapter of the device this operation applies to.

`InputFormat`

The DXGI_FORMAT of the input frame and reference frame. Currently only DXGI_FORMAT_NV12 is allowed.

`BlockSizeFlags`

The block sizes supported by driver. At least one bit must be set to support the motion estimation operation. Set to NONE if not supported. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_FLAGS_0053` for more details.

`PrecisionFlags`

The precision supported by driver. At least one bit must be set to support the motion estimation operation. Set to NONE if not supported. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_FLAGS_0053` for more details.

`SizeRange`

Sets the minimum and maximum input and reference size in Pixels supported by driver. Set to zeros when motion estimation is not supported.

#### D3D12DDI_VIDEO_MOTION_ESTIMATOR_SIZE_DATA_0060

```cpp
typedef struct D3D12DDI_VIDEO_MOTION_ESTIMATOR_SIZE_DATA_0060
{
    UINT NodeMask;                                                              // input
    DXGI_FORMAT InputFormat;                                                    // input
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053 BlockSize;           // input
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053 Precision;            // input
    D3D12DDI_VIDEO_SIZE_RANGE_0032 SizeRange;                                   // input
    BOOL Protected;                                                             // input
    UINT64 MotionEstimatorMemoryPoolL0Size;                                     // output
    UINT64 MotionEstimatorMemoryPoolL1Size;                                     // output
    UINT64 MotionVectorHeapMemoryPoolL0Size;                                    // output
    UINT64 MotionVectorHeapMemoryPoolL1Size;                                    // output
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_SIZE_DATA_0060;
```

This cap determines the residency size for the motion estimator and the hardware dependent output buffer when called with the same creation parameters.

`NodeMask`

In multi-adapter operation, this indicates which physical adapter of the device this operation applies to.

`InputFormat`

The DXGI_FORMAT of the input frame and reference frame. Currently only DXGI_FORMAT_NV12 is allowed.

`BlockSize`

The block size to use with the motion estimator. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053` for details.

`Precision`

The precision to use with the motion estimator. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053` for more details.

`SizeRange`

The size range allowed with the Motion Estimator. This may be a subset of the size range supported by the driver to optimize memory usage.

`Protected`

TRUE if the motion estimator operates on protected resource input and produces protected output. The driver must also support protected resources for D3D12 to set TRUE. FALSE otherwise.

`MotionEstimatorMemoryPoolL0Size`

Driver outputs the L0 size of the motion estimator object. L0 is system memory.

`MotionEstimatorMemoryPoolL1Size`

Driver outputs the L1 size of the motion estimator object. L1 is local video memory for discrete/NUMA adapters and unavailable for UMA adapters.

`MotionVectorHeapMemoryPoolL0Size`

Driver outputs the L0 size of the motion vector heap object. L0 is system memory.

`MotionVectorHeapMemoryPoolL1Size`

Driver outputs the L1 size of the motion vector heap object. L1 is local video memory for discrete/NUMA adapters and unavailable for UMA adapters.

<a id="motion-driver-objects"></a>

### Driver object lifetime and allocation

The allocation-at-creation, residency/debug-allocation support and
cross-queue exclusion requirements below are published source requirements.
The bounded creation/release inspection corroborates the context/heap object
split and driver creation/destruction dispatch, not every driver's allocation
timing, residency accounting or concurrency behavior.

The motion estimation operation has a context object to associate the lifetime of internal allocations needed to perform the operation. All allocations associated with this object should be allocated when the object is created and deallocated when the object is de-allocated. This should include any buffers used as temporary/scratch storage.

Operations against this object may be recorded to command lists in a different order than execution. No two API queue instances may be executing command lists containing this object at the same time and expect valid results. The application is responsible for synchronizing access across multiple queue instances.

#### Make Resident and Evict

This object must support `PFND3D12DDI_MAKERESIDENT` and `PFND3D12DDI_EVICT`.

#### Get Debug Allocation Info

This object must support the `PFND3D12DDI_GET_DEBUG_ALLOCATION_INFO` DDI. This pre-existing DDI returns the associated kernel mode allocation handles and GPU Virtual Address ranges.

#### Query Size

D3D12 applications manage a memory budget and may need to know the size of the motion estimator to do so. See the `D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR_SIZE` caps check.

`D3D12DDI_HVIDEOMOTIONESTIMATOR_0053` is the driver handle. Its `D3D12DDI_HANDLETYPE` value is `D3D12DDI_HT_0053_VIDEO_MOTION_ESTIMATOR = 45`.

A handle and Handle Type are defined for the motion estimator object.

#### Creating the Motion Estimator

These methods create a motion estimator instance.

#### D3D12DDIARG_CREATE_VIDEO_MOTION_ESTIMATOR_0060

```cpp
typedef struct D3D12DDIARG_CREATE_VIDEO_MOTION_ESTIMATOR_0060
{
    UINT NodeMask;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053 BlockSize;
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053 Precision;
    D3D12DDI_VIDEO_SIZE_RANGE_0032 SizeRange;
    D3D12DDI_HPROTECTEDRESOURCESESSION_0030 hDrvProtectedResourceSession;
} D3D12DDIARG_CREATE_VIDEO_MOTION_ESTIMATOR_0060;
```

Specifies the creation arguments for the motion estimator. Valid arguments are determined by the D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR caps check.

`NodeMask`

For single GPU operation, set this to zero. If there are multiple GPU nodes, set a bit to identify the node (the device\'s physical adapter) on which the motion object will be used. Each bit in the mask corresponds to a single node. Only 1 bit may be set.

`InputFormat`

The DXGI_FORMAT of the input and reference frames. This motion estimator may only be used with input textures of this format.

`BlockSize`

The search block size to use with this motion estimator. This parameter determines the number of motion vectors output during the resolve step. This must be a block size reported as supported by the driver. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053`.

`Precision`

The precision of motion vector components. This must be a precision reported as supported by the driver. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053`.

`SizeRange`

Indicates the minimum and maximum size of the inputs to the motion estimation operation. Actual size is provided at motion estimation time. Driver should allocate at the creation of the motion estimator to support any size within the range. This size range must be a subset of the size range supported by the driver.

`hDrvProtectedResourceSession`

Indicates the protected resource session to use for the motion estimator. This can inform internal allocations for the motion estimator. This value is NULL if the motion estimator will operate on unprotected resources.

#### PFND3D12DDI_CALCPRIVATEVIDEOMOTIONESTIMATORSIZE_0060

```cpp
typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALCPRIVATEVIDEOMOTIONESTIMATORSIZE_0060 )( D3D12DDI_HDEVICE hDrvDevice, _In_ CONST D3D12DDIARG_CREATE_VIDEO_MOTION_ESTIMATOR_0060* pArgs);
```

The D3D runtime allocates memory for storing the driver's CPU object representing the motion estimator. This method is used to calculate the driver object size.

#### PFND3D12DDI_CREATEVIDEOMOTIONESTIMATOR_0060

```cpp
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATEVIDEOMOTIONESTIMATOR_0060 )( D3D12DDI_HDEVICE hDrvDevice, _In_ CONST D3D12DDIARG_CREATE_VIDEO_MOTION_ESTIMATOR_0060* pArgs, D3D12DDI_HVIDEOMOTIONESTIMATOR_0053 hDrvMotionEstimator);
```

Creates the Motion Estimator.

#### Destroy the Motion Estimator

#### PFND3D12DDI_DESTROYVIDEOMOTIONESTIMATOR_0053

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROYVIDEOMOTIONESTIMATOR_0053 )( D3D12DDI_HDEVICE hDrvDevice, D3D12DDI_HVIDEOMOTIONESTIMATOR_0053 hDrvMotionEstimator );
```

Destroys the Motion Estimator.

### Motion Estimation Output Object

The Motion Vector Heap object is created to store hardware dependent motion vector output. Motion vector output is opaque.  A resolve operation will take this opaque object as input and resolve it to the specified format. The size of this object is driver controlled and determined by `D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR_SIZE`.

#### Make Resident and Evict

This object must support `PFND3D12DDI_MAKERESIDENT` and `PFND3D12DDI_EVICT`.

#### Get Debug Allocation Info

This object must support the `PFND3D12DDI_GET_DEBUG_ALLOCATION_INFO` DDI. This pre-existing DDI returns the associated kernel mode allocation handles and GPU Virtual Address ranges.

#### Query Size

D3D12 applications manage a memory budget and may need to know the size of the motion vector heap to do so. See the `D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR_SIZE` caps check.

`D3D12DDI_HVIDEOMOTIONVECTORHEAP_0053` is the driver handle. Its `D3D12DDI_HANDLETYPE` value is `D3D12DDI_HT_0053_VIDEO_MOTION_VECTOR_HEAP = 46`.

A handle and Handle Type are defined for the motion vector heap object.

#### Creating the Motion Vector Heap

These methods create a motion vector heap instance.

#### D3D12DDIARG_CREATE_VIDEO_MOTION_VECTOR_HEAP_0060

```cpp
typedef struct D3D12DDIARG_CREATE_VIDEO_MOTION_VECTOR_HEAP_0060
{
    UINT NodeMask;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053 BlockSize;
    D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053 Precision;
    D3D12DDI_VIDEO_SIZE_RANGE_0032 SizeRange;
    D3D12DDI_HPROTECTEDRESOURCESESSION_0030 hDrvProtectedResourceSession;
} D3D12DDIARG_CREATE_VIDEO_MOTION_VECTOR_HEAP_0060;
```

Specifies the creation arguments for the motion vector heap. Valid arguments are determined by the D3D12DDICAPS_TYPE_VIDEO_0053_MOTION_ESTIMATOR caps check.

`NodeMask`

For single GPU operation, set this to zero. If there are multiple GPU nodes, set a bit to identify the node (the device\'s physical adapter) on which the motion object will be used. Each bit in the mask corresponds to a single node. Only 1 bit may be set.

`InputFormat`

The DXGI_FORMAT of the input and reference frames. This motion vector heap may only be used with input textures of this format.

`BlockSize`

The search block size to use with this motion vector heap. This parameter determines the number of motion vectors output during the resolve step. This must be a block size reported as supported by the driver. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053`.

`Precision`

The precision of motion vector components. This must be a precision reported as supported by the driver. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_VECTOR_PRECISION_0053`.

`SizeRange`

Indicates the minimum and maximum size of the inputs to the motion estimation operation. Actual size is provided at motion estimation time. Driver should allocate at the creation of the motion vector heap to support any size within the range. This size range must be a subset of the size range supported by the driver.

`hDrvProtectedResourceSession`

Indicates the protected resource session to use for the motion vector heap. This can inform if internal allocations must be allocated as protected. This value is NULL if the motion estimation operation will operate on only unprotected resources.

#### PFND3D12DDI_CALCPRIVATEVIDEOMOTIONVECTORHEAPSIZE_0060

```cpp
typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALCPRIVATEVIDEOMOTIONVECTORHEAPSIZE_0060 )( D3D12DDI_HDEVICE hDrvDevice, _In_ CONST D3D12DDIARG_CREATE_VIDEO_MOTION_VECTOR_HEAP_0060* pArgs);
```

The D3D runtime allocates memory for storing the driver's CPU object representing the motion vector heap. This method is used to calculate the driver object size.

#### PFND3D12DDI_CREATEVIDEOMOTIONVECTORHEAP_0060

```cpp
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATEVIDEOMOTIONVECTORHEAP_0060 )( D3D12DDI_HDEVICE hDrvDevice, _In_ CONST D3D12DDIARG_CREATE_VIDEO_MOTION_VECTOR_HEAP_0060* pArgs, D3D12DDI_HVIDEOMOTIONVECTORHEAP_0053 hDrvMotionEstimator);
```

Creates the Motion Vector Heap.  This object represents the hardware dependent output of a motion search.

#### Destroy the Motion Vector Heap

#### PFND3D12DDI_DESTROYVIDEOMOTIONVECTORHEAP_0053

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROYVIDEOMOTIONVECTORHEAP_0053 )( D3D12DDI_HDEVICE hDrvDevice, D3D12DDI_HVIDEOMOTIONVECTORHEAP_0053 hDrvMotionEstimator );
```

Destroys the Motion Vector Heap.

### Motion Estimation Operation

#### D3D12DDI_VIDEO_MOTION_ESTIMATOR_OUTPUT_0053

```cpp
typedef struct D3D12DDI_VIDEO_MOTION_ESTIMATOR_OUTPUT_0053
{
    D3D12DDI_HVIDEOMOTIONVECTORHEAP_0053                  hDrvMotionVectorHeap;
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_OUTPUT_0053;
```

Describes the output of the motion estimation operation.

*hDrvMotionVectorHeap*
The motion vector heap stores the motion estimator output in hardware dependent layout.

#### D3D12DDI_VIDEO_MOTION_ESTIMATOR_INPUT_0053

```cpp
typedef struct D3D12DDI_VIDEO_MOTION_ESTIMATOR_INPUT_0053
{
    D3D12DDI_HRESOURCE                          hDrvInputTexture2D;
    UINT                                        InputSubresourceIndex;
    D3D12DDI_HRESOURCE                          hDrvReferenceTexture2D;
    UINT                                        ReferenceSubresourceIndex;
    D3D12DDI_HVIDEOMOTIONVECTORHEAP_0053        hDrvPreviousMotionVectorHeap;
} D3D12DDI_VIDEO_MOTION_ESTIMATOR_INPUT_0053;
```

Describes the input to the motion estimation operation.

`hDrvInputTexture2D`

The handle of the current frame.  The operation applies to the entire frame.

`InputSubresourceIndex`

Indicates the base plane of the Mip and Array Slice to use for the input.

`hDrvReferenceTexture2D`

The handle of the reference frame, or Past frame, used for motion estimation.

`ReferenceSubresourceIndex`

Indicates the base plane of the Mip and Array Slice to use for the reference.

`hDrvPreviousMotionVectorHeap`

This parameter may be NULL indicating that previous motion estimator output should not be considered for this operation. If non-NULL, this buffer contains the hardware dependent output of the previous motion estimator operation and may be used for hinting the current operation.

#### PFND3D12DDI_ESTIMATE_MOTION_0053

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_ESTIMATE_MOTION_0053 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    D3D12DDI_HVIDEOMOTIONESTIMATOR_0053 hDrvMotionEstimator,
    CONST D3D12DDI_VIDEO_MOTION_ESTIMATOR_OUTPUT_0053* pOutputArguments,
    CONST D3D12DDI_VIDEO_MOTION_ESTIMATOR_INPUT_0053* pInputArguments
    );
```

Function to perform the motion estimation operation.

`hDrvCommandList`

Driver handle for a video encode command list.

`hDrvMotionEstimator`

 Handle to the motion estimator context object. See [driver object lifetime and allocation](#motion-driver-objects) for more details.

`pOutputArguments`

 The output arguments for the motion estimation operation. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_OUTPUT_0053`.

`pInputArguments`

The input arguments for the motion estimation operation. See `D3D12DDI_VIDEO_MOTION_ESTIMATOR_INPUT_0053`.

### Resolving hardware-dependent output

The resolve operation is expected to be a light-weight translation of hardware dependent output to the API spec'd format.

#### D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT_0060

```cpp
typedef struct D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT_0060
{
    D3D12DDI_HRESOURCE hDrvMotionVectorTexture2D;
    D3D12DDI_RESOURCE_COORDINATE_0053 MotionVectorCoordinate;
} D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT_0060;
```

Describes the output of the resolve operation.

`hDrvMotionVectorTexture2D`

The output resource for resolved motion vectors. Motion vectors are resolved to DXGI_FORMAT_R16G16_SINT 2D textures. The resolved data is expected to be a pair of signed 16-bit integers with quarter pel units with the X vector component stored in the R component and the Y vector component stored in the G component. Motion vectors are stored in a 2D layout that corresponds to the pixel layout of the original input textures.

`MotionVectorCoordinate`

Specifies the output origin of the motion vectors. The remaining sub-region must be large enough to store all motion vectors per block specified by the input PixelWidth/Pixelheight and the D3D12DDI_VIDEO_MOTION_ESTIMATOR_SEARCH_BLOCK_SIZE_0053.

#### D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT_0053

```cpp
typedef struct D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT_0053
{
    D3D12DDI_HVIDEOMOTIONVECTORHEAP_0053                    hDrvMotionVectorHeap;
    UINT                                                    PixelWidth;
    UINT                                                    PixelHeight;
} D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT_0053;
```

Describes the input of the resolve operation.

`hDrvMotionVectorHeap`

The motion vector heap containing the hardware dependent data layout of the motion search.

`PixelWidth`

The pixel width of the texture that the motion estimation operation was performed on.  The motion vector heap may be allocated to support a size range, this parameter informs the size of the last motion estimation operation.

`PixelHeight`

The pixel height of the texture that the motion estimation operation was performed on.  The motion vector heap may be allocated to support a size range, this parameter informs the size of the last motion estimation operation.

#### PFND3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_0060

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_0060 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    CONST D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT_0060* pOutputArguments,
    CONST D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT_0053* pInputArguments
    );
```

The function used to record the resolve operation for the motion estimation data.

`hDrvCommandList`

The video encode command list used to record the resolve operation.

`pOutputArguments`

The output arguments for the resolve operation. See `D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_OUTPUT_0060`.

`pInputArguments`

The input arguments for the resolve operation. See `D3D12DDI_RESOLVE_VIDEO_MOTION_VECTOR_HEAP_INPUT_0053`.
