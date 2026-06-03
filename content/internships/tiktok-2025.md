---
title: "TikTok (Summer 2025)"
tags: [internship, tiktok, ml-engineering, data-engineering]
---

**Team:** T&S Algorithms

**Period:** Summer 2025

## What I Worked On

### Large Scale Feature Selection

To scale down 2,000+ video features for quick dataset rollout, I applied [mRMR](https://github.com/smazzanti/mrmr) (minimum Redundancy Maximum Relevance), a greedy feature selection algorithm that uses **mutual information** as the scoring mechanism for both relevance and redundancy.

The objective is to select a subset $S$ of $k$ features that jointly maximizes relevance to the target while minimizing pairwise redundancy:

$$\max_{x_i \notin S} \left[ I(x_i;\, y) - \frac{1}{|S|} \sum_{x_j \in S} I(x_i;\, x_j) \right]$$

**Algorithm (greedy):**

1. Compute $I(x_i; y)$ for all features. Select the feature with the highest mutual information with the target $y$.
2. For each remaining feature, compute the mRMR score: relevance $I(x_i; y)$ minus mean redundancy $\frac{1}{|S|}\sum_{x_j \in S} I(x_i; x_j)$ with the already-selected set $S$.
3. Add the feature with the highest score to $S$.
4. Repeat steps 2–3 until $k$ features are selected.

> $$I(X; Y) = \sum_{x,y} p(x,y) \log \frac{p(x,y)}{p(x),p(y)}$$, which measures how much knowing one variable reduces uncertainty about the other

This reduced training memory and fine-tuning time, and helped regularize against overfitting by removing highly correlated features. Eventually, I helped accelerate dataset rollout of Spark feature engineering jobs, reducing 2,000+ video features by 15%.

### GPU Resource Circulation

The goal was to free up a pool of target GPU types (e.g. H100s) for better ciruclation and distribution. This is done by swapping them out for compatible alternatives held by those stale models.

I wrote code to crawl model metadata to identify stale lineages: models whose utilization was below a threshold and had no active downstream traffic. This surfaced a pool of candidate models that could safely yield their GPUs.

The core challenge was compatibility: a model deployed on multiple GPU types could only be safely migrated to one type completely it had already been used with. For example, if model A used both L20, V100 and H100, etc and we wanted to extract out H100s, we could swap it for L20s and V100s without any negative effects.

To operationalize this, I built a **GPU compute conversion graph** where each node is a GPU type and each edge connects two types that a model has been co-deployed on. Then, finding valid substitutes for a target GPU then becomes a graph traversal problem, a DFS from the target node surfaces all reachable GPU types that are transitively compatible, giving the swap logic a candidate pool to work from automatically.

Apart from this, I also wrote code to estimate the GPU card to allocate different teams based on several metrics, such as utilization, estimated scale up, model time out rates, etc.

### Moderation Service Level Indicator Metrics

I was also in charge of maintaining some of the team's important dashboards. In weeks within my internship noticed inconsistencies in reported service level indicator metrics some dashboards showed sudden performance drops while others remained stable. Inaccurate metrics risked misleading teams about the health of the moderation system, so my task was to identify the root cause.

I traced the pipeline end-to-end, examining feature generation jobs, aggregation logic, and downstream metrics computation. The bug caused certain metrics to be overcounted during aggregation, which skewed downstream dashboards.

**Fix:**

1. Root cause analysis across multiple pipeline stages
2. Reproduced the issue locally using sampled data
3. Corrected the filtering logic and validated with historical data reprocessing

### Multi-Agent Livestream Analysis

Piloted and evaluated a multi-agent framework for processing TikTok livestreams, where multiple specialized agents collaborate to analyze a stream. The agents included multimodal agents, embedding agents, ASR agents, and other specialized agents to pass video frames, transcribed audio, and video text signals to identify if a video was violative. I also developed heuristics to identify if a video was selling fraudulent goods.

