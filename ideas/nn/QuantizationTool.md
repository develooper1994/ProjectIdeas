# Quantization Tool

**Quantization Tool** is a model optimization tool designed to convert floating-point machine learning models into lower-precision representations while controlling accuracy degradation, memory footprint, computational cost, and deployment compatibility.

It is the second component of the model tooling ecosystem:

```text
Model Inspector
       │
       ▼
Quantization Tool
       │
       ▼
Model Transformer
       │
       ▼
Deployment Toolchain
```

The tool is designed to be **backend-independent**. Its primary responsibility is quantization analysis, calibration, quantization policy evaluation, quantization parameter generation, execution, validation, and reporting.

---

## 1. Project Goals

### Primary Goals

* Provide a backend-independent quantization framework.
* Support **Post-Training Quantization (PTQ)** as the initial methodology.
* Provide a reusable calibration pipeline.
* Support INT8 quantization as the primary target.
* Support per-tensor and per-channel quantization.
* Support symmetric and asymmetric quantization.
* Calculate and persist quantization parameters.
* Provide fine-grained control over quantized nodes and tensors.
* Measure numerical and model-level degradation.
* Generate detailed quantization reports.
* Integrate with **Model Inspector**.
* Prepare a stable abstraction layer for future **Model Transformer** integration.
* Keep deployment/runtime-specific logic outside the core.

### Secondary Goals

* Mixed-precision quantization.
* Weight-only quantization.
* Additional integer precisions.
* Advanced calibration algorithms.
* QAT integration.
* Hardware/backend-specific quantization policies.

---

## 2. Non-Goals

The initial project should **not** attempt to:

* Become a general-purpose graph transformation framework.
* Implement a complete inference runtime.
* Implement hardware-specific kernels.
* Replace TVM, IREE, ONNX Runtime, TensorRT, TFLite, or similar runtimes.
* Perform arbitrary model graph rewriting.
* Implement QAT as the first quantization methodology.
* Optimize every possible quantization format from the beginning.
* Couple quantization logic directly to a specific embedded processor.

Graph rewriting required to materialize quantization should remain minimal and eventually be delegated to **Model Transformer**.

---

## 3. Project Ecosystem

The project belongs to a larger model tooling ecosystem.

```text
                    Model Tooling Core
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
       Model Inspector  Quantization  Model Transformer
                           Tool
```

The development order is intentionally:

1. **Model Inspector**
2. **Quantization Tool**
3. **Model Transformer**

The three projects should share a small common foundation rather than independently implementing model abstractions.

`Model Tooling Core` should initially contain only abstractions proven to be shared between projects. It should not become a large framework before real use cases require those abstractions.

---

## 4. Architectural Position

**Model Inspector** provides model understanding.

**Quantization Tool** determines how model precision can be reduced and generates the corresponding quantization information.

**Model Transformer** eventually materializes quantization decisions and performs graph-level transformations.

```text
                         Input Model
                              │
                              ▼
                    ┌───────────────────┐
                    │ Model Inspector   │
                    └─────────┬─────────┘
                              │
                              ▼
                    Model Understanding
                              │
                              ▼
                    ┌───────────────────┐
                    │ Quantization Tool │
                    └─────────┬─────────┘
                              │
                              ▼
                    Quantization Plan
                              │
                              ▼
                    ┌───────────────────┐
                    │ Model Transformer │
                    └─────────┬─────────┘
                              │
                              ▼
                     Optimized Model
                              │
                              ▼
                    Deployment Toolchain
```

A critical architectural rule is:

> **Quantization Tool and Model Transformer must not become tightly coupled.**

Quantization Tool answers:

> How should this tensor or node be quantized?

Model Transformer answers:

> How should this decision be represented in the model graph?

---

## 5. Core Architecture

```text
                         Input Model
                              │
                              ▼
                    ┌───────────────────┐
                    │ Model Inspector   │
                    │ / Model Adapter   │
                    └─────────┬─────────┘
                              │
                              ▼
                    Model Representation
                              │
                              ▼
                    ┌───────────────────┐
                    │ Quantization      │
                    │ Analysis          │
                    └─────────┬─────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              Calibration          Quantization
                 Engine                Policy
                    │                   │
                    └─────────┬─────────┘
                              ▼
                    Quantization Parameters
                              │
                              ▼
                    ┌───────────────────┐
                    │ Quantization      │
                    │ Executor          │
                    └─────────┬─────────┘
                              │
                              ▼
                    Quantized Model
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              Accuracy Analysis    Quantization Report
```

---

## 6. Model Abstraction

The quantization engine should consume a normalized model representation instead of directly depending on a single model format.

The common abstraction should expose concepts such as:

```text
Model
 ├── Graph
 │    ├── Node
 │    ├── Input
 │    └── Output
 │
 ├── Tensor
 │    ├── Shape
 │    ├── DType
 │    ├── Layout
 │    └── QuantizationInfo
 │
 └── Metadata
```

Each model-format adapter is responsible for translating its native representation into the common abstraction.

Initial support may target a single format, but the architecture must not make that format fundamental to the quantization engine.

---

## 7. Quantization Pipeline

The primary PTQ pipeline should be:

```text
Load Model
    │
    ▼
Inspect Model
    │
    ▼
Select Quantization Policy
    │
    ▼
Prepare Calibration Dataset
    │
    ▼
Run Calibration
    │
    ▼
Collect Tensor Statistics
    │
    ▼
Calculate Quantization Parameters
    │
    ▼
Evaluate Quantization Plan
    │
    ▼
Apply Quantization
    │
    ▼
Validate Numerical Accuracy
    │
    ▼
Generate Report
    │
    ▼
Export Quantized Model
```

Each stage should have an explicit interface so individual stages can later be replaced or extended.

---

## 8. Quantization Schemes

### Initial Support

The initial implementation should support:

* INT8 activations.
* INT8 weights.
* FP32 → INT8 conversion.
* Per-tensor quantization.
* Per-channel quantization.
* Symmetric quantization.
* Asymmetric quantization.

Quantization parameters should be represented explicitly:

```text
scale
zero_point
quantized_min
quantized_max
axis
dtype
scheme
```

### Future Support

Potential future formats and strategies include:

* INT4.
* INT16.
* FP16.
* BF16.
* Weight-only quantization.
* Mixed-precision quantization.
* Group-wise quantization.
* Block-wise quantization.
* Hardware-specific quantization formats.

---

## 9. Calibration Engine

Calibration is one of the central components of the project.

The calibration system should provide a common interface:

```text
CalibrationMethod
 ├── MinMax
 ├── Percentile
 ├── Histogram
 └── Future Methods
```

### Initial Calibration Method

The first implementation should use **MinMax calibration**.

The engine should collect statistics such as:

```text
min
max
absolute_max
```

and derive:

```text
scale
zero_point
```

according to the selected quantization scheme.

Future calibration algorithms should be implemented as pluggable strategies rather than being embedded into the quantization executor.

---

## 10. Calibration Dataset

The calibration system must not assume a specific dataset format.

The dataset abstraction should support:

* Sample iteration.
* Model input generation.
* Sample count limits.
* Preprocessing.
* Batching.
* Dataset metadata.
* Reuse of existing datasets.

Conceptually:

```text
CalibrationDataset
        │
        ▼
Preprocessor
        │
        ▼
Model Input
        │
        ▼
Calibration Runner
        │
        ▼
Tensor Statistics
```

The tool should support configurable calibration sample limits because embedded and edge deployment workflows often require controlled calibration datasets.

---

## 11. Quantization Policy

Quantization decisions must be configurable rather than hardcoded.

A policy should be capable of defining:

* Quantizable operators.
* Quantizable tensors.
* Weight quantization scheme.
* Activation quantization scheme.
* Per-tensor/per-channel behavior.
* Symmetric/asymmetric behavior.
* Calibration method.
* Precision.
* Excluded nodes.
* Excluded tensors.
* Forced precision.
* Layer-specific overrides.

Conceptually:

```text
Default Policy
      │
      ├── Operator Rules
      ├── Tensor Rules
      ├── Precision Rules
      └── Override Rules
```

The policy system should later integrate naturally with the project's broader configuration architecture.

---

## 12. Quantization Eligibility Analysis

Before applying quantization, the tool should determine whether each graph component can safely be quantized.

Each relevant node or tensor should be classified as:

```text
Quantizable
Not Quantizable
Conditionally Quantizable
Unknown
```

The tool should provide a reason for the decision.

Example:

```text
Node: Conv_42
Status: Quantizable
Weight: INT8 per-channel
Activation: INT8 asymmetric
Reason: Supported operator and tensor types
```

Example:

```text
Node: CustomOp_7
Status: Not Quantizable
Reason: Unsupported operator
```

This analysis should reuse information provided by Model Inspector whenever possible.

---

## 13. Mixed Precision

Mixed precision should not be part of the first implementation, but the architecture must support it.

Example:

```text
Conv      → INT8
BatchNorm → FP32
ReLU      → INT8
MatMul    → INT8
CustomOp  → FP32
```

Real-world models may not tolerate full INT8 quantization without unacceptable accuracy loss.

Therefore, the quantization policy should be capable of expressing precision boundaries from the beginning, even if automatic mixed-precision optimization is implemented later.

---

## 14. Accuracy Evaluation

A model conversion is not considered successful merely because the model was converted.

The tool should compare:

```text
Original Model
      │
      ├── Reference Inference
      │
      ▼
Reference Outputs

Quantized Model
      │
      ├── Quantized Inference
      │
      ▼
Quantized Outputs
```

Initial numerical metrics should include:

* Absolute error.
* Relative error.
* Mean Absolute Error (MAE).
* Root Mean Square Error (RMSE).
* Maximum absolute error.
* Cosine similarity.

Where application-level evaluation is available, task-specific metrics should be supported through a pluggable evaluation interface.

---

## 15. Layer-Level Analysis

Global accuracy alone is insufficient.

The tool should support comparison of intermediate tensors and nodes:

```text
Node       MAE       Max Error       Cosine Similarity
------------------------------------------------------
Conv_1     ...
Conv_2     ...
Relu_2     ...
Conv_3     ...
Output     ...
```

This allows the user to identify where quantization error begins to accumulate.

Layer-level analysis should also support debugging deployment failures and investigating unexpected accuracy degradation.

---

## 16. Quantization Sensitivity Analysis

A future analysis mode should evaluate individual nodes or groups of nodes by quantizing them independently.

Example:

```text
Node A → accuracy loss: 0.01%
Node B → accuracy loss: 0.03%
Node C → accuracy loss: 2.71%
Node D → accuracy loss: 0.08%
```

This allows the system to identify sensitive layers and keep them in higher precision when necessary.

Sensitivity analysis should eventually become a foundation for automatic mixed-precision optimization.

---

## 17. Output

The tool should produce two major outputs.

### Quantized Model

The actual quantized model containing the required quantization representation and parameters.

### Quantization Report

The report should contain:

* Original model information.
* Quantization configuration.
* Calibration information.
* Quantized nodes and tensors.
* Excluded nodes and tensors.
* Quantization parameters.
* Operator coverage.
* Tensor statistics.
* Accuracy comparison.
* Error analysis.
* Warnings.
* Unsupported operations.
* Potential deployment issues.

Both human-readable and machine-readable report formats should be supported.

---

## 18. CLI

The first user interface should be CLI-based.

Conceptual commands:

```text
quant-tool inspect
quant-tool calibrate
quant-tool analyze
quant-tool quantize
quant-tool evaluate
quant-tool report
```

A complete pipeline should eventually be executable as a single command:

```text
quant-tool quantize model.onnx \
    --config quantization.yaml \
    --calibration calibration.dataset \
    --output model-int8.onnx
```

The exact CLI syntax should be finalized during implementation rather than prematurely fixed.

---

## 19. Configuration

Configuration should cover:

```text
Model
Calibration
Quantization
Operators
Tensors
Precision
Overrides
Evaluation
Output
```

Example:

```yaml
quantization:
  precision: int8
  activation:
    scheme: asymmetric
    granularity: per_tensor
  weight:
    scheme: symmetric
    granularity: per_channel

calibration:
  method: minmax
  samples: 500

evaluation:
  enabled: true
```

Configuration must remain independent of the underlying model format.

---

## 20. Model Inspector Integration

Model Inspector should be the primary source of model metadata.

Quantization Tool should consume:

* Graph information.
* Tensor metadata.
* Operator information.
* DType information.
* Shape information.
* Compatibility analysis.
* Model statistics.

Quantization Tool may extend this information with:

```text
Calibration Statistics
Quantization Parameters
Sensitivity Information
Quantization Decisions
```

The boundary between inspection and optimization must remain explicit.

---

## 21. Model Transformer Integration

Model Transformer is intentionally the third project in the ecosystem.

Quantization Tool should therefore expose quantization decisions without requiring a complete graph transformation engine.

Example:

```text
Quantization Tool
       │
       ▼
Quantization Plan
       │
       ├── Tensor A → INT8
       ├── Tensor B → INT8
       ├── Tensor C → FP32
       └── Tensor D → INT8
       │
       ▼
Model Transformer
       │
       ▼
Materialized Quantized Graph
```

This separation allows quantization algorithms to evolve independently from graph rewriting.

---

## 22. Model Tooling Core

A minimal shared library should eventually provide:

```text
Model abstraction
Graph abstraction
Node abstraction
Tensor abstraction
DType abstraction
Shape abstraction
Model metadata
Diagnostic system
Configuration interfaces
```

Potential future shared components include:

```text
Model loading
Model serialization
Graph traversal
Pattern matching
Statistics
Reporting
```

The governing rule is:

> **Extract abstractions when multiple projects demonstrably need them; do not design the entire shared framework upfront.**

---

## 23. Backend Architecture

The core must remain backend-independent.

Possible future model-format adapters include:

```text
ONNX
TFLite
TorchScript
MLIR
TensorFlow
```

Possible deployment targets include:

```text
CPU
ARM
x86
NPU
GPU
DSP
Custom Accelerator
```

The quantization engine must not contain Rockchip-, ARM-, CUDA-, TVM-, or IREE-specific assumptions.

Backend-specific constraints should be represented through adapters or policies.

---

## 24. Testing Strategy

### Unit Tests

Test:

* Quantization parameter calculation.
* Scale calculation.
* Zero-point calculation.
* Rounding.
* Clamping.
* Per-tensor quantization.
* Per-channel quantization.
* Symmetric quantization.
* Asymmetric quantization.
* Calibration algorithms.
* Policy resolution.

### Graph Tests

Test:

* Operator eligibility.
* Tensor propagation.
* Quantization boundary detection.
* Unsupported operators.
* Mixed-precision boundaries.

### Numerical Tests

Compare:

```text
FP32
vs.
Dequantized INT8
```

using known tensors and defined numerical tolerances.

### End-to-End Tests

```text
Model
 → Calibration
 → Quantization
 → Export
 → Inference
 → Evaluation
```

### Regression Tests

Every supported model should become a regression case once its expected behavior is established.

---

## 25. Performance Requirements

The tool is primarily an offline development and deployment tool, so correctness has priority over raw execution speed.

Nevertheless:

* Calibration should support batching.
* Large models should not require unnecessary full-model copies.
* Tensor statistics should be accumulated incrementally.
* Memory usage should be bounded where practical.
* Model processing should support streaming/chunked operations where the format permits it.
* Parallel calibration should be considered in later phases.

---

## 26. Error Handling and Diagnostics

Diagnostics should be first-class.

The tool should distinguish:

```text
Error
Warning
Info
Unsupported
Fallback
```

Example:

```text
WARNING:
Conv_17 cannot use per-channel activation quantization.
Falling back to per-tensor quantization.
```

Example:

```text
ERROR:
Calibration data does not provide required input: "image".
```

Example:

```text
WARNING:
Quantization caused significant degradation at node "Conv_42".
```

Diagnostics should be available in both human-readable and machine-readable forms.

---

## 27. Development Phases

### Phase 1 — Foundation

* Repository structure.
* Build system.
* Core model abstractions.
* DType abstraction.
* Tensor abstraction.
* Graph/node abstraction.
* Diagnostics.
* Basic configuration system.
* Initial model-format adapter.

### Phase 2 — Inspector Integration

* Consume Model Inspector output.
* Model compatibility analysis.
* Quantization eligibility analysis.
* Operator capability registry.
* Tensor selection mechanism.

### Phase 3 — Calibration

* Calibration dataset abstraction.
* Calibration runner.
* MinMax calibration.
* Tensor statistics.
* Calibration caching.
* Calibration report.

### Phase 4 — INT8 Quantization

* Symmetric quantization.
* Asymmetric quantization.
* Per-tensor quantization.
* Per-channel weight quantization.
* Quantization parameter generation.
* INT8 tensor conversion.

### Phase 5 — Quantization Policy

* Global policy.
* Operator rules.
* Tensor rules.
* Exclusion rules.
* Precision overrides.
* Configuration file support.

### Phase 6 — Evaluation

* FP32/reference execution.
* Quantized execution.
* Numerical comparison.
* Layer-level comparison.
* Accuracy report.
* Error localization.

### Phase 7 — Model Export

* Quantized model serialization.
* Quantization metadata.
* Validation after export.
* Re-import validation.
* Machine-readable report.

### Phase 8 — Sensitivity Analysis

* Node-level sensitivity.
* Tensor-level sensitivity.
* Quantization impact ranking.
* Sensitive-layer detection.

### Phase 9 — Advanced Quantization

* Histogram calibration.
* Percentile calibration.
* Mixed precision.
* Weight-only quantization.
* INT16/INT4.
* Advanced calibration strategies.

### Phase 10 — Transformer Integration

* Quantization plan generation.
* Q/DQ insertion.
* Quantized graph materialization.
* Fusion opportunities.
* Graph cleanup.
* Backend-specific transformation hooks.

### Phase 11 — Deployment Integration

Potential integrations:

```text
TVM
IREE
ONNX Runtime
TFLite
Custom Embedded Runtimes
```

Deployment-specific functionality should remain outside the core quantization engine.

---

## 28. MVP

The first usable release should deliberately remain small.

```text
Model
  │
  ▼
Model Inspector
  │
  ▼
INT8 PTQ
  │
  ├── MinMax calibration
  ├── Symmetric / asymmetric
  ├── Per-tensor
  └── Per-channel weights
  │
  ▼
Quantized Model
  │
  ▼
Numerical Evaluation
  │
  ▼
Quantization Report
```

The MVP should **not** attempt:

* Automatic mixed-precision optimization.
* QAT.
* Automatic sensitivity-based optimization.
* Extensive backend-specific support.
* A general graph transformation framework.

---

## 29. Long-Term Vision

The eventual ecosystem should provide a complete model optimization pipeline:

```text
                 ┌──────────────────┐
                 │   Original Model │
                 └────────┬─────────┘
                          │
                          ▼
                 ┌──────────────────┐
                 │ Model Inspector  │
                 └────────┬─────────┘
                          │
                   Model Understanding
                          │
                          ▼
                 ┌──────────────────┐
                 │ Quantization Tool│
                 └────────┬─────────┘
                          │
                   Quantization Plan
                          │
                          ▼
                 ┌──────────────────┐
                 │ Model Transformer│
                 └────────┬─────────┘
                          │
                   Optimized Model
                          │
                          ▼
                ┌─────────────────────┐
                │ Deployment Toolchain│
                └─────────┬───────────┘
                          │
                          ▼
                  Embedded / Edge Target
```

The long-term objective is not merely to create an INT8 converter.

**Quantization Tool should become the quantization intelligence layer of the broader model optimization ecosystem**, while keeping model inspection, graph transformation, and deployment as clearly separated responsibilities.

---

## 30. Development Priority

The overall project ecosystem should be developed in this order:

| Priority | Project               | Primary Responsibility                  |
| -------: | --------------------- | --------------------------------------- |
|    **1** | **Model Inspector**   | Understand the model                    |
|    **2** | **Quantization Tool** | Determine and apply precision reduction |
|    **3** | **Model Transformer** | Rewrite and optimize the model graph    |

This ordering is the current architectural roadmap.

> **Inspect first, optimize precision second, transform the graph third.**
