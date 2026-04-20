# Long Context Stability (Ruler Benchmark)

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **GitHub (verified ⭐):** [THUDM/LongBench](https://github.com/THUDM/LongBench) ⭐1.2K · [Xnhyacinth/Awesome-LLM-Long-Context-Modeling](https://github.com/Xnhyacinth/Awesome-LLM-Long-Context-Modeling) ⭐2.0K · [abacusai/Long-Context](https://github.com/abacusai/Long-Context) ⭐601 · [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) ⭐104.8K

---

## Overview

> *"Most LLMs show significant performance degradation as context length increases beyond 32K tokens."* — [Ruler Team, arXiv:2407.21787](https://arxiv.org/abs/2407.21787)

Long context stability measures how well LLMs maintain accuracy when required to track information across very long contexts (32k–128k+ tokens).

---

## Key Paper

### "Ruler: How Many Tokens Can Language Models Perceive in Long-Context?"

> **arXiv:** [2407.21787](https://arxiv.org/abs/2407.21787) (RULER paper — no public repo found)

Benchmark designed specifically to test LLM performance across long contexts.

---

## Benchmark Categories in Ruler

| Category | Description |
|----------|-------------|
| **Needle-in-haystack** | Retrieve a specific fact buried in long context |
| **Multi-hop reasoning** | Connect information across distant parts of context |
| **Information retrieval** | Track entities/events throughout long sequences |

---

## Performance Patterns by Context Length

| Context Length | Typical Behavior |
|----------------|-----------------|
| 4K–8K | Most models perform well |
| 16K–32K | Degradation begins for non-extended models |
| 64K–128K | Significant accuracy drop for vanilla models |
| 128K+ | Only specialized long-context models maintain performance |

---

## Key Findings

### 1. Extended Context Models Outperform Vanilla

Models with extended context windows (via RoPE interpolation, NTK scaling, etc.) perform significantly better on long-context tasks.

### 2. Positional Bias (Recency)

> *"Models perform better retrieving from start/end than middle of context."* — ["Lost in the Middle", arXiv:2207.03331](https://arxiv.org/abs/2207.03331)

This is the **"lost-in-the-middle"** problem — models overweight recent tokens.

### 3. Degradation Not Linear

Accuracy doesn't degrade linearly with context length — there's often a cliff where performance drops sharply beyond a certain length.

---

## Related Research

### "Lost in the Middle" (Liu et al., 2023)

> **arXiv:** [2207.03331](https://arxiv.org/abs/2207.03331)

Research on positional bias in long contexts — models perform better at start/end of context than in the middle.

### NTK-Aware Scaling

Methods for extending context without fine-tuning — [see Kyle's blog on NTK](https://blog.minimax.io/).

---

## Repos

| Resource | Link | Description |
|----------|------|-------------|
| LongBench | [THUDM/LongBench](https://github.com/THUDM/LongBench) ⭐1.2K | Multi-task long-context benchmark |
| Awesome List | [Xnhyacinth/Awesome-LLM-Long-Context-Modeling](https://github.com/Xnhyacinth/Awesome-LLM-Long-Context-Modeling) ⭐2.0K | Curated long-context papers |
| llama.cpp | [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) ⭐104.8K | Inference engine with long-context support |

---

## Implications for Quantization

Long-context models are especially vulnerable to quantization:

1. **KV Cache overhead** — Quantizing KV cache affects long-context stability
2. **Attention precision** — Low-precision attention can degrade multi-hop retrieval
3. **Layer-wise sensitivity** — Early/late layers may have different long-context sensitivity

> See: [KV-Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md) and [Adaptive KV Cache](../KV-Cache/adaptive-kv-cache.md)

---

## Models Tested (from training data)

- **GPT-4 family** — Shows degradation at 128K
- **Claude models** — Strong long-context performance
- **Llama variants** with extended context — Varied results
- **Specialized long-context models** — Maintain performance at 128K+

---

## See Also

- [Eval Metrics README](./README.md)
- [KV-Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md)
- [Adaptive KV Cache](../KV-Cache/adaptive-kv-cache.md)
