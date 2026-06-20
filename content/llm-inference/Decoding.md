---
title: Decoding
---

## Overview

To be written

## Greedy & Beam Search

## Sampling

### Temperature

### Top-k

### Top-p

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

## References

https://arxiv.org/pdf/2210.15097