# Expert Adaption in MoE

## 📌 Overview

Expert adaption in Mixture-of-Experts systems refers to methods for training, adapting, and merging expert modules within MoE architectures. This covers parameter-efficient adaptation (PEFT for MoE), expert merging techniques, and strategies to prevent expert collapse or domination.

> *"State-of-the-art MoE merging methods only work with homogeneous model architectures and rely on simple unweighted averaging to merge expert layers, which does not address parameter interference."*
> — [MergeME arXiv:2502.00997](https://arxiv.org/html/2502.00997v2)

---

## Core Concepts

### Parameter Interference in Merging

When merging independently trained experts, **parameter interference** occurs when conflicting weight directions degrade performance:

> *"Simple unweighted averaging of expert parameters does not address parameter interference."*
> — [MergeME](https://arxiv.org/html/2502.00997v2)

### Expert Specialization vs. Load Balancing

> *"The conflict between expert specialization (encouraging distinct expert representations) and routing uniformity (load balancing). Load balancing losses encourage uniform routing, but this conflicts with expert specialization."*
> — [Expert Specialization arXiv:2505.22323](https://arxiv.org/html/2505.22323v3)

---

## Key Methods

### BAR (Branch-Adapt-Route)

> *"Pre-training primarily updates knowledge representations (well captured by FFNs), while post-training requires adapting broader behaviors, including attention patterns and handling new tokens."*
> — [Ai2 BAR Blog](https://allenai.org/blog/bar)

Each expert trained as 2-expert MoE (frozen anchor + trainable domain expert) with progressive unfreezing of shared layers.

### BTX (Branch-Train-MiX)

> *"BTX branches from a seed model, trains experts in parallel on specialized domains (math, code, world knowledge), then merges feedforward parameters into MoE layers while averaging attention weights."*
> — [arXiv:2403.07816](https://arxiv.org/abs/2403.07816)

### MoRE (Mixture of Low-Rank Experts)

> *"Treats rank values as 'experts' rather than entire LoRA modules. Uses adaptive rank selection via task embeddings and contrastive learning."*
> — [OpenReview LLM-QAT/Low-Rank](https://openreview.net/pdf?id=LWvgajBmNH)

| Method | GLUE AVG | Notes |
|--------|----------|-------|
| MoRE | **87.3** | +1.7% over standard LoRA r=16 |
| Standard LoRA r=16 | 85.6 | Baseline |

### SMEAR (Soft Merging with Adaptive Routing)

> *"Avoids discrete routing by constructing a single merged expert via weighted average of all expert parameters. Enables standard gradient-based training."*
> — [arXiv:2306.03745](https://arxiv.org/abs/2306.03745)

### MoECondenser

> *"Introduces shared condenser experts with router biases for SFT. Addresses router fragility issue where 'super experts' dominate activation."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=DxbLY3Fctc)

> *"Pruning and interexpert correlation analyses confirm that our condenser experts aggregate knowledge from the long-tail experts, preserving performance under sparse routing."*
> — [MoECondenser](https://openreview.net/forum?id=DxbLY3Fctc)

---

## Expert Merging Methods

### Comparison Table

| Method | Approach | Benefit | Source |
|--------|----------|---------|--------|
| **TIES** | Sign alignment + pruning | Drops bottom (100-p)% of redundant params, keeps only same-sign values | [MergeME](https://arxiv.org/html/2502.00997v2) |
| **DARE** | Random dropout + rescaling | +9.72% over BTX averaging | [MergeME](https://arxiv.org/html/2502.00997v2) |
| **Perplexity Routing** | Route to lowest PPL experts | No fine-tuning needed | [MergeME](https://arxiv.org/html/2502.00997v2) |
| **SMEAR** | Weighted average → single merged expert | Standard gradient training works | [arXiv:2306.03745](https://arxiv.org/abs/2306.03745) |

### TIES Merging

> *"TIES Merging: Drop bottom (100-p)% of redundant parameters, keep only same-sign values."*
> — [MergeME](https://arxiv.org/html/2502.00997v2)

### DARE Merging

> *"DARE: Randomly drop parameters, rescale task vectors. Achieves +9.72% over BTX averaging."*
> — [MergeME](https://arxiv.org/html/2502.00997v2)

### Perplexity-Based Routing

> *"Perplexity routing requires only inference inputs, no training data access."*
> — [MergeME](https://arxiv.org/html/2502.00997v2)

```
PPL(x_inf | θᵢ) = exp(-(1/t) × Σⱼ₌₁ᵗ log P(xⱼ | x<ⱼ, θᵢ))
α = SoftMax(top-K(1/PPL₁, ..., 1/PPLₗ))
```

---

## Expert Specialization Enhancement

> *"Proposes orthogonality loss (encourages distinct expert representations) and variance loss (discriminative routing) to address expert overlap from load balancing losses."*
> — [arXiv:2505.22323](https://arxiv.org/html/2505.22323v3)

---

## Task-Aware Online Expert Merging

### Tanbr

> *"Neural bandit framework for continuous merging weight space. Achieves 45%+ latency reduction, 25% memory reduction while maintaining accuracy."*
> — [arXiv:2509.19781](https://arxiv.org/html/2509.19781v1)

Uses tree-structured partitioning and UCB-based selection.

---

## Key Principles

| Principle | Description |
|-----------|-------------|
| **Shared frozen params work for pretraining but fail for post-training** | BAR finding: FFN-only freezing works for pretraining but fails for post-training behaviors |
| **Domain-only SFT degrades general capabilities** | Mixing domain-specific and general data is critical |
| **Parameter interference is major challenge** | Merging divergent expert models causes degradation |
| **Progressive unfreezing enables proper divergence** | Embeddings → LM head → all shared params during RLVR |
| **Sophisticated merging essential for small models** | TIES/DARE needed for small; simple averaging suffices for large instruction-tuned |
| **Router training is critical final step** | Lightweight but necessary after expert merging |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2604.18473 — BAR](https://arxiv.org/abs/2604.18473) |
| **Paper** | [arXiv:2403.07816 — BTX](https://arxiv.org/abs/2403.07816) |
| **Paper** | [arXiv:2502.00997 — MergeME](https://arxiv.org/html/2502.00997v2) |
| **Paper** | [arXiv:2306.03745 — SMEAR](https://arxiv.org/abs/2306.03745) |
| **Paper** | [OpenReview — MoECondenser](https://openreview.net/forum?id=DxbLY3Fctc) |
| **Paper** | [arXiv:2505.22323 — Expert Specialization](https://arxiv.org/html/2505.22323v3) |
| **Paper** | [arXiv:2509.19781 — Tanbr](https://arxiv.org/html/2509.19781v1) |
| **Paper** | [OpenReview — MoRE](https://openreview.net/pdf?id=LWvgajBmNH) |

---

## Related Topics

- [BAR](./BAR.md) — Branch-Adapt-Route pipeline
- [Router-Tuning](./Router-Tuning.md) — Router training after expert merging
