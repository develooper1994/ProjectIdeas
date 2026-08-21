# Visual ML & Intelligent System Tooling Platform

## 1. Idea

Create a modular ecosystem of developer and research tools that makes data-driven solution development dramatically easier.

The platform should not attempt to reinvent neural networks, machine learning, datasets, compilers, inference engines, cloud infrastructure, or deployment runtimes.

Its role is to **bring existing technologies together, provide a common workflow, hide unnecessary complexity, and make the entire process easier to use**.

The central philosophy is:

> **Do not reinvent the pipeline. Connect it, simplify it, automate it, and make it accessible.**

The platform should support neural networks, classical machine learning, signal processing, computer vision, rule-based systems, optimization, and other computational approaches.

It should work for both:

* users who want to build solutions visually with little or no code
* researchers and developers who need complete control and custom functionality

---

# 2. Core Philosophy

## 2.1 LCNC without becoming restrictive

Common workflows should require little or no code.

Users should be able to:

* select data
* prepare datasets
* construct pipelines
* select models or algorithms
* configure experiments
* run experiments
* compare results
* optimize
* export
* deploy

through visual interfaces or simple commands.

However:

> **Low-code/no-code must reduce complexity, not remove capability.**

Advanced users must be able to introduce custom components and code.

---

## 2.2 Neural networks are only one possible solution

The platform should not be designed around the assumption that every problem needs an NN.

Possible components include:

### Neural Networks

* CNN
* RNN
* Transformer
* MLP
* custom architectures
* custom layers
* custom losses
* custom optimizers
* custom training procedures

### Classical ML

* Linear Regression
* Logistic Regression
* Decision Trees
* Random Forest
* SVM
* k-NN
* Naive Bayes
* Gradient Boosting
* XGBoost
* LightGBM
* clustering
* dimensionality reduction

### Non-ML Approaches

* rule-based systems
* finite state machines
* thresholding
* statistical methods
* signal processing
* image processing
* feature engineering
* search
* optimization
* heuristics

A solution can combine them.

Example:

```text
Input
  ↓
Preprocessing
  ↓
Classical CV
  ↓
Feature Extraction
  ↓
Random Forest
  ↓
Rule Engine
  ↓
Decision
```

or:

```text
Input
  ↓
Neural Network
  ↓
Tracker
  ↓
Kalman Filter
  ↓
Rule Engine
  ↓
Decision
```

---

# 3. The Role of the Project

The project is fundamentally a **connector, coordinator, and simplifier**.

It should connect existing tools rather than replace them.

Examples:

```text
PyTorch
ONNX
scikit-learn
XGBoost
OpenCV
ncnn
MNN
ONNX Runtime
TVM
IREE
MLIR
CVAT
Label Studio
FiftyOne
...
```

The project should provide:

* common abstractions
* adapters
* orchestration
* experiment management
* visualization
* automation
* recommendations
* reproducibility
* easier configuration
* simpler user interfaces
* consistent CLI interfaces

It should not attempt to rebuild every underlying technology.

---

# 4. Modular Architecture

Independent tools are a fundamental requirement.

A user should **never need to install the entire ecosystem** just to use one capability.

The installation experience should resemble the modularity of large developer environments:

```text
Install only what you need.
Add components later.
Remove components you do not need.
```

For example:

```text
User A
├── Model Visualizer
├── Quantization
└── Model Transformer

User B
├── Dataset
├── Labeling
├── Experiment Runner
└── Benchmark

User C
├── Everything
└── Desktop Suite
```

A full suite is optional.

---

# 5. Independent Tools

Each major capability should ideally be usable as:

```text
CLI
Library / SDK
Plugin
Desktop UI adapter
```

A tool should not fundamentally depend on the desktop application.

Example:

```text
quantize
    ↓
CLI
Library
GUI
Plugin
```

All of them should use the same underlying implementation.

---

# 6. Possible Independent Tools

The following tools can become separate components.

## Data

```text
dataset
dataset-inspect
dataset-curate
dataset-split
dataset-validate
```

## Labeling

```text
label
auto-label
annotation-review
```

## Model

```text
model-visualizer
model-editor
model-transformer
model-analyzer
model-benchmark
```

## Optimization

```text
quantize
optimize
benchmark
```

## Experimentation

```text
experiment
sweep
automl
```

## Deployment

```text
export
deploy
package
```

## Infrastructure

```text
worker
runtime
device
```

These names are placeholders and can be refined later.

---

# 7. Standalone Development Strategy

The project should allow independent components to be developed whenever time is available.

This is important because the project is intentionally too large to build all at once.

A component should be useful independently even if the complete suite does not yet exist.

For example:

```text
Quantization Tool
        ↓
Useful standalone product
        ↓
Later integrated into Suite
```

This makes development incremental instead of requiring completion of the entire platform before anything becomes useful.

---

# 8. Suggested Independent Tools by Difficulty

The exact order can change after technical investigation, but a practical progression is:

## Level 1 — Small / Immediately Useful

### Model Transformer

Convert models between supported formats or representations.

Example:

```bash
model-transformer input.onnx --target ncnn
```

The first implementation should support only a small number of useful conversions.

The architecture should allow additional backends later.

### Model Inspector

Inspect:

* operators
* shapes
* parameters
* inputs
* outputs
* metadata
* unsupported operations

This can share infrastructure with the transformer.

### Model Benchmark

Run a model against a selected runtime/device and report:

* latency
* throughput
* memory
* model size
* device information

### Dataset Utilities

Simple CLI utilities for:

* inspect
* validate
* split
* statistics
* duplicate detection

These tools can become useful without the complete platform.

---

## Level 2 — Medium

### Quantization Tool

A major standalone component.

Possible workflow:

```text
Model
 ↓
Analyze
 ↓
Calibration
 ↓
Quantize
 ↓
Validate
 ↓
Benchmark
 ↓
Compare
```

Potential features:

* FP32 → FP16
* FP32 → INT8
* lower precision formats where supported
* static quantization
* dynamic quantization
* calibration
* mixed precision
* sensitivity analysis
* accuracy comparison
* performance comparison
* backend-specific export

The tool should not assume one runtime.

Conceptually:

```text
Quantization Engine
       ↓
Backend Adapter
 ┌─────┼─────────┐
ONNX  MNN       ncnn
```

---

### Dataset Curator

Automate:

* duplicate detection
* invalid sample detection
* outlier detection
* class imbalance analysis
* sample statistics
* filtering
* dataset splitting
* sample selection

The purpose is to reduce wasted training data and improve dataset quality.

---

### Labeling Automation

Instead of implementing every labeling feature manually, integrate existing tools and automate repetitive work.

Possible workflow:

```text
Raw Data
   ↓
Model / Detector
   ↓
Auto Label
   ↓
Human Review
   ↓
Correction
   ↓
Dataset Version
```

Possible automation:

* pre-labeling
* object detection proposals
* segmentation proposals
* face detection
* classification suggestions
* tracking-assisted annotation
* duplicate sample filtering
* uncertainty-based sample selection

The platform should eventually help answer:

> Which samples should a human label next?

This can lead to active-learning workflows.

---

## Level 3 — Large

### Experiment Runner

Handle:

* dataset
* pipeline
* model
* parameters
* runtime
* device
* execution
* metrics
* artifacts

with one reproducible experiment definition.

### Experiment Comparator

Compare multiple runs according to:

* accuracy
* latency
* memory
* model size
* throughput
* energy
* other domain-specific metrics

### Automated Optimization

Search automatically over:

* model parameters
* training parameters
* augmentation
* architecture
* precision
* backend
* runtime
* hardware

---

## Level 4 — Very Large

### Visual Pipeline Builder

Graphically construct:

```text
Data
 ↓
Preprocess
 ↓
Model
 ↓
Postprocess
 ↓
Evaluation
```

with arbitrary combinations of components.

### NN Editor

Visually create and modify neural network graphs.

### General Solution Editor

Go beyond NN graphs and allow classical ML, algorithms, rules, filters, and custom components in the same visual environment.

### Deployment System

Integrate:

* optimization
* quantization
* export
* compilation
* packaging
* target configuration
* deployment

---

# 9. Quantization as a First-Class Capability

Quantization should receive unusually high priority.

The project should not treat it as a simple:

```text
FP32 → INT8
```

conversion.

A better workflow is:

```text
              Quantization
                   │
        ┌──────────┼──────────┐
        ↓          ↓          ↓
      Analyze   Calibrate   Strategy
        │          │          │
        └──────────┼──────────┘
                   ↓
               Quantize
                   ↓
               Evaluate
                   ↓
               Benchmark
                   ↓
             Compare / Select
```

Quantization may become part of the automated optimization loop.

---

# 10. Quantization + AutoML

Quantization should eventually be considered one of the optimization dimensions.

For example:

```text
Experiment
    ↓
Search
 ┌──┼───────────────┐
 ↓  ↓               ↓
FP32 FP16          INT8
 ↓  ↓               ↓
Eval Eval           Eval
 └──┼───────────────┘
    ↓
Pareto Results
```

The system could automatically determine that:

```text
INT8:
- accuracy acceptable
- memory much lower
- latency much lower
```

and recommend it over FP32.

Quantization therefore becomes part of **solution optimization**, not merely model conversion.

---

# 11. Intelligent Recommendations

The platform should eventually understand the user's environment and provide useful recommendations.

It should not merely execute commands.

It should be able to say things such as:

```text
Available memory is low.
Consider reducing batch size from 64 to 16.
```

or:

```text
This experiment is unlikely to fit into available GPU memory.
Estimated requirement: 18 GB.
Available: 12 GB.
```

or:

```text
This model has unsupported operations for the selected deployment backend.
```

or:

```text
INT8 quantization is likely to provide a significant memory reduction.
```

or:

```text
The selected device is significantly slower for this workload.
Consider another worker.
```

or:

```text
This dataset is highly imbalanced.
Consider resampling or class weighting.
```

or:

```text
Your calibration dataset may not represent the deployment distribution.
```

The system should distinguish:

```text
Fact
Recommendation
Warning
Prediction
```

so that users know how confident the system is.

---

# 12. System-Aware Experiment Assistant

Eventually the platform could inspect:

* CPU
* GPU
* accelerator
* RAM
* VRAM
* storage
* operating system
* installed runtimes
* supported backends

and combine that information with the requested experiment.

Conceptually:

```text
Experiment
    +
Machine Profile
    ↓
Feasibility Analysis
    ↓
Recommended Configuration
```

Possible output:

```text
Recommended:
Batch Size: 16
Precision: INT8
Backend: CUDA
Worker: workstation-02

Reason:
Current machine has insufficient memory for the default configuration.
```

This could become one of the platform's most valuable differentiators.

---

# 13. Execution Model

The platform should separate:

```text
Experiment
    ↓
Execution Request
    ↓
Executor
```

An executor could be:

```text
LocalExecutor
RemoteExecutor
ContainerExecutor
PythonExecutor
NativeExecutor
```

This allows experiments to run on:

* the user's workstation
* another machine on the local network
* a dedicated server
* a cloud machine
* an embedded device

without changing the experiment definition.

---

# 14. Local Network Workers

The cloud should not be considered mandatory.

A research group may have:

```text
Researcher PC
      │
      ├── Local GPU workstation
      ├── CPU server
      ├── NAS
      └── Embedded devices
```

The platform should eventually allow these machines to act as workers.

Conceptually:

```text
                  Experiment
                      │
                 Scheduler
                      │
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
    Worker A      Worker B      Worker C
      CPU           GPU         Embedded
```

The same architecture can later extend to cloud workers.

---

# 15. GPU Support

GPU support should be treated as a core requirement rather than an optional later feature.

The architecture should separate:

```text
Device
 ↓
Compute Backend
 ↓
Runtime
```

Possible devices:

* CPU
* NVIDIA GPU
* other GPUs
* NPU / accelerator
* remote GPU

Possible backends may include:

* CUDA
* Vulkan
* Metal
* OpenCL
* backend-specific accelerators

The first implementation should prioritize actual use cases rather than attempting to support every GPU technology immediately.

---

# 16. Deployment Ecosystem

Deployment should support multiple runtimes rather than locking the project to one framework.

Important candidates include:

```text
ONNX Runtime
ncnn
MNN
TensorRT
OpenVINO
TFLite
TVM
IREE
MLIR
```

They should be treated according to their role.

For example:

```text
Interchange
    ↓
ONNX

Inference Runtime
    ↓
ONNX Runtime
ncnn
MNN
TensorRT
OpenVINO
...

Compiler / Optimization Ecosystem
    ↓
MLIR
IREE
TVM
...
```

Deployment should ultimately be:

```text
Model / Solution
       ↓
Optimize
       ↓
Quantize
       ↓
Convert
       ↓
Compile
       ↓
Package
       ↓
Deploy
```

---

# 17. Model Transformer

Model transformation should be an independent tool.

It should eventually handle:

* model import
* model export
* operator conversion
* graph transformation
* operator replacement
* constant folding
* graph optimization
* layout conversion
* precision conversion
* backend-specific transformations

The architecture should avoid implementing every format-to-format combination separately.

Prefer:

```text
        Format A
            ↓
       Common IR / Model
            ↓
    ┌───────┼────────┐
    ↓       ↓        ↓
 Format B Format C Format D
```

The exact internal representation should be investigated before implementation.

---

# 18. Custom Research Components

Research support is essential.

A researcher may invent:

* a new NN layer
* a new loss
* a new training procedure
* a new optimization method
* a new preprocessing algorithm
* a new metric
* an entirely new algorithm

The platform must therefore support custom components.

A custom component could expose:

```text
Name
Version
Inputs
Outputs
Parameters
Capabilities
Implementation
Runtime Requirements
```

and be usable in the same pipeline as built-in components.

---

# 19. Plugin System

A good plugin system should be a major architectural feature.

Plugins should eventually be capable of providing:

* components
* algorithms
* runtimes
* backends
* importers
* exporters
* quantizers
* dataset adapters
* labeling adapters
* execution providers
* CLI commands
* GUI panels

Plugins should not necessarily be limited to shared libraries.

Possible implementations:

```text
Native Plugin
Python Plugin
External Executable
Remote Plugin
```

The plugin interface should prioritize stability and versioning.

A manifest-based system is useful:

```yaml
name: example-plugin
version: 1.0
api_version: 1

capabilities:
  - importer
  - exporter
  - backend
  - benchmark
```

---

# 20. Python Strategy

Python should be supported because the research and ML ecosystem depends heavily on it.

However, Python should not become the foundation of the entire core system.

Preferred architecture:

```text
              C++ Core
                  │
        ┌─────────┼─────────┐
        ↓         ↓         ↓
     Native     Python    Remote
    Backend     Worker     Worker
```

C++ should own the core abstractions and orchestration.

Python should primarily provide access to ecosystems such as:

* PyTorch
* scikit-learn
* XGBoost
* NumPy
* scientific Python
* research code

Python environments should be isolated from the user's general Python installation.

This minimizes future dependency and environment problems.

---

# 21. Common Core

The ecosystem should share a small core rather than reinventing common concepts in every executable.

Likely core abstractions:

```text
Component
Graph
DataObject
Dataset
Pipeline
Model
Experiment
Run
Metric
Artifact
Device
Backend
Runtime
Worker
Plugin
```

The exact API should be designed only after the first proof-of-concept.

---

# 22. Desktop Suite

A desktop application is still useful, but it should be treated as an optional suite.

Possible interface:

```text
Project
├── Data
├── Labeling
├── Models
├── Pipelines
├── Experiments
├── Quantization
├── Benchmarks
└── Deployment
```

The desktop application should call the same tools and SDK available from the command line.

It should not contain a second implementation of the underlying logic.

---

# 23. CLI-First Principle

A major architectural principle:

> **Anything important should be possible without the GUI.**

For example:

```bash
model inspect model.onnx
model transform model.onnx --target ncnn
quantize analyze model.onnx
dataset validate dataset
experiment run experiment.yaml
benchmark model --device gpu
deploy build model --target arm-linux
```

The GUI should improve usability, visualization, and discovery.

It should not be the only way to use the system.

---

# 24. Web

The web should initially be considered mainly for:

* documentation
* examples
* tutorials
* interactive demonstrations
* product presentation
* model demonstrations
* benchmark visualization
* plugin discovery

A future web execution environment is possible, but it should not be the architectural center.

The primary development environment can remain desktop/native/CLI oriented.

---

# 25. Marketplace / Store

A component marketplace is an attractive future extension.

Possible contents:

```text
Models
Plugins
Backends
Datasets
Pipelines
Labeling integrations
Deployment targets
Custom components
```

Users could:

```text
Search
Install
Update
Remove
Publish
Share
```

This should remain a future phase.

The plugin architecture should be designed so that a marketplace can eventually be built on top of it.

---

# 26. Data and Labeling

Dataset management should be broader than simply splitting data into train/validation/test.

A complete workflow may be:

```text
Raw Data
  ↓
Inspect
  ↓
Curate
  ↓
Label
  ↓
Auto Label
  ↓
Review
  ↓
Clean
  ↓
Split
  ↓
Version
```

The system should support multiple data types:

* image
* video
* audio
* text
* tabular
* time series
* point cloud
* multimodal
* custom data

Initially, only the most useful data types need to be implemented.

---

# 27. Dataset Automation

Automation should reduce human effort rather than simply add more configuration.

Potential capabilities:

* automatic labeling
* automatic quality checks
* duplicate detection
* outlier detection
* class imbalance detection
* dataset recommendations
* sample prioritization
* active learning
* automatic split generation
* calibration-set selection

Example:

```text
10,000 samples
      ↓
Quality Analysis
      ↓
1,200 questionable samples
      ↓
Human Review
      ↓
Final Dataset
```

---

# 28. Experiment as a First-Class Object

An experiment should contain enough information to reproduce it.

Conceptually:

```text
Experiment
├── Dataset
├── Dataset Version
├── Pipeline
├── Model
├── Model Version
├── Parameters
├── Runtime
├── Device
├── Random Seed
├── Metrics
└── Artifacts
```

Experiments should be:

* reproducible
* comparable
* shareable
* resumable
* exportable

---

# 29. Automated Experimentation

Eventually the system should be able to search automatically.

Possible optimization dimensions:

```text
Model
Architecture
Learning Rate
Batch Size
Optimizer
Augmentation
Precision
Quantization
Backend
Runtime
Hardware
```

The system could perform:

```text
Search
 ↓
Run Experiments
 ↓
Evaluate
 ↓
Compare
 ↓
Select
```

Quantization should be one of the optimization dimensions rather than an isolated final step.

---

# 30. Feasibility and Recommendation Engine

A useful future subsystem:

```text
System Profile
       +
Experiment
       ↓
Feasibility Analysis
       ↓
Recommendations
```

It could identify:

* insufficient RAM
* insufficient VRAM
* unsupported operators
* incompatible runtime
* unsuitable hardware
* excessive model size
* unrealistic batch size
* expected execution bottlenecks
* inappropriate quantization
* inadequate calibration data

The engine should recommend alternatives.

Examples:

```text
Batch size 64 is unlikely to fit.
Recommended: 16.
```

```text
Selected runtime does not support one operator.
Try another backend or replace the operator.
```

```text
This model is too large for the selected embedded target.
Consider quantization or a smaller architecture.
```

```text
The selected GPU is inefficient for this workload.
Another available worker is recommended.
```

The goal is not to automate every decision.

The goal is to make the user aware of problems before expensive experiments start.

---

# 31. Reproducibility

Every experiment should record relevant information such as:

```text
Dataset Version
Pipeline Version
Model Version
Parameters
Source Revision
Runtime Version
Hardware
Software Environment
Random Seed
Metrics
Artifacts
```

A user should eventually be able to say:

> Run this experiment again.

and the system should reconstruct the required environment as far as possible.

---

# 32. Development Strategy

The project is too large to build as a single application.

It should therefore be developed as a sequence of independently useful components.

The development path should resemble:

```text
Small Standalone Tools
        ↓
Shared Core
        ↓
Integrated Workflows
        ↓
Visual Suite
        ↓
Automation
        ↓
Cloud / Marketplace
```

This allows development to continue even when there is no time to work on the complete system.

---

# 33. Recommended Project Phases

## Phase 0 — Research

Study:

* existing model converters
* quantization systems
* inference runtimes
* dataset tools
* labeling tools
* experiment managers
* workflow systems
* plugin architectures
* compiler ecosystems
* GPU backends

Main objective:

> Determine where this project adds value instead of duplicating existing software.

---

## Phase 1 — Standalone Tools

Build small tools that are immediately useful.

Suggested first candidates:

```text
1. model-inspector
2. model-transformer
3. model-benchmark
4. dataset-inspector
5. dataset-validator
6. quantization-tool
```

These tools should be useful even if the rest of the ecosystem does not exist yet.

---

## Phase 2 — Shared Core

Extract reusable abstractions:

```text
Component
Graph
DataObject
Dataset
Model
Backend
Runtime
Artifact
Experiment
Plugin
```

Do not overdesign these abstractions before there are real use cases.

---

## Phase 3 — Dataset + Labeling

Add:

```text
Dataset
Labeling
Curation
Automation
Versioning
```

Integrate existing labeling systems where appropriate rather than immediately replacing them.

---

## Phase 4 — Experiment Runner

Build:

```text
Experiment
 ↓
Execution
 ↓
Metrics
 ↓
Artifacts
 ↓
Comparison
```

Support local machines first.

---

## Phase 5 — Custom Plugin System

Enable developers and researchers to add:

* components
* algorithms
* backends
* importers
* exporters
* CLI commands
* custom training code

---

## Phase 6 — Visual Pipeline

Build the visual editor once the underlying abstractions are proven.

This avoids creating a GUI around an unstable architecture.

---

## Phase 7 — Optimization / AutoML

Introduce:

* hyperparameter search
* model selection
* quantization search
* backend selection
* hardware selection
* multi-objective optimization

---

## Phase 8 — Local Network Workers

Allow experiments to run on other machines in the user's environment.

---

## Phase 9 — Deployment

Integrate:

* export
* quantization
* model conversion
* compilation
* packaging
* deployment

with multiple runtimes and targets.

---

## Phase 10 — Desktop Suite

Combine the independent tools into a coherent desktop experience.

The suite should remain optional.

---

## Phase 11 — Web / Cloud

Later additions:

* remote execution
* cloud workers
* online demonstrations
* dashboards
* collaborative projects
* web interface

---

## Phase 12 — Marketplace

Eventually:

```text
Search
Install
Update
Publish
Share
```

plugins and other reusable assets.

---

# 34. Initial Technology Direction

The current technology direction is:

| Area               | Candidate                            |
| ------------------ | ------------------------------------ |
| Core               | C++20/23                             |
| Build              | CMake                                |
| Desktop UI         | Qt 6                                 |
| CLI                | C++                                  |
| Python             | Isolated worker/process              |
| RPC                | gRPC + Protobuf                      |
| Metadata           | SQLite                               |
| Serialization      | JSON + versioned schema              |
| Data interchange   | Apache Arrow                         |
| NN training        | PyTorch                              |
| Classical ML       | scikit-learn / XGBoost               |
| Computer Vision    | OpenCV                               |
| Model interchange  | ONNX                                 |
| General inference  | ONNX Runtime                         |
| Edge inference     | ncnn / MNN                           |
| GPU                | CUDA + Vulkan initially              |
| Compiler ecosystem | MLIR / IREE / TVM                    |
| Labeling           | Existing tool integrations initially |
| Dataset curation   | Existing tools + own orchestration   |
| Containers         | OCI-compatible approach              |
| Local execution    | Native process / worker              |
| Remote execution   | Worker protocol                      |
| Cloud              | Later                                |

These are **candidates, not final commitments**.

Technology selection should be validated through small PoCs.

---

# 35. GUI Technology Strategy

Qt remains a strong candidate for the eventual desktop application.

However:

> **The core architecture must not depend on Qt.**

The desired relationship is:

```text
                 Core SDK
              /     |      \
             /      |       \
          CLI     Qt GUI   Remote API
```

This preserves the possibility of replacing or adding frontends later.

The first GUI should therefore be considered a client of the core rather than the owner of the architecture.

---

# 36. Python Strategy

Python should be an ecosystem integration layer rather than the core architecture.

Preferred direction:

```text
C++ Core
   ↓
Python Worker
   ↓
PyTorch / sklearn / XGBoost / Research Code
```

Advantages:

* access to the existing ML ecosystem
* isolation from the core process
* easier environment management
* easier failure isolation
* same execution protocol can later be used remotely

Potential problems:

* dependency management
* version conflicts
* CUDA compatibility
* startup cost
* environment isolation

These should be managed rather than used as a reason to exclude Python.

---

# 37. First Proof of Concept

Before building the complete product, prove the architecture with a tiny system.

Target:

```text
CLI
 ↓
Core
 ↓
Execution Engine
 ↓
Python Worker
 ↓
Backend
```

A first experiment could be:

```text
Dataset
  ↓
Preprocessing
  ↓
Random Forest
  ↓
Evaluation
```

A second:

```text
Image
  ↓
Resize
  ↓
Neural Network
  ↓
Evaluation
```

Both should run through the same generic experiment mechanism.

If that works, the project has demonstrated that:

> **NN and non-NN solutions can coexist inside the same abstraction.**

---

# 38. MVP

A realistic first integrated MVP does not need everything.

Potential scope:

```text
Generic Graph
    ↓
Dataset Input
    ↓
A few Components
    ↓
Local Execution
    ↓
Experiment
    ↓
Metrics
    ↓
Artifact Storage
```

Possible first backends:

```text
PyTorch
scikit-learn
```

Possible first standalone tools:

```text
model-inspector
model-transformer
quantize
benchmark
dataset-inspector
```

No cloud.
No marketplace.
No full AutoML.
No complete labeling platform.
No huge deployment matrix.

---

# 39. What Should Be Avoided

Do not initially try to:

* recreate PyTorch
* recreate ONNX
* recreate ncnn
* recreate MNN
* recreate a complete labeling platform
* recreate a cloud provider
* implement every model format
* implement every GPU backend
* implement every deployment target
* implement a complete MLOps system
* build the entire desktop suite first

The project's value comes from **integration and simplification**, not from replacing the entire ecosystem.

---

# 40. Long-Term Vision

The final user experience should be approximately:

```text
I have a problem.
      ↓
I provide my data.
      ↓
I inspect / label / curate it.
      ↓
I construct or select a solution.
      ↓
I configure an experiment.
      ↓
The system checks whether it is feasible.
      ↓
It recommends improvements.
      ↓
I run the experiment.
      ↓
It compares the results.
      ↓
It can automatically explore alternatives.
      ↓
I select the best solution.
      ↓
I quantize / optimize / transform it.
      ↓
I benchmark it on my target.
      ↓
I export and deploy it.
```

The user should be able to perform all of this visually.

The developer should be able to perform the same operations through CLI, SDK, scripts, or plugins.

The researcher should be able to introduce completely new components without waiting for the platform itself to implement them.

---

# 41. Guiding Principles

1. **Do not reinvent established tools without a compelling reason.**
2. **Integrate first, replace only when necessary.**
3. **Everything important should be scriptable.**
4. **Every major capability should be independently usable where practical.**
5. **The full suite should be optional.**
6. **The GUI should be a client of the core, not the core itself.**
7. **Python should be supported without making it the architectural foundation.**
8. **GPU support should be designed in from the beginning.**
9. **Quantization should be a first-class capability.**
10. **Labeling and dataset curation should be treated as important engineering problems.**
11. **Custom research components must be supported.**
12. **Plugins should be a major extension mechanism.**
13. **Local compute should be a first-class target.**
14. **Cloud should be an extension, not a requirement.**
15. **Experiments must be reproducible.**
16. **Automation should remove repetitive work, not remove control.**
17. **System-aware recommendations should help users avoid expensive mistakes.**
18. **The architecture should be extensible, while the initial implementation remains small.**

---

# 42. Current Scope

This is a long-term project.

The complete vision is intentionally much larger than what can be implemented simultaneously.

The practical strategy is:

```text
Independent Tool
      ↓
Useful Component
      ↓
Shared Core
      ↓
Integrated Workflow
      ↓
Visual Suite
      ↓
Automation
      ↓
Cloud / Marketplace
```

The project does not need to be built from top to bottom.

It can grow organically as individual components become useful.

---

# 43. Immediate Candidates

The most promising early standalone projects are currently:

```text
Model Inspector
Model Transformer
Model Benchmark
Quantization Tool
Dataset Inspector
Dataset Validator
Dataset Curator
Labeling Automation
```

Among these, **Model Transformer** and especially **Quantization** appear to be strong candidates for early independent development because they can provide value without requiring the complete platform.

The next architectural milestone is to determine which of these tools can share a minimal common core without creating unnecessary coupling.
