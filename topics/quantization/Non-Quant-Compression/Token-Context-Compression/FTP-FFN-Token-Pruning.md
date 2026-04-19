# FTP — FFN Token Pruning

> **Key Papers:** [FTP ICLR 2025](https://openreview.net/forum?id=fL8Zp8o6RL) · [FTP Token Routing arXiv:2412.11494](https://arxiv.org/abs/2412.11494)  
> **Related:** [LazyLLM (Apple) ICML 2024](https://arxiv.org/html/2407.14057v1) · [PyramidInfer ACL 2024](https://aclanthology.org/2024.findings-acl.195.pdf)  
> **GitHub:** [zbwxp/Dynamic-Token-Pruning](https://github.com/zbwxp/Dynamic-Token-Pruning)

---

## Overview

> *"The FFN module accounts for over 60% of the inference time."* — FTP ICLR 2025

FTP (FFN Token Pruning) addresses a critical gap in token compression research — while most methods focus on **attention/KV-cache optimization**, the **FFN module consumes 60%+ of inference time**. FTP prunes non-critical tokens *before* FFN computation, reducing FLOPs without losing information through residual connections.

| Aspect | Details |
|--------|---------|
| **Target** | FFN computation (not attention/KV cache) |
| **Speedup Focus** | Prefilling stage (not just decoding) |
| **Key Metric** | FLOPs reduction |
| **Training-free** | Yes (ICLR 2025 paper) |
| **Result** | 1.24–1.39× speedup with subtle accuracy drop |

---

## Why FFN Over Attention?

| Model | FFN % | Attention % | LayerNorm % |
|-------|-------|-------------|-------------|
| Llama3-8B-Instruct | **62.4%** | 32.9% | 4.7% |
| Qwen2-7B-Instruct | **61.3%** | 35.3% | 3.4% |

> *"Critically, these approaches prioritize the attention module, overlooking the significant computations in the Feed-Forward Network (FFN) module."* — FTP Paper

**Attention-based methods** (LazyLLM, FastV) only compress KV cache — they skip attention computation but still run full FFN. FTP addresses the elephant in the room: FFN is where most time is spent.

---

## Key Methods

### 1. FTP — Efficient Prefilling via FFN Token Pruning (ICLR 2025)

> **Paper:** [OpenReview](https://openreview.net/forum?id=fL8Zp8o6RL) · **PDF:** [OpenReview PDF](https://openreview.net/pdf?id=fL8Zp8o6RL)  
> **Authors:** Guo-Hao Xu, Jingzhen Ding, Huping Ding, Zhao Xu, Kaifu Zhang

**Core Method:**

1. **Token Importance Scoring** — Uses attention scores from last N queries across all heads
2. **Dynamic Pruning per Layer** — Number of pruned tokens varies per layer based on attention distribution
3. **First F layers fully preserved** (η=1.0) — shallow layers are more sensitive to pruning
4. **Information preserved via residual connection** — pruned tokens still flow through residual to preserve their contribution

**FLOPs Reduction:**
```
Original: 6LCI  (L=seq_len, C=hidden, I=intermediate)
After FTP: 6L_I·CI  (L_I = important tokens)
```

**Results:**

| Model | Avg Speedup | Performance Impact |
|-------|-------------|-------------------|
| Qwen2-7B-Instruct | **1.24×** | 1.30% performance drop |
| Qwen1.5-32B-Chat | **1.39×** | Subtle accuracy drop |
| Qwen2-72B-Instruct | **1.33×** | Subtle accuracy drop |

**LongBench Results (Qwen2-7B-Instruct):**

| Task | Baseline | FTP | Speedup |
|------|----------|-----|---------|
| Single-Doc QA | 39.00 | 38.75 | 1.22× |
| Multi-Doc QA | 37.48 | 35.21 | 1.26× |
| Summarization | 26.70 | 25.01 | 1.23× |
| Few-shot Learning | 70.17 | 69.11 | 1.25× |
| Synthetic | 37.50 | 36.75 | 1.30× |
| Code Completion | 58.43 | 56.74 | 1.22× |

**vs. PyramidInfer:**
> *"PyramidInfer's official implementation fails to accelerate prefilling and causes OOM on Qwen2 (32k context)."*

---

### 2. FTP — Token Routing via Learnable Router (arXiv 2412.11494)

> **Paper:** [arXiv:2412.11494](https://arxiv.org/abs/2412.11494)

**Key Finding:**
> *"89.94% and 93.16% of tokens in LLaMA2-7B and Qwen1.5-7B, respectively, exhibit a similarity score higher than 0.8, suggesting minimal changes and a high potential for pruning."*

**Token Redundancy Patterns:**
| Block Position | LLaMA2-7B | Qwen1.5-7B |
|----------------|-----------|------------|
| First/last 3 blocks | 49.74% | 35.42% |
| Middle blocks | 99.10% | 99.76% |

**Method: Three-Step Pipeline**
1. **Sparsity Scheduler Search (Static Router)** — Genetic algorithm finds optimal per-block sparsity ratios
2. **Dynamic Router Training** — 2-layer MLP using 4 input factors (position, attention score, rank, sparsity)
3. **Sparsity Scheduler Fine-tuning** — Iterative refinement

**Results:**

| Model | Method | Sparsity | Avg Percentage |
|-------|--------|----------|----------------|
| LLaMA2-7B | Dense | 0% | 100% |
| | BlockPruner | 20.99% | 84.91% |
| | ShortGPT | 27.0% | 80.22% |
| | **FTP** | 22.0% | **99.21%** |

---

## Comparison: Attention-Based vs FFN-Based

| Aspect | Attention-Based (LazyLLM, FastV) | FFN-Based (FTP) |
|--------|----------------------------------|------------------|
| **Target** | KV cache compression | FFN computation reduction |
| **Speedup Focus** | Decoding stage | Prefilling stage |
| **Key Metric** | KV cache size | FLOPs reduction |
| **FFN Optimization** | Not addressed | Accounts for 60%+ of time |
| **Training Required** | Usually no | No (FTP-ICLR) / Yes (FTP-arXiv) |
| **KV Cache** | Pruning/eviction | Full preservation through residual |

> *"The attention scores of input tokens w.r.t. to the first generated token are very sparse, indicating that many tokens in the input prompt are redundant."* — LazyLLM

---

## Related Methods

### LazyLLM (Apple ML Research, ICML 2024)

> **Paper:** [arXiv:2407.14057](https://arxiv.org/html/2407.14057v1) · **Blog:** [Apple ML](https://machinelearning.apple.com/research/dynamic-token-pruning)

**Key Quote:**
> *"The attention scores of input tokens w.r.t. to the first generated token are very sparse, indicating that many tokens in the input prompt are redundant."*

**Method:** Dynamically selects different token subsets at each generation step; can revive previously pruned tokens via Aux Cache.

**Results:** 2.34× TTFT speedup on multi-doc QA with ≤1% accuracy loss.

### PyramidInfer (ACL 2024 Findings)

> **Paper:** [ACL Anthology](https://aclanthology.org/2024.findings-acl.195.pdf)

**Focus:** KV cache compression based on layer uncertainty.

### CAPA — Contribution-Aware Pruning + FFN Approximation

> **Paper:** [arXiv:2602.00247](https://arxiv.org/abs/2602.00247)

Dual-strategy: token pruning + FFN approximation for vision-language models.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **FTP (FFN Pruning)** | 2025 | ICLR 2025 | [OpenReview](https://openreview.net/forum?id=fL8Zp8o6RL) | FFN token pruning for prefilling speedup |
| **FTP (Token Routing)** | 2024 | — | [arXiv](https://arxiv.org/abs/2412.11494) | Learnable router, 99%+ performance at 22% sparsity |
| **LazyLLM** | 2024 | ICML 2024 | [arXiv](https://arxiv.org/html/2407.14057v1) | Attention-based dynamic token pruning |
| **PyramidInfer** | 2024 | ACL 2024 | [ACL](https://aclanthology.org/2024.findings-acl.195.pdf) | KV cache via layer uncertainty |
| **CAPA** | 2026 | — | [arXiv](https://arxiv.org/abs/2602.00247) | Token pruning + FFN approximation for VLMs |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [zbwxp/Dynamic-Token-Pruning](https://github.com/zbwxp/Dynamic-Token-Pruning) | Dynamic Token Pruning (DToP) |
| [Theia-4869/FasterVLM](https://github.com/Theia-4869/FasterVLM) | FastV — Visual Token Pruning |
| [liyunqianggyn/Awesome-LLMs-Pruning](https://github.com/liyunqianggyn/Awesome-LLMs-Pruning) | Comprehensive LLM pruning collection |
| [naho/TokenSkipping](https://github.com/naho/TokenSkipping) | Token skipping references |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Addresses FFN as dominant cost (60%+) — where attention-focused methods don't reach |
| **Key innovation** | Prune tokens before FFN, preserve via residual; dynamic per-layer pruning ratios |
| **Speedup** | 1.24–1.39× prefilling speedup; complements attention-based methods |
| **Best for** | Long-context prefilling where FFN dominates; works alongside KV cache eviction |
| **Key insight** | First/last layers are sensitive — preserve them; middle layers have 99%+ redundancy |

---

*Last updated: 2026-04-19*
