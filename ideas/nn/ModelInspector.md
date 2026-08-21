# Model Inspector

## Overview

**Model Inspector** is a static analysis and model intelligence tool for machine learning models.

It analyzes a model without executing inference and produces structured information about its graph, tensors, operators, shapes, data types, computational complexity, memory requirements, runtime compatibility, and deployment risks.

The project is designed as both a reusable library and a CLI application.

Model Inspector is intentionally **read-only**. It does not modify the model.

Its primary purpose is to answer:

> **What is inside this model, what does it require, and can it be deployed safely on the target runtime and hardware?**

---

## Project Ecosystem

Model Inspector is part of a larger model tooling ecosystem.

The ecosystem should be built around a shared **Model Tooling Core**.

```text
                         Model Tooling Core
                                │
              ┌─────────────────┼──────────────────┐
              │                 │                  │
              ▼                 ▼                  ▼
       Model Inspector   Model Transformer   Future Tools
              │                 │                  │
              └─────────────────┼──────────────────┘
                                │
                         Model Tooling
                           Ecosystem
```

The architectural principle is:

> **Model Tooling Core provides the common language and infrastructure; Model Inspector understands the model; Model Transformer changes the model.**

---

# Model Tooling Core

## Purpose

**Model Tooling Core** is the shared foundation for model-oriented tools.

It should contain concepts that are common to multiple projects rather than functionality specific to inspection or transformation.

The Core should prevent the Inspector and Transformer from developing incompatible representations of the same model concepts.

---

## Core Responsibilities

The Core should provide abstractions for:

* Model representation.
* Graph representation.
* Node/operator representation.
* Tensor representation.
* Shape representation.
* Data type representation.
* Model metadata.
* Model inputs and outputs.
* Attributes.
* Initializers and constants.
* Model format abstraction.
* Model analysis results.
* Findings.
* Severity levels.
* Rules.
* Target profiles.
* Compatibility information.
* Report schemas.
* Model snapshots.
* Model comparison primitives.
* Configuration integration.

---

## Internal Model Representation

The Core should define a normalized internal representation.

```text
Model
├── Metadata
├── Inputs
├── Outputs
├── Graph
│   ├── Nodes
│   ├── Edges
│   └── Subgraphs
├── Tensors
├── Initializers
└── Constants
```

Format-specific adapters should convert external model formats into this representation.

For example:

```text
ONNX
  │
  ▼
ONNX Adapter
  │
  ▼
Model Tooling Core IR
```

The same representation can then be consumed by both Inspector and Transformer.

---

## Core IR

The internal representation should be designed as an abstraction rather than a direct copy of any specific model format.

It should support:

* Static shapes.
* Dynamic shapes.
* Multiple inputs/outputs.
* Nested/subgraphs.
* Constants.
* Attributes.
* Operator versions.
* Dtype information.
* Layout information where available.
* Format-specific metadata where necessary.

The IR should remain extensible so future model formats can be represented without redesigning the entire ecosystem.

---

## Model Format Abstraction

Model loading should be separated from the Core IR.

```text
ModelFormat
├── ONNX
├── TFLite
├── TorchScript
├── TensorFlow Graph / SavedModel
└── MLIR
```

Each format adapter is responsible for:

```text
External Format
      │
      ▼
Parser
      │
      ▼
Normalizer
      │
      ▼
Core IR
```

The analysis and transformation layers should operate primarily on the normalized representation.

---

# Model Inspector

## Goals

* Inspect ML model structure and metadata.
* Validate graph integrity.
* Analyze tensors, shapes, dtypes, and layouts.
* Enumerate and analyze operators.
* Detect unsupported or problematic operations.
* Estimate parameters, FLOPs/MACs, and memory requirements.
* Analyze static and dynamic shapes.
* Detect deployment risks.
* Evaluate compatibility with runtimes, compilers, and target platforms.
* Produce human-readable and machine-readable reports.
* Provide deterministic analysis suitable for CI/CD.
* Compare models before and after transformation.
* Provide analysis information to Model Transformer.

---

## Non-Goals

Model Inspector does not:

* Modify model graphs.
* Optimize models.
* Quantize models.
* Fuse operators.
* Convert models.
* Execute inference.
* Replace model runtimes.
* Replace runtime profilers.
* Perform benchmark measurements.

These responsibilities belong to other components of the ecosystem.

---

# Architecture

```text
                    ┌─────────────────────┐
                    │      CLI / API      │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │   Inspector Engine  │
                    └──────────┬──────────┘
                               │
                     ┌─────────▼─────────┐
                     │ Model Tooling Core │
                     └─────────┬─────────┘
                               │
       ┌───────────────────────┼────────────────────────┐
       │                       │                        │
       ▼                       ▼                        ▼
 ┌───────────┐          ┌────────────┐          ┌──────────────┐
 │  Format   │          │ Validation │          │ Target Rules │
 │  Adapters │          │            │          │              │
 └─────┬─────┘          └─────┬──────┘          └──────┬───────┘
       │                       │                        │
       └───────────────────────┼────────────────────────┘
                               ▼
                      ┌──────────────────┐
                      │ Analysis Passes  │
                      └────────┬─────────┘
                               │
          ┌────────────────────┼─────────────────────┐
          ▼                    ▼                     ▼
     Graph Analysis      Tensor Analysis       Complexity
          │                    │                     │
          └────────────────────┼─────────────────────┘
                               ▼
                        ┌──────────────┐
                        │  Rule Engine │
                        └──────┬───────┘
                               ▼
                        ┌──────────────┐
                        │ Model Report │
                        └──────┬───────┘
                               │
                    ┌──────────┴──────────┐
                    ▼                     ▼
               Human Output         JSON / API
```

---

# Analysis Pipeline

The standard inspection pipeline is:

```text
Load
  ↓
Normalize
  ↓
Validate
  ↓
Analyze
  ↓
Apply Rules
  ↓
Generate Report
```

Each stage should be independently testable.

---

# Graph Analysis

The graph analyzer should provide:

* Node count.
* Edge count.
* Input/output nodes.
* Producer/consumer relationships.
* Graph depth.
* Graph topology.
* Reachability analysis.
* Dead/unreachable nodes.
* Unused tensors.
* Constant-only branches.
* Subgraph detection.
* Operator distribution.
* Graph-level structural anomalies.

---

# Tensor Analysis

For every tensor:

* Name.
* Shape.
* Rank.
* Data type.
* Layout where identifiable.
* Static/dynamic dimensions.
* Memory size.
* Producer.
* Consumers.
* Constant/initializer status.

Detect:

* Dynamic dimensions.
* Unknown dimensions.
* Shape inconsistencies.
* Unexpected dtype conversions.
* Large activation tensors.
* Large constant tensors.
* Potential memory bottlenecks.

---

# Shape Analysis

A dedicated shape analysis pass should perform shape propagation whenever possible.

Detect:

* Invalid shapes.
* Incompatible dimensions.
* Unknown dimensions.
* Dynamic shapes.
* Shape-dependent operators.
* Potential runtime shape failures.

Shape analysis should remain independent of model execution whenever static information is sufficient.

---

# Operator Analysis

The operator analyzer should report:

* Operator type.
* Operator version/opset.
* Attributes.
* Input/output tensors.
* Supported data types.
* Operator frequency.
* Operator-specific memory requirements.
* Operator-specific computational cost.

Operators should be classified as:

```text
Standard
Supported
Conditionally Supported
Unsupported
Unknown
```

---

# Complexity Analysis

The complexity analyzer should estimate:

* Parameter count.
* Weight size.
* FLOPs.
* MACs.
* Activation memory.
* Estimated peak memory.
* Per-operator computational cost.

Memory should be separated into:

```text
Weights
Activations
Temporary Buffers
Runtime Overhead
```

Estimates must clearly identify their assumptions.

Static estimates must never be presented as measured runtime performance.

---

# Data Type Analysis

The analyzer should identify:

* FP32.
* FP16.
* BF16.
* INT8.
* INT16.
* INT32.
* UINT8.
* Other supported types.

It should detect:

* Mixed precision.
* Implicit/explicit casts.
* Unsupported dtypes.
* Quantized tensors.
* Quantize operations.
* Dequantize operations.
* Potential precision-related deployment issues.

---

# Compatibility Analysis

Compatibility analysis evaluates a model against a selected target.

Examples:

```text
ONNX Runtime
TVM
IREE
TFLite
CPU
GPU
NPU
Embedded ARM
```

The analysis should determine:

* Supported operators.
* Unsupported operators.
* Supported dtypes.
* Unsupported attributes.
* Shape limitations.
* Runtime-specific restrictions.
* Target-specific constraints.

---

# Target Profiles

Targets should be represented through reusable profiles rather than hard-coded checks.

```text
TargetProfile
├── Runtime
├── Backend
├── Architecture
├── Supported Operators
├── Supported DTypes
├── Shape Constraints
├── Memory Constraints
└── Custom Rules
```

Examples:

```text
model-inspector inspect model.onnx --target tvm
model-inspector inspect model.onnx --target iree
model-inspector inspect model.onnx --target rv1126
```

Target profiles should be extensible.

The underlying profile abstraction should live in **Model Tooling Core**.

---

# Rule Engine

Deployment and structural checks should use a rule engine.

```text
Rule
├── Condition
├── Finding
├── Severity
├── Evidence
└── Recommendation
```

Example:

```text
Rule:
    DynamicInputShape

Condition:
    Input contains unresolved dimension

Finding:
    Model requires dynamic shape handling

Severity:
    WARNING

Recommendation:
    Consider static shape specialization for embedded deployment
```

The rule abstraction belongs in Model Tooling Core.

Inspector provides the analysis context and built-in inspection rules.

---

# Findings

Every detected issue should have a structured finding.

Severity levels:

```text
INFO
WARNING
ERROR
CRITICAL
```

A finding should contain:

```text
ID
Category
Severity
Message
Evidence
Location
Recommendation
```

Example:

```text
MODEL-OP-001

Category:
    Compatibility

Severity:
    ERROR

Finding:
    Unsupported operator: GridSample

Location:
    node=GridSample_42

Recommendation:
    Replace or transform the operator before deployment.
```

The finding schema should be shared by Inspector and Transformer.

---

# Model Report

The report should expose a stable machine-readable schema.

```text
ModelReport
├── Metadata
├── Summary
├── Inputs
├── Outputs
├── Graph
├── Tensors
├── Operators
├── Shapes
├── DataTypes
├── Complexity
├── Memory
├── Compatibility
├── Findings
└── Recommendations
```

Supported output formats should include:

```text
Terminal
Markdown
JSON
DOT / Graphviz
```

The report schema should be defined centrally where practical, with Model Tooling Core owning the stable data structures.

---

# Model Snapshot

Inspector should be able to produce a serializable analysis snapshot.

```text
Model
   │
   ▼
Inspector
   │
   ▼
ModelSnapshot
```

A snapshot should contain enough information to:

* Compare models.
* Run regression checks.
* Store analysis artifacts.
* Validate transformation results.
* Use reports in CI/CD.

The snapshot format should be versioned.

---

# Model Comparison

Two snapshots should be comparable.

```text
Inspector(model_v1)
        │
        ▼
 ModelSnapshot A

Inspector(model_v2)
        │
        ▼
 ModelSnapshot B

        │
        ▼

     ModelDiff
```

Comparison should identify:

* Added/removed operators.
* Changed operators.
* Graph topology changes.
* Input/output changes.
* Shape changes.
* Dtype changes.
* Parameter count changes.
* FLOPs changes.
* Memory changes.
* Compatibility changes.
* New findings.
* Resolved findings.

The generic comparison primitives should live in Model Tooling Core.

Inspector provides the model-specific analysis data.

---

# Model Transformer Integration

The intended workflow is:

```text
Original Model
      │
      ▼
Model Inspector
      │
      │ ModelReport
      ▼
Model Transformer
      │
      ▼
Transformed Model
      │
      ▼
Model Inspector
      │
      ▼
ModelDiff
```

Example:

```text
Before:
    FP32
    120M parameters
    4.2 GFLOPs
    3 unsupported operators

After:
    INT8
    120M parameters
    1.1 GFLOPs
    0 unsupported operators
```

This provides an automated mechanism for verifying transformation effects.

---

# Model Tooling Core and Transformer

The Core should not become a hidden dependency containing Transformer-specific behavior.

The dependency direction should remain:

```text
Model Tooling Core
       ▲
       │
 ┌─────┴─────┐
 │           │
Inspector  Transformer
```

Not:

```text
Inspector ──► Transformer
```

and not:

```text
Transformer ──► Inspector
```

Both applications should depend on common abstractions.

---

# Library and CLI

The Inspector implementation should expose a reusable library:

```text
libmodelinspector
```

The CLI should be a thin layer over the library:

```text
model-inspector
```

The broader ecosystem may eventually expose:

```text
libmodeltooling
libmodelinspector
libmodeltransformer
```

with dependencies approximately structured as:

```text
libmodeltooling
      ▲
      │
 ┌────┴─────────────┐
 │                  │
libmodelinspector  libmodeltransformer
```

---

# Configuration

Configuration should support:

* Enabled analysis passes.
* Target profiles.
* Rule configuration.
* Severity thresholds.
* Output format.
* Report verbosity.
* Custom rules.
* Custom operator databases.
* Analysis options.

The configuration layer should integrate with the project's **ConfigForge** infrastructure where appropriate.

Model Tooling Core should consume configuration abstractions without coupling its domain model to ConfigForge implementation details.

---

# Determinism

Analysis should be deterministic.

Given:

```text
same model
same Inspector version
same target profile
same configuration
```

the analysis should produce equivalent results.

This is required for:

* CI/CD.
* Regression testing.
* Model transformation pipelines.
* Reproducible research.
* Deployment validation.

---

# CI/CD Integration

Model Inspector should be usable as a model validation gate.

```text
Build Model
     ↓
Model Inspector
     ↓
Compatibility Check
     ↓
Quality Gate
     ↓
Deploy
```

A pipeline should be able to fail when:

```text
CRITICAL findings > 0
ERROR findings > allowed threshold
Unsupported operators > 0
Memory estimate > target limit
```

The quality-gate primitives should be reusable by future tooling.

---

# Extensibility

The ecosystem should support extensions for:

```text
Format Adapter
Analysis Pass
Target Profile
Compatibility Database
Rule
Report Formatter
```

The extension architecture should be designed so that new formats, runtimes and hardware targets can be introduced without modifying the Inspector core.

---

# Testing Strategy

## Unit Tests

Test independently:

* Core IR.
* Graph operations.
* Tensor representation.
* Shape inference.
* Complexity calculations.
* Memory calculations.
* Rule evaluation.
* Compatibility checks.
* Snapshot serialization.
* Model diff.

## Model Fixtures

Maintain small purpose-built models covering:

* Basic operators.
* Dynamic shapes.
* Invalid graphs.
* Quantized models.
* Mixed precision.
* Unsupported operators.
* Large tensors.
* Multi-input/output models.
* Subgraphs.

## Golden Reports

Known models should produce deterministic expected reports.

## Integration Tests

Validate:

```text
Model
→ Load
→ Normalize
→ Core IR
→ Analyze
→ Rules
→ Snapshot
→ Report
```

## Regression Tests

Every discovered bug should result in a permanent regression fixture.

---

# Development Phases

## Phase 0 — Model Tooling Core

Establish the shared foundation before implementing advanced Inspector functionality.

* Repository/project structure.
* Core package.
* Internal model representation.
* Graph abstraction.
* Tensor abstraction.
* Shape abstraction.
* Dtype abstraction.
* Node/operator abstraction.
* Metadata abstraction.
* Finding abstraction.
* Severity model.
* Rule abstraction.
* Target profile abstraction.
* Snapshot abstraction.
* ModelDiff primitives.
* Serialization contracts.
* ConfigForge integration boundary.

**Deliverable:**

A stable, format-independent model tooling foundation.

---

## Phase 1 — Inspector Core

* Inspector project.
* CLI.
* Core integration.
* First model format adapter.
* Model loading.
* Normalization.
* Basic graph inspection.
* Basic tensor inspection.
* Basic operator inspection.
* JSON report.

**Deliverable:**

A usable static model inspection tool.

---

## Phase 2 — Static Analysis

* Graph validation.
* Shape analysis.
* Dtype analysis.
* Parameter counting.
* FLOPs/MACs estimation.
* Memory estimation.
* Operator-level analysis.
* Structural findings.

**Deliverable:**

A complete static analysis engine.

---

## Phase 3 — Rule Engine

* Rule execution.
* Built-in rules.
* Severity handling.
* Evidence collection.
* Recommendations.
* Rule configuration.
* Custom rule support.

**Deliverable:**

A configurable model validation system.

---

## Phase 4 — Deployment Intelligence

* Runtime capability database.
* Target profiles.
* Operator compatibility.
* Dtype compatibility.
* Shape constraints.
* Memory constraints.
* Embedded deployment analysis.
* TVM profile.
* IREE profile.

**Deliverable:**

Target-aware deployment analysis.

---

## Phase 5 — Reporting

* Terminal report.
* Markdown report.
* Stable JSON schema.
* Graphviz export.
* Machine-readable findings.
* Snapshot serialization.

**Deliverable:**

Reports suitable for developers, automation and CI/CD.

---

## Phase 6 — Model Comparison

* Model snapshots.
* ModelDiff.
* Before/after comparison.
* Structural diff.
* Complexity diff.
* Memory diff.
* Compatibility diff.
* Finding diff.

**Deliverable:**

Reliable model regression and transformation analysis.

---

## Phase 7 — Model Transformer Integration

```text
Inspector
    ↓
ModelReport
    ↓
Transformer
    ↓
Transformed Model
    ↓
Inspector
    ↓
ModelDiff
```

Integration should enable:

* Transformation validation.
* Automatic quality gates.
* Before/after analysis.
* Regression detection.
* Optimization verification.

**Deliverable:**

A complete inspect → transform → inspect workflow.

---

## Phase 8 — Ecosystem Expansion

* Additional model formats.
* Additional runtime profiles.
* Additional hardware profiles.
* Plugin architecture.
* External compatibility databases.
* CI/CD integrations.
* Additional tooling built on Model Tooling Core.

**Deliverable:**

A reusable ML model tooling ecosystem rather than isolated utilities.

---

# Recommended Repository Structure

The exact structure may evolve, but the conceptual organization should resemble:

```text
model-tooling/
├── core/
│   ├── ir/
│   ├── graph/
│   ├── tensor/
│   ├── shape/
│   ├── dtype/
│   ├── operator/
│   ├── findings/
│   ├── rules/
│   ├── targets/
│   ├── snapshots/
│   └── diff/
│
├── inspector/
│   ├── loaders/
│   ├── analyzers/
│   ├── rules/
│   ├── reports/
│   └── cli/
│
├── transformer/
│   └── ...
│
├── formats/
│   ├── onnx/
│   ├── tflite/
│   ├── torchscript/
│   └── ...
│
├── targets/
│   ├── tvm/
│   ├── iree/
│   ├── cpu/
│   ├── gpu/
│   └── embedded/
│
└── tests/
```

Whether these are ultimately separate repositories or a monorepo is an implementation decision. The **architectural separation of Core, Inspector and Transformer should remain regardless of repository layout**.

---

# End-to-End Vision

```text
                         ML Model
                            │
                            ▼
                  ┌───────────────────┐
                  │ Model Tooling Core│
                  └─────────┬─────────┘
                            │
                 ┌──────────┴──────────┐
                 ▼                     ▼
        ┌────────────────┐    ┌─────────────────┐
        │ Model Inspector│    │Model Transformer│
        └───────┬────────┘    └────────┬────────┘
                │                      │
                ▼                      ▼
          Model Report            Transformed Model
                │                      │
                └──────────┬───────────┘
                           ▼
                    Model Inspector
                           │
                           ▼
                       ModelDiff
                           │
                           ▼
                    Quality Gate
                           │
                           ▼
                       Deployment
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
            CPU           GPU           NPU
             │             │             │
             └─────────────┼─────────────┘
                           ▼
                        Runtime
```

---

# Long-Term Vision

Model Tooling Core should become the **shared domain foundation for ML model engineering tools**.

The long-term ecosystem can grow beyond Inspector and Transformer:

```text
                    Model Tooling Core
                           │
       ┌───────────┬───────┼────────┬───────────┐
       ▼           ▼       ▼        ▼           ▼
   Inspector  Transformer Quantizer Optimizer  Analyzer
       │           │       │        │           │
       └───────────┴───────┼────────┴───────────┘
                           ▼
                    Deployment Pipeline
```

The separation of responsibilities should remain explicit:

* **Model Tooling Core** — common model abstractions and infrastructure.
* **Model Inspector** — understand and evaluate models.
* **Quantization Tool** — reduce numerical precision and prepare quantized models.
* **Model Transformer** — structurally modify and optimize models.
* **Deployment tooling** — compile/package models for target runtimes and hardware.

The resulting ecosystem forms a coherent model engineering pipeline:

```text
Inspect
   ↓
Analyze
   ↓
Quantize / Transform
   ↓
Inspect Again
   ↓
Compare
   ↓
Validate
   ↓
Deploy
```

The central architectural principle is:

> **Keep the model representation and common tooling in Model Tooling Core, keep analysis in Model Inspector, and keep model modification in Model Transformer.**

This provides a clean foundation for expanding the project family without allowing individual tools to become tightly coupled or duplicate the same model abstractions.
