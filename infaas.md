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
    - updates **Metadata Store** with new compute and memory utilization, loading latencies and average inference latencies, along with the variants states
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

#### On arrival of a query

- check **active** variants with lowest load on its worker
- check **inactive** variants with lowest {loading model + inference} latency, choose the worker with lowest hardware utilization
- suggest closest target accuracy and latency variant

##### mitigate performance degradation

- co-locate variants on hardwares
- if interfere:
  - won't select them for new allocation
  - triggers mitigation process in the background:
    - if there were idle resource on the worker, migrate variant on them
    - if no, worker asks Controller Dispatcher to place variant on least-loaded worker
- if overloaded:
  - Model-Autoscaler assess whether scale to a different version or not

#### changes in query load

- revisit variant selection decision to check whether a different variant is more cost-efficient
- INFaaS auto scaling is a joint work between the **workers** and the **Controller**

##### Model-Autoscaler

- replicate, downgrade, upgrade model variant to minimize costs
- formulate this as ILP: minimizes the total cost of scaling actions for all variants to meet incoming query load
  - this ILP problem is NP-complete (large search place)
  - for Gurobi took 23 seconds across 50 model architecture, 50 seconds for 100 model arch
  - realtime requirements of latency-sensitive apps need **sub-second** response time to workload change
- instead, we use a greedy heuristic algorithm which use only a subset of variants
  - Model Autoscaler in each worker machine along with the model selection policy approximates the ILP
  - WHEN WE SCALE: checks headroom is below **slack-headroom** or not:
    - measures ratio of current load no each variant to its saturated throughput
    - measures ratio of combined load to combined saturated throughput
  - HOW WE SCALE: compare the cost and latency of upgrading to a variant with higher throughput and replicating a current one

##### VM-Autoscaler at controller

VM-Autoscaler adds a new worker with the corresponding hardware resource when:

- utilization of a HW is above some threshold in all workers
- variants on a particular HW are in **Interfered** state in all workers
- more than 80% of workers have variants in **Overloaded** state

### implementation

- Model-Autoscaler model variant deployment strategies:
  - Pytorch, Inferentia: Docker containers
  - TensorRT, Caffe2, TensorFlow - on GPU: Triton Inference Server
  - TensorFLow on CPU: TensorFlow Serving container

- Variant Generator using TensorRT and Neuron

**NOTE**: INFaaS builds on **Triton Inference Server** for GPUs and **Neuron SDK** for Inferentia which both provide spatial sharing of the resource

## evaluation

### experimental setup (testbed)

- controller on:
  - m5.2xlarge (8vCPU + 32GB DRAM)
- workers on:
  - inf1.2xlarge (one AWS Inferentia)
  - p3.2xlarge (one NVIDIA V100 GPU)
  - m5.2xlarge

- Intel Xeon Platinum 8175M CPUs
- Ubuntu 16.04
- Kernel 4.4.0
- Up to 10Gbps networking speed

### Baseline

- TensorFlow Serving (TFS)
- Triton Inference Server (TIS)
- Clipper
- AWS SageMaker (SM)
- Google AI Platform

### Model-variants

- 8 model families
- 22 architectures
- 175 variant
  - framework - TensorFlow, PyTorch, Caffe2
  - compilers - TensorRT, Neuron
  - batch size - 1 to 64
  - hardware platform - CPU, GPU, Inferentia

### Workload

- synthetic
  - flat and fluctuating with Poisson inter-arrival rate
- real
  - timing information form Twitter over a month collected in 2018
  - both diurnal and unexpected spikes
  - in each experiment one day of trace has been used

## Related works

- Swayam: only focus on VM-scaling (adding or removing worker machines) => latency of machines spawning
- TensorFlow Serving (TFS): one of the first production-level model servers for TF framework
- Clipper: generalized TFS to enable use of different frameworks and SLOs
- Pretzel, Nexus, InferLine: built upon Clipper for pipelines inference serving
- SageMaker, AI Platform, Azure ML: inference services with VM autoscaling
- Triton Inference Server: Optimize GPU inference serving and supports CPU models but requires static instance configuration
- DeepRecSys: static optimize of batching and HW choice for recommender systems, but needs developers to manually specify variant and mange and scale model resources
- ClockWork: reduces GPU latency variability by ordering queries based on SLO and only run one query at a time
- ModelSwitching: switch between models to keep correct prediction fraction, but the models are pre-loaded and does not consider heterogenous HW resource
- Tolerance Tiers: allows developers to trade-off between accuracy and latency in programming level
- Autoscale: uses scaling techniques and provides simple approach to maintain right amount of slack resources while meeting SLOs
- MArK: SLO-aware model scheduling and scaling using AWS Lambda to absorb unpredictable load bursts
- NVIDIA MPS: enabled efficient sharing of GPUs
- NVIDIA MIG: NVIDIA Multi-instance GPU

But INFaaS provides:

- easy to use and auto navigate the variants search space and dynamically leverage variants
- auto use of model optimizers to generate variants and auto select and manage deployed ones
- model-vertical scaling (in addition to model-horizontal and VM-level scaling)
- SLO-aware accelerator sharing

## notes

- a query in INFaaS is batch of requests
- time to instantiate a VM is: 20-30 seconds
- AWS EC2 Pricing when be normalized:
  - 0.031 per GB/s for CPU
  - 0.190 per GB/s for Inferentia
  - 0.498 per GB/s for GPU
- different workloads definitions:
  - flat and low: 4QPS
  - steady, high: slow increase from 650QPS to 700QPS
  - fluctuating: 4-80 QPS
