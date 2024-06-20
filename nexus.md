# Nexus: A GPU Cluster Engine for Accelerating DNN-Based Video Analysis

## Problem

Goal: Make using GPUs very low-cost, i.e., have high accelerator utilization and acceptable
latency.
To achive goal, we should do `cluster-scale` reousrce management to be able to schedule GPUs in `detail`, executin framgemnts of DNNs and reasoning on `group of DNN requests` to co-schedule. Great for cluster of GPUs (100 GPU)

### Domain & Applications

Cloud-scale video analysis service supporting 1000s tenants to see 1000s streams concurrently

DNN: Network (also called model) of layers or kernels (dense linear algebra computations, like matrix multiplications and convolution)

Kernel execution -> needs MFLOPS to GFLOPS (10^3 to 10^6) and 1MB to 100s MB

Loading Model (Network) into memory -> 100(s)ms to 1(s)sec

Modern GPU devices provide over 100 TFLOPS (TeraFlops -> 10^12 Floating-point operations per second)

traffic monitoring applications

game-stream analysis

live applications -> 10ms - 100ms

batch applicationos  -> serveral hours

NVIDIA V100 -> 125TFLOPS

### Challenges

- different types of networks on same GPU
- specify groups of DNNs feed into each other
- batch the inputs of DNNs
- transfer learning -> small customized models -> moslty identical networks (but not identical)

Sustainted high utilization: Funnel adequete work to each accelerator + Group right type of work in the right place

- Utilizing Accelerators
- Placing, pakcing and batching DNNs

### Formulation

Goal: attain high execution efficiency on GPU clusters while serving video analysis requests within a specified latency SLO.

Integer Programming to co-locate sessions
Used CPLEX to solve our Integer Programming problem (incldues some non-linear constraints) -> expensive => go for greedy

## Solution

3 Tecchniquse to resolve challenges

- batching-aware scheduler
- querys as related DNNs queries
- batching parts of network with different batch sizes

Let the user specify the related DNN-based tasks joinlty -> query
Let the user to determine the latency SLO for whole query
Find out the common subgraphs between different models and apply batch on them -> good for specialized models which only differ on output layer

We need:

1. squishy bin-packing
2. complex query scheduling
3. Adaptvie batching

### Design & Implementation

Management Plane: ingest and deploy models by developers
control Plane: via global scheduler (interacts with underlying cluster resource manager) -> resource allocation and scheduling
Data Plane: Nexus runtime -> Nexus library instances + backend moduels -> dispatch and executes requests

#### Management Plane

- Store moedls in model database
- sample dataset to produce batching profile (latency and memory for each batch size)

#### Control Plane

- add frontend or backend nodes using global scheduler (uses load statistics)
- call epoch scheduler to determine models, their batch size, and their backends each backend wiht its execution schedule
- backend uses multiple threads to queue frontend requests and run according its execution schedule
- So, Allocation, scheduling and routing will update at 30-60s level
- uses early-drop policy as admission control

#### Data Plane

- Application instance receives the request and send DNN infernece through Nexus Libaray API
- Nexus library (on frontend) checks routing table to find backend and dispatch the request accordingly
- Application pakcs the results to conclude the query result

Schedule -> (models + arrival rate + SLO + batching profiles) => number of GPUs for each sesssion + co-located (bin-pack) remained workload

Containerzied system
10k C++ code

## Evaluation

### Setup

### Baselines

- Clipper
- TenserFlow

### Performance Metric

Maximal request rate (throughput) processed by serving system
