# MArK: Exploiting Cloud Services for Cost-Effective, SLO-Aware Machine Learning Inference Serving

ML model serving system which employs `dynamic batching`, `predictive autoscaling`, and `serverless instances` to provide `low latency` meanwhile keep the `cost` at minimum (`cost-effective`)

## Problem

We need 10s-100s Miliseconds per query to deliver inference result -> Response time SLOs

service provisioning options:

- CPU
- memory
- GPU
- pricing models (on-demand along with spot instances)

Cloud services:

- Infrastructure as a service (IaaS) => best performance/cost + long instance provisioning
- Container as a service (CaaS) => like IaaS but less provision latency + worse performance/cost
- Function as a service (FaaS) => sacles fast + most expensive
- Tran

A. ML models are usually stateless => the potential of serverlsess functions

B. Many ML models have determinsitic inference time:

- get fixed-size input vectors
- input-independent control flows

B => we can do better resoruce planning and latency control

Robsut enough to:

- not depend on predictive scaling (in unpredictable, highly bursty workloads)
- handle stop interruptions
- handle unexpected demand surges

### Domain & Applications

prototpyed as general-purpose serving platform in AWS

#### ML Model Serving

Black-box:

- Clipepr
- Rafiki
- MXNet Model Server
- TensorFlow Serving

White-box:

- PRITZEL

#### Autoscaling Dynamic Workload in Cloud

Feedback Control Scaling:

- SageMaker
- Kuberenetes
-
- Clipper

Predective Scaling:

- VideoEdge

### cloud provisiing services

IaaS:

- on-demand
- spot isntances
- reserve
- burstable instances

CaaS
FaaS

### Challenges

- What service? IaaS, CaaS, or FaaS
- Should we use burstable instances?
- Big instance or small ones? -> small instances offer higher perfomrance-cost ratio than bigger ones, though the bigger ones show better latency
- GPU along with CPU -> GPU shows up to 40* speedup + TPU needs high batch to be cost-effective

### Formulation

Formulate problem in a latency-aware context

Formulate bathcing to find the maximum batch size and its wait time

## Solution

- IaaS as primary provisioning service
- Use FaaS as complementary
- Queue of client requests
- Batch by batch manager
- Proactive Controller to monitor workload metrics + check the health status of running instances
- Bouncer on each instance which monitors serving metrics and performs request admission control
- SLO monitor
- Multi-step workload prediction to predict max request rate in near future

### Design & Implementation

- Workload Prediction using LSTM
- Bathcing heuristic algorihtm
- Provisioning heuristic algorithm
- SLO tracking (feedback based scaling)
- Spot instance (complemented with burstable instances)
- Lambda concurrent warm-up to resolve the cold start problem

## Evaluation

evaluated with ML models in :

- image recongnition
- language modeling
- machine translation

### Setup

AWS EC2 and Lambda

### Baselines

SageMaker

### Performance Metric

Cost and Latency (SLO compliance)
