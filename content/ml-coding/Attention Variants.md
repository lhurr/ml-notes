---
title: Attention Variants
tags:
  - ml
  - coding
  - attention
---

## Goal

Implement causal, cross, and self attention.

### Self attention

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class Attention(nn.Module):
    def __init__(self, d_model=128):
        super().__init__()
        self.d_model = d_model
        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(d_model, d_model, bias=False)
        self.W_v = nn.Linear(d_model, d_model, bias=False)
        # self.W_o = nn.Linear(d_model, d_model, bias=False)

    def forward(self, x):
        B, T, D = x.shape

        q = self.W_q(x)  # (batch, seq_len, dim)
        k = self.W_k(x)
        v = self.W_v(x)

        # (batch, seq_len, dim) x (batch, dim, seq_len)
        scores = q @ k.transpose(-2, -1)

        scores = scores / (self.d_model ** 0.5) # (batch, seq_len, seq_len)

        attn = F.softmax(scores, dim=-1)           # (batch, seq_len, seq_len)

        out = attn @ v                             # (batch, seq_len, dim)

        return out
```

### Cross attention

Cross attention uses query from input sequence (`q`), but key and value, `k` and `v` from another sequence.

```python
import torch
import torch.nn as nn
import torch.nn.functional as F


class CrossAttention(nn.Module):
    def __init__(self, d_model=128, context_dim=None):
        super().__init__()

        context_dim = context_dim or d_model
        self.d_model = d_model

        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(context_dim, d_model, bias=False)
        self.W_v = nn.Linear(context_dim, d_model, bias=False)

    def forward(self, x, x2, mask=None):

        q = self.W_q(x)        # (batch_size, seq_len, dim)
        k = self.W_k(x2)  # (batch_size, x2_seq_len, dim)
        v = self.W_v(x2)  # (batch_size, x2_seq_len, dim)

        scores = q @ k.transpose(-2, -1)
        # (batch_size, seq_len, x2_seq_len)

        scores = scores / (self.d_model ** 0.5)

        # (batch_size, seq_len, x2_seq_len)
        attn = F.softmax(scores, dim=-1)
        # each query attends over all context tokens

        out = attn @ v
        # (batch_size, seq_len, dim)

        return out
```

### Multihead/casual attention

```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class MultiHeadAttention(nn.Module):
    def __init__(self, d_model=128, n_heads=4):
        super().__init__()
        assert d_model % n_heads == 0
        self.n_heads = n_heads
        self.d_head = d_model // n_heads          # 32

        self.W_q = nn.Linear(d_model, d_model, bias=False)
        self.W_k = nn.Linear(d_model, d_model, bias=False)
        self.W_v = nn.Linear(d_model, d_model, bias=False)
        self.W_o = nn.Linear(d_model, d_model, bias=False)

    def forward(self, x, causal=True):
        B, T, D = x.shape

        q = self.W_q(x)  # (batch, seq_len, dim)
        k = self.W_k(x)
        v = self.W_v(x)

        q = q.view(B, T, self.n_heads, self.d_head).transpose(1, 2)   # (batch, seq_len, n_heads, dim_head) -> (batch, n_heads, seq_len, dim_head)
        k = k.view(B, T, self.n_heads, self.d_head).transpose(1, 2)
        v = v.view(B, T, self.n_heads, self.d_head).transpose(1, 2)

        # (batch, n_heads, seq_len, dim_head) @ (batch, n_heads, dim_head, seq_len)
        scores = q @ k.transpose(-2, -1) # (batch, n_heads, seq_len, seq_len), attention score matrix
        scores = scores / (self.d_head ** 0.5)

        # casual attention, fill triu with -inf
        if causal:
            mask = torch.ones(T, T, dtype=torch.bool, device=x.device).triu(1)
            scores = scores.masked_fill(mask, float('-inf'))

        attn = F.softmax(scores, dim=-1)           # (batch, n_heads, seq_len, seq_len)

        out = attn @ v                             # (batch, n_heads, seq_len, dim_head)

        out = out.transpose(1, 2).contiguous().view(B, T, D)          # (batch, n_heads, seq_len, dim_head) -> (batch, seq_len, dim)
        return self.W_o(out) # (batch, seq_len, dim)
```

The outcome of casual attention is that each output only depends on its prefix. On the other hand, bidrectional models like BERT, T5 rely on masked language modelling as the pre-training objective. 

Casual attention makes past state immutable, and immutable state can be cached. In bidirectional, we append a token and every previous token's K/V is now stale. There is nothing to cache, by construction. Hence casual attention wins during inference.