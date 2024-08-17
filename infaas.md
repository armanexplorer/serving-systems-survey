# INFaaS: Automated Model-less Inference Serving

Automated model-less system for distributed inference serving, selects model variant, hardware, and model optimizations for each query

## Problem

The model developers has not choice but a naive autoscaling for their models, which is choose only one model variant and replicate it when load increases.

The search space is large because of:

- model variants
- load
- app requirements
- available resources

Many works ignored `model variants`, including:

- Clipper
- TensorFlow Serving
- AWS SageMaker
- Nvidia Triton (2020)
- Mark

### Challenges

- Diverse application requirements
- Heterogeneous execution environment
- Diverse model-variants

### Concepts

**Model Variant**: versions of already-trained models by `Graph Optimizers` (like TVM, TensorRT, Neuron) or methods like `layer fusion` and `quantization`. They may differ in:

- hardware platforms (Haswell, Skylake CPUs, V100, T4 GPUs, FPGA, and accelerators like Inferentia, TPU, Catapult, NPU)
- model architecture
- programming framework
- model graph optimizer
- hyperparameters (e.g., to be optimized for specific batch size)
- resource footprints (x50)
- latencies - including model load latency (x18.7), inference latency (x3700)
- costs (x32)
- accuracies (x1.6)

Also, the variants may differ only in `underlying hardware` resources or be optimized for specific `batch` size

**Hyperparameter Search**: this is what the training phase targets

**Application Requirements**: the application inference query requirements differ in

- latency
- cost
- accuracy
- privacy (new...)

## Solution

This work combines `VM-level` horizontal autoscaling with `Model-level` horizontal and vertical autoscaling, which means we will have multiple machines (VMs) and each one has multiple model variants.

The different model variant profiling costs will be amortized over long-term serving time in production settings

### Design Principles

1. support declarative API to get only high-level performance, cost or accuracy requirements
2. auto selection of model variant based on load and model state (for serving and scaling)
3. share hardware resources across model-variants and applications without violating performance-cost constraints
4. system design should be modular and extensible

### Architecture

- Front-end
- Controller
  - Dispatcher (choose worker)
  - VM Autoscaler
  - Model Register
- Worker
  - Dispatcher (choose hardware executor for selected model variant)
  - Model Autoscaler
    - replicate existing variants or vertically scale to a different model variant
  - Monitoring Daemon
- Variant Profiler
- Variant Generator
- Metadata Store
- Model Repository
- Model Variant Selection Policy

### Selecting and Scaling Model-Variants

INFaaS uses the model-variant selection policy in two cases:

- controller to decide about model variant => lies on the critical path of inference serving (impacts latency)
- worker Model Autoscaler when we have query load change to decide about scaling strategy

Should consider both static and dynamic data:

- static: accuracy, latency
- dynamic: resource contention, load

INFaaS used state machine to keep track of status of model variants:

- Inactive -> not loaded to any worker
- Active -> less than peak
- Overloaded -> peak throughput
- Interfered -> higher latency than profiled ones; because of the shared resource (caches, memory bandwidth, hardware threads) contention of co-located model variants

## Related works

Swayam: only focus on VM-scaling (adding or removing worker machines) => latency of machines spawning
