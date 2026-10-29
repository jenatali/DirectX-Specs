<a id="video-encoding"></a>

# Video encoding

This subordinate [Video](video.md) reference specifies encoding objects,
capability tuples, application-managed reference pictures, recording and
metadata conversion for H.264, HEVC and AV1. The compact route is
[capabilities](#encoding-capabilities), [objects](#encoding-objects),
[recording](#encoding-recording), [metadata](#encoding-metadata), and the
[declaring API reference](#encoding-api). Detailed controls, codec syntax,
released types and actual published driver layouts are subordinate references,
not independent entry points.

This is a **qualified draft**. Codec
syntax, algorithms, numeric/statistical details and hardware behavior retained
from the published sources remain source-only unless explicitly qualified
otherwise. Specific source discrepancies are qualified beside the affected
contracts.

<a id="encoding-capabilities"></a>

## Capabilities and versioned inputs

[`ID3D12VideoDevice::CheckFeatureSupport`](video-decode.md#video-check-feature-support)
owns the declaring API and common return protocol. Encoding queries contribute
fixed payload cases to that owner; they are not new methods. `FeatureSupportDataSize`
is `sizeof` the selected payload. `NodeIndex` is a zero-based node index;
creation uses `NodeMask`. Initialize inputs and supply writable storage for
pointed-to outputs. A successful HRESULT alone is not feature support.

Individual codec, profile/level, input format, rate control, intra-refresh and
subregion-mode support are not a guarantee of simultaneous support. The
`SUPPORT`, `SUPPORT1` and `SUPPORT2` payloads query the combined tuple and return
validation/support flags, a suggested profile/level, and one
`pResolutionDependentSupport` entry per `pResolutionList` entry. The application
provides both arrays with `ResolutionsListCount` entries and codec-selected
storage for `SuggestedProfile` and `SuggestedLevel`. `SUPPORT2` uses the
version-1 per-resolution limits, not the old element stride.

`OUTPUT_RESOLUTION_RATIOS_COUNT` returns the count for the codec;
`OUTPUT_RESOLUTION` receives that same count and an application-allocated
output ratio array. It also reports minimum/maximum dimensions and divisibility
requirements. This query is not itself a profile-and-level tuple query.
`RESOURCE_REQUIREMENTS` sizes/aligns the opaque hardware metadata and
compressed-bitstream access for the selected codec/profile/format/resolution.
Version 1 additionally accounts for the enabled optional statistics; its
opaque buffer size is not the portable resolved-record size.

`SUPPORT1` is required for AV1: the older `SUPPORT` query reports codec-not-supported
rather than accepting AV1. Rate-control extension 1 and quality-versus-speed
are shared capabilities, not AV1-only controls. The complete legacy/extension-1
selection and `DataSize` tables are in
[rate control](video-encoding-controls.md#control-rate-control).
QP-map, dirty-region, motion-search and lower-resolution frame-analysis inputs
join the combined tuple through `SUPPORT2`. Disabled additions preserve the
specified older query semantics; CPU-QP compatibility reporting has its own
explicit exception in the detailed controls.

The [detailed capability/parameter reference](video-encoding-controls.md)
retains per-field input/output directions, count protocols and per-resolution
limits. The [codec reference](video-encoding-codecs.md) retains codec-specific
support/configuration, format, tile, header and DPB requirements.
The [type reference](video-encoding-types.md) supplies one selected final
declaration for each common type; separate versioned types remain separate.

<a id="encoding-objects"></a>

## Encoder objects and heaps

An encoder fixes codec, profile, input format, sequence-level codec
configuration and maximum motion-estimation precision. A heap fixes codec,
profile, level and its supported resolution list. Their descriptions are
different contracts: the encoder does not acquire the heap's level or resolution
list, and the heap does not replace the encoder's codec configuration.
The objects used together must be compatible with the supported tuple.
Reflection retrieves creation values; modifying a returned copy does not
modify either object.

Both object families derive from `ID3D12Pageable`. Successful creation returns
a caller-owned COM reference. The descriptions and their pointed-to creation
data are CPU call inputs and are copied during creation, including the heap's
resolution list. Resource storage, fences and the encoder/heap are separate
GPU-lifetime objects; retaining a CPU description is not a substitute for
retaining them.

`ID3D12VideoEncoderHeap1` adds the immutable `Pow2DownscaleFactor`. Lower-resolution
two-pass requires the frame-analysis heap flag and supported frame-analysis
configuration. The factor applies to every dynamic resolution in that heap's
list. Encoding performs no implicit downscaling. The application prepares the
downscaled input and maintains the mirrored full/downscaled DPB.
Changing the factor requires a new heap under the published key-frame/DPB
restrictions; supported older full-resolution frame analysis is not removed.

<a id="encoding-recording"></a>

## Recording, ordering, resources and lifetime

Encoding uses `D3D12_COMMAND_LIST_TYPE_VIDEO_ENCODE` lists, allocators and
queues. [Queue/list compatibility](execution.md#command-list-types),
[CPU threading](execution.md#threading), [submission threading](execution.md#submission-threading),
[submission ordering](execution.md#submission), [recording storage](execution.md#command-storage),
and [object/resource lifetime](execution.md#lifetime) retain their canonical
contracts. The base declaring entries for `ClearState`, `Close` and `Reset`
remain in [motion estimation](video-motion-estimation.md); they are not
declared again here.

CPU frame-argument structures, arrays and codec payloads are recording-call
inputs and must remain valid for that call. Their storage is distinct from
the referenced COM objects and GPU allocations; keeping only those CPU
structures does not retain the objects or make their backing memory resident.

The application serializes recording and submission against each encoder and
heap and records commands in GPU execution order. Multiple compatible
reconfiguration requests may be in flight against one heap, but this does not
permit concurrent unsynchronized access to the encoder/heap from independent
execution flows. Reconfiguration flags and capabilities do not mutate immutable
creation properties or silently change the codec headers/DPB.

The encoder, heap, picture resources, metadata/bitstream buffers, input-map
buffers and notification fences must remain alive for every recorded use
that will execute, including recordings not yet submitted, and until their
GPU uses complete. The caller retains its owned references; recording is not
assumed to take an additional COM reference. Resetting a list does not complete
earlier execution or reclaim an in-use allocator.

Input pictures, DPB references and opaque resolved input maps are read by
encoding; reconstructed pictures, compressed bitstreams and opaque output
metadata are written. Legacy transitions use `VIDEO_ENCODE_READ` and
`VIDEO_ENCODE_WRITE` as appropriate. Metadata resolution reads the opaque
metadata and writes portable destinations. Input-map resolution reads its
GPU textures and writes the opaque input buffer, which must be made readable
for the consuming encode.
[Enhanced encode barriers](synchronization.md#encode-enhanced-barrier),
[legacy encode barriers](synchronization.md#encode-resource-barrier),
[access scopes](synchronization.md#barrier-access),
[synchronization scopes](synchronization.md#barrier-sync), and
[barrier-model interoperation](synchronization.md#model-interoperation)
own the actual barrier contracts. These commands do not implicitly supply the
application's transitions, residency or cross-queue waits.

[Video resources](video.md#video-resource-contract) own common allocation and
residency. Input pictures must not be `VIDEO_ENCODE_REFERENCE_ONLY`.
Reference-only reconstructed pictures remain supported. Optional readable
reconstruction removes that requirement only for a tuple reporting the
readable-layout support flag; it must not hide a driver conversion/performance
penalty. The detailed readable-reconstruction contribution is retained in
[controls](video-encoding-controls.md#control-readable-reconstruction).

The application owns DPB allocation, snapshots, lifetime and reference strategy,
submission in encode rather than display order, and non-payload codec headers.
An IDR/key-frame or resolution change does not automatically implement the
host's DPB flushing, reference reordering or header production.

[Encode predication](counters-queries-and-predication.md#videoencode-setpredication)
and the [operation matrix](counters-queries-and-predication.md#predication-operation-matrix)
remain the policy owners; no software-only observation is generalized into
hardware policy here. [Encode queries](counters-queries-and-predication.md#videoencode-beginquery),
[timestamp ending](counters-queries-and-predication.md#videoencode-endquery), and
[query resolution](counters-queries-and-predication.md#videoencode-resolvequerydata)
own query eligibility and lifecycle. Encoder statistics are metadata, not
timestamp or completion queries.

<a id="encoding-metadata"></a>

## Metadata, optional statistics and early notifications

`EncodeFrame`/`EncodeFrame1` write hardware-layout metadata into the
application's opaque output buffer. `ResolveEncoderOutputMetadata` and its
version-1 form record conversion to portable records. The CPU argument
structures contain resource references, not the resolved record bytes.
The portable header and subregion records contain fixed-width values, not
pointers into a packed array.

For H.264/HEVC, the mandatory resolved layout is
`D3D12_VIDEO_ENCODER_OUTPUT_METADATA` immediately followed by
`WrittenSubregionsCount` packed `D3D12_VIDEO_ENCODER_FRAME_SUBREGION_METADATA`
records. AV1 additionally appends its tile-grid structure and post-encode
values. Resolved capacity is calculated from this layout and the maximum
supported subregion count; do not use the opaque hardware metadata size as
that calculation. Offsets and `bSize` are in bytes; `bHeaderSize` is
in bits. Each subregion's `bSize` includes its initial padding; `bStartOffset`
is the amount to skip within that region, not an absolute bitstream offset.

Optional QP/SATD/bit-allocation maps and frame/subregion PSNR retain separate
support, enable, sizing and output-destination rules. Disabling optional
statistics does not remove mandatory old metadata. Spatial adaptive QP has
separate support and enable flags and applies to all supported rate-control
modes, not just modes that budget a bitrate.

Subregion notification can make a slice/tile interval available before the
whole frame completes. It does not make frame statistics or AV1 post-encode
header values available early. The application supplies all output arrays,
base offsets, zeroed size/offset feedback buffers, fences and target values;
it waits without assuming notification order. `SUBREGIONS` notification
uses the subregion outputs rather than `FrameOutputBuffer` and reports zero
mandatory written-subregion count/total bitstream bytes, with no ordinary
subregion-record array. AV1's appended records still follow the mandatory
header and the zero-length array. The detailed contracts retain the exact
per-interval flush/signaling obligations, whole-frame completion fallback,
count and alignment rules.

The [controls reference](video-encoding-controls.md) retains the full
statistics, notification, input-map and two-pass contracts, including their
formulas, examples and figures. The [codec reference](video-encoding-codecs.md)
retains the additional AV1 metadata and header responsibilities.
The [driver reference](video-encoding-driver.md) contains actual published
versioned DDI payloads and callback types; API layouts are not assumed to be
DDI layouts.

<a id="encoding-api"></a>

## Declaring API reference

<a id="id3d12videodevice3--createvideoencoder"></a>

### ID3D12VideoDevice3::CreateVideoEncoder

#### Declaration

```cpp
HRESULT CreateVideoEncoder(
        const D3D12_VIDEO_ENCODER_DESC* pDesc,
        REFIID riid,
        void** ppVideoEncoder);
```

#### Behavior

Creates a video encoder instance.

#### Parameters

`pDesc` points to the input [`D3D12_VIDEO_ENCODER_DESC`](video-encoding-types.md#d3d12-video-encoder-desc).
`riid` identifies the requested interface. `ppVideoEncoder` receives a caller-owned
reference to `ID3D12VideoEncoder` on successful creation. CPU description storage, including
pointed-to profile and codec-configuration data, is copied
during creation; it is not GPU picture storage. Releasing the returned COM
reference remains the caller's responsibility.

#### Return values

Returns an HRESULT. The scoped inspected creation wrappers propagate validation
or object-creation failure; their null-output validation-only path returns
`S_FALSE` after successful validation and creates no object. This observation
does not change the required output annotation in the released declaration.

<a id="id3d12videodevice3--createvideoencoderheap"></a>

### ID3D12VideoDevice3::CreateVideoEncoderHeap

#### Declaration

```cpp
HRESULT CreateVideoEncoderHeap(
        const D3D12_VIDEO_ENCODER_HEAP_DESC* pDesc,
        REFIID riid,
        void** ppVideoEncoderHeap);
```

#### Behavior

Creates a video encoder heap instance.

#### Parameters

`pDesc` points to the input [`D3D12_VIDEO_ENCODER_HEAP_DESC`](video-encoding-types.md#d3d12-video-encoder-heap-desc).
`riid` identifies the requested interface. `ppVideoEncoderHeap` receives a caller-owned
reference to `ID3D12VideoEncoderHeap` on successful creation. CPU description storage, including
pointed-to profile/level data and the resolution list, is copied
during creation; it is not GPU picture storage. Releasing the returned COM
reference remains the caller's responsibility.

#### Return values

Returns an HRESULT. The scoped inspected creation wrappers propagate validation
or object-creation failure; their null-output validation-only path returns
`S_FALSE` after successful validation and creates no object. This observation
does not change the required output annotation in the released declaration.

<a id="id3d12videodevice4--createvideoencoderheap1"></a>

### ID3D12VideoDevice4::CreateVideoEncoderHeap1

#### Declaration

```cpp
HRESULT CreateVideoEncoderHeap1(
        const D3D12_VIDEO_ENCODER_HEAP_DESC1* pDesc,
        REFIID riid,
        void** ppVideoEncoderHeap);
```

#### Behavior

Creates a video encoder heap instance. The heap description additionally fixes `Pow2DownscaleFactor`.

#### Parameters

`pDesc` points to the input [`D3D12_VIDEO_ENCODER_HEAP_DESC1`](video-encoding-types.md#d3d12-video-encoder-heap-desc1).
`riid` identifies the requested interface. `ppVideoEncoderHeap` receives a caller-owned
reference to `ID3D12VideoEncoderHeap1` on successful creation. CPU description storage, including
pointed-to profile/level data and the resolution list, is copied
during creation; it is not GPU picture storage. Releasing the returned COM
reference remains the caller's responsibility.

#### Return values

Returns an HRESULT. The scoped inspected creation wrappers propagate validation
or object-creation failure; their null-output validation-only path returns
`S_FALSE` after successful validation and creates no object. This observation
does not change the required output annotation in the released declaration.

<a id="id3d12videoencoder--getnodemask"></a>

### ID3D12VideoEncoder::GetNodeMask

#### Declaration

```cpp
UINT GetNodeMask();
```

#### Behavior

Retrieves the immutable `NodeMask` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `NodeMask` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoder--getencoderflags"></a>

### ID3D12VideoEncoder::GetEncoderFlags

#### Declaration

```cpp
D3D12_VIDEO_ENCODER_FLAGS GetEncoderFlags();
```

#### Behavior

Retrieves the immutable `Flags` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `Flags` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoder--getcodec"></a>

### ID3D12VideoEncoder::GetCodec

#### Declaration

```cpp
D3D12_VIDEO_ENCODER_CODEC GetCodec();
```

#### Behavior

Retrieves the immutable `EncodeCodec` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `EncodeCodec` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoder--getcodecprofile"></a>

### ID3D12VideoEncoder::GetCodecProfile

#### Declaration

```cpp
HRESULT GetCodecProfile(D3D12_VIDEO_ENCODER_PROFILE_DESC dstProfile);
```

#### Behavior

Retrieves a copy of the immutable profile selected at creation; it does not reconfigure the object.

#### Parameters

`dstProfile` is passed by value. Initialize its `DataSize` and codec-selected
pointer to caller-owned writable storage of exactly the selected payload's
size. The getter writes that pointed-to payload; it does not allocate storage
or return a pointer into the object's private state.

#### Return values

The inspected getter returns `S_OK` on copying the payload and `E_INVALIDARG` for a null selected pointer, mismatched `DataSize`, or unrecognized codec. These are scoped runtime observations, not a hardware validation guarantee.

<a id="id3d12videoencoder--getcodecconfiguration"></a>

### ID3D12VideoEncoder::GetCodecConfiguration

#### Declaration

```cpp
HRESULT GetCodecConfiguration(D3D12_VIDEO_ENCODER_CODEC_CONFIGURATION dstCodecConfig);
```

#### Behavior

Retrieves a copy of the immutable codec configuration selected at creation; it does not reconfigure the object.

#### Parameters

`dstCodecConfig` is passed by value. Initialize its `DataSize` and codec-selected
pointer to caller-owned writable storage of exactly the selected payload's
size. The getter writes that pointed-to payload; it does not allocate storage
or return a pointer into the object's private state.

#### Return values

The inspected getter returns `S_OK` on copying the payload and `E_INVALIDARG` for a null selected pointer, mismatched `DataSize`, or unrecognized codec. These are scoped runtime observations, not a hardware validation guarantee.

<a id="id3d12videoencoder--getinputformat"></a>

### ID3D12VideoEncoder::GetInputFormat

#### Declaration

```cpp
DXGI_FORMAT GetInputFormat();
```

#### Behavior

Retrieves the immutable `InputFormat` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `InputFormat` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoder--getmaxmotionestimationprecision"></a>

### ID3D12VideoEncoder::GetMaxMotionEstimationPrecision

#### Declaration

```cpp
D3D12_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE GetMaxMotionEstimationPrecision();
```

#### Behavior

Retrieves the immutable `MaxMotionEstimationPrecision` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `MaxMotionEstimationPrecision` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoderheap--getnodemask"></a>

### ID3D12VideoEncoderHeap::GetNodeMask

#### Declaration

```cpp
UINT GetNodeMask();
```

#### Behavior

Retrieves the immutable `NodeMask` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `NodeMask` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoderheap--getencoderheapflags"></a>

### ID3D12VideoEncoderHeap::GetEncoderHeapFlags

#### Declaration

```cpp
D3D12_VIDEO_ENCODER_HEAP_FLAGS GetEncoderHeapFlags();
```

#### Behavior

Retrieves the immutable `Flags` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `Flags` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoderheap--getcodec"></a>

### ID3D12VideoEncoderHeap::GetCodec

#### Declaration

```cpp
D3D12_VIDEO_ENCODER_CODEC GetCodec();
```

#### Behavior

Retrieves the immutable `EncodeCodec` value used at object creation.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `EncodeCodec` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoderheap--getcodecprofile"></a>

### ID3D12VideoEncoderHeap::GetCodecProfile

#### Declaration

```cpp
HRESULT GetCodecProfile(D3D12_VIDEO_ENCODER_PROFILE_DESC dstProfile);
```

#### Behavior

Retrieves a copy of the immutable profile selected at creation; it does not reconfigure the object.

#### Parameters

`dstProfile` is passed by value. Initialize its `DataSize` and codec-selected
pointer to caller-owned writable storage of exactly the selected payload's
size. The getter writes that pointed-to payload; it does not allocate storage
or return a pointer into the object's private state.

#### Return values

The inspected getter returns `S_OK` on copying the payload and `E_INVALIDARG` for a null selected pointer, mismatched `DataSize`, or unrecognized codec. These are scoped runtime observations, not a hardware validation guarantee.

<a id="id3d12videoencoderheap--getcodeclevel"></a>

### ID3D12VideoEncoderHeap::GetCodecLevel

#### Declaration

```cpp
HRESULT GetCodecLevel(D3D12_VIDEO_ENCODER_LEVEL_SETTING dstLevel);
```

#### Behavior

Retrieves a copy of the immutable level selected at creation; it does not reconfigure the object.

#### Parameters

`dstLevel` is passed by value. Initialize its `DataSize` and codec-selected
pointer to caller-owned writable storage of exactly the selected payload's
size. The getter writes that pointed-to payload; it does not allocate storage
or return a pointer into the object's private state.

#### Return values

The inspected getter returns `S_OK` on copying the payload and `E_INVALIDARG` for a null selected pointer, mismatched `DataSize`, or unrecognized codec. These are scoped runtime observations, not a hardware validation guarantee.

<a id="id3d12videoencoderheap--getresolutionlistcount"></a>

### ID3D12VideoEncoderHeap::GetResolutionListCount

#### Declaration

```cpp
UINT GetResolutionListCount();
```

#### Behavior

Retrieves the immutable `ResolutionsListCount` value used at object creation. This count sizes the destination for `GetResolutionList`; it is not an adapter-wide list of supported resolutions.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `ResolutionsListCount` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencoderheap--getresolutionlist"></a>

### ID3D12VideoEncoderHeap::GetResolutionList

#### Declaration

```cpp
HRESULT GetResolutionList(
        const UINT ResolutionsListCount,
        D3D12_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC* pResolutionList);
```

#### Behavior

Retrieves the immutable list of resolutions requested when creating this heap.

#### Parameters

`ResolutionsListCount` must equal `GetResolutionListCount()`.
`pResolutionList` points to caller-allocated writable storage for exactly that
many `D3D12_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC` entries. This is a copy of the
heap's list, not a query that allocates or grows a returned array.

#### Return values

The inspected getter returns `S_OK` on copying the list and `E_INVALIDARG` for a count unequal to the stored count. No capacity/MORE_DATA convention is established.

<a id="id3d12videoencoderheap1--getpow2downscalefactor"></a>

### ID3D12VideoEncoderHeap1::GetPow2DownscaleFactor

#### Declaration

```cpp
UINT GetPow2DownscaleFactor();
```

#### Behavior

Retrieves the immutable `Pow2DownscaleFactor` value used at object creation. When the heap was not created with `D3D12_VIDEO_ENCODER_HEAP_FLAG_ALLOW_RATE_CONTROL_FRAME_ANALYSIS`, the published contract returns zero.

#### Parameters

This method has no parameters.

#### Return values

Returns the stored `Pow2DownscaleFactor` value directly; no HRESULT or additional COM reference is returned.

<a id="id3d12videoencodecommandlist2--encodeframe"></a>

### ID3D12VideoEncodeCommandList2::EncodeFrame

#### Declaration

```cpp
void EncodeFrame(
        ID3D12VideoEncoder* pEncoder,
        ID3D12VideoEncoderHeap *pHeap,
        const D3D12_VIDEO_ENCODER_ENCODEFRAME_INPUT_ARGUMENTS *pInputArguments,
        const D3D12_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_ARGUMENTS *pOutputArguments);
```

#### Behavior

Records encoding of a frame's codec payload. The application supplies the
sequence controls, picture controls and DPB snapshot and codes the remaining
bitstream headers. [Recording and lifetime](#encoding-recording) and the
[detailed controls reference](video-encoding-controls.md) apply.
This is the supported older full-frame path. It uses the original picture-control
union, including HEVC/HEVC1, and does not request optional metadata or GPU map
inputs. Legacy full-resolution frame analysis remains available when supported.

#### Parameters

`pEncoder` is the encoder for this operation. `pHeap` is the compatible encoder
heap. `pInputArguments` points to
[`D3D12_VIDEO_ENCODER_ENCODEFRAME_INPUT_ARGUMENTS`](video-encoding-types.md#d3d12-video-encoder-encodeframe-input-arguments).
`pOutputArguments` points to
[`D3D12_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_ARGUMENTS`](video-encoding-types.md#d3d12-video-encoder-encodeframe-output-arguments),
which identifies application allocations written by GPU execution, not CPU
output storage written when recording returns.

#### Return values

Returns no value. Recording returns before GPU completion. Execution errors are reported through encoder metadata; recording/list errors are distinct.

<a id="id3d12videoencodecommandlist2--resolveencoderoutputmetadata"></a>

### ID3D12VideoEncodeCommandList2::ResolveEncoderOutputMetadata

#### Declaration

```cpp
void ResolveEncoderOutputMetadata(
        const D3D12_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS* pInputArguments,
        const D3D12_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS* pOutputArguments);
```

#### Behavior

Records conversion of the opaque encoder output into the portable resolved
metadata layout. It does not encode a frame or generate the host's codec headers.
The source codec/profile/format/resolution must describe the associated encoding.
This form resolves the mandatory metadata. Optional statistics use
`ResolveEncoderOutputMetadata1`, not this method.

#### Parameters

`pInputArguments` points to
[`D3D12_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS`](video-encoding-types.md#d3d12-video-encoder-resolve-metadata-input-arguments),
including the opaque input buffer and byte offset. `pOutputArguments` points to
[`D3D12_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS`](video-encoding-types.md#d3d12-video-encoder-resolve-metadata-output-arguments),
including application-owned resolved destinations and offsets. These destinations
are GPU outputs. The mandatory layout remains unchanged for H.264/HEVC; AV1
appends its tile-grid and post-encode records. This form has no optional-statistics output destinations.

#### Return values

Returns no value. The resolved records become consumable only after the application establishes GPU completion and the required resource access/synchronization.

<a id="id3d12videoencodecommandlist4--encodeframe1"></a>

### ID3D12VideoEncodeCommandList4::EncodeFrame1

#### Declaration

```cpp
void EncodeFrame1(
        ID3D12VideoEncoder* pEncoder,
        ID3D12VideoEncoderHeap1* pHeap,
        const D3D12_VIDEO_ENCODER_ENCODEFRAME_INPUT_ARGUMENTS1* pInputArguments,
        const D3D12_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_ARGUMENTS1* pOutputArguments);
```

#### Behavior

Records encoding of a frame's codec payload. The application supplies the
sequence controls, picture controls and DPB snapshot and codes the remaining
bitstream headers. [Recording and lifetime](#encoding-recording) and the
[detailed controls reference](video-encoding-controls.md) apply.
This version uses HEVC2 through the version-1 picture-control union, optional
statistics, CPU/GPU map inputs, subregion notification and frame-analysis
parameters. Enabling one optional feature does not imply support for another.

#### Parameters

`pEncoder` is the encoder for this operation. `pHeap` is the compatible encoder
heap. `pInputArguments` points to
[`D3D12_VIDEO_ENCODER_ENCODEFRAME_INPUT_ARGUMENTS1`](video-encoding-types.md#d3d12-video-encoder-encodeframe-input-arguments1).
`pOutputArguments` points to
[`D3D12_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_ARGUMENTS1`](video-encoding-types.md#d3d12-video-encoder-encodeframe-output-arguments1),
which identifies application allocations written by GPU execution, not CPU
output storage written when recording returns.

#### Return values

Returns no value. Recording returns before GPU completion. Execution errors are reported through encoder metadata; recording/list errors are distinct.

<a id="id3d12videoencodecommandlist4--resolveencoderoutputmetadata1"></a>

### ID3D12VideoEncodeCommandList4::ResolveEncoderOutputMetadata1

#### Declaration

```cpp
void ResolveEncoderOutputMetadata1(
        const D3D12_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS1* pInputArguments,
        const D3D12_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS1* pOutputArguments);
```

#### Behavior

Records conversion of the opaque encoder output into the portable resolved
metadata layout. It does not encode a frame or generate the host's codec headers.
The source codec/profile/format/resolution must describe the associated encoding.
This version additionally selects the optional statistics collected for
that frame and supplies the associated codec configuration.

#### Parameters

`pInputArguments` points to
[`D3D12_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS1`](video-encoding-types.md#d3d12-video-encoder-resolve-metadata-input-arguments1),
including the opaque input buffer and byte offset. `pOutputArguments` points to
[`D3D12_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS1`](video-encoding-types.md#d3d12-video-encoder-resolve-metadata-output-arguments1),
including application-owned resolved destinations and offsets. These destinations
are GPU outputs. The mandatory layout remains unchanged for H.264/HEVC; AV1
appends its tile-grid and post-encode records. Optional statistics use their separate output textures/buffers.

#### Return values

Returns no value. The resolved records become consumable only after the application establishes GPU completion and the required resource access/synchronization.

<a id="id3d12videoencodecommandlist4--resolveinputparamlayout"></a>

### ID3D12VideoEncodeCommandList4::ResolveInputParamLayout

#### Declaration

```cpp
void ResolveInputParamLayout(
        const D3D12_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_INPUT_ARGUMENTS* pInputArguments,
        const D3D12_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_OUTPUT_ARGUMENTS* pOutputArguments);
```

#### Behavior

Converts hardware-agnostic GPU map textures into the hardware-specific opaque
input layout consumed by `EncodeFrame1`. Record it before the consuming encode
on the same list, or complete it and synchronize the producer/consumer queues
with fences. It is not metadata-output resolution.

#### Parameters

`pInputArguments` supplies session information and the selected QP, dirty-region
or motion-vector map data. `pOutputArguments` identifies the application-owned
opaque output buffer, allocated to
`D3D12_FEATURE_DATA_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT.MaxResolvedBufferAllocationSize`.
Inputs are VIDEO_ENCODE_READ; the destination is VIDEO_ENCODE_WRITE for the resolve,
then VIDEO_ENCODE_READ when consumed by encoding. The opaque layout is not a
portable CPU record format.

#### Return values

Returns no value. It records work; it neither waits for completion nor inserts the caller's cross-queue fence dependencies.
