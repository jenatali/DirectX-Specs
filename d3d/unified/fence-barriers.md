# Fence barriers

Fence barriers allow app developers to perform GPU command execution timeline synchronization between command queues or with the CPU. This improves both GPU/CPU and GPU/GPU concurrency, and gives developers an alternative to split barriers.

<a id="availability"></a>

## Availability

```cpp
typedef enum D3D12_FENCE_BARRIERS_TIER
{
    D3D12_FENCE_BARRIERS_TIER_NOT_SUPPORTED = 0,
    D3D12_FENCE_BARRIERS_TIER_1 = 1,
} D3D12_FENCE_BARRIERS_TIER;
```

```cpp
typedef struct D3D12_FEATURE_DATA_FENCE_BARRIERS
{
    UINT NodeIndex;
    D3D12_COMMAND_LIST_TYPE CommandListType;
    D3D12_FENCE_BARRIERS_TIER FenceBarriersTier;
} D3D12_FEATURE_DATA_FENCE_BARRIERS;
```

Set `NodeIndex` and `CommandListType` to the node and command-list type being used, then query `D3D12_FEATURE_FENCE_BARRIERS` with `CheckFeatureSupport`. A failed query does not advertise support. The rules below describe `D3D12_FENCE_BARRIERS_TIER_1`; `D3D12_FENCE_BARRIERS_TIER_NOT_SUPPORTED` provides no fence-barrier operations. The interface `ID3D12CommandListFenceBarriers` is obtained from the owning command-list object. Queue types include direct, compute, copy, and the supported video types; bundles do not support these recording operations.

`D3D12_FENCE_BARRIERS_TIER_1`:
- Supports command list `SignalBarrier` methods using fences created with `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` bit set.
- Supports command list `WaitBarrier` methods using fences created with `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS` bit set, except for fences created with `D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC` bit also set (see [Command List Scoped Fences](execution.md#command-list-scoped-fences)).
- Requires driver support for fence barriers
- Requires device support for 64-bit atomic ops. Must be able to write 64-bit fence values atomically to avoid torn values

<a id="barrier-operations"></a>

## Barrier operations and groups

Drivers typically handle legacy Resource Barriers using three separate operations:

1. Synchronize GPU work
2. Perform any necessary cache flush operations
3. Perform any necessary layout changes

The enhanced Barrier API's give developers the ability to control each of these operations separately.

Enhanced Barrier APIs provide three barrier types:

- Texture Barriers
- Buffer Barriers
- Global Barriers

### Texture Barriers

Texture Barriers control cache flush, memory layout and synchronization for texture subresources.  Texture Barriers must only be used with texture resources.  Texture barriers allow a selection of a single subresource, all subresources, or a coherent range of subresources (i.e. mip range and array range).  Texture barriers must provide a valid, non-NULL resource pointer.

### Buffer Barriers

Buffer Barriers control cache flush and synchronization for buffer resources.  Buffer Barriers must only be used with buffer resources.  Unlike textures, buffers have only a single subresource and do not have a transitionable layout.  Buffer barriers must provide a valid, non-NULL resource pointer.

`Offset` must be zero and `Size` must be either the buffer size in bytes or `UINT64_MAX`.  Note that enhanced barriers support concurrent read and write on buffers without the need for intervening barriers (see [Single-Queue Simultaneous Access](../D3D12EnhancedBarriers.md#single-queue-simultaneous-access)).

The enhanced buffer barrier DDIs do not use `Offset` or `Size`.

### Global Barriers

Global barriers control cache flush and synchronization for all indicated resource access types in a single command queue.  Global Barriers have no effect on texture layout.  Global Barriers are needed to provide functionality similar to legacy NULL UAV barriers and NULL/NULL aliasing barriers.

Since global barriers do not transition texture layout, global barriers may not be used in transitions that otherwise would require a layout change. For example, a global barrier cannot be used to transition a non-simultaneous-access texture from `D3D12_BARRIER_ACCESS_RENDER_TARGET` to `D3D12_BARRIER_ACCESS_SHADER_RESOURCE`, since that would also require a change from `D3D12_BARRIER_LAYOUT_RENDER_TARGET` to `D3D12_BARRIER_LAYOUT_SHADER_RESOURCE`.

<a id="barrier-groups"></a>

### Barrier group representation

```cpp
typedef enum D3D12_BARRIER_TYPE
{
    D3D12_BARRIER_TYPE_GLOBAL,
    D3D12_BARRIER_TYPE_TEXTURE,
    D3D12_BARRIER_TYPE_BUFFER,
} D3D12_BARRIER_TYPE;
```

#### D3D12_BARRIER_SUBRESOURCE_RANGE

```cpp
typedef struct D3D12_BARRIER_SUBRESOURCE_RANGE
{
    UINT IndexOrFirstMipLevel;
    UINT NumMipLevels;
    UINT FirstArraySlice;
    UINT NumArraySlices;
    UINT FirstPlane;
    UINT NumPlanes;
} D3D12_BARRIER_SUBRESOURCE_RANGE;
```

| Members                |                                                                                                                                                                       |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `IndexOrFirstMipLevel` | Subresource Index (If `NumMipLevels` is zero) or index of first mip level in the range. May be `0xffffffff` if used as subresource index to specify all subresources. |
| `NumMipLevels`         | Number of mip levels in the range, or zero to indicate `IndexOrFirstMipLevel` is a subresource index.                                                                 |
| `FirstArraySlice`      | Index of first array slice in the range. Ignored if `NumMipLevels` is zero.                                                                                           |
| `NumArraySlices`       | Number of array slices in the range. Ignored if `NumMipLevels` is zero.                                                                                               |
| `FirstPlane`           | First plane slice in the range.  Ignored if `NumMipLevels` is zero.                                                                                                   |
| `NumPlanes`            | Number of plane slices in the range.  Ignored if `NumMipLevels` is zero.                                                                                              |

#### D3D12_GLOBAL_BARRIER

```cpp
typedef struct D3D12_GLOBAL_BARRIER
{
    D3D12_BARRIER_SYNC SyncBefore;
    D3D12_BARRIER_SYNC SyncAfter;
    D3D12_BARRIER_ACCESS AccessBefore;
    D3D12_BARRIER_ACCESS AccessAfter;
} D3D12_GLOBAL_BARRIER;
```

| Member         |                                                                                                                                                                                                              |
|----------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `SyncBefore`   | Synchronization scope bits of all preceding GPU work that must be completed before executing the barrier. Can be a bitwise-or combination of multiple `D3D12_BARRIER_SYNC` bits.                             |
| `SyncAfter`    | Synchronization scope bits of all subsequent GPU work that must wait until the barrier execution is finished. Can be a bitwise-or combination of multiple `D3D12_BARRIER_SYNC` bits.                         |
| `AccessBefore` | Access bits corresponding with any relevant resource usage since the preceding barrier or the start of `ExecuteCommandLists` scope. Can be a bitwise-or combination of multiple `D3D12_BARRIER_ACCESS` bits. |
| `AccessAfter`  | Access bits corresponding with any relevant resource usage after the barrier completes. Can be a bitwise-or combination of multiple `D3D12_BARRIER_ACCESS` bits.                                             |

#### D3D12_TEXTURE_BARRIER_FLAGS

```cpp
typedef enum D3D12_TEXTURE_BARRIER_FLAGS
{
    D3D12_TEXTURE_BARRIER_FLAG_NONE = 0x0,
    D3D12_TEXTURE_BARRIER_FLAG_DISCARD = 0x1,
} D3D12_TEXTURE_BARRIER_FLAGS;
```

##### D3D12_TEXTURE_BARRIER_FLAG_DISCARD

`D3D12_TEXTURE_BARRIER_FLAG_DISCARD` can only be used when `LayoutBefore` is `D3D12_BARRIER_LAYOUT_UNDEFINED`.  Typically, this is used to initialize compression metadata as part of a barrier that activates an aliased resource.

When `D3D12_TEXTURE_BARRIER_FLAG_DISCARD` is specified, `LayoutAfter` may be any valid layout appropriate for the resource and barrier command list type.  This is more flexible than the legacy `DiscardResource` API, which requires subresources to be in `D3D12_RESOURCE_STATE_RENDER_TARGET` if the resource was created with `D3D12_RESOURCE_FLAG_ALLOW_RENDER_TARGET`, or `D3D12_RESOURCE_STATE_DEPTH_WRITE` if created with `D3D12_RESOURCE_FLAG_ALLOW_DEPTH_STENCIL`. As such, applications can discard-initialize a subresource directly into the layout needed for subsequent access, without barrier layout restriction.

#### D3D12_TEXTURE_BARRIER

```cpp
typedef struct D3D12_TEXTURE_BARRIER
{
    D3D12_BARRIER_SYNC SyncBefore;
    D3D12_BARRIER_SYNC SyncAfter;
    D3D12_BARRIER_ACCESS AccessBefore;
    D3D12_BARRIER_ACCESS AccessAfter;
    D3D12_BARRIER_LAYOUT LayoutBefore;
    D3D12_BARRIER_LAYOUT LayoutAfter;
    ID3D12Resource *pResource;
    D3D12_BARRIER_SUBRESOURCE_RANGE Subresources;
    D3D12_TEXTURE_BARRIER_FLAGS Flags;
} D3D12_TEXTURE_BARRIER;
```

| Member         |                                                                                                                                                                                                 |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `SyncBefore`   | Synchronization scope bits of all preceding GPU work that must be completed before executing the barrier. Can be a bitwise-or combination of multiple `D3D12_BARRIER_SYNC` bits.                |
| `SyncAfter`    | Synchronization scope bits of all subsequent GPU work that must wait until the barrier execution is finished. Can be a bitwise-or combination of multiple `D3D12_BARRIER_SYNC` bits.            |
| `AccessBefore` | Access bits corresponding with resource usage since the preceding barrier or the start of `ExecuteCommandLists` scope. Can be a bitwise-or combination of multiple `D3D12_BARRIER_ACCESS` bits. |
| `AccessAfter`  | Access bits corresponding with resource usage after the barrier completes. Can be a bitwise-or combination of multiple `D3D12_BARRIER_ACCESS` bits.                                             |
| `LayoutBefore` | Layout of texture preceding the barrier execution.                                                                                                                                              |
| `LayoutAfter`  | Layout of texture upon completion of barrier execution.                                                                                                                                         |
| `pResource`    | Pointer to the texture resource used by the barrier.                                                                                                                                         |
| `Subresources` | Range of texture subresources being barriered.                                                                                                                                                  |
| `Flags`        | Optional flags values.                                                                                                                                                                          |

#### D3D12_BUFFER_BARRIER

```cpp
typedef struct D3D12_BUFFER_BARRIER
{
    D3D12_BARRIER_SYNC SyncBefore;
    D3D12_BARRIER_SYNC SyncAfter;
    D3D12_BARRIER_ACCESS AccessBefore;
    D3D12_BARRIER_ACCESS AccessAfter;
    ID3D12Resource *pResource;
    UINT64 Offset;
    UINT64 Size;
} D3D12_BUFFER_BARRIER;
```

| Member         |                                                                                                                                                                                                 |
|----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `SyncBefore`   | Synchronization scope bits of all preceding GPU work that must be completed before executing the barrier. Can be a bitwise-or combination of multiple `D3D12_BARRIER_SYNC` bits.                |
| `SyncAfter`    | Synchronization scope bits of all subsequent GPU work that must wait until the barrier execution is finished. Can be a bitwise-or combination of multiple `D3D12_BARRIER_SYNC` bits.            |
| `AccessBefore` | Access bits corresponding with resource usage since the preceding barrier or the start of `ExecuteCommandLists` scope. Can be a bitwise-or combination of multiple `D3D12_BARRIER_ACCESS` bits. |
| `AccessAfter`  | Access bits corresponding with resource usage after the barrier completes. Can be a bitwise-or combination of multiple `D3D12_BARRIER_ACCESS` bits.                                             |
| `pResource`    | Pointer to the buffer resource used by the barrier.                                                                                                                                         |
| `Offset`       | Offset value must be `0`.                                                                                                                                                                       |
| `Size`         | Size must either be `UINT64_MAX` or the size of the buffer in bytes.                                                                                                                            |

#### D3D12_BARRIER_GROUP

```cpp
typedef struct D3D12_BARRIER_GROUP
{
    D3D12_BARRIER_TYPE Type;
    UINT32 NumBarriers;
    union
    {
        const D3D12_GLOBAL_BARRIER *pGlobalBarriers;
        const D3D12_TEXTURE_BARRIER *pTextureBarriers;
        const D3D12_BUFFER_BARRIER *pBufferBarriers;
    };
} D3D12_BARRIER_GROUP;
```

`Type` selects the corresponding pointer in the union, and `NumBarriers` is the number of barriers in that array. `NumBarrierGroups` counts the outer array passed to the commands. Global barriers express memory-access/synchronization scopes; buffer barriers add a buffer/range, and texture barriers add texture subresources and layout transitions.

<a id="sync-access"></a>

### Synchronization and access

The enhanced Barrier API's use explicit `SyncBefore` and `SyncAfter` values as bitfield masks that can describe one or more combined synchronization scopes.  A Barrier must wait for all preceding command `SyncBefore` scopes to complete before executing the barrier.  Similarly, a Barrier must block all subsequent `SyncAfter` scopes until the barrier completes.

Since many GPU-write operations are cached, any Barrier from a write access to another write access or a read-only access may require a cache flush.  The enhanced Barrier API's use access transitions to indicate that a subresource's memory needs to be made visible for a specific new access type.  Like the layout transitions, some access transitions may not be needed if it is known that the memory of the associated subresource is already accessible for the desired use.

Barrier `AccessBefore` and/or `AccessAfter` values may be a bitwise-or combination of `D3D12_BARRIER_ACCESS` bits when more than one access type applies before or after the barrier.

Access transitions for textures are expressed as part of the `D3D12_TEXTURE_BARRIER` structure data.  Access transitions for buffers are expressed using the `D3D12_BUFFER_BARRIER` structure.

Access transitions do not perform synchronization.  It is expected that synchronization between dependent accesses is handled using appropriate `SyncBefore` and `SyncAfter` values in the barrier.

An `AccessBefore` made visible to a specified `AccessAfter` DOES NOT guarantee that the resource memory is also visible for a *different* access type.  For example:

```c++
MyTexBarrier.AccessBefore=D3D12_BARRIER_ACCESS_UNORDERED_ACCESS;
MyTexBarrier.AccessAfter=D3D12_BARRIER_ACCESS_SHADER_RESOURCE|D3D12_BARRIER_ACCESS_COPY_SOURCE;
```

This access transition indicates that subsequent shader-resource and copy-source accesses depend on a preceding unordered-access-write. However, this may not actually flush the UAV cache if the hardware is capable of reading shader-resource and copy-source data directly from the UAV cache.

`D3D12_BARRIER_ACCESS_COMMON` is a special access type that indicates any layout-compatible access. Transitioning to `D3D12_BARRIER_ACCESS_COMMON` means that subresource data must be available for any layout-compatible access after a barrier. Since buffers have no layout, `D3D12_BARRIER_ACCESS_COMMON` simply means any buffer-compatible access.

Specifying `D3D12_BARRIER_ACCESS_COMMON` as `AccessBefore` in a barrier implies the set of all write-access types. Using `D3D12_BARRIER_ACCESS_COMMON` as `AccessBefore` is strongly discouraged since this could result in costly, unintended cache flushes. Instead, developers are encouraged to use only the most narrowly required write-access bits to properly constrain barrier overhead. A debug layer warning is issued when `AccessBefore` is set to `D3D12_BARRIER_ACCESS_COMMON`.

<a id="fence-kinds"></a>

## Fence kinds and dependency scope

Fence creation flags, [command-list-scoped fence rules](execution.md#command-list-scoped-fences), and [CPU/queue observation restrictions](execution.md#fence-flags) are defined in the common execution reference. Sharing does not implicitly enable command-list barriers: the barrier flag must be requested explicitly. A fence must actually support GPU barrier access; ordinary queue synchronization capability alone is not sufficient.

<a id="access-and-producer-order"></a>

### Subsequent accesses and producer ordering

`SignalBarrier` is still a normal barrier operation, and the transitioning subresource can still be accessed in the `ExecuteCommandLists` scope based on the `AccessAfter` bits used in the barrier; This includes write operations so long as the resource is simultaneous-access and no external dependent accesses write to the resource. However, consecutive layout transitioning `SignalBarrier` operations with no intervening `WaitBarrier` on a given texture subresource in a single `ExecuteCommandLists` scope is invalid since it risks racing with external dependent accesses.

Consecutive layout transitioning `WaitBarrier` operations with no intervening `SignalBarrier` on a given texture subresource in a single `ExecuteCommandLists` scope is invalid. Such a scenario would imply concurrent write operations to the same subresource, which is not supported.

`SignalBarrier` operations on the same fence require an ordered dependency chain: barrier synchronization must ensure the preceding signal completes before the next signal is reached. Resolve its outstanding synchronization scopes with the current or intervening barriers; an unresolved split signal requires its dependent wait. A scoped wait must be satisfied by its most recent applicable producer, as defined in the [scoped-fence rules](execution.md#command-list-scoped-fences).

<a id="split-dependencies"></a>

### Local split dependencies

To initiate a split barrier using a Command List Scoped fence, a `SignalBarrier` operation specifies any layout transition and access after bits, setting `SyncAfter` to `D3D12_BARRIER_SYNC_NONE` since the barrier is not expected to block subsequent command processing other than a dependent `WaitBarrier`. The values of `AccessBefore` and `AccessAfter` reflect the before and after accesses upon completion of the dependent `WaitBarrier`.

The split barrier is terminated by a dependent `WaitBarrier`, satisfied when the initiating `SignalBarrier` completes texture layout transition and cache flush. The terminating `WaitBarrier` typically sets `SyncBefore` to `D3D12_BARRIER_SYNC_NONE` since it only needs to sync with the fence condition. For texture barriers, `LayoutBefore` and `LayoutAfter` are set to `D3D12_BARRIER_LAYOUT_UNDEFINED` since all the layout transitions are completed by the time the fence is signaled. The values of `AccessBefore` and `AccessAfter` must match those of the initiating `SignalBarrier` to properly invalidate any relevant caches.

Fence barriers using Command List Scoped fence are the only way to split global barriers. Classic split barriers require drivers to set aside storage in resource memory to track completion state. Thus, global barriers cannot be split using `D3D12_BARRIER_SYNC_SPLIT` since no resource is provided.

<a id="external-dependencies"></a>

### External dependencies

Resource caches on the GPU are required to be coherent at the start of `ExecuteCommandLists` workloads, and completion of `ExecuteCommandLists` guarantees that all caches are flushed to main memory. However, with fence barriers it is possible to represent dependencies that do not benefit from this coherency guarantee.

It is expected that all caches are coherent across all COMPUTE and DIRECT queues on the same device. However, CPU/GPU dependencies or cross-queue dependencies involving other queue types such as COPY and VIDEO queues may require full cache flush or invalidation. As such, apps must use the `D3D12_BARRIER_ACCESS_GLOBAL` access bit in fence barriers to manage cache coherency for these external dependencies.

Unnecessary cache flushes and cache invalidation can be costly. Cross-queue dependencies specifically between COMPUTE and DIRECT queues on the same device should avoid using the `D3D12_BARRIER_ACCESS_GLOBAL` bit to avoid unneeded overhead.

A `SignalBarrier` with [external dependencies](#external-dependencies) must set the `D3D12_BARRIER_ACCESS_GLOBAL` bit in `AccessAfter` to guarantee data coherency.

A `WaitBarrier` with [external dependencies](#external-dependencies) must set the `D3D12_BARRIER_ACCESS_GLOBAL` bit in `AccessBefore` to guarantee data coherency.

<a id="global-access-legality"></a>

#### GLOBAL access legality

`D3D12_BARRIER_ACCESS_GLOBAL` may be combined with other access bits.

`D3D12_BARRIER_ACCESS_GLOBAL` cannot be used in non-fence barriers, nor can `D3D12_BARRIER_ACCESS_GLOBAL` be used in `AccessBefore` in a `SignalBarrier` barrier or in `AccessAfter` in a `WaitBarrier` barrier.

<a id="no-access-exception"></a>

#### NO_ACCESS exception

Resource is either not accessed before/after the barrier in the same `ExecuteCommandLists` context, or the data is no longer needed. `D3D12_BARRIER_ACCESS_NO_ACCESS` may not be combined with other access bits.

`D3D12_BARRIER_ACCESS_NO_ACCESS` may be used when synchronizing between CPU and GPU access if there are no writes to flush or caches do not need invalidating. This is typically the case with write-after-read barriers.

A Tier-1 signal barrier may export completed barrier work to a CPU observer or another queue through a queue/CPU-capable fence. The consumer uses ordinary CPU notification or queue `Wait`; Tier-1 `WaitBarrier` uses only command-list-scoped fences. The global-access requirement does not itself create an execution dependency or change a texture's layout.

<a id="commands"></a>

## Commands

<a id="signal-barrier"></a>

### ID3D12CommandListFenceBarriers::SignalBarrier

#### Declaration

```cpp
void SignalBarrier(
    UINT NumBarrierGroups,
    const D3D12_BARRIER_GROUP *pBarrierGroups,
    ID3D12Fence *pFence,
    UINT64 FenceValue
);
```

#### Behavior

`SignalBarrier` executes the provided barriers, signalling a fence after all barrier transactions have fully completed. An outstanding local split-barrier dependency must be completed by a `WaitBarrier` with matching fence/value pairs before its dependent operations.

If a wait barrier on a texture subresource corresponds to a preceding signal barrier, the `LayoutBefore` on the wait barrier must match the `LayoutAfter` from the signal barrier or be set to `D3D12_BARRIER_LAYOUT_UNDEFINED`, which avoids a layout transition. This effectively matches split barrier behavior where a barrier transaction is started by the split-begin barrier, and completed by the split-end barrier.

#### Parameters

| Parameter          |                                                         |
|--------------------|---------------------------------------------------------|
| `NumBarrierGroups` | Number of barrier groups pointed to by `pBarrierGroups` |
| `pBarrierGroups`   | Pointer to an array of `D3D12_BARRIER_GROUP` objects    |
| `pFence`           | Fence to signal upon completion of barriers             |
| `FenceValue`       | Fence value to signal                                   |

#### Requirements

The target must be a writable fence explicitly enabled with `D3D12_FENCE_FLAG_ALLOW_COMMAND_LIST_BARRIERS`. [Read-only status fences](execution.md#status-fences) cannot be signaled. For CPU or other-queue dependencies, the fence also requires `D3D12_FENCE_FLAG_ALLOW_QUEUE_AND_CPU_SYNC` and the [external-dependency coherency rules](#external-dependencies).

#### Return values and validation

This method returns no value. It records GPU work; it is not a CPU wait for GPU completion. Recording failures are reported through `Close` on the owning graphics or video command-list interface, following [deferred recording error rules](execution.md#lifecycle-driver-errors). Debug validation also checks dependency/layout requirements during recording and submission; applications must obey them independently of diagnostic coverage.

<a id="wait-barrier"></a>

### ID3D12CommandListFenceBarriers::WaitBarrier

#### Declaration

```cpp
void WaitBarrier(
    UINT NumBarrierGroups,
    const D3D12_BARRIER_GROUP *pBarrierGroups,
    UINT NumFences,
    ID3D12Fence * const *ppFences,
    const UINT64 *pFenceValues
);
```

#### Behavior

`WaitBarrier` waits for the full set of fence/value pairs to be satisfied then executes the provided barrier operations.

#### Parameters

| Parameter          |                                                                   |
|--------------------|-------------------------------------------------------------------|
| `NumBarrierGroups` | Number of barrier groups pointed to by `pBarrierGroups`           |
| `pBarrierGroups`   | Pointer to an array of `D3D12_BARRIER_GROUP` objects              |
| `NumFences`        | Number of fence/value pairs                                       |
| `ppFences`         | Pointer to an array of `ID3D12Fence` pointers of size `NumFences` |
| `pFenceValues`     | Pointer to an array of fence values of size `NumFences`           |

#### Requirements

For `D3D12_FENCE_BARRIERS_TIER_1`, every input fence must be [command-list scoped](execution.md#command-list-scoped-fences). `pFenceValues[i]` is the threshold for `ppFences[i]`; the full set of conditions must be satisfied, not merely any one condition. The scoped-fence producer rules apply across the command lists in the same `ExecuteCommandLists` group.

#### Return values and validation

This method returns no value. It records GPU work; it is not a CPU wait for GPU completion. Recording failures are reported through `Close` on the owning graphics or video command-list interface, following [deferred recording error rules](execution.md#lifecycle-driver-errors). Debug validation also checks dependency/layout requirements during recording and submission; applications must obey them independently of diagnostic coverage.

<a id="lifetime"></a>

## Lifetime and completion

A signal proves completion only for the work/cache operations covered by its barrier scopes. It is not automatically a full-command-list or allocator-idle marker. Resource/storage reuse requires completion of every remaining use, and fence storage must remain valid while GPU references remain. Resetting a command-list object does not reset an independent fence or establish completion; see [resource lifetime](execution.md#lifetime) and [allocator storage](execution.md#command-storage).
