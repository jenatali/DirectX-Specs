# Driver command-recording objects

**Coverage: selected draft.** This subordinate reference defines recording-object
lifecycle and its versioned driver interfaces. The application contracts remain
in [Command recording and execution](execution.md#recording-lifecycle).

<a id="object-model"></a>

## Objects and recording lifetime

The application command allocator supplies storage for recorded commands; an
application command list is the object through which commands are recorded.
At the driver interface using the `_0040` recording-object protocol, a command
pool owns command storage, a command recorder supplies the CPU recording
machinery, and a driver command list identifies a command-list object.
The older protocol combines the pool/recording-storage role in a driver
command allocator.

An application allocator permits at most one recording list at a time.
This API restriction is not a one-recorder-per-pool DDI restriction. Several
driver recorders may target the same pool and record concurrently. Each
recorder may be used for only one recording at a time. The driver must support
concurrent recording through distinct recorders sharing a pool; this does not
permit concurrent calls on the same command-list object.

Reset associates a driver list's new recording with the supplied recorder.
Normal Close ends recording and releases that recorder's recording use.
The [failed-recording rules](#failed-recording) distinguish logical API
closure from delivery of the Close DDI. The recorder's use is a
recording lease, not allocation and destruction of a recorder for every
recording. Creation of an initially recording application list can require
both driver-list creation and recording initialization; there is no requirement
that each such DDI call identify a different recording.

Recorded commands remain backed by their pool after Close. An idle recorder
may be reused, retargeted or destroyed while older recordings execute, provided
their pool storage remains valid. Reusing the recorder or resetting the
command-list object does not reset that storage or prove GPU completion.
The [recorded-stream and bundle-storage rules](execution.md#command-storage)
apply independently of recorder reuse.

<a id="recording-identifiers"></a>

### Recording-generation identifiers

The `UINT64 ID` in driver-list creation and reset identifies a recording
generation, not the persistent application or driver command-list object.
Valid IDs must be unique within the driver device, apart from eventual
64-bit wraparound. Zero and `UINT64_MAX` are reserved; `UINT64_MAX` denotes
no current recording, including initially closed creation.

Creation and an implicit reset that initialize the same recording may use
the same ID. A later recording established by application Reset receives
a new valid ID. The contract does not prescribe a counter, starting value,
ordering of IDs or uniqueness across distinct driver devices.

<a id="target-and-reclamation"></a>

### Targets, reset and destruction

The runtime may change a recorder's pool target only while that recorder is
idle. An idle recorder may retain a stale, non-owning target identity; it does
not keep the target pool alive. Before recording resumes, the runtime must
establish a valid target association. This is not a null-handle unbinding
protocol.

A pool must not be reset while a CPU recording is using it or before all
GPU uses of its recorded storage have completed. As with application allocator
Reset, the application is responsible for GPU-completion synchronization.
Reset makes that storage available for reuse and invalidates the old
recordings, even if their command-list objects still exist. An invalidated
recording must not subsequently be submitted or selected for execution.
Destroying the pool likewise requires no active CPU recording or outstanding
GPU use and ends the validity of its recorded storage. Neither operation
waits for the application to make otherwise invalid use safe.

Before destroying a recorder, its associated command list must be closed or
destroyed. Destroying a command list may abandon an open recording without a
preceding Close DDI; the driver must release that recording's CPU recorder
use during abandonment. This does not allow destruction of a recorder still
used by a live recording list.

Destroying a list ends its driver-object lifetime, not the lifetime of
pool-backed recordings that are still executing. Driver cleanup must also
obey the [command-list resource-lifetime rule](execution.md#driver-responsibilities):
referenced resources may already have been destroyed. Cleanup must not
require an implicit Close that accesses those resources.

<a id="failed-recording"></a>

### Failed recording and recorder reuse

A stored recording error makes the failed recording non-executable and
prevents that command-list object from being reset again. Application Close
still ends the logical recording and releases its allocator/recorder recording
use, even when the runtime omits the Close DDI. If the device remains usable,
a different compatible list may begin recording with that allocator/recorder
while the failed list object remains alive. This permission does not reset
pool storage or relax its GPU-use and reclamation requirements.

An error callback does not itself release the runtime's recording ownership.
In particular, an error reported during Reset or its state initialization
can be stored for Close to report even when the application Reset returns
`S_OK`. Until application Close or list destruction ends that recording,
the runtime must not reuse its recorder for another recording. The driver
may clean up failed-recording state earlier, provided it preserves the
other recordings and object lifetimes required by this contract. A retained
lifetime reference is distinct from active recording ownership.

The driver must tolerate later reuse, idle retargeting or destruction of the
recorder after logical recording closure without a preceding Close DDI.
The [associated-list destruction precondition](#target-and-reclamation)
uses this logical closed state, not guaranteed delivery of a Close DDI.
Later cleanup or destruction of the failed list must not release, reset or
invalidate recording state or backing allocations now used by a successor
recording, or dereference a recorder or pool object whose lifetime has ended.
The driver must not require an implicit Close or access to already-destroyed
referenced resources to perform that cleanup.

<a id="private-storage"></a>

## Private storage and errors

For each allocator, pool, recorder or list, the runtime queries the matching
private size, allocates the supplied driver-private region, and supplies its
driver handle at creation. Size queries return a byte count as `SIZE_T`, not
an `HRESULT`, and do not construct the object. Creation arguments used for
sizing describe the object to be created; output/private handles are supplied
for creation, not returned as newly allocated COM objects by the driver.

The actual supplied private pointer is aligned to `MEMORY_ALLOCATION_ALIGNMENT`:
16 bytes on Win64 and 8 bytes on Win32. The runtime owns this region. The
driver owns auxiliary allocations it makes for the object and must release
them during destruction. The driver's Destroy DDI does not free the supplied
runtime-owned private region.

Creation returns an `HRESULT`. On failure the driver must clean up its partial
initialization and auxiliary allocations; the runtime does not call Destroy
for a failed creation. After successful creation, subsequent abandonment or
initialization failure requires destruction of that successfully created
object before its supplied storage is released.

The VOID command-list Reset and Close DDIs do not return the application method's `HRESULT`.
The [recording-error callback and deferred-error rules](execution.md#lifecycle-driver-errors)
govern recording failures and the runtime's suppression of later recording
calls. Application allocator Reset's GPU-completion precondition is not
implemented by waiting inside the pool/allocator reset DDI.

The [CPU threading contract](execution.md#threading) and the pool/recorder
concurrency rules above apply together. Interface selection and table/callback
lifetimes follow [Runtime UMD initialization](driver-initialization.md#table-selection).
The runtime owns the API-to-DDI adaptation; applications do not negotiate
these DDI versions.

<a id="recording-descriptions"></a>

## Shared recording descriptions

`D3D12DDI_COMMAND_LIST_TYPE` distinguishes direct-style recording from bundles.
It is not the application's full `D3D12_COMMAND_LIST_TYPE` enumeration;
compute, copy and video API enum values must not be cast to this DDI enum.
`QueueFlags` identifies the workload capabilities of the recording machinery.
The paging flag is a driver-interface flag, not an additional application
command-list type.

```cpp
typedef enum D3D12DDI_COMMAND_LIST_TYPE
{
    D3D12DDI_COMMAND_LIST_TYPE_DIRECT  = 0,
    D3D12DDI_COMMAND_LIST_TYPE_BUNDLE  = 1,
} D3D12DDI_COMMAND_LIST_TYPE;

typedef enum D3D12DDI_COMMAND_QUEUE_FLAGS
{
    D3D12DDI_COMMAND_QUEUE_FLAG_NONE                = 0x00000000,
    D3D12DDI_COMMAND_QUEUE_FLAG_3D                  = 0x00000001,
    D3D12DDI_COMMAND_QUEUE_FLAG_COMPUTE             = 0x00000002,
    D3D12DDI_COMMAND_QUEUE_FLAG_COPY                = 0x00000004,
    D3D12DDI_COMMAND_QUEUE_FLAG_PAGING              = 0x00000008,
    D3D12DDI_COMMAND_QUEUE_FLAG_0020_VIDEO_LEGACY   = 0x00000010, // Deprecated
    D3D12DDI_COMMAND_QUEUE_FLAG_0022_VIDEO_DECODE   = 0x00000010,
    D3D12DDI_COMMAND_QUEUE_FLAG_0022_VIDEO_PROCESS  = 0x00000020,
    D3D12DDI_COMMAND_QUEUE_FLAG_0053_VIDEO_ENCODE   = 0x00000040,

} D3D12DDI_COMMAND_QUEUE_FLAGS;

typedef enum D3D12DDI_COMMAND_LIST_FLAGS
{
    D3D12DDI_COMMAND_LIST_FLAG_NONE                             = 0x00000000,
    D3D12DDI_COMMAND_LIST_FLAG_ENABLE_MARKERS                   = 0x00000001,
    D3D12DDI_COMMAND_LIST_FLAG_0010_ENABLE_FULLPIPELINE_MARKERS = 0x00000002,
} D3D12DDI_COMMAND_LIST_FLAGS;
```

`CommandListFlags` carries diagnostic-marker controls, including the full-pipeline
marker flag named with the `_0010` suffix. It is not the application's
`D3D12_COMMAND_LIST_FLAGS` creation-flags type. `NodeMask` selects the list's
device node under the [device/node model](devices-and-capabilities.md#adapter-identity-and-linked-nodes).
Neither these fields nor queue flags replace the recording-generation ID.

<a id="command-pools"></a>

## Command pools

The `_0040` protocol separates pool storage from the recorder. `PoolFlags`
has only the `NONE` value in this declaration. The size query reports private
pool bytes; Create initializes the supplied pool handle for the driver device.
Reset reclaims that pool's recording storage under the
[CPU/GPU preconditions](#target-and-reclamation); Destroy releases its
driver-owned allocations and ends the pool object.

```cpp
typedef enum D3D12DDI_COMMAND_POOL_FLAGS
{
    D3D12DDI_COMMAND_POOL_FLAG_NONE          = 0x00000000
} D3D12DDI_COMMAND_POOL_FLAGS;

typedef struct D3D12DDIARG_CREATE_COMMAND_POOL_0040
{
    D3D12DDI_COMMAND_POOL_FLAGS PoolFlags;
} D3D12DDIARG_CREATE_COMMAND_POOL_0040;

typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALC_PRIVATE_COMMAND_POOL_SIZE_0040 )( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_POOL_0040* );
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATE_COMMAND_POOL_0040 ) ( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_POOL_0040*, D3D12DDI_HCOMMANDPOOL_0040 );
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROY_COMMAND_POOL_0040 ) ( D3D12DDI_HDEVICE, D3D12DDI_HCOMMANDPOOL_0040 );
typedef VOID ( APIENTRY* PFND3D12DDI_RESET_COMMAND_POOL_0040 ) ( D3D12DDI_HDEVICE, D3D12DDI_HCOMMANDPOOL_0040 );
```

<a id="command-recorders"></a>

## Command recorders

`QueueFlags` describes the recorder's workload capabilities; `RecorderFlags`
has only the `NONE` value in this declaration. The size query reports private
recorder bytes; Create initializes the supplied recorder handle. Creation does
not begin a command-list recording or supply a pool target.

`PFND3D12DDI_COMMAND_RECORDER_SET_COMMAND_POOL_AS_TARGET_0040` takes, in order,
the driver device, recorder and pool handles. It establishes the recorder's
target under the [idle-target rules](#target-and-reclamation); it does not
reset the pool or invalidate its older recordings. Destroy releases the
recorder's driver-owned allocations under the associated-list precondition,
without reclaiming unrelated, still-valid pool-backed recordings.

```cpp
typedef enum D3D12DDI_COMMAND_RECORDER_FLAGS
{
    D3D12DDI_COMMAND_RECORDER_FLAG_NONE = 0x00000000
} D3D12DDI_COMMAND_RECORDER_FLAGS;

typedef struct D3D12DDIARG_CREATE_COMMAND_RECORDER_0040
{
    D3D12DDI_COMMAND_QUEUE_FLAGS QueueFlags;
    D3D12DDI_COMMAND_RECORDER_FLAGS RecorderFlags;
} D3D12DDIARG_CREATE_COMMAND_RECORDER_0040;

typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALC_PRIVATE_COMMAND_RECORDER_SIZE_0040 )( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_RECORDER_0040* );
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATE_COMMAND_RECORDER_0040 ) ( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_RECORDER_0040*, D3D12DDI_HCOMMANDRECORDER_0040 );
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROY_COMMAND_RECORDER_0040 ) ( D3D12DDI_HDEVICE, D3D12DDI_HCOMMANDRECORDER_0040 );
typedef VOID ( APIENTRY* PFND3D12DDI_COMMAND_RECORDER_SET_COMMAND_POOL_AS_TARGET_0040 ) ( D3D12DDI_HDEVICE, D3D12DDI_HCOMMANDRECORDER_0040, D3D12DDI_HCOMMANDPOOL_0040 );
```

<a id="command-lists"></a>

## Command-list creation and recording

The `_0040` size query reports private list bytes. Create takes the driver
device, creation description, supplied driver-private list handle and runtime
list handle, in that order. The runtime handle identifies the list when the
driver calls its runtime callbacks; it is not the private-storage handle.
The creation description contains no pool or recorder handle.

Reset takes the existing driver list and a description containing the recorder
for the new recording, its generation ID and marker flags. The recorder must
be available for that recording and have a valid target pool. Reset starts a
new recording without reclaiming old pool storage. The runtime supplies the
initial pipeline state and other applicable state through state-setting DDIs,
not through these reset arguments; the canonical
[initial/reset state](command-list-state.md) defines those contributions.

Close takes the driver list, finishes its recording and releases recorder use;
it neither submits the commands nor proves GPU completion. Destroy takes the
driver device and list handles. It releases list-owned auxiliary state and
supports [abandonment without Close](#target-and-reclamation). Both functions
are shared with the older allocator-based protocol.

```cpp
typedef struct D3D12DDIARG_CREATE_COMMAND_LIST_0040
{
    D3D12DDI_COMMAND_LIST_TYPE   Type;
    D3D12DDI_COMMAND_QUEUE_FLAGS QueueFlags;
    UINT64                       ID;
    D3D12DDI_COMMAND_LIST_FLAGS  CommandListFlags;
    UINT                         NodeMask;
} D3D12DDIARG_CREATE_COMMAND_LIST_0040;

typedef struct D3D12DDIARG_RESETCOMMANDLIST_0040
{
    D3D12DDI_HCOMMANDRECORDER_0040   hDrvCommandRecorder;
    UINT64                           ID;
    D3D12DDI_COMMAND_LIST_FLAGS      CommandListFlags;
} D3D12DDIARG_RESETCOMMANDLIST_0040;

typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALC_PRIVATE_COMMAND_LIST_SIZE_0040 )( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_LIST_0040* );
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATE_COMMAND_LIST_0040 ) ( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_LIST_0040*, D3D12DDI_HCOMMANDLIST, D3D12DDI_HRTCOMMANDLIST );
typedef VOID ( APIENTRY* PFND3D12DDI_RESETCOMMANDLIST_0040 )( D3D12DDI_HCOMMANDLIST, _In_ CONST D3D12DDIARG_RESETCOMMANDLIST_0040*);
typedef VOID ( APIENTRY* PFND3D12DDI_CLOSECOMMANDLIST )( D3D12DDI_HCOMMANDLIST );
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROYCOMMANDLIST ) ( D3D12DDI_HDEVICE, D3D12DDI_HCOMMANDLIST );
```

<a id="legacy-allocators"></a>

## Older allocator-based interfaces

Before the `_0040` protocol, driver command allocators provide the recording
storage association. The allocator size query reports private allocator
bytes; Create initializes the supplied `hDrvCommandAllocator` contained in
the description. `Type` and `QueueFlags` describe the compatible recordings.
Destroy takes the driver device and allocator handles and releases the
allocator's driver-owned allocations.

`PFND3D12DDI_RESETCOMMANDALLOCATOR` takes only the allocator handle, not a
separate device handle. It reclaims the allocator's recorded-command storage,
with the same application CPU-recording/GPU-completion preconditions as
[ID3D12CommandAllocator::Reset](execution.md#allocator-reset). It does not
reset a command-list object or wait for outstanding GPU work.

```cpp
typedef struct D3D12DDIARG_CREATECOMMANDALLOCATOR
{
    D3D12DDI_HCOMMANDALLOCATOR   hDrvCommandAllocator;
    D3D12DDI_COMMAND_LIST_TYPE   Type;
    D3D12DDI_COMMAND_QUEUE_FLAGS QueueFlags;
} D3D12DDIARG_CREATECOMMANDALLOCATOR;

typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALCPRIVATECOMMANDALLOCATORSIZE )( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATECOMMANDALLOCATOR* );
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATECOMMANDALLOCATOR ) ( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATECOMMANDALLOCATOR* );
typedef VOID ( APIENTRY* PFND3D12DDI_DESTROYCOMMANDALLOCATOR ) ( D3D12DDI_HDEVICE, D3D12DDI_HCOMMANDALLOCATOR );
typedef VOID ( APIENTRY* PFND3D12DDI_RESETCOMMANDALLOCATOR ) ( D3D12DDI_HCOMMANDALLOCATOR );
```

The `_0001` list size/create functions take the driver device and the older
creation description. Unlike `_0040`, that description contains the supplied
driver-private and runtime list handles and the allocator association.
The older Reset takes the existing driver list and a reset description with
an allocator handle rather than a recorder handle. The ID still identifies
the recording generation, not an allocator or persistent list object.
`Slot` is retained legacy ABI; the runtime supplies `UINT_MAX`. It does not
establish an allocation-index, concurrent-recording-limit or slot-exhaustion
contract.

```cpp
typedef struct D3D12DDIARG_CREATE_COMMAND_LIST_0001
{
    D3D12DDI_HCOMMANDLIST        hDrvCommandList;
    D3D12DDI_HRTCOMMANDLIST      hRTCommandList;
    D3D12DDI_HCOMMANDALLOCATOR   hDrvCommandAllocator;
    D3D12DDI_COMMAND_LIST_TYPE   Type;
    D3D12DDI_COMMAND_QUEUE_FLAGS QueueFlags;
    UINT64                       ID;
    D3D12DDI_COMMAND_LIST_FLAGS  CommandListFlags;
    UINT                        NodeMask;
} D3D12DDIARG_CREATE_COMMAND_LIST_0001;

typedef struct D3D12DDIARG_RESETCOMMANDLIST
{
    D3D12DDI_HCOMMANDALLOCATOR  hDrvCommandAllocator;
    UINT                        Slot;
    UINT64                      ID;
    D3D12DDI_COMMAND_LIST_FLAGS CommandListFlags;
} D3D12DDIARG_RESETCOMMANDLIST;

typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALC_PRIVATE_COMMAND_LIST_SIZE_0001 )( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_LIST_0001* );
typedef HRESULT ( APIENTRY* PFND3D12DDI_CREATE_COMMAND_LIST_0001 ) ( D3D12DDI_HDEVICE, _In_ CONST D3D12DDIARG_CREATE_COMMAND_LIST_0001* );
typedef VOID ( APIENTRY* PFND3D12DDI_RESETCOMMANDLIST )( D3D12DDI_HCOMMANDLIST, _In_ CONST D3D12DDIARG_RESETCOMMANDLIST*);
```
