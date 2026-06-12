---
title: "TikTok (Feb 2026 - May 2026)"
tags: [internship, tiktok, ml-engineering, search, nlp]
---

**Team:** Search

**Period:** Feb 2026 – May 2026

## What I Worked On

I interned at TikTok Search team during 2026 spring. I primarily worked on serving and scaling location signals to the search engine for downstream retrieval and ranking.

### Small Language Model Optimization

I designed and optimized **Small Language Models** for **query rewriting** and **Point of Interest (POI) listwise matching**. Given a query with a local life service intent, the pipeline works as follows:

```mermaid
flowchart LR
    A["Raw Query"]
    B["Query Rewriting"]
    C["POI Name"]
    D["Recall (Location-Based Services)"]
    E["Candidate POIs\n+ metadata"]
    F["Listwise Matching"]
    G["Best POI"]

    A --> B --> C --> D --> E --> F --> G
```

1. **Query rewriting**: rewrite the raw query to a POI name (e.g. *best hotels in bali* → *bali*)
2. **Recall**: retrieve a list of candidate POIs and their metadata using location-based and vertical recall
3. **Listwise matching**: select the single POI that best matches the original query

To enable resource-efficient serving, I applied **model distillation** fine-tuning along with **GRPO** post-training to optimize a 1B parameter model.

I also brainstormed and applied novel data augmentations to compact and transform the training data, reducing deployment resource requirements and simplifying the scenario for the model.

I iterated across several techniques including **QLoRA** and **LoRA**, ultimately achieving 94% precision/recall.

### Location Signal Retrieval Engine

US local search queries had lacklustre recall due to upstream issues with the existing location-based services. I designed a **multimodal vertical-based recall engine** to resolve this, combining three recall modules:

| Module | Description |
| --- | --- |
| Google-based recall | Leverages Google Places signals as an additional recall source |
| Video anchor multimodal recall | Uses multimodal signals from video anchors to surface relevant POIs |
| Vertical places recall | Taps vertical-specific place indices for finer-grained coverage |

Together these modules improved US query coverage by **~15%**.

### Nearline Cache Layer

Location signals are served across 3 layers, offline, nearline and online. The ultimate goal is to have a great coverage when a search happens.

| Tier | Latency | How it works |
| --- | --- | --- |
| **Offline** | Daily job | Batch jobs (e.g. Spark) produce precomputed signals written to a data cache |
| **Nearline** | Milliseconds (cached) | A streaming pipeline (e.g. Kafka/Flink) continuously updates a fast cache so signals are pre-computed but recent |
| **Online** | Milliseconds | Signals are computed live at query time |

I deployed a **Kafka/Flink** nearline cache layer to continuously ingest and update location signals, serving them to the search engine in under **250 ms** per query. This had improved location signal coverage by 12%. After integrating it into the C++ search engine, while monitoring its metrics, we eventually ran A/B tests.

As a result of this increased coverage, this had resulted in an improvement of conversion rate by **1.1%**, while also being able to handle 16 queries per second (QPS).

### ETL Expansion

### Anchor Search with Multilingual BERT

To serve location signals in real-time, Small Language Models are not feasible as they take too long. I distilled and tuned a **BERT** model for named entity recognition and pointwise re-ranking, to serve 5 regions.

In contrast to the SLM in the nearline layer, it can afford to be **listwise** in POI matching stage as it receives the full candidate list and scores items relative to one another, capturing inter-candidate dependencies. This is more expressive but requires encoding all candidates together, making latency grow with list size.

At query time, that trade-off inverts. Anchor search runs **online**, so every millisecond matters. A **pointwise** model scores each candidate independently, which means it can run in parallel across candidates and its latency stays constant regardless of list size. The cost is that it cannot compare candidates against each other directly, but for NER and first-pass ranking this is an acceptable trade-off.

I distilled and fine-tuned multilingual BERT for this role, achieving **89% anchor search accuracy**.