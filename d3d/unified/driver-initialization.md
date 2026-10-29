# Runtime UMD initialization

The user-mode display driver (UMD) interface is distinct from the application's COM interfaces. The runtime opens a driver adapter object, selects a mutually supported DDI version, provides private-device storage and creates the driver device. Driver function tables and runtime callback tables carry calls in opposite directions.

One DDI device is associated with each opened UMD adapter object. This does not restrict the number of application devices associated with a physical adapter.

The Direct3D runtime can open multiple graphics adapter objects from a single graphics adapter.

Driver handles identify driver objects or their supplied private storage. Runtime handles identify objects used when calling back into the runtime. These handle domains are not interchangeable. [Application device creation](devices-and-capabilities.md#ordinary-device-creation), [CPU threading and lifetime](execution.md#threading) and [device-removal reporting](diagnostics.md#device-removal-status) retain their separate contracts.

<a id="driver-entry"></a>

## Driver entry and adapter objects

### PFND3D12DDI_OPENADAPTER

The UMD exports `OpenAdapter12` with this function type:

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_OPENADAPTER)(
    _Inout_ D3D12DDIARG_OPENADAPTER* pArgs);

typedef struct D3D12DDIARG_OPENADAPTER
{
    D3D12DDI_HRTADAPTER hRTAdapter;
    D3D12DDI_HADAPTER hAdapter;
    CONST D3DDDI_ADAPTERCALLBACKS* pAdapterCallbacks;
    D3D12DDI_ADAPTERFUNCS* pAdapterFuncs;
} D3D12DDIARG_OPENADAPTER;
```

The runtime supplies `hRTAdapter`, its adapter callback table and writable adapter-function-table storage. On success, the driver returns `hAdapter` and fills the adapter function table. `hRTAdapter` is used with the runtime's adapter callbacks; `hAdapter` identifies the opened driver object in subsequent adapter DDIs.

The adapter callback table includes `pfnQueryAdapterInfoCb`, which the driver can use with `hRTAdapter` to query adapter information from the display miniport driver.

Adapter opening does not receive a selected DDI version. Version selection is a separate operation. The base adapter table and `D3D12DDI_ADAPTERFUNCS_0109` are ABI-compatible; a driver may fill the `_0109` form without requiring the runtime to have selected version 0109 for a device.

```cpp
typedef struct D3D12DDI_ADAPTERFUNCS
{
    PFND3D12DDI_CALCPRIVATEDEVICESIZE pfnCalcPrivateDeviceSize;
    PFND3D12DDI_CREATEDEVICE_0003 pfnCreateDevice;
    PFND3D12DDI_CLOSEADAPTER pfnCloseAdapter;
    PFND3D12DDI_GETSUPPORTEDVERSIONS pfnGetSupportedVersions;
    PFND3D12DDI_GETCAPS pfnGetCaps;
    PFND3D12DDI_GETOPTIONALDDITTABLES pfnGetOptionalDDITables;
    PFND3D12DDI_FILLDDITTABLE pfnFillDDITable;
    PFND3D12DDI_DESTROYDEVICE pfnDestroyDevice;
} D3D12DDI_ADAPTERFUNCS;

typedef struct D3D12DDI_ADAPTERFUNCS_0109
{
    PFND3D12DDI_CALCPRIVATEDEVICESIZE pfnCalcPrivateDeviceSize;
    PFND3D12DDI_CREATEDEVICE_0109 pfnCreateDevice;
    PFND3D12DDI_CLOSEADAPTER pfnCloseAdapter;
    PFND3D12DDI_GETSUPPORTEDVERSIONS pfnGetSupportedVersions;
    PFND3D12DDI_GETCAPS pfnGetCaps;
    PFND3D12DDI_GETOPTIONALDDITTABLES pfnGetOptionalDDITables;
    PFND3D12DDI_FILLDDITTABLE pfnFillDDITable;
    PFND3D12DDI_DESTROYDEVICE pfnDestroyDevice;
} D3D12DDI_ADAPTERFUNCS_0109;
```

The table's create-device function type does not establish the extent of every creation argument the runtime will supply. The driver must use the creation arguments' negotiated `Interface`/`Version` pair before accessing the [extended creation fields](#creation-tail).

`OpenAdapter12` returns an `HRESULT` indicating whether adapter opening succeeded. Required allocation failure can be reported as `E_OUTOFMEMORY`. The driver owns the adapter object and its allocations; the runtime uses `CloseAdapter` to end its successful opened lifetime.

### PFND3D12DDI_CLOSEADAPTER

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_CLOSEADAPTER)(
    D3D12DDI_HADAPTER hAdapter);
```

Closes the opened driver adapter object and releases its driver-owned resources. The device associated with that adapter is no longer in use when the adapter is closed. The return value reports completion or failure of closing the adapter; it is not a request to create or renegotiate a device.

<a id="interface-versions"></a>

## Supported interface versions

### PFND3D12DDI_GETSUPPORTEDVERSIONS

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_GETSUPPORTEDVERSIONS)(
    D3D12DDI_HADAPTER hAdapter,
    _Inout_ UINT32* puEntries,
    _Out_writes_opt_(*puEntries) UINT64* pSupportedDDIInterfaceVersions);
```

Each returned `UINT64` is one exact supported DDI version, not a ceiling that implicitly advertises every lower build. A selected version must appear in the driver's returned list and be supported by the runtime. Runtime and driver are independently versioned.

Runtime selection normally prefers the highest mutually supported version. Compatibility exclusions, including runtime or driver bugs, can require a shared version to be skipped. The driver must not rely on unconditional selection of the numerically highest advertised value.

A null version-array pointer queries the number of entries through `puEntries`. For retrieval, `*puEntries` supplies array capacity in `UINT64` elements; on successful retrieval it reports the actual returned count, which does not exceed that capacity. The runtime can accept fewer entries than the preceding size query reported. Returned order does not establish version preference.

The function returns an `HRESULT`. This is a full-list query/retrieval protocol, not a specified capacity-clamped prefix or `DXGI_ERROR_MORE_DATA` convention. No generic failure-count or untouched-array guarantee is implied.

The selected value is communicated for device sizing and creation through the `Interface` and `Version` fields. Together they represent the selected 64-bit DDI version; they are not the API SDK version, feature level or shader-model version. If no version is supported by both sides, a device cannot be created for that runtime/driver pair.

<a id="device-storage"></a>

## Private device storage

### PFND3D12DDI_CALCPRIVATEDEVICESIZE

```cpp
typedef SIZE_T (APIENTRY *PFND3D12DDI_CALCPRIVATEDEVICESIZE)(
    D3D12DDI_HADAPTER hAdapter,
    _In_ CONST D3D12DDIARG_CALCPRIVATEDEVICESIZE* pArgs);

typedef struct D3D12DDIARG_CALCPRIVATEDEVICESIZE
{
    UINT Interface;
    UINT Version;
    D3D12DDI_CREATE_DEVICE_FLAGS Flags;
} D3D12DDIARG_CALCPRIVATEDEVICESIZE;

typedef enum D3D12DDI_CREATE_DEVICE_FLAGS
{
    D3D12DDI_CREATE_DEVICE_FLAG_NONE = 0x0,
    D3D12DDI_CREATE_DEVICE_FLAG_DISABLE_IMPLICIT_MGPU = 0x1,
    D3D12DDI_CREATE_DEVICE_FLAG_DEBUGGABLE = 0x2,
} D3D12DDI_CREATE_DEVICE_FLAGS;
```

The CalcPrivateDeviceSize function determines the size of a memory region that the user-mode display driver requires from the Microsoft Direct3D runtime to store frequently-accessed data.

The return value is a byte size, not an `HRESULT`. The query describes the selected DDI interface/version and creation-mode flags. The flags identify requests to disable implicit multi-GPU operation or create a debuggable driver device; they are DDI creation flags, not the application device-factory flag enumeration.

The runtime allocates and frees the reported private-device region. The driver initializes that storage through `hDrvDevice` and owns any auxiliary allocations it makes. The storage supplied for creation must satisfy the size requirement for the actual creation parameters.

The private-device pointer is aligned to `MEMORY_ALLOCATION_ALIGNMENT`: 16 bytes on 64-bit Windows and 8 bytes on 32-bit Windows. This is not a 64-byte alignment guarantee.

<a id="device-creation"></a>

## Driver-device creation and destruction

### PFND3D12DDI_CREATEDEVICE_0003 and PFND3D12DDI_CREATEDEVICE_0109

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_CREATEDEVICE_0003)(
    D3D12DDI_HADAPTER hAdapter,
    _In_ CONST D3D12DDIARG_CREATEDEVICE_0003* pArgs);

typedef HRESULT (APIENTRY *PFND3D12DDI_CREATEDEVICE_0109)(
    D3D12DDI_HADAPTER hAdapter,
    _In_ CONST D3D12DDIARG_CREATEDEVICE_0109* pArgs);

typedef struct D3D12DDIARG_CREATEDEVICE_0003
{
    D3D12DDI_HRTDEVICE hRTDevice;
    UINT Interface;
    UINT Version;
    CONST D3DDDI_DEVICECALLBACKS* pKTCallbacks;
    D3D12DDI_HDEVICE hDrvDevice;
    union
    {
        CONST D3D12DDI_CORELAYER_DEVICECALLBACKS_0003* p12UMCallbacks;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0022* p12UMCallbacks_0022;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0050* p12UMCallbacks_0050;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0062* p12UMCallbacks_0062;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0112* p12UMCallbacks_0112;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0116* p12UMCallbacks_0116;
    };
    D3D12DDI_CREATE_DEVICE_FLAGS Flags;
} D3D12DDIARG_CREATEDEVICE_0003;

typedef struct D3D12DDIARG_CREATEDEVICE_0109
{
    D3D12DDI_HRTDEVICE hRTDevice;
    UINT Interface;
    UINT Version;
    CONST D3DDDI_DEVICECALLBACKS* pKTCallbacks;
    D3D12DDI_HDEVICE hDrvDevice;
    union
    {
        CONST D3D12DDI_CORELAYER_DEVICECALLBACKS_0003* p12UMCallbacks;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0022* p12UMCallbacks_0022;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0050* p12UMCallbacks_0050;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0062* p12UMCallbacks_0062;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0112* p12UMCallbacks_0112;
        CONST struct D3D12DDI_CORELAYER_DEVICECALLBACKS_0116* p12UMCallbacks_0116;
    };
    D3D12DDI_CREATE_DEVICE_FLAGS Flags;
    D3D12DDI_GPU_VIRTUAL_ADDRESS_RANGE* pReserveRanges;
    UINT NumReserveRanges;
} D3D12DDIARG_CREATEDEVICE_0109;
```

`hRTDevice` is the runtime handle used for callbacks. `hDrvDevice` identifies the supplied driver-private device storage, not a new allocation whose ownership transfers from the driver. `Interface`, `Version` and `Flags` specify the device being created.

`pKTCallbacks` supplies runtime callbacks that access kernel services. The `p12UMCallbacks` union supplies the applicable versioned core user-mode callback table. The union members are alternative views of one callback-table pointer, not independently supplied tables; the driver uses the layout applicable to the negotiated interface.

Creation supplies the adapter's selected core-version context for subsequent table population. It does not change the opened adapter table into a new negotiated table shape.

<a id="creation-tail"></a>

### Creation-tail availability and reserved GPUVAs

The two fields after `Flags` are `pReserveRanges` and `NumReserveRanges`. Their presence is guaranteed only when the creation `Interface`/`Version` pair indicates `D3D12DDI_SUPPORTED_0111` or later. A driver must establish that field-availability guarantee before reading them, including before reading the count. Accessing absent fields is undefined behavior.

**History:** The `_0109` type name predates the 0111 field-presence guarantee. Earlier versions do not guarantee that the tail exists; the type name is not an availability test.

```cpp
typedef struct D3D12DDI_GPU_VIRTUAL_ADDRESS_RANGE
{
    D3D12DDI_GPU_VIRTUAL_ADDRESS StartAddress;
    UINT64 SizeInBytes;
} D3D12DDI_GPU_VIRTUAL_ADDRESS_RANGE;
```

For a creation payload that includes the tail, `pReserveRanges` supplies `NumReserveRanges` GPU virtual address ranges. `StartAddress` is a GPU virtual address and `SizeInBytes` is the extent in bytes. The driver must immediately reserve the supplied ranges during creation, before its initialization allocations can consume addresses needed for replay.

Ranges provided to `CreateDevice` are not processed in any way and may or may not be aligned in offset or size and may or may not overlap, depending on driver behavior during record.

The driver reports creation completion through the `HRESULT`. A failing creation must clean up the driver's partial work and auxiliary allocations; it cannot depend on `DestroyDevice` being called to finish a failed creation.

### PFND3D12DDI_DESTROYDEVICE

```cpp
typedef VOID (APIENTRY *PFND3D12DDI_DESTROYDEVICE)(
    D3D12DDI_HDEVICE hDevice);
```

Destroys the successfully created driver device and releases its driver-owned auxiliary allocations. The runtime retains ownership of the supplied private-device region and frees that region after its use ends.

Successful DDI device creation establishes a destruction obligation even if the application device is not yet complete. If a subsequent initialization DDI fails and initial creation is abandoned, the runtime calls `DestroyDevice` before closing the adapter. This does not make a later failure of a delay-loaded feature a device-creation failure.

`DestroyDevice` is an adapter-table entry, so cleanup does not depend on successfully obtaining the device function tables. It has no return value.

<a id="capability-queries"></a>

## Adapter capability queries

### PFND3D12DDI_GETCAPS

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_GETCAPS)(
    D3D12DDI_HADAPTER hAdapter,
    _In_ CONST D3D12DDIARG_GETCAPS* pArgs);

typedef struct D3D12DDIARG_GETCAPS
{
    D3D12DDICAPS_TYPE Type;
    VOID* pInfo;
    VOID* pData;
    UINT DataSize;
} D3D12DDIARG_GETCAPS;
```

`Type` selects the driver capability query. `pInfo` and `pData` carry its query-specific information and payload; their inputs, outputs and permitted null values are defined by that query. `DataSize` describes the `pData` payload size in bytes. The function returns an `HRESULT` reporting the query result.

This is an adapter-level interface, not a one-to-one dispatch of an application `CheckFeatureSupport` call. Runtime capability results can be cached, combined or translated. [Feature-level reporting](devices-and-capabilities.md#driver-responsibilities) defines concrete query/result mappings, including the input and output fields of the 3D-pipeline-support query.

<a id="table-discovery"></a>

## Required and additional function tables

Core-DDI table families are `D3D12DDI_TABLE_TYPE_DEVICE_CORE`, `D3D12DDI_TABLE_TYPE_COMMAND_LIST_3D`, `D3D12DDI_TABLE_TYPE_COMMAND_QUEUE_3D` and, from core version 0020, the [applicable extended-feature table](#extended-features). The selected core version determines which layout of each family applies. The `_0096` extended-feature table replaces the older selector at that version; both are not required simultaneously.

Most required families have one copy. A graphics-capable device has two baseline 3D command-list copies, for DIRECT and BUNDLE recordings. A device without graphics-pipeline support has one baseline 3D command-list copy. These counts do not establish a contractual numeric DIRECT/BUNDLE assignment for `tableNum`.

### PFND3D12DDI_GETOPTIONALDDITTABLES

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_GETOPTIONALDDITTABLES)(
    D3D12DDI_HADAPTER hAdapter,
    _Inout_ UINT32* puEntries,
    _Out_writes_opt_(*puEntries) D3D12DDI_TABLE_REQUEST* pRequests);

typedef struct D3D12DDI_TABLE_REQUEST
{
    D3D12DDI_TABLE_TYPE tableType;
    UINT numTables;
} D3D12DDI_TABLE_REQUEST;
```

This query describes additional table instances requested by the driver. Only `D3D12DDI_TABLE_TYPE_COMMAND_LIST_3D` participates in this optional-table mechanism. Command-list tables belonging to separately negotiated extended features do not use it.

`puEntries` counts request records, not table copies. A null request-array pointer queries that count; a corresponding retrieval supplies storage for the reported records, with `puEntries` identifying the returned record count. The function returns an `HRESULT`. An empty request list requests no additional tables.

Within each record, `numTables` is the number of additional copies of `tableType`, not the total including required copies. Thus a graphics-capable device's two required 3D copies plus an optional request for three copies produces five. A zero-copy request adds no copies.

These are alternative dispatch-table instances, not additional advertised DDI versions. The driver can select an appropriate instance for a command list through the [table-selection callback](#table-selection).

### PFND3D12DDI_FILLDDITTABLE

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_FILLDDITTABLE)(
    D3D12DDI_HADAPTER hAdapter,
    D3D12DDI_TABLE_TYPE tableType,
    _Inout_ VOID* pTable,
    SIZE_T tableSize,
    UINT tableNum,
    _In_opt_ D3D12DDI_HRTTABLE hRTTable);
```

The driver fills the requested table instance and returns an `HRESULT`. `tableType` identifies the table family, `tableSize` is the output buffer size in bytes, and `tableNum` identifies the requested instance of that family. The index is not a DDI version number.

For core tables, the device creation `Interface`/`Version` context determines the applicable versioned layout. For an extended feature, the enabled feature version determines that feature's layout. The driver writes within the supplied table extent.

`pTable` is caller-owned writable storage for the call, not storage owned by the driver and not the stable runtime table handle. The driver provides function pointers according to each entry's contract. Presence of an entry in the layout does not itself require a non-null value: per-entry optionality rules apply.

For 3D command-list tables, `hRTTable` is the opaque runtime identity used for table selection. Its validity lasts until device destruction; it must not be interpreted as ownership of, or a right to retain or modify, the output buffer.

<a id="table-selection"></a>

### PFND3D12DDI_SETCOMMANDLISTDDITABLE_CB

```cpp
typedef VOID (APIENTRY CALLBACK *PFND3D12DDI_SETCOMMANDLISTDDITABLE_CB)(
    D3D12DDI_HRTCOMMANDLIST hRTCommandList,
    D3D12DDI_HRTTABLE hRTTable);
```

The driver calls this runtime callback to select which populated 3D command-list table is used for the specified runtime command-list object. That selection may change during the command list's lifetime. The callback has no return value.

Table selection does not renegotiate the core DDI version. The stable identity is the supplied runtime handle, not the transient table-population buffer. [Command recording](execution.md#recording-lifecycle) retains its own object, storage and threading rules.

<a id="extended-features"></a>

## Extended-feature negotiation

Extended features have their own `UINT32` version values in addition to the selected core DDI version. A version returned by the driver is an exact supported version; activation selects a value supported by both runtime and driver. Highest compatible is the normal intention, subject to compatibility exclusions as for the core interface.

The core version selects the extended-feature interface:

| Core version | Table selector and layout |
| --- | --- |
| Before `D3D12DDI_SUPPORTED_0020` | No generic extended-feature table or negotiation protocol |
| `D3D12DDI_SUPPORTED_0020` | `D3D12DDI_TABLE_TYPE_0020_EXTENDED_FEATURES`, `D3D12DDI_EXTENDED_FEATURES_FUNCS_0020` |
| From `D3D12DDI_SUPPORTED_0021`, before `_0096` | Same selector, `D3D12DDI_EXTENDED_FEATURES_FUNCS_0021`, adding callback installation |
| From `D3D12DDI_SUPPORTED_0096` | `D3D12DDI_TABLE_TYPE_0096_EXTENDED_FEATURES`, `D3D12DDI_EXTENDED_FEATURES_FUNCS_0096` |

```cpp
typedef struct D3D12DDI_EXTENDED_FEATURES_FUNCS_0020
{
    PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURES_0020 pfnGetSupportedExtendedFeatures;
    PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURE_VERSIONS_0020 pfnGetSupportedExtendedFeatureVersions;
    PFND3D12DDI_ENABLE_EXTENDED_FEATURE_0020 pfnEnableExtendedFeature;
} D3D12DDI_EXTENDED_FEATURES_FUNCS_0020;

typedef struct D3D12DDI_EXTENDED_FEATURES_FUNCS_0021
{
    PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURES_0020 pfnGetSupportedExtendedFeatures;
    PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURE_VERSIONS_0020 pfnGetSupportedExtendedFeatureVersions;
    PFND3D12DDI_ENABLE_EXTENDED_FEATURE_0020 pfnEnableExtendedFeature;
    PFND3D12DDI_SET_EXTENDED_FEATURE_CALLBACKS_0021 pfnSetExtendedFeatureCallbacks;
} D3D12DDI_EXTENDED_FEATURES_FUNCS_0021;

typedef struct D3D12DDI_EXTENDED_FEATURES_FUNCS_0096
{
    PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURES_0096 pfnGetSupportedExtendedFeatures;
    PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURE_VERSIONS_0020 pfnGetSupportedExtendedFeatureVersions;
    PFND3D12DDI_ENABLE_EXTENDED_FEATURE_0020 pfnEnableExtendedFeature;
    PFND3D12DDI_SET_EXTENDED_FEATURE_CALLBACKS_0021 pfnSetExtendedFeatureCallbacks;
} D3D12DDI_EXTENDED_FEATURES_FUNCS_0096;
```

`D3D12DDI_FEATURE_0020` identifies a feature family, not a feature version. Its identifier values are:

```cpp
typedef enum D3D12DDI_FEATURE_0020
{
    D3D12DDI_FEATURE_0020_VIDEO = 2,
    D3D12DDI_FEATURE_0021_SHADERCACHING = 4,
    D3D12DDI_FEATURE_0030_CONTENT_PROTECTION_RESOURCES = 5,
    D3D12DDI_FEATURE_0030_CONTENT_PROTECTION_STREAMING = 6,
    D3D12DDI_FEATURE_0033_METACOMMAND = 9,
    D3D12DDI_FEATURE_0043_RENDER_PASS = 10,
    D3D12DDI_FEATURE_0054_DOWNLEVEL_SUPPORT = 11,
    D3D12DDI_FEATURE_0076_PIN_RESOURCES = 12,
} D3D12DDI_FEATURE_0020;
```

An identifier's presence in this enumeration is not an advertisement of device support or an enabled feature. Support and versions are obtained through the queries below.

### PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURES_0020 and _0096

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURES_0020)(
    D3D12DDI_HDEVICE hDevice,
    _Inout_ UINT32* puFeatures,
    _Out_writes_opt_(*puFeatures) D3D12DDI_FEATURE_0020* pFeatures);

typedef HRESULT (APIENTRY *PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURES_0096)(
    D3D12DDI_HDEVICE hDevice,
    D3D12DDI_FEATURE_0020 HighestAllowedFeatureNumber,
    _Inout_ UINT32* puFeatures,
    _Out_writes_opt_(*puFeatures) D3D12DDI_FEATURE_0020* pFeatures);
```

Returns the supported extended-feature identifiers for the device. A null array queries the number of feature identifiers. A corresponding fetch supplies that reported element count for the same query inputs; the count must remain consistent between that query and fetch. A zero feature count means that no extended features are advertised.

The legacy form returns only defined feature identifiers with numeric values no greater than `D3D12DDI_FEATURE_0076_PIN_RESOURCES` (12), inclusive. Revision suffixes in identifier names do not remove those identifiers from that fixed range. The `_0096` form adds a caller-specified inclusive highest allowed feature identifier: returned feature IDs must not exceed `HighestAllowedFeatureNumber`. This is a bound on feature identifiers, not on their version numbers.

The function returns an `HRESULT`. These calls do not establish a capacity-clamped prefix, oversized-capacity normalization or `DXGI_ERROR_MORE_DATA` protocol. A failure does not imply that the output array was left untouched.

### PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURE_VERSIONS_0020

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_GET_SUPPORTED_EXTENDED_FEATURE_VERSIONS_0020)(
    D3D12DDI_HDEVICE hDevice,
    D3D12DDI_FEATURE_0020 Feature,
    _Inout_ UINT32* puFeatureVersions,
    _Out_writes_opt_(*puFeatureVersions) UINT32* pFeatureVersions);
```

For an advertised `Feature`, queries its supported exact version values. A null version array obtains the element count. A corresponding fetch supplies that count for the same device and feature; the reported count must remain consistent. An advertised feature has at least one supported version.

The returned `UINT32` values are the feature's versions, not core DDI version values or feature IDs. A selected version must be explicitly advertised and recognized by the runtime. If no version is supported by both sides, that feature cannot be activated through this protocol.

The function returns an `HRESULT`; no generic short-buffer status, partial-prefix contract or failure-output guarantee is specified here.

### PFND3D12DDI_ENABLE_EXTENDED_FEATURE_0020

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_ENABLE_EXTENDED_FEATURE_0020)(
    D3D12DDI_HDEVICE hDevice,
    D3D12DDI_FEATURE_0020 Feature,
    UINT32 FeatureVersion);
```

Enables `Feature` at the selected exact `FeatureVersion` and establishes its version context for subsequent table population. The return value reports whether enabling succeeded.

Successful enablement precedes requesting that feature's driver tables or supplying its callback tables. Those tables and callbacks must be ready before the feature is used. This defines prerequisites, not a fixed sequence between driver-table retrieval and callback installation.

Some feature table initialization is delayed until the feature is used through the API. It need not occur before application device creation completes. Failure of eager feature initialization fails device creation and follows the [creation cleanup obligation](#device-creation); failure of deferred initialization is handled by the triggering operation's API/feature error contract.

### PFND3D12DDI_SET_EXTENDED_FEATURE_CALLBACKS_0021

```cpp
typedef HRESULT (APIENTRY *PFND3D12DDI_SET_EXTENDED_FEATURE_CALLBACKS_0021)(
    D3D12DDI_HDEVICE hDevice,
    D3D12DDI_TABLE_TYPE Table,
    _In_reads_(TableSize) const void* pTable,
    SIZE_T TableSize);
```

Installs the runtime callback table identified by `Table` for an enabled feature. `pTable` supplies that feature/version's callback layout; `TableSize` is its size in bytes. The function returns an `HRESULT`. It is not limited to video callbacks.

Unlike the caller-owned output buffer of `FillDDITable`, this is runtime-provided callback storage whose [lifetime](#callback-storage) permits the driver to retain the supplied read-only pointer.

<a id="callback-storage"></a>

## Runtime callback storage

| Supplied table | Storage lifetime guaranteed to the driver |
| --- | --- |
| Adapter `pAdapterCallbacks` | Through adapter close |
| Device `pKTCallbacks` | Through device destruction |
| Applicable versioned device `p12UMCallbacks` | Through device destruction |
| Extended-feature table supplied by `SetExtendedFeatureCallbacks` | Through device destruction |

The runtime owns this storage. The driver may retain the supplied read-only table pointer for the stated lifetime; it does not free or modify the table. The guarantee concerns table storage and does not extend the validity of destroyed object handles or waive a callback's individual preconditions.

The callback direction is driver to runtime. Device/core errors and command-list recording errors retain their distinct reporting paths; [deferred recording errors](execution.md#lifecycle-driver-errors) defines the command-list error callback's accepted errors and propagation. The [runtime cache callback reference](runtime-cache-discovery.md#compiled-value-lookup) defines its separate data ownership and result protocol.
