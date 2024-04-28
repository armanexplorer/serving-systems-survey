# Clipper: A Low-Latency Online Prediction Serving System (2017)

A **general-purpose** **low-latency** prediction serving system interposing between apps and ML frameworks

![alt text](image.png)

## problem

ML is being used with different ML frameworks and in many applications. But, most ML frameworks and systems only address model **training** and not **deployment**.

ML develoeprs should deal with deploying their models under differnet frameworks and for different scalings. So, **deployment**, **optimization** and **maintenance** of ML services is difficult and error-prone.

Domain:

- object recongnition
- speech recongnition

## solution

- simplify model deployment by **layered modular architecture**
- reduce and bound latency, and improve throughput, accuracy, and robustness by **caching, batching, and adaptive (online) model selection**

Also, introduces **caching**, **adaptive batching** and **adaptive model selection** (+online learning (user feedback) +model composition) which improves **latency**, **throughput**, **accuracy**, and **robustness** without change in ML frameworks.

Dynmically trade-off accuracy and latency under heavy query load

## formulation

batch: allowing users to specify a latency objective -> to maximize throughput (with high utilization)

## deisng

The main contributinos of Clipper:

### Two layers

- Model abstraction layer: API for using different ML frameworks -> brings trasparency between applications and models
- Model selection layer: sits above the model abstraction layer -> dynamically selects and combines models

### Optimizations (latency, throughput)

- caches per-model basis -> reduce latency
- adaptive bathcing -> improve throughput

### Model selection layer (accuracy, robustenss, latency)

- bandit + ensemble for each user/session -> improve accuracy (and robustness)
- straggler mitigation -> improve latency

## implementation

- with Rust
- support for frameworks:
  - Apache Spark MLLib
    - Scikit-Learn
    - Caffe
    - TensorFlow
    - HTK

## evaluate

Object Recognition datasets:

- MNIST
  - 70KB
  - 28x28
  - 10 lables
- CIFAR-10
  - 60KB
  - 32x32x3
  - 10 lables
- ImageNet
  - 1.26MB
  - 299x299x3
  - 1000 lables

Speech Recognition datasets:

- TIMIT (with HTK framework)
  - 630 spekers
  - 8 dialects of English
  - 5 sec
  - 30 lables

All experiments were conducted on a single server:

- 2 Intel Haswell-EP CPUs
- 256 GB of RAM
- Ubuntu 14.04 on Linux 4.2.0
- Nvidia Tesla K20c GPUs
  - 5 GB of GPU memory and 2496 cores

Result:

- <20ms latency
- tradeoff accuracy and latency under heavy load
- compared to Google TensorFlow Serving System
- minimal performance cost

## notes

- Interactive Latency = < 100ms
- evaluate Clipper using four **common ML benchmark datasets**
- substantial latencies = 50-100ms
- when predictions can influence future queries (like content recommendation):
  - offline evaluation can be heavily biased
  - A/B testing is statistically inefficient
- Unless otherwise noted -> can be used to say something generally is true unless specified
