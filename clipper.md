# Clipper: A Low-Latency Online Prediction Serving System

## problem

ML is being used with different ML frameworks and in many applications. But, most ML frameworks and systems only address model **training** and not **deployment**.

Ml develoeprs should deal with deploying their models under differnet frameworks and for different scalings. So, **deployment**, **optimization** and **maintenance** of ML services is difficult and error-prone.

## solution

Clipper is a **general-purpose** **low-latency** serving system interposing between ML frameworks and applications. To this end, Clipper has **layered** modular architecture to simplify model deployment. This reduces the complexity of implementing the ML serving systeme stack.

Also, introduces **caching**, **batching** and **adaptive model selection** (+model composition) [and **online learning**!] which improves **latency**, **throughput**, **accuracy**, and **robustness** without change in ML frameworks.

Dynmically trade-off accuracy and latency under heavy query load

## deisng

Two layers:

- Model abstraction layer: API for using different ML frameworks -> brings trasparency between applications and models
- Model selection layer: sits above the model abstraction layer -> dynamically selects and combines models

Optimizations:

- caches -> low latency
- adaptive bathcing -> better throughput

Model selection layer:

- bandit + ensemble -> improve accuracy
- straggler mitigation -> improve latency

## notes

- Interactive Latency = < 100ms
- evaluate Clipper using **common ML benchmark datasets**
- low latency = < 20ms
