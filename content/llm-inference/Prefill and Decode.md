---
title: Prefill and Decode
---

## Overview

How I reason about LLM inference

## Prefill & Decode

Autoregressive inference runs in two phases

### Prefill
- Process the entire input prompt, consisting of system prompt + user msg in a **single forward pass**.
- Builds the **[[Inference Optimizations#Caching|KV cache]]** for every prompt token (kv stored per layer, per head).
- **Compute-bound:** one big matmul over `seq_len` tokens which results in high GPU utilization.
- Produces the logits for the *first* generated token.
- Latency metric: **TTFT** (time to first token).

### Decode
- Generate one token at a time, each conditioned on all previous tokens.
- Each step only does a forward pass for the **single new token**: reuses the KV cache.
- **Memory-bandwidth-bound:** tiny matmul (1 token) but must read the whole KV cache + weights each step → GPU underutilized. This is because we need to read weights from the HBM
- Latency metric: **TPS** (tokens per second).

### Difference
- KV cache can be thought as the link where prefill fills it, decode reads and appends to it.
- Prefill is executed essentially once (1 pass), while decode has output_length - 1 passes, which is why decode is memory bottlenecked

## Greedy & Beam Search

## Sampling

### Temperature
Temperature controls the diversity of text generated, a low temperature results in more deterministic and repetitive texts, while higher results in more diversity. The token distribution is more uniform at higher temperatures.

![[temperature-softmax.png]]
*Low temperature sharpens the softmax into a peaked distribution; high temperature flattens it toward uniform. ([source](https://nipunbatra.github.io/blog/posts/2025-07-09-temperature-softmax.html))*

### Top-k
Top `k` limits to `k` vocab for the LLM to decide, and discards everything else.  

1. Get logits over the full vocab → softmax to probabilities.
2. Keep the top `k` tokens by probability; set the other tokens' probability to 0.
3. Renormalize the remaining `k` so they sum to 1.
4. Sample from that limited distribution.

### Top-p
Top `p` restricts the selection of words that has a combination of probabilities that sums up to a value of `p` 

1. Softmax over the vocab to get probabilities
2. Sort it
3. Gather all tokens where cumulative prob less than p
4. Renormalize to 1 and sample from that new distribution

## Contrastive Decoding (CD)

To address repetitive tokens, CD proposes a method that returns the difference of likelihood under a large LLM and a small LLM. CD is built on the idea where drawbacks of larger models are even more common in smaller models.

### Code snippet:

````python
import transformers as tr
import torch

amateur_path = "Qwen/Qwen2.5-Coder-0.5B-Instruct"
expert_path = "Qwen/Qwen2.5-Coder-1.5B-Instruct"

tokenizer = tr.AutoTokenizer.from_pretrained(amateur_path)

user_message = """Give a very very brief docstring for the following function:\n```\nfunction updateEloScores(
  scores,
  results,
  kFactor = 4,
) {
  for (const result of results) {
    const { first, second, outcome } = result;
    const firstScore = scores[first] ?? 1000;
    const secondScore = scores[second] ?? 1000;

    const expectedScoreFirst = 1 / (1 + Math.pow(10, (secondScore - firstScore) / 400));
    const expectedScoreSecond = 1 / (1 + Math.pow(10, (firstScore - secondScore) / 400));
    let sa = 0.5;
    if (outcome === 1) {
      sa = 1;
    } else if (outcome === -1) {
      sa = 0;
    }
    scores[first] = firstScore + kFactor * (sa - expectedScoreFirst);
    scores[second] = secondScore + kFactor * (1 - sa - expectedScoreSecond);
  }
  return scores;
}\n```"""

# turn list of messages into formatted prompt the model was trained on
prompt = tokenizer.apply_chat_template(
    [
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": user_message},
    ],
    add_generation_prompt=True,
    tokenize=False,
)

device = "cuda" if torch.cuda.is_available() else "mps" if torch.backends.mps.is_available() else "cpu"
dtype = torch.bfloat16 if device == "cuda" else torch.float32

# get the model directly
amateur = tr.AutoModelForCausalLM.from_pretrained(amateur_path, torch_dtype=dtype).to(device).eval()
expert = tr.AutoModelForCausalLM.from_pretrained(expert_path, torch_dtype=dtype).to(device).eval()


@torch.no_grad()
def contrastive_generation(amateur, expert, prompt, max_tokens, alpha=0.1) -> str:
    # apply the tokenizer to get the indices within the model vocab
    input_ids = tokenizer(prompt, return_tensors="pt").input_ids.to(device)
    out = []
    for _ in range(max_tokens):
        # forward pass on both expert and amateur, softmaxxing the logits of the last token of sequence to get probabilities, then applying log
        # [1, seq_len, vocab_size] to [1, vocab size], last sequence position
        # softmax applied to vocab (-1 last dim)
        expert_logp = torch.log_softmax(expert(input_ids).logits[:, -1, :], dim=-1)
        amateur_logp = torch.log_softmax(amateur(input_ids).logits[:, -1, :], dim=-1)

        # plausibility: keep only tokens the expert rates within alpha of its top token
        cutoff = expert_logp.max(dim=-1, keepdim=True).values + torch.log(torch.tensor(alpha))
        # how much expert prefers each token vs amateur. log - log basically is log(exp/amateur)
        scores = expert_logp - amateur_logp
        scores[expert_logp < cutoff] = -float("inf")
        # [1, vocab size] -> get token with highest log ratio
        next_id = scores.argmax(dim=-1, keepdim=True)
        if next_id.item() == tokenizer.eos_token_id:
            break
        # store to decode later
        out.append(next_id.item())
        # append latest token to previous
        input_ids = torch.cat([input_ids, next_id], dim=-1)
    return tokenizer.decode(out) # map indices of vocab to actual vocab


print(contrastive_generation(amateur, expert, prompt, 200))
````

Output:
```txt
This function `updateEloScores` takes three arguments:
- `scores`: a dictionary or similar mapping type that maps player identifiers (typically user IDs or usernames) to Elo scores.
- `results`: a list or iterable of result tuples. Each result tuple should contain three elements:
  - `first`: the identifier of the first player in the match.
  - `second`: the identifier of the second player in the match.
  - `outcome`: an integer indicating the result of the match:
    - `1`: first player won
    - `-1`: second player won
    - `0`: a draw
- `kFactor` (optional, default: `4`): the K-factor parameter for updating Elo scores, which controls how quickly players' ratings change based on match results.
The function calculates the Elo score update for each player based on the match result and then applies these updates to the `scores` mapping. It first converts missing player IDs to default Elo score
```

**My intuition:** The repetition tokens have high p_expert and high p_amateur, so the subtraction cancels them out. What survives is what the expert prefers beyond the amateur's generic tendencies. It helps to encourage more quality by maximising the difference between expert and amateur

Some drawbacks:
1. 2 models to deploy, incurring more compute and VRAM
2. 2 sets of KV cache for both models
3. Extra forward pass per step


## Speculative Decoding

Speculative decoding improves token per second through the use of 2 models, the draft model and target model (the one we want to accelerate).

1. We have a draft model, usually a smaller one to generate `N` draft tokens autoregressively
2. We append these draft tokens to the prompt and run one forward pass on the target model to score all these draft tokens. 
3. We run rejection sampling on these draft tokens.
4. So total tokens we get is up to `N+1` best case, if none are rejected

### Code snippet:

````python
import transformers as tr
import torch

draft_path = "Qwen/Qwen2.5-Coder-0.5B-Instruct"   # small + fast, proposes tokens
target_path = "Qwen/Qwen2.5-Coder-1.5B-Instruct"  # the model we want to accelerate

tokenizer = tr.AutoTokenizer.from_pretrained(target_path)

device = "cuda" if torch.cuda.is_available() else "mps" if torch.backends.mps.is_available() else "cpu"
dtype = torch.bfloat16 if device == "cuda" else torch.float32

draft = tr.AutoModelForCausalLM.from_pretrained(draft_path, torch_dtype=dtype).to(device).eval()
target = tr.AutoModelForCausalLM.from_pretrained(target_path, torch_dtype=dtype).to(device).eval()


@torch.no_grad()
def speculative_generation(draft, target, prompt, max_tokens, gamma=4, temperature=1.0) -> str:
    input_ids = tokenizer(prompt, return_tensors="pt").input_ids.to(device)
    prompt_len = input_ids.shape[1] # [batch, seq_len], kept only to slice the output at the end

    generated = 0
    while generated < max_tokens:
        # 1. DRAFT: autoregressively propose gamma tokens, remembering each token's draft prob q
        seq = input_ids
        q_probs, proposed = [], []
        for _ in range(gamma):
            logits = draft(seq).logits[:, -1, :].squeeze(0)          # [vocab]
            probs = torch.softmax(logits / temperature, dim=-1)
            tok = torch.multinomial(probs, num_samples=1)            # sample from draft
            q_probs.append(probs)
            proposed.append(tok.item())
            seq = torch.cat([seq, tok.view(1, 1)], dim=-1)

        # 2. TARGET: ONE forward pass scores all gamma proposals + 1 bonus position.
        #    the last gamma+1 logits predict the gamma drafted tokens plus one extra token.
        target_logits = target(seq).logits[:, -(gamma + 1):, :].squeeze(0)   # [gamma+1, vocab]
        p_probs = torch.softmax(target_logits / temperature, dim=-1)         # [gamma+1, vocab]

        # 3. VERIFY: accept a prefix via rejection sampling, stop at the FIRST reject
        n_accept = 0
        for i in range(gamma):
            tok = proposed[i]
            p, q = p_probs[i, tok], q_probs[i][tok]   # target vs draft prob of the drafted token
            r = torch.rand(1, device=device)
            if r < torch.clamp(p / q, max=1.0):       # accept w.p. min(1, p/q)
                n_accept += 1
            else:
                break                                 # first rejection: cut here

        # commit accepted prefix
        for tok in proposed[:n_accept]:
            input_ids = torch.cat([input_ids, torch.tensor([[tok]], device=device)], dim=-1)

        # 4. CORRECTION token
        if n_accept == gamma:
            next_probs = p_probs[gamma]                                  # all accepted -> free bonus token
        else:
            adjusted = torch.clamp(p_probs[n_accept] - q_probs[n_accept], min=0.0)  # resample from (p - q)+
            next_probs = adjusted / adjusted.sum()
        next_tok = torch.multinomial(next_probs, num_samples=1)
        input_ids = torch.cat([input_ids, next_tok.view(1, 1)], dim=-1)
        generated += n_accept + 1   # accepted prefix + correction token

        if next_tok.item() == tokenizer.eos_token_id:
            break

    return tokenizer.decode(input_ids[0, prompt_len:])   # decode only the generated tokens
````

Some drawbacks:
1. 2 models to deploy, extra VRAM (draft weights + its KV cache)
2. Speedup depends on acceptance rate, a poor draft is more negative
3. Draft runs `gamma` sequential forward passes per round, this latency is not free


## Speculative Speculative Decoding
Will write another day

## References

1. https://arxiv.org/pdf/2210.15097
2. Inference Engineering book by Baseten
3. https://arxiv.org/abs/2211.17192