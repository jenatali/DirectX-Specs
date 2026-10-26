<a id="compiler-invocation-and-cache-output"></a>

# Compiler invocation and cache output

A compiler cache session maps application group keys to vendor-generated value
keys and stores the typed results of offline pipeline-state and state-object
compilation. A compiler state object retains compilation state for collections
and subsequent additions; it is not an `ID3D12StateObject` created by a device
and cannot be submitted for GPU execution.

These APIs compile pipeline/state descriptions through the IHV compiler plugin;
they are not HLSL source-language frontend compilation calls.

The [factory and target contracts](offline-compilation.md#offline-compilation)
and [plugin negotiation](compiler-plugin-foundation.md#negotiation) establish
the compiler context. Shared
[application metadata](state-object-databases-and-identity.md#application-metadata)
and [serialized root input forms](state-object-databases-and-identity.md#d3d12_serialized_root_signature_desc)
are defined with the state-object database inputs. The output storage
described here is the Precompiled Shader Database (PSDB).

The [registration catalog](shader-cache-registration.md#catalog-identity) associates application components with SODB and PSDB paths after production. [RegisterComponent](shader-cache-registration.md#id3dshadercacheapplication-registercomponent) and its path getters do not replace this chapter's typed/group storage or validate registered file contents.

`ID3D12Compiler` and `ID3D12CompilerCacheSession` derive from
`ID3D12CompilerFactoryChild`; their inherited
[`GetFactory`](offline-compilation.md#id3d12compilerfactorychild-getfactory)
contract is not a new operation on either interface. `ID3D12CompilerStateObject`
derives from `IUnknown` and exposes
[`GetCompiler`](#id3d12compilerstateobject-getcompiler).

The compiler plugin is responsible for synchronization as necessary,
applications do not need to add external synchronization. Caller-owned
descriptions, arrays, keys, strings, callback functions and context storage must
remain valid while the receiving call uses them. That synchronization contract
does not remove the object-reference or structural-dependency requirements
below.

<a id="output-storage"></a>

## Output storage and object subdivision

Each compiler exposes API to compile a PSO or SO that take arguments that are similar to the object creation counterparts in the D3D12 runtime.  The implementation validates API state objects and translates them into DDI representations for calling the IHV plugin.  The implementation for performing this validation and translation is shared with the D3D12 runtime.  The runtime calls the plugin with DDI arguments similar to the runtime to compile.  The plugin is expected to decompose this object, compile, and store each compiled object as a value via callbacks to the runtime.  The plugin finally sets the list of values associated with the input object via a callback to the runtime.  The runtime stores this list of values as a value group in the cache session using the group key supplied by the caller during the compile operation.

A collection compilation that produces no compiled output may omit the
output-key callback and have no output group. This exception is specific to
[`CompileStateObject`](#id3d12compiler-compilestateobject); it does not apply
to PSO/executable compilation or additions.

The Precompiled Shader Database(PSDB) is implemented as Sqlite3 databases for storage.  The compiler runtime API wraps the database as an ID3D12CompilerCacheSession.

The PSDB Stores:

- Metadata about the target application and adapter family.
- Compiled objects produced by the IHV compiler plugin are stored as values.  A single PSO or SO compilation may produce multiple values.
- A value group.  Each PSO compilation and each SO compilation that produces output has a single value group that maps its associated values.  Values may appear in more than one group (For example, if the same shader was part of another PSO or SO).

The compiler plugin may sub-divide DDI objects (PSOs, SOs, and Partial SOs) in an IHV dependent way to support deduplication and multiple values may be stored per compile operation.  The IHV plugin is responsible for generating value keys from the inputs that contribute to the compiled output and other internal data. The compiler may also check for previously compiled duplicates.

Stored values in cache sessions may be directly compared with previous compiler version output, adapter families considered to be similar in some fashion, etc. for further deduplication.  Compilers should refrain from storing additional data in the same value buffer that causes comparison failure when compiled binaries are otherwise identical.  Compilers can instead store compiler metadata for each compiler output for this type of data instead.  For example, storing compiler version or application version information that produced a binary with the value will cause comparison to fail even if they are otherwise identical.

Application group keys and group versions identify source-object revisions.
Vendor-generated value keys identify reusable compiled parts; a group can
reference several keys, and a value can belong to several groups. The
`GroupVersion` argument is not a member of
`D3D12_COMPILER_CACHE_GROUP_KEY`. A change to a source-object definition
requires the corresponding group version to be revised.

The target's adapter-family index is local to its factory; the stored identity
uses the family identifier, ABI and compiler version. The
[compiler, ABI and application-profile version domains](offline-compilation.md#version-domains)
are distinct from application and engine versions in the
[application descriptor](state-object-databases-and-identity.md#d3d12_application_desc),
group versions, value-type flags and database-format versions.

<a id="output-types"></a>

## Public output types

<a id="d3d12_compiler_value_type"></a>

### D3D12_COMPILER_VALUE_TYPE

```cpp
typedef enum D3D12_COMPILER_VALUE_TYPE
{
    D3D12_COMPILER_VALUE_TYPE_OBJECT_CODE       = 0,
    D3D12_COMPILER_VALUE_TYPE_METADATA          = 1,
    D3D12_COMPILER_VALUE_TYPE_DEBUG_PDB         = 2,
    D3D12_COMPILER_VALUE_TYPE_PERFORMANCE_DATA  = 3,

} D3D12_COMPILER_VALUE_TYPE;
```

A type enumeration used when only a single value type may be selected.

**Constants**

*D3D12_COMPILER_VALUE_TYPE_OBJECT_CODE*

Specifies object code.  The compiled executable code that is run on GPU.

*D3D12_COMPILER_VALUE_TYPE_METADATA*

Metadata that the compiler may provide about the compile.  For example it may store a compiler version number used for validation.  Such a thing should not be put inside of the object code as users of this system expect to be able to do memcmp diffing between compiler versions.

*D3D12_COMPILER_VALUE_TYPE_DEBUG_PDB*

The Debug PDB for the object code.  Used in pix and other debug scenarios.  An opaque blob that still requires IHV interpretation to be used with PIX.

*D3D12_COMPILER_VALUE_TYPE_PERFORMANCE_DATA*

Performance data about the compile or produced object code.  An opaque blob that still requires IHV interpretation to be used with PIX.

**Remarks**

Used when defining a typed value, see [D3D12_COMPILER_CACHE_TYPED_VALUE](#d3d12_compiler_cache_typed_value).

<a id="d3d12_compiler_value_type_flags"></a>

### D3D12_COMPILER_VALUE_TYPE_FLAGS

```cpp
typedef enum D3D12_COMPILER_VALUE_TYPE_FLAGS
{
    D3D12_COMPILER_VALUE_TYPE_FLAGS_NONE                = 0x00000000,
    D3D12_COMPILER_VALUE_TYPE_FLAGS_OBJECT_CODE         = (1 << D3D12_COMPILER_VALUE_TYPE_OBJECT_CODE),
    D3D12_COMPILER_VALUE_TYPE_FLAGS_METADATA            = (1 << D3D12_COMPILER_VALUE_TYPE_METADATA),
    D3D12_COMPILER_VALUE_TYPE_FLAGS_DEBUG_PDB           = (1 << D3D12_COMPILER_VALUE_TYPE_DEBUG_PDB),
    D3D12_COMPILER_VALUE_TYPE_FLAGS_PERFORMANCE_DATA    = (1 << D3D12_COMPILER_VALUE_TYPE_PERFORMANCE_DATA),

} D3D12_COMPILER_VALUE_TYPE_FLAGS;
```

```cpp
DEFINE_ENUM_FLAG_OPERATORS( D3D12_COMPILER_VALUE_TYPE_FLAGS )
```

A flags enumeration used where multiple values may be selected.

**Constants**

*D3D12_COMPILER_VALUE_TYPE_FLAGS_NONE*

No selected value types.

*D3D12_COMPILER_VALUE_TYPE_FLAGS_OBJECT_CODE*

Specifies that the operation applies to object code.

*D3D12_COMPILER_VALUE_TYPE_FLAGS_METADATA*

Selects compiler metadata.

*D3D12_COMPILER_VALUE_TYPE_FLAGS_DEBUG_PDB*

Specifies that the operation applies to the debug PDB.

*D3D12_COMPILER_VALUE_TYPE_FLAGS_PERFORMANCE_DATA*

Specifies that the operation applies to performance data.

**Remarks**

Used during compilation to indicate if object code or metadata should be stored or if debug pdb and performance data need to be produced and stored.

<a id="d3d12_compiler_database_path"></a>

### D3D12_COMPILER_DATABASE_PATH

```cpp
typedef struct D3D12_COMPILER_DATABASE_PATH
{
    D3D12_COMPILER_VALUE_TYPE_FLAGS Types;
    LPCWSTR pPath;
} D3D12_COMPILER_DATABASE_PATH;
```

Specifies a database path and the value types it stores.

**Members**

*Types*

The types stored in pPath.  See [D3D12_COMPILER_VALUE_TYPE_FLAGS](#d3d12_compiler_value_type_flags).

*pPath*

A file path specifying the location of a database file.

**Remarks**

Databases store one or more value types.  The types member specifies which value types are stored in pPath.

<a id="d3d12_compiler_cache_group_key"></a>

### D3D12_COMPILER_CACHE_GROUP_KEY

```cpp
typedef struct D3D12_COMPILER_CACHE_GROUP_KEY
{
    const void* pKey;
    UINT KeySize;
} D3D12_COMPILER_CACHE_GROUP_KEY;
```

A structure representing a shader cache group key.

**Members**

*pKey*

A unique sequence of bytes that uniquely identifies the group in the database.

*KeySize*

The size in bytes of pKey.

**Remarks**

This key is used to match state object definitions in the state object database with the group of cached binaries in the shader cache session.  The database author is expected to rev the version number of a source object anytime its definition changes in a state object database.

<a id="d3d12_compiler_cache_value_key"></a>

### D3D12_COMPILER_CACHE_VALUE_KEY

```cpp
typedef struct D3D12_COMPILER_CACHE_VALUE_KEY
{
    const void* pKey;
    UINT KeySize;
} D3D12_COMPILER_CACHE_VALUE_KEY;
```

**Members**

*pKey*

A unique sequence of bytes that uniquely identifies an object in the database.

*KeySize*

The size in bytes of pKey.

**Remarks**

Used during cache operations like find and store value to identify a value.

Group keys and value keys are opaque byte sequences, not necessarily text
or null-terminated strings. `KeySize` counts bytes. Each `pKey` must address
the bytes described by that count while the receiving call uses it. Application
group keys and vendor-generated value keys occupy different roles even when
their byte sequences happen to match.

<a id="d3d12_compiler_cache_value"></a>

### D3D12_COMPILER_CACHE_VALUE

```cpp
typedef struct D3D12_COMPILER_CACHE_VALUE
{
    void* pValue;
    UINT ValueSize;
} D3D12_COMPILER_CACHE_VALUE;
```

A non-const value buffer and buffer size structure for cached values.

**Members**

*pValue*

A pointer to a memory buffer containing the value.

*ValueSize*

The size of the pValue buffer in bytes.

**Remarks**

Used for object code, compiler metadata, debug PDB, and performance data.

<a id="d3d12_compiler_cache_typed_value"></a>

### D3D12_COMPILER_CACHE_TYPED_VALUE

```cpp
typedef struct D3D12_COMPILER_CACHE_TYPED_VALUE
{
    D3D12_COMPILER_VALUE_TYPE Type;
    D3D12_COMPILER_CACHE_VALUE Value;
} D3D12_COMPILER_CACHE_TYPED_VALUE;
```

Specifies a value and its type, such as object code, metadata etc.

**Members**

*Type*

The type of the value, such as object code, metadata, etc.  See [D3D12_COMPILER_VALUE_TYPE](#d3d12_compiler_value_type) for more information on types.

*Value*

The buffer and size for the value.  See [D3D12_COMPILER_CACHE_VALUE](#d3d12_compiler_cache_value).

**Remarks**

Used in API where the compiler may modify the value buffer, such as [ID3D12CompilerCacheSession::FindValue](#id3d12compilercachesession-findvalue).

<a id="d3d12_compiler_cache_const_value"></a>

### D3D12_COMPILER_CACHE_CONST_VALUE

```cpp
typedef struct D3D12_COMPILER_CACHE_CONST_VALUE
{
    const void* pValue;
    UINT ValueSize;
} D3D12_COMPILER_CACHE_CONST_VALUE;
```

Specifies a const value buffer and buffer size.

**Members**

*pValue*

A const pointer to a memory buffer containing the value.

*ValueSize*

The size of the pValue buffer in bytes.

**Remarks**

See also [D3D12_COMPILER_CACHE_TYPED_CONST_VALUE](#d3d12_compiler_cache_typed_const_value).

<a id="d3d12_compiler_cache_typed_const_value"></a>

### D3D12_COMPILER_CACHE_TYPED_CONST_VALUE

```cpp
typedef struct D3D12_COMPILER_CACHE_TYPED_CONST_VALUE
{
    D3D12_COMPILER_VALUE_TYPE Type;
    D3D12_COMPILER_CACHE_CONST_VALUE Value;
} D3D12_COMPILER_CACHE_TYPED_CONST_VALUE;
```

**Members**

*Type*

The type of the value, such as object code, metadata, etc.  See [D3D12_COMPILER_VALUE_TYPE](#d3d12_compiler_value_type) for more information on types.

*Value*

The const buffer and size for the value.  See [D3D12_COMPILER_CACHE_CONST_VALUE](#d3d12_compiler_cache_const_value).

**Remarks**

Used in API where the compiler may not modify the value buffer, such as [ID3D12CompilerCacheSession::StoreValue](#id3d12compilercachesession-storevalue).

<a id="d3d12_compiler_existing_collection_desc"></a>

### D3D12_COMPILER_EXISTING_COLLECTION_DESC

```cpp
typedef struct D3D12_COMPILER_EXISTING_COLLECTION_DESC
{
    ID3D12CompilerStateObject* pExistingCollection;
    UINT NumExports;
    const D3D12_EXPORT_DESC* pExports;
} D3D12_COMPILER_EXISTING_COLLECTION_DESC;
```

```cpp
D3D12_STATE_SUBOBJECT_TYPE_COMPILER_EXISTING_COLLECTION = 33, // D3D12_COMPILER_EXISTING_COLLECTION_DESC
```

Use `D3D12_STATE_SUBOBJECT_TYPE_COMPILER_EXISTING_COLLECTION` with this
descriptor. `pExistingCollection` is a live compiler state-object collection,
not an `ID3D12StateObject` from a device or a database key.
`NumExports` counts `D3D12_EXPORT_DESC` entries; when zero, all collection
exports are surfaced. Otherwise `pExports` points to that many readable export
descriptors, with the export selection/renaming semantics of
[existing collections](state-objects.md#d3d12_existing_collection_desc).
The referenced collection must come from the compiler receiving the description. The application must
retain it while its dependents need it; compilation does not automatically
retain a collection API reference.

<a id="public-callbacks"></a>

## Public callbacks

<a id="d3d12compilercachesessionallocationfunc"></a>

### D3D12CompilerCacheSessionAllocationFunc

```cpp
typedef void* (__stdcall* D3D12CompilerCacheSessionAllocationFunc) (
    SIZE_T SizeInBytes,
    _Inout_opt_ void* pContext
    );
```

An allocation callback to enable a single call to get size and buffer contents.

**Parameters**

*SizeInBytes*

 The size in bytes of the requested allocation.

*pContext*

An application defined context pointer.

**Remarks**

Returns a non-nullptr value on success which points to buffer allocation meeting the size requirements.  Allocations must meet the fundamental alignment requirements and be suitably aligned for any object, see std::max_align_t.

Returns nullptr on allocation failure.

This function must not throw uncaught exceptions.

<a id="d3d12compilercachesessiongroupvaluekeysfunc"></a>

### D3D12CompilerCacheSessionGroupValueKeysFunc

```cpp
typedef void(__stdcall* D3D12CompilerCacheSessionGroupValueKeysFunc) (
    _In_ const D3D12_COMPILER_CACHE_VALUE_KEY* pValueKey,
    _Inout_opt_ void* pContext
    );
```

A callback function for retrieving one or more shader cache value keys for use with the [ID3D12CompilerCacheSession::FindGroupValueKeys](#id3d12compilercachesession-findgroupvaluekeys) method.

**Arguments**

*pValueKey*

A value key for a value that is part of the specified group.  The allocations backing these keys are de-allocated and should no longer be accessed once D3D12CompilerCacheSessionGroupValueKeysFunc returns.

*pContext*

An application defined context pointer.

**Remarks**

This function is called once for each value that is part of the group.

This function must not throw uncaught exceptions.  The allocations backing pValueKey and pValueKey->pKey are valid until this function returns.

<a id="d3d12compilercachesessiongroupvaluesfunc"></a>

### D3D12CompilerCacheSessionGroupValuesFunc

```cpp
typedef void(__stdcall* D3D12CompilerCacheSessionGroupValuesFunc) (
    UINT ValueKeyIndex,
    _In_ const D3D12_COMPILER_CACHE_TYPED_CONST_VALUE* pTypedValue,
    _Inout_opt_ void* pContext
    );
```

A callback function for retrieving the binary values of each value in a group.

**Arguments**

*ValueKeyIndex*

An index grouping the typed values for one value key within the filtered backing-database query.  For example, if both OBJECT_CODE and METADATA are requested and stored for a value key in the same backing database, this function is called twice with ValueKeyIndex 0 before incrementing for the next set of values.

*pTypedValue*

Indicates a single value type and value for the callback.  See [D3D12_COMPILER_CACHE_TYPED_CONST_VALUE](#d3d12_compiler_cache_typed_const_value).

*pContext*

An application defined context pointer.

**Remarks**

This function is called once for each selected stored value type that is part of the group.  Use ValueKeyIndex to associate value types to the same value key.

This function must not throw uncaught exceptions.

`pTypedValue` and its value buffer passed to a group-value callback are borrowed
for that callback invocation. Copy any data needed afterward; do not free the
borrowed storage. The application owns `pContext`; the lookup passes it through.
The callbacks execute as part of the lookup before that lookup returns.

The allocation callback instead supplies application-owned storage. The caller
must release successful allocations with the matching deallocator, including
allocations returned before a later failure. The cache session does not take
ownership of those allocations.

<a id="id3d12compilerfactory-createcompilercachesession"></a>

## ID3D12CompilerFactory::CreateCompilerCacheSession

### Declaration

```cpp
HRESULT CreateCompilerCacheSession(
    const D3D12_COMPILER_DATABASE_PATH* pPaths,
    UINT NumPaths,
    const D3D12_COMPILER_TARGET* pTarget,
    const D3D12_APPLICATION_DESC* pApplicationDesc,
    REFIID riid,
    void** ppCompilerCacheSession
);
```

### Behavior

Database Files:

- If a specified file does not exist, it is created.  If it exists, it is opened for modification.
- Value types may be stored in separate databases.  For example, PDBs may be stored in a separate database file than object code.

`pPaths` is required and `NumPaths` must be nonzero. Each `Types` mask must
select at least one of the four declared value types and no undeclared bits.
The masks must be pairwise disjoint: each type is assigned to exactly one
database entry. `GetValueTypes` returns their union. Object code and metadata
must either both be in this union or both be absent; they may be assigned to
different entries. Debug PDB and performance-data storage can be selected
independently.

Each `pPath` names a Windows file, using a filename, local path or UNC path;
the containing directory must exist. A null path selects a session-local
in-memory database. It is not the filename `:memory:`; SQLite URI filenames
and that special filename are not accepted by this API. The path descriptor
contains no filename-extension inference and no access-mode flag.
Existing files are opened for modification and missing files are created;
this is not a read-only open API. Read-only files or inaccessible paths can
fail to open. SODB's
[read-only flag](state-object-databases-and-identity.md#d3d12_state_object_database_flags)
does not apply to this call.

Disjoint masks route types; distinct path strings can still identify the
same physical file. Group writes are made to every database entry. Physical-file
deduplication and an all-files transaction are not guaranteed, so aliased
entries must not be treated as independent storage.

`pTarget` and `pApplicationDesc` are optional only when the database metadata
supplies the corresponding identity. If no opened database supplies an
identity, that descriptor must be provided; otherwise creation returns
`E_INVALIDARG`. New databases receive the resolved identities. Missing metadata
in one file may be filled from consistent metadata in another file or from the
supplied descriptors.

All existing compiler identities must agree on family identifier, ABI and
compiler version. All application identities must agree on executable
filename, application name/version, engine name/version and application-profile
version. Supplied descriptors must agree with stored identities. Null
application string members compare as empty strings; application string
comparisons are case-sensitive. Inconsistent identities
return `E_INVALIDARG`, rather than silently changing the identity of a populated
database.

The factory resolves a stored family identifier to its own local family index.
An explicit target's ABI zero resolves to the newest supported ABI under
[target resolution](offline-compilation.md#d3d12_compiler_target); an explicit
nonzero ABI must be supported. The resolved family, ABI and compiler version
must match stored metadata. The application-profile version must match the
version reported by this compiler for the resolved target and application.
An unsupported stored family can fail lookup with `DXGI_ERROR_NOT_FOUND`;
identity/version mismatches return `E_INVALIDARG`.

Path strings and application strings are consumed or copied during creation;
the caller retains their ownership. `ppCompilerCacheSession` is required
writable interface-pointer storage, not a null-output availability probe.
On success it receives an owned reference selected by `riid`. Failures before
the interface query do not promise to clear an existing output value. Opening
and initializing earlier files may already have happened when a later file or
metadata check fails; no all-files rollback is guaranteed.

When `pApplicationDesc` is explicitly supplied, `pExeFilename` and `pName`
are required non-null, null-terminated strings identifying the target
application's executable filename and title. `Version` supplies its application
version. `pEngineName` is optional; if supplied it must be null-terminated and
`EngineVersion` must be provided. These are the
[application metadata requirements](state-object-databases-and-identity.md#d3d12_application_desc),
not a fallback to the compiler-hosting process's identity.

The optional descriptor pointer used to resolve existing database metadata is
a separate condition. Comparison or copying of null string members as empty
strings does not waive the required fields of an explicitly supplied
descriptor.

### Parameters

`[in] pPaths`

A list of paths that specify which types of values are stored by this cache session, and which database they should be stored in.  See [D3D12_COMPILER_DATABASE_PATH](#d3d12_compiler_database_path).

`[in] NumPaths`

The number of paths specified in pPaths.

`[in, optional] pTarget`

Describes the adapter family and ABI version to target.  Specify zero for ABIVersion to default to the latest supported ABI Version.  See [D3D12_COMPILER_TARGET](offline-compilation.md#d3d12_compiler_target).

`[in, optional] pApplicationDesc`

Describes the target application and version.  See [D3D12_APPLICATION_DESC](state-object-databases-and-identity.md#d3d12_application_desc).

`[in] riid`

The globally unique identifier (GUID) for the compiler cache session interface. See [ID3D12CompilerCacheSession](#id3d12compilerfactory-createcompilercachesession).

`[out] ppCompilerCacheSession`

On return, a pointer to the compiler cache-session interface specified by riid.

### Return values

`S_OK` returns the requested cache-session interface. Invalid routing masks, a missing path array, zero path count, and the
documented missing or mismatched identities return `E_INVALIDARG`; an unsupported requested interface
returns `E_NOINTERFACE`. Allocation, file-open/database and plugin query
failures propagate as failing `HRESULT`s. There is no universal filesystem-error
mapping or all-files rollback guarantee.

<a id="id3d12compilerfactory-createcompiler"></a>

## ID3D12CompilerFactory::CreateCompiler

### Declaration

```cpp
HRESULT CreateCompiler(
    ID3D12CompilerCacheSession* pCompilerCacheSession,
    REFIID riid,
    void** ppCompiler
);
```

### Behavior

Create an instance of a compiler.

Use [D3D12_APPLICATION_DESC](state-object-databases-and-identity.md#d3d12_application_desc) to query the profile version from the compiler plugin with [ID3D12CompilerFactory::GetApplicationProfileVersion](offline-compilation.md#id3d12compilerfactory-getapplicationprofileversion).  Compilers may revise this number to indicate application specific changes to compiler output.

`pCompilerCacheSession` and `ppCompiler` are required; null returns
`E_INVALIDARG`. The compiler retains a reference to its cache session and
inherits its target, application descriptor and selected value types.
The caller owns the returned compiler reference and releases it when no
longer needed. The retained session keeps its application strings alive.

The cache session need not have been created by this factory, but its target
index and metadata must be meaningful to this factory. No cross-factory index
translation is performed; see [input ownership](#input-ownership).
`ppCompiler` is not a null-output validation probe. Failure before the final
interface query need not initialize it. Plugin creation and allocation failures
fail this call.

### Parameters

`[in] pCompilerCacheSession`

The cache session where compiled results are stored.  The compiler plugin may also check if objects were previously compiled.  See [ID3D12CompilerCacheSession](#id3d12compilerfactory-createcompilercachesession).

`[in] riid`

The globally unique identifier (GUID) for the compiler interface. See [ID3D12Compiler](#id3d12compilerfactory-createcompiler).

`[out] ppCompiler`

On return, a pointer to the compiler interface specified by riid.

### Return values

`S_OK` returns the requested compiler interface; null required arguments
return `E_INVALIDARG`, and an unsupported interface returns `E_NOINTERFACE`.
Allocation and plugin compiler-creation failures propagate. A returned
compiler does not certify that every later description is compilable.

<a id="input-ownership"></a>

## Compilation inputs and ownership

Pipeline descriptions use
[`D3D12_PIPELINE_STATE_STREAM_DESC`](pipeline-state.md#streams).
For explicit roots, use
`D3D12_PIPELINE_STATE_SUBOBJECT_TYPE_SERIALIZED_ROOT_SIGNATURE` carrying
[`D3D12_SERIALIZED_ROOT_SIGNATURE_DESC`](state-object-databases-and-identity.md#d3d12_serialized_root_signature_desc),
not a device root-signature pointer. Roots embedded in shaders can be resolved
and validated by the compiler. Legacy cached PSO blobs and pipeline-library
inputs are not compiler-host inputs.

State-object descriptions use
[`D3D12_STATE_OBJECT_DESC`](state-objects.md#d3d12_state_object_desc).
Explicit roots use
[`D3D12_GLOBAL_SERIALIZED_ROOT_SIGNATURE`](state-object-databases-and-identity.md#d3d12_global_serialized_root_signature)
or
[`D3D12_LOCAL_SERIALIZED_ROOT_SIGNATURE`](state-object-databases-and-identity.md#d3d12_local_serialized_root_signature)
with their corresponding serialized-root subobject tags. An existing
compiler collection uses
[`D3D12_COMPILER_EXISTING_COLLECTION_DESC`](#d3d12_compiler_existing_collection_desc)
and `D3D12_STATE_SUBOBJECT_TYPE_COMPILER_EXISTING_COLLECTION`.
The database's
[`D3D12_EXISTING_COLLECTION_BY_KEY_DESC`](state-object-databases-and-identity.md#d3d12_existing_collection_by_key_desc)
is a storage reference, not a directly accepted compile input. Reconstruct the
dependency as a live compiler collection; a parent key from an SODB similarly
identifies a dependency to supply as a live parent to the addition call.

The compiler runtime validates descriptions and translates them to DDI
representations using shared runtime validation. Pipeline stream parsing,
stage composition, input/stream-output layouts, render-target and
depth/stencil formats, samples, rasterization and linkage retain the
[pipeline construction rules](pipeline-state.md#id3d12device2-createpipelinestate).
Root contents and shader compatibility retain
[Binding](binding.md#root-signatures).
State-object type/payload, exports, associations, defaults, collection linking
and construction retain
[State objects](state-objects.md#createstateobject),
[association requirements](state-objects.md#subobject-association-requirements)
and [default resolution](state-objects.md#default-resolution).
Incremental additions retain the
[addition restrictions](state-objects.md#addtostateobject).

Capability checks are made against the compiler's target using the
[negotiated plugin capabilities](compiler-plugin-foundation.md#get-caps),
including applicable feature level, shader model, shader features, formats,
multisampling, raytracing/state-object or work-graph requirements. Serializing
or storing a description does not establish that the target can compile it.
Invalid descriptions and unsupported target capabilities are distinct from
an IHV compiler failure.

Descriptions, their reachable arrays, bytecode and serialized roots must be
readable for the call. The compiler retains the internal descriptions and
dependency state needed by later additions; callers need not keep the
original top-level description allocation alive after a successful call.

Every referenced compiler collection and addition parent must have been created
by the same `ID3D12Compiler` that receives the dependent compile call. These
are caller preconditions; no particular mismatch `HRESULT` is guaranteed.

A cache session need not come from the compiler's factory. Its target,
application descriptor and selected value types must be meaningful to the
receiving factory. The target's family index is interpreted in that factory's
enumeration; the compiler does not remap an index from another factory.
The compiler retains its cache session, and a compiler state object retains
the compiler that created it.

Applications must retain compiler collections while their dependents need them.
Compilation does not automatically retain a collection API reference.
Retaining a compiler or copying description data is not a substitute for
keeping the referenced collection alive.

An addition retains the parent's description data, not the parent's API
reference or plugin handle. After the addition succeeds, the application need
not retain the addition-parent API reference for the child's lifetime.
Releasing the last parent reference destroys the parent's plugin handle;
the plugin must retain whatever dependency state it needs for the child.
This is the description/handle distinction in
[state-object driver lifetime](state-objects.md#driver-lifetime), not the
collection-retention rule above. Original caller-owned description allocations
can be released after successful compilation.

<a id="id3d12compiler-compilepipelinestate"></a>

## ID3D12Compiler::CompilePipelineState

### Declaration

```cpp
HRESULT CompilePipelineState(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    UINT GroupVersion,
    const D3D12_PIPELINE_STATE_STREAM_DESC* pDesc
);
```

### Behavior

Compile a graphics or compute pipeline state.

Compiled output is stored via [ID3D12CompilerCacheSession::StoreValue](#id3d12compilercachesession-storevalue).

Compiler may sub-divide objects in an IHV dependent way to support deduplication.  Therefore, multiple values may be stored per compile operation. The compiler may also check for previously compiled values.

The Group Key and Group Version are used to store a group that associates all the values stored for this compilation via [ID3D12CompilerCacheSession::StoreGroupValueKeys](#id3d12compilercachesession-storegroupvaluekeys).

On success, this method will not return until all compiled output is stored.

The cache session's `GetValueTypes()` mask selects the output types; there is
no value-type-flags argument on this API. The group key is required; the
description and all required referenced data must be valid.
Compilation completes synchronously, including the plugin's cache callbacks.
This is not an asynchronous job handle and does not create a GPU-executable
device object. Completion does not imply a cross-file atomic commit, crash
durability, or rollback of values stored before an error.

A null `pGroupKey` returns `E_INVALIDARG`. The
[compilation input and target-validation contract](#input-ownership)
defines the required description forms and shared construction rules.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group. This key is used to store a group with the values produced by this compiled operation.  This group must not already be present in the cache session.

`[in] GroupVersion`

The version number of the cached group.

`[in] pDesc`

The pipeline state desc describing the PSO.  See [D3D12_PIPELINE_STATE_STREAM_DESC](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_pipeline_state_stream_desc).

### Return values

`S_OK` indicates successful validation and plugin completion. Description
validation failures return `E_INVALIDARG`; unsupported target capabilities can return
`DXGI_ERROR_UNSUPPORTED`. Allocation failures return `E_OUTOFMEMORY`.

Plugin compile failures preserve `E_OUTOFMEMORY`; other failing plugin compile
results are translated to `DXGI_ERROR_DRIVER_INTERNAL_ERROR`. This is distinct
from the cache callbacks' own results, which are returned to the plugin.
The plugin must handle or propagate callback failures; a callback result is
not necessarily the public compile call's final `HRESULT`. Missing required
group-key registration also fails compilation with
`DXGI_ERROR_DRIVER_INTERNAL_ERROR`.
Stored values can survive a failing compile.

<a id="id3d12compiler-compilestateobject"></a>

## ID3D12Compiler::CompileStateObject

### Declaration

```cpp
HRESULT CompileStateObject(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    UINT GroupVersion,
    const D3D12_STATE_OBJECT_DESC* pDesc,
    REFIID riid,
    void** ppCompilerStateObject
);
```

### Behavior

Compiled output is stored via [ID3D12CompilerCacheSession::StoreValue](#id3d12compilercachesession-storevalue).

Compiler may sub-divide objects in an IHV dependent way to support deduplication.  Therefore, multiple values may be stored per compile. The compiler may also check for previously compiled duplicates.

The Group Key and Group Version are used to store a group that associates all the values stored for this compilation via [ID3D12CompilerCacheSession::StoreGroupValueKeys](#id3d12compilercachesession-storegroupvaluekeys).

On success, this method will not return until all compiled output is stored.

This method creates an object that implements [ID3D12CompilerStateObject](#id3d12compiler-compilestateobject) to support using it as a base object when deriving new state objects with [ID3D12Compiler::CompileAddToStateObject](#id3d12compiler-compileaddtostateobject).

The cache session's `GetValueTypes()` mask selects the output types; there is
no value-type-flags argument on this API. The group key is required; the
description and all required referenced data must be valid.
Compilation completes synchronously, including the plugin's cache callbacks.
This is not an asynchronous job handle and does not create a GPU-executable
device object. Completion does not imply a cross-file atomic commit, crash
durability, or rollback of values stored before an error.

`riid` selects `ID3D12CompilerStateObject` or a supported base interface.
The output pointer is required writable storage, not a validation-only probe.
The returned interface is an owned reference; release it when finished.
An error before the final interface query need not clear the caller's output.
The handle retains the compiler and internal construction data needed by
dependent compilations. State-object compilation and addition require the
negotiated build-2 compiler DDI support; otherwise they return `E_NOTIMPL`.

A collection that produces no compiled output can return its compiler state
object without registering output keys or creating a group. The input group
key remains required. This exception does not apply to executable state-object
compilation or incremental additions.

A null `pGroupKey`, `pDesc` or `ppCompilerStateObject` returns `E_INVALIDARG`. The
[compilation input and target-validation contract](#input-ownership)
defines the required description forms and shared construction rules.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group. This key is used to store a group with the values produced by this compiled operation.  This group must not already be present in the cache session.

`[in] GroupVersion`

The version number of the cached group.

`[in] pDesc`

Creation arguments for the state object.  See [D3D12_STATE_OBJECT_DESC](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_state_object_desc).

`[in] riid`

The globally unique identifier (GUID) for the new state object interface. See [ID3D12CompilerStateObject](#id3d12compiler-compilestateobject).

`[out] ppCompilerStateObject`

On return, a pointer to the compiler state object interface specified by riid.

### Return values

`S_OK` indicates successful validation and plugin completion. Description
validation failures return `E_INVALIDARG`; unsupported target capabilities can return
`DXGI_ERROR_UNSUPPORTED`. Allocation failures return `E_OUTOFMEMORY`.

An unsupported requested interface returns `E_NOINTERFACE`; the output pointer
is required. That interface failure can occur after compilation and storage.

Plugin compile failures preserve `E_OUTOFMEMORY`; other failing plugin compile
results are translated to `DXGI_ERROR_DRIVER_INTERNAL_ERROR`. This is distinct
from the cache callbacks' own results, which are returned to the plugin.
The plugin must handle or propagate callback failures; a callback result is
not necessarily the public compile call's final `HRESULT`. Missing required
group-key registration also fails compilation with
`DXGI_ERROR_DRIVER_INTERNAL_ERROR`. A collection with no compiled output may omit registration and have no group.
Stored values can survive a failing compile.

<a id="id3d12compiler-compileaddtostateobject"></a>

## ID3D12Compiler::CompileAddToStateObject

### Declaration

```cpp
HRESULT CompileAddToStateObject(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    UINT GroupVersion,
    const D3D12_STATE_OBJECT_DESC* pAddition,
    ID3D12CompilerStateObject* pCompilerStateObjectToGrowFrom,
    REFIID riid,
    void** ppNewCompilerStateObject
);
```

### Behavior

Compiled output is stored via [ID3D12CompilerCacheSession::StoreValue](#id3d12compilercachesession-storevalue).

Compiler may sub-divide objects in an IHV dependent way to support deduplication.  Therefore, multiple values may be stored per compile. The compiler may also check for previously compiled duplicates.

On success, this method will not return until all compiled output is stored.

The Group Key and Group Version are used to store a group that associates all the values stored for this compilation via [ID3D12CompilerCacheSession::StoreGroupValueKeys](#id3d12compilercachesession-storegroupvaluekeys).

This method creates an object that implements [ID3D12CompilerStateObject](#id3d12compiler-compilestateobject) to support using it as a base object when deriving new state objects with [ID3D12Compiler::CompileAddToStateObject](#id3d12compiler-compileaddtostateobject).

The cache session's `GetValueTypes()` mask selects the output types; there is
no value-type-flags argument on this API. The group key is required; the
description and all required referenced data must be valid.
Compilation completes synchronously, including the plugin's cache callbacks.
This is not an asynchronous job handle and does not create a GPU-executable
device object. Completion does not imply a cross-file atomic commit, crash
durability, or rollback of values stored before an error.

`riid` selects `ID3D12CompilerStateObject` or a supported base interface.
The output pointer is required writable storage, not a validation-only probe.
The returned interface is an owned reference; release it when finished.
An error before the final interface query need not clear the caller's output.
The handle retains the compiler and internal construction data needed by
dependent compilations. State-object compilation and addition require the
negotiated build-2 compiler DDI support; otherwise they return `E_NOTIMPL`.

A null `pGroupKey`, `pAddition`, `pCompilerStateObjectToGrowFrom` or `ppNewCompilerStateObject` returns `E_INVALIDARG`. The
[compilation input and target-validation contract](#input-ownership)
defines the required description forms and shared construction rules.

`pCompilerStateObjectToGrowFrom` is required and must have been created by this
compiler. The result is a new compiler state object, not a replacement of the
parent. Its description storage retains the parent description data. It does
not retain the parent's API reference or guarantee the parent's plugin handle
remains alive; the plugin retains the dependency state it needs.
The application need not hold an addition-parent reference until the child is
destroyed. Collections remain subject to their distinct application-retention
rule.

The base must permit additions under
[`ALLOW_STATE_OBJECT_ADDITIONS`](state-objects.md#d3d12_state_object_flags),
and the new description must satisfy the
[growth and addition rules](state-objects.md#addtostateobject).

ID3D12Device7::AddToStateObject allows an application developer to incrementally add to an existing state object during game runtime.

During offline compilation, CPU overhead during compile is much less of an issue, so these incremental additions to a base state object instead could be factored as independent state objects.  However, this may result in higher runtime memory usage than the original extended state objects that may share state, so the compiler API and state object database continue to support both.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group. This key is used to store a group with the values produced by this compiled operation.  This group must not already be present in the cache session.

`[in] GroupVersion`

The version number of the cached group.

`[in] pAddition`

Addition arguments to add to pCompilerStateObjectToGrowFrom and create a new state object. See [D3D12_STATE_OBJECT_DESC](https://learn.microsoft.com/en-us/windows/win32/api/d3d12/ns-d3d12-d3d12_state_object_desc).

`[in] pCompilerStateObjectToGrowFrom`

The base state object to build from.

`[in] riid`

The globally unique identifier (GUID) for the new state object interface. See [ID3D12CompilerStateObject](#id3d12compiler-compilestateobject).

`[out] ppNewCompilerStateObject`

On return, a pointer to the compiler state object interface specified by riid.

### Return values

`S_OK` indicates successful validation and plugin completion. Description
validation failures return `E_INVALIDARG`; unsupported target capabilities can return
`DXGI_ERROR_UNSUPPORTED`. Allocation failures return `E_OUTOFMEMORY`.

An unsupported requested interface returns `E_NOINTERFACE`; the output pointer
is required. That interface failure can occur after compilation and storage.

Plugin compile failures preserve `E_OUTOFMEMORY`; other failing plugin compile
results are translated to `DXGI_ERROR_DRIVER_INTERNAL_ERROR`. This is distinct
from the cache callbacks' own results, which are returned to the plugin.
The plugin must handle or propagate callback failures; a callback result is
not necessarily the public compile call's final `HRESULT`. Missing required
group-key registration also fails compilation with
`DXGI_ERROR_DRIVER_INTERNAL_ERROR`.
Stored values can survive a failing compile.

<a id="id3d12compiler-getcachesession"></a>

## ID3D12Compiler::GetCacheSession

### Declaration

```cpp
HRESULT GetCacheSession(
    REFIID riid,
    void** ppCompilerCacheSession
);
```

### Behavior

Returns the cache session used to create this compiler.

### Parameters

`[in] riid`

Selects the requested interface.

`[out] ppCompilerCacheSession`

Required writable storage for the owned interface reference returned on success.

### Return values

`S_OK` returns an owned reference to the requested session interface;
`E_NOINTERFACE` reports an unsupported `riid`. A null output pointer is not a
probe and returns `E_POINTER` at the interface query. That query clears a
valid output slot on unsupported-interface failure.

<a id="id3d12compilerstateobject-getcompiler"></a>

## ID3D12CompilerStateObject::GetCompiler

### Declaration

```cpp
HRESULT GetCompiler(
    REFIID riid,
    void** ppCompiler
);
```

### Behavior

Returns the compiler that created this compiler state object.

### Parameters

`[in] riid`

Selects the requested interface.

`[out] ppCompiler`

Required writable storage for the owned interface reference returned on success.

### Return values

`S_OK` returns an owned reference to the compiler selected by `riid`;
`E_NOINTERFACE` reports an unsupported interface. A null output pointer is not
a probe and returns `E_POINTER` at the interface query. That query clears a
valid output slot on unsupported-interface failure.

<a id="id3d12compilercachesession-findgroup"></a>

## ID3D12CompilerCacheSession::FindGroup

### Declaration

```cpp
HRESULT FindGroup(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    UINT* pGroupVersion
);
```

### Behavior

Check if a group with pGroupKey is in the database.  Optionally retrieve the version number of the group.

Returns S_OK when the group exists and DXGI_ERROR_NOT_FOUND if it does not.

GroupVersion is used to match State Object definitions in an SODB with a group of compiled binaries in a PSDB.  Only one version of a group with a given key may be stored in a PSDB at a time.

`pGroupKey` is required. A null `pGroupVersion` is a valid existence-only
query. On a successful version query the output receives the stored group
version. Failure does not promise a zero-initialized version output.
Finding the group entry does not prove its referenced typed values exist.
The group lookup uses the common mapping represented by the session's first
database; callers combining existing files must supply consistent mappings.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group.

`[out, optional] pGroupVersion`

Specify a non-null pointer to receive the version number of the group.  See Behavior.

### Return values

`S_OK` means the group entry exists. `DXGI_ERROR_NOT_FOUND` means it was not
found. Database failures can also be returned; a failure does not initialize
`pGroupVersion` to zero.

<a id="id3d12compilercachesession-findgroupvaluekeys"></a>

## ID3D12CompilerCacheSession::FindGroupValueKeys

### Declaration

```cpp
HRESULT FindGroupValueKeys(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    const UINT* pExpectedGroupVersion,
    D3D12CompilerCacheSessionGroupValueKeysFunc CallbackFunc,
    void* pContext
);
```

### Behavior

Looks up a group entry in the cache whose key exactly matches the provided key and version and provides the list of value keys in the group to a callback.

DXGI_ERROR_NOT_FOUND is returned if a group with pGroupKey and the appropriate version is not found.

Use a nullptr callback to determine if the group is present in the cache without actually retrieving the group keys.  This does not guarantee that the value keys are present in the cache, see [ID3D12CompilerCacheSession::FindGroupValues](#id3d12compilercachesession-findgroupvalues).

`CallbackFunc` may be null to probe the group without retrieving its keys.
The optional expected version is checked in either mode. A missing group or
version mismatch returns `DXGI_ERROR_NOT_FOUND`; the probe does not certify
that the referenced values exist.

With a callback, each key is borrowed only until that invocation returns.
No stable enumeration order across separate calls is specified.
`pExpectedGroupVersion` may be null to omit the version test; otherwise it
supplies the expected `UINT` version, not a field embedded in `pGroupKey`.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group.

`[in, optional] pExpectedGroupVersion`

Optionally specify the version of the Group specified by pGroupKey expected in the database.  Used to fail early without reading the value keys when the version is not the expected version.  In this case of a version mismatch, DXGI_ERROR_NOT_FOUND is returned.

`[in, optional] CallbackFunc`

A D3D12CompilerCacheSessionGroupValueKeysFunc that is called once for each value key in the group.

`[in, out, optional] pContext`

An application specified context pointer that is passed to the callback.  Use this to pass parameters to the callback function that receives the value keys.

### Return values

`S_OK` reports a matching group/key enumeration or null-callback probe;
`DXGI_ERROR_NOT_FOUND` reports a missing group or version mismatch.
Database failures can propagate. A successful key probe does not prove
referenced values exist.

<a id="id3d12compilercachesession-findgroupvalues"></a>

## ID3D12CompilerCacheSession::FindGroupValues

### Declaration

```cpp
HRESULT FindGroupValues(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    const UINT* pExpectedGroupVersion,
    D3D12_COMPILER_VALUE_TYPE_FLAGS ValueTypeFlags,
    D3D12CompilerCacheSessionGroupValuesFunc CallbackFunc,
    void* pContext
);
```

### Behavior

Looks up a group entry in the cache whose key exactly matches the provided key and version and provides the list of values in the group to a callback.

DXGI_ERROR_NOT_FOUND is returned if a group with pGroupKey and the appropriate version is not found.

This method directly retrieves stored matching typed values from the backing databases. It is not equivalent to requiring a successful FindValue for every key returned by FindGroupValueKeys; keys without selected stored values are omitted.

Use a nullptr callback to determine whether matching selected stored values of the group are present without retrieving their buffers.

`ValueTypeFlags` must be nonzero, contain only declared value-type bits, and
select only types assigned to this session. Invalid masks return
`E_INVALIDARG`. `CallbackFunc` may be null to probe without retrieving value
buffers; the same group/version and selected-row tests apply.

This is a sparse enumeration of stored selected typed values. A group can
reference keys A and B while only A has selected stored values. A query of
that backing database can return `S_OK` and enumerate A without reporting B.
An absent selected type does not receive a synthetic callback. Neither a null
callback probe nor `S_OK` certifies every group key or every requested type.

The lookup queries each backing database assigned at least one selected type.
Each backing query returns `S_OK` when it finds at least one matching selected
row, or `DXGI_ERROR_NOT_FOUND` when it finds none, including when the group or
expected version does not match. The overall call fails if any queried backing
database fails. Other backing queries can already have delivered callbacks.
Success in one database does not override a failure in another, and no generic
error-priority or rollback guarantee is established.

`ValueKeyIndex` is a dense rank over the value keys represented by the filtered
selected rows of that backing query, starting at zero. A key with no selected
rows is absent from that rank; changing the type filter or stored rows can
renumber later keys. The index groups typed values within that enumeration.
It does not identify a key across queries, databases or a separate
`FindGroupValueKeys` enumeration.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group.

`[in, optional] pExpectedGroupVersion`

Optionally specify the version of the Group specified by pGroupKey expected in the database.  Used to fail early without reading the values when the version is not the expected version.  In this case of a version mismatch, DXGI_ERROR_NOT_FOUND is returned.

`[in] ValueTypeFlags`

Specify which values are retrieved from the cache session and passed to the D3D12CompilerCacheSessionGroupValuesFunc callback.  Types not included in this flags parameter are not retrieved.  Used as a performance optimization to prevent retrieval of unused values.  See [D3D12_COMPILER_VALUE_TYPE_FLAGS](#d3d12_compiler_value_type_flags).

`[in, optional] CallbackFunc`

A D3D12CompilerCacheSessionGroupValuesFunc that is called once for each selected stored typed value in the group.

`[in, out, optional] pContext`

An application specified context pointer that is passed to the callback.  Use this to pass parameters to the callback function that receives the values.

### Return values

Each queried backing database returns `S_OK` if it has matching selected rows,
or `DXGI_ERROR_NOT_FOUND` if it has none. The overall call succeeds only if
all queried databases succeed; a failing query does not undo callbacks from
other databases. Invalid masks return `E_INVALIDARG`. Other database failures
can propagate. `S_OK` is sparse-enumeration success, not complete-group
certification; a null callback is a legal probe with the same result rules.

<a id="id3d12compilercachesession-findvalue"></a>

## ID3D12CompilerCacheSession::FindValue

### Declaration

```cpp
HRESULT FindValue(
    const D3D12_COMPILER_CACHE_VALUE_KEY* pValueKey,
    D3D12_COMPILER_CACHE_TYPED_VALUE* pTypedValues,
    UINT NumTypedValues,
    D3D12CompilerCacheSessionAllocationFunc pCallbackFunc,
    void* pContext
);
```

### Behavior

Find a previously stored value in the cache.

Each D3D12_COMPILER_CACHE_TYPED_VALUE has a D3D12_COMPILER_CACHE_VALUE.  On input, each D3D12_COMPILER_CACHE_VALUE ValueSize member specifies the size in bytes of the buffer pointed to by pValue. Use zero when querying size or using the allocation callback. On output, the ValueSize member is assigned the size of the value.

When a value is found, for each requested D3D12_COMPILER_CACHE_VALUE:

- If the ValueSize member is a non-zero value, the pValue member must point to a buffer of that size.  If ValueSize is greater than or equal to the value's actual size, the value is copied to pValue and the ValueSize member is assigned the actual size of the value.  If the caller specified ValueSize is insufficient, DXGI_ERROR_MORE_DATA is returned and the required size is assigned to the ValueSize member.
- If the ValueSize member is zero and pCallbackFunc is non-nullptr, pCallbackFunc is called to allocate a buffer and the value is copied into it.  The allocated buffer pointer returned by pCallbackFunc is assigned to the pValue member on return. If pCallbackFunc returns nullptr, the call fails and returns E_OUTOFMEMORY.
- The ValueSize member is assigned the value's actual size before returning.

To determine if a particular value type is stored without retrieving the value, use a size check for the value. To perform a size check, use a nullptr pValue with a zero ValueSize for the types to query and a nullptr pCallbackFunc.  Within one backing-database query, S_OK indicates that at least one requested value type is cached. DXGI_ERROR_NOT_FOUND indicates that none of that query's requested types is cached. The overall call fails if any queried backing database fails; successful output from another database can still be returned.  ValueSize is non-zero for a cached value type.

Example using D3D12CompilerCacheSessionAllocationFunc to retrieve values:

```cpp

const char testKey[] = "TestKey";

const D3D12_COMPILER_CACHE_VALUE_KEY valueKey = { testKey, sizeof(testKey) };
D3D12_COMPILER_CACHE_TYPED_VALUE values[] = { {D3D12_COMPILER_VALUE_TYPE_OBJECT_CODE, {} }, {D3D12_COMPILER_VALUE_TYPE_METADATA, {} } };

HRESULT hr = pCompilerCacheSession->FindValue(
    &valueKey,
    values,
    ARRAYSIZE(values),
    [](size_t SizeInBytes, _Inout_opt_ void* /*pContext*/){return malloc(SizeInBytes);},
    nullptr // No calling context needed
    );

if (SUCCEEDED(hr))
{
    // Each nonzero returned size identifies a corresponding cached typed value.
}

// Use the deallocator that corresponds with allocation even when FindValue does not succeed.  For example, the first
// allocation may have succeeded, but a subsequent call to the allocation function may have failed resulting
// in FindValue returning E_OUTOFMEMORY.
free(values[0].Value.pValue);
free(values[1].Value.pValue);
```

`pValueKey` and the typed-value array must be valid for the call;
`NumTypedValues` counts entries, not bytes. Each requested type must be a
declared type assigned to a database in this session and may occur only once.
Invalid, duplicate or unmapped types return `E_INVALIDARG`. Public API value
sizes are byte counts of type `UINT`. An empty typed list is not a key-existence
probe; request at least one named type to determine whether it is stored.

A nonzero input capacity requires a non-null writable `pValue`. Zero capacity
with no allocator requests only the size. Zero capacity with an allocator
requests allocation even when the input `pValue` is null. The allocator receives
the actual byte count and the caller's `pContext`; the returned pointer and
actual size are assigned to that typed value.

Results are per requested type. A missing type receives size zero. Within one
backing-database query, finding at least one requested type gives `S_OK` if no
capacity, allocation or storage error occurs, even if another requested type
is absent. Finding none of that query's requested types gives
`DXGI_ERROR_NOT_FOUND`; this does not imply that the key has no other types.

The session queries only databases assigned requested types. The overall call
fails if any queried database fails, including a database with none of its
requested types. Other databases can still supply sizes, copied bytes or
allocations. An overall failure does not discard those outputs. There is no
global any-type-anywhere success rule, generic error-priority rule or atomic
all-types output guarantee.

An undersized nonzero buffer returns `DXGI_ERROR_MORE_DATA` for that lookup and
receives the required size without copying that value. Other typed values can
still be copied or allocated. A null allocation result returns `E_OUTOFMEMORY`
but does not reclaim earlier allocations. The caller must free every
allocation it owns, including on failure.

### Parameters

`[in] pValueKey`

A unique sequence of bytes that uniquely identifies an object in the database.

`[in, out] pTypedValues`

An array of typed values to retrieve the size or values of during the operation.  The array size is specified by NumTypedValues.  See [D3D12_COMPILER_CACHE_TYPED_VALUE](#d3d12_compiler_cache_typed_value).

`[in] NumTypedValues`

The number of typed values in the pTypedValues array.

`[in, optional] pCallbackFunc`

An optional callback function for allocating value buffers.  See Behavior.

`[in, out, optional] pContext`

An application specified context pointer that is passed to the callback.  Use this to pass parameters to the allocation callback function.

### Return values

`S_OK`, `DXGI_ERROR_NOT_FOUND`, `DXGI_ERROR_MORE_DATA`, `E_OUTOFMEMORY` and
`E_INVALIDARG` have the per-type meanings above. Other database failures can
propagate. A failing result does not imply untouched buffers, zero sizes or
no successful allocations.

<a id="id3d12compilercachesession-getapplicationdesc"></a>

## ID3D12CompilerCacheSession::GetApplicationDesc

### Declaration

```cpp
const D3D12_APPLICATION_DESC* GetApplicationDesc();
```

### Behavior

Gets the application desc for this cache session.

This information is stored in the database.

The returned value (and contained strings) have the same lifetime as the ID3D12CompilerCacheSession cache session object.

### Parameters

None.

### Return values

Returns a borrowed, read-only pointer to the resolved application descriptor.
The descriptor and its strings remain valid for the cache session's lifetime.
This method has no `HRESULT` or allocation-transfer result.

<a id="id3d12compilercachesession-getcompilertarget"></a>

## ID3D12CompilerCacheSession::GetCompilerTarget

### Declaration

```cpp
D3D12_COMPILER_TARGET GetCompilerTarget();
```

### Behavior

Gets the Compiler Target used with this cache session.

This information is stored in the database.

### Parameters

None.

### Return values

Returns the resolved target by value, including the nonzero resolved ABI and
this factory's local family index. It has no `HRESULT` or caller-owned output
buffer.

<a id="id3d12compilercachesession-getvaluetypes"></a>

## ID3D12CompilerCacheSession::GetValueTypes

### Declaration

```cpp
D3D12_COMPILER_VALUE_TYPE_FLAGS GetValueTypes();
```

### Behavior

Reflects the value types used to create the cache session.

### Parameters

None.

### Return values

Returns the union of value-type masks assigned to the session's database
paths. It does not enumerate which individual values have already been stored
and has no `HRESULT` result.

<a id="id3d12compilercachesession-storevalue"></a>

## ID3D12CompilerCacheSession::StoreValue

### Declaration

```cpp
HRESULT StoreValue(
    const D3D12_COMPILER_CACHE_VALUE_KEY* pValueKey,
    const D3D12_COMPILER_CACHE_TYPED_CONST_VALUE* pTypedValues,
    UINT NumTypedValues
);
```

### Behavior

Add a key/value pair to the compiler cache.

At least one value must be stored in the store operation, so the function returns E_INVALIDARG if pTypedValues is nullptr or NumTypedValues is zero. Each value must have a non-zero size.  Only one value per type may be stored at a time.

This API returns DXGI_ERROR_ALREADY_EXISTS if one of the non-nullptr values with pValueKey already exists.

`pValueKey` identifies a byte sequence; `KeySize` is its byte count, not a
string-character count. The same key can have different typed values.
`NumTypedValues` is an entry count. Each entry must select a declared, mapped
type without duplicates and supply a non-null readable buffer of nonzero byte
size. Invalid typed entries return `E_INVALIDARG`. A null `pTypedValues` or zero
`NumTypedValues` must return `E_INVALIDARG`; neither is a legal empty store.
The buffers remain caller-owned and are borrowed during the call.

An existing key with a previously absent type can receive that type. An
already stored `(key, type)` conflicts with a repeated store even when its
bytes match; this is not an overwrite or success-by-equality operation.
Typed entries are routed to their assigned databases. Failure or duplication
in one database does not roll back stores completed in another.

### Parameters

`[in] pValueKey`

A unique sequence of bytes that uniquely identifies an object in the database.

`[in] pTypedValues`

An array of typed values to store.  The array size is specified by NumTypedValues.  See [D3D12_COMPILER_CACHE_TYPED_CONST_VALUE](#d3d12_compiler_cache_typed_const_value).

`[in] NumTypedValues`

The number of values in the pTypedValues array.

### Return values

`S_OK` reports a successful store. `DXGI_ERROR_ALREADY_EXISTS` reports an
existing typed value; invalid typed arguments return `E_INVALIDARG`.
Storage/allocation failures can propagate. Multi-file partial persistence is
possible on failure; no universal rollback is promised.

<a id="id3d12compilercachesession-storegroupvaluekeys"></a>

## ID3D12CompilerCacheSession::StoreGroupValueKeys

### Declaration

```cpp
HRESULT StoreGroupValueKeys(
    const D3D12_COMPILER_CACHE_GROUP_KEY* pGroupKey,
    UINT GroupVersion,
    const D3D12_COMPILER_CACHE_VALUE_KEY* pValueKeys,
    UINT NumValueKeys
);
```

### Behavior

Stores a group of value keys in the cache.

This method does not check if the value keys in the group are present in the database.

Group keys are application local.  It is only required that each state object's group key is unique to the application.  Each group has a Group Version that is revised to indicate the corresponding state object with this key has changed.  When PSDBs are generated, this version number is stored with the group of cached binaries for this group.

Only one group with this key may be stored.  You cannot store multiple versions of a group in a single PSDB.

`pGroupKey` is required and supplies the application's group-key bytes.
`GroupVersion` is a separate `UINT`. `pValueKeys` supplies `NumValueKeys`
entries, each with its own key byte count; a valid group store requires a
nonempty key list. The buffers remain caller-owned and readable for the call.
The same group key cannot be inserted again with a new version or a different
list; a duplicate returns `DXGI_ERROR_ALREADY_EXISTS`. This call does not
replace existing mappings.

The group mapping is stored in each database entry, including entries storing
different value types. Mapping existence does not validate referenced values.
Failure after writing one database can leave other mappings unchanged;
the call does not establish an all-files transaction or repair mismatched
mappings across existing files.

### Parameters

`[in] pGroupKey`

A unique key that identifies the group.

`[in] GroupVersion`

The version number of the cached group.

`[in] pValueKeys`

An array of variable size keys in the group.  Each D3D12_COMPILER_CACHE_VALUE_KEY contains its own KeySize.

`[in] NumValueKeys`

The number of value keys in the group.

### Return values

`S_OK` reports successful mapping storage. An existing group key returns
`DXGI_ERROR_ALREADY_EXISTS`, irrespective of the proposed new version.
Invalid lists or database failures can fail the call. Values need not exist,
and failure does not imply a rollback of other databases.

<a id="invoked-ddi"></a>

## Compiler and cache DDI behavior

The [published compiler table](compiler-plugin-foundation.md#compiler-table-dependencies),
[cache callback table](compiler-plugin-foundation.md#callback-dependencies), and
[handles](compiler-plugin-foundation.md#handles-and-versions) are defined once
in the plugin foundation. The behavior below uses those declarations and their
dependent types. The negotiated version selects the applicable ABI; the
published build-2 add-operation uses `D3D12DDIARG_ADD_TO_STATE_OBJECT_0072`,
not an extra standalone parent-handle parameter.

The runtime's `D3D12DDI_HRTCOMPILERCACHESESSION` identifies one compile call.
It is valid only during that call and must not be retained for later callbacks.
The plugin compiler object (`D3D12DDI_HCOMPILER`) is distinct from that
call-scoped handle. `D3D12DDI_HCOMPILERSTATEOBJECT` retains plugin compilation
state between creation and later additions; it is not a GPU execution handle.

The runtime installs the
[cache callback table](compiler-plugin-foundation.md#callback-dependencies)
before creating compiler objects, through
[`PFND3D12DDI_COMPILER_SET_CALLBACK_DDI_TABLE`](compiler-plugin-foundation.md#set-callback-table).
The published table, callbacks and enum tags/typedef aliases define the
ABI. The `APIENTRY`, `CALLBACK` and `__stdcall` conventions are part of the
function-pointer types.

The DDI value types select object code, metadata, Debug PDB and performance
data with the same semantic categories as the
[API output types](#d3d12_compiler_value_type). Keys and buffers have the
published DDI widths; API values have the widths in the fixed API declarations.
Code must not infer binary layout compatibility from the semantic mapping.

<a id="d3d12ddi_compiler_cache_value_type"></a>

### D3D12DDI_COMPILER_CACHE_VALUE_TYPE

**Declaration:** [D3D12DDI_COMPILER_CACHE_VALUE_TYPE](compiler-plugin-foundation.md#callback-dependencies).

A type enumeration used when only a single value type may be selected.

**Constants**

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_OBJECT_CODE*

Specifies object code.  The compiled executable code that is run on GPU.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_METADATA*

Metadata that the compiler may provide about the compile.  For example it may store a compiler version number used for validation.  Such a thing should not be put inside of the object code as users of this system expect to be able to do memcmp diffing between compiler versions.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_DEBUG_PDB*

The Debug PDB for the object code.  Used in pix and other debug scenarios.  An opaque blob that still requires IHV interpretation to be used with PIX.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_PERFORMANCE_DATA*

Performance data about the compile or produced object code.  An opaque blob that still requires IHV interpretation to be used with PIX.

**Remarks**

Used when defining a typed value, see [D3D12DDI_COMPILER_CACHE_TYPED_VALUE](compiler-plugin-foundation.md#callback-dependencies).

<a id="d3d12ddi_compiler_value_type_flags"></a>

### D3D12DDI_COMPILER_VALUE_TYPE_FLAGS

**Declaration:** [D3D12DDI_COMPILER_VALUE_TYPE_FLAGS](compiler-plugin-foundation.md#callback-dependencies).

A flags enumeration used where multiple values may be selected.

**Constants**

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_FLAG_NONE*

No selected value types.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_FLAG_OBJECT_CODE*

Indicates object code, see D3D12DDI_COMPILER_CACHE_VALUE_TYPE_OBJECT_CODE.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_FLAG_METADATA*

Selects compiler metadata.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_FLAG_DEBUG_PDB*

Indicates Debug PDB, see D3D12DDI_COMPILER_CACHE_VALUE_TYPE_DEBUG_PDB.

*D3D12DDI_COMPILER_CACHE_VALUE_TYPE_FLAG_PERFORMANCE_DATA*

Indicates Performance Data, see D3D12DDI_COMPILER_CACHE_VALUE_TYPE_PERFORMANCE_DATA.

**Remarks**

Example uses:

- Used for selecting which value types must be loaded/stored during [PFND3D12DDI_COMPILER_COMPILE_PIPELINE_STATE](#pfnd3d12ddi_compiler_compile_pipeline_state) and other compile operations.

<a id="d3d12ddi_compiler_cache_value_key"></a>

### D3D12DDI_COMPILER_CACHE_VALUE_KEY

**Declaration:** [D3D12DDI_COMPILER_CACHE_VALUE_KEY](compiler-plugin-foundation.md#callback-dependencies).

**Members**

*pKey*

A unique sequence of bytes that uniquely identifies an object in the database.

*KeySize*

The size in bytes of pKey.

**Remarks**

Used during cache operations like find and store value to identify a value.

<a id="d3d12ddi_compiler_cache_value"></a>

### D3D12DDI_COMPILER_CACHE_VALUE

**Declaration:** [D3D12DDI_COMPILER_CACHE_VALUE](compiler-plugin-foundation.md#callback-dependencies).

Specifies a non-const value buffer and buffer size.

**Members**

*pValue*

A pointer to a memory buffer containing the value.

*ValueSize*

The size of the pValue buffer in bytes.

**Remarks**

See also [D3D12DDI_COMPILER_CACHE_TYPED_VALUE](compiler-plugin-foundation.md#callback-dependencies).

<a id="d3d12ddi_compiler_cache_typed_value"></a>

### D3D12DDI_COMPILER_CACHE_TYPED_VALUE

**Declaration:** [D3D12DDI_COMPILER_CACHE_TYPED_VALUE](compiler-plugin-foundation.md#callback-dependencies).

Specifies a value and its type, such as object code, metadata etc.

**Members**

*Type*

The type of the value, such as object code, metadata, etc.  See [D3D12DDI_COMPILER_CACHE_VALUE_TYPE](#d3d12ddi_compiler_cache_value_type) for more information on types.

*Value*

The buffer and size for the value.  See [D3D12DDI_COMPILER_CACHE_VALUE](#d3d12ddi_compiler_cache_value).

**Remarks**

Used in DDI where the compiler may modify the value buffer, such as [PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB](#pfnd3d12ddi_compiler_cache_find_value_cb).

<a id="d3d12ddi_compiler_cache_const_value"></a>

### D3D12DDI_COMPILER_CACHE_CONST_VALUE

**Declaration:** [D3D12DDI_COMPILER_CACHE_CONST_VALUE](compiler-plugin-foundation.md#callback-dependencies).

Specifies a const value buffer and buffer size.

**Members**

*pValue*

A const pointer to a memory buffer containing the value.

*ValueSize*

The size of the pValue buffer in bytes.

**Remarks**

See also [D3D12DDI_COMPILER_CACHE_TYPED_CONST_VALUE](compiler-plugin-foundation.md#callback-dependencies).

<a id="d3d12ddi_compiler_cache_typed_const_value"></a>

### D3D12DDI_COMPILER_CACHE_TYPED_CONST_VALUE

**Declaration:** [D3D12DDI_COMPILER_CACHE_TYPED_CONST_VALUE](compiler-plugin-foundation.md#callback-dependencies).

**Members**

*Type*

The type of the value, such as object code, metadata, etc.  See [D3D12DDI_COMPILER_CACHE_VALUE_TYPE](#d3d12ddi_compiler_cache_value_type) for more information on types.

*Value*

The const buffer and size for the value.  See [D3D12DDI_COMPILER_CACHE_CONST_VALUE](#d3d12ddi_compiler_cache_const_value).

**Remarks**

Used in DDI where the compiler may not modify the value buffer, such as [PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB](#pfnd3d12ddi_compiler_cache_store_value_cb).

<a id="d3d12ddi_compiler_cache_callbacks"></a>

### D3D12DDI_COMPILER_CACHE_CALLBACKS

**Declaration:** [D3D12DDI_COMPILER_CACHE_CALLBACKS](compiler-plugin-foundation.md#callback-dependencies).

**Members**

*pfnCompilerCacheFindValue*

See [PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB](#pfnd3d12ddi_compiler_cache_find_value_cb).

*pfnCompilerCacheStoreValue*

See [PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB](#pfnd3d12ddi_compiler_cache_store_value_cb).

*pfnCompilerCacheSetObjectValueKeys*

See [PFND3D12DDI_COMPILER_CACHE_SET_OBJECT_VALUE_KEYS_CB](#pfnd3d12ddi_compiler_cache_set_object_value_keys_cb).

<a id="d3d12ddi_hrtcompilercachesession"></a>

### D3D12DDI_HRTCOMPILERCACHESESSION

The runtime's compiler cache session object.

**Remarks**

This object is supplied during a compilation operation and must be used to call D3D12DDI_COMPILER_CACHE_CALLBACKS.  A given cache session handle is only valid and has the lifetime of a single compile DDI call.

<a id="d3d12ddi_hcompiler"></a>

### D3D12DDI_HCOMPILER

A compiler object is instanced from an adapter family and application identity.  It is used to compile state objects and store them in a cache session.

A handle type representing the plugin's compiler object.

<a id="d3d12ddi_hrtcompiler"></a>

### D3D12DDI_HRTCOMPILER

A handle type representing the runtime compiler object.  Allows compiler plugins to call back into runtime.

<a id="d3d12ddi_hcompilerstateobject"></a>

### D3D12DDI_HCOMPILERSTATEOBJECT

A handle type representing the compiler plugin's state object.  This object type allows compiler plugins to store state between Create and AddTo state object calls.

<a id="pfnd3d12ddi_compiler_calc_private_compiler_size"></a>

### PFND3D12DDI_COMPILER_CALC_PRIVATE_COMPILER_SIZE

**Declaration:** [PFND3D12DDI_COMPILER_CALC_PRIVATE_COMPILER_SIZE](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 356–360](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L356-L360).

Determine the size of the driver compiler object.

**Parameters**

*pTarget*

Contains the adapter family and ABI version.  See [D3D12DDI_COMPILER_TARGET](compiler-plugin-foundation.md#ddi-target-descriptions).

*pApplicationDesc*

Describes the target application and version.  See [D3D12DDI_APPLICATION_DESC](compiler-plugin-foundation.md#d3d12ddi_application_desc).

**Remarks**

The memory for storing the plugin's CPU object representing the compiler is allocated by the caller. This method is used to calculate the plugin's object size.

<a id="pfnd3d12ddi_compiler_create_compiler"></a>

### PFND3D12DDI_COMPILER_CREATE_COMPILER

**Declaration:** [PFND3D12DDI_COMPILER_CREATE_COMPILER](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 362–368](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L362-L368).

Create the driver compiler object.

**Parameters**

*pTarget*

Contains the adapter family and ABI version.  See [D3D12DDI_COMPILER_TARGET](compiler-plugin-foundation.md#ddi-target-descriptions).

*pApplicationDesc*

Describes the target application and version.  See [D3D12DDI_APPLICATION_DESC](compiler-plugin-foundation.md#d3d12ddi_application_desc).

*hCompiler*

Handle for plugin compiler object.  Memory is allocated by the caller, and plugin places compiler object.

`hRTCompiler` is the runtime compiler handle supplied to the plugin.
The size query and create/destroy entries also receive the owning
`hCompilerDDI` in the published ABI. The runtime allocates private object
storage using the reported byte size; destruction ends the placed plugin
object before that storage is reclaimed. The DDI create operation returns
an `HRESULT`; a failed plugin creation fails compiler creation.

<a id="pfnd3d12ddi_compiler_destroy_compiler"></a>

### PFND3D12DDI_COMPILER_DESTROY_COMPILER

**Declaration:** [PFND3D12DDI_COMPILER_DESTROY_COMPILER](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 370–373](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L370-L373).

Destroy the plugin object placed in the caller-provided private allocation. The caller reclaims that allocation after placement destruction.

**Parameters**

*hCompiler*

Handle for plugin compiler object.

<a id="pfnd3d12ddi_compiler_compile_pipeline_state"></a>

### PFND3D12DDI_COMPILER_COMPILE_PIPELINE_STATE

**Declaration:** [PFND3D12DDI_COMPILER_COMPILE_PIPELINE_STATE](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 375–380](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L375-L380).

Compile a graphics or compute pipeline state.

**Parameters**

*hCompiler*

Handle for plugin compiler object.

*hrtCompilerCacheSession*

The compiler cache session callback handle.  Used with cache operations to find/store values and set the values associated with the object being compiled.  See [Cache Session Callbacks](#pfnd3d12ddi_compiler_cache_find_value_cb).

*ValueTypeFlags*

Specify which values types to store in the cache session after compilation succeeds.  See [D3D12DDI_COMPILER_VALUE_TYPE_FLAGS](compiler-plugin-foundation.md#callback-dependencies).

*pPipelineStateArg*

Creation arguments for the pipeline state object to compile.

**Remarks**

Compiled output is stored via the [PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB](#pfnd3d12ddi_compiler_cache_store_value_cb) callback.  Check for previously compiled duplicates via the [PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB](#pfnd3d12ddi_compiler_cache_find_value_cb) callback.  D3D12DDI_COMPILER_CACHE_CALLBACKS are installed before compiler creation through PFND3D12DDI_COMPILER_SET_CALLBACK_DDI_TABLE.

Compiler is expected to sub-divide objects in an IHV dependent way to support deduplication.  Multiple values may be stored per compile.

On success, this method must not return until all compiled output is stored.

The driver must call [PFND3D12DDI_COMPILER_CACHE_SET_OBJECT_VALUE_KEYS_CB](#pfnd3d12ddi_compiler_cache_set_object_value_keys_cb) once before returning success to set the values associated with this pipeline state object definition.

<a id="pfnd3d12ddi_compiler_calc_private_state_object_size"></a>

### PFND3D12DDI_COMPILER_CALC_PRIVATE_STATE_OBJECT_SIZE

**Declaration:** [PFND3D12DDI_COMPILER_CALC_PRIVATE_STATE_OBJECT_SIZE](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 382–385](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L382-L385).

Calculate the size needed for compiler's state object implementation.

**Parameters**

*hCompiler*

Handle for plugin compiler object.

*pCreateStateObject*

Creation arguments for the state object.

<a id="pfnd3d12ddi_compiler_compile_create_state_object"></a>

### PFND3D12DDI_COMPILER_COMPILE_CREATE_STATE_OBJECT

**Declaration:** [PFND3D12DDI_COMPILER_COMPILE_CREATE_STATE_OBJECT](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 387–393](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L387-L393).

**Parameters**

*hCompiler*

Handle for plugin compiler object.

*hrtCompilerCacheSession*

The compiler cache session callback handle.  Used with cache operations to find/store values and set the values associated with the object being compiled.  See [Cache Session Callbacks](#pfnd3d12ddi_compiler_cache_find_value_cb).

*ValueTypeFlags*

Specify which values types to store in the cache session after compilation succeeds.  See [D3D12DDI_COMPILER_VALUE_TYPE_FLAGS](compiler-plugin-foundation.md#callback-dependencies).

*pCreateStateObject*

Creation arguments for the state object.

*hStateObject*

A handle pointing to the pre-allocated space for the compiler object.  Compiler places the object in this allocation and uses it to store state to support future calls to [PFND3D12DDI_COMPILER_COMPILE_ADD_TO_STATE_OBJECT](#pfnd3d12ddi_compiler_compile_add_to_state_object) calls.

**Remarks**

The lifetime of pCreateStateObject is guaranteed until hStateObject is destroyed; therefore, compiler only needs to store a pointer to this structure to support future AddToStateObject calls.  Compiler must store other state needed for future compilation with hStateObject, such as keys to lookup stored values.

Compiled output is stored via the [PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB](#pfnd3d12ddi_compiler_cache_store_value_cb) callback.  Check for previously compiled duplicates via the [PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB](#pfnd3d12ddi_compiler_cache_find_value_cb) callback.  D3D12DDI_COMPILER_CACHE_CALLBACKS are installed before compiler creation through PFND3D12DDI_COMPILER_SET_CALLBACK_DDI_TABLE.

Compiler is expected to sub-divide objects in an IHV dependent way to support deduplication.  Multiple values may be stored per compile.

On success, this method must not return until all compiled output is stored.

The driver must call [PFND3D12DDI_COMPILER_CACHE_SET_OBJECT_VALUE_KEYS_CB](#pfnd3d12ddi_compiler_cache_set_object_value_keys_cb) once before returning success to set the values associated with this state object definition. A collection CompileStateObject that produces no compiled output may omit this callback and have no group.

A no-output collection may consequently have no output group. The exception
does not apply to pipeline-state compilation, executable state-object
compilation or additions.

<a id="pfnd3d12ddi_compiler_calc_private_add_to_state_object_size"></a>

### PFND3D12DDI_COMPILER_CALC_PRIVATE_ADD_TO_STATE_OBJECT_SIZE

**Declaration:** [PFND3D12DDI_COMPILER_CALC_PRIVATE_ADD_TO_STATE_OBJECT_SIZE](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 395–398](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L395-L398).

Calculate the size needed for compiler's state object implementation when adding to an existing state object.

**Parameters**

*hCompiler*

Handle for plugin compiler object.

*pAddToStateObject*

Creation arguments for the add operation.

The parent handle is carried in pAddToStateObject->StateObjectToGrowFrom.

<a id="pfnd3d12ddi_compiler_compile_add_to_state_object"></a>

### PFND3D12DDI_COMPILER_COMPILE_ADD_TO_STATE_OBJECT

**Declaration:** [PFND3D12DDI_COMPILER_COMPILE_ADD_TO_STATE_OBJECT](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 400–406](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L400-L406).

**Parameters**

*hCompiler*

Handle for plugin compiler object.

*hrtCompilerCacheSession*

The compiler cache session callback handle.  Used with cache operations to find/store values and set the values associated with the object being compiled.  See [Cache Session Callbacks](#pfnd3d12ddi_compiler_cache_find_value_cb).

*ValueTypeFlags*

Specify which values types to store in the cache session after compilation succeeds.  See [D3D12DDI_COMPILER_VALUE_TYPE_FLAGS](compiler-plugin-foundation.md#callback-dependencies).

*pAddToStateObject*

Creation arguments for the add operation.

The parent handle is carried in pAddToStateObject->StateObjectToGrowFrom.

*hStateObject*

A handle pointing to the pre-allocated space for the compiler object.  Compiler places the object in this allocation and uses it to store state to support future AddToStateObject calls.

**Remarks**

The pAddToStateObject description remains valid until hStateObject is destroyed, so the compiler can retain a pointer to that description for later additions. The StateObjectToGrowFrom plugin handle need not remain alive with the child; the plugin must retain the dependency state it needs.  Compiler must store other state needed for future compilation with hStateObject, such as keys to lookup stored values.

Compiled output is stored via PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB.  Check for previously compiled duplicates via PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB.

Compiler is expected to sub-divide objects in an IHV dependent way to support deduplication.  Multiple values may be stored per compile.

On success, this method must not return until all compiled output is stored.

The driver must call [PFND3D12DDI_COMPILER_CACHE_SET_OBJECT_VALUE_KEYS_CB](#pfnd3d12ddi_compiler_cache_set_object_value_keys_cb) once before returning success to set the values associated with this state object definition.

<a id="pfnd3d12ddi_compiler_destroy_state_object"></a>

### PFND3D12DDI_COMPILER_DESTROY_STATE_OBJECT

**Declaration:** [PFND3D12DDI_COMPILER_DESTROY_STATE_OBJECT](compiler-plugin-foundation.md#compiler-table-dependencies); [published header lines 408–411](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L408-L411).

Destroys the compiler state object placed in the caller-provided private allocation. The caller reclaims the allocation after the plugin destroys its placed object.

**Parameters**

*hCompiler*

Handle for plugin compiler object.

*hStateObject*

The state object to destroy.

<a id="pfnd3d12ddi_compiler_cache_find_value_cb"></a>

### PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB

**Declaration:** [PFND3D12DDI_COMPILER_CACHE_FIND_VALUE_CB](compiler-plugin-foundation.md#callback-dependencies); [published header lines 236–243](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L236-L243).

Find a previously stored value in the cache.

**Parameters**

*hrtCompilerCache*

The compiler cache session  callback handle received during a compilation operation.

*pValueKey*

The unique identifier for the value.  See [D3D12DDI_COMPILER_CACHE_VALUE_KEY](compiler-plugin-foundation.md#callback-dependencies).

*pValues*

An array of typed values to retrieve the size or values of during the operation.  The array size is specified by NumValues.  See [D3D12DDI_COMPILER_CACHE_TYPED_VALUE](compiler-plugin-foundation.md#callback-dependencies).

*NumValues*

The number of values in the pValues array.

*pCallbackFunc*

An optional callback function for allocating value buffers.  See Remarks.

*pContext*

A compiler plugin specified context pointer that is passed to the callback.  Use this to pass parameters to the allocation callback function.

**Remarks**

Each D3D12DDI_COMPILER_CACHE_TYPED_VALUE has a D3D12DDI_COMPILER_CACHE_VALUE.  On input, each D3D12DDI_COMPILER_CACHE_VALUE ValueSize member specifies the size in bytes of the buffer pointed to by pValue. Use zero when querying size or using the allocation callback. On output, the ValueSize member is assigned the size of the value.

When a value is found, for each requested D3D12DDI_COMPILER_CACHE_VALUE:

- If the ValueSize member is a non-zero value, the pValue member must point to a buffer of that size.  If ValueSize is greater than or equal to the value's actual size, the value is copied to pValue and the ValueSize member is assigned the actual size of the value.  If the caller specified ValueSize is insufficient, DXGI_ERROR_MORE_DATA is returned and the required size is assigned to the ValueSize member.
- If the ValueSize member is zero and pCallbackFunc is non-nullptr, pCallbackFunc is called to allocate a buffer and the value is copied into it.  The allocated buffer pointer returned by pCallbackFunc is assigned to the pValue member on return. If pCallbackFunc returns nullptr, the call fails and returns E_OUTOFMEMORY.
- The ValueSize member is assigned the value's actual size before returning.

To determine if a particular value type is stored without retrieving the value, use a size check for the value. To perform a size check, use a nullptr pValue with a zero ValueSize for the types to query and a nullptr pCallbackFunc.  Within one backing-database query, S_OK indicates that at least one requested value type is cached. DXGI_ERROR_NOT_FOUND indicates that none of that query's requested types is cached. The overall call fails if any queried backing database fails; successful output from another database can still be returned.  ValueSize is non-zero for a cached value type.

The typed-size, short-buffer, missing-type, allocator and partial-output rules
are the corresponding
[`FindValue` contract](#id3d12compilercachesession-findvalue), applied through the
published DDI structures and calling conventions. A zero-sized lookup with
no allocator is a per-type size probe, not a complete-group check.
`PFND3D12DDI_COMPILER_ALLOCATION` must return suitably aligned storage or null
and must not let exceptions escape; successful allocations remain the plugin
caller's responsibility, including after a later lookup failure.

<a id="pfnd3d12ddi_compiler_cache_store_value_cb"></a>

### PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB

**Declaration:** [PFND3D12DDI_COMPILER_CACHE_STORE_VALUE_CB](compiler-plugin-foundation.md#callback-dependencies); [published header lines 245–250](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L245-L250).

Add a key/value pair to the compiler cache.

**Parameters**

*hrtCompilerCacheSession*

The compiler cache session  callback handle received during a compilation operation.

*pValueKey*

The unique identifier for the value.  See [D3D12DDI_COMPILER_CACHE_VALUE_KEY](compiler-plugin-foundation.md#callback-dependencies).

*pValues*

An array of typed values to store.  The array size is specified by NumValues.  See [D3D12DDI_COMPILER_CACHE_TYPED_CONST_VALUE](compiler-plugin-foundation.md#callback-dependencies).

*NumValues*

The number of values in the pValues array.

**Remarks**

At least one value must be stored in the store operation, so the function returns E_INVALIDARG if pValues is nullptr or NumValues is zero.  Each value must have a non-zero size.  Only one value per type may be stored at a time.

This callback returns DXGI_ERROR_ALREADY_EXISTS if one of the non-nullptr values with pValueKey already exists.

Flags during the compile operation indicate which values must be stored.  See [D3D12DDI_COMPILER_VALUE_TYPE_FLAGS](compiler-plugin-foundation.md#callback-dependencies).  Values are optionally stored to accommodate various scenarios such as generating PDB for existing object code with a matching compiler and source or Shader Explorer only needing the performance data output.

Value types requested by the compile mask must be made available through the
cache callbacks before successful completion. Types can be stored in separate
calls; deduplication can reuse a previously stored value. The callback reports
storage conflicts and failures through its `HRESULT`. A successful store
copies the supplied data; it does not take ownership of the plugin's buffer.
The plugin must handle a failed store rather than treating it as evidence that
all requested output was persisted. Earlier stores need not be undone.

<a id="pfnd3d12ddi_compiler_cache_set_object_value_keys_cb"></a>

### PFND3D12DDI_COMPILER_CACHE_SET_OBJECT_VALUE_KEYS_CB

**Declaration:** [PFND3D12DDI_COMPILER_CACHE_SET_OBJECT_VALUE_KEYS_CB](compiler-plugin-foundation.md#callback-dependencies); [published header lines 252–256](../../spec-maintenance/d3d12/references/compiler-foundation-wdk-10.0.28000.2526.h#L252-L256).

A callback to set the list of value keys associated with an object.

**Parameters**

*hrtCompilerCacheSession*

The compiler cache session  callback handle received during a compilation operation.

*pValueKeys*

The unique identifiers for the values.  See [D3D12DDI_COMPILER_CACHE_VALUE_KEY](compiler-plugin-foundation.md#callback-dependencies).

*NumValueKeys*

The number of keys specified by pValueKeys.

**Remarks**

This stored list specifies the values that the driver receives during title cooperative scenarios.  It may also be used for other database management operations to map values back to state object definitions.  The compiler plugin must call this function once before returning success from a compile operation. A collection CompileStateObject that produces no compiled output may omit this callback and have no group.

`pValueKeys` supplies vendor-generated keys for the object compiled by this
call. The callback associates the list with the API application group key and
`GroupVersion`. Registration does not validate that every requested typed value
exists. The cache-session handle is valid only during the active compile call;
the plugin must not retain it for later callbacks.

An already-existing group result is accepted by the compiler runtime for this
callback; other group-store failures are returned to the plugin. A callback
invocation alone does not prove that a new mapping or every typed value was
stored. Synchronous successful completion is not a transaction or universal
error-propagation guarantee.

<a id="compilation-example"></a>

## Compilation example

This example demonstrates how to compile a Pipeline State Object (PSO) and store it in a Precompiled Shader Database (PSDB) using the State Object Compiler API.

```cpp
    //
    // Create the factory
    //

    Microsoft::WRL::ComPtr<ID3D12CompilerFactory> spFactory;
    VERIFY_SUCCEEDED(D3D12CompilerCreateFactory(CompilerPluginDll, IID_PPV_ARGS(&spFactory)));

    //
    // Cache output path and type
    //

    D3D12_COMPILER_VALUE_TYPE_FLAGS compileValues =
        D3D12_COMPILER_VALUE_TYPE_FLAGS_OBJECT_CODE | D3D12_COMPILER_VALUE_TYPE_FLAGS_METADATA;

    D3D12_COMPILER_DATABASE_PATH databasePath = { compileValues, L"out.psdb" };

    //
    // Target adapter family and ABI
    //

    // Use the first available adapter family.
    UINT AdapterFamilyIndex = 0;

    // Zero targets the latest available ABI version for this adapter family.
    UINT64 ABIVersion = 0;

    const D3D12_COMPILER_TARGET target =
    {
        AdapterFamilyIndex,
        ABIVersion
    };

    //
    //  Application metadata
    //

    constexpr D3D12_APPLICATION_DESC applicationDesc =
    {
        L"Sample.exe",                                  // pExeFilename
        L"Sample Application Title",                    // pName
        {1},                                            // Version
        L"Sample Engine Name",                          // pEngineName
        {7}                                             // EngineVersion
    };

    //
    // Create the Cache Session and Compiler
    //

    Microsoft::WRL::ComPtr<ID3D12CompilerCacheSession> spCompilerCacheSession;
    VERIFY_SUCCEEDED(spFactory->CreateCompilerCacheSession(
        &databasePath,
        1u, // NumPaths
        &target,
        &applicationDesc,
        IID_PPV_ARGS(&spCompilerCacheSession)
    ));

    Microsoft::WRL::ComPtr<ID3D12Compiler> spCompiler;
    VERIFY_SUCCEEDED(spFactory->CreateCompiler(
        spCompilerCacheSession.Get(),
        IID_PPV_ARGS(&spCompiler)));

    //
    // Prepare the PSO arguments.
    //

    const D3D12_INPUT_ELEMENT_DESC DefaultILDescs[1] =
    {
        { "POS", 0, DXGI_FORMAT_R32G32B32A32_FLOAT, 0, 0, D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA, 0 }
    };

    const D3D12_INPUT_LAYOUT_DESC DefaultIL = { DefaultILDescs, _countof(DefaultILDescs) };

    CD3DX12_DESCRIPTOR_RANGE1 DescRange[] =
    {
        {D3D12_DESCRIPTOR_RANGE_TYPE_SRV, 1, 0},    // t0
        {D3D12_DESCRIPTOR_RANGE_TYPE_UAV, 2, 0},    // u0-u1
        {D3D12_DESCRIPTOR_RANGE_TYPE_SAMPLER, 2, 0} // s0-s1
    };

    CD3DX12_ROOT_PARAMETER1 rootParameters[_countof(DescRange)];

    for (UINT i = 0; i < _countof(rootParameters); ++i)
    {
        rootParameters[i].InitAsDescriptorTable(1u, &DescRange[i], D3D12_SHADER_VISIBILITY_ALL);
    }

    CD3DX12_VERSIONED_ROOT_SIGNATURE_DESC rootSignatureDesc(_countof(rootParameters), rootParameters, 0, nullptr,
        D3D12_ROOT_SIGNATURE_FLAG_ALLOW_INPUT_ASSEMBLER_INPUT_LAYOUT | D3D12_ROOT_SIGNATURE_FLAG_ALLOW_STREAM_OUTPUT);

    Microsoft::WRL::ComPtr <ID3DBlob> signature;
    Microsoft::WRL::ComPtr <ID3DBlob> error;
    VERIFY_SUCCEEDED(D3D12CompilerSerializeVersionedRootSignature(&rootSignatureDesc, &signature, &error));

    D3D12_SERIALIZED_ROOT_SIGNATURE_DESC serializedRootSignatureDesc = {};
    serializedRootSignatureDesc.pSerializedBlob = signature->GetBufferPointer();
    serializedRootSignatureDesc.SerializedBlobSizeInBytes = signature->GetBufferSize();

    struct PSO_STREAM
    {
        CD3DX12_PIPELINE_STATE_STREAM_PRIMITIVE_TOPOLOGY PrimitiveTopologyType;
        CD3DX12_PIPELINE_STATE_STREAM_INPUT_LAYOUT InputLayout;
        CD3DX12_PIPELINE_STATE_STREAM_VS VS;
        CD3DX12_PIPELINE_STATE_STREAM_PS PS;
        CD3DX12_PIPELINE_STATE_STREAM_SERIALIZED_ROOT_SIGNATURE RootSigDesc;
    } PSOStream =
    {
        D3D12_PRIMITIVE_TOPOLOGY_TYPE_TRIANGLE,
        DefaultIL,
        AssignShaderBytecode(pVS),
        AssignShaderBytecode(pPS),
        CD3DX12_SERIALIZED_ROOT_SIGNATURE_DESC{signature->GetBufferPointer(), signature->GetBufferSize()}
    };

    D3D12_PIPELINE_STATE_STREAM_DESC StreamDesc{ sizeof(PSOStream), &PSOStream };

    //
    //  Compile the PSO and store the results with the cache session.
    //

    const UINT groupVersion = 1u;
    const UINT keySize = static_cast<UINT>(strlen(pGroupName) + 1u);
    D3D12_COMPILER_CACHE_GROUP_KEY groupKey = { pGroupName, keySize };
    VERIFY_SUCCEEDED(spCompiler->CompilePipelineState(&groupKey, groupVersion, &StreamDesc));
```
