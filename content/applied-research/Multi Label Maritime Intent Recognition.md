---
title: Data-Free Multi-Label Intention Recognition
tags:
  - paper
  - nlp
  - maritime
  - agentic-ai
---

## Motivation and reflection

The goal was to tackle a harder problem: instead of predicting a single intent from a user query, we needed to handle **multi-label** intent recognition, where a single maritime specific query can simultaneously map to multiple maritime contexts (e.g. ETA prediction *and* fuel consumption estimation). On top of that, we wanted to eliminate the need for any manually annotated training data entirely.

This pushed me deeper into contrastive learning, LLM-based synthetic data generation, and the design of custom loss function to support this, a three-stage modular pipeline that I think generalises well in maritime to any agentic AI routing problem.

## Summary

The DMTC pipeline has three core components:

1. **Synthetic data generation** : We stochastically guide a SLM to generate diverse, realistic user queries covering eight maritime transportation scenarios (ETA, berthing, fuel consumption, vessel trajectory, risk evaluation, etc.), eliminating the need for manual annotation entirely.

2. **Sentence-T5 encoding** : Each query is encoded with a Sentence-T5 model to produce compact, high-quality semantic embeddings. Ablations show Sentence-T5 outperforms BERT, RoBERTa, MPNet, and MiniLM on this task.

3. **Online Focal-Contrastive (OFC) loss** : A lightweight MLP classifier is trained with a novel OFC loss that combines focal weighting (emphasis on hard-to-classify samples) with contrastive objectives (maximising inter-class separability). This yields **+0.98%** accuracy over standard contrastive loss and **+2.07%** over the same pipeline without contrastive learning.

### Online Focal-Contrastive (OFC) Loss

The OFC loss mines hard and semi-hard pairs from each batch, then applies a focal-weighted binary cross-entropy directly on cosine similarity scores, down-weighting easy pairs and focusing gradient on the ambiguous boundary cases.

```python
def forward(self, sentence_features, labels):
    sim = F.cosine_similarity(embed_a, embed_b).clamp(1e-7, 1 - 1e-7)

    # mine hard pairs: positives above min(D_neg), negatives below max(D_pos)
    h_pos = sim[(labels == 1) & (sim > sim[labels == 0].min())]
    h_neg = sim[(labels == 0) & (sim < sim[labels == 1].max())]

    # semi-hard: top-p% of remaining pairs near the boundary
    sel_pos = torch.cat([h_pos, topk(o_pos, p, largest=True)])
    sel_neg = torch.cat([h_neg, topk(o_neg, p, largest=False)])

    # focal BCE: −α [(1−p)^γ log p + p^γ log(1−p)]
    loss = focal_bce(sel_pos, y=1) + focal_bce(sel_neg, y=0)
```

### Results

DMTC was evaluated on a hand-curated test set of **918 samples** across eight maritime AI model categories.

| Model | Accuracy (%) | Hamming Loss (%) | AUC (%) | F1-Score (%) |
|-------|-------------|-----------------|---------|-------------|
| GloVe + SVM | 31.70 | 14.39 | 84.72 | 59.76 |
| BERT + LSTM | 61.44 | 6.69 | 95.79 | 83.58 |
| GPT-4 prompting | 32.14 | 16.00 | : | 57.66 |
| **DMTC** | **70.15** | **5.35** | **95.92** | **86.06** |

DMTC substantially outperforms all baselines, demonstrating that a lightweight, data-free modular pipeline can beat large models on fine-grained complex multi-label intent classification.

## Paper

*arXiv preprint arXiv:2511.03363, submitted November 2025*

<iframe
  src="A_Modular_Data-Free_Pipeline_for_Multi-Label_Inten.pdf"
  width="100%"
  height="900px"
  style="border: none; border-radius: 4px;"
  title="A Modular, Data-Free Pipeline for Multi-Label Intention Recognition in Transportation Agentic AI Applications"
>
  <a href="A_Modular_Data-Free_Pipeline_for_Multi-Label_Inten.pdf">Download PDF</a>
</iframe>

## Citation

```bibtex
@article{zhang2025modular,
  title={A Modular, Data-Free Pipeline for Multi-Label Intention Recognition in Transportation Agentic AI Applications},
  author={Zhang, Xiaocai and Lim, Hur and Wang, Ke and Xiao, Zhe and Wang, Jing and Lee, Kelvin and Fu, Xiuju and Qin, Zheng},
  journal={arXiv preprint arXiv:2511.03363},
  year={2025}
}
```
