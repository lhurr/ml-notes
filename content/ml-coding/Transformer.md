---
title: Transformer from Scratch
tags:
  - ml
  - coding
  - transformer
---

## Goal

Naive implementation of Transformer end to end, from raw text to generated tokens.

## Architecture

```text
Text
  |
  v
Tokenizer
  |
  v
Token IDs
  |
  v
Token + positional embeddings
  |
  v
Transformer blocks x N
  |-- Multi-head self-attention
  |-- Residual connection + normalization
  |-- Feed-forward network
  `-- Residual connection + normalization
  |
  v
Linear output projection
  |
  v
Logits over vocabulary
  |
  v
Loss or generated token
```

## Components

1. Tokenizer
2. Token embeddings
3. Positional embeddings
4. Multi-head self-attention
5. Feed-forward network
6. Residual connections and normalization
7. Linear output projection
8. Autoregressive generation

### Tokenizer

```python
chars = sorted(set(text))
stoi = {c: i for i, c in enumerate(chars)}
itos = {i: c for c, i in stoi.items()}
def encode(s):
  result = []
  for c in s:
    result.append(stoi[c])
  return result
def decode(ids):
  out = []
  for id in ids:
    s = itos[id]
    out.append(s)
  return ''.join(out)
vocab_size = len(chars)
```

### Token Embeddings
```python
class TokenEmbedding(nn.Module):
    def __init__(self, vocab_size, d_model):
        super().__init__()
        self.emb = nn.Embedding(vocab_size, d_model)
        self.d_model = d_model

    def forward(self, ids):          # ids: (batch, seq_len)
        return self.emb(ids)         # maps sequence into a embedding table (batch, seq_len, d_model)
```


### Positional embeddings

```python
import torch
import torch.nn as nn

class PositionalEmbedding(nn.Module):
    def __init__(self, max_len, d_model):
        super().__init__()
        self.embedding = nn.Embedding(max_len, d_model)

    def forward(self, x):
        # x: (batch, seq_len, d_model)
        B, T, D = x.shape

        positions = torch.arange(T, device=x.device)      # (seq_len,)
        positions = self.embedding(positions)             # (seq_len, d_dim)

        return x + positions
```


### Attention block

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


```python
import torch
import torch.nn as nn
import torch.nn.functional as F

class TransformerBlock(nn.Module):
    def __init__(self, d_model=128):
        super().__init__()
        self.attn = Attention(d_model)
        self.ffn = nn.Sequential(
            nn.Linear(d_model, 4 * d_model),
            nn.GELU(),
            nn.Linear(4 * d_model, d_model)
        )
        self.ln = nn.LayerNorm(d_model)
        self.ln2 = nn.LayerNorm(d_model)

    def forward(self, x):
        B, T, D = x.shape

        out = self.attn(x)
        out = out + x
        x = self.ln(out) # (batch_size, seq_len, d_model)

        ffn_out = self.ffn(x) # (batch_size, seq_len, d_model)
        out = self.ln2(ffn_out + x) # (batch_size, seq_len, d_model)

        return out
```

### Linear projection + transformer blocks

```py
import torch
import torch.nn as nn
import torch.nn.functional as F

class Transformer(nn.Module):
    def __init__(self, vocab_size, max_seq_len, d_model=128, n_layers = 8):
        super().__init__()
        self.blocks = nn.ModuleList(
    [TransformerBlock(d_model) for _ in range(n_layers)]
)
        self.n_layers = n_layers
        self.lm_head = nn.Linear(d_model, vocab_size)
        self.te = TokenEmbedding(vocab_size, 128)
        self.pe = PositionalEmbedding(max_seq_len, 128)

    def forward(self, x):
        B, T = x.shape
        token = self.te(x)
        x = self.pe(token)

        for b in self.blocks:
          x = b(x)
        
        out = self.lm_head(x) # (batch_size, seq_len, dim) -> (batch_size, seq_len, vocab_size) 

        return out
```

For autoregressive generation, we start off with a prompt:
1. "The car" gets tokenized and converted into `input_ids`, e.g [100, 170]
2. Run the model, model(`input_ids`), output will be (1, 2, vocab_size)
3. Get the next token's logits, model(`input_ids`).logits[:, -1, :], size of (1, vocab)
4. Apply softmax to get probabilities and apply decoding to get the token
5. Afterwards, append it and repeat the process