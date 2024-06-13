# VideoEdge: Processing Camera Streams using Hierarchical Clusters

A system that introduces **dominant demand** to indentify the best tradeoff between multiple resources and accuracy

## problem

large scale analytics of video streams

resource-accuracy tradeoff

- video analytics queries are a pipeline of computer vision components
- each component has many implementations (background substraction and DNN) and knobs (frame resolution, frame rate) with varying cost and accuracy

a video query has thousands of **combinations** of implementations and knob values

**query planning**: selecting the best combination of implementations and knob values

Resources:

- compute
  - handful of cores in private ones
  - hunders of cores in public ones

- network
  - camera to private cluster needs hunderd Kb/s to many Mb/s
  - private to public is few tens of Mb/s
  - edge is important (hierarchy of edges)

Vision primitives: each has 100s choices

So each query plan -> 1000s choices

Each chamera streams -> 10s of queries
Note: These camera streams reuse the same primitive components

Note: **object tracker** is key building block for many video queries

In video queires, we have Resource-Acuuracy profiles to resolve the trade-off between them

### targeted applications

### challenges

- exponentially **large serach space** (joint optimization) => use Pareto Band to drastically reduce search space + greedy accuracy at the cost of a bit more dominant demand
- **merging** conflicts => we should consider the change in aggregated accuracy against resource reduction

### hypothesis and objective

- Plan: accuracy depends only on the query plan
- Place: resource demands (cpu and network) depends on component placing
- Merge: common components can be merged for less resource => needs to have same plan and place

objective: maximizing the average query accuracy

### Formulation

Maximize sum of accuracies using BIP -> exponential time complexity even without merging

Greedy Heuristic

## Solution

‍1- Dominant Resource Demand (max S)

2- Greedy Heuristic

3- Merging

4- Pareto Band

5- Resource Pricing

Merging conditions:

- same camera feed
- common prefix in their pipelines

Pareto Band Definition:

For a particular query i, a configuration c is on the Pareto boundary if there does not exist any other configuration with lower demand and higher accuracy.

### Design & Implementation

1- Monitor Bandwidth
2- Resource-Accuracy Profiler

- efficinet profiler which is 100* cheaper than a naive exhaustive exploreation of the space

## Evaluation

Setup on 24 node Azure cluster

Each experiment is run 5 times, and the median result reported.

### Baselines

Compare against 4 approaches

- Fair Allocation
- VideoStorm
- Optimal Plan and Placement (using solving BIP with Gurobi) without merging AND with merging using brute-force (scales only to small deployments)

### Performance Metric

Average accuracy across all queries
