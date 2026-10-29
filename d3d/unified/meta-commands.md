<a id="meta-commands"></a>

# Meta commands

A meta command is a Direct3D 12 object representing an algorithm that is accelerated by independent hardware vendors (IHVs). It's an opaque reference to a command generator that is implemented by the driver.

A meta command can encapsulate a set of pipeline state objects (PSOs), bindings, intermediate resource states, and Draw/Dispatch calls. You can think of the signature of a meta command as being similar to a C-style function, with multiple in/out parameters, and no return value.

`ID3D12Device5` declares meta-command enumeration, parameter enumeration and creation. `ID3D12MetaCommand` derives from `ID3D12Pageable`, then `ID3D12DeviceChild`, `ID3D12Object` and `IUnknown`; it declares `GetRequiredParameterResourceSize`. `ID3D12GraphicsCommandList4` declares initialization and execution. Inherited methods retain their declaring identities.

An enumerated `Id` identifies an available algorithm, not a feature-level guarantee or a portable algorithm implementation. The algorithm's definition supplies the meanings of its parameters. Generic meta-command reflection does not standardize an algorithm catalog. The optional algorithm-specific [meta-command capability query](devices-and-capabilities.md#d3d12_feature_data_query_meta_command) has its own input/output payload contract.

The runtime validates the meta-command GUID against an approved list. This check can only be bypassed in developer mode. Outside developer mode, only predefined meta-commands are allowed.

<a id="parameter-layout"></a>

## Reflection and parameter layout

Creation parameters describe the instance created on the CPU. Initialization and execution parameters are supplied to their respective recorded operations. Reflection is selected separately for each stage. `pTotalStructureSizeInBytes` describes the parameter-value block, not the array of parameter descriptors and not the size of a referenced GPU resource.

<a id="d3d12_meta_command_parameter_stage"></a>

### D3D12_META_COMMAND_PARAMETER_STAGE

```cpp
typedef enum D3D12_META_COMMAND_PARAMETER_STAGE
{
    D3D12_META_COMMAND_PARAMETER_STAGE_CREATION = 0,
    D3D12_META_COMMAND_PARAMETER_STAGE_INITIALIZATION = 1,
    D3D12_META_COMMAND_PARAMETER_STAGE_EXECUTION = 2,
} D3D12_META_COMMAND_PARAMETER_STAGE;
```

| Constant | Meaning |
| --- | --- |
| `D3D12_META_COMMAND_PARAMETER_STAGE_CREATION` | Specifies that the parameter is used at the meta command creation stage. |
| `D3D12_META_COMMAND_PARAMETER_STAGE_INITIALIZATION` | Specifies that the parameter is used at the meta command initialization stage. |
| `D3D12_META_COMMAND_PARAMETER_STAGE_EXECUTION` | Specifies that the parameter is used at the meta command execution stage. |

<a id="d3d12_meta_command_parameter_type"></a>

### D3D12_META_COMMAND_PARAMETER_TYPE

```cpp
typedef enum D3D12_META_COMMAND_PARAMETER_TYPE
{
    D3D12_META_COMMAND_PARAMETER_TYPE_FLOAT = 0,
    D3D12_META_COMMAND_PARAMETER_TYPE_UINT64 = 1,
    D3D12_META_COMMAND_PARAMETER_TYPE_GPU_VIRTUAL_ADDRESS = 2,
    D3D12_META_COMMAND_PARAMETER_TYPE_CPU_DESCRIPTOR_HANDLE_HEAP_TYPE_CBV_SRV_UAV = 3,
    D3D12_META_COMMAND_PARAMETER_TYPE_GPU_DESCRIPTOR_HANDLE_HEAP_TYPE_CBV_SRV_UAV = 4,
} D3D12_META_COMMAND_PARAMETER_TYPE;
```

| Constant | Meaning |
| --- | --- |
| `D3D12_META_COMMAND_PARAMETER_TYPE_FLOAT` | Specifies that the parameter is of type [FLOAT](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types). |
| `D3D12_META_COMMAND_PARAMETER_TYPE_UINT64` | Specifies that the parameter is of type [UINT64](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types). |
| `D3D12_META_COMMAND_PARAMETER_TYPE_GPU_VIRTUAL_ADDRESS` | Specifies that the parameter is a GPU virtual address. |
| `D3D12_META_COMMAND_PARAMETER_TYPE_CPU_DESCRIPTOR_HANDLE_HEAP_TYPE_CBV_SRV_UAV` | Specifies that the parameter is a CPU descriptor handle to a heap containing either constant buffer views, shader resource views, or unordered access views. |
| `D3D12_META_COMMAND_PARAMETER_TYPE_GPU_DESCRIPTOR_HANDLE_HEAP_TYPE_CBV_SRV_UAV` | Specifies that the parameter is a GPU descriptor handle to a heap containing either constant buffer views, shader resource views, or unordered access views. |

<a id="d3d12_meta_command_parameter_flags"></a>

### D3D12_META_COMMAND_PARAMETER_FLAGS

```cpp
typedef enum D3D12_META_COMMAND_PARAMETER_FLAGS
{
    D3D12_META_COMMAND_PARAMETER_FLAG_INPUT = 0x00000001,
    D3D12_META_COMMAND_PARAMETER_FLAG_OUTPUT = 0x00000002
} D3D12_META_COMMAND_PARAMETER_FLAGS;
```

Defines constants that specify the flags for a parameter to a meta command. Values can be bitwise OR'd together.

| Constant | Meaning |
| --- | --- |
| `D3D12_META_COMMAND_PARAMETER_FLAG_INPUT` | Specifies that the parameter is an input resource. |
| `D3D12_META_COMMAND_PARAMETER_FLAG_OUTPUT` | Specifies that the parameter is an output resource. |

<a id="d3d12_meta_command_parameter_desc"></a>

### D3D12_META_COMMAND_PARAMETER_DESC

```cpp
typedef struct D3D12_META_COMMAND_PARAMETER_DESC
{
    LPCWSTR Name;
    D3D12_META_COMMAND_PARAMETER_TYPE Type;
    D3D12_META_COMMAND_PARAMETER_FLAGS Flags;
    D3D12_RESOURCE_STATES RequiredResourceState;
    UINT StructureOffset;
} D3D12_META_COMMAND_PARAMETER_DESC;
```

`Name`

The parameter name.

`Type`

A [D3D12_META_COMMAND_PARAMETER_TYPE](#d3d12_meta_command_parameter_type) specifying the parameter type.

`Flags`

A [D3D12_META_COMMAND_PARAMETER_FLAGS](#d3d12_meta_command_parameter_flags) specifying the parameter flags.

`RequiredResourceState`

A [D3D12_RESOURCE_STATES](https://learn.microsoft.com/en-us/windows/desktop/api/d3d12/ne-d3d12-d3d12_resource_states) specifying the expected state of a resource parameter.

`StructureOffset`

The 4-byte aligned offset for this parameter, within the structure containing the parameter values, which you pass when creating/initializing/executing the meta command, as appropriate.

The driver reports the list of parameters and their type and the runtime calculates a 4-byte aligned offset in a structure that the user developer passes these parameters in.

The application must supply each stage's values at the returned `StructureOffset` positions in a block of the returned total byte size. The 4-byte offset alignment does not imply 8-byte alignment of a `UINT64`, GPU virtual address or descriptor-handle parameter. A native C/C++ structure's padding is not a replacement for the reflected layout.

A parameter's input/output flags describe resource usage; they do not make the stage's parameter block optional. A null parameter-block pointer is paired with a zero byte size. A stage with parameters requires its parameter-value block.

<a id="d3d12_meta_command_desc"></a>

### D3D12_META_COMMAND_DESC

```cpp
typedef struct D3D12_META_COMMAND_DESC
{
    GUID Id;
    LPCWSTR Name;
    D3D12_GRAPHICS_STATES InitializationDirtyState;
    D3D12_GRAPHICS_STATES ExecutionDirtyState;
} D3D12_META_COMMAND_DESC;
```

`Id`

A [GUID](https://learn.microsoft.com/en-us/windows/win32/api/guiddef/ns-guiddef-guid) uniquely identifying the meta command.

`Name`

The meta command name.

`InitializationDirtyState` and `ExecutionDirtyState` use [`D3D12_GRAPHICS_STATES`](#d3d12_graphics_states). Their operative state effects and subsequent restoration are defined in [command-list state: meta commands](command-list-state.md#meta-commands), including its unresolved all-dirty-bits/`ClearState` qualification. This reference does not assert that equivalence.

The application owns the descriptor-array storage supplied to enumeration. Both command-description `Name` strings and parameter-description `Name` strings are borrowed device-lifetime storage. The application must not free these strings; their lifetime is distinct from that of the caller's descriptor arrays.

<a id="d3d12_graphics_states"></a>

### D3D12_GRAPHICS_STATES

```cpp
typedef enum D3D12_GRAPHICS_STATES
{
    D3D12_GRAPHICS_STATE_NONE = 0,
    D3D12_GRAPHICS_STATE_IA_VERTEX_BUFFERS = (1 << 0),
    D3D12_GRAPHICS_STATE_IA_INDEX_BUFFER = (1 << 1),
    D3D12_GRAPHICS_STATE_IA_PRIMITIVE_TOPOLOGY = (1 << 2),
    D3D12_GRAPHICS_STATE_DESCRIPTOR_HEAP = (1 << 3),
    D3D12_GRAPHICS_STATE_GRAPHICS_ROOT_SIGNATURE = (1 << 4),
    D3D12_GRAPHICS_STATE_COMPUTE_ROOT_SIGNATURE = (1 << 5),
    D3D12_GRAPHICS_STATE_RS_VIEWPORTS = (1 << 6),
    D3D12_GRAPHICS_STATE_RS_SCISSOR_RECTS = (1 << 7),
    D3D12_GRAPHICS_STATE_PREDICATION = (1 << 8),
    D3D12_GRAPHICS_STATE_OM_RENDER_TARGETS = (1 << 9),
    D3D12_GRAPHICS_STATE_OM_STENCIL_REF = (1 << 10),
    D3D12_GRAPHICS_STATE_OM_BLEND_FACTOR = (1 << 11),
    D3D12_GRAPHICS_STATE_PIPELINE_STATE = (1 << 12),
    D3D12_GRAPHICS_STATE_SO_TARGETS = (1 << 13),
    D3D12_GRAPHICS_STATE_OM_DEPTH_BOUNDS = (1 << 14),
    D3D12_GRAPHICS_STATE_SAMPLE_POSITIONS = (1 << 15),
    D3D12_GRAPHICS_STATE_VIEW_INSTANCE_MASK = (1 << 16),

} D3D12_GRAPHICS_STATES;
```

Defines flags that specify states related to a graphics command list. Values can be bitwise OR'd together.

| Constant | Meaning |
| --- | --- |
| `D3D12_GRAPHICS_STATE_NONE` | Specifies no state. |
| `D3D12_GRAPHICS_STATE_IA_VERTEX_BUFFERS` | Specifies the state of the vertex buffer bindings on the input assembler stage. |
| `D3D12_GRAPHICS_STATE_IA_INDEX_BUFFER` | Specifies the state of the index buffer binding on the input assembler stage. |
| `D3D12_GRAPHICS_STATE_IA_PRIMITIVE_TOPOLOGY` | Specifies the state of the primitive topology value set on the input assembler stage. |
| `D3D12_GRAPHICS_STATE_DESCRIPTOR_HEAP` | Specifies the state of the currently bound descriptor heaps. |
| `D3D12_GRAPHICS_STATE_GRAPHICS_ROOT_SIGNATURE` | Specifies the state of the currently set graphics root signature. |
| `D3D12_GRAPHICS_STATE_COMPUTE_ROOT_SIGNATURE` | Specifies the state of the currently set compute root signature. |
| `D3D12_GRAPHICS_STATE_RS_VIEWPORTS` | Specifies the state of the viewports bound to the rasterizer stage. |
| `D3D12_GRAPHICS_STATE_RS_SCISSOR_RECTS` | Specifies the state of the scissor rectangles bound to the rasterizer stage. |
| `D3D12_GRAPHICS_STATE_PREDICATION` | Specifies the predicate state. |
| `D3D12_GRAPHICS_STATE_OM_RENDER_TARGETS` | Specifies the state of the render targets bound to the output merger stage. |
| `D3D12_GRAPHICS_STATE_OM_STENCIL_REF` | Specifies the state of the reference value for depth stencil tests set on the output merger stage. |
| `D3D12_GRAPHICS_STATE_OM_BLEND_FACTOR` | Specifies the state of the blend factor set on the output merger stage. |
| `D3D12_GRAPHICS_STATE_PIPELINE_STATE` | Specifies the state of the pipeline state object. |
| `D3D12_GRAPHICS_STATE_SO_TARGETS` | Specifies the state of the buffer views bound to the stream output stage. |
| `D3D12_GRAPHICS_STATE_OM_DEPTH_BOUNDS` | Specifies the state of the depth bounds set on the output merger stage. |
| `D3D12_GRAPHICS_STATE_SAMPLE_POSITIONS` | Specifies the state of the sample positions. |
| `D3D12_GRAPHICS_STATE_VIEW_INSTANCE_MASK` | Specifies the state of the view instances mask. |

The affected families are defined by [vertex/index and stream-output buffer bindings](command-list-state.md#buffer-bindings), [primitive topology, viewports, scissor rectangles and blend factor](command-list-state.md#base-graphics), [descriptor heaps and root signatures](command-list-state.md#resource-binding), [predication](command-list-state.md#queries), [render targets](command-list-state.md#output-bindings), [stencil reference](command-list-state.md#stencil-references), [depth bounds](command-list-state.md#depth-bounds), [pipeline state](command-list-state.md#pipeline-selection), [sample positions](command-list-state.md#sample-positions) and [view-instance mask](command-list-state.md#view-mask). This flag reference does not redefine their initial values.

<a id="discovery-and-creation"></a>

## Enumeration and creation

<a id="enumeratemetacommands"></a>

### ID3D12Device5::EnumerateMetaCommands

#### Declaration

```cpp
HRESULT EnumerateMetaCommands(
    UINT* pNumMetaCommands,
    D3D12_META_COMMAND_DESC* pDescs
);
```

#### Behavior

Queries reflection metadata about available meta commands.

For a valid query, on entry `*pNumMetaCommands` is the capacity of the caller's descriptor array. On return, it is the total number of available commands, not the number copied. With a null `pDescs`, the input capacity is ignored and the method returns the full count and `S_OK` without writing descriptors.

With a non-null `pDescs`, the method copies the first `min(capacity, total count)` descriptions. An insufficient capacity returns `DXGI_ERROR_MORE_DATA` after writing that prefix and the full count. A capacity at least as large as the total returns `S_OK`; an oversized capacity does not cause extra descriptors to be read or written. If no commands are available, the returned count is zero, no descriptors are written, and the method returns `S_OK`.

#### Parameters

`pNumMetaCommands`

A pointer to the array capacity on input and total available count on output. With a null *pDescs*, the input capacity is ignored.

`pDescs`

An optional pointer to an array of [D3D12_META_COMMAND_DESC](#d3d12_meta_command_desc) containing the descriptions of the available meta commands. Pass `nullptr` to have the number of available meta commands returned in *pNumMetaCommands*.

#### Return values

If this method succeeds, it returns **S\_OK**. Otherwise, it returns an [HRESULT](https://learn.microsoft.com/en-us/windows/win32/com/structure-of-com-error-codes) error code.

<a id="enumeratemetacommandparameters"></a>

### ID3D12Device5::EnumerateMetaCommandParameters

#### Declaration

```cpp
HRESULT EnumerateMetaCommandParameters(
    REFGUID CommandId,
    D3D12_META_COMMAND_PARAMETER_STAGE Stage,
    UINT* pTotalStructureSizeInBytes,
    UINT* pParameterCount,
    D3D12_META_COMMAND_PARAMETER_DESC* pParameterDescs
);
```

#### Behavior

Queries reflection metadata about the parameters of the specified meta command.

For a valid query, on entry `*pParameterCount` is the capacity of the caller's descriptor array. On return, it is the total number of parameters for `CommandId` and `Stage`, not the number copied. With a null `pParameterDescs`, the input capacity is ignored and the method returns the full count and `S_OK` without writing descriptors.

With a non-null `pParameterDescs`, the method copies the first `min(capacity, total count)` descriptions. An insufficient capacity returns `DXGI_ERROR_MORE_DATA` after writing that prefix and the full count. A capacity at least as large as the total returns `S_OK`; an oversized capacity does not cause extra descriptors to be read or written. For a valid stage with no parameters, the returned count is zero, no descriptors are written, and the method returns `S_OK`.

When supplied, `pTotalStructureSizeInBytes` receives the complete stage's parameter-value block size, including on a count query or `DXGI_ERROR_MORE_DATA` result. It is not shortened to match a descriptor prefix. A valid stage with no parameters returns zero through this output.

#### Parameters

`CommandId`

A reference to the globally unique identifier (GUID) of the meta command whose parameters you wish to be returned in *pParameterDescs*.

`Stage`

A [D3D12_META_COMMAND_PARAMETER_STAGE](#d3d12_meta_command_parameter_stage) specifying the stage of the parameters that you wish to be included in the query.

`pTotalStructureSizeInBytes`

An optional pointer to a [UINT](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types) containing the size of the structure containing the parameter values, which you pass when creating/initializing/executing the meta command, as appropriate.

`pParameterCount`

A pointer to the array capacity on input and total available count on output. With a null *pParameterDescs*, the input capacity is ignored.

`pParameterDescs`

An optional pointer to an array of [D3D12_META_COMMAND_PARAMETER_DESC](#d3d12_meta_command_parameter_desc) containing the descriptions of the parameters. Pass **nullptr** to have the parameter count returned in *pParameterCount*.

#### Return values

If this method succeeds, it returns **S\_OK**. Otherwise, it returns an **HRESULT** error code.

<a id="createmetacommand"></a>

### ID3D12Device5::CreateMetaCommand

#### Declaration

```cpp
HRESULT CreateMetaCommand(
    REFGUID CommandId,
    UINT NodeMask,
    const void * pCreationParametersData,
    SIZE_T CreationParametersDataSizeInBytes,
    REFIID riid,
    void** ppMetaCommand
);
```

#### Behavior

Creates an instance of the specified meta command.

#### Parameters

`CommandId`

A reference to the globally unique identifier (GUID) of the meta command that you wish to instantiate.

`NodeMask`

For single-adapter operation, set this to zero. If there are multiple adapter nodes, set a bit to identify the node (one of the device's physical adapters) to which the meta command applies. Each bit in the mask corresponds to a single node. Only one bit must be set. See [Multi-adapter systems](https://learn.microsoft.com/en-us/windows/win32/direct3d12/multi-engine).

`pCreationParametersData`

An optional pointer to a constant structure containing the values of the parameters for creating the meta command.

`CreationParametersDataSizeInBytes`

A [SIZE_T](https://learn.microsoft.com/en-us/windows/win32/WinProg/windows-data-types) containing the size of the structure pointed to by *pCreationParametersData*, if set, otherwise 0.

`riid`

A reference to the globally unique identifier (GUID) of the interface that you wish to be returned in *ppMetaCommand*. This is expected to be the GUID of [ID3D12MetaCommand](#meta-commands).

`ppMetaCommand`

A pointer to a memory block that receives a pointer to the meta command. This is the address of a pointer to an [ID3D12MetaCommand](#meta-commands), representing the meta command created.

#### Return values

If this method succeeds, it returns **S\_OK**. Otherwise, it returns an **HRESULT** error code.

| Return value | Description |
| --- | --- |
| `DXGI_ERROR_UNSUPPORTED` | The requested algorithm is unsupported, or a creation-parameter combination for a known algorithm is unsupported |

<a id="parameter-resources"></a>

## Parameter resources and recorded use

Initialization and execution are supported on direct and compute command lists. The application must not record either operation in a bundle or on a copy command list. This recording admission does not waive the selected algorithm's capability or resource requirements.

Creation returns a caller-owned COM interface reference. The application must keep the meta-command object alive for all of its recorded GPU uses under the [execution lifetime contract](execution.md#lifetime), including recorded uses not yet submitted. The absence of a currently executing command list does not by itself make release safe. Command-list Reset is not proof that earlier GPU uses have completed.

The application owns the CPU parameter-value blocks passed to creation, initialization and execution. It must keep each block's bytes valid through the call; after the call returns, that CPU storage may be reused or freed. GPU resources and descriptor storage identified by values in the block have their separate GPU-use lifetime; releasing the CPU block does not release or shorten those obligations.

Parameter-resource size and parameter-block size are independent: the resource-size query below is indexed by stage and parameter, not by byte offset. The application must keep resources and descriptor storage supplied to GPU operations valid for their use under [execution lifetime](execution.md#lifetime), [resource residency](resources.md#residency) and [binding lifetime](binding.md#table-indexing-and-descriptor-lifetime). The algorithm's initialization contract identifies any required persistent cache resource; an execution's temporary storage and input/output storage are resource parameters, not allocations implicitly made by the parameter-value block.

The expected state is the reflected `RequiredResourceState` and the algorithm's parameter definition. [Resource-state transitions and UAV ordering](synchronization.md) retain their canonical contract; passing a resource parameter does not perform an application-required transition.

Persistent-cache and temporary/scratch roles belong to the selected algorithm's parameter definition; the generic input/output flags do not distinguish those roles. Storage needed by later executions must remain valid for those uses. Repeated execution is permitted, but does not replace the required GPU initialization. Repeated initialization, persistent-storage replacement and scratch reuse follow the algorithm's definition and normal resource lifetime and ordering rules; the generic interface adds no extra reinitialization protocol. The generic interface does not supply an implicit allocation or a default resource binding when a parameter is omitted.

<a id="getrequiredparameterresourcesize"></a>

### ID3D12MetaCommand::GetRequiredParameterResourceSize

#### Declaration

```cpp
UINT64 GetRequiredParameterResourceSize(D3D12_META_COMMAND_PARAMETER_STAGE Stage, UINT ParameterIndex);
```

#### Behavior

Retrieves the amount of memory required for the specified runtime parameter resource for a meta command, for the specified stage.

#### Parameters

`Stage`

A **D3D12\_META\_COMMAND\_PARAMETER\_STAGE** specifying the stage to which the parameter belongs.

`ParameterIndex`

The zero-based index of the parameter within the stage.

#### Return values

The number of bytes required for the specified runtime parameter resource.

The application must supply a valid `Stage` and the enumerated ordinal of a resource parameter for this meta-command instance. `ParameterIndex` uses the stage's zero-based parameter order, not a resource-only filtered order. The query's result is a byte requirement for that parameter resource; it is not a parameter-count result, a resource-state transition or an initialization operation. No zero sentinel, HRESULT or other result guarantee is specified outside that valid input domain.

<a id="initializemetacommand"></a>

### ID3D12GraphicsCommandList4::InitializeMetaCommand

#### Declaration

```cpp
void InitializeMetaCommand(
    ID3D12MetaCommand * pMetaCommand,
    const void * pInitializationParametersData,
    SIZE_T InitializationParametersDataSizeInBytes
);
```

#### Behavior

You must initialize a meta command at least once prior (on the GPU's timeline) to executing it. Initializing gives the implementation the chance to perform any work necessary to accelerate the invocation of the meta command. You must supply sufficient resource parameters declared by the algorithm, including a persistent cache resource when the algorithm requires one.

#### Parameters

`pMetaCommand`

A pointer to an [ID3D12MetaCommand](#meta-commands) representing the meta command to initialize.

`pInitializationParametersData`

An optional pointer to a constant structure containing the values of the parameters for initializing the meta command.

`InitializationParametersDataSizeInBytes`

A [SIZE_T](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types) containing the size of the structure pointed to by *pInitializationParametersData*, if set, otherwise 0.

#### Return values

This method returns no value. Recording failures follow the [command-list error contract](execution.md#deferred-recording-errors).

#### State and execution context

The initialization state contribution is [`InitializationDirtyState`](command-list-state.md#meta-commands). Initialization ordering is on the GPU timeline, not merely the order of CPU calls recording different lists. The [render-pass operation-admission table](output-merger.md#render-pass-operation-surface) forbids initialization inside an open render pass. Initialization ignores predication, as defined in the [predication matrix](counters-queries-and-predication.md#predication-operation-matrix).

<a id="executemetacommand"></a>

### ID3D12GraphicsCommandList4::ExecuteMetaCommand

#### Declaration

```cpp
void ExecuteMetaCommand(
    ID3D12MetaCommand * pMetaCommand,
    const void * pExecutionParametersData,
    SIZE_T ExecutionParametersDataSizeInBytes
);
```

#### Behavior

Records the execution (or invocation) of the specified meta command into a graphics command list.

Call [ID3D12GraphicsCommandList4::InitializeMetaCommand](#initializemetacommand) before executing a meta command. During invocation, you can specify overrides for values of any of the runtime parameters. You can execute multiple meta commands on the same graphics command list. And you can execute the same meta command multiple times on the same command list.

#### Parameters

`pMetaCommand`

A pointer to an **ID3D12MetaCommand** representing the meta command to execute.

`pExecutionParametersData`

An optional pointer to a constant structure containing the values of the parameters for executing the meta command.

`ExecutionParametersDataSizeInBytes`

A [SIZE_T](https://learn.microsoft.com/en-us/windows/desktop/WinProg/windows-data-types) containing the size of the structure pointed to by *pExecutionParametersData*, if set, otherwise 0.

#### Resource synchronization

The application must place each supplied resource in the state required by the algorithm's parameter definition and reflected `RequiredResourceState`. The application must provide the ordering required by actual resource-access hazards under the [common synchronization contract](synchronization.md), using legacy barriers or the applicable enhanced-barrier equivalents. Input resources do not require a UAV barrier merely because they are inputs; the required ordering depends on their preceding and subsequent accesses.

During an invocation, the driver may insert UAV barriers needed to synchronize the algorithm's internal output-resource usage. These internal barriers do not replace the application's external state transitions and hazard ordering. The application must treat all output and input/output resources, including scratch memory, as written by the meta command.

#### Return values

This method returns no value. Recording failures follow the [command-list error contract](execution.md#deferred-recording-errors).

#### State and execution context

The execution state contribution is [`ExecutionDirtyState`](command-list-state.md#meta-commands). The [render-pass operation-admission table](output-merger.md#render-pass-operation-surface) forbids execution inside an open render pass. The [predication matrix](counters-queries-and-predication.md#predication-operation-matrix) defines execution predication as algorithm-defined; a dirty-state mask is not a predicate-support declaration.

<a id="driver-interface"></a>

## Published driver interface

The driver supplies meta-command descriptions, parameter descriptions and implementation-specific resource requirements. The public `0052` callbacks below separate driver-private object allocation from the resource memory addressed by algorithm parameters. `pfnCalcPrivateMetaCommandSize` describes driver-private object storage; `pfnGetMetaCommandRequiredParameterInfo` reports `ResourceSize` for a stage/parameter of an existing instance.

The driver parameter descriptor contains `Name`, `Type`, `Flags` and `RequiredResourceState`; it has no `StructureOffset`. The runtime supplies the application-facing layout as described [above](#d3d12_meta_command_parameter_desc). Device enumeration/creation/private-size/destruction/resource-info callbacks are members of `D3D12DDI_DEVICE_FUNCS_CORE_0052`; initialization and execution are members of `D3D12DDI_COMMAND_LIST_FUNCS_3D_0052`. The declarations do not establish a one-to-one API/DDI call count or require a particular hardware algorithm.

```cpp
typedef enum D3D12DDI_META_COMMAND_PARAMETER_TYPE
{
    D3D12DDI_META_COMMAND_PARAMETER_TYPE_FLOAT = 0,
    D3D12DDI_META_COMMAND_PARAMETER_TYPE_UINT64 = 1,
    D3D12DDI_META_COMMAND_PARAMETER_TYPE_GPU_VIRTUAL_ADDRESS = 2,
    D3D12DDI_META_COMMAND_PARAMETER_TYPE_CPU_DESCRIPTOR_HANDLE_HEAP_TYPE_CBV_SRV_UAV = 3,
    D3D12DDI_META_COMMAND_PARAMETER_TYPE_GPU_DESCRIPTOR_HANDLE_HEAP_TYPE_CBV_SRV_UAV = 4,
} D3D12DDI_META_COMMAND_PARAMETER_TYPE;

typedef enum D3D12DDI_META_COMMAND_PARAMETER_FLAGS
{
    D3D12DDI_META_COMMAND_PARAMETER_FLAG_INPUT = 0x00000001,
    D3D12DDI_META_COMMAND_PARAMETER_FLAG_OUTPUT = 0x00000002
} D3D12DDI_META_COMMAND_PARAMETER_FLAGS;
DEFINE_ENUM_FLAG_OPERATORS( D3D12DDI_META_COMMAND_PARAMETER_FLAGS )


typedef enum D3D12DDI_META_COMMAND_PARAMETER_STAGE
{
    D3D12DDI_META_COMMAND_PARAMETER_STAGE_CREATION = 0,
    D3D12DDI_META_COMMAND_PARAMETER_STAGE_INITIALIZATION = 1,
    D3D12DDI_META_COMMAND_PARAMETER_STAGE_EXECUTION = 2,
} D3D12DDI_META_COMMAND_PARAMETER_STAGE;

typedef struct D3D12DDIARG_META_COMMAND_PARAMETER_DESC
{
    LPCWSTR Name;
    D3D12DDI_META_COMMAND_PARAMETER_TYPE Type;
    D3D12DDI_META_COMMAND_PARAMETER_FLAGS Flags;
    D3D12DDI_RESOURCE_STATES RequiredResourceState;
} D3D12DDIARG_META_COMMAND_PARAMETER_DESC;

typedef struct D3D12DDIARG_META_COMMAND_REQUIRED_PARAMETER_INFO
{
    UINT64 ResourceSize;
} D3D12DDIARG_META_COMMAND_REQUIRED_PARAMETER_INFO;

typedef enum D3D12DDI_GRAPHICS_STATES
{
    D3D12DDI_GRAPHICS_STATE_NONE                    =        0,
    D3D12DDI_GRAPHICS_STATE_IA_VERTEX_BUFFERS       = (1 <<  0),
    D3D12DDI_GRAPHICS_STATE_IA_INDEX_BUFFER         = (1 <<  1),
    D3D12DDI_GRAPHICS_STATE_IA_PRIMITIVE_TOPOLOGY   = (1 <<  2),
    D3D12DDI_GRAPHICS_STATE_DESCRIPTOR_HEAP         = (1 <<  3),
    D3D12DDI_GRAPHICS_STATE_GRAPHICS_ROOT_SIGNATURE = (1 <<  4),
    D3D12DDI_GRAPHICS_STATE_COMPUTE_ROOT_SIGNATURE  = (1 <<  5),
    D3D12DDI_GRAPHICS_STATE_RS_VIEWPORTS            = (1 <<  6),
    D3D12DDI_GRAPHICS_STATE_RS_SCISSOR_RECTS        = (1 <<  7),
    D3D12DDI_GRAPHICS_STATE_PREDICATION             = (1 <<  8),
    D3D12DDI_GRAPHICS_STATE_OM_RENDER_TARGETS       = (1 <<  9),
    D3D12DDI_GRAPHICS_STATE_OM_STENCIL_REF          = (1 <<  10),
    D3D12DDI_GRAPHICS_STATE_OM_BLEND_FACTOR         = (1 <<  11),
    D3D12DDI_GRAPHICS_STATE_PIPELINE_STATE          = (1 << 12),
    D3D12DDI_GRAPHICS_STATE_SO_TARGETS              = (1 << 13),
    D3D12DDI_GRAPHICS_STATE_OM_DEPTH_BOUNDS         = (1 << 14),
    D3D12DDI_GRAPHICS_STATE_SAMPLE_POSITIONS        = (1 << 15),
    D3D12DDI_GRAPHICS_STATE_VIEW_INSTANCE_MASK      = (1 << 16),

} D3D12DDI_GRAPHICS_STATES;
DEFINE_ENUM_FLAG_OPERATORS( D3D12DDI_GRAPHICS_STATES )

typedef struct D3D12DDIARG_META_COMMAND_DESC
{
    GUID Id;
    LPCWSTR Name;
    D3D12DDI_GRAPHICS_STATES InitializationDirtyState;
    D3D12DDI_GRAPHICS_STATES ExecutionDirtyState;
} D3D12DDIARG_META_COMMAND_DESC;


typedef HRESULT ( APIENTRY* PFND3D12DDI_ENUMERATE_META_COMMANDS_0052 ) ( D3D12DDI_HDEVICE, _Inout_ UINT* pNumMetaCommands, _Out_writes_opt_( *pNumMetaCommands ) D3D12DDIARG_META_COMMAND_DESC* pDescs);
typedef HRESULT ( APIENTRY* PFND3D12DDI_ENUMERATE_META_COMMAND_PARAMETERS_0052 ) ( D3D12DDI_HDEVICE, GUID CommandId, D3D12DDI_META_COMMAND_PARAMETER_STAGE Stage, _Inout_ UINT* pParameterCount, _Out_writes_opt_( *pParameterCount ) D3D12DDIARG_META_COMMAND_PARAMETER_DESC* pParameterDescs);

typedef HRESULT(APIENTRY* PFND3D12DDI_CREATE_META_COMMAND_0052) (D3D12DDI_HDEVICE, GUID CommandId, UINT NodeMask, CONST void* pCreationParameters, SIZE_T CreationParametersDataSizeInBytes, D3D12DDI_HMETACOMMAND_0052, D3D12DDI_HRTMETACOMMAND_0052);
typedef VOID(APIENTRY* PFND3D12DDI_DESTROY_META_COMMAND_0052) (D3D12DDI_HDEVICE, D3D12DDI_HMETACOMMAND_0052);
typedef SIZE_T ( APIENTRY* PFND3D12DDI_CALC_PRIVATE_META_COMMAND_SIZE_0052 )( D3D12DDI_HDEVICE,  GUID CommandId, UINT NodeMask, CONST void* pCreationParameters, SIZE_T CreationParametersDataSizeInBytes);
typedef VOID(APIENTRY* PFND3D12DDI_GET_META_COMMAND_REQUIRED_PARAMETER_INFO_0052) (D3D12DDI_HMETACOMMAND_0052, D3D12DDI_META_COMMAND_PARAMETER_STAGE Stage, UINT ParameterIndex, _Out_ D3D12DDIARG_META_COMMAND_REQUIRED_PARAMETER_INFO*);
```

```cpp
typedef VOID(APIENTRY* PFND3D12DDI_INITIALIZE_META_COMMAND_0052)(
    D3D12DDI_HCOMMANDLIST, D3D12DDI_HMETACOMMAND_0052, _In_ CONST void *, _In_ SIZE_T );
typedef VOID(APIENTRY* PFND3D12DDI_EXECUTE_META_COMMAND_0052)(
    D3D12DDI_HCOMMANDLIST, D3D12DDI_HMETACOMMAND_0052, _In_ CONST void *, _In_ SIZE_T );
```

The meta-command initialize call must be executed at least once prior to any execute call being executed on the GPU timeline.

### Driver-owned command names

`D3D12DDIARG_META_COMMAND_DESC::Name` is a pointer to a wide string that holds the name of the meta-command. The driver allocates and keeps this string for the lifetime of the device.

### Parameter validation

The debug layer validates parameter-block sizes and supplied GPU virtual addresses and descriptor handles during creation, initialization and execution. These checks do not replace application resource-state and synchronization responsibilities.
