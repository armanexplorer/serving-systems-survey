# InferLine: latency-aware provisioning and scaling for prediction serving pipelines

A system which efficiently provisions prediction pipelines subject to end-to-end latency
constraints cost-efficiently

## Problem

Provisioning and managing prediction pipelines to meet end-to-end tail latency requirements at low cost and across heterogeneous parallel hardware

### Domain & Applications

We can compose this with prediction serving frameworks like Clipper and Tensorflow Server because InferLine will manage and provision resources for them

In DAGs, each vertex can be two kind:

- model -> like mapping images to objects in the image
- data transformation -> like extracting key frames of a video stream

And each edge is data flow between the vertices

InferLine runs on top of prediction serving systems which:

- supports scaling models with their number of replicas
- supports batched inference and maximum batch size setting
- use a centralized batched queuing system

Target applications: low-latency applications (SLO is the most important thing)

### Challenges

- Combinatorial configuration space
  - CPU or GPU
  - maximum batch size to meet SLO
  - replicate stage-wise
  - choices of one stage affects others stages of pipeline because of SLO (exponential grow in space)
- Queuing delays
- Stochastic and unpredictable workloads

Most hardware accelerators operate at vector level parallelism -> the first query in a batch is not returned until the last query is completed => we need **maximum batch size**

### Formulation

Goal: configure and manage multi-stage prediction pipelines SLO-aware and with min cost

## Solution

To use InferLine, developer should provide these:

- driver program -> provide application code the ability to async call the models
- sample query trace
- latency SLO

### Design & Implementation

- Two main component:
  - low-frequency planner
    - Profiler: stage-wise profile -> throughput of model for different hardware and maximum batch size
    - Estimator: Discrete Event Simulation (DES)
      - rapidly estimate SLO of given pipeline for the sample query trace
    - Planner: search space to choose these for each model (stage) of pipeline:
      - hardware type
      - replication
      - batch size
  - high-frequency tuner
    - uses network calculus to auto-scale each stage
    - only adjusts per-model replica to be fast and cost-effective

Planner:

- workload drift
- fundamental change in steady-state and long-term query arrival
- integrate new models
- integrate new resources

Tuner:

- runs at time scales 3 times faster
- monitor live traffic
- accommodate to unexpected query spikes
- for bursty and stochastic workloads

**Scaling factor**: conditional probability that a model will be queried given a pipeline query

## Evaluation

- SLO attainment and cost against baselines
- Robust to dynamics and bursty workloads
  - Accuracy of Estimator
  - Planner's response to varying arrival rates, SLOs, and burstiness factors
  - Tuner's ability to re-scale stages after change in arrival process
- Ablation study -> system benefits from both planner and tuner

### Setup

- Distributed cluster
- Amazon EC2
- driver client on _m4.16xlarge_ instance (64 vCPUs, 256 GiB memory, 25Gbps network)
- models on a cluster of 16 _p2.8xlarge_ GPU instances (8 NVIDIA K80 GPUs, 32 vCPUs, 488GiB memory, 10Gbps network)

### Baselines

- Coarse-grained baselines -> pipeline as black-box

### Performance Metric

- Cost
- SLO attainment
