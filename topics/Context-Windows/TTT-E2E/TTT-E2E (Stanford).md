# TTT-E2E (Stanford)

> **Paper:** [arXiv:2512.23675](https://arxiv.org/abs/2512.23675)  
> **GitHub:** [test-time-training/e2e](https://github.com/test-time-training/e2e) (592 stars, JAX)  
> **License:** CC BY 4.0

---

## Overview

> *"We formulate long-context language modeling as a problem in continual learning rather than architecture design. Under this formulation, we only use a standard architecture -- a Transformer with sliding-window attention. However, our model continues learning at test time via next-token prediction on the given context, compressing the context it reads into its weights."*
> — Tandon et al., arXiv:2512.23675

**TTT-E2E** is End-to-End Test-Time Training for Long Context, developed by researchers at Stanford (Tandon, Sun, McCaleb, Choi, Leskovec, Hashimoto, Guestrin, Wang, and others).

### Core Innovation

Unlike previous TTT methods, TTT-E2E is **end-to-end** in two ways:
1. **Test-time:** Uses next-token prediction as the self-supervised training objective (no auxiliary losses)
2. **Training-time:** Uses meta-learning (MAML-style) to improve the model's initialization for fast test-time adaptation

---

## Method

### How It Works

1. **Standard Architecture:** Uses a Transformer with **sliding-window attention** — not a new architectural design
2. **Test-Time Learning:** At inference, the model learns on the given context via gradient updates on next-token prediction
3. **Meta-Learning Initialization:** Training uses meta-learning so the model starts from an initialization that's easy to adapt at test time
4. **Context Compression:** The context is compressed into the model's weights rather than stored in a KV cache

### Key Properties

| Property | Description |
|----------|-------------|
| **Context Handling** | Compresses context into weights via gradient-based learning |
| **Inference Latency** | **Constant** regardless of context length (like RNNs) |
| **Speed** | **2.7x faster** than full attention at 128K context |
| **Scaling** | Scales with context length the same way as full attention |
| **Architecture** | Standard Transformer with sliding-window attention |

---

## Benchmark Results

> *"For 3B models trained with 164B tokens, our method (TTT-E2E) scales with context length in the same way as Transformer with full attention, while others, such as Mamba 2 and Gated DeltaNet, do not."*

| Context Length | TTT-E2E Speedup vs Full Attention |
|----------------|-----------------------------------|
| 4K | ~1x |
| 32K | ~1.5x |
| 128K | **2.7x** |

---

## Comparison with Other Approaches

| Method | Architecture | KV Cache | Constant Latency | Scales with Context |
|--------|-------------|----------|------------------|---------------------|
| **TTT-E2E** | Transformer + SWA | None | ✅ | ✅ |
| Full Attention | Transformer | Full | ❌ | N/A |
| Mamba 2 | SSM | None | ✅ | ❌ |
| Gated DeltaNet | Linear RNN | None | ✅ | ❌ |

---

## GitHub Repository

| Repo | Stars | Framework | Notes |
|------|-------|-----------|-------|
| [test-time-training/e2e](https://github.com/test-time-training/e2e) | 592 ⭐ | JAX | Official Stanford implementation |
| [release_checkpoints](https://github.com/test-time-training/e2e/tree/release_checkpoints) | — | — | Pre-trained model checkpoints available |

### Quick Start

```bash
# Clone the repo
git clone https://github.com/test-time-training/e2e.git
cd e2e

# Training
python train.py

# Inference / Test-time adaptation
python evaluate.py
```

---

## Authors & Affiliations

| Author | Affiliation |
|--------|-------------|
| Arnuv Tandon | Stanford |
| Karan Dalal | Stanford |
| Xinhao Li | Stanford |
| Yu Sun | Stanford |
| Jed McCaleb | — |
| Yejin Choi | Stanford |
| Jure Leskovec | Stanford |
| Tatsunori Hashimoto | Stanford |
| Carlos Guestrin | Stanford |
| Xiaolong Wang | CMU |
| Sam Buchanan | — |
| Daniel Koceja | — |
| Marcel Rød | — |
| Sanmi Koyejo | — |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem Solved** | Long-context inference with quadratic attention cost |
| **Key Innovation** | TTT + meta-learning for RNN-like constant latency with Transformer quality |
| **Best For** | Long-context tasks where KV cache memory is bottleneck |
| **Limitations** | Additional compute per token for gradient updates; recent work (late 2025) |

---

*Papers: [arXiv:2512.23675](https://arxiv.org/abs/2512.23675) · [arXiv:2601.00894](https://arxiv.org/abs/2601.00894)*  
*GitHub: [test-time-training/e2e](https://github.com/test-time-training/e2e)*
