---
title: Inference Optimizations
---

## Overview

## Quantization

Quantization improves tokens per second and time to first token. When models are trained, they are usually in brain floating pt 16 or 16 bit floating pt. Prefill now runs on lower-precision Tensor Cores with higher FLOPS. And decode now loads half the data per value, effectively doubling memory bandwidth.

### Approaches
1. **Weights:** Quantizing the weights in linear layers, less sensitive
2. **Activations:** Activation outputs are quantized
3. **KV cache:** Attention calculation result are quantized
4. **Attention:** Attention layers of models, like softmax, usually more sensitive


As attention layers stack, compounding errors may occur, hence its wiser to quantize the linear layers

## Caching

During prefill, the KV cache is constructed, which is a key and value pair for each token in the sequence. The value of each new token depends on previous token

Caches live somewhere in the memory hierarchy, and each tier trades speed for capacity. When the cache outgrows fast VRAM (G1), it can be offloaded down to slower but larger tiers (host RAM, SSD).

Places to store KV cache:
![[memory-hierarchy.jpeg]]
*Memory hierarchy: fast and small at the top (GPU VRAM), slower and larger toward the bottom.*

### KV cache

The KV cache stores past keys and values so prefill does not need to recompute attention over the whole sequence every step.

- In attention, token `t` attends to all previous tokens via their **keys** and **values**.
- Without a cache, each decode step would recompute K and V for *every* past token.
- The K and V for past tokens never change, so cache them once and reuse → each step is O(n).

### Prefix caching

Imagine 2 requests with same prefix tokens, KV cache can be used to improve the TTFT on second request by skipping the key/value prefill calculation, and just read from KV cache instead.

**Example:** a chatbot where every request starts with the same 500-token system prompt.

- Request A: `[system prompt] + "What's the capital of France?"`
- Request B: `[system prompt] + "Write me a poem"`

Without prefix caching, both requests prefill the 500 system tokens from scratch, computing the same KV twice.
With prefix caching, the KV for those 500 identical tokens is computed once and reused, so Request B only prefills its own new tokens before jumping to decode, which lowers TTFT.

This works because of causal attention: a token's KV depends only on itself and the tokens before it, so an identical prefix produces identical KV regardless of what comes after.

### Attention techniques
1. **Flash attention:** Uses specialized kernels for attention to reduce read/write from and to memory. It relies heavily on kernel fusion.
2. **Paged attention:** It partitions the KV caches into pages and uses a lookup table, so the KV cache doesnt have to be stored in contiguous memory.

## Parallelism


### Tensor parallelism
Splits the tensor per layer across GPUs. It shares VRAM resources to make LLMs fast

### Pipeline parallelism
It splits the layers of model across multiple GPUs

### Expert parallelism
Enables mixture of experts to be served across different GPUs


## Disaggregation

### Prefill/decode separation

## References
