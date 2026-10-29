<a id="video-encoding-driver"></a>

# Video encoding driver reference

**Qualified draft.** This supporting reference supplies selected published
encoding capability payloads, versioned layouts, private-size/create/destroy
and recording callbacks, and callback-table associations.
[Video encoding](video-encoding.md) owns the application-facing contract;
its API declarations belong to the [type reference](video-encoding-types.md).
API structure shapes are not assumed to be DDI binary layouts.

<a id="source-s01-091"></a>

### PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0

Published callback: [`PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0`](video-encoding-driver.md#pfnd3d12ddi-calcprivatevideoencodersize-0082-0).

This method is used to calculate the required driver object size. The D3D runtime allocates the memory for storing the driver's object representing the video encoder.

**Parameters**

*hDrvDevice*

Handle for the D3D12 device.

*pCreationArgs*

Encoder creation parameters.

<a id="source-s01-092"></a>

### PFND3D12DDI_CREATEVIDEOENCODER_0082_0

Published callback: [`PFND3D12DDI_CREATEVIDEOENCODER_0082_0`](video-encoding-driver.md#pfnd3d12ddi-createvideoencoder-0082-0).

Creates a video encoder object.

**Parameters**

*hDrvDevice*

Handle for the D3D12 device.

*pCreationArgs*

Encoder creation parameters.

*hDrvVideoEncoder*

Handle to the hDrvVideoEncoder. Runtime has allocated space according to the result of PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0 for the driver to place its object.

<a id="source-s01-093"></a>

### PFND3D12DDI_DESTROYVIDEOENCODER_0080

Published callback: [`PFND3D12DDI_DESTROYVIDEOENCODER_0080`](video-encoding-driver.md#pfnd3d12ddi-destroyvideoencoder-0080).

**Parameters**

*hDrvDevice*

Handle for the D3D12 device.

*hDrvVideoEncoder*

Handle to the hDrvVideoEncoder.

<a id="source-s01-095"></a>

### PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0080_2

Published callback: [`PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0080_2`](video-encoding-driver.md#pfnd3d12ddi-calcprivatevideoencoderheapsize-0080-2).

This method is used to calculate the required driver object size. The D3D runtime allocates the memory for storing the driver's object representing the video encoder heap.

**Parameters**

*hDrvDevice*

Handle for the D3D12 device.

*pCreationArgs*

Encoder heap creation parameters.

<a id="source-s01-096"></a>

### PFND3D12DDI_CREATEVIDEOENCODERHEAP_0080_2

Published callback: [`PFND3D12DDI_CREATEVIDEOENCODERHEAP_0080_2`](video-encoding-driver.md#pfnd3d12ddi-createvideoencoderheap-0080-2).

Creates a video encoder heap object.

**Parameters**

*hDrvDevice*

Handle for the D3D12 device.

*pCreationArgs*

Encoder heap creation parameters.

*hDrvVideoEncoderHeap*

Handle to the hDrvVideoEncoderHeap. Runtime has allocated space according to the result of PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0080_2 for the driver to place its object.

<a id="source-s01-097"></a>

### PFND3D12DDI_DESTROYVIDEOENCODERHEAP_0080

Published callback: [`PFND3D12DDI_DESTROYVIDEOENCODERHEAP_0080`](video-encoding-driver.md#pfnd3d12ddi-destroyvideoencoderheap-0080).

**Parameters**

*hDrvDevice*

Handle for the D3D12 device.

*hDrvVideoEncoderHeap*

Handle to the hDrvVideoEncoderHeap.

<a id="source-s01-099"></a>

### Video-device callback tables

The [published video-device callback tables](#encoding-ddi-table-associations) contain the encoder/heap creation and destruction slots.

The [published callback-table associations](video-encoding-driver.md#encoding-ddi-table-associations) identify the actual video-device and encode-list slots.
---

<a id="source-s01-145"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0082_0

Released declaration: [`D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0082_0`](video-encoding-driver.md#d3d12ddi-video-encoder-encodeframe-input-stream-arguments-0082-0).

Parameters for the input of the encoding operation.

<a id="source-s01-149"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0080_2

Released declaration: [`D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0080_2`](video-encoding-driver.md#d3d12ddi-video-encoder-encodeframe-output-stream-arguments-0080-2).

Parameters for the output of the encoding operation.

<a id="source-s01-150"></a>

### Frame-recording callback

Published callback: [`PFND3D12DDI_VIDEO_ENCODE_FRAME_0082_0`](video-encoding-driver.md#pfnd3d12ddi-video-encode-frame-0082-0).

Records an encode frame operation to the command list.

<a id="source-s01-151"></a>

### Metadata-resolution callback

Published callback: [`PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0082_0`](video-encoding-driver.md#pfnd3d12ddi-video-encode-resolve-output-metadata-0082-0).

Records a command to resolve a encoding operation output metadata into a readable format.

<a id="encoding-published-ddi"></a>

## Published versioned layouts and callbacks

The following are actual published WDK declarations, not API-shaped
pseudocode. Resource and fence parameters are driver handles; size/offset
feedback are arrays of resource handles. Creation has separate private-size,
create and destroy callbacks. Encode, output-metadata resolution and input-map
resolution are different recording callbacks. Numeric suffixes are part of
the DDI identity, not an API interface version. The callback table associations
below identify the published slots; they do not imply a one-to-one mapping
from an API interface revision to one driver table.

The common [DDI capability-payload protocol](video-decode.md#ddi-capability-payloads)
owns capability-query framing, not encoding object lifetime.
[Core object/resource lifetime](execution.md#lifetime) and
[residency](resources.md#residency) apply together with this family's
[immutable objects](video-encoding.md#encoding-objects),
[recording/ordering/lifetime contract](video-encoding.md#encoding-recording)
and [private-size/create/destroy callbacks](#source-s01-091).
The callback descriptions and [published slots](#encoding-ddi-table-associations)
state this family's allocation responsibilities; motion-estimator or
decoder-specific allocation/cross-queue rules are not inherited here.
The selected native observation
corroborates creation copies, exact-count reflection and wrapper dispatch only.
It does not establish each driver's codec implementation, error timing,
precision, performance, or device-specific support.

For the selected current API and public WDK declarations, the
[API metadata layout](video-encoding.md#encoding-metadata) corresponds to
[`D3D12DDI_VIDEO_ENCODER_OUTPUT_METADATA_0083_0`](#d3d12ddi-video-encoder-output-metadata-0083-0),
its statistics and the 64-bit subregion records. This is a selected-layout
correspondence, not a record-selection rule for every historically negotiated
DDI. The `_0082_0` callback suffix alone establishes neither selection of the
older `_0082_0` record nor universal selection of `_0083_0` on historical paths.
Older record references retained in the original published header comments
do not settle that version question. Historical selection remains qualified
pending reconciliation with the applicable negotiated-version contract;
the current API's portable layout is not changed by that unresolved question.
AV1 adds its tile grid and post-encode record;
optional statistics retain their separate destinations.

API object/resource interfaces are translated to driver handles. In particular,
`pInputFrame` becomes `hDrvInputTexture2D`, reconstructed/reference textures
become resource handles, and subregion resources/fences become arrays of
resource/fence handles. Byte offsets are retained as explicit offset fields.
The API version-1 HEVC union selects HEVC2; its published DDI counterpart is
the `_0108_0` picture-control union. These named translations are not permission
to reinterpret an entire API argument structure as a DDI structure.

<a id="encoding-ddi-capability-cases"></a>

## Encoding capability cases

These are the published `D3D12DDICAPS_TYPE` cases and the selected
versioned capability payloads. The case values are DDI values, not the
`D3D12_FEATURE_VIDEO` API ordinals. The fixed `pData` size is `sizeof` the
selected payload; codec-specific pointers and arrays have their own storage.
Input/output directions are retained in the payloads and in the matching
[API capability contracts](video-encoding-controls.md#control-capabilities).

| DDI capability case | Value | Payload |
|---|---|---|
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_CODEC` | 31 | [`D3D12DDICAPS_VIDEO_ENCODER_CODEC_DATA_0080`](#d3d12ddicaps-video-encoder-codec-data-0080) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_PROFILE_LEVEL` | 32 | [`D3D12DDICAPS_VIDEO_ENCODER_PROFILE_LEVEL_DATA_0080_2`](#d3d12ddicaps-video-encoder-profile-level-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_OUTPUT_RESOLUTION_RATIOS_COUNT` | 33 | [`D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_RATIOS_COUNT_DATA_0080_2`](#d3d12ddicaps-video-encoder-output-resolution-ratios-count-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_OUTPUT_RESOLUTION` | 34 | [`D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_DATA_0080_2`](#d3d12ddicaps-video-encoder-output-resolution-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_INPUT_FORMAT` | 35 | [`D3D12DDICAPS_VIDEO_ENCODER_INPUT_FORMAT_DATA_0080_2`](#d3d12ddicaps-video-encoder-input-format-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_RATE_CONTROL_MODE` | 36 | [`D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_MODE_DATA_0080`](#d3d12ddicaps-video-encoder-rate-control-mode-data-0080) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_INTRA_REFRESH_MODE` | 37 | [`D3D12DDICAPS_VIDEO_ENCODER_INTRA_REFRESH_MODE_DATA_0080_2`](#d3d12ddicaps-video-encoder-intra-refresh-mode-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_FRAME_SUBREGION_LAYOUT_MODE` | 38 | [`D3D12DDICAPS_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_DATA_0080_2`](#d3d12ddicaps-video-encoder-frame-subregion-layout-mode-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_HEAP_SIZE` | 39 | [`D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0080_2`](#d3d12ddicaps-video-encoder-heap-size-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_CODEC_CONFIGURATION_SUPPORT` | 40 | [`D3D12DDICAPS_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_DATA_0083_0`](#d3d12ddicaps-video-encoder-codec-configuration-support-data-0083-0) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_SUPPORT` | 41 | [`D3D12DDICAPS_VIDEO_ENCODER_SUPPORT_DATA_0083_0`](#d3d12ddicaps-video-encoder-support-data-0083-0) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT` | 42 | [`D3D12DDICAPS_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_DATA_0080_2`](#d3d12ddicaps-video-encoder-codec-picture-control-support-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0080_ENCODER_RESOURCE_REQUIREMENTS` | 43 | [`D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS_DATA_0080_2`](#d3d12ddicaps-video-encoder-resource-requirements-data-0080-2) |
| `D3D12DDICAPS_TYPE_VIDEO_0095_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG` | 44 | [`D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_0095`](#d3d12ddi-feature-data-video-encoder-frame-subregion-layout-config-0095) |
| `D3D12DDICAPS_TYPE_VIDEO_0096_ENCODER_SUPPORT1` | 45 | [`D3D12DDICAPS_VIDEO_ENCODER_SUPPORT1_DATA_0096`](#d3d12ddicaps-video-encoder-support1-data-0096) |
| `D3D12DDICAPS_TYPE_VIDEO_0102_ENCODER_RESOURCE_REQUIREMENTS1` | 47 | [`D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS1_DATA_0102`](#d3d12ddicaps-video-encoder-resource-requirements1-data-0102) |
| `D3D12DDICAPS_TYPE_VIDEO_0103_ENCODER_QPMAP_INPUT` | 48 | [`D3D12DDICAPS_VIDEO_ENCODER_QPMAP_INPUT_0103`](#d3d12ddicaps-video-encoder-qpmap-input-0103) |
| `D3D12DDICAPS_TYPE_VIDEO_0103_ENCODER_DIRTY_REGIONS` | 49 | [`D3D12DDICAPS_ENCODER_DIRTY_REGIONS_0103`](#d3d12ddicaps-encoder-dirty-regions-0103) |
| `D3D12DDICAPS_TYPE_VIDEO_0107_ENCODER_MOTION_SEARCH` | 50 | [`D3D12DDICAPS_VIDEO_ENCODER_MOTION_SEARCH_0107`](#d3d12ddicaps-video-encoder-motion-search-0107) |
| `D3D12DDICAPS_TYPE_VIDEO_0103_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT` | 51 | [`D3D12DDICAPS_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_0103`](#d3d12ddicaps-video-encoder-resolve-input-param-layout-0103) |
| `D3D12DDICAPS_TYPE_VIDEO_0107_ENCODER_SUPPORT2` | 54 | [`D3D12DDICAPS_VIDEO_ENCODER_SUPPORT2_DATA_0107`](#d3d12ddicaps-video-encoder-support2-data-0107) |
| `D3D12DDICAPS_TYPE_VIDEO_0110_ENCODER_HEAP_SIZE1` | 55 | [`D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0110_0`](#d3d12ddicaps-video-encoder-heap-size-data-0110-0) |
| `D3D12DDICAPS_TYPE_VIDEO_0110_ENCODER_RATE_CONTROL_FRAME_ANALYSIS` | 56 | [`D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_DATA_0110_0`](#d3d12ddicaps-video-encoder-rate-control-frame-analysis-data-0110-0) |

<a id="d3d12ddiarg-create-video-encoder-0082-0"></a>

### D3D12DDIARG_CREATE_VIDEO_ENCODER_0082_0

```cpp
typedef struct D3D12DDIARG_CREATE_VIDEO_ENCODER_0082_0
{
    UINT NodeMask;
    D3D12DDI_VIDEO_ENCODER_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 EncodeCodec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 EncodeProfile;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080 MaxMotionEstimationPrecision;
} D3D12DDIARG_CREATE_VIDEO_ENCODER_0082_0;
```

<a id="d3d12ddiarg-create-video-encoder-heap-0080-2"></a>

### D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0080_2

```cpp
typedef struct D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0080_2
{
    UINT NodeMask;
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 EncodeCodec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 EncodeProfile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 EncodeLevel;
    UINT ResolutionsListCount;
    const D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080* pResolutionList;
} D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0080_2;
```

<a id="d3d12ddiarg-create-video-encoder-heap-0110-0"></a>

### D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0110_0

```cpp
typedef struct D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0110_0
{
    UINT NodeMask;
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 EncodeCodec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 EncodeProfile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 EncodeLevel;
    UINT ResolutionsListCount;
    const D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080* pResolutionList;
    UINT Pow2DownscaleFactor;
} D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0110_0;
```

<a id="d3d12ddicaps-encoder-dirty-regions-0103"></a>

### D3D12DDICAPS_ENCODER_DIRTY_REGIONS_0103

```cpp
typedef struct D3D12DDICAPS_ENCODER_DIRTY_REGIONS_0103
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103 SessionInfo;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103 MapValuesType;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAGS_0104 SupportFlags;
    UINT MapSourcePreferenceRanking;
} D3D12DDICAPS_ENCODER_DIRTY_REGIONS_0103;
```

<a id="d3d12ddicaps-video-encoder-codec-configuration-support-data-0083-0"></a>

### D3D12DDICAPS_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_DATA_0083_0

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_DATA_0083_0
{
    UINT NodeIndex;                                                              // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                                     // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;                            // input
    BOOL IsSupported;                                                            // output
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_0083_0 CodecSupportLimits;  // output
} D3D12DDICAPS_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_DATA_0083_0;
```

<a id="d3d12ddicaps-video-encoder-codec-data-0080"></a>

### D3D12DDICAPS_VIDEO_ENCODER_CODEC_DATA_0080

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_CODEC_DATA_0080
{
    UINT NodeIndex;                             // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;    // input
    BOOL IsSupported;                           // output
} D3D12DDICAPS_VIDEO_ENCODER_CODEC_DATA_0080;
```

<a id="d3d12ddicaps-video-encoder-codec-picture-control-support-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_DATA_0080_2
{
    UINT NodeIndex;                                                              // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                                     // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;                            // input
    BOOL IsSupported;                                                            // output
    D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_0080_2 PictureSupport;// output
} D3D12DDICAPS_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-frame-subregion-layout-mode-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_DATA_0080_2
{
    UINT NodeIndex;                                                         // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                                // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;                       // input
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 Level;                        // input
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionMode;  // input
    BOOL IsSupported;                                                       // output
} D3D12DDICAPS_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-heap-size-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0080_2
{
  D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0080_2 HeapDesc;  // input
  BOOL                          IsSupported;            // output
  UINT64                        MemoryPoolL0Size;       // output
  UINT64                        MemoryPoolL1Size;       // output
} D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-heap-size-data-0110-0"></a>

### D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0110_0

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0110_0
{
  D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0110_0 HeapDesc;  // input
  BOOL                          IsSupported;            // output
  UINT64                        MemoryPoolL0Size;       // output
  UINT64                        MemoryPoolL1Size;       // output
} D3D12DDICAPS_VIDEO_ENCODER_HEAP_SIZE_DATA_0110_0;
```

<a id="d3d12ddicaps-video-encoder-input-format-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_INPUT_FORMAT_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_INPUT_FORMAT_DATA_0080_2
{
    UINT NodeIndex;                                    // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;           // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;  // input
    DXGI_FORMAT Format;                                // input
    BOOL IsSupported;                                  // output
} D3D12DDICAPS_VIDEO_ENCODER_INPUT_FORMAT_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-intra-refresh-mode-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_INTRA_REFRESH_MODE_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_INTRA_REFRESH_MODE_DATA_0080_2
{
    UINT NodeIndex;                                                     // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                            // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;                   // input
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 Level;                    // input
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080 IntraRefreshMode;    // input
    BOOL IsSupported;                                                   // output
} D3D12DDICAPS_VIDEO_ENCODER_INTRA_REFRESH_MODE_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-motion-search-0107"></a>

### D3D12DDICAPS_VIDEO_ENCODER_MOTION_SEARCH_0107

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_MOTION_SEARCH_0107
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103 SessionInfo;
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103 MotionSearchMode;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    BOOL BidirectionalRefFrameEnabled;
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAGS_0107 SupportFlags;
    UINT MaxMotionHints;
    UINT MinDeviation;
    UINT MaxDeviation;
    UINT MapSourcePreferenceRanking;
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAGS_0105 MotionUnitPrecisionSupport;
} D3D12DDICAPS_VIDEO_ENCODER_MOTION_SEARCH_0107;
```

<a id="d3d12ddicaps-video-encoder-output-resolution-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_DATA_0080_2
{
    UINT NodeIndex;                                                      // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                             // input
    UINT ResolutionRatiosCount;                                             // input
    BOOL IsSupported;                                                         // output
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 MinResolutionSupported; // output
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 MaxResolutionSupported; // output
    UINT ResolutionWidthMultipleRequirement;    // output
    UINT ResolutionHeightMultipleRequirement;   // output
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_RATIO_DESC_0080* pResolutionRatios;
} D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-output-resolution-ratios-count-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_RATIOS_COUNT_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_RATIOS_COUNT_DATA_0080_2
{
    UINT NodeIndex;                                                      // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                             // input
    UINT ResolutionRatiosCount;                                          // output
} D3D12DDICAPS_VIDEO_ENCODER_OUTPUT_RESOLUTION_RATIOS_COUNT_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-profile-level-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_PROFILE_LEVEL_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_PROFILE_LEVEL_DATA_0080_2
{
    UINT NodeIndex;                                              // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                     // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;            // input
    BOOL IsSupported;                                            // output
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 MinSupportedLevel; // output
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 MaxSupportedLevel; // output
} D3D12DDICAPS_VIDEO_ENCODER_PROFILE_LEVEL_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-qpmap-input-0103"></a>

### D3D12DDICAPS_VIDEO_ENCODER_QPMAP_INPUT_0103

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_QPMAP_INPUT_0103
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103 SessionInfo;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    BOOL IsSupported;
    UINT MapSourcePreferenceRanking;
    UINT BlockSize;
} D3D12DDICAPS_VIDEO_ENCODER_QPMAP_INPUT_0103;
```

<a id="d3d12ddicaps-video-encoder-rate-control-frame-analysis-data-0110-0"></a>

### D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_DATA_0110_0

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_DATA_0110_0
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 Level;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 InputResolution;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionFrameEncoding;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2 SubregionFrameEncodingData;
    D3D12DDI_VIDEO_ENCODER_QPMAP_CONFIGURATION_0107 QPMap;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_CONFIGURATION_0107 DirtyRegions;
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_CONFIGURATION_0107 MotionSearch;
    UINT Pow2DownscaleFactor;
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAGS_0110_0 SupportFlags;
} D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_DATA_0110_0;
```

<a id="d3d12ddicaps-video-encoder-rate-control-frame-analysis-support-flags-0110-0"></a>

### D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAGS_0110_0

```cpp
typedef enum D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAGS_0110_0
{
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_NONE = 0x0,
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_INTRACODED_FRAME_SUPPORTED = 0x1,
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_UNIDIR_INTER_FRAME_SUPPORTED = 0x2,
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_BIDIR_INTER_FRAME_SUPPORTED = 0x4,
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_EXTERNAL_DPB_DOWNSCALING = 0x8,
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_DYNAMIC_1ST_PASS_SKIP = 0x10,
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_DYNAMIC_DOWNSCALE_FACTOR_CHANGE_KEY_FRAME = 0x20,

    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_SUPPORTED =
        D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_INTRACODED_FRAME_SUPPORTED |
        D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_UNIDIR_INTER_FRAME_SUPPORTED |
        D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAG_0110_0_BIDIR_INTER_FRAME_SUPPORTED,

} D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAGS_0110_0;
```

<a id="d3d12ddicaps-video-encoder-rate-control-mode-data-0080"></a>

### D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_MODE_DATA_0080

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_MODE_DATA_0080
{
    UINT NodeIndex;                                                 // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                        // input
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080 RateControlMode;  // input
    BOOL IsSupported;                                               // output
} D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_MODE_DATA_0080;
```

<a id="d3d12ddicaps-video-encoder-resolve-input-param-layout-0103"></a>

### D3D12DDICAPS_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_0103

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_0103
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103 SessionInfo;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103 MapType;
    BOOL IsSupported;
    UINT64 MaxResolvedBufferAllocationSize;
} D3D12DDICAPS_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_0103;
```

<a id="d3d12ddicaps-video-encoder-resource-requirements1-data-0102"></a>

### D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS1_DATA_0102

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS1_DATA_0102
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 PictureTargetResolution;

    BOOL IsSupported;
    UINT CompressedBitstreamBufferAccessAlignment;
    UINT EncoderMetadataBufferAccessAlignment;
    UINT MaxEncoderOutputMetadataBufferSize;
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0 OptionalMetadata;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 EncoderOutputMetadataQPMapTextureDimensions;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 EncoderOutputMetadataSATDMapTextureDimensions;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 EncoderOutputMetadataBitAllocationMapTextureDimensions;
    UINT EncoderOutputMetadataFramePSNRComponentsNumber;
    UINT EncoderOutputMetadataSubregionsPSNRComponentsNumber;
    UINT EncoderOutputMetadataSubregionsPSNRResolvedMetadataBufferSize;
} D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS1_DATA_0102;
```

<a id="d3d12ddicaps-video-encoder-resource-requirements-data-0080-2"></a>

### D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS_DATA_0080_2

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS_DATA_0080_2
{
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 PictureTargetResolution;

    BOOL IsSupported;
    UINT CompressedBitstreamBufferAccessAlignment;
    UINT EncoderMetadataBufferAccessAlignment;
    UINT MaxEncoderOutputMetadataBufferSize;
} D3D12DDICAPS_VIDEO_ENCODER_RESOURCE_REQUIREMENTS_DATA_0080_2;
```

<a id="d3d12ddicaps-video-encoder-support1-data-0096"></a>

### D3D12DDICAPS_VIDEO_ENCODER_SUPPORT1_DATA_0096

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_SUPPORT1_DATA_0096
{
    /*
    * This struct is an expansion on D3D12DDICAPS_VIDEO_ENCODER_SUPPORT_DATA_0083_0
    * First all the exact same parameters as in it are defined
    * After that, new arguments are added at the bottom of this new struct.
    */

    // D3D12DDICAPS_VIDEO_ENCODER_SUPPORT_DATA_0083_0
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0 CodecGopSequence;
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2 RateControl;
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080 IntraRefresh;
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionFrameEncoding;
    UINT ResolutionsListCount;
    const D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080* pResolutionList;
    UINT MaxReferenceFramesInDPB;
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAGS_0082_0 ValidationFlags;
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAGS_0083_0 SupportFlags;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 SuggestedProfile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 SuggestedLevel;
    D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2* pResolutionDependentSupport;

    // New arguments
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2 SubregionFrameEncodingData;
    UINT MaxQualityVsSpeed;
} D3D12DDICAPS_VIDEO_ENCODER_SUPPORT1_DATA_0096;
```

<a id="d3d12ddicaps-video-encoder-support2-data-0107"></a>

### D3D12DDICAPS_VIDEO_ENCODER_SUPPORT2_DATA_0107

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_SUPPORT2_DATA_0107
{
    /*
     * Below params match D3D12DDICAPS_VIDEO_ENCODER_SUPPORT1_DATA_0096 binary size
     * please note pResolutionDependentSupport type changes from D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2 to D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107
     */

    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0 CodecGopSequence;
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2 RateControl;
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080 IntraRefresh;
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionFrameEncoding;
    UINT ResolutionsListCount;
    const D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080* pResolutionList;
    UINT MaxReferenceFramesInDPB;
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAGS_0082_0 ValidationFlags;
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAGS_0083_0 SupportFlags;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 SuggestedProfile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 SuggestedLevel;
    D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107* pResolutionDependentSupport;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2 SubregionFrameEncodingData;
    UINT MaxQualityVsSpeed;

    /* Below are new arguments for D3D12DDICAPS_VIDEO_ENCODER_SUPPORT2_DATA_0107 */
    D3D12DDI_VIDEO_ENCODER_QPMAP_CONFIGURATION_0107 QPMap;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_CONFIGURATION_0107 DirtyRegions;
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_CONFIGURATION_0107 MotionSearch;
    D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_CONFIGURATION_0110 FrameAnalysis;

} D3D12DDICAPS_VIDEO_ENCODER_SUPPORT2_DATA_0107;
```

<a id="d3d12ddicaps-video-encoder-support-data-0083-0"></a>

### D3D12DDICAPS_VIDEO_ENCODER_SUPPORT_DATA_0083_0

```cpp
typedef struct D3D12DDICAPS_VIDEO_ENCODER_SUPPORT_DATA_0083_0
{
    // input
    UINT NodeIndex;
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0 CodecGopSequence;
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2 RateControl;
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080 IntraRefresh;
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionFrameEncoding;
    UINT ResolutionsListCount;
    const D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080* pResolutionList;
    UINT MaxReferenceFramesInDPB;

    // single value output
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAGS_0082_0 ValidationFlags;
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAGS_0083_0 SupportFlags;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 SuggestedProfile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 SuggestedLevel;

    // resolution dependent output
    D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2* pResolutionDependentSupport;
} D3D12DDICAPS_VIDEO_ENCODER_SUPPORT_DATA_0083_0;
```

<a id="d3d12ddi-build-version-0111"></a>

### D3D12DDI_BUILD_VERSION_0111

```cpp
#define D3D12DDI_BUILD_VERSION_0111 111
```

<a id="d3d12ddi-feature-data-video-encoder-frame-subregion-layout-config-0095"></a>

### D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_0095

```cpp
typedef struct D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_0095
{
    UINT NodeIndex; // input
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;                           // input
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;                  // input
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 Level;                   // input
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionMode; // input
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 FrameResolution; // input
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095 CodecSupport; // input/output
    BOOL IsSupported; // output
} D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_0095;
```

<a id="d3d12ddi-feature-data-video-encoder-resolution-support-dirty-regions-0107"></a>

### D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_DIRTY_REGIONS_0107

```cpp
typedef struct D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_DIRTY_REGIONS_0107
{
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAGS_0104 DirtyRegionsSupportFlags;
    UINT MapSourcePreferenceRanking;
} D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_DIRTY_REGIONS_0107;
```

<a id="d3d12ddi-feature-data-video-encoder-resolution-support-frame-analysis-0110"></a>

### D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_FRAME_ANALYSIS_0110

```cpp
typedef struct D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_FRAME_ANALYSIS_0110
{
    D3D12DDICAPS_VIDEO_ENCODER_RATE_CONTROL_FRAME_ANALYSIS_SUPPORT_FLAGS_0110_0 SupportFlags;
} D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_FRAME_ANALYSIS_0110;
```

<a id="d3d12ddi-feature-data-video-encoder-resolution-support-motion-search-0107"></a>

### D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_MOTION_SEARCH_0107

```cpp
typedef struct D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_MOTION_SEARCH_0107
{
    UINT MaxMotionHints;
    UINT MinDeviation;
    UINT MaxDeviation;
    UINT MapSourcePreferenceRanking;
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAGS_0105 MotionUnitPrecisionSupportFlags;
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAGS_0107 SupportFlags;
} D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_MOTION_SEARCH_0107;
```

<a id="d3d12ddi-feature-data-video-encoder-resolution-support-qpmap-0107"></a>

### D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_QPMAP_0107

```cpp
typedef struct D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_QPMAP_0107
{
    // Reuse existing D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107.QPMapRegionPixelsSize for BlockSize
    UINT MapSourcePreferenceRanking;
} D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_QPMAP_0107;
```

<a id="d3d12ddi-hvideoencoderheap-0080"></a>

### D3D12DDI_HVIDEOENCODERHEAP_0080

```cpp
D3D10DDI_H( D3D12DDI_HVIDEOENCODERHEAP_0080 )
```

<a id="d3d12ddi-hvideoencoder-0080"></a>

### D3D12DDI_HVIDEOENCODER_0080

```cpp
D3D10DDI_H( D3D12DDI_HVIDEOENCODER_0080 )
```

<a id="d3d12ddi-interface-version-r8"></a>

### D3D12DDI_INTERFACE_VERSION_R8

```cpp
#define D3D12DDI_INTERFACE_VERSION_R8 ((D3D12DDI_MAJOR_VERSION << 16) | D3D12DDI_MINOR_VERSION_R8)
```

<a id="d3d12ddi-major-version"></a>

### D3D12DDI_MAJOR_VERSION

```cpp
#define D3D12DDI_MAJOR_VERSION 12
```

<a id="d3d12ddi-minor-version-r8"></a>

### D3D12DDI_MINOR_VERSION_R8

```cpp
#define D3D12DDI_MINOR_VERSION_R8 80
```

<a id="d3d12ddi-supported-0111"></a>

### D3D12DDI_SUPPORTED_0111

```cpp
#define D3D12DDI_SUPPORTED_0111 ((((UINT64)D3D12DDI_INTERFACE_VERSION_R8) << 32) | (((UINT64)D3D12DDI_BUILD_VERSION_0111) << 16))
```

<a id="d3d12ddi-video-encoder-av1-cdef-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_CDEF_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_CDEF_CONFIG_0095 {
    UINT64 CdefBits;
    UINT64 CdefDampingMinus3;
    UINT64 CdefYPriStrength[8];
    UINT64 CdefUVPriStrength[8];
    UINT64 CdefYSecStrength[8];
    UINT64 CdefUVSecStrength[8];
} D3D12DDI_VIDEO_ENCODER_AV1_CDEF_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-av1-codec-configuration-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAGS FeatureFlags;
    UINT OrderHintBitsMinus1;
} D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_0095;
```

<a id="d3d12ddi-video-encoder-av1-codec-configuration-support-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_SUPPORT_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_SUPPORT_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAGS SupportedFeatureFlags;
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAGS RequiredFeatureFlags;
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAGS SupportedInterpolationFilters;
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAGS SupportedRestorationParams[3][3];
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAGS SupportedSegmentationModes;
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAGS SupportedTxModes[4]; // Different for D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095 types
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095 SegmentationBlockSize;
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAGS PostEncodeValuesFlags;
    UINT MaxTemporalLayers;
    UINT MaxSpatialLayers;
} D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_SUPPORT_0095;
```

<a id="d3d12ddi-video-encoder-av1-comp-prediction-type-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095_SINGLE_REFERENCE = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095_COMPOUND_REFERENCE = 1,
} D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095;
```

<a id="d3d12ddi-video-encoder-av1-feature-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAGS
{
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_128x128_SUPERBLOCK = 0x1,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_FILTER_INTRA = 0x2,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_INTRA_EDGE_FILTER = 0x4,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_INTERINTRA_COMPOUND = 0x8,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_MASKED_COMPOUND = 0x10,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_WARPED_MOTION = 0x20,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_DUAL_FILTER = 0x40,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_JNT_COMP = 0x80,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_FORCED_INTEGER_MOTION_VECTORS = 0x100,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_SUPER_RESOLUTION = 0x200,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_LOOP_RESTORATION_FILTER = 0x400,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_PALETTE_ENCODING = 0x800,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_CDEF_FILTERING = 0x1000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_INTRA_BLOCK_COPY = 0x2000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_FRAME_REFERENCE_MOTION_VECTORS = 0x4000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_ORDER_HINT_TOOLS = 0x8000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_AUTO_SEGMENTATION = 0x10000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_CUSTOM_SEGMENTATION = 0x20000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_LOOP_FILTER_DELTAS = 0x40000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_QUANTIZATION_DELTAS = 0x80000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_QUANTIZATION_MATRIX = 0x100000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_REDUCED_TX_SET = 0x200000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_MOTION_MODE_SWITCHABLE = 0x400000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_ALLOW_HIGH_PRECISION_MV = 0x800000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_SKIP_MODE_PRESENT = 0x1000000,
    D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAG_DELTA_LF_PARAMS = 0x2000000,
} D3D12DDI_VIDEO_ENCODER_AV1_FEATURE_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-frame-subregion-layout-config-support-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095
{
    BOOL Use128SuperBlocks;
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_TILES_0095 TilesConfiguration;
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAGS ValidationFlags;
    UINT MinTileRows;
    UINT MaxTileRows;
    UINT MinTileCols;
    UINT MaxTileCols;
    UINT MinTileWidth;
    UINT MaxTileWidth;
    UINT MinTileArea;
    UINT MaxTileArea;
    UINT TileSizeBytesMinus1;
} D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095;
```

<a id="d3d12ddi-video-encoder-av1-frame-subregion-layout-config-validation-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAGS
{
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_NOT_SPECIFIED = 0x1,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_CODEC_CONSTRAINT = 0x2,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_HARDWARE_CONSTRAINT = 0x4,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_ROWS_COUNT = 0x8,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_COLS_COUNT = 0x10,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_WIDTH = 0x20,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_AREA = 0x40,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAG_TOTAL_TILES = 0x80,

} D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_VALIDATION_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-frame-type-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_KEY_FRAME = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_INTER_FRAME = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_INTRA_ONLY_FRAME = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_SWITCH_FRAME = 3,
} D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095;
```

<a id="d3d12ddi-video-encoder-av1-frame-type-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAG_KEY_FRAME = (1 << D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_KEY_FRAME),
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAG_INTER_FRAME = (1 << D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_INTER_FRAME),
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAG_INTRA_ONLY_FRAME = (1 << D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_INTRA_ONLY_FRAME),
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAG_SWITCH_FRAME = (1 << D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_SWITCH_FRAME),
} D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-interpolation-filters-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_EIGHTTAP = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_EIGHTTAP_SMOOTH = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_EIGHTTAP_SHARP = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_BILINEAR = 3,
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_SWITCHABLE = 4,
} D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095;
```

<a id="d3d12ddi-video-encoder-av1-interpolation-filters-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAG_EIGHTTAP = (1 << D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_EIGHTTAP),
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAG_EIGHTTAP_SMOOTH = (1 << D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_EIGHTTAP_SMOOTH),
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAG_EIGHTTAP_SHARP = (1 << D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_EIGHTTAP_SHARP),
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAG_BILINEAR = (1 << D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_BILINEAR),
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAG_SWITCHABLE = (1 << D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_SWITCHABLE),
} D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-levels-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095
{   // These values are defined according to the AV1 codec standard spec
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_2_0 = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_2_1 = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_2_2 = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_2_3 = 3,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_3_0 = 4,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_3_1 = 5,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_3_2 = 6,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_3_3 = 7,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_4_0 = 8,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_4_1 = 9,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_4_2 = 10,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_4_3 = 11,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_5_0 = 12,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_5_1 = 13,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_5_2 = 14,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_5_3 = 15,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_6_0 = 16,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_6_1 = 17,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_6_2 = 18,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_6_3 = 19,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_7_0 = 20,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_7_1 = 21,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_7_2 = 22,
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095_7_3 = 23,
} D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095;
```

<a id="d3d12ddi-video-encoder-av1-level-tier-constraints-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_LEVEL_TIER_CONSTRAINTS_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_LEVEL_TIER_CONSTRAINTS_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_LEVELS_0095 Level;
    D3D12DDI_VIDEO_ENCODER_AV1_TIER_0095 Tier;
} D3D12DDI_VIDEO_ENCODER_AV1_LEVEL_TIER_CONSTRAINTS_0095;
```

<a id="d3d12ddi-video-encoder-av1-max-tile-cols-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_MAX_TILE_COLS_0095

```cpp
#define D3D12DDI_VIDEO_ENCODER_AV1_MAX_TILE_COLS_0095 64 // As defined by AV1 spec
```

<a id="d3d12ddi-video-encoder-av1-max-tile-rows-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_MAX_TILE_ROWS_0095

```cpp
#define D3D12DDI_VIDEO_ENCODER_AV1_MAX_TILE_ROWS_0095 64 // As defined by AV1 spec
```

<a id="d3d12ddi-video-encoder-av1-picture-control-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAGS
{
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ENABLE_ERROR_RESILIENT_MODE = 0x1,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_DISABLE_CDF_UPDATE = 0x2,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ENABLE_PALETTE_ENCODING = 0x4,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ENABLE_SKIP_MODE = 0x8,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_FRAME_REFERENCE_MOTION_VECTORS = 0x10,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_FORCE_INTEGER_MOTION_VECTORS = 0x20,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ALLOW_INTRA_BLOCK_COPY = 0x40,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_USE_SUPER_RESOLUTION = 0x80,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_DISABLE_FRAME_END_UPDATE_CDF = 0x100,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ENABLE_FRAME_SEGMENTATION_AUTO = 0x200,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ENABLE_FRAME_SEGMENTATION_CUSTOM = 0x400,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ENABLE_WARPED_MOTION = 0x800,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_REDUCED_TX_SET = 0x1000,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_MOTION_MODE_SWITCHABLE = 0x2000,
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAG_ALLOW_HIGH_PRECISION_MV = 0x4000,

} D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-picture-control-codec-data-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_CODEC_DATA_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_CODEC_DATA_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_0095_FLAGS Flags;
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095 FrameType;
    D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095 CompoundPredictionType;
    D3D12DDI_VIDEO_ENCODER_AV1_INTERPOLATION_FILTERS_0095 InterpolationFilter;
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_CONFIG_0095 FrameRestorationConfig;
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095 TxMode;
    UINT SuperResDenominator;
    UINT OrderHint;
    UINT PictureIndex;
    UINT TemporalLayerIndexPlus1;
    UINT SpatialLayerIndexPlus1;
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_DESCRIPTOR_0095 ReferenceFramesReconPictureDescriptors[8];
    UINT ReferenceIndices[7];
    UINT PrimaryRefFrame;
    UINT RefreshFrameFlags;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_CONFIG_0095 LoopFilter;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_DELTA_CONFIG_0095 LoopFilterDelta;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_CONFIG_0095 Quantization;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_DELTA_CONFIG_0095 QuantizationDelta;
    D3D12DDI_VIDEO_ENCODER_AV1_CDEF_CONFIG_0095 CDEF;
    UINT QPMapValuesCount;
    INT16* pRateControlQPMap;
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_CONFIG_0095 CustomSegmentation;
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MAP_0095 CustomSegmentsMap;
} D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_CODEC_DATA_0095;
```

<a id="d3d12ddi-video-encoder-av1-picture-control-subregions-layout-data-tiles-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_TILES_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_TILES_0095
{
    UINT64 RowCount;
    UINT64 ColCount;
    UINT64 RowHeights[D3D12DDI_VIDEO_ENCODER_AV1_MAX_TILE_ROWS_0095];
    UINT64 ColWidths[D3D12DDI_VIDEO_ENCODER_AV1_MAX_TILE_COLS_0095];
    UINT64 ContextUpdateTileId;
} D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_TILES_0095;
```

<a id="d3d12ddi-video-encoder-av1-post-encode-values-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095
{
    UINT64 CompoundPredictionType; // D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_CONFIG_0095 LoopFilter;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_DELTA_CONFIG_0095 LoopFilterDelta;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_CONFIG_0095 Quantization;
    D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_DELTA_CONFIG_0095 QuantizationDelta;
    D3D12DDI_VIDEO_ENCODER_AV1_CDEF_CONFIG_0095 CDEF;
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_CONFIG_0095 SegmentationConfig;
    UINT64 PrimaryRefFrame; // Aligned to 64 for use of post encode metadata with predication
    UINT64 ReferenceIndices[7]; // Aligned to 64 for use of post encode metadata with predication
} D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095;
```

<a id="d3d12ddi-video-encoder-av1-post-encode-values-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_NONE = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_QUANTIZATION = 0x1,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_QUANTIZATION_DELTA = 0x2,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_LOOP_FILTER = 0x4,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_LOOP_FILTER_DELTA = 0x8,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_CDEF_DATA = 0x10,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_CONTEXT_UPDATE_TILE_ID = 0x20,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_COMPOUND_PREDICTION_MODE = 0x40,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_PRIMARY_REF_FRAME = 0x80,
    D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAG_REFERENCE_INDICES = 0x100,
} D3D12DDI_VIDEO_ENCODER_AV1_POST_ENCODE_VALUES_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-profile-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095_MAIN = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095_HIGH = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095_PROFESSIONAL = 2,
} D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095;
```

<a id="d3d12ddi-video-encoder-av1-reference-picture-descriptor-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_DESCRIPTOR_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_DESCRIPTOR_0095
{
    UINT ReconstructedPictureResourceIndex;
    UINT TemporalLayerIndexPlus1;
    UINT SpatialLayerIndexPlus1;
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095 FrameType;
    // Only used if supported in D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAGS, ignored by driver otherwise
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_WARPED_MOTION_INFO_0095 WarpedMotionInfo;
    UINT OrderHint;
    UINT PictureIndex;
} D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_DESCRIPTOR_0095;
```

<a id="d3d12ddi-video-encoder-av1-reference-picture-warped-motion-info-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_WARPED_MOTION_INFO_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_WARPED_MOTION_INFO_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095 TransformationType;
    INT TransformationMatrix[8];
    BOOL InvalidAffineSet; // Valid or invalid on affine set
} D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_PICTURE_WARPED_MOTION_INFO_0095;
```

<a id="d3d12ddi-video-encoder-av1-reference-warped-motion-transformation-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_IDENTITY = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_TRANSLATION = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_ROTZOOM = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_AFFINE = 3,
} D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095;
```

<a id="d3d12ddi-video-encoder-av1-reference-warped-motion-transformation-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAG_NONE = 0, // None is supported, reference warp motion arguments are ignored and is_global syntax is false for all references
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAG_IDENTITY = (1 << D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_IDENTITY),
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAG_TRANSLATION = (1 << D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_TRANSLATION),
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAG_ROTZOOM = (1 << D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_ROTZOOM),
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAG_AFFINE = (1 << D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_AFFINE),
} D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-restoration-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_CONFIG_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095 FrameRestorationType[3];
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095 LoopRestorationPixelSize[3];
} D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-av1-restoration-support-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAG_NOT_SUPPORTED = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAG_32x32 = 0x1,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAG_64x64 = 0x2,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAG_128x128 = 0x4,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAG_256x256 = 0x8,
} D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_SUPPORT_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-restoration-tilesize-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095_DISABLED = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095_32x32 = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095_64x64 = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095_128x128 = 3,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095_256x256 = 4,
} D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TILESIZE_0095;
```

<a id="d3d12ddi-video-encoder-av1-restoration-type-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095_DISABLED = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095_SWITCHABLE = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095_WIENER = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095_SGRPROJ = 3,
} D3D12DDI_VIDEO_ENCODER_AV1_RESTORATION_TYPE_0095;
```

<a id="d3d12ddi-video-encoder-av1-segmentation-block-size-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095_4x4 = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095_8x8 = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095_16x16 = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095_32x32 = 3,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095_64x64 = 4,
} D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_BLOCK_SIZE_0095;
```

<a id="d3d12ddi-video-encoder-av1-segmentation-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_CONFIG_0095 {
    UINT64 UpdateMap;
    UINT64 TemporalUpdate;
    UINT64 UpdateData;
    UINT64 NumSegments;
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENT_DATA_0095 SegmentsData[8];
} D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-av1-segmentation-map-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MAP_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MAP_0095 {
    UINT SegmentsMapByteSize;
    UINT8* pSegmentsMap;
} D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MAP_0095;
```

<a id="d3d12ddi-video-encoder-av1-segmentation-mode-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_DISABLED = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_Q = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_Y_V = 2,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_Y_H = 3,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_U = 4,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_V = 5,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_REF_FRAME = 6,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_SKIP = 7,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_GLOBALMV = 8,
} D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095;
```

<a id="d3d12ddi-video-encoder-av1-segmentation-mode-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_NONE = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_DISABLED = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_DISABLED),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_Q = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_Q),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_LF_Y_V = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_Y_V),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_LF_Y_H = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_Y_H),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_LF_U = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_U),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_LF_V = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_LF_V),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_REF_FRAME = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_REF_FRAME),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_SKIP = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_SKIP),
    D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAG_ALT_GLOBALMV = (1 << D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_ALT_GLOBALMV),
} D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-av1-segment-data-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEGMENT_DATA_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_SEGMENT_DATA_0095 {
    UINT64 EnabledFeatures; // D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095_FLAGS
    // For the enabled features in EnabledFeatures, the array below
    // is indexed by D3D12DDI_VIDEO_ENCODER_AV1_SEGMENTATION_MODE_0095 - 1
    INT64 FeatureValue[8];
} D3D12DDI_VIDEO_ENCODER_AV1_SEGMENT_DATA_0095;
```

<a id="d3d12ddi-video-encoder-av1-sequence-structure-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_SEQUENCE_STRUCTURE_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_AV1_SEQUENCE_STRUCTURE_0095
{
    UINT IntraDistance;
    UINT InterFramePeriod;
} D3D12DDI_VIDEO_ENCODER_AV1_SEQUENCE_STRUCTURE_0095;
```

<a id="d3d12ddi-video-encoder-av1-tier-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_TIER_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_TIER_0095
{
    D3D12DDI_VIDEO_ENCODER_AV1_TIER_0095_MAIN = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_TIER_0095_HIGH = 1,
} D3D12DDI_VIDEO_ENCODER_AV1_TIER_0095;
```

<a id="d3d12ddi-video-encoder-av1-tx-mode-0095"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_ONLY4x4 = 0,
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_LARGEST = 1,
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_SELECT = 2,
} D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095;
```

<a id="d3d12ddi-video-encoder-av1-tx-mode-0095-flags"></a>

### D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAGS

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAGS {
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAG_ONLY4x4 = (1 << D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_ONLY4x4),
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAG_LARGEST = (1 << D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_LARGEST),
    D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAG_SELECT = (1 << D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_SELECT),
} D3D12DDI_VIDEO_ENCODER_AV1_TX_MODE_0095_FLAGS;
```

<a id="d3d12ddi-video-encoder-codec-0080"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_0080
{
    D3D12DDI_VIDEO_ENCODER_CODEC_0080_H264  = 0,
    D3D12DDI_VIDEO_ENCODER_CODEC_0080_HEVC  = 1,
    D3D12DDI_VIDEO_ENCODER_CODEC_0095_AV1 = 2,
} D3D12DDI_VIDEO_ENCODER_CODEC_0080;
```

<a id="d3d12ddi-video-encoder-codec-av1-loop-filter-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_CONFIG_0095 {
    UINT64 LoopFilterLevel[2];
    UINT64 LoopFilterLevelU;
    UINT64 LoopFilterLevelV;
    UINT64 LoopFilterSharpnessLevel;
    UINT64 LoopFilterDeltaEnabled;
    // loop_filter_delta_update = UpdateRefDelta || UpdateModeDelta
    UINT64 UpdateRefDelta;
    INT64 RefDeltas[8];
    UINT64 UpdateModeDelta;
    INT64 ModeDeltas[2];
} D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-codec-av1-loop-filter-delta-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_DELTA_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_DELTA_CONFIG_0095 {
    UINT64 DeltaLFPresent;
    UINT64 DeltaLFMulti;
    UINT64 DeltaLFRes;
} D3D12DDI_VIDEO_ENCODER_CODEC_AV1_LOOP_FILTER_DELTA_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-codec-av1-picture-control-support-0095"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_AV1_PICTURE_CONTROL_SUPPORT_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_AV1_PICTURE_CONTROL_SUPPORT_0095 {
    D3D12DDI_VIDEO_ENCODER_AV1_COMP_PREDICTION_TYPE_0095 PredictionMode;
    UINT MaxUniqueReferencesPerFrame;
    D3D12DDI_VIDEO_ENCODER_AV1_FRAME_TYPE_0095_FLAGS SupportedFrameTypes;
    D3D12DDI_VIDEO_ENCODER_AV1_REFERENCE_WARPED_MOTION_TRANSFORMATION_0095_FLAGS SupportedReferenceWarpedMotionFlags;
} D3D12DDI_VIDEO_ENCODER_CODEC_AV1_PICTURE_CONTROL_SUPPORT_0095;
```

<a id="d3d12ddi-video-encoder-codec-av1-quantization-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_CONFIG_0095 {
    UINT64 BaseQIndex;
    INT64 YDCDeltaQ;
    INT64 UDCDeltaQ;
    INT64 UACDeltaQ;
    INT64 VDCDeltaQ;
    INT64 VACDeltaQ;
    UINT64 UsingQMatrix;
    UINT64 QMY;
    UINT64 QMU;
    UINT64 QMV;
} D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-codec-av1-quantization-delta-config-0095"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_DELTA_CONFIG_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_DELTA_CONFIG_0095 {
    UINT64 DeltaQPresent;
    UINT64 DeltaQRes;
} D3D12DDI_VIDEO_ENCODER_CODEC_AV1_QUANTIZATION_DELTA_CONFIG_0095;
```

<a id="d3d12ddi-video-encoder-codec-configuration-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_0082_0* pH264Config;
        D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0082_0* pHEVCConfig;
        D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_0095* pAV1Config;
    };
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-h264-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_0082_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAGS_0080 ConfigurationFlags;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080 DirectModeConfig;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODES_0082_0 DisableDeblockingFilterConfig;
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-h264-direct-modes-0080"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080_DISABLED = 0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080_TEMPORAL = 1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080_SPATIAL = 2,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_DIRECT_MODES_0080;
```

<a id="d3d12ddi-video-encoder-codec-configuration-h264-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAG_NONE_0080 = 0x0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAG_USE_CONSTRAINED_INTRAPREDICTION_0080 = 0x1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAG_USE_ADAPTIVE_8x8_TRANSFORM_0080 = 0x2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAG_ENABLE_CABAC_ENCODING_0080 = 0x4,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAG_ALLOW_REQUEST_INTRA_CONSTRAINED_SLICES_0080 = 0x8,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-codec-configuration-h264-slices-deblocking-modes-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODES_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODES_0082_0 {
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_0_ALL_LUMA_CHROMA_SLICE_BLOCK_EDGES_ALWAYS_FILTERED = 0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_1_DISABLE_ALL_SLICE_BLOCK_EDGES = 1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_2_DISABLE_SLICE_BOUNDARIES_BLOCKS = 2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_3_USE_TWO_STAGE_DEBLOCKING = 3,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_4_DISABLE_CHROMA_BLOCK_EDGES = 4,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_5_DISABLE_CHROMA_BLOCK_EDGES_AND_LUMA_BOUNDARIES = 5,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_6_DISABLE_CHROMA_BLOCK_EDGES_AND_USE_LUMA_TWO_STAGE_DEBLOCKING = 6,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODES_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-h264-slices-deblocking-mode-flags-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_FLAGS_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_FLAGS_0082_0 {
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_0_ALL_LUMA_CHROMA_SLICE_BLOCK_EDGES_ALWAYS_FILTERED = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_0_ALL_LUMA_CHROMA_SLICE_BLOCK_EDGES_ALWAYS_FILTERED),
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_1_DISABLE_ALL_SLICE_BLOCK_EDGES = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_1_DISABLE_ALL_SLICE_BLOCK_EDGES),
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_2_DISABLE_SLICE_BOUNDARIES_BLOCKS = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_2_DISABLE_SLICE_BOUNDARIES_BLOCKS),
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_3_USE_TWO_STAGE_DEBLOCKING = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_3_USE_TWO_STAGE_DEBLOCKING),
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_4_DISABLE_CHROMA_BLOCK_EDGES = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_4_DISABLE_CHROMA_BLOCK_EDGES),
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_5_DISABLE_CHROMA_BLOCK_EDGES_AND_LUMA_BOUNDARIES = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_5_DISABLE_CHROMA_BLOCK_EDGES_AND_LUMA_BOUNDARIES),
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_FLAG_6_DISABLE_CHROMA_BLOCK_EDGES_AND_USE_LUMA_TWO_STAGE_DEBLOCKING = (1 << D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_0082_0_6_DISABLE_CHROMA_BLOCK_EDGES_AND_USE_LUMA_TWO_STAGE_DEBLOCKING),
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_FLAGS_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-hevc-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0082_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAGS_0082_0 ConfigurationFlags;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080 MinLumaCodingUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080 MaxLumaCodingUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 MinLumaTransformUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 MaxLumaTransformUnitSize;
    UCHAR max_transform_hierarchy_depth_inter;
    UCHAR max_transform_hierarchy_depth_intra;
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-hevc-cusize-0080"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0080_CUSIZE_8x8 = 0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0080_CUSIZE_16x16 = 1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0080_CUSIZE_32x32 = 2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_0080_CUSIZE_64x64 = 3,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080;
```

<a id="d3d12ddi-video-encoder-codec-configuration-hevc-flags-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAGS_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAGS_0082_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_DISABLE_LOOP_FILTER_ACROSS_SLICES = 0x1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_ALLOW_REQUEST_INTRA_CONSTRAINED_SLICES = 0x2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_ENABLE_SAO_FILTER = 0x4,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_ENABLE_LONG_TERM_REFERENCES = 0x8,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_USE_ASYMETRIC_MOTION_PARTITION = 0x10,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_ENABLE_TRANSFORM_SKIPPING = 0x20,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0082_0_USE_CONSTRAINED_INTRAPREDICTION = 0x40,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_TRANSFORM_SKIP_ROTATION = 0x80,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_TRANSFORM_SKIP_CONTEXT = 0x100,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_IMPLICIT_RDPCM = 0x200,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_EXPLICIT_RDPCM = 0x400,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_EXTENDED_PRECISION_PROCESSING = 0x800,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_INTRA_SMOOTHING_DISABLED = 0x1000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_HIGH_PRECISION_OFFSETS = 0x2000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_PERSISTENT_RICE_ADAPTATION = 0x4000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_CABAC_BYPASS_ALIGNMENT = 0x8000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAG_0100_0_SEPARATE_COLOUR_PLANE = 0x10000,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_FLAGS_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-hevc-tusize-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 {
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0_4x4 = 0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0_8x8 = 1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0_16x16 = 2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0_32x32 = 3,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_0083_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_0083_0
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_0082_0* pH264Support;
        D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_0083_0* pHEVCSupport;
        D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC1_0100_0* pHEVCSupport1;
        D3D12DDI_VIDEO_ENCODER_AV1_CODEC_CONFIGURATION_SUPPORT_0095* pAV1Support;
    };
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_0083_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-h264-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_0082_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAGS_0080 SupportFlags;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_H264_SLICES_DEBLOCKING_MODE_FLAGS_0082_0 DisableDeblockingFilterSupportedModes;
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_0082_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-h264-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_CABAC_ENCODING_SUPPORT = 0x1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_INTRA_SLICE_CONSTRAINED_ENCODING_SUPPORT = 0x2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_BFRAME_LTR_COMBINED_SUPPORT = 0x4,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_ADAPTIVE_8x8_TRANSFORM_ENCODING_SUPPORT = 0x8,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_DIRECT_SPATIAL_ENCODING_SUPPORT = 0x10,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_DIRECT_TEMPORAL_ENCODING_SUPPORT = 0x20,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0080_CONSTRAINED_INTRAPREDICTION_SUPPORT = 0x40,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAG_0099_0_NUM_REF_IDX_ACTIVE_OVERRIDE_FLAG_SLICE_SUPPORT = 0x80,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_H264_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-hevc1-0100-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC1_0100_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC1_0100_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS_0083_0 SupportFlags;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080 MinLumaCodingUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080 MaxLumaCodingUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 MinLumaTransformUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 MaxLumaTransformUnitSize;
    UCHAR max_transform_hierarchy_depth_inter;
    UCHAR max_transform_hierarchy_depth_intra;
    UINT allowed_diff_cu_chroma_qp_offset_depth_values;
    UINT allowed_log2_sao_offset_scale_luma_values;
    UINT allowed_log2_sao_offset_scale_chroma_values;
    UINT allowed_log2_max_transform_skip_block_size_minus2_values;
    UINT allowed_chroma_qp_offset_list_len_minus1_values;
    UINT allowed_cb_qp_offset_list_values[6];
    UINT allowed_cr_qp_offset_list_values[6];
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS1_0100_0 SupportFlags1;
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC1_0100_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-hevc-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_0083_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_0083_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS_0083_0 SupportFlags;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080 MinLumaCodingUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_CUSIZE_0080 MaxLumaCodingUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 MinLumaTransformUnitSize;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_HEVC_TUSIZE_0082_0 MaxLumaTransformUnitSize;
    UCHAR max_transform_hierarchy_depth_inter;
    UCHAR max_transform_hierarchy_depth_intra;
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_0083_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-hevc-flags1-0100-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS1_0100_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS1_0100_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG1_0100_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG1_0100_0_SEPARATE_COLOUR_PLANE_SUPPORT = 0x1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG1_0100_0_SEPARATE_COLOUR_PLANE_REQUIRED = 0x2,
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS1_0100_0;
```

<a id="d3d12ddi-video-encoder-codec-configuration-support-hevc-flags-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS_0083_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS_0083_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_BFRAME_LTR_COMBINED_SUPPORT = 0x1,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_INTRA_SLICE_CONSTRAINED_ENCODING_SUPPORT = 0x2,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_CONSTRAINED_INTRAPREDICTION_SUPPORT = 0x4,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_SAO_FILTER_SUPPORT = 0x8,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_ASYMETRIC_MOTION_PARTITION_SUPPORT = 0x10,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_ASYMETRIC_MOTION_PARTITION_REQUIRED = 0x20,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_TRANSFORM_SKIP_SUPPORT = 0x40,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_DISABLING_LOOP_FILTER_ACROSS_SLICES_SUPPORT = 0x80,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0083_0_P_FRAMES_IMPLEMENTED_AS_LOW_DELAY_B_FRAMES = 0x100,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0099_0_NUM_REF_IDX_ACTIVE_OVERRIDE_FLAG_SLICE_SUPPORT = 0x200,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_TRANSFORM_SKIP_ROTATION_ENABLED_SUPPORT = 0x400,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_TRANSFORM_SKIP_ROTATION_ENABLED_REQUIRED = 0x800,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_TRANSFORM_SKIP_CONTEXT_ENABLED_SUPPORT = 0x1000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_TRANSFORM_SKIP_CONTEXT_ENABLED_REQUIRED = 0x2000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_IMPLICIT_RDPCM_ENABLED_SUPPORT = 0x4000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_IMPLICIT_RDPCM_ENABLED_REQUIRED = 0x8000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_EXPLICIT_RDPCM_ENABLED_SUPPORT = 0x10000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_EXPLICIT_RDPCM_ENABLED_REQUIRED = 0x20000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_EXTENDED_PRECISION_PROCESSING_SUPPORT = 0x40000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_EXTENDED_PRECISION_PROCESSING_REQUIRED = 0x80000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_INTRA_SMOOTHING_DISABLED_SUPPORT = 0x100000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_INTRA_SMOOTHING_DISABLED_REQUIRED = 0x200000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_HIGH_PRECISION_OFFSETS_ENABLED_SUPPORT = 0x400000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_HIGH_PRECISION_OFFSETS_ENABLED_REQUIRED = 0x800000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_PERSISTENT_RICE_ADAPTATION_ENABLED_SUPPORT = 0x1000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_PERSISTENT_RICE_ADAPTATION_ENABLED_REQUIRED = 0x2000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_CABAC_BYPASS_ALIGNMENT_ENABLED_SUPPORT = 0x4000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_CABAC_BYPASS_ALIGNMENT_ENABLED_REQUIRED = 0x8000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_CROSS_COMPONENT_PREDICTION_ENABLED_FLAG_SUPPORT = 0x10000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_CROSS_COMPONENT_PREDICTION_ENABLED_FLAG_REQUIRED = 0x20000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_CHROMA_QP_OFFSET_LIST_ENABLED_FLAG_SUPPORT = 0x40000000,
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAG_0100_0_CHROMA_QP_OFFSET_LIST_ENABLED_FLAG_REQUIRED = 0x80000000, // 2^31 - 32 bit integer enum type
} D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_SUPPORT_HEVC_FLAGS_0083_0;
```

<a id="d3d12ddi-video-encoder-codec-picture-control-support-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_0080_2
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_H264_0080_2* pH264Support;
        D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_HEVC_0080_2* pHEVCSupport;
        D3D12DDI_VIDEO_ENCODER_CODEC_AV1_PICTURE_CONTROL_SUPPORT_0095* pAV1Support;
    };
} D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_0080_2;
```

<a id="d3d12ddi-video-encoder-codec-picture-control-support-h264-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_H264_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_H264_0080_2
{
    UINT MaxL0ReferencesForP;
    UINT MaxL0ReferencesForB;
    UINT MaxL1ReferencesForB;
    UINT MaxLongTermReferences;
    UINT MaxDPBCapacity;
} D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_H264_0080_2;
```

<a id="d3d12ddi-video-encoder-codec-picture-control-support-hevc-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_HEVC_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_HEVC_0080_2
{
    UINT MaxL0ReferencesForP;
    UINT MaxL0ReferencesForB;
    UINT MaxL1ReferencesForB;
    UINT MaxLongTermReferences;
    UINT MaxDPBCapacity;
} D3D12DDI_VIDEO_ENCODER_CODEC_PICTURE_CONTROL_SUPPORT_HEVC_0080_2;
```

<a id="d3d12ddi-video-encoder-compressed-bitstream-0080"></a>

### D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0080
{
    D3D12DDI_HRESOURCE hDrvBuffer;
    UINT64 Offset;
} D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0080;
```

<a id="d3d12ddi-video-encoder-compressed-bitstream-0102"></a>

### D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0102

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0102
{
    D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_NOTIFICATION_MODE_0102 NotificationMode;
    union
    {
        D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0080 FrameOutputBuffer;
        D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_0102 SubregionOutputBuffers;
    };
} D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0102;
```

<a id="d3d12ddi-video-encoder-compressed-bitstream-notification-mode-0102"></a>

### D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_NOTIFICATION_MODE_0102

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_NOTIFICATION_MODE_0102
{
    D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_NOTIFICATION_MODE_0102_FULL_FRAME = 0,
    D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_NOTIFICATION_MODE_0102_SUBREGIONS = 1,
} D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_NOTIFICATION_MODE_0102;
```

<a id="d3d12ddi-video-encoder-dirty-rect-info-0103"></a>

### D3D12DDI_VIDEO_ENCODER_DIRTY_RECT_INFO_0103

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_DIRTY_RECT_INFO_0103
{
    BOOL FullFrameIdentical;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103 MapValuesType;
    UINT NumDirtyRects;
    RECT* pDirtyRects;
    UINT SourceDPBFrameReference;
} D3D12DDI_VIDEO_ENCODER_DIRTY_RECT_INFO_0103;
```

<a id="d3d12ddi-video-encoder-dirty-regions-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_0103_0
{
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    union
    {
        // Use with: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103_GPU_TEXTURE
        D3D12DDI_HRESOURCE hDrvOpaqueLayoutBuffer;
        // Use with: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103_CPU_BUFFER
        D3D12DDI_VIDEO_ENCODER_DIRTY_RECT_INFO_0103* pCPUBuffer;
    };
} D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_0103_0;
```

<a id="d3d12ddi-video-encoder-dirty-regions-configuration-0107"></a>

### D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_CONFIGURATION_0107

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_CONFIGURATION_0107
{
    BOOL Enabled;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103 MapValuesType;
} D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_CONFIGURATION_0107;
```

<a id="d3d12ddi-video-encoder-dirty-regions-map-values-mode-0103"></a>

### D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103
{
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103_DIRTY = 0,
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103_SKIP = 1,
} D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103;
```

<a id="d3d12ddi-video-encoder-dirty-regions-support-flags-0104"></a>

### D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAGS_0104

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAGS_0104
{
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAG_0104_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAG_0104_REPEAT_FRAME = 0x1,
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAG_0104_DIRTY_REGIONS = 0x2,
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAG_0104_DIRTY_REGIONS_REQUIRE_FULL_ROW = 0x4,
} D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_SUPPORT_FLAGS_0104;
```

<a id="d3d12ddi-video-encoder-encodeframe-input-stream-arguments-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0082_0
{
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_DESC_0082_0 SequenceControlDesc;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0082_0 PictureControlDesc;
    D3D12DDI_HRESOURCE hDrvInputTexture2D;
    UINT InputFrameSubresource;
    UINT CurrentFrameBitstreamMetadataSize;
} D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0082_0;
```

<a id="d3d12ddi-video-encoder-encodeframe-input-stream-arguments-0102-0"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0102_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0102_0
{
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_DESC_0082_0 SequenceControlDesc;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0082_0 PictureControlDesc;
    D3D12DDI_HRESOURCE hDrvInputTexture2D;
    UINT InputFrameSubresource;
    UINT CurrentFrameBitstreamMetadataSize;
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0 OptionalMetadata;
} D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0102_0;
```

<a id="d3d12ddi-video-encoder-encodeframe-input-stream-arguments-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0103_0
{
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_DESC_0082_0 SequenceControlDesc;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0103_0 PictureControlDesc;
    D3D12DDI_HRESOURCE hDrvInputTexture2D;
    UINT InputFrameSubresource;
    UINT CurrentFrameBitstreamMetadataSize;
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0 OptionalMetadata;
} D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0103_0;
```

<a id="d3d12ddi-video-encoder-encodeframe-output-stream-arguments-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0080_2
{
    D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0080 CompressedBitstream;
    D3D12DDI_VIDEO_ENCODE_RECONSTRUCTED_PICTURE_0080 ReconstructedPicture;
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 HWLayoutEncoderMetadataDrvBuffer;
} D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0080_2;
```

<a id="d3d12ddi-video-encoder-encodeframe-output-stream-arguments-0102-0"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0102_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0102_0
{
    D3D12DDI_VIDEO_ENCODER_COMPRESSED_BITSTREAM_0102 CompressedBitstream;
    D3D12DDI_VIDEO_ENCODE_RECONSTRUCTED_PICTURE_0080 ReconstructedPicture;
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 HWLayoutEncoderMetadataDrvBuffer;
    D3D12DDI_VIDEO_ENCODE_RECONSTRUCTED_PICTURE_0080 FrameAnalysisReconstructedPicture;
} D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0102_0;
```

<a id="d3d12ddi-video-encoder-encode-error-flags-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAGS_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAGS_0082_0
{
    D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAG_0082_0_NO_ERROR = 0x0,
    D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAG_0082_0_CODEC_PICTURE_CONTROL_NOT_SUPPORTED = 0x1,
    D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAG_0082_0_SUBREGION_LAYOUT_CONFIGURATION_NOT_SUPPORTED = 0x2,
    D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAG_0082_0_INVALID_REFERENCE_PICTURES = 0x4,
    D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAG_0082_0_RECONFIGURATION_REQUEST_NOT_SUPPORTED = 0x8,
    D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAG_0082_0_INVALID_METADATA_BUFFER_SOURCE = 0x10,
} D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAGS_0082_0;
```

<a id="d3d12ddi-video-encoder-encode-operation-metadata-buffer-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2
{
    D3D12DDI_HRESOURCE hDrvBuffer;
    UINT64 Offset;
} D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2;
```

<a id="d3d12ddi-video-encoder-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_FLAG_0080_NONE = 0x0,
} D3D12DDI_VIDEO_ENCODER_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-frame-analysis-0110-0"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_0110_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_0110_0
{
    D3D12DDI_HRESOURCE hDrvDownscaledFrame;
    UINT64 Subresource;
    D3D12DDI_VIDEO_ENCODE_REFERENCE_FRAMES_0080 DownscaledReferences;
} D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_0110_0;
```

<a id="d3d12ddi-video-encoder-frame-analysis-configuration-0110"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_CONFIGURATION_0110

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_CONFIGURATION_0110
{
    BOOL Enabled;
    UINT Pow2DownscaleFactor;
} D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_CONFIGURATION_0110;
```

<a id="d3d12ddi-video-encoder-frame-input-motion-unit-precision-0105"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105
{
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105_FULL_PIXEL = 0,
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105_HALF_PIXEL = 1,
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105_QUARTER_PIXEL = 2,
} D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105;
```

<a id="d3d12ddi-video-encoder-frame-input-motion-unit-precision-support-flags-0105"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAGS_0105

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAGS_0105 {
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAG_0105_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAG_0105_FULL_PIXEL = (1 << D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105_FULL_PIXEL), // 0x1
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAG_0105_HALF_PIXEL = (1 << D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105_HALF_PIXEL), // 0x2
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAG_0105_QUARTER_PIXEL = (1 << D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105_QUARTER_PIXEL), // 0x4
} D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_SUPPORT_FLAGS_0105;
```

<a id="d3d12ddi-video-encoder-frame-motion-search-mode-0103"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103
{
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103_FULL_SEARCH = 0,
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103_START_HINT = 1,
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103_START_HINT_LIMITED_DISTANCE = 2,
} D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103;
```

<a id="d3d12ddi-video-encoder-frame-motion-search-mode-config-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_CONFIG_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_CONFIG_0103_0
{
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103    MotionSearchMode;
    UINT                                                    SearchDeviationLimit;
} D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_CONFIG_0103_0;
```

<a id="d3d12ddi-video-encoder-frame-motion-vectors-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_VECTORS_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_VECTORS_0103_0
{
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    union
    {
        // Use with: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103_GPU_TEXTURE
        D3D12DDI_HRESOURCE hDrvOpaqueLayoutBuffer;
        // Use with: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103_CPU_BUFFER
        D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_0103_0* pCPUBuffer;
    };
} D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_VECTORS_0103_0;
```

<a id="d3d12ddi-video-encoder-frame-subregion-layout-config-support-0095"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095 {
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_AV1_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095* pAV1Support;
    };
} D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_CONFIG_SUPPORT_0095;
```

<a id="d3d12ddi-video-encoder-frame-subregion-layout-mode-0080"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080
{
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_FULL_FRAME = 0,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_BYTES_PER_SUBREGION = 1,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_SQUARE_UNITS_PER_SUBREGION_ROW_UNALIGNED  = 2,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_UNIFORM_PARTITIONING_ROWS_PER_SUBREGION = 3,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_UNIFORM_PARTITIONING_SUBREGIONS_PER_FRAME = 4,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0095_UNIFORM_GRID_PARTITION = 5,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0095_CONFIGURABLE_GRID_PARTITION = 6,
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0103_AUTO = 7,
} D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080;
```

<a id="d3d12ddi-video-encoder-frame-subregion-metadata-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_METADATA_0083_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_METADATA_0083_0
{
    UINT64 bSize;
    UINT64 bStartOffset;
    UINT64 bHeaderSize;
} D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_METADATA_0083_0;
```

<a id="d3d12ddi-video-encoder-frame-type-h264-0080"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080
{
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080_I_FRAME = 0,
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080_P_FRAME = 1,
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080_B_FRAME = 2,
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080_IDR_FRAME = 3,
} D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080;
```

<a id="d3d12ddi-video-encoder-frame-type-hevc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080
{
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080_I_FRAME = 0,
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080_P_FRAME = 1,
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080_B_FRAME = 2,
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080_IDR_FRAME = 3,
} D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080;
```

<a id="d3d12ddi-video-encoder-heap-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_HEAP_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_HEAP_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAG_NONE_0080 = 0x0,
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAG_ALLOW_SUBREGION_NOTIFICATION_ARRAY_OF_BUFFERS_0104_0 = 0x1,
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAG_ALLOW_SUBREGION_NOTIFICATION_SINGLE_BUFFER_0104_0 = 0x2,
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAG_ALLOW_DIRTY_REGIONS_0107_0 = 0x4,
    D3D12DDI_VIDEO_ENCODER_HEAP_FLAG_ALLOW_RATE_CONTROL_FRAME_ANALYSIS_0110_0 = 0x8,
} D3D12DDI_VIDEO_ENCODER_HEAP_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-input-map-data-0103"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_0103

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_0103
{
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103 MapType;
    union
    {
        // For MapType: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103_QUANTIZATION_MATRIX
        D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_QUANTIZATION_MATRIX_0103 Quantization;
        // For MapType: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103_DIRTY_REGIONS
        D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_DIRTY_REGIONS_0103 DirtyRegions;
        // For MapType: D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103_MOTION_VECTORS
        D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_MOTION_VECTORS_0107 MotionVectors;
    };
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_0103;
```

<a id="d3d12ddi-video-encoder-input-map-data-dirty-regions-0103"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_DIRTY_REGIONS_0103

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_DIRTY_REGIONS_0103
{
    BOOL FullFrameIdentical;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_MAP_VALUES_MODE_0103 MapValuesType;
    D3D12DDI_HRESOURCE hDrvDirtyRegionsMap;
    UINT SourceDPBFrameReference;
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_DIRTY_REGIONS_0103;
```

<a id="d3d12ddi-video-encoder-input-map-data-motion-vectors-0107"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_MOTION_VECTORS_0107

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_MOTION_VECTORS_0107
{
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_CONFIG_0103_0 MotionSearchModeConfiguration;
    UINT NumHintsPerPixel;
    D3D12DDI_HRESOURCE* hDrvsMotionVectorMaps;
    UINT* pMotionVectorMapsSubresources;
    D3D12DDI_HRESOURCE* hDrvsMotionVectorMapsMetadata;
    UINT* pMotionVectorMapsMetadataSubresources;
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105 MotionUnitPrecision;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0108_0 PictureControlConfiguration;
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_MOTION_VECTORS_0107;
```

<a id="d3d12ddi-video-encoder-input-map-data-quantization-matrix-0103"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_QUANTIZATION_MATRIX_0103

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_QUANTIZATION_MATRIX_0103
{
    D3D12DDI_HRESOURCE hDrvQuantizationMap;
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_QUANTIZATION_MATRIX_0103;
```

<a id="d3d12ddi-video-encoder-input-map-session-info-0103"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103
{
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 Codec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 Profile;
    D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2 Level;
    DXGI_FORMAT InputFormat;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 InputResolution;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SubregionFrameEncoding;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2 SubregionFrameEncodingData;
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103;
```

<a id="d3d12ddi-video-encoder-input-map-source-0103"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103
{
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103_CPU_BUFFER = 0,
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103_GPU_TEXTURE = 1,
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103;
```

<a id="d3d12ddi-video-encoder-input-map-type-0103"></a>

### D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103
{
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103_QUANTIZATION_MATRIX = 0,
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103_DIRTY_REGIONS = 1,
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103_MOTION_VECTORS = 2,
} D3D12DDI_VIDEO_ENCODER_INPUT_MAP_TYPE_0103;
```

<a id="d3d12ddi-video-encoder-intra-refresh-0080"></a>

### D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_0080
{
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080 Mode;
    UINT IntraRefreshDuration;
} D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_0080;
```

<a id="d3d12ddi-video-encoder-intra-refresh-mode-0080"></a>

### D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080
{
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_NONE_0080 = 0,
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_ROW_BASED_0080 = 1,
} D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_MODE_0080;
```

<a id="d3d12ddi-video-encoder-levels-h264-0080"></a>

### D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080
{
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_1 = 0,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_1b = 1,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_11 = 2,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_12 = 3,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_13 = 4,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_2 = 5,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_21 = 6,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_22 = 7,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_3 = 8,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_31 = 9,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_32 = 10,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_4 = 11,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_41 = 12,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_42 = 13,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_5 = 14,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_51 = 15,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_52 = 16,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_6 = 17,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_61 = 18,
    D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080_62 = 19,
} D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080;
```

<a id="d3d12ddi-video-encoder-levels-hevc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080
{
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_1 = 0,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_2 = 1,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_21 = 2,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_3 = 3,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_31 = 4,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_4 = 5,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_41 = 6,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_5 = 7,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_51 = 8,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_52 = 9,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_6 = 10,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_61 = 11,
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080_62 = 12,
} D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080;
```

<a id="d3d12ddi-video-encoder-level-setting-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_LEVELS_H264_0080* pH264LevelSetting;
        D3D12DDI_VIDEO_ENCODER_LEVEL_TIER_CONSTRAINTS_HEVC_0080* pHEVCLevelSetting;
        D3D12DDI_VIDEO_ENCODER_AV1_LEVEL_TIER_CONSTRAINTS_0095* pAV1LevelSetting;
    };
} D3D12DDI_VIDEO_ENCODER_LEVEL_SETTING_0080_2;
```

<a id="d3d12ddi-video-encoder-level-tier-constraints-hevc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_LEVEL_TIER_CONSTRAINTS_HEVC_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_LEVEL_TIER_CONSTRAINTS_HEVC_0080
{
    D3D12DDI_VIDEO_ENCODER_LEVELS_HEVC_0080 Level;
    D3D12DDI_VIDEO_ENCODER_TIER_HEVC_0080 Tier;
} D3D12DDI_VIDEO_ENCODER_LEVEL_TIER_CONSTRAINTS_HEVC_0080;
```

<a id="d3d12ddi-video-encoder-metadata-statistics-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_METADATA_STATISTICS_0083_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_METADATA_STATISTICS_0083_0
{
    UINT64 AverageQP;
    UINT64 IntraCodingUnitsCount;
    UINT64 InterCodingUnitsCount;
    UINT64 SkipCodingUnitsCount;
    UINT64 AverageMotionEstimationXDirection;
    UINT64 AverageMotionEstimationYDirection;
} D3D12DDI_VIDEO_ENCODER_METADATA_STATISTICS_0083_0;
```

<a id="d3d12ddi-video-encoder-motion-estimation-precision-mode-0080"></a>

### D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080
{
    D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080_MAXIMUM = 0,
    D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080_FULL_PIXEL = 1,
    D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080_HALF_PIXEL = 2,
    D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080_QUARTER_PIXEL = 3,
    D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0095_EIGHTH_PIXEL = 4,
} D3D12DDI_VIDEO_ENCODER_MOTION_ESTIMATION_PRECISION_MODE_0080;
```

<a id="d3d12ddi-video-encoder-motion-search-configuration-0107"></a>

### D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_CONFIGURATION_0107

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_CONFIGURATION_0107
{
    BOOL Enabled;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_0103 MotionSearchMode;
    BOOL BidirectionalRefFrameEnabled;
} D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_CONFIGURATION_0107;
```

<a id="d3d12ddi-video-encoder-motion-search-support-flags-0107"></a>

### D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAGS_0107

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAGS_0107
{
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAG_0107_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAG_0107_SUPPORTED = 0x1,
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAG_0107_MULTIPLE_HINTS = 0x2,
    D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAG_0107_GPU_TEXTURE_MULTIPLE_REFERENCES = 0x4,

} D3D12DDI_VIDEO_ENCODER_MOTION_SEARCH_SUPPORT_FLAGS_0107;
```

<a id="d3d12ddi-video-encoder-moveregion-info-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_0103_0
{
    UINT NumMoveRegions;
    D3D12DDI_VIDEO_ENCODER_MOVE_RECT_0103_0* pMoveRegions;
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_SEARCH_MODE_CONFIG_0103_0 MotionSearchModeConfiguration;
    UINT SourceDPBFrameReference;
    D3D12DDI_VIDEO_ENCODER_FRAME_INPUT_MOTION_UNIT_PRECISION_0105 MotionUnitPrecision;
    D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_FLAGS_0107 Flags;
} D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_0103_0;
```

<a id="d3d12ddi-video-encoder-moveregion-info-flags-0107"></a>

### D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_FLAGS_0107

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_FLAGS_0107
{
    D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_FLAG_0107_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_FLAG_0107_MULTIPLE_HINTS = 0x1,
} D3D12DDI_VIDEO_ENCODER_MOVEREGION_INFO_FLAGS_0107;
```

<a id="d3d12ddi-video-encoder-move-rect-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_MOVE_RECT_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_MOVE_RECT_0103_0
{
    POINT   SourcePoint;
    RECT    DestRect;
} D3D12DDI_VIDEO_ENCODER_MOVE_RECT_0103_0;
```

<a id="d3d12ddi-video-encoder-optional-metadata-enable-flags-0102-0"></a>

### D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0
{
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAG_0102_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAG_0102_0_QP_MAP = 0x1,
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAG_0102_0_SATD_MAP = 0x2,
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAG_0102_0_RC_BIT_ALLOCATION_MAP = 0x4,
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAG_0109_0_FRAME_PSNR = 0x8,
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAG_0109_0_SUBREGIONS_PSNR = 0x10,
} D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0;
```

<a id="d3d12ddi-video-encoder-output-metadata-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_OUTPUT_METADATA_0083_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_OUTPUT_METADATA_0083_0
{
    UINT64 EncodeErrorFlags; // D3D12DDI_VIDEO_ENCODER_ENCODE_ERROR_FLAGS_0082_0
    D3D12DDI_VIDEO_ENCODER_METADATA_STATISTICS_0083_0 EncodeStats;
    UINT64 EncodedBitstreamWrittenBytesCount;
    UINT64 WrittenSubregionsCount;
} D3D12DDI_VIDEO_ENCODER_OUTPUT_METADATA_0083_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0082_0
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_0082_0* pH264PicData;
        D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_0082_0* pHEVCPicData;
        D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC1_0100_0* pHEVCPicData1;
        D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_CODEC_DATA_0095* pAV1PicData;
    };
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-0108-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0108_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0108_0
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_0082_0* pH264PicData;
        D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC2_0108_0* pHEVCPicData;
        D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_CODEC_DATA_0095* pAV1PicData;
    };
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0108_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-h264-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_0082_0
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_H264_0080 FrameType;
    UINT pic_parameter_set_id;
    UINT idr_pic_id;
    UINT PictureOrderCountNumber;
    UINT FrameDecodingOrderNumber;
    UINT TemporalLayerIndex;
    UINT List0ReferenceFramesCount;
    UINT* pList0ReferenceFrames;
    UINT List1ReferenceFramesCount;
    UINT* pList1ReferenceFrames;
    UINT ReferenceFramesReconPictureDescriptorsCount;
    D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_H264_0080* pReferenceFramesReconPictureDescriptors;
    UCHAR adaptive_ref_pic_marking_mode_flag;
    UINT RefPicMarkingOperationsCommandsCount;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_MARKING_OPERATION_0082_0* pRefPicMarkingOperationsCommands;
    UINT List0RefPicModificationsCount;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_LIST_MODIFICATION_OPERATION_0082_0* pList0RefPicModifications;
    UINT List1RefPicModificationsCount;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_LIST_MODIFICATION_OPERATION_0082_0* pList1RefPicModifications;
    UINT QPMapValuesCount;
    INT8* pRateControlQPMap;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-h264-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAG_0080_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAG_0080_REQUEST_INTRA_CONSTRAINED_SLICES = 0x1,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAG_0099_0_REQUEST_NUM_REF_IDX_ACTIVE_OVERRIDE_FLAG_SLICE = 0x2,
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-h264-reference-picture-list-modification-operation-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_LIST_MODIFICATION_OPERATION_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_LIST_MODIFICATION_OPERATION_0082_0
{
    UCHAR modification_of_pic_nums_idc;
    UINT abs_diff_pic_num_minus1;
    UINT long_term_pic_num;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_LIST_MODIFICATION_OPERATION_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-h264-reference-picture-marking-operation-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_MARKING_OPERATION_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_MARKING_OPERATION_0082_0
{
    UCHAR memory_management_control_operation;
    UINT difference_of_pic_nums_minus1;
    UINT long_term_pic_num;
    UINT long_term_frame_idx;
    UINT max_long_term_frame_idx_plus1;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_H264_REFERENCE_PICTURE_MARKING_OPERATION_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-hevc1-0100-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC1_0100_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC1_0100_0
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAGS_0082_0 Flags;
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080 FrameType;
    UINT slice_pic_parameter_set_id;
    UINT PictureOrderCountNumber;
    UINT TemporalLayerIndex;
    UINT List0ReferenceFramesCount;
    UINT* pList0ReferenceFrames;
    UINT List1ReferenceFramesCount;
    UINT* pList1ReferenceFrames;
    UINT ReferenceFramesReconPictureDescriptorsCount;
    D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_HEVC_0082_0* pReferenceFramesReconPictureDescriptors;
    UINT List0RefPicModificationsCount;
    UINT* pList0RefPicModifications;
    UINT List1RefPicModificationsCount;
    UINT* pList1RefPicModifications;
    UINT QPMapValuesCount;
    INT8* pRateControlQPMap;
    UCHAR diff_cu_chroma_qp_offset_depth;
    UCHAR log2_sao_offset_scale_luma;
    UCHAR log2_sao_offset_scale_chroma;
    UCHAR log2_max_transform_skip_block_size_minus2;
    UCHAR chroma_qp_offset_list_len_minus1;
    CHAR cb_qp_offset_list[6];
    CHAR cr_qp_offset_list[6];
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC1_0100_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-hevc2-0108-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC2_0108_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC2_0108_0
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAGS_0082_0 Flags;
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080 FrameType;
    UINT slice_pic_parameter_set_id;
    UINT PictureOrderCountNumber;
    UINT TemporalLayerIndex;
    UINT List0ReferenceFramesCount;
    UINT* pList0ReferenceFrames;
    UINT List1ReferenceFramesCount;
    UINT* pList1ReferenceFrames;
    UINT ReferenceFramesReconPictureDescriptorsCount;
    D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_HEVC_0082_0* pReferenceFramesReconPictureDescriptors;
    UINT List0RefPicModificationsCount;
    UINT* pList0RefPicModifications;
    UINT List1RefPicModificationsCount;
    UINT* pList1RefPicModifications;
    UINT QPMapValuesCount;
    INT8* pRateControlQPMap;
    UCHAR diff_cu_chroma_qp_offset_depth;
    UCHAR log2_sao_offset_scale_luma;
    UCHAR log2_sao_offset_scale_chroma;
    UCHAR log2_max_transform_skip_block_size_minus2;
    UCHAR chroma_qp_offset_list_len_minus1;
    CHAR cb_qp_offset_list[6];
    CHAR cr_qp_offset_list[6];
    UINT num_ref_idx_l0_active_minus1;
    UINT num_ref_idx_l1_active_minus1;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC2_0108_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-hevc-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_0082_0
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAGS_0082_0 Flags;
    D3D12DDI_VIDEO_ENCODER_FRAME_TYPE_HEVC_0080 FrameType;
    UINT slice_pic_parameter_set_id;
    UINT PictureOrderCountNumber;
    UINT TemporalLayerIndex;
    UINT List0ReferenceFramesCount;
    UINT* pList0ReferenceFrames;
    UINT List1ReferenceFramesCount;
    UINT* pList1ReferenceFrames;
    UINT ReferenceFramesReconPictureDescriptorsCount;
    D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_HEVC_0082_0* pReferenceFramesReconPictureDescriptors;
    UINT List0RefPicModificationsCount;
    UINT* pList0RefPicModifications;
    UINT List1RefPicModificationsCount;
    UINT* pList1RefPicModifications;
    UINT QPMapValuesCount;
    INT8* pRateControlQPMap;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-codec-data-hevc-flags-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAGS_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAGS_0082_0
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAG_0082_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAG_0082_0_REQUEST_INTRA_CONSTRAINED_SLICES = 0x1,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAG_0099_0_REQUEST_NUM_REF_IDX_ACTIVE_OVERRIDE_FLAG_SLICE = 0x2,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAG_0100_0_CROSS_COMPONENT_PREDICTION = 0x4,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAG_0100_0_CHROMA_QP_OFFSET_LIST = 0x8,
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_HEVC_FLAGS_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-desc-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0082_0
{
    UINT IntraRefreshFrameIndex;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0082_0 PictureControlCodecData;
    D3D12DDI_VIDEO_ENCODE_REFERENCE_FRAMES_0080 ReferenceFrames;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0082_0;
```

<a id="d3d12ddi-video-encoder-picture-control-desc-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0103_0
{
    UINT IntraRefreshFrameIndex;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_CODEC_DATA_0108_0 PictureControlCodecData;
    D3D12DDI_VIDEO_ENCODE_REFERENCE_FRAMES_0080 ReferenceFrames;
    D3D12DDI_VIDEO_ENCODER_FRAME_MOTION_VECTORS_0103_0 MotionVectors;
    D3D12DDI_VIDEO_ENCODER_DIRTY_REGIONS_0103_0 DirtyRects;
    D3D12DDI_VIDEO_ENCODER_QUANTIZATION_OPAQUE_MAP_0103_0 QuantizationTextureMap;
    D3D12DDI_VIDEO_ENCODER_FRAME_ANALYSIS_0110_0 FrameAnalysis;
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_DESC_0103_0;
```

<a id="d3d12ddi-video-encoder-picture-control-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAG_0080_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAG_0080_USED_AS_REFERENCE_PICTURE = 0x1,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAG_0103_ENABLE_QUANTIZATION_MATRIX_INPUT = 0x2,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAG_0103_ENABLE_DIRTY_REGIONS_INPUT = 0x4,
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAG_0103_ENABLE_MOTION_VECTORS_INPUT = 0x8,
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-picture-control-subregions-layout-data-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2
{
    UINT DataSize;
    union
    {
        const D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_SLICES_0080* pSlicesPartition_H264;
        const D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_SLICES_0080* pSlicesPartition_HEVC;
        const D3D12DDI_VIDEO_ENCODER_AV1_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_TILES_0095* pTilesPartition_AV1;
    };
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2;
```

<a id="d3d12ddi-video-encoder-picture-control-subregions-layout-data-slices-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_SLICES_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_SLICES_0080
{
    // Mutually exclusive options according to selected mode
    union
    {
        // Use with mode: D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_BYTES_PER_SUBREGION
        UINT MaxBytesPerSlice;

        // Use with mode: D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_SQUARE_UNITS_PER_SUBREGION_ROW_UNALIGNED
        UINT NumberOfCodingUnitsPerSlice;

        // Use with mode: D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_UNIFORM_PARTITIONING_ROWS_PER_SUBREGION
        UINT NumberOfRowsPerSlice;

        // Use with mode: D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080_UNIFORM_PARTITIONING_SUBREGIONS_PER_FRAME
        UINT NumberOfSlicesPerFrame;
    };
} D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_SLICES_0080;
```

<a id="d3d12ddi-video-encoder-picture-resolution-desc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080
{
    UINT Width;
    UINT Height;
} D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080;
```

<a id="d3d12ddi-video-encoder-picture-resolution-ratio-desc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_RATIO_DESC_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_RATIO_DESC_0080
{
    UINT WidthRatio;
    UINT HeightRatio;
} D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_RATIO_DESC_0080;
```

<a id="d3d12ddi-video-encoder-profile-desc-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080* pH264Profile;
        D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0080* pHEVCProfile;
        D3D12DDI_VIDEO_ENCODER_AV1_PROFILE_0095* pAV1Profile;
    };
} D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2;
```

<a id="d3d12ddi-video-encoder-profile-h264-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080
{
    D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080_MAIN = 0,
    D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080_HIGH = 1,
    D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080_HIGH_10 = 2,
} D3D12DDI_VIDEO_ENCODER_PROFILE_H264_0080;
```

<a id="d3d12ddi-video-encoder-profile-hevc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0080
{
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0080_MAIN = 0,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0080_MAIN10 = 1,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN12 = 2,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN10_422 = 3,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN12_422 = 4,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN_444 = 5,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN10_444 = 6,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN12_444 = 7,
    D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0100_0_MAIN16_444 = 8,
} D3D12DDI_VIDEO_ENCODER_PROFILE_HEVC_0080;
```

<a id="d3d12ddi-video-encoder-qpmap-configuration-0107"></a>

### D3D12DDI_VIDEO_ENCODER_QPMAP_CONFIGURATION_0107

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_QPMAP_CONFIGURATION_0107
{
    BOOL Enabled;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SOURCE_0103 MapSource;
} D3D12DDI_VIDEO_ENCODER_QPMAP_CONFIGURATION_0107;
```

<a id="d3d12ddi-video-encoder-quantization-opaque-map-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_QUANTIZATION_OPAQUE_MAP_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_QUANTIZATION_OPAQUE_MAP_0103_0
{
    D3D12DDI_HRESOURCE hDrvOpaqueLayoutBuffer;
} D3D12DDI_VIDEO_ENCODER_QUANTIZATION_OPAQUE_MAP_0103_0;
```

<a id="d3d12ddi-video-encoder-rate-control-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2
{
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080 Mode;
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAGS_0080 Flags;
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CONFIGURATION_PARAMS_0080_2 ConfigParams;
    DXGI_RATIONAL TargetFrameRate;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2;
```

<a id="d3d12ddi-video-encoder-rate-control-absolute-qp-map-0096"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_ABSOLUTE_QP_MAP_0096

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_ABSOLUTE_QP_MAP_0096
{
    UINT QualityVsSpeed;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_ABSOLUTE_QP_MAP_0096;
```

<a id="d3d12ddi-video-encoder-rate-control-cbr1-0096"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR1_0096

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR1_0096
{
    UINT InitialQP;
    UINT MinQP;
    UINT MaxQP;
    UINT64 MaxFrameBitSize;
    UINT64 TargetBitRate;
    UINT64 VBVCapacity;
    UINT64 InitialVBVFullness;
    UINT QualityVsSpeed;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR1_0096;
```

<a id="d3d12ddi-video-encoder-rate-control-cbr-0080"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR_0080
{
    UINT InitialQP;
    UINT MinQP;
    UINT MaxQP;
    UINT64 MaxFrameBitSize;
    UINT64 TargetBitRate;
    UINT64 VBVCapacity;
    UINT64 InitialVBVFullness;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR_0080;
```

<a id="d3d12ddi-video-encoder-rate-control-configuration-params-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CONFIGURATION_PARAMS_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CONFIGURATION_PARAMS_0080_2
{
    UINT DataSize;
    union
    {
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP_0080* pConfiguration_CQP;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR_0080* pConfiguration_CBR;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR_0080* pConfiguration_VBR;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR_0080_2* pConfiguration_QVBR;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP1_0096* pConfiguration_CQP1;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CBR1_0096* pConfiguration_CBR1;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR1_0096* pConfiguration_VBR1;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR1_0096* pConfiguration_QVBR1;
        const D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_ABSOLUTE_QP_MAP_0096* pConfiguration_AbsoluteQPMap;
    };
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CONFIGURATION_PARAMS_0080_2;
```

<a id="d3d12ddi-video-encoder-rate-control-cqp1-0096"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP1_0096

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP1_0096
{
    UINT ConstantQP_FullIntracodedFrame;
    UINT ConstantQP_InterPredictedFrame_PrevRefOnly;
    UINT ConstantQP_InterPredictedFrame_BiDirectionalRef;
    UINT QualityVsSpeed;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP1_0096;
```

<a id="d3d12ddi-video-encoder-rate-control-cqp-0080"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP_0080
{
    UINT ConstantQP_FullIntracodedFrame;
    UINT ConstantQP_InterPredictedFrame_PrevRefOnly;
    UINT ConstantQP_InterPredictedFrame_BiDirectionalRef;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_CQP_0080;
```

<a id="d3d12ddi-video-encoder-rate-control-flags-0080"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAGS_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAGS_0080
{
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_ENABLE_DELTA_QP = 0x1,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_ENABLE_FRAME_ANALYSIS = 0x2,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_ENABLE_QP_RANGE = 0x4,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_ENABLE_INITIAL_QP = 0x8,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_ENABLE_MAX_FRAME_SIZE = 0x10,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0080_1_ENABLE_VBV_SIZES = 0x20,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0096_ENABLE_EXTENSION1_SUPPORT = 0x40,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0096_ENABLE_QUALITY_VS_SPEED = 0x80,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAG_0113_ENABLE_SPATIAL_ADAPTIVE_QP = 0x100,
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_FLAGS_0080;
```

<a id="d3d12ddi-video-encoder-rate-control-mode-0080"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080
{
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080_ABSOLUTE_QP_MAP = 0,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080_CQP  = 1,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080_CBR  = 2,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080_VBR  = 3,
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080_QVBR = 4,
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_MODE_0080;
```

<a id="d3d12ddi-video-encoder-rate-control-qvbr1-0096"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR1_0096

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR1_0096
{
    UINT InitialQP;
    UINT MinQP;
    UINT MaxQP;
    UINT64 MaxFrameBitSize;
    UINT64 TargetAvgBitRate;
    UINT64 PeakBitRate;
    UINT ConstantQualityTarget;
    UINT64 VBVCapacity;
    UINT64 InitialVBVFullness;
    UINT QualityVsSpeed;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR1_0096;
```

<a id="d3d12ddi-video-encoder-rate-control-qvbr-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR_0080_2
{
    UINT InitialQP;
    UINT MinQP;
    UINT MaxQP;
    UINT64 MaxFrameBitSize;
    UINT64 TargetAvgBitRate;
    UINT64 PeakBitRate;
    UINT ConstantQualityTarget;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_QVBR_0080_2;
```

<a id="d3d12ddi-video-encoder-rate-control-vbr1-0096"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR1_0096

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR1_0096
{
    UINT InitialQP;
    UINT MinQP;
    UINT MaxQP;
    UINT64 MaxFrameBitSize;
    UINT64 TargetAvgBitRate;
    UINT64 PeakBitRate;
    UINT64 VBVCapacity;
    UINT64 InitialVBVFullness;
    UINT QualityVsSpeed;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR1_0096;
```

<a id="d3d12ddi-video-encoder-rate-control-vbr-0080"></a>

### D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR_0080
{
    UINT InitialQP;
    UINT MinQP;
    UINT MaxQP;
    UINT64 MaxFrameBitSize;
    UINT64 TargetAvgBitRate;
    UINT64 PeakBitRate;
    UINT64 VBVCapacity;
    UINT64 InitialVBVFullness;
} D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_VBR_0080;
```

<a id="d3d12ddi-video-encoder-reference-picture-descriptor-h264-0080"></a>

### D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_H264_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_H264_0080
{
    UINT ReconstructedPictureResourceIndex;
    BOOL IsLongTermReference;
    UINT LongTermPictureIdx;
    UINT PictureOrderCountNumber;
    UINT FrameDecodingOrderNumber;
    UINT TemporalLayerIndex;
} D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_H264_0080;
```

<a id="d3d12ddi-video-encoder-reference-picture-descriptor-hevc-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_HEVC_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_HEVC_0082_0
{
    UINT ReconstructedPictureResourceIndex;
    BOOL IsRefUsedByCurrentPic;
    BOOL IsLongTermReference;
    UINT PictureOrderCountNumber;
    UINT TemporalLayerIndex;
} D3D12DDI_VIDEO_ENCODER_REFERENCE_PICTURE_DESCRIPTOR_HEVC_0082_0;
```

<a id="d3d12ddi-video-encoder-resolution-support-limits-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2
{
    UINT MaxSubregionsNumber;
    UINT MaxIntraRefreshFrameDuration;
    UINT SubregionBlockPixelsSize;
    UINT QPMapRegionPixelsSize;
} D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2;
```

<a id="d3d12ddi-video-encoder-resolution-support-limits-0107"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107
{
    /* Below match existing D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0080_2 */
    UINT MaxSubregionsNumber;
    UINT MaxIntraRefreshFrameDuration;
    UINT SubregionBlockPixelsSize;
    UINT QPMapRegionPixelsSize;

    /* Below are new arguments for D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107 */
    D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_QPMAP_0107 QPMap;
    D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_DIRTY_REGIONS_0107 DirtyRegions;
    D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_MOTION_SEARCH_0107 MotionSearch;
    D3D12DDI_FEATURE_DATA_VIDEO_ENCODER_RESOLUTION_SUPPORT_FRAME_ANALYSIS_0110 FrameAnalysis;
} D3D12DDI_VIDEO_ENCODER_RESOLUTION_SUPPORT_LIMITS_0107;
```

<a id="d3d12ddi-video-encoder-resolve-input-param-layout-input-arguments-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_INPUT_ARGUMENTS_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_INPUT_ARGUMENTS_0103_0
{
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_SESSION_INFO_0103 SessionInfo;
    D3D12DDI_VIDEO_ENCODER_INPUT_MAP_DATA_0103 InputData;
} D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_INPUT_ARGUMENTS_0103_0;
```

<a id="d3d12ddi-video-encoder-resolve-input-param-layout-output-arguments-0103-0"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_OUTPUT_ARGUMENTS_0103_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_OUTPUT_ARGUMENTS_0103_0
{
    D3D12DDI_HRESOURCE hDrvOpaqueLayoutBuffer;
} D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_OUTPUT_ARGUMENTS_0103_0;
```

<a id="d3d12ddi-video-encoder-resolve-metadata-input-arguments-0080-2"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0080_2

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0080_2
{
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 EncoderCodec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 EncoderProfile;
    DXGI_FORMAT EncoderInputFormat;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 EncodedPictureEffectiveResolution;
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 HWLayoutEncoderMetadataDrvBuffer;
} D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0080_2;
```

<a id="d3d12ddi-video-encoder-resolve-metadata-input-arguments-0102-0"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0102_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0102_0
{
    D3D12DDI_VIDEO_ENCODER_CODEC_0080 EncoderCodec;
    D3D12DDI_VIDEO_ENCODER_PROFILE_DESC_0080_2 EncoderProfile;
    DXGI_FORMAT EncoderInputFormat;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 EncodedPictureEffectiveResolution;
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 HWLayoutEncoderMetadataDrvBuffer;
    D3D12DDI_VIDEO_ENCODER_OPTIONAL_METADATA_ENABLE_FLAGS_0102_0 OptionalMetadata;
    D3D12DDI_VIDEO_ENCODER_CODEC_CONFIGURATION_0082_0 CodecConfiguration;
} D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0102_0;
```

<a id="d3d12ddi-video-encoder-resolve-metadata-output-arguments-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0082_0
{
    // Selected current API-corresponding layout: D3D12DDI_VIDEO_ENCODER_OUTPUT_METADATA_0083_0; not a historical negotiation rule
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 ResolvedLayoutEncoderMetadataDrvBuffer;
} D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0082_0;
```

<a id="d3d12ddi-video-encoder-resolve-metadata-output-arguments-0102-0"></a>

### D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0102_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0102_0
{
    // Selected current API-corresponding layout: D3D12DDI_VIDEO_ENCODER_OUTPUT_METADATA_0083_0; not a historical negotiation rule
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 ResolvedLayoutEncoderMetadataDrvBuffer;
    D3D12DDI_HRESOURCE hDrvQPMap;
    D3D12DDI_HRESOURCE hDrvSATDMap;
    D3D12DDI_HRESOURCE hDrvBitAllocationMap;
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 ResolvedFramePSNRData;
    D3D12DDI_VIDEO_ENCODER_ENCODE_OPERATION_METADATA_BUFFER_0080_2 ResolvedSubregionsPSNRData;
} D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0102_0;
```

<a id="d3d12ddi-video-encoder-sequence-control-desc-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_DESC_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_DESC_0082_0
{
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAGS_0082_0 Flags;
    D3D12DDI_VIDEO_ENCODER_INTRA_REFRESH_0080 IntraRefreshConfig;
    D3D12DDI_VIDEO_ENCODER_RATE_CONTROL_0080_2 RateControl;
    D3D12DDI_VIDEO_ENCODER_PICTURE_RESOLUTION_DESC_0080 PictureTargetResolution;
    D3D12DDI_VIDEO_ENCODER_FRAME_SUBREGION_LAYOUT_MODE_0080 SelectedLayoutMode;
    D3D12DDI_VIDEO_ENCODER_PICTURE_CONTROL_SUBREGIONS_LAYOUT_DATA_0080_2 FrameSubregionsLayoutData;
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0 CodecGopSequence;
} D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_DESC_0082_0;
```

<a id="d3d12ddi-video-encoder-sequence-control-flags-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAGS_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAGS_0082_0
{
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAG_0082_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAG_0082_0_RESOLUTION_CHANGE = 0x1,
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAG_0082_0_RATE_CONTROL_CHANGE = 0x2,
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAG_0082_0_SUBREGION_LAYOUT_CHANGE = 0x4,
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAG_0082_0_REQUEST_INTRA_REFRESH = 0x8,
    D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAG_0082_0_GOP_SEQUENCE_CHANGE = 0x10,
} D3D12DDI_VIDEO_ENCODER_SEQUENCE_CONTROL_FLAGS_0082_0;
```

<a id="d3d12ddi-video-encoder-sequence-gop-structure-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0
{
    UINT DataSize;
    union
    {
        D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_H264_0082_0* pH264GroupOfPictures;
        D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_HEVC_0082_0* pHEVCGroupOfPictures;
        D3D12DDI_VIDEO_ENCODER_AV1_SEQUENCE_STRUCTURE_0095* pAV1SequenceStructure;
    };
} D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_0082_0;
```

<a id="d3d12ddi-video-encoder-sequence-gop-structure-h264-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_H264_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_H264_0082_0
{
    UINT GOPLength;
    UINT PPicturePeriod;
    UCHAR pic_order_cnt_type;
    UCHAR log2_max_frame_num_minus4;
    UCHAR log2_max_pic_order_cnt_lsb_minus4;
} D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_H264_0082_0;
```

<a id="d3d12ddi-video-encoder-sequence-gop-structure-hevc-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_HEVC_0082_0

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_HEVC_0082_0
{
    UINT GOPLength;
    UINT PPicturePeriod;
    UCHAR log2_max_pic_order_cnt_lsb_minus4;
} D3D12DDI_VIDEO_ENCODER_SEQUENCE_GOP_STRUCTURE_HEVC_0082_0;
```

<a id="d3d12ddi-video-encoder-subregion-compressed-bitstream-0102"></a>

### D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_0102

```cpp
typedef struct D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_0102
{
    D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_BUFFER_MODE_0104 BufferMode;
    UINT ExpectedSubregionCount;
    UINT64* pSubregionBitstreamsBaseOffsets;
    D3D12DDI_HRESOURCE* hDrvSubregionBitstreams;
    D3D12DDI_HRESOURCE* hDrvSubregionSizes;
    D3D12DDI_HRESOURCE* hDrvSubregionOffsets;
    D3D12DDI_HFENCE* hDrvSubregionFences;
    UINT64 *pSubregionFenceValues;
} D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_0102;
```

<a id="d3d12ddi-video-encoder-subregion-compressed-bitstream-buffer-mode-0104"></a>

### D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_BUFFER_MODE_0104

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_BUFFER_MODE_0104
{
    D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_BUFFER_MODE_0104_ARRAY_OF_BUFFERS = 0,
    D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_BUFFER_MODE_0104_SINGLE_BUFFER = 1,
} D3D12DDI_VIDEO_ENCODER_SUBREGION_COMPRESSED_BITSTREAM_BUFFER_MODE_0104;
```

<a id="d3d12ddi-video-encoder-support-flags-0083-0"></a>

### D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAGS_0083_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAGS_0083_0
{
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_GENERAL_SUPPORT_OK = 0x1,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_RECONFIGURATION_AVAILABLE = 0x2,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RESOLUTION_RECONFIGURATION_AVAILABLE = 0x4,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_VBV_SIZE_CONFIG_AVAILABLE = 0x8,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_FRAME_ANALYSIS_AVAILABLE = 0x10,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RECONSTRUCTED_FRAMES_REQUIRE_TEXTURE_ARRAYS = 0x20,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_DELTA_QP_AVAILABLE = 0x40,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_SUBREGION_LAYOUT_RECONFIGURATION_AVAILABLE = 0x80,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_ADJUSTABLE_QP_RANGE_AVAILABLE = 0x100,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_INITIAL_QP_AVAILABLE = 0x200,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_RATE_CONTROL_MAX_FRAME_SIZE_AVAILABLE = 0x400,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_SEQUENCE_GOP_RECONFIGURATION_AVAILABLE = 0x800,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0083_0_MOTION_ESTIMATION_PRECISION_MODE_LIMIT_AVAILABLE = 0x1000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0096_RATE_CONTROL_EXTENSION1_SUPPORT = 0x2000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0096_RATE_CONTROL_QUALITY_VS_SPEED_AVAILABLE = 0x4000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0101_READABLE_RECONSTRUCTED_PICTURE_LAYOUT_AVAILABLE = 0x8000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0102_PER_BLOCK_QP_MAP_METADATA_AVAILABLE = 0x10000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0102_PER_BLOCK_SATD_MAP_METADATA_AVAILABLE = 0x20000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0102_PER_BLOCK_RC_BIT_ALLOCATION_MAP_METADATA_AVAILABLE = 0x40000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0104_SUBREGION_NOTIFICATION_ARRAY_OF_BUFFERS_AVAILABLE = 0x80000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0104_SUBREGION_NOTIFICATION_SINGLE_BUFFER_AVAILABLE = 0x100000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0109_FRAME_PSNR_METADATA_AVAILABLE = 0x200000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0109_SUBREGIONS_PSNR_METADATA_AVAILABLE = 0x400000,
    D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAG_0113_RATE_CONTROL_SPATIAL_ADAPTIVE_QP_AVAILABLE = 0x800000,
} D3D12DDI_VIDEO_ENCODER_SUPPORT_FLAGS_0083_0;
```

<a id="d3d12ddi-video-encoder-tier-hevc-0080"></a>

### D3D12DDI_VIDEO_ENCODER_TIER_HEVC_0080

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_TIER_HEVC_0080
{
    D3D12DDI_VIDEO_ENCODER_TIER_HEVC_0080_MAIN = 0,
    D3D12DDI_VIDEO_ENCODER_TIER_HEVC_0080_HIGH = 1,
} D3D12DDI_VIDEO_ENCODER_TIER_HEVC_0080;
```

<a id="d3d12ddi-video-encoder-validation-flags-0082-0"></a>

### D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAGS_0082_0

```cpp
typedef enum D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAGS_0082_0
{
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_NONE = 0x0,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_CODEC_NOT_SUPPORTED = 0x1,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_INPUT_FORMAT_NOT_SUPPORTED = 0x8,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_CODEC_CONFIGURATION_NOT_SUPPORTED = 0x10,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_RATE_CONTROL_MODE_NOT_SUPPORTED = 0x20,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_RATE_CONTROL_CONFIGURATION_NOT_SUPPORTED = 0x40,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_INTRA_REFRESH_MODE_NOT_SUPPORTED = 0x80,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_SUBREGION_LAYOUT_MODE_NOT_SUPPORTED = 0x100,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_RESOLUTION_NOT_SUPPORTED_IN_LIST = 0x200,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0082_0_GOP_STRUCTURE_NOT_SUPPORTED = 0x800,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0095_SUBREGION_LAYOUT_DATA_NOT_SUPPORTED = 0x1000,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0107_QPMAP_NOT_SUPPORTED = 0x2000,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0107_DIRTY_REGIONS_NOT_SUPPORTED = 0x4000,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0107_MOTION_SEARCH_NOT_SUPPORTED = 0x8000,
    D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAG_0110_FRAME_ANALYSIS_NOT_SUPPORTED = 0x10000,
} D3D12DDI_VIDEO_ENCODER_VALIDATION_FLAGS_0082_0;
```

<a id="d3d12ddi-video-encode-reconstructed-picture-0080"></a>

### D3D12DDI_VIDEO_ENCODE_RECONSTRUCTED_PICTURE_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODE_RECONSTRUCTED_PICTURE_0080
{
    D3D12DDI_HRESOURCE hDrvTexture2D;
    UINT InputFrameSubresource;
} D3D12DDI_VIDEO_ENCODE_RECONSTRUCTED_PICTURE_0080;
```

<a id="d3d12ddi-video-encode-reference-frames-0080"></a>

### D3D12DDI_VIDEO_ENCODE_REFERENCE_FRAMES_0080

```cpp
typedef struct D3D12DDI_VIDEO_ENCODE_REFERENCE_FRAMES_0080
{
    UINT NumTexture2Ds;
    D3D12DDI_HRESOURCE* hDrvTexture2Ds;
    UINT* pSubresources;
} D3D12DDI_VIDEO_ENCODE_REFERENCE_FRAMES_0080;
```

<a id="pfnd3d12ddi-calcprivatevideoencoderheapsize-0080-2"></a>

### PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0080_2

```cpp
typedef SIZE_T (APIENTRY* PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0080_2)(
    D3D12DDI_HDEVICE hDrvDevice,
    const D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0080_2 *pCreationArgs);
```

<a id="pfnd3d12ddi-calcprivatevideoencoderheapsize-0110-0"></a>

### PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0110_0

```cpp
typedef SIZE_T (APIENTRY* PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0110_0)(
    D3D12DDI_HDEVICE hDrvDevice,
    const D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0110_0 *pCreationArgs);
```

<a id="pfnd3d12ddi-calcprivatevideoencodersize-0082-0"></a>

### PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0

```cpp
typedef SIZE_T (APIENTRY* PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0)(
    D3D12DDI_HDEVICE hDrvDevice,
    const D3D12DDIARG_CREATE_VIDEO_ENCODER_0082_0 *pCreationArgs);
```

<a id="pfnd3d12ddi-createvideoencoderheap-0080-2"></a>

### PFND3D12DDI_CREATEVIDEOENCODERHEAP_0080_2

```cpp
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATEVIDEOENCODERHEAP_0080_2 )(
    D3D12DDI_HDEVICE hDrvDevice,
    const D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0080_2* pCreationArgs,
    D3D12DDI_HVIDEOENCODERHEAP_0080 hDrvVideoEncoderHeap );
```

<a id="pfnd3d12ddi-createvideoencoderheap-0110-0"></a>

### PFND3D12DDI_CREATEVIDEOENCODERHEAP_0110_0

```cpp
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATEVIDEOENCODERHEAP_0110_0 )(
    D3D12DDI_HDEVICE hDrvDevice,
    const D3D12DDIARG_CREATE_VIDEO_ENCODER_HEAP_0110_0* pCreationArgs,
    D3D12DDI_HVIDEOENCODERHEAP_0080 hDrvVideoEncoderHeap );
```

<a id="pfnd3d12ddi-createvideoencoder-0082-0"></a>

### PFND3D12DDI_CREATEVIDEOENCODER_0082_0

```cpp
typedef HRESULT (APIENTRY* PFND3D12DDI_CREATEVIDEOENCODER_0082_0)(
    D3D12DDI_HDEVICE hDrvDevice,
    const D3D12DDIARG_CREATE_VIDEO_ENCODER_0082_0* pCreationArgs,
    D3D12DDI_HVIDEOENCODER_0080 hDrvVideoEncoder);
```

<a id="pfnd3d12ddi-destroyvideoencoderheap-0080"></a>

### PFND3D12DDI_DESTROYVIDEOENCODERHEAP_0080

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROYVIDEOENCODERHEAP_0080 )(
    D3D12DDI_HDEVICE hDrvDevice,
    D3D12DDI_HVIDEOENCODERHEAP_0080 hDrvVideoEncoderHeap);
```

<a id="pfnd3d12ddi-destroyvideoencoder-0080"></a>

### PFND3D12DDI_DESTROYVIDEOENCODER_0080

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROYVIDEOENCODER_0080 )(
    D3D12DDI_HDEVICE hDrvDevice,
    D3D12DDI_HVIDEOENCODER_0080 hDrvVideoEncoder);
```

<a id="pfnd3d12ddi-video-encode-frame-0082-0"></a>

### PFND3D12DDI_VIDEO_ENCODE_FRAME_0082_0

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_VIDEO_ENCODE_FRAME_0082_0 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    D3D12DDI_HVIDEOENCODER_0080 hDrvEncoder,
    D3D12DDI_HVIDEOENCODERHEAP_0080 hDrvVideoEncoderHeap,
    const D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0082_0* pInputStreamArguments,
    const D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0080_2* pOutputStreamArguments);
```

<a id="pfnd3d12ddi-video-encode-frame-0102-0"></a>

### PFND3D12DDI_VIDEO_ENCODE_FRAME_0102_0

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_VIDEO_ENCODE_FRAME_0102_0 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    D3D12DDI_HVIDEOENCODER_0080 hDrvEncoder,
    D3D12DDI_HVIDEOENCODERHEAP_0080 hDrvVideoEncoderHeap,
    const D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0102_0* pInputStreamArguments,
    const D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0102_0* pOutputStreamArguments);
```

<a id="pfnd3d12ddi-video-encode-frame-0103-0"></a>

### PFND3D12DDI_VIDEO_ENCODE_FRAME_0103_0

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_VIDEO_ENCODE_FRAME_0103_0 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    D3D12DDI_HVIDEOENCODER_0080 hDrvEncoder,
    D3D12DDI_HVIDEOENCODERHEAP_0080 hDrvVideoEncoderHeap,
    const D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_INPUT_STREAM_ARGUMENTS_0103_0* pInputStreamArguments,
    const D3D12DDI_VIDEO_ENCODER_ENCODEFRAME_OUTPUT_STREAM_ARGUMENTS_0102_0* pOutputStreamArguments);
```

<a id="pfnd3d12ddi-video-encode-resolve-output-metadata-0082-0"></a>

### PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0082_0

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0082_0 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    const D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0080_2* pInputArguments,
    const D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0082_0* pOutputArguments);
```

<a id="pfnd3d12ddi-video-encode-resolve-output-metadata-0102-0"></a>

### PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0102_0

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0102_0 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    const D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_INPUT_ARGUMENTS_0102_0* pInputArguments,
    const D3D12DDI_VIDEO_ENCODER_RESOLVE_METADATA_OUTPUT_ARGUMENTS_0102_0* pOutputArguments);
```

<a id="pfnd3d12ddi-video-resolve-input-param-layout-0103-0"></a>

### PFND3D12DDI_VIDEO_RESOLVE_INPUT_PARAM_LAYOUT_0103_0

```cpp
typedef VOID ( APIENTRY* PFND3D12DDI_VIDEO_RESOLVE_INPUT_PARAM_LAYOUT_0103_0 )(
    D3D12DDI_HCOMMANDLIST hDrvCommandList,
    const D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_INPUT_ARGUMENTS_0103_0* pInputArguments,
    const D3D12DDI_VIDEO_ENCODER_RESOLVE_INPUT_PARAM_LAYOUT_OUTPUT_ARGUMENTS_0103_0* pOutputArguments);
```

<a id="encoding-ddi-table-associations"></a>

## Published callback-table associations

| Table | Actual member declaration |
|---|---|
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0082_0` | `PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0                      pfnCalcPrivateVideoEncoderSize;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0082_0` | `PFND3D12DDI_CREATEVIDEOENCODER_0082_0                               pfnCreateVideoEncoder;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0082_0` | `PFND3D12DDI_DESTROYVIDEOENCODER_0080                                pfnDestroyVideoEncoder;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0082_0` | `PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0080_2                  pfnCalcPrivateVideoEncoderHeapSize;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0082_0` | `PFND3D12DDI_CREATEVIDEOENCODERHEAP_0080_2                           pfnCreateVideoEncoderHeap;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0082_0` | `PFND3D12DDI_DESTROYVIDEOENCODERHEAP_0080                            pfnDestroyVideoEncoderHeap;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0110` | `PFND3D12DDI_CALCPRIVATEVIDEOENCODERSIZE_0082_0                      pfnCalcPrivateVideoEncoderSize;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0110` | `PFND3D12DDI_CREATEVIDEOENCODER_0082_0                               pfnCreateVideoEncoder;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0110` | `PFND3D12DDI_DESTROYVIDEOENCODER_0080                                pfnDestroyVideoEncoder;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0110` | `PFND3D12DDI_CALCPRIVATEVIDEOENCODERHEAPSIZE_0110_0                  pfnCalcPrivateVideoEncoderHeapSize;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0110` | `PFND3D12DDI_CREATEVIDEOENCODERHEAP_0110_0                           pfnCreateVideoEncoderHeap;` |
| `D3D12DDI_DEVICE_FUNCS_VIDEO_0110` | `PFND3D12DDI_DESTROYVIDEOENCODERHEAP_0080                            pfnDestroyVideoEncoderHeap;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0082_0` | `PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0082_0 pfnResolveEncoderOutputMetadata;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0082_0` | `PFND3D12DDI_VIDEO_ENCODE_FRAME_0082_0                   pfnEncodeFrame;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0102` | `PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0102_0 pfnResolveEncoderOutputMetadata;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0102` | `PFND3D12DDI_VIDEO_ENCODE_FRAME_0102_0                   pfnEncodeFrame;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0103` | `PFND3D12DDI_VIDEO_ENCODE_RESOLVE_OUTPUT_METADATA_0102_0 pfnResolveEncoderOutputMetadata;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0103` | `PFND3D12DDI_VIDEO_ENCODE_FRAME_0103_0                   pfnEncodeFrame;` |
| `D3D12DDI_COMMAND_LIST_FUNCS_VIDEO_ENCODE_0103` | `PFND3D12DDI_VIDEO_RESOLVE_INPUT_PARAM_LAYOUT_0103_0     pfnResolveInputParamLayout;` |
