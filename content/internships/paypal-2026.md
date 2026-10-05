---
title: "PayPal (Summer 2026)"
tags: [internship, paypal, ml-engineering]
---

**Team:** PayPal ML engineering

**Period:** Summer 2026

## What I Worked On
I spent the summer working under the disputes domain, on large scale context retrieval & browser agents engineering

### Instant resolution context retrieval

**Goal:** When a new dispute comes in, we want to instantly recommend a resolution right away based on how similar cases were decided in the past. The 2 main types of dispute I dealt with were Item Not Received (INR) & Significantly Not As Described (SNAD).

I helped built a context retrieval system that searches across historical resolved cases to find the closest matches. The adjudication decisions from those neighbors are aggregated to produce a resolution recommendation for the new dispute.

```mermaid
flowchart TD
    A(["New dispute case"]) --> B["<b>Feature engineering</b><br/>200+ features from transaction details, behavioral signals, account age and other data sources"]
    A --> N["<b>Buyer note text</b><br/>rewritten by an SLM to preserve facts and clean phrasing"]
    B --> C["<b>Embedding generation</b><br/>dense vector from a fine-tuned text embedding model"]
    N --> C
    C --> D["<b>k-NN search</b><br/>L2 on structured features, similarity search on note text, across millions+ historical cases to find similar precedents"]
    D --> E["<b>Neighbor aggregation</b><br/>retrieves similar resolved cases and votes"]
    E --> F["<b>Local ML model</b><br/>trained on the retrieved candidates"]
    F --> G("<b>Resolution output</b><br/>final adjudication decision")
    T["<b>Teammate data reasoning</b><br/>improves feature quality and context interpretation"] -.-> B
    T -.-> C
    H["<b>Clustering heuristics</b><br/>improve neighbor selection and search relevance"] -.-> D
    H -.-> E
```

## Achievements:

**Case representation:** I engineered a set of 200+ engineered features drawn from transaction details, behavioral signals, account signals, etc. Next, I also used the buyer free-text note, which is often noisy, with typos, and contain spam texts. To enable a cleaner embedding representation, I leveraged a Small Language Model to `1.` rewrite each note to keep the facts and `2.` filter buyer note text that do not provide concrete value/

**Embedding generation:** Each case ends up with two vectors: a structured vector $q_s$ built from the engineered features, and a text embedding $q_t$ of the buyer note from a text embedding model.

**k-NN search:** A case is searched across millions of historical resolved cases.

**Recency decay weighting:** To account for the fact that an older dispute may be less useful in adjudicating a new dispute, each retrieved neighbor $i$ has its similarity score decayed exponentially by the number of days since it was resolved.

**In-context learning:** After retrieving the top $k$ candidates, I developed and trained an ensemble of classical ML models (LightGBM, XGB, etc) on the retrieved candidates to produce the final decision. This can be thought of as a test-time training/in-context learning approach.

**Improving retrieval quality:** I also researched and applied clustering heuristics on past teammate domain knowledge to refine which neighbors are selected and improve search relevance. These clusters contained condense knowledge across millions of past reasoning traces from teammates while they were processing the dispute cases.

**Data engineering:** Due to the diversity of INR cases, they tend to contain many outliers, including high-risk disputes and disputes with unusually large amounts, which made them poor candidates for instant resolution.
Through analysis and ablation testing, I set a threshold on the score from an online model and used it to filter these disputes out before retrieval. This helped keep the candidate pool clean and representative of the disputes seen at test time.

**Metrics and impact:** The system reached 90+% precision at an operating point of 10+%, while keeping the cost per case very low.
As a result, the improvements increased coverage of online cases by about 2.5x, reaching cases that the existing instant resolution solution could not handle.

### Agents for parcel case validation

**Goal:** Given a dispute case that relates to parcel shipping, we needed to validate the shipping address and determine the parcel's tracking status so we can provide more context for the review process.


#### Agents:

**Routing agent:** I worked on a routing agent that decides how to fetch tracking status for a given case, choosing between an internal carrier API and the browser agent. The problem was that the API could not cover certain carriers, and the browser agent could not operate on some carriers, so both of them are needed for improved coverage.

To ensure routing works effectively, I proposed that we used historical dispute data to engineer and derive carrier statistics and heuristics (CAPTCHA occurrences, anti-bot measures, page responsiveness, page layouts, etc), to accurately route each case to the path that has worked most reliably for it.

**Tool orchestration:** When we route a case to the browser path, a browser tool use is orchestrated to navigate the carrier's site, read the tracking page, and extract the delivery status and address.

**Self-learning:** When a routing decision turns out to be wrong (for example, the chosen path fails to return a valid status), the wrong outcome is fed back into the routing agent so future routing improves without manual tuning.

**Metrics**: The system achieved an overall accuracy of 75+%, with the capability to process up to 17k+ parcel shipping disputes weekly, greatly alleviating manual and repetitive work. Furthermore, this framework would be extended to other domains beyond disputes, which greatly strengthens its utility.