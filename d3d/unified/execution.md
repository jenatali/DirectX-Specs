# Command recording and execution

**Coverage: source-preserving draft.** This chapter assembles selected recording, submission, completion, lifetime, and threading text. It does not yet cover every execution contract.

<a id="execution-objects"></a>

## Command lists and queues

To render, applications can
create a *command list* off of the device, on which traditional
rendering APIs can be called. Each individual command list is
not free-threaded, but multiple command lists *can* be recorded
concurrently (which enables multi-threading).

### Bundles

Bundles are a 'second level' of command lists. The purpose of Bundles is to allow apps to
group a small number of API commands together for later execution. At
Bundle creation time, the driver will perform as much pre-processing as
is possible to make these cheap to execute later. Command lists, on the
other hand, are typically executed only a single time (to enable
multi-threaded command buffer recording). However, a command list can be
executed multiple times (as long as the application ensures that the
previous executions have completed before submitting new executions).

<a id="command-list-types"></a>

### Queue types

Command queues can be created by calling [`ID3D12Device::CreateCommandQueue`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12device-createcommandqueue). The `Type` field of `D3D12_COMMAND_QUEUE_DESC` is a [`D3D12_COMMAND_LIST_TYPE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type) indicating what type of queue should be created, and therefore, what type of commands can be submitted to that queue. Remember that bundles can only be called from direct command lists and can't be submitted directly to a queue. The supported queue types are:

- [`D3D12_COMMAND_LIST_TYPE_DIRECT`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type)
- [`D3D12_COMMAND_LIST_TYPE_COMPUTE`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type)
- [`D3D12_COMMAND_LIST_TYPE_COPY`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_command_list_type)
- [`D3D12_COMMAND_LIST_TYPE_VIDEO_DECODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_command_list_type)
- [`D3D12_COMMAND_LIST_TYPE_VIDEO_PROCESS`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_command_list_type)
- [`D3D12_COMMAND_LIST_TYPE_VIDEO_ENCODE`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_command_list_type)

In general, DIRECT queues and command lists accept graphics, compute, and copy commands, COMPUTE queues and command lists accept compute and copy related commands, and COPY queues and command lists accept only copy commands.

Video-decode recording, lifecycle, ordering and lifetime are defined in [Video](video.md#video-recording-lifetime), using the common queue/submission and completion contracts.

The distinct video-process list/queue and processor route is defined in [Video processing](video.md#video-processing-model), using the common creation, submission, threading and completion contracts.

The VIDEO_ENCODE [base list lifecycle and motion operations](video-motion-estimation.md#motion-recording) use these common creation, allocator-storage, threading, submission and completion contracts. The motion estimator's [cross-queue exclusion rule](video-motion-estimation.md#motion-driver-objects) is an additional object-specific requirement, not decoder recording serialization.

Video [GPU extension initialization/execution](video-extension-commands.md#extension-recording) records into the selected compatible video list and follows these common submission/storage/lifetime contracts. The distinct [device extension method](video-extension-commands.md#videodevice2-executeextensioncommand) has a published completion-before-return contract; its parameter stages are not GPU execution stages.

Encoding's [object and recording contract](video-encoding.md#encoding-recording) includes encoder/heap serialization and recorded execution order, creation-description copies distinct from GPU storage, caller-owned COM references without assumed recording AddRef, and lifetime through unsubmitted recordings that will execute and completion of submitted uses. Reconfiguration does not mutate immutable creation properties or supply codec-header/DPB policy.

Meta-command discovery, resource-parameter layouts and initialization/execution are defined in the subordinate [Meta commands reference](meta-commands.md).

<a id="recording-lifecycle"></a>

## Recording lifecycle

<a id="create-command-allocator"></a>

### ID3D12Device::CreateCommandAllocator

#### Declaration

```cpp
HRESULT CreateCommandAllocator(
    D3D12_COMMAND_LIST_TYPE type,
    REFIID riid,
    void** ppCommandAllocator
);
```

#### Behavior

Creates a command allocator object.

The allocator supplies storage for recordings of the selected command-list
type; creation does not begin a command-list recording. The same allocator
may store several recordings, but only one command list may be recording to
it at a time. [Allocator storage and reuse](#command-storage), the
[Reset preconditions](#allocator-reset) and [CPU threading](#threading)
apply to this object.

#### Parameters

`type` specifies the command-list type for the allocator: DIRECT, BUNDLE,
COMPUTE, COPY, VIDEO_DECODE, VIDEO_PROCESS or VIDEO_ENCODE. The type must be
supported by the device; video types require the corresponding video support.
The allocator type must match the type of the command list that records to it.

`riid` identifies the requested interface on the command allocator, normally
`ID3D12CommandAllocator`.

`ppCommandAllocator` is a required output pointer that receives the requested
interface on the created allocator.

#### Return values

Successful object creation returns `S_OK`. An invalid or unsupported
command-list type returns `E_INVALIDARG`.

This method returns `E_OUTOFMEMORY` if there is insufficient memory to create the command allocator.

Other creation/interface failures are reported through the returned `HRESULT`.

<a id="command-list-get-type"></a>

### ID3D12CommandList::GetType

#### Declaration

```cpp
D3D12_COMMAND_LIST_TYPE GetType();
```

#### Behavior

Gets the type of the command list, such as direct, bundle, compute, or copy.

#### Return values

This method returns the type of the command list,
as a `D3D12_COMMAND_LIST_TYPE` enumeration constant,
such as direct, bundle, compute, or copy.

<a id="creation"></a>

### ID3D12Device::CreateCommandList

#### Declaration

```cpp
HRESULT CreateCommandList(
    UINT nodeMask,
    D3D12_COMMAND_LIST_TYPE type,
    ID3D12CommandAllocator* pCommandAllocator,
    ID3D12PipelineState* pInitialState,
    REFIID riid,
    void** ppCommandList
);
```

#### Behavior

This API creates a command list or a bundle. The command list starts out
in the "recording" state.

This API also takes a command allocator as input. A given allocator can
be associated no more than 1 recording command list at a time. The
allocator type must match the type of command
list which is being created.

### ID3D12Device4::CreateCommandList1

#### Declaration

```cpp
HRESULT CreateCommandList1(
    UINT nodeMask,
    D3D12_COMMAND_LIST_TYPE type,
    D3D12_COMMAND_LIST_FLAGS flags,
    REFIID riid,
    void** ppCommandList
);
```

#### Behavior

`CreateCommandList1` creates a command list in the closed state. Also see [ID3D12Device::CreateCommandList](#creation).

<a id="lifecycle"></a>

## Closing and reusing command lists

This section covers DIRECT, COMPUTE, COPY, and BUNDLE recordings made through `ID3D12GraphicsCommandList`, and the corresponding `ID3D12CommandAllocator` objects. The three methods below belong to the baseline D3D12 interfaces; they do not require an optional feature query. Calls on the same command-list or allocator object must obey the [CPU threading requirements](#threading).

<a id="command-storage"></a>

### Recorded commands and allocator storage

You can record multiple lists to the same allocator before it is reset, provided only one list is recording to a given allocator at one time. You can visualise each list as owning a portion of the allocator which indicates what [`ID3D12CommandQueue::ExecuteCommandLists`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists) will execute.

Resetting a command-list object does not reset command-allocator storage. After resetting and re-recording a command-list object, the new recorded command stream may be submitted while an older stream recorded through the same object executes, provided that allocator storage and referenced resources remain valid for both streams. An unchanged recorded command stream must not be resubmitted while an earlier execution of that stream is still in progress.

`ExecuteBundle` records an execution of the bundle recording selected at that call. Resetting and re-recording the bundle object later does not replace the commands already selected by an earlier `ExecuteBundle` in a parent recording. The application may begin a new bundle recording with the same allocator without resetting that allocator, or use another compatible allocator.

The allocator storage backing the earlier recording must remain valid for every parent recording that will execute it. Resetting the allocator invalidates the recordings stored there, including bundle recordings already referenced by unsubmitted parents. The application must finish all GPU uses before reclaiming that storage and must not subsequently submit or resubmit an invalidated recording. An unsubmitted parent may instead be discarded without executing it.

Selecting a recorded command stream does not snapshot the contents of resources that its commands access, or remove their lifetime, residency, descriptor, and synchronization requirements. Neither closing a list nor resetting a list object or allocator provides proof of GPU completion.

Use a fence to determine when a given allocator is able to be reused.

<a id="close"></a>

### ID3D12GraphicsCommandList::Close

#### Declaration

```cpp
HRESULT Close();
```

#### Behavior

This API is called to indicate that recording to a command list has
finished.

`Close` ends CPU recording; it does not submit the recorded commands to a queue or wait for GPU execution. Queries begun in the command list must be ended before it is closed. An active render pass must also be ended before closing the list. These unmatched query/render-pass cases record an `E_FAIL` error against the list.

#### Return values

Returns `S_OK` if successful; otherwise, returns one of the following values:

- `E_INVALIDARG`: An invalid argument was passed to the command list API
  during recording

- `E_FAIL`: The command list has already been closed, an invalid API was
  called during command list recording, or a query or render pass is still open

- `E_OUTOFMEMORY`: The system ran out of memory during recording or closing

- `DXGI_ERROR_DEVICE_REMOVED`: A device-removing driver error was stored as the command-list error.

For a driver-reported device-removal failure, the application can query [ID3D12Device::GetDeviceRemovedReason](diagnostics.md#id3d12device-getdeviceremovedreason) for the device's removal reason.

#### Runtime validation

The runtime will validate that the command list has not previously been
closed. If an error was encountered during recording, the error code is
returned here. The runtime will not call the close DDI in this case.

#### Debug layer

Validation failures can generate debug-layer errors. A failed `Close`, including a driver-reported failure, does not necessarily produce a corresponding debug-layer message.

<a id="list-reset"></a>

### ID3D12GraphicsCommandList::Reset

#### Declaration

```cpp
HRESULT Reset(
    ID3D12CommandAllocator* pAllocator,
    ID3D12PipelineState* pInitialState
);
```

#### Behavior

This API is called to reset a command list back to its initial state (as
if a new command list was just created). This enables command list
tracking structures to be re-used.

After `Reset` succeeds, the command list is left in the "recording" state.

Applications **are allowed** to call this method before the
GPU has finished all previous executions of the command list (unlike
`ID3D12CommandAllocator::Reset`).

This method works for both command lists and bundles.

#### Parameters

**pAllocator**

A pointer to the [`ID3D12CommandAllocator`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12commandallocator) object that the device creates command lists from.

This API also takes a command allocator as input. A given allocator can
be associated with no more than 1 recording command list at a time. The
allocator type must match the type of command
list that is being reset.

**pInitialState**

A pointer to the [`ID3D12PipelineState`](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12pipelinestate) object that contains the initial pipeline state for the command list. This is optional and can be NULL. If NULL, the runtime sets a dummy initial pipeline state so that drivers don't have to deal with undefined state.

On a direct, compute, or bundle command list that uses graphics or compute pipeline state, `pInitialState` supplies the initial pipeline state. If it is NULL, the runtime selects its default graphics state for direct lists and bundles, or its default compute state for compute lists. This default does not supply an application shader program; the application must establish the pipeline state required by a draw or dispatch. Copy command lists have no pipeline state and do not use `pInitialState`.

<a id="reset-initial-state"></a>

#### Initial recording state

Creation/Reset values and bundle entry/exit behavior are defined in [Command-list state initialization](command-list-state.md#state-index). This includes the depth-bounds, sample-position, view-mask, VRS, dynamic-PSO, protection, and recording-context contributions.

#### Return values

Returns `S_OK` if successful; otherwise, returns one of the following values:

- `E_FAIL`: The command list was not in the "closed" state when the call
  was made.

- `E_INVALIDARG`: The allocator is currently being used with another
  command list in the "recording" state

- `E_INVALIDARG`: The specified allocator was created with the wrong type

- `E_OUTOFMEMORY`: The system ran out of memory

For a closed command list with a recorded error, `Reset` returns that recorded error; see [Runtime validation](#list-reset-validation) and [Deferred recording errors](#lifecycle-driver-errors).

<a id="list-reset-validation"></a>

#### Runtime validation

Before calling `Reset`, the command list must be in the "closed" state.
The API call will fail if this is not the case. Note that if an error has been recorded against the command list when
`ID3D12GraphicsCommandList::Close` returns, then that command list can never be
reset. Calling `ID3D12GraphicsCommandList::Reset` will result in the recorded error
being returned.

Applications must specify a command list allocator. The runtime will
ensure that an allocator is never associated with more than one
recording command list at the same time.

Calling `Close` on a command list that is already closed returns `E_FAIL` but does not, by itself, prevent a subsequent `Reset`.

<a id="allocator-reset"></a>

### ID3D12CommandAllocator::Reset

#### Declaration

```cpp
HRESULT Reset();
```

#### Behavior

Your app calls `Reset` to re-use the memory that is associated with a command allocator. From this call to `Reset`, the runtime and driver assume that the graphics processing unit (GPU) is no longer executing any command lists that have recorded commands with the command allocator. So you should ensure that you don't call `Reset` until the GPU is done executing command lists associated with the allocator.

It's undefined behavior to call **Reset** on a command allocator while it has a command list still being executed.

#### Return values

Returns `S_OK` on success. The active-recording failure is described below. Callers must check the returned HRESULT; success is not proof of GPU completion.

This method returns `E_FAIL` if there is an actively recording command list referencing the command allocator. The debug layer will also issue an error in this case. See [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

#### Debug layer

The debug layer will issue an error if it can't prove that there are no pending GPU references to command lists that have recorded commands in the allocator.

The debug layer will issue an error if `Reset` is called concurrently by multiple threads (on the same allocator object).

<a id="lifecycle-driver"></a>

### Driver responsibilities and error reporting

The driver must preserve previously recorded command streams and bundle references across a command-list-object Reset while their allocator storage remains valid. A driver may copy recorded commands or reference their storage, but resetting the bundle object must not retarget an existing parent recording to the bundle's new commands. Allocator reset, not list-object reset, permits reclamation of the affected command storage under the application preconditions described above.

For DDI versions using the `_0040` reset arguments, the runtime supplies a command-recorder handle associated with the allocator's command pool. Earlier reset arguments supply a command-allocator handle. The initial pipeline-state object is not an argument to either reset structure: the runtime initializes pipeline and other applicable state with state-setting DDIs. This API behavior does not require an application to negotiate a DDI version.

Closing a list invokes the close-list DDI when no previously recorded error prevents it. Resetting an allocator invokes the corresponding pool/allocator storage-reset DDI; it is not a request to wait for GPU completion. The subordinate [driver command-recording objects reference](command-recording-objects.md#object-model) defines allocator/pool/recorder/list associations, recording-generation IDs, private storage and versioned creation/reset/destruction interfaces. Its [failed-recording rules](command-recording-objects.md#failed-recording) define logical closure, recorder reuse and cleanup without guaranteed Close-DDI delivery. [Runtime UMD initialization](driver-initialization.md) owns capability and table negotiation; [command-list state](command-list-state.md) owns pipeline and other initial/reset state.

<a id="lifecycle-driver-errors"></a>

#### Deferred recording errors

`SetCommandListErrorCb` is used for errors encountered while executing
command list recording DDIs. There are only 3 errors which drivers
should pass to this function:

- `E_OUTOFMEMORY`
- `D3DDDIERR_DEVICEREMOVED`
- `D3DDDIERR_APPLICATIONERROR`

After the `SetCommandListErrorCb` DDI is called, the runtime will drop all
calls into the driver which record commands on the specified command
list.

A successful command-list Reset starts a new recording, but errors reported through the command-list error callback while initializing that recording are stored for Close to report. Reset's HRESULT does not replace the requirement to check Close.

Driver-reported errors are translated before the resulting command-list error is stored. A driver-reported `E_OUTOFMEMORY` remains `E_OUTOFMEMORY` unless the device has already been removed. Driver-reported device removal or application error removes the device and is translated to `DXGI_ERROR_DEVICE_REMOVED`; the more specific device reason is available from [GetDeviceRemovedReason](diagnostics.md#id3d12device-getdeviceremovedreason). If an earlier command-list error has already been recorded, it remains the error returned by Close and later Reset. Device removal does not replace that earlier stored error.

<a id="submission"></a>

## Submission and ordering

The submission and completion rules below apply to all supporting D3D12 command-queue types, including video decode, process, and encode. The core queue/fence methods are declared by `ID3D12CommandQueue`, `ID3D12Device`, and `ID3D12Fence`. `SetEventOnMultipleFenceCompletion` is declared by `ID3D12Device1`; `GetCreationFlags` is declared by `ID3D12Fence1`.

<a id="execute-command-lists"></a>

### ID3D12CommandQueue::ExecuteCommandLists

#### Declaration

```cpp
void ExecuteCommandLists(
    UINT NumCommandLists,
    ID3D12CommandList * const * ppCommandLists
);
```

#### Behavior

`ExecuteCommandLists` submits an array of command lists for execution.

#### Parameters

**NumCommandLists**

The number of command lists to be executed.

**ppCommandLists**

The array of [ID3D12CommandList](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12commandlist) command lists to be executed.

#### Ordering and recording lifetime

Calling `ExecuteCommandLists` twice in succession on the same queue (from the same thread, or different threads) guarantees that the first workload (A) finishes before the second workload (B). Calling `ExecuteCommandLists` with *two* command lists allows the driver to merge the two command lists such that the second command list (D) may begin executing work before all work from the first (C) has finished. Specifically, your application is allowed to insert a fence signal or wait between A and B, and the driver has no visibility into this, so the driver must ensure that everything in A is complete before the fence operation. There is no such opportunity in a single call to the API, so the driver is able to optimize that scenario.

The driver is free to patch the submitted command lists. It is the calling application’s responsibility to ensure that the graphics processing unit (GPU) is not currently reading any of the submitted recorded command streams from a previous execution.

Applications are encouraged to batch together command list executions to reduce fixed costs associated with submitted commands to the GPU.

The no-concurrent-reexecution rule concerns a particular recorded stream, not every recording ever made through the list object. Reset/re-recording may produce a new stream while an older stream executes, with valid backing storage and resources; see [recorded commands and allocator storage](#command-storage). Returning from `ExecuteCommandLists` does not establish GPU completion or make allocator/resource storage reusable.

#### Return values

`ExecuteCommandLists` returns no value. Submission failures are not returned as an `HRESULT` from this method.

<a id="validation"></a>

#### Runtime validation

Bundles can't be submitted to a command queue directly. If a bundle is passed to this method, the runtime will drop the call. The runtime will also drop the call if the [Close](#close) function has not successfully closed one or more of the command lists.

The runtime will detect if the command allocators associated with the command lists have been reset after [Close](#close) was called. The runtime will drop the call and remove the device in this situation.

Submitted lists must match the queue's command-list type and have compatible node masks. A list created initially closed by `CreateCommandList1` must first begin a recording with `Reset` and successfully close it; an object that has never begun a recording is not executable. An empty but successfully closed recording is distinct from an object that has never begun recording.

Runtime-detected submission failures drop the submission and remove the device. This includes invalid recorded/closed state, allocator-generation invalidation, and type/node incompatibility. It is not a guarantee that every invalid application action is detected by the retail runtime.

#### Debug layer

Debug validation checks previous execution of the recorded stream, deferred resource-transition and query requirements, and tracked resource lifetimes. Applications must meet those usage requirements whether or not validation is enabled; they must not depend on universal retail detection or device removal for every barrier/query or overlapping-execution error.

The debug layer issues an error if it detects that any resource referenced by the command lists, including queries, has been destroyed.

<a id="submission-threading"></a>

### Submission threading

Applications can submit command lists to any command queue from multiple threads. The runtime will perform the work of serializing these requests in the order of submission.

Serialization is per queue; it does not establish dependencies between independent queues. Use [queue signals](#queue-signal) and [queue waits](#queue-waits) to express inter-queue dependencies. If CPU threads require a particular submission order, synchronize those threads rather than inferring an order from unsynchronized concurrent calls.

<a id="submission-barriers"></a>

### Submission boundaries and resource barriers

For successive submissions on the same queue, D3D12 requires a full finish and flush of all GPU work at `ExecuteCommandLists` completion. Therefore, any writes to a resource during a preceding `ExecuteCommandLists` scope will be fully completed when that resource is needed in a subsequent `ExecuteCommandLists` scope.  However, drivers are not expected to keep track of when a resource was last written to.  So in addition to sync, a transition from a WRITE state may force a cache flush, even if the resource is already up-to-date from an earlier cache flush.

#### Legacy resource-state rules

Legacy resource-state behavior at submission boundaries is defined in [Common-state promotion and decay](synchronization.md#legacy-promotion-and-decay).

#### Enhanced layouts and cross-queue access

Texture layout transitions and queue/layout compatibility are defined in [Synchronization and barriers](synchronization.md#layout-transitions).

Fence ordering does not select a resource's legacy state or change a texture's enhanced layout. Apply the relevant [legacy multi-queue state rules](https://learn.microsoft.com/en-us/windows/win32/direct3d12/user-mode-heap-synchronization#multi-queue-resource-access), [enhanced queue-layout compatibility](synchronization.md#queue-compatibility), [access compatibility](synchronization.md#queue-compatibility), and [copy-queue rules](synchronization.md#queue-compatibility). These requirements are distinct from waiting for a producer's queue signal.

Suspended/resumed render passes must be paired within one `ExecuteCommandLists` group; see [render-pass recording and execution context](command-list-state.md#render-pass). A list boundary inside a batch is not a new submission boundary. Fine-grained [fence barriers](fence-barriers.md) synchronize selected barrier work within a submission.

<a id="submission-driver"></a>

### Runtime and driver submission responsibilities

The user-mode driver's `PFND3D12DDI_EXECUTECOMMANDLISTS` function executes an array of command lists submitted by an application.

`PFND3D12DDI_EXECUTECOMMANDLISTS` receives the driver's queue handle, a count, and an array of driver command-list handles. The application API submits interface pointers; the driver entry point does not receive that application pointer array. Driver execution must preserve the submission-boundary ordering and completion/flush requirements above.

Queue `Signal`/`Wait` operations preserve the queue's ordering contract even when runtime, operating-system, and driver synchronization transports differ. Do not infer a universal one-to-one fence-DDI call sequence from the API signatures. [Recorded storage and driver error reporting](#lifecycle-driver) remain applicable.

<a id="completion"></a>

## Completion and fences

<a id="fence-operations"></a>

### Fence values and timelines

The application-controlled timelines below use writable fences. System-controlled [protected-session status fences](#status-fences) are read-only and do not represent application-work completion. [Command-list-scoped fences](#command-list-scoped-fences) instead use the scoped barrier model, not this ordinary queue timeline.

The multi-engine API provides explicit APIs to create and synchronize using fences. A fence is a synchronization construct controlled by a `UINT64` value. Fence values are set by the application. A signal operation modifies the fence value and a wait operation blocks until the fence has reached the requested value or greater. An event can be fired when a fence reaches a certain value.

The [`Signal`](#fence-signal) method on the [`ID3D12Fence`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12fence) interface updates a fence from the CPU side. This update occurs immediately. The [`Signal`](#queue-signal) method on [`ID3D12CommandQueue`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12commandqueue) updates a fence from the GPU side. This update occurs after all other operations on the command queue have been completed.

A fence *may* be *rewound*. This means that the fence value does not need to solely increment. If a `Signal` operation is enqueued on two different command queues, or if two CPU threads are both calling `Signal` on a fence, there may be a race to determine which `Signal` completes last, and therefore which fence value is the one which will remain. If a fence is rewound, any new waits (including `SetEventOnCompletion` requests) will be compared against the new lower fence value, and therefore may not be satisfied, even if the fence value had previously been high enough to satisfy them. If a race does occur, between a value which will satisfy an outstanding wait, and a lower value which will not, the wait *will* be satisfied regardless of which value remains afterwards.

The fence APIs provide powerful synchronization functionality but can create potentially difficult to debug issues. It is recommended that each fence only be used to indicate progress on one timeline to prevent races between signalers.

A queued wait blocks subsequent work on that queue, not the calling CPU thread. The operation that can satisfy it must be able to run: placing the only satisfying signal behind an unsatisfied wait on the same queue does not make progress.

<a id="status-fences"></a>

### Protected-session status fences

The D3D12 protected resource session has a status fence whose fence value is incremented whenever protected content becomes invalid.  Status may be pulled from the protected session.  This typically happens when the system enters a sleep state and wakes back up, but could also be triggered by tamper protection from driver or hardware.

[`ID3D12ProtectedSession::GetStatusFence`](../ProtectedResources.md#method-id3d12protectedsessiongetstatusfence) returns the fence for the protected session. From the fence, we can get
the current uniqueness validity value (using `GetCompletedValue`), and add
monitors for change of its value. This is a read-only fence.

Read-only status fences must not be signaled through `ID3D12Fence::Signal`, `ID3D12CommandQueue::Signal`, or `ID3D12Device3::EnqueueMakeResident`. Their value changes indicate protected-content invalidation, not completion of submitted application work. These writability and value-meaning rules are separate from the [monitoring-capability restrictions](#fence-flags).

<a id="fence-flags"></a>

### Fence options and CPU observation

```cpp
typedef enum D3D12_FENCE_FLAGS
{
    D3D12_FENCE_FLAG_NONE = 0x0,
    D3D12_FENCE_FLAG_SHARED = 0x1,
    D3D12_FENCE_FLAG_SHARED_CROSS_ADAPTER = 0x2,
    D3D12_FENCE_FLAG_NON_MONITORED = 0x4,
    D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS = 0x8,
    D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC = 0x10,
} D3D12_FENCE_FLAGS;
```

<a id="base-fence-options"></a>

#### Base fence options

| Flag | Value | Meaning |
|---|---|---|
| `D3D12_FENCE_FLAG_NONE` | `0` | No options are specified. |
| `D3D12_FENCE_FLAG_SHARED` | `0x1` | The fence is shared. |
| `D3D12_FENCE_FLAG_SHARED_CROSS_ADAPTER` | `0x2` | The fence is shared with another GPU adapter. |
| `D3D12_FENCE_FLAG_NON_MONITORED` | `0x4` | The fence is of the non-monitored type. Non-monitored fences should only be used when the adapter doesn't support monitored fences, or when a fence is shared with an adapter that doesn't support monitored fences. |

<a id="fence-barrier-options"></a>

#### Fence-barrier options

<a id="allow-command-list-barriers"></a>

##### D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS

`D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` requests barrier support; the actual fence must report that flag to be eligible for command-list `SignalBarrier` or `WaitBarrier`.
Fences created with `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` set but without `D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC` bit also set are referred to as [Command List Scoped Fences](#command-list-scoped-fences).

<a id="allow-queue-and-cpu-sync"></a>

##### D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC

This flag is necessary to support GPU command-buffer-level fence synchronization between multiple queues or between the GPU and CPU.

`D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC` must be combined with `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS`.

If an app doesn't require queue or CPU sync, then it is recommended not setting this flag as it may significantly reduce barrier performance.

All fences created with `D3D12_FENCE_FLAG_SHARED` or `D3D12_FENCE_FLAG_SHARED_CROSS_ADAPTER` require Queue and CPU sync. As such, the `D3D12_FENCE_FLAG_SHARED` and `D3D12_FENCE_FLAG_SHARED_CROSS_ADAPTER` bits cannot be used when `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` is set without also setting `D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC`.

Fence-barrier use requires `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` explicitly. For a shared or cross-adapter fence, also request `D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC`. Sharing alone does not supply barrier eligibility. Creation can fail when the device/platform cannot supply the required GPU-accessible fence; a flag-only `S_FALSE` probe does not test that construction path.

<a id="fence-observation-requirements"></a>

#### Creation and observation requirements

`D3D12_FENCE_FLAG_SHARED_CROSS_ADAPTER` requires `D3D12_FENCE_FLAG_SHARED`. `D3D12_FENCE_FLAG_NON_MONITORED` creation requires both sharing flags and adapter support for non-monitored fences. Unsupported flag combinations fail with `E_INVALIDARG`.

Adapter fence support is described by [`DXGI_ADAPTER_DESC3::Flags`](https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_6/ns-dxgi1_6-dxgi_adapter_desc3), including `DXGI_ADAPTER_FLAG3_SUPPORT_MONITORED_FENCES` and `DXGI_ADAPTER_FLAG3_SUPPORT_NON_MONITORED_FENCES`. When `IDXGIAdapter4` is available, obtain the description through [`IDXGIAdapter4::GetDesc3`](https://learn.microsoft.com/en-us/windows/win32/api/dxgi1_6/nf-dxgi1_6-idxgiadapter4-getdesc3).

The complete [shared-fence export/open and handle contract](sharing-and-interoperability.md#sharing-synchronization) defines how participating devices and processes obtain the same fence. Exporting or opening a handle does not signal or wait for the fence, supply command-list-barrier eligibility or relax these creation and CPU-observation requirements.

Non-monitored fences are used for GPU-side synchronization; they do not provide the CPU progress-observation operations of monitored fences. `ID3D12Fence::Signal` and `ID3D12Fence::SetEventOnCompletion` reject a non-monitored fence with `E_INVALIDARG`. `GetCompletedValue` is invalid for a non-monitored fence and returns `UINT64_MAX`; that value is not a completed-work measurement. Multi-fence CPU notification/wait also requires CPU-wait-capable fences, not non-monitored fences. `GetCreationFlags` is a property query, not a progress query, and is not prohibited by this restriction. Non-monitored instances do not report `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` and are not eligible for command-list fence barriers.

[`ID3D12Device3::EnqueueMakeResident`](resources.md#enqueue-make-resident) is another fence-value producer. Its eligibility and completion rules are defined with residency. Its signal indicates completion of residency work, not completion of the application's GPU accesses to those resources.

<a id="command-list-scoped-fences"></a>

### Command-list-scoped fences

Command List Scoped fences are fences created with the `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` bit set but without the `D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC` bit set. Command List Scoped fences are usable with `ID3D12Fence::Signal` and command list `SignalBarrier` and `WaitBarrier` methods, but cannot support `ID3D12CommandQueue::Wait`, `ID3D12CommandQueue::Signal`, or `ID3D12Fence::SetEventOnCompletion`. Command List Scoped fences are designed to efficiently manage barrier dependencies within a single command list or a set of command lists in the same `ExecuteCommandLists` scope. The primary use for Command List Scoped fences is to perform efficient split barrier operations.

`WaitBarrier` values using Command List Scoped fences must already be satisfied either by the most recent preceding `SignalBarrier` operation in the same command queue or by `ID3D12Fence::Signal` called prior to GPU timeline execution of the submitted command list containing the `WaitBarrier`.

`ID3D12Fence::GetCompletedValue` always returns UINT64_MAX for Command List Scoped fences. Signaling a Command List Scoped fence currently in use in an in-execution command list by other command list, CPU or queue is invalid.

Unlike Monitored fences, Command List Scoped fences are not internally signaled to UINT64_MAX upon device removal. However, `ID3D12Fence::GetCompletedValue` for Command List Scoped fences still returns UINT64_MAX while a device is in a removed state.

A CPU `Signal` initializes the scoped fence value before GPU use; it must not race an in-flight scoped-fence use. The scalar `GetCompletedValue` result is always `UINT64_MAX`, not a progress measurement. Multi-fence CPU notification also requires CPU-wait-capable fences and cannot observe command-list-scoped fence completion. Creation/property queries do not turn this mode into an ordinary CPU/queue timeline.

<a id="create-fence"></a>

### ID3D12Device::CreateFence

#### Declaration

```cpp
HRESULT CreateFence(
    UINT64 InitialValue,
    D3D12_FENCE_FLAGS Flags,
    REFIID riid,
    void** ppFence
);
```

#### Behavior

`CreateFence` creates a fence object.

#### Parameters

**InitialValue**

The initial value for the fence.

**Flags**

A combination of [D3D12_FENCE_FLAGS](#fence-flags)-typed values that are combined by using a bitwise OR operation. The resulting value specifies options for the fence.

**riid**

The globally unique identifier (**GUID**) for the fence interface ([ID3D12Fence](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12fence)). The **REFIID**, or **GUID**, of the interface to the fence can be obtained by using the `__uuidof()` macro. For example, `__uuidof(ID3D12Fence)` will get the **GUID** of the interface to a fence.

**ppFence**

A pointer to a memory block that receives a pointer to the [ID3D12Fence](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12fence) interface that is used to access the fence.

`ppFence` may be `NULL` to check the fence flags without creating a fence object. This path returns `S_FALSE` when flag validation succeeds; it does not establish that object allocation, GPU-accessible backing, or the requested interface would succeed.

#### Return values

Returns `S_OK` if a fence is created; failures return one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/desktop/direct3d12/d3d12-graphics-reference-returnvalues).

`S_FALSE` is the no-object validation result described above, not a returned fence interface.

<a id="fence-creation-flags"></a>

### ID3D12Fence1::GetCreationFlags

#### Declaration

```cpp
D3D12_FENCE_FLAGS GetCreationFlags();
```

#### Return values

`GetCreationFlags` returns the flags used to create the fence.

The result describes the fence's effective creation properties, not a guarantee that every requested option survived construction. In particular, a non-monitored instance does not report `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS`; requesting the bit alone does not make that object barrier eligible. Use the actual properties and capability rules, not the requested mask, to determine eligibility.

#### Remarks

The flags returned by `GetCreationFlags` are used mainly for opening a shared fence.

<a id="queue-signal"></a>

### ID3D12CommandQueue::Signal

#### Declaration

```cpp
HRESULT Signal(
    ID3D12Fence* pFence,
    UINT64 Value
);
```

#### Behavior

Use this method to set a fence value from the GPU side. Use [ID3D12Fence::Signal](#fence-signal) to set a fence from the CPU side.

The target must be writable; [read-only status fences](#status-fences) cannot be signaled. Command-list-scoped fences cannot be used with queue `Signal`.

#### Parameters

**pFence**

A pointer to the [ID3D12Fence](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12fence) object.

**Value**

The value to set the fence to.

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues).

A successful return queues the signal; it does not wait for the GPU to reach it. The completion relationship is defined in [Fence values and timelines](#fence-operations).

<a id="queue-waits"></a>

### ID3D12CommandQueue::Wait

#### Declaration

```cpp
HRESULT Wait(
    ID3D12Fence* pFence,
    UINT64 Value
);
```

#### Behavior

Because a wait is being queued, the API returns immediately. It's the command queue that waits (during which time no work is executed) until the fence specified reaches the requested value.

If you want to perform a CPU-side wait (where the calling thread blocks until a fence reaches a particular value), then you should use the [`ID3D12Fence::SetEventOnCompletion`](#id3d12fenceseteventoncompletion) API in conjunction with [`WaitForSingleObject`](https://learn.microsoft.com/en-us/windows/win32/api/synchapi/nf-synchapi-waitforsingleobject) (or a similar API).

#### Parameters

**pFence**

A pointer to the [ID3D12Fence](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nn-d3d12-id3d12fence) object.

**Value**

The value that the command queue is waiting for the fence to reach or exceed.

The threshold refers to the fence's synchronization value, not to a sentinel from an invalid CPU query. See [CPU-observation restrictions](#fence-flags) and [device-removal interpretation](#completion-lifetime). Command-list-scoped fences cannot be used with queue `Wait`.

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues).

<a id="fence-signal"></a>

### ID3D12Fence::Signal

#### Declaration

```cpp
HRESULT Signal(
    UINT64 Value
);
```

#### Behavior

Use this method to set a fence value from the CPU side. Use [ID3D12CommandQueue::Signal](#queue-signal) to set a fence from the GPU side.

CPU signaling does not wait for preceding GPU work. A CPU-written value is therefore not evidence that unrelated queued work has completed. This method must not be used on [read-only status fences](#status-fences). For command-list-scoped fences, CPU signaling is only an initialization before the relevant GPU use, not an in-flight signaling operation.

#### Parameters

**Value**

The value to set the fence to.

#### Return values

This method returns one of the [Direct3D 12 Return Codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues).

<a id="id3d12fencegetcompletedvalue"></a>

### ID3D12Fence::GetCompletedValue

#### Declaration

```cpp
UINT64 GetCompletedValue();
```

<a id="fence-completed-value"></a>

#### Return values

Returns the current value of the fence. If the device has been removed, the return value will be `UINT64_MAX`.

The CPU-observation restrictions in [Fence options](#fence-flags) apply. An observed [status-fence counter](#status-fences) tracks protected-content validity changes, not GPU-work completion. For device-removal interpretation, see [Completion, device removal, and lifetime](#completion-lifetime). Command-list-scoped fences also return a constant sentinel, as defined [above](#command-list-scoped-fences).

<a id="id3d12fenceseteventoncompletion"></a>

### ID3D12Fence::SetEventOnCompletion

#### Declaration

```cpp
HRESULT SetEventOnCompletion(
    UINT64 Value,
    HANDLE hEvent
);
```

#### Behavior

`SetEventOnCompletion` specifies an event that's raised when the fence reaches a certain value.

If `hEvent` is a null handle, then this API will not return until the specified fence value has been reached.

This method can be safely called from multiple threads at one time.

#### Parameters

**Value**

The fence value when the event is to be signaled.

**hEvent**

A handle to the event object.

#### Return values

This method returns `E_OUTOFMEMORY` if the kernel components don’t have sufficient memory to store the event in a list. See [Direct3D 12 return codes](https://learn.microsoft.com/en-us/windows/win32/direct3d12/d3d12-graphics-reference-returnvalues) for other possible return values.

With a non-NULL event, successful registration is not itself a CPU wait for completion. With `hEvent == NULL`, the blocking behavior describes a successful wait; an error can return without establishing completion. Device removal can also satisfy a notification, as described [below](#completion-lifetime). For system-controlled status fences, the [status interpretation](#status-fences) also applies. [Command-list-scoped fences](#command-list-scoped-fences) do not support this notification.

<a id="multiple-fence-completion"></a>

### ID3D12Device1::SetEventOnMultipleFenceCompletion

#### Declaration

```cpp
HRESULT SetEventOnMultipleFenceCompletion(
    ID3D12Fence* const* ppFences,
    const UINT64* pFenceValues,
    UINT NumFences,
    D3D12_MULTIPLE_FENCE_WAIT_FLAGS Flags,
    HANDLE hEvent
);
```

#### Behavior

`SetEventOnMultipleFenceCompletion` specifies an event that should be fired when one or more of a collection of fences reach specific values.

#### Parameters

**ppFences**

An array of length `NumFences` that specifies the [ID3D12Fence](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nn-d3d12-id3d12fence) objects.

**pFenceValues**

An array of length `NumFences` that specifies the fence values required for the event to be signaled.

**NumFences**

Specifies the number of fences to be included.

**Flags**

Specifies one of the [D3D12_MULTIPLE_FENCE_WAIT_FLAGS](#multiple-fence-flags) that determines how to proceed.

**hEvent**

A handle to the event object.

`pFenceValues[i]` is the threshold for `ppFences[i]`; both arrays contain `NumFences` entries. The arrays describe the registration and are consumed by the call, not retained as application memory for later GPU use.

<a id="multiple-fence-flags"></a>

#### Wait selection

```cpp
typedef enum D3D12_MULTIPLE_FENCE_WAIT_FLAGS
{
    D3D12_MULTIPLE_FENCE_WAIT_FLAG_NONE = 0x0,
    D3D12_MULTIPLE_FENCE_WAIT_FLAG_ANY = 0x1,

    D3D12_MULTIPLE_FENCE_WAIT_FLAG_ALL = 0x0, // Alias, default behavior is ALL
} D3D12_MULTIPLE_FENCE_WAIT_FLAGS;
```

| Flag | Value | Meaning |
|---|---|---|
| `D3D12_MULTIPLE_FENCE_WAIT_FLAG_NONE` | `0` | No flags are being passed. This means to use the default behavior, which is to wait for all fences before signaling the event. |
| `D3D12_MULTIPLE_FENCE_WAIT_FLAG_ANY` | `0x1` | Modifies behavior to indicate that the event should be signaled after any one of the fence values has been reached by its corresponding fence. |
| `D3D12_MULTIPLE_FENCE_WAIT_FLAG_ALL` | `0` | An alias for `D3D12_MULTIPLE_FENCE_WAIT_FLAG_NONE`, meaning to use the default behavior and wait for all fences. |

#### Blocking and return values

If `hEvent` is a null handle, then this API will not return until the selected ANY/ALL fence condition has been satisfied.

This method returns an HRESULT success or error code.

Unrecognized `Flags` bits return `E_INVALIDARG`. Storage allocation can fail with `E_OUTOFMEMORY`. Successful non-NULL registration and blocking NULL-event use have the same completion/error distinction as [single-fence notification](#id3d12fenceseteventoncompletion). Read-only status-fence inputs retain their [status interpretation](#status-fences). Command-list-scoped fences are not CPU-wait-capable inputs.

<a id="completion-lifetime"></a>

### Completion, device removal, and lifetime

The full [`ID3D12Device::GetDeviceRemovedReason` contract](diagnostics.md#id3d12device-getdeviceremovedreason) defines device-removal status. To be called back when a device is removed, consider using [ID3D12Fence::SetEventOnCompletion](#id3d12fenceseteventoncompletion) with a value of `UINT64_MAX`. That's because device removal causes CPU-wait-capable fences to be signaled to that value (which also implies completing all events waited on, because their thresholds are at most `UINT64_MAX`).

An event becoming signaled or a fence reading `UINT64_MAX` is not by itself proof of successful GPU execution. Check device status rather than interpreting removal-induced notification as normally completed rendering or valid output data.

A completion value is useful for storage reuse only when its signal follows the relevant work and competing CPU/queue signals cannot satisfy the same threshold prematurely. Apply the [allocator storage](#command-storage) and [resource lifetime](#lifetime) requirements; neither API return nor list Reset is a completion proof.

`hEvent` names an application-supplied Windows event, not an event created by these D3D12 methods. Event-handle ownership and CPU waits follow the [Windows event-object contract](https://learn.microsoft.com/en-us/windows/win32/sync/event-objects). These notifications do not retain unrelated resources referenced by recorded GPU work.

<a id="lifetime"></a>

## Object and resource lifetime

### Reference counting

Most D3D12 APIs continue to use reference counting as per COM
conventions. 1 notable exception to this is the D3D12 command list APIs.
All APIs on ID3D12GraphicsCommandList do not hold references to the objects
passed into those APIs. This means applications are responsible for
ensuring that a command list is never submitted for execution that
references a destroyed resource.

### Resource lifetime separated from binding

Before freeing any resource, such as a texture, applications must
make sure the GPU has completed referencing it.

Whole-command-list completion is sufficient, but is not required when a correctly scoped fence-barrier signal proves every GPU use of the resource has completed and no later use remains. This does not permit resetting allocator storage while any recorded stream still uses it.

### CPU/GPU access to mapped memory

Applications have the responsibility to synchronize CPU and GPU memory accesses. To
help with this the system provides mechanisms for the application to
request sleeping of a CPU thread until work completes. Polling could
also be done, but can be wasteful.

Resources on CPU-accessible heaps can be persistently mapped, meaning **Map** can be called once, immediately after resource creation. [Unmap](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12resource-unmap) never needs to be called, but the address returned from **Map** must no longer be used after the last reference to the resource is released. When using persistent map, the application must ensure the CPU finishes writing data into memory before the GPU executes a command list that reads or writes the memory. In common scenarios, the application merely must write to memory before calling [ExecuteCommandLists](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/nf-d3d12-id3d12commandqueue-executecommandlists); but using a fence to delay command list execution works as well.
All CPU-accessible memory types support persistent mapping usage, where the resource is mapped but then never unmapped, provided the application does not access the pointer after the resource has been disposed.

### Command-list object destruction

Note that applications are allowed to call ID3D12GraphicsCommandList::Reset
before previous executions of the associated command list have
completed. Additionally, applications can destroy command lists before
previous executions have completed.

<a id="threading"></a>

## CPU threading

All methods on the device interface are free-threaded.

Methods on a specific command list object are not free-threaded with
respect to each other. Applications must ensure that they never
concurrently call methods on the same command list.

Methods on a command allocator are not free-threaded with respect to
each other. Applications must ensure that they never concurrently call
methods on the same command list allocator.

All methods on the command queue are free-threaded.

The D3D12 threading behavior is not configurable.

<a id="driver-responsibilities"></a>

## Driver responsibilities

### Threading

Drivers are expected to implement the threading model described in this
document. The one exception is the atomicity of command queue operation.
The D3D12 runtime acquires a lock around all command queue APIs.
Additionally, the runtime acquires the command queue lock around the
DXGI Present and Blt DDIs. Drivers can thus assume that the following
DDI calls are serialized:

- ExecuteCommandLists
- [UpdateTileMappings](resources.md#update-tile-mappings)
- [CopyTileMappings](resources.md#copy-tile-mappings)

### Command-list resource lifetime

The runtime implementation of command lists does not hold references
objects (resources, pipeline state objects, queries, etc) used by a
command list. Applications must guarantee that they do not call
ID3D12CommandQueue::ExecuteCommandLists if any resource
referenced by the command list has been destroyed. This however does not
guarantee the order of destruction. Drivers must be robust to the case
where a command list is destroyed after the resources that it references
have been destroyed.
