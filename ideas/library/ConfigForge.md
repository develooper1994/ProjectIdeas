# [ConfigForge](https://github.com/develooper1994/configforge)

**ConfigForge** is a cross-platform configuration resolution and management framework designed to make application configuration easy to consume without making configuration behavior implicit.

It provides a unified abstraction over environment variables, command-line arguments, configuration files, build-time values, OS-native configuration systems, runtime state, and user-defined configuration sources.

The project is designed around a simple principle:

> **Make configuration easy without making configuration behavior implicit.**

ConfigForge should provide strong defaults for common applications while remaining deterministic, explicit, extensible, and suitable for performance-sensitive systems ranging from embedded devices to desktop and server applications.

---

## 1. Motivation

Application configuration is usually distributed across several unrelated mechanisms:

* Environment variables
* Command-line arguments
* Multiple configuration files
* User-level configuration
* System-level configuration
* OS-native configuration stores
* Build-time definitions
* Runtime configuration
* Application defaults

Each mechanism has its own API, precedence rules, data representation, validation behavior, and write semantics.

Applications therefore tend to accumulate configuration-specific code such as:

* manual environment variable parsing
* command-line parsing
* configuration file loading
* precedence handling
* type conversion
* validation
* fallback logic
* runtime overrides
* file writing
* reload/watch logic
* platform-specific configuration access
* debugging code explaining where a value came from

ConfigForge aims to provide a single configuration model for these concerns while keeping the underlying sources independent.

---

## 2. Design Goals

### 2.1 Simplicity

The common case should require very little code.

A typical application should be able to define sources and simply ask:

```cpp
auto port = config.get<int>("server.port");
```

The application should not need to manually implement resolution logic.

### 2.2 Deterministic Resolution

Given the same:

* definitions
* source values
* capabilities
* policies
* runtime state

the resolver must produce the same result.

Implicit behavior based on registration order, filesystem discovery order, or object creation order should not affect resolution.

### 2.3 Explicit Mutation

Reading configuration must never implicitly modify a source.

Runtime mutation and persistent source mutation are separate operations.

```text
get()
    → read resolved state

set()
    → modify runtime state

write()
    → explicitly modify selected source(s)

propagate()
    → explicitly apply runtime state to policy-selected source(s)
```

### 2.4 Performance

Convenience must not require sacrificing runtime performance.

Configuration loading, parsing, policy compilation, and resolution should be separated from the hot path.

The intended runtime path is:

```text
Source Data
    ↓
Policy
    ↓
Resolver
    ↓
Immutable Snapshot
    ↓
Fast get()
```

Repeated `get()` operations should not perform file I/O, configuration parsing, or policy compilation.

### 2.5 Extensibility

Users should be able to add their own configuration sources without modifying ConfigForge.

Examples include:

* Remote configuration
* Database-backed configuration
* Device-specific configuration
* Company-specific configuration systems
* Custom embedded-system storage
* Application-specific configuration services

### 2.6 Cross-platform Design

The core should not depend on a particular operating system.

Platform-specific functionality should be implemented as optional providers.

### 2.7 Static Linking

The complete library must be statically linkable.

Optional providers and features should not become mandatory dependencies.

### 2.8 No Unnecessary Requirements

ConfigForge should provide useful defaults and should not force users to configure concepts they do not need.

An advanced configuration system must not require advanced configuration to use it.

---

## 3. Core Conceptual Model

ConfigForge is built around several distinct responsibilities.

```text
Definition
    ↓
Source
    ↓
Policy
    ↓
Resolver
    ↓
Snapshot
    ↓
Application
```

Mutation follows a separate path:

```text
Application
    ↓
set()
    ↓
Runtime State

Application
    ↓
write()/propagate()
    ↓
Materializer
    ↓
Writer
    ↓
Source
```

The major abstractions are intentionally separated because they answer different questions.

| Abstraction       | Responsibility                                                     |
| ----------------- | ------------------------------------------------------------------ |
| `Definition`      | What is this configuration parameter?                              |
| `Value`           | What typed value does it contain?                                  |
| `Source`          | Where can configuration data come from or go to?                   |
| `SourceProvider`  | How is a type of source implemented?                               |
| `SourceInstance`  | Which concrete source is being used?                               |
| `Capability`      | What can a source technically do?                                  |
| `Policy`          | What is ConfigForge allowed to do and which source has precedence? |
| `PolicyLayer`     | Which group of policy rules belongs together?                      |
| `PolicySource`    | Where did a policy come from?                                      |
| `PolicyCompiler`  | How are policy sources combined into an effective policy?          |
| `EffectivePolicy` | The immutable policy actually used by the resolver                 |
| `Resolver`        | How are source values converted into resolved configuration state? |
| `Snapshot`        | Immutable resolved configuration state                             |
| `ConfigContext`   | The complete configuration world and its lifecycle                 |
| `Config`          | Application-facing configuration API                               |
| `Writer`          | Writes data to one source                                          |
| `Materializer`    | Applies configuration state to selected sources                    |
| `WatchManager`    | Detects source changes and triggers reload/resolution              |
| `Diagnostics`     | Explains and inspects configuration behavior                       |

---

## 4. Definitions

A definition describes the application's configuration contract.

For example:

```cpp
config.define<int>("server.port")
      .defaultValue(8080)
      .range(1, 65535);
```

A definition may contain:

* Name
* Type
* Default value
* Validation rules
* Description
* Metadata
* Aliases
* Optional source mappings

A definition does **not** automatically create or modify values in every source.

For example, defining:

```text
server.port
```

must not automatically create:

```text
SERVER_PORT
```

in the environment, write a configuration file, modify an OS setting, or create a build variable.

Definition and source mapping are separate concepts.

---

## 5. Sources

A `Source` provides configuration data to ConfigForge and may optionally accept configuration data from ConfigForge.

Typical sources include:

### Standard Sources

* Default
* Memory
* Environment
* Command Line
* File

### Optional Sources

* Build-time configuration
* OS-native settings
* User configuration
* System configuration

### Custom Sources

Users may implement their own providers.

A source has an identity, capabilities, and source-specific implementation.

---

## 6. Source Type vs Source Instance

A source provider describes how a source type works.

For example:

```text
FileSource
```

may have multiple instances:

```text
config.toml
config.local.toml
config.production.toml
```

These are separate source instances and can have independent policies.

This distinction is important because configuration systems commonly contain multiple files with different purposes and priorities.

For example:

```text
CLI
    >
Environment
    >
Local File
    >
Production File
    >
Project File
    >
Default
```

Source registration order must not implicitly determine this precedence.

---

## 7. Source Capabilities

A source exposes its technical capabilities.

Initial capability set:

```text
READ
WRITE
CREATE
DELETE
WATCH
ENUMERATE
ATOMIC_WRITE
```

Capabilities describe what the underlying source can technically perform.

For example:

```text
FileSource
    READ
    WRITE
    CREATE
    DELETE
    WATCH
```

A read-only source may expose:

```text
READ
```

A source's capabilities cannot be increased by policy.

If a policy requests:

```text
WRITE
```

for a source that only supports:

```text
READ
```

the policy is invalid.

Capability validation should happen before the policy becomes effective.

The validation mechanism should also be available explicitly so applications can validate configuration before execution.

---

## 8. Policy

Policy determines how ConfigForge is allowed to use sources.

Policy may control:

* Read precedence
* Source permissions
* Write targets
* Parameter-specific restrictions
* Source-specific behavior
* Propagation targets
* Overrides

For example:

```text
Read precedence:

CLI
>
Environment
>
Local File
>
Project File
>
Default
```

and:

```text
Environment:
    READ

Local File:
    READ WRITE

Project File:
    READ
```

Capability and policy are intentionally different:

```text
Capability
    = What can this source technically do?

Policy
    = What may ConfigForge do with it?
```

---

## 9. Parameter-Level Policy

Policies can apply globally or to individual parameters.

For example:

```text
Global:
    CLI > Environment > Local > Project > Default
```

while:

```text
database.password:
    Environment > Local
```

The parameter-level policy overrides or specializes the inherited global policy.

Users should not need to define policy for every parameter.

Policy inheritance should provide sensible defaults.

---

## 10. Policy Sources and Layers

Policies can originate from several places:

```text
Built-in Defaults
Programmatic Policy
Declarative DSL
Runtime Overrides
```

These are represented internally as policy sources/layers and compiled into one effective policy.

A default precedence model is:

```text
Runtime Override
    >
DSL Policy
    >
Programmatic Policy
    >
Built-in Default
```

Therefore, when a DSL policy and a programmatic policy conflict, the DSL policy wins.

This behavior must be deterministic and explainable.

---

## 11. Multiple Policy Files

ConfigForge supports multiple declarative policy files.

For example:

```text
base.policy.toml
project.policy.toml
deployment.policy.toml
```

Policy files are represented as policy layers.

A layer ordering can be explicitly defined:

```text
base
    <
project
    <
deployment
```

Policies that do not conflict may be merged.

If two policies at the same effective precedence conflict and no explicit precedence exists, ConfigForge should report a policy conflict rather than silently selecting one.

The following behaviors should not be used as implicit conflict resolution:

* Last loaded wins
* Alphabetical file order
* Registration order
* Filesystem order

---

## 12. Policy DSL

ConfigForge supports a declarative policy format in addition to its programmatic policy API.

The initial DSL target is **TOML**.

Programmatic APIs remain the primary API for applications.

The DSL exists for cases where policy should be:

* externally managed
* deployment-specific
* editable without recompiling
* shared across environments
* maintained separately from application code

The DSL should be an optional provider and must not become a mandatory dependency of the core.

---

## 13. Policy Compilation

Policy sources are compiled before resolution.

```text
Policy Sources
    ↓
Policy Layers
    ↓
Policy Compiler
    ↓
Capability Validation
    ↓
Conflict Validation
    ↓
Effective Policy
```

`EffectivePolicy` is immutable and can safely be used by the resolver.

This prevents the resolver from needing to understand whether a policy originated from:

* C++
* TOML
* defaults
* runtime overrides

---

## 14. Resolver

The resolver is responsible for deterministic configuration resolution.

Conceptually:

```text
Definitions
+
Source Values
+
Effective Policy
+
Runtime State
    ↓
Resolver
    ↓
Resolved State
    ↓
Snapshot
```

The resolver must not:

* modify sources
* write files
* parse policy files
* perform application logic
* implicitly watch sources

Its responsibility is limited to producing the correct resolved configuration state.

---

## 15. Snapshot

A `Snapshot` represents an immutable resolved configuration state.

```text
Sources
    ↓
Resolver
    ↓
Snapshot
```

Applications can obtain a snapshot:

```cpp
auto snapshot = config.snapshot();
```

and repeatedly access it without accessing the underlying sources.

This is particularly useful for hot loops and concurrent applications.

A source change should produce a new snapshot rather than mutating an existing snapshot.

```text
Snapshot #1
     │
     │ source changes
     ↓
Snapshot #2
```

Existing users of Snapshot #1 can continue using it safely.

---

## 16. ConfigContext

A `ConfigContext` represents one complete configuration world.

It owns or coordinates:

* Source instances
* Definitions
* Effective policy
* Resolver
* Current snapshot
* Runtime state
* Diagnostics
* Watch management

A context provides isolation.

For example:

```text
ConfigContext A
    Sources A
    Policy A
    Snapshot A

ConfigContext B
    Sources B
    Policy B
    Snapshot B
```

Changes in A must not implicitly modify B.

There is no implicit global configuration state and there is no "last Config instance wins" behavior.

---

## 17. Config

`Config` is the application-facing API over a `ConfigContext`.

Typical operations include:

```cpp
config.get<T>("key");
config.set("key", value);
config.write("key", value, source);
config.snapshot();
config.watch("key", callback);
config.reload();
config.explain("key");
```

The exact syntax may evolve, but the semantics should remain stable.

---

## 18. Multiple Config Instances

Multiple `Config` objects must be supported.

By default they should be isolated through their contexts.

If applications explicitly want shared configuration state, multiple facades may reference the same `ConfigContext`.

```text
             ConfigContext
             /            \
         Config A       Config B
```

Sharing must be explicit.

ConfigForge must never rely on global mutable configuration state to make multiple instances interact.

---

## 19. Reading Configuration

The common operation is:

```cpp
auto port = config.get<int>("server.port");
```

This uses:

```text
Current Snapshot
```

and therefore should be cheap.

The normal `get()` operation performs policy-based resolution through the current configuration state.

---

## 20. Scoped Reads

Applications may sometimes want to restrict resolution to selected sources.

For example:

```cpp
config.get<int>(
    "server.port",
    Sources{Environment, ProjectFile}
);
```

This does not mutate the global policy.

It performs a scoped resolution using the selected sources.

Raw source access remains the responsibility of the source API:

```text
config.get()
    → resolved configuration

source.read()
    → source-specific raw data
```

---

## 21. Runtime `set()`

`set()` modifies runtime configuration state.

```cpp
config.set("server.port", 9000);
```

The default behavior is:

```text
Runtime State
    ↓
New Snapshot
```

It does not automatically modify:

* Environment variables
* Configuration files
* OS-native settings
* Build configuration
* Other sources

This makes `set()` safe and predictable.

---

## 22. Explicit Persistent Writes

Persistent source mutation is explicit.

```cpp
config.write(
    "server.port",
    9000,
    SourceId{"config.local"}
);
```

The operation performs:

```text
Policy Check
    ↓
Capability Check
    ↓
Writer
    ↓
Selected Source
```

`WRITE` capability and policy permissions must both allow the operation.

---

## 23. Automatic Propagation

ConfigForge also supports optional automatic propagation.

A user may explicitly request:

```text
set()
    +
Propagation
```

For example:

```cpp
config.set(
    "server.port",
    9000,
    SetMode::Propagate
);
```

In this mode ConfigForge can:

```text
Runtime State
    ↓
Materializer
    ↓
Effective Policy
    ↓
Writable Target Sources
    ↓
Writer
```

This operation may be slower than a runtime-only `set()`.

That is intentional.

The user explicitly accepts the additional work in exchange for convenience.

Automatic propagation must never be the default behavior of `set()`.

---

## 24. Propagation Targets

Propagation can use policy-selected targets or explicit targets.

Conceptually:

```text
Propagation::PolicyTargets
Propagation::SelectedSources
Propagation::AllWritable
```

An explicit all-writable operation may update every source permitted by capability and policy.

Read-only sources must not be modified.

If propagation partially fails, ConfigForge should report the individual results rather than pretending that the operation was globally atomic.

A propagation result may contain:

```text
Successful Sources
Failed Sources
Skipped Sources
Errors
```

ConfigForge does not promise cross-source transactions in the initial architecture.

---

## 25. Materializer

The `Materializer` is responsible for applying configuration state to one or more sources.

It is deliberately separated from `Writer`.

```text
Materializer
    ↓
Target Selection
    ↓
Policy
    ↓
Writer(s)
    ↓
Source(s)
```

`Writer` answers:

> How do I write to this source?

`Materializer` answers:

> How should this configuration state be applied to the selected sources?

This separation becomes important when propagation targets multiple sources.

---

## 26. Watch and Reload

Sources may optionally support `WATCH`.

ConfigForge should not require every source to implement watching.

For sources that support it:

```text
Source Change
    ↓
WatchManager
    ↓
Reload
    ↓
Resolution
    ↓
New Snapshot
    ↓
Notification
```

Watching must not implicitly trigger writes.

A watched value changing should not automatically propagate that value to other sources.

---

## 27. Selective Reload

Reloading should be possible at different scopes.

Conceptually:

```text
reload()
reload(source)
reload(key)
```

The implementation may optimize these operations using source-level and parameter-level invalidation.

A source change should not require rebuilding unrelated configuration state when an incremental strategy can safely be used.

---

## 28. Performance Model

ConfigForge should distinguish between the cold path and the hot path.

### Cold Path

May include:

* Source initialization
* File I/O
* Parsing
* Policy compilation
* Validation
* Resolution
* Snapshot creation

### Hot Path

Should ideally be:

```text
snapshot.get()
```

with:

* no file I/O
* no parser invocation
* no policy compilation
* no source discovery
* minimal allocation
* minimal synchronization

Performance should be treated as a design requirement rather than a later optimization.

---

## 29. Diagnostics

One of ConfigForge's major usability features is explaining why a value has a particular result.

For example:

```cpp
config.explain("server.port");
```

could report:

```text
Parameter: server.port

Sources:
  CLI          → not present
  Environment  → 8080
  Local File   → 3000
  Project File → not present
  Default      → 80

Policy:
  CLI > Environment > Local File > Project File > Default

Selected:
  Environment

Reason:
  Environment has higher precedence than Local File.
```

The purpose is to answer:

> **Why is this value what it is?**

---

## 30. Configuration Inspection

ConfigForge should provide a matrix-oriented inspection API.

Conceptually:

```text
Parameter       CLI   ENV   Local   Project   Default   Selected
----------------------------------------------------------------
server.port     -     8080  3000    -         80        ENV
server.host     -     -     -       localhost localhost Project
tls.enabled     false -     true    -         true      CLI
```

Inspection should be available programmatically and, where useful, through human-readable diagnostic output.

---

## 31. Error Handling

The core library should avoid requiring exceptions.

The primary error model should be based on explicit result/error values.

C:

```text
config_result_t
config_error_t
```

C++:

```text
Result<T, ConfigError>
```

Rust:

```text
Result<T, ConfigError>
```

A convenience API such as `getOrThrow()` may be provided for environments where exceptions are preferred, but exceptions must not be required by the core.

Important error categories include:

* Invalid definition
* Type conversion failure
* Validation failure
* Source failure
* Unsupported capability
* Policy violation
* Policy conflict
* Resolution failure
* Write failure
* Propagation failure
* Watch failure

Errors should contain enough structured information to explain the failure.

---

## 32. Providers and Plugins

ConfigForge should support user-defined providers.

A provider may implement a source such as:

```text
Redis
Consul
Database
Remote API
Device NVRAM
Custom embedded storage
Company-specific configuration service
```

Providers should not necessarily be dynamically loaded.

The architecture should support:

```text
Built-in Provider
Static Provider
Dynamic Provider
```

without requiring dynamic loading.

This is important for embedded and statically linked applications.

---

## 33. Optional Features

ConfigForge should use feature-level build configuration.

For example:

```text
CONFIGFORGE_ENABLE_ENV
CONFIGFORGE_ENABLE_FILE
CONFIGFORGE_ENABLE_TOML
CONFIGFORGE_ENABLE_NATIVE
CONFIGFORGE_ENABLE_WATCH
```

The exact build interface may differ.

The important requirement is:

> Disabled functionality must not become an unnecessary runtime or build-time dependency.

For example, an application that does not use JSON should not need a JSON parser in its final binary.

---

## 34. C as the Foundation

The lowest-level public API should be designed around C.

The reason is portability.

C provides:

* broad compiler support
* C ABI interoperability
* easy bindings
* embedded-system suitability
* static linking
* compatibility with C++
* compatibility with Rust
* compatibility with other languages through FFI
* future WASM integration

C++ should provide the primary ergonomic API for C++ applications.

Rust should be supported through bindings built over the stable C interface.

The architecture should therefore follow:

```text
                 Language APIs

          C++            Rust            Other
           │              │                │
           └──────────────┼────────────────┘
                          ↓
                     Stable C ABI
                          ↓
                    ConfigForge Core
```

---

## 35. Header-Only Strategy

ConfigForge should not require the entire implementation to be header-only.

A fully header-only architecture would become increasingly expensive as optional providers, parsers, OS integrations, watchers, and ABI boundaries are added.

The preferred architecture is:

```text
ConfigForge Core
    → compiled library

C ABI
    → stable public interface

C++ API
    → lightweight ergonomic headers

Providers
    → independently compiled optional components
```

Static linking remains fully supported.

---

## 36. Native OS Configuration

OS-native configuration is an important optional source category.

The design is inspired by systems such as `QSettings`, where platform-specific configuration mechanisms can be exposed through a common API.

Examples may include:

* Windows Registry
* macOS preferences
* Linux desktop configuration systems
* Other platform-native stores

These should be optional providers rather than core dependencies.

---

## 37. Build-Time Configuration

Build-time values may be represented as read-only configuration sources.

Examples include:

* Compiler definitions
* Generated headers
* CMake-generated values
* Build metadata
* Platform information

Runtime ConfigForge operations must not pretend that a build-time value can be modified.

The source capability model naturally represents this:

```text
BuildSource
    READ
```

---

## 38. Configuration Mapping

Definitions may optionally map parameters to source-specific representations.

For example:

```text
server.port

Environment:
    MYAPP_SERVER_PORT

Command Line:
    --server-port

File:
    server.port
```

This mapping is explicit.

Defining a parameter does not automatically create variables or files in every configured source.

---

## 39. Configuration Lifecycle

A configuration context follows a conceptual lifecycle:

```text
Create
  ↓
Register Sources
  ↓
Register Definitions
  ↓
Load Policy Sources
  ↓
Compile Policy
  ↓
Validate Capabilities / Policy
  ↓
Load Source Data
  ↓
Resolve
  ↓
Create Snapshot
  ↓
Run
  ↓
Watch / Reload / Runtime Mutation
  ↓
New Snapshot
```

Policy changes may trigger recompilation and re-resolution.

Existing snapshots remain immutable.

---

## 40. Isolation and Shared State

ConfigForge should not use implicit global mutable state.

Each `ConfigContext` defines its own configuration universe.

Applications that require shared configuration may explicitly share a context.

This makes behavior predictable in:

* multi-threaded applications
* libraries embedded into larger applications
* tests
* plugins
* multiple application components
* multiple independent configuration environments

---

## 41. Testing Strategy

The architecture should make deterministic testing possible without requiring real operating-system resources.

Every major abstraction should be testable independently.

Examples:

```text
Definition Tests
Source Tests
Capability Tests
Policy Tests
Policy Compiler Tests
Resolver Tests
Snapshot Tests
Materializer Tests
Watch Tests
Diagnostics Tests
```

A fake in-memory source should be sufficient for most core tests.

The resolver should be testable as a deterministic component without filesystem or environment dependencies.

---

## 42. Security and Safety Principles

ConfigForge should follow several strict principles:

1. Reading configuration must not mutate sources.
2. `set()` must not persist data by default.
3. Persistent writes must be explicit or explicitly enabled through propagation.
4. A policy cannot grant capabilities a source does not possess.
5. Conflicting policies must not be silently resolved without a defined precedence.
6. Source failures must be observable.
7. Propagation must report partial failures.
8. Secrets should not automatically appear in diagnostics.
9. Sensitive values should support redaction in inspection and tracing.

Secrets management itself is outside the initial core scope.

---

## 43. Future Direction

The architecture should leave room for future capabilities without making them mandatory.

Potential future extensions include:

* Remote configuration providers
* Distributed configuration
* Advanced transactional writes
* Rollback mechanisms
* Configuration synchronization
* Secrets providers
* Schema registries
* Additional declarative policy formats
* WASM support
* Language bindings
* Dynamic provider loading

These should be implemented as extensions rather than requirements of the core.

---

## 44. Roadmap

## Phase 1 — Core

* C core API
* C ABI
* Definitions
* Values
* Source abstraction
* Source capabilities
* In-memory source
* Default source
* Environment source
* Command-line source
* File source
* Policy model
* Policy compiler
* Deterministic resolver
* Immutable snapshots
* Error/result model

## Phase 2 — Runtime

* `ConfigContext`
* `Config`
* Runtime `get`
* Runtime `set`
* Explicit `write`
* Materialization
* Propagation
* Reload
* Watch support

## Phase 3 — Diagnostics

* Explain
* Inspection
* Debug matrix
* Resolution tracing
* Policy diagnostics
* Source diagnostics

## Phase 4 — Policy DSL

* TOML policy format
* Multiple policy layers
* Policy precedence
* Conflict detection
* Runtime policy overrides

## Phase 5 — Ecosystem

* C++ ergonomic API
* Rust bindings
* Native OS providers
* Additional providers
* Static and optional feature configurations
* WASM research

---

## 45. Non-Goals

ConfigForge is not intended to become:

* A general-purpose database
* A secrets management platform
* A distributed configuration service
* A service discovery system
* A network synchronization framework
* A general-purpose serialization framework
* A dependency injection framework

These systems may integrate with ConfigForge through providers, but they should not become part of the core.

---

## 46. Architectural Principles

ConfigForge should preserve the following principles throughout development:

```text
Simple by default.
Explicit when behavior has side effects.
Deterministic by design.
Fast on the hot path.
Immutable snapshots.
No implicit global state.
Capabilities describe reality.
Policies describe permission and precedence.
Definitions describe application contracts.
Sources provide data.
Resolvers resolve.
Materializers propagate.
Diagnostics explain.
Providers extend.
```

The most important rule is:

> **Convenience must never hide a significant side effect.**

---

## 47. Project Vision

ConfigForge is intended to become a general-purpose configuration infrastructure rather than another configuration-file parser.

Its long-term value comes from treating configuration as a system:

```text
Sources
    +
Definitions
    +
Policies
    +
Resolution
    +
Immutable Runtime State
    +
Controlled Mutation
    +
Diagnostics
```

The result should be a library that can be used for a simple desktop application with a few environment variables, while still being capable of supporting complex applications with multiple configuration files, runtime overrides, native OS settings, custom providers, policy layers, hot reload, and performance-sensitive embedded environments.

The core should remain small, predictable, portable, statically linkable, and free of unnecessary mandatory dependencies.

**ConfigForge should make configuration management easier without taking control away from the programmer.**
