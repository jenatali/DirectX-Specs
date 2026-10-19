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

The shader reference will specify the complete DXBC shader-model 4.0, 4.1,
5.0, and 5.1 contract, not merely their incremental additions. Shader-model
availability and D3D12 applicability are distinct.

DXIL and shader-model 6.x contracts are external. Stable D3D12 API/DDI/hardware
features requiring SM6 remain in scope, with references to their external
shader contracts.

## Chapters

| Chapter | Current coverage |
|---|---|
| [Command recording and execution](execution.md) | Recording/lifecycle baseline; integrated core submission and fence contracts with flag/status, completion/lifetime and barrier interactions; bounded DDI coverage |
| [Command-list state initialization](command-list-state.md) | Canonical creation/Reset state, bundle entry/exit, and explicit feature/operation contributions for the pinned graphics interfaces |
| [Resources and memory](resources.md) | Resource/heap ownership and backing; descriptions, creation and allocation; memory budgets, residency, CPU mapping and driver responsibilities |
| [Synchronization and barriers](synchronization.md) | Common foundations; enhanced barriers first; legacy states/barriers and promotion/decay; model interoperation and driver translation |
| [Fence barriers](fence-barriers.md) | Barrier signals, scoped waits, capability and cache-coherency requirements, with common fence kinds defined in execution |
