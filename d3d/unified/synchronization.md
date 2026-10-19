# Synchronization and barriers

<a id="legacy-resource-states"></a>

## Legacy resource-state model

At any given time, a [subresource](resources.md#subresources) is in exactly one state, determined by the set of [`D3D12_RESOURCE_STATES`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) flags.

The resource state usage bits that are used to describe a resource state are divided into read-only and read/write states. The reference topic for the [`D3D12_RESOURCE_STATES`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ne-d3d12-d3d12_resource_states) indicates the read/write access level for each bit in the enumeration.

At most, only one read/write bit can be set for any resource. If a write bit is set, then no read-only bit may be set for that resource. If no write bit is set, then any number of read bits may be set.

<a id="legacy-promotion-and-decay"></a>

## Common-state promotion and decay

Resources can only be "promoted" out of `D3D12_RESOURCE_STATE_COMMON`. Similarly, resources will only "decay" to `D3D12_RESOURCE_STATE_COMMON`.

<a id="legacy-common-state-promotion"></a>

### Common-state promotion

All buffer resources as well as textures with the `D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS` flag set are implicitly promoted from `D3D12_RESOURCE_STATE_COMMON` to the relevant state on first GPU access, including `GENERIC_READ` to cover any read scenario. Any resource in the `COMMON` state can be accessed as though it were in a single state with 1 WRITE flag, or 1 or more READ flags.

Resources can be promoted from the `COMMON` state based on the following table:

| State flag | Buffers and Simultaneous-Access Textures | Non-Simultaneous-Access Textures |
| --- | --- | --- |
| `VERTEX_AND_CONSTANT_BUFFER` | Yes | No |
| `INDEX_BUFFER` | Yes | No |
| `RENDER_TARGET` | Yes | No |
| `UNORDERED_ACCESS` | Yes | No |
| `DEPTH_WRITE` | No\* | No |
| `DEPTH_READ` | No\* | No |
| `NON_PIXEL_SHADER_RESOURCE` | Yes | Yes |
| `PIXEL_SHADER_RESOURCE` | Yes | Yes |
| `STREAM_OUT` | Yes | No |
| `INDIRECT_ARGUMENT` | Yes | No |
| `COPY_DEST` | Yes | Yes |
| `COPY_SOURCE` | Yes | Yes |
| `RESOLVE_DEST` | Yes | No |
| `RESOLVE_SOURCE` | Yes | No |
| `PREDICATION` | Yes | No |

\*Depth-stencil resources must be non-simultaneous-access textures and thus can never be implicitly promoted.

When this access occurs the promotion acts like an implicit resource barrier. For subsequent accesses, resource barriers will be required to change the resource state if necessary. Note that promotion from one promoted read state into multiple read state is valid, but this is not the case for write states. For example, if a resource in the common state is promoted to `PIXEL_SHADER_RESOURCE` in a Draw call, it can still be promoted to `NON_PIXEL_SHADER_RESOURCE` | `PIXEL_SHADER_RESOURCE` in another Draw call. However, if it is used in a write operation, such as a copy destination, a resource state transition barrier from the combined promoted read states, here `NON_PIXEL_SHADER_RESOURCE` | `PIXEL_SHADER_RESOURCE`, to `COPY_DEST` is needed. Similarly, if promoted from `COMMON` to `COPY_DEST`, a barrier is still required to transition from `COPY_DEST` to `RENDER_TARGET`.

Note that common state promotion is "free" in that there is no need for the GPU to perform any synchronization waits. The promotion represents the fact that resources in the `COMMON` state should not require additional GPU work or driver tracking to support certain accesses.

<a id="legacy-state-decay"></a>

### State decay to COMMON

The flip-side of common state promotion is decay back to `D3D12_RESOURCE_STATE_COMMON`. Resources that meet certain requirements are considered to be stateless and effectively return to the common state when the GPU finishes execution of an [`ExecuteCommandLists`](execution.md#execute-command-lists) operation. Decay does not occur between command lists executed together in the same `ExecuteCommandLists` call.

The following resources will decay when an [`ExecuteCommandLists`](execution.md#execute-command-lists) operation is completed on the GPU:

- Resources being accessed on a Copy queue, *or*
- Buffer resources on any queue type, *or*
- Texture resources on any queue type that have the `D3D12_RESOURCE_FLAG_ALLOW_SIMULTANEOUS_ACCESS` flag set, *or*
- Any resource implicitly promoted to a read-only state.

Like common state promotion, the decay is free in that no additional synchronization is needed. Combining common state promotion and decay can help to eliminate many unnecessary [`ResourceBarrier`](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/nf-d3d12-id3d12graphicscommandlist-resourcebarrier) transitions. In some cases, this can provide significant performance improvements.

Buffers and Simultaneous-Access resources will decay to the common state regardless of whether they were explicitly transitioned using resource barriers or implicitly promoted.
