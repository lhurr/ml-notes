---
title: LLM Serving
tags:
  - ml
  - llm
  - inference
  - systems
---

## Overview


## Request Lifecycle

```mermaid
flowchart LR
    A["Client request"] --> B["Router / load balancer"]
    B --> C["Scheduler queue"]
    C --> D["Prefill"]
    D --> E["Decode loop"]
    E --> F["Streamed tokens"]
```

## Batching

### Static batching

### Dynamic batching

### Continuous batching

## Scheduling

### Admission control

### Preemption

### Chunked prefill

## Memory Management

### PagedAttention

### KV cache eviction and offloading

## Routing and Scaling

### Prefix-aware routing

### Autoscaling

### Multi-LoRA serving

## Serving Frameworks

| Framework | Notes |
| --- | --- |
| vLLM | |
| SGLang | |
| TensorRT-LLM | |
| TGI | |

