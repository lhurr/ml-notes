---
title: Maritime AI Research
tags:
  - paper
  - nlp
  - maritime
---

## Motivation and reflection

I worked on this research paper in 2024 during my stint at the **Institute of High Performance Computing (IHPC), A\*STAR Singapore**. The goal was to develop a model understand complex queries in the maritime domain. This model was used as part of a larger framework that helps users navigate the maritime AI/data repository that IHPC developed. To that end, I experimented with **lightweight language models** coupled with **cost-sensitive learning** to address the challenge of limited domain-specific training data. This was my first time getting my hands dirty into AI research, model fine-tuning and manuscript writing

## Summary

The proposed method fine-tunes a **ConvBERT** model with **Focal Loss** to learn text representations, followed by a simple **MLP** classifier to predict intent. We achieved an accuracy of **~0.99** while keeping training and inference costs low. We have also experimented with other notable deep learning approaches including LSTM, GRU, Temporal Conv nets architecture.

### ConvBERT architecture

ConvBERT replaces a subset of standard self-attention heads with **span-based dynamic convolution** heads, forming a *mixed attention* module.

**Step 1: Multi-Head Self-Attention (global heads)**

Standard scaled dot-product attention over the full sequence:

$$\text{Attention}(Q, K, V) = \text{softmax}\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$

where $Q = XW^Q$, $K = XW^K$, $V = XW^V$. Each global head $i$ computes:

$$\text{head}_i = \text{Attention}(XW_i^Q,\, XW_i^K,\, XW_i^V)$$

**Step 2: Span-Based Dynamic Convolution (local heads)**

Instead of attending to the full sequence, each token only looks at a **local window** of $2w+1$ neighbours centred on itself: positions $[i-w,\, i+w]$. This is the **span**.

The attention weights over that window are computed from the token's own query $\mathbf{q}_i$, making the kernel **dynamic**: two different tokens will produce different weights even over the same span of text:

$$\hat{A}_i = \text{softmax}\!\left(\frac{\mathbf{q}_i\, K_{[i-w:i+w]}^\top}{\sqrt{d_k}}\right)$$

The output is then a weighted sum of the local values using those weights:

$$\text{SpanConv}(X)_i = \hat{A}_i\, V_{[i-w:i+w]}$$

The key payoff over full self-attention: complexity drops from $O(n^2)$ to $O(n \cdot w)$, since each token interacts with only $2w+1$ neighbours instead of all $n$ tokens.

**Step 3: Mixed Attention output**

Global and span heads are concatenated and projected:

$$\text{MixedAttn}(X) = \text{Concat}\!\left(\text{head}_1,\,\ldots,\,\text{head}_g,\;\text{SpanConv}_1,\,\ldots,\,\text{SpanConv}_s\right)W^O$$

### Pre-training & fine-tuning
1. ConvBERT is pre-trained using masked language modelling. 
2. E.g The cat sat on the [MASK], and the ground truth is *car*, then the loss is 

$$\mathcal{L}_{\text{MLM}} = -\log P(w_{\text{true}} \mid \tilde{X})$$

More generally, the loss is averaged over all masked positions $\mathcal{M}$ in the sequence:

$$\mathcal{L}_{\text{MLM}} = -\frac{1}{|\mathcal{M}|} \sum_{i \in \mathcal{M}} \log P(w_i \mid \tilde{X})$$

where $P(w_i \mid \tilde{X})$ is the softmax probability assigned to the true token $w_i$ at masked position $i$, and $\tilde{X}$ is the corrupted input sequence.

Afterwards, the [CLS] token representing the full sequence is passed into a MLP for classification

## Paper

*IEEE Conference on Artificial Intelligence (CAI), 2024: pp. 899–904*

<iframe
  src="Maritime-Context_Text_Identification_for_Connecting_Artificial_Intelligence_AI_Models.pdf"
  width="100%"
  height="900px"
  style="border: none; border-radius: 4px;"
  title="Maritime-Context Text Identification for Connecting AI Models"
>
  <a href="Maritime-Context_Text_Identification_for_Connecting_Artificial_Intelligence_AI_Models.pdf">Download PDF</a>
</iframe>

## Citation

```bibtex
@inproceedings{zhang2024maritime,
  title={Maritime-context text identification for connecting artificial intelligence (ai) models},
  author={Zhang, Xiaocai and Lim, Hur and Fu, Xiuju and Wang, Ke and Xiao, Zhe and Qin, Zheng},
  booktitle={2024 IEEE Conference on Artificial Intelligence (CAI)},
  pages={899--904},
  year={2024},
  organization={IEEE}
}
```
