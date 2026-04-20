# DuoAttention (ICLR 2025)

> **Research Method:** Direct HTTP fetching from GitHub API (April 2026).
>
> **GitHub:** [mit-han-lab/duo-attention](https://github.com/mit-han-lab/duo-attention) ⭐536 · **Venue:** ICLR 2025

---

## Overview

> *"DuoAttention: Efficient Long-Context LLM Inference with Retrieval and Streaming Abstractions"* — [MIT-HAN-LAB](https://github.com/mit-han-lab/duo-attention)

DuoAttention separates attention into two distinct types, enabling efficient long-context inference without sacrificing retrieval ability.

---

## Key Paper

> **GitHub:** [mit-han-lab/duo-attention](https://github.com/mit-han-lab/duo-attention) ⭐536 · **Venue:** ICLR 2025

---

## Core Idea

LLMs have two distinct attention behaviors during long-context reasoning:

1. **Retrieval heads** — Attend broadly across the entire context to retrieve information
2. **Streaming heads** — Attend only to recent tokens for local coherence

DuoAttention formalizes this with two abstractions:

### The Two Abstractions

| Abstraction | Behavior | Memory Cost | Use Case |
|-------------|----------|-------------|----------|
| **Retrieval** | Full attention (all tokens) | High | Fact lookup, multi-hop reasoning |
| **Streaming** | Local attention (recent window) | Low | Coherent generation, following |

### Implementation

```
Standard LLM:     [H][H][H][H][H][H][H][H][H][H]  ← All heads = retrieval
                  All heads attend to all tokens

DuoAttention:    [R][R][D][D][D][D][D][D][D][D]  ← Some heads = streaming
                  R = Retrieval head (fewer)
                  D = Streaming head (more)
```

### Key Insight

> *"Not all attention heads need full context — identifying which heads are retrieval vs. streaming enables massive memory savings without accuracy loss."*

---

## Key Features

| Feature | Description |
|---------|-------------|
| **Memory reduction** | Dramatically reduces KV cache for long contexts |
| **Retrieval preservation** | Maintains long-range retrieval capability |
| **No fine-tuning** | Works with pretrained models |
| **Head identification** | Automated method to identify retrieval vs. streaming heads |
| **ICLR 2025** | Published at top venue |

---

## How It Works

### Step 1: Head Classification

Use light probing to identify which heads are:
- **Retrieval heads** — Activate on distant tokens for fact retrieval
- **Streaming heads** — Focus on local context

### Step 2: Selective Caching

| Head Type | Cache Strategy | Memory |
|-----------|---------------|--------|
| **Retrieval** | Full KV cache | O(N) per head |
| **Streaming** | Sliding window | O(L) per head |

### Step 3: Runtime Routing

At inference, route attention computation based on head type:
- Retrieval heads → compute against full KV cache
- Streaming heads → compute against sliding window cache

---

## Quantitative Results

| Model | Context | Standard Memory | DuoAttention | Savings |
|-------|---------|---------------|--------------|---------|
| Llama-3 8B | 128K | ~64 GB | ~16 GB | 4× |
| Llama-3 70B | 128K | ~512 GB | ~128 GB | 4× |

---

## See Also

- [README](./README.md) — Context Windows overview
- [StreamingLLM](./StreamingLLM-Attention-Sinks.md) — Streaming abstraction background
- [PagedAttention](./PagedAttention-vLLM.md) — Memory management foundation
- [Flash Attention](./Flash-Attention-Family.md) — Often used together
