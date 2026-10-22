# Unified Direct3D 12 specification

**Status: incomplete.** The chapters are bounded, source-preserving drafts.
Existing specifications remain the reference material for functionality not
yet consolidated.

This specification describes the Direct3D 12 API, device driver interface
(DDI), and hardware contract in chapterized Markdown. It incorporates
applicable hardware behavior from the
[Direct3D 11.3 functional specification](../archive/D3D11_3_FunctionalSpec.htm)
and integrates incremental feature specifications into an explicit description
of supported behavior.

## Scope

The specification describes shipped D3D12 functionality, including supported older
paths, capability-dependent behavior, and relevant compatibility history.

The supporting shader appendices will specify the complete DXBC shader-model 4.0, 4.1,
5.0, and 5.1 contract, not merely their incremental additions. Shader-model
availability and D3D12 applicability are distinct.

The DXIL specification and its shader-model 6.x contracts are also in scope as
a supporting appendix. Shared API, DDI, and hardware requirements have canonical
definitions in the main chapters, referenced by the shader appendices.
Main stage chapters describe overall behavior for both shader representations;
the DXIL appendix supplies the complete behavior reference for every intrinsic.

## Chapters

| Chapter | Current coverage |
|---|---|
| [Command recording and execution](execution.md) | Recording/lifecycle baseline; integrated core submission and fence contracts with flag/status, completion/lifetime and barrier interactions; bounded DDI coverage |
| [Command-list state initialization](command-list-state.md) | Canonical creation/Reset state, bundle entry/exit, and explicit feature/operation contributions for the pinned graphics interfaces |
| [Resources and memory](resources.md) | Resource/heap ownership and backing; descriptions, creation and allocation; memory budgets, residency, CPU mapping and driver responsibilities |
| [Synchronization and barriers](synchronization.md) | Common foundations; enhanced barriers first; legacy states/barriers and promotion/decay; model interoperation and driver translation |
| [Fence barriers](fence-barriers.md) | Barrier signals, scoped waits, capability and cache-coherency requirements, with common fence kinds defined in execution |
| [Resource binding and root signatures](binding.md) | Descriptor heaps/tables/handles, root arguments and signature versions, static/data promises, direct indexing, sampler extensions, ordinary APIs and selected driver contracts |
| [Formats and views](formats.md) | Format interpretation/casting; complete selected view descriptions; ordinary and feedback view creation; plane, support-query and selected driver contracts |
| [Pipeline state and shader linkage](pipeline-state.md) | Ordinary graphics/compute PSO descriptions and streams, supporting state, shader/root compatibility, cache blobs, selection and selected driver contracts |
| [Input assembly and vertex processing](vertex-processing.md) | IA/direct draws, fetching and primitive assembly, generated IDs, VS, shared interface capacities and view-instancing rules, and selected driver responsibilities |
| [Tessellation: hull and domain processing](tessellation.md) | DXIL/DXBC HS models, fixed-function patterns and factor processing, precision/watertightness, patch routing, DS and selected driver responsibilities |
| [Geometry shaders and stream output](geometry-shaders-and-stream-output.md) | Invocation/streams/emission and output budgets, DXBC/DXIL models, PSO capture declarations, SO target/counter binding, whole-primitive overflow, canonical query links and selected driver responsibilities |
| [Amplification and mesh shaders](amplification-and-mesh-shaders.md) | AS/MS groups, payloads, output/connectivity/attribute/storage rules, DXIL model, DispatchMesh, view/culling/order, canonical statistics links and selected driver responsibilities |
| [Rasterization](rasterization.md) | Clipping/transforms and view selection, snapping/culling/depth bias, primitive/sample coverage, conservative rasterization and programmable sample patterns; four full setters and selected driver contracts |
| [Pixel shaders](pixel-shaders.md) | Invocation/input/interpolation and barycentric models, derivatives/quads, outputs/early tests/helpers, ROV ordering and VRS; two full setters and selected driver callbacks |
| [Output merger](output-merger.md) | Depth/stencil/bounds tests and updates, blending/logic/output masks, seven attachment/dynamic/render-pass APIs, complete selected pass access/flow and published driver contracts |
| [Counters, queries, and predication](counters-queries-and-predication.md) | Query heaps/types/lifecycle and graphics/video APIs, canonical pipeline/SO statistics, GPU/CPU result resolution, queue clocks, buffer predicates and explicit current operation policies |
| [Compute execution](compute.md) | Thread groups and IDs, DXBC/DXIL models, per-group TGSM allocation/lifetime, waves and quad derivatives; full Dispatch API and shared grid limits |
| [Memory model](memory-model.md) | **Provisional; substantial revision required.** Relocated shader memory consistency, atomics/coherency, shader barriers and shared-access rules; not a complete general model |
| [Common shader internals](common-shader-internals.md) | Shared execution and DXBC/DXIL program/storage/resource models; texture addressing/filtering/LOD/anisotropy/comparison/gather, precision and shader-visible feedback; stage-specific groups/quads/helpers retain their owners |
