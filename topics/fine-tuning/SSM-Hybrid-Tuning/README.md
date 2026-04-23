# SSM-Hybrid-Tuning

## 📌 Overview

Fine-tuning strategies and architectures for hybrid State Space Model (SSM) systems. Covers the Mamba SSM architecture lineage, hybrid Transformer-Mamba models (Jamba), and state optimization techniques (compression, expansion, efficient passing).

> *"PEFT works better for Mamba than Transformers; proposes HybridPEFT combining 8 methods."*
> — [ICLR 2025 MambaPEFT](https://openreview.net/forum?id=LtzmeSMBTW)

---

## Topics

| Topic | Description |
|-------|-------------|
| [Mamba-3](./Mamba-3.md) | Mamba-3 SSM architecture: exponential-trapezoidal discretization, complex-valued states, MIMO |
| [Jamba](./Jamba.md) | AI21's hybrid Transformer-Mamba-MoE architecture at 52B-398B scale |
| [State-Optimization](./State-Optimization.md) | SSM state compression, expansion, and efficient passing methods |

---

## Hybrid Architecture Comparison

| Architecture | SSM Component | Attention Component | MoE | KV Cache | Throughput |
|-------------|--------------|--------------------|-----|----------|------------|
| **Jamba** | 7× Mamba layers | 1× Attention layer | 16 experts, top-2 | 8× smaller than vanilla | 3× faster than Mixtral at 128K |
| **SAmBA/Samba** | Mamba + SWA | — | — | — | 3.73× faster at 128K |
| **Hymba** | Parallel SSM | Parallel Attention | — | ~20× smaller | 3× higher |
| **Nemotron-3** | Mamba | Transformer | Yes | — | — |

---

## Key Findings

### Why Hybrid?

> *"Pure Mamba struggles with datasets requiring format adherence (IMDB, QuAC, NarrativeQA). Hybrid Jamba outperforms both pure attention and pure Mamba models."*
> — [arXiv:2403.19887](https://arxiv.org/abs/2403.19887)

### Retrieval Depends on Attention

> *"SSM layers do not contribute to retrieval — even fuzzy memory observed in pure SSMs is absent. Retrieval depends exclusively on self-attention layers."*
> — [arXiv:2403.19887](https://arxiv.org/abs/2403.19887)

### PEFT for SSMs

> *"LoRA on linear projections outperforms LoRA on SSM modules."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

> *"Input-injection methods (prefix-tuning) FAIL on SSM-based models — expressivity reduces to tuning only initial hidden state."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

### State Compression

> *"SSMs trained large and shrunk during training outperform models trained directly at smaller dimensions."*
> — [ICLR 2026 CompreSSM](https://openreview.net/forum?id=LtzmeSMBTW)

---

## Mamba Architecture Lineage

| Version | Key Innovation | State | Primary Benefit |
|---------|---------------|-------|----------------|
| **Mamba-1** | Selective state passing | d_state=16 | O(1) inference |
| **Mamba-2** | State Space Duality (SSD) | Larger | 2-3× faster training |
| **Mamba-3** | Exp-trapezoidal discret., complex-valued, MIMO | Optimized | Fastest inference |

> *"Mamba-2 connects SSMs to structured matrices via State Space Duality (SSD). Preserves efficient FLOP counts as SSMs while dramatically speeding up training using matmuls."*
> — [Tri Dao Blog](https://tridao.me/blog/2024/mamba2-part3-algorithm/)

---

## Fine-Tuning Summary

| Method | Works for SSM? | Notes |
|--------|---------------|-------|
| LoRA on linear projections | ✅ Yes | q_proj, k_proj, v_proj, o_proj |
| LoRA on SSM modules (A/B) | ⚠️ Less | Not recommended |
| Prefix/Prompt Tuning | ❌ No | "Maximum expressiveness = tuning initial state h₀ only" |
| State-offset Tuning | ✅ Yes | ACL 2025, consistent timestep impact |
| SDLoRA (Sparse Dim Tuning) | ✅ Yes | NeurIPS 2024 |

---

## Related Topics

- [Modular-MOE-Merging](./Modular-MOE-Merging/) — Expert merging for MoE models
- [Reasoning-r1](./Reasoning-r1/) — Reasoning-focused post-training
- [Acceleration-Frameworks](./Acceleration-Frameworks/) — Fine-tuning acceleration frameworks
