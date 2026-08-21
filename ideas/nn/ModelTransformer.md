# Model Transformer

**Model Transformer** is an extensible model tooling framework for converting machine learning models between different model representations and, in later stages, applying graph-level transformations.

The project is designed as part of a broader model tooling ecosystem alongside **Model Inspector**, **Quantization Tool**, and **Model Tooling Core**.

The primary goal is not to build another inference framework or compiler. The goal is to provide a practical, modular, and reusable foundation for **model conversion and transformation**.

---

## Goals

* Convert models between supported formats.
* Provide a common model representation abstraction.
* Avoid `N × N` format-to-format converter dependencies.
* Preserve model semantics during conversion.
* Provide structural and numerical validation.
* Support extensible importers and exporters.
* Provide a reusable graph transformation infrastructure.
* Integrate naturally with Model Inspector and Quantization Tool.
* Keep the architecture extensible without prematurely becoming a compiler project.

---

## Non-Goals

The initial project will **not** attempt to become:

* An inference runtime.
* A deep learning framework.
* A full model compiler.
* A hardware-specific compiler.
* A universal model IR implementation.
* A replacement for existing model ecosystems.

Compiler-oriented capabilities may be considered much later only if real transformation and lowering requirements justify them.

---

## Architecture

The high-level architecture is:

```text
                         Model Transformer
                                |
                    +-----------+-----------+
                    |                       |
              Format Conversion      Graph Transformation
                    |                       |
                    +-----------+-----------+
                                |
                       Model Representation
                                |
                    +-----------+-----------+
                    |                       |
                  ONNX                    Future
                Backend                 Backends
```

The project will separate **model representation**, **conversion**, and **transformation** concerns.

---

## Model Representation

The project will use a small abstraction layer representing concepts that are common across model formats.

Typical concepts include:

```text
Model
 └── Graph
      ├── Node / Operation
      ├── Value / Tensor
      ├── Attribute
      ├── Shape
      └── DataType
```

This abstraction is **not intended to become a complete compiler IR**.

The first implementation will use ONNX as the initial backend and will evolve the abstraction according to real conversion requirements.

The abstraction should only contain concepts that are genuinely useful across multiple model formats.

Format-specific concepts must remain inside their respective backends.

---

## Conversion Pipeline

The initial conversion pipeline is:

```text
Input Model
    |
    v
Importer
    |
    v
Model Representation
    |
    v
Exporter
    |
    v
Output Model
```

This avoids direct format-to-format dependencies such as:

```text
ONNX -> TFLite
ONNX -> MLIR
TFLite -> ONNX
TFLite -> MLIR
...
```

Instead, formats communicate through the common model representation:

```text
ONNX ----\
          \
TFLite ----> Model Representation <---- Future Formats
          /
MLIR ----/
```

---

## Development Roadmap

### Phase 1 — Format Converter

The first version focuses exclusively on model format conversion.

#### Initial format

* ONNX

#### First milestone

Implement a complete ONNX round-trip:

```text
ONNX
  |
  v
ONNX Importer
  |
  v
Model Representation
  |
  v
ONNX Exporter
  |
  v
ONNX
```

The primary objective is to prove that the internal representation and conversion architecture are viable.

#### Initial capabilities

* Model loading
* Graph loading
* Node/operator representation
* Tensor/value representation
* Shape representation
* Data type representation
* Attribute representation
* Initializer/constant handling
* Model input/output handling
* Model validation
* ONNX export
* Round-trip validation

---

### Phase 2 — Multi-Format Conversion

After the ONNX round-trip is stable, additional formats can be introduced.

Potential candidates:

* TFLite
* MLIR-based representations
* Other formats based on actual project requirements

The exact order should be determined by practical use cases rather than attempting to support every format.

Example:

```text
ONNX
  |
  v
Model Representation
  |
  +----> ONNX
  |
  +----> TFLite
  |
  +----> Future Format
```

---

### Phase 3 — Graph Transformation

Once the common representation and conversion infrastructure are proven, graph transformation capabilities can be introduced.

The transformation system should be modular and pass-based.

Conceptually:

```text
Model
  |
  v
Transformation Pipeline
  |
  +--> Constant Folding
  |
  +--> Dead Node Elimination
  |
  +--> Identity Elimination
  |
  +--> Operator Replacement
  |
  +--> Pattern-based Rewriting
  |
  +--> Operator Fusion
  |
  v
Transformed Model
```

Potential future transformations include:

* Constant folding
* Dead node elimination
* Identity elimination
* Operator replacement
* Operator decomposition
* Pattern-based graph rewriting
* Operator fusion
* Layout transformation
* Backend-specific transformations

Transformations should be independently composable and usable through a transformation pipeline.

---

## Validation

Model conversion and transformation must be validated.

Validation should operate at multiple levels:

### Structural validation

Check:

* Graph connectivity
* Inputs and outputs
* Operator validity
* Tensor shapes
* Data types
* Attributes
* Required model metadata

### Conversion validation

Verify that the exported model can be loaded successfully by the target format/runtime.

### Numerical validation

Where applicable:

```text
Input
  |
  +----> Original Model ------> Output A
  |
  +----> Converted Model -----> Output B
```

Compare:

```text
Output A vs Output B
```

using configurable numerical tolerances.

This becomes particularly important when the project later integrates with Quantization Tool.

---

## Extensibility

The architecture should make new formats and transformations independently extensible.

Conceptually:

```text
Importer
Exporter
Transformation
Pass
Pipeline
Validator
```

should be replaceable or extendable without modifying unrelated parts of the system.

A new model format should primarily require its own importer/exporter implementation rather than changes throughout the framework.

---

## Integration With Other Projects

### Model Inspector

Model Inspector analyzes models.

```text
Model
  |
  v
Model Inspector
  |
  v
Analysis
```

Model Transformer modifies or converts them.

```text
Model
  |
  v
Model Transformer
  |
  v
Converted / Transformed Model
```

Inspector results may later be used to identify transformation opportunities.

---

### Quantization Tool

Quantization Tool focuses on quantization decisions and analysis.

Model Transformer can provide the transformation infrastructure required to apply those decisions.

```text
Quantization Tool
        |
        | Quantization Plan
        v
Model Transformer
        |
        v
Quantized / Transformed Model
```

The responsibilities of the two projects should remain separate.

---

### Model Tooling Core

Common infrastructure should be extracted into **Model Tooling Core** when it is genuinely shared between the tooling projects.

Potential shared components include:

* Common data structures
* Error handling
* Logging
* Configuration integration
* Plugin infrastructure
* Validation utilities
* CLI infrastructure
* Common model metadata
* Shared testing utilities

Model Tooling Core should remain an infrastructure layer rather than absorbing model-specific functionality.

---

## Design Principles

### Start Small

Begin with ONNX round-trip conversion instead of attempting universal format support.

### Real Requirements Over Theoretical Abstractions

The common model representation should evolve from actual conversion requirements.

### No Premature Compiler

Compiler functionality is explicitly deferred until transformation and lowering requirements make it necessary.

### Modular Architecture

Importers, exporters, transformations, validators, and pipelines should remain independently replaceable.

### Semantic Preservation

Conversion should preserve the model's intended behavior whenever the source and target formats allow it.

### Validation First

Every major conversion or transformation capability should have structural and, where possible, numerical validation.

### Reuse Existing Ecosystems

Existing standards and mature infrastructures should be reused instead of unnecessarily recreated.

### Minimum Effort, Maximum Utility

The project should solve practical model tooling problems without becoming an unnecessarily large compiler/framework project.

---

## Suggested Repository Structure

```text
model-transformer/
├── core/
├── representation/
├── formats/
│   └── onnx/
│       ├── importer/
│       └── exporter/
├── conversion/
├── transformation/
├── validation/
├── cli/
├── tests/
├── examples/
└── docs/
```

The exact structure can evolve as the implementation grows.

---

## Long-Term Direction

The intended evolution is:

```text
Phase 1
Format Conversion
       |
       v
Phase 2
Multi-Format Conversion
       |
       v
Phase 3
Graph Transformation
       |
       v
Phase 4
Optimization / Backend-Aware Transformation
       |
       v
Future
Compiler-like Capabilities
```

The final phase is optional.

The project should only move toward compiler functionality if real-world transformation, lowering, or backend requirements demonstrate that such capabilities are necessary.

---

## Summary

**Model Transformer** starts as a practical and extensible **model format conversion framework**.

Its initial foundation is:

```text
Importer
    ↓
Model Representation
    ↓
Exporter
```

The first concrete implementation targets **ONNX round-trip conversion**.

Once this foundation is proven, the project can naturally evolve toward:

```text
Format Conversion
       +
Graph Transformation
       +
Validation
       +
Optimization
```

without prematurely becoming a model compiler.

The project is intended to **complement**, rather than compete with, existing model ecosystems and tooling.
