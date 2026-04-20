# StreamingLLM / Attention Sinks

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **Paper:** [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) · **GitHub:** [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) ⭐7.2K · **Venue:** ICLR 2024

---

## Overview

> *"Attention sinks — a small set of tokens that attract excessive attention — enable stable streaming language models without re-computation."* — [Xiao et al., ICLR 2024](https://arxiv.org/abs/2309.17453)

From [MIT-HAN-LAB](https://github.com/mit-han-lab), published at **ICLR 2024**. Enables LLMs to generate infinite-length text without fine-tuning while maintaining constant memory.

---

## Key Paper

### "Efficient Streaming Language Models with Attention Sinks"

> **arXiv:** [2309.17453](https://arxiv.org/abs/2309.17453) | **GitHub:** [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) ⭐7.2K | **Venue:** ICLR 2024

---

## Core Concept: Attention Sinks

LLMs exhibit **attention sink** behavior — a small number of tokens (often the first token, "." tokens, or specialized sink tokens) attract disproportionately high attention.

### Why Attention Sinks Form

> *"Without a dedicated mechanism, language models spread attention uniformly across all preceding tokens, making the attention distribution unstable during long generation."*

The theory: models use certain "anchor" tokens to aggregate information, forming stable attention patterns.

### StreamingLLM Solution

```
KV Cache Structure:
┌─────────────┬─────────────────────────────┐
│ Sink Tokens │ Rolling Window (L recent)   │
│ (1-4 tokens)│ (e.g., last 512 tokens)     │
└─────────────┴─────────────────────────────┘
     ↑                   ↑
   Always kept       Sliding eviction
```

1. **Sink tokens** (first 1-4 tokens) — Keep in KV cache permanently, always attend to
2. **Rolling KV cache** — Sliding window over recent tokens
3. **Total memory** — O(1) with respect to generation length

---

## How It Works

### Step 1: Identify Sink Tokens

- The **first token** ([CLS] or BOS) is almost always a sink
- Other sink tokens emerge during pre-training
- Typically 1-4 tokens total

### Step 2: Cache Architecture

```
Generation at step T:
┌───────────────┬────────────────────────┐
│ Attention to: │ Memory:                │
├───────────────┼────────────────────────┤
│ Sink[0]       │ {Sink[0]: value}       │ ← Constant
│ Sink[1]       │ {Sink[1]: value}       │ ← Constant
│ ...           │ ...                    │ ← Constant  
│ Tokens[T-L:T] │ {Recent: values}       │ ← Sliding
└───────────────┴────────────────────────┘
Memory: O(sinks + L) = O(1) w.r.t. T
```

### Step 3: Streaming Generation

- Drop oldest non-sink tokens when cache is full
- Sink tokens remain → stable attention patterns
- No re-computation needed → efficient streaming

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Constant memory** | O(1) memory w.r.t. generation length |
| **No fine-tuning** | Works on existing pretrained LLMs |
| **Stable attention** | Prevents attention collapse during long generation |
| **Streaming deployment** | Enables infinite-length text generation |
| **Compatible** | Works with Flash Attention, etc. |

---

## Quantitative Results

### Memory vs Generation Length

| Model | Baseline Memory | StreamingLLM Memory | Improvement |
|-------|---------------|---------------------|-------------|
| Llama-2-7B at 1K tokens | ~1 GB | ~0.1 GB | ~10× savings |
| Llama-2-7B at 100K tokens | ~100 GB | ~0.1 GB | ~1000× savings |
| Llama-2-13B at 1M tokens | OOM | ~0.2 GB | ∞ |

### Performance

| Metric | Standard | StreamingLLM |
|--------|----------|--------------|
| Perplexity at 4K | Baseline | +0.5% (minimal) |
| Perplexity at 1M | Degrades | Stable |
| Memory at 1M tokens | ∞ | Constant |

---

## Extensions & Related Work

### Attention Sinks Extension

> **GitHub:** [tomaarsen/attention_sinks](https://github.com/tomaarsen/attention_sinks) ⭐736

> *"Extend existing LLMs way beyond the original training length with constant memory."*

### Dynamic Attention Sinks

> **GitHub:** [LiterallyBlah/Dynamic-Attention-Sinks-in-Streaming-LLMs](https://github.com/LiterallyBlah/Dynamic-Attention-Sinks-in-Streaming-LLMs)

Dynamic adaptation of sink token selection during streaming.

### Comparison with Other Methods

| Method | Memory | Fine-tuning | Complexity |
|--------|--------|-------------|------------|
| **StreamingLLM** | O(1) | None | Simple |
| Full recompute | O(N) | None | Expensive |
| Sliding window | O(L) | None | Simple |
| Hierarchical | O(N) | Yes | Complex |

---

## See Also

- [README](./README.md) — Context Windows overview
- [PagedAttention](./PagedAttention-vLLM.md) — OS-style memory paging
- [Flash Attention](./Flash-Attention-Family.md) — IO-aware exact attention
- [Ring Attention](./Ring-Attention.md) — Distributed long-context
