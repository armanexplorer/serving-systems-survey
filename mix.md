# Scrooge

- Nexus and InferLine relies on batching to highly utilize GPU and increase throughput
- Rim, Triton, and Gandiva uses Spatial Multiplexing (parallelism)
- InferLine and LLAMA only supports one type of GPU
- InferLine and Nexus mainly focused on CNN models (constant execution time)
- Nexus, INFaaS, and Rim used the DAG concept and Scrooge used that same (mDAG)
- Scrooge supports stateful variables
- Scrooge future work: adding the possibility to have a session (collection of streams) with different SLOs
- in Nexus and clipper, the modules of the pipeline are not stateful
- Scrooge assumes that a client input rate in a session is fixed
- the allocation algorithm is MILP formulation
- the elasticity of cloud made us to have choice for any configuration in any moment and so we can decouple the allocation problem from its placement
- scrooge only supports TensorFlow model server and others should be convert using tools like ONNX. It mentioned as future work to support TorchServe for PyTorch models.
- nexus cannot support the pipelines which have audio or stateful modules like pose, caption, audio, and acdet
- in scrooge concurrency means using spatial multiplexing
- scrooge made use of the word `chain` instead of pipeline somewhere
- InferLine and INFaaS only support one type of GPU
- Mark and INFaaS do not support module DAGs ?
- clipper has no resource adaption according ot dynamic input and has no spatial multiplex
- triton combines batching and spatial multiplexing
- Mark only worked on provisioning GPU modules not CPU, and uses heuristic methods to allocate
- Gandiva should be read because of adding spatial multiplexing and heterogenous hardware inspired scrooge with

----------------------------------------------------
InfAdapter:
Swayam (microsoft): 2017 -> high responsive
Cocktail (2022) -> high accurate
Jellyfish (2022) -> high accurate
ModelSwitching (2020) ->

autoscaling -> maybe high cost in scaling, or maybe not enough accuracy
mode-switching -> in high workload, least accurate and most accurate maybe not efficient
=> if we jointly resize and switch between models we can hit them

ME:
we can use also FaaS to mitigate over-provisioning along with predicting the workload as we see in Mark and InfAdapter

MLServer:
Multi-model serving is an important feature of MLServer, as it allows teams to run multiple models on the same server, reducing the memory footprint of the system and allowing for better CPU/GPU sharing.

2 points:
can we use cross application resource autoscaling?
whey we not use simultaneous multiple batch sizes when we have some fluctuated workloads?

good sentence in FA2:
GrandSLAm has static SLO partitioning => cannot adopt to change in workload of different execution paths

me:
what is the relation between making a app with different execution paths to multiple applications! and then we actually are using same infrastructure for different applications

Kubernetes and SageMaker -> use feedback control scaling -> based on customized rules

- problem: long provisioning delay -> needs over-provisioning

Predictive Scaling -> widely used for general workloads
complements with feedback control -> each in different time scale -> PS: hours to days, FCS: minutes

Mark:
auto-scaling dynamic workload on cloud
predictive scaling
IaaS and FaaS
SLO-aware
Heterogenous instances
Interruptible instances
Hardware Accelerator

The queue is intrusive and may include all records stored in a zone
inconsistency window of the cluster
The data becomes consistent eventually
In terms of the CAP theorem, the zone sync cluster can be categorized as an AP system.
cluster brain split
front-end web servers
application servers

I should set a threshold for latency to cut off the too long running inference requests (make experiments shorter)
Storage to store experiments results and retrieve in case of loss of data on cc and cg servers
