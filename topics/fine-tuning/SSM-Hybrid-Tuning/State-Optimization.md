# State Optimization in SSMs

## 📌 Overview

State optimization in State Space Models encompasses techniques for compressing, expanding, and efficiently passing the SSM hidden state. The SSM state (of dimension `d_state`) acts as a compressed memory — larger states are more expressive but slower; smaller states are faster but less capable.

> *"d_state: SSM state expansion factor (typically 16 in Mamba). Total state size: P × N (channels × state dimension). Trade-off: Larger state = more expressive but slower inference."*
> — [Mamba Architecture Documentation](https://github.com/state-spaces/mamba)

---

## State Compression

### CompreSSM (ICLR 2026)

> *"Uses Hankel Singular Value (HSV) analysis from control theory to identify and truncate low-energy state dimensions during training."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=LtzmeSMBTW)

**Key Finding:**

> *"SSMs trained large and shrunk during training outperform models trained directly at smaller dimensions."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=LtzmeSMBTW)

**Error Bound:**

```
‖G - Ĝ‖∞ ≤ 2·Σᵢ₌ᵣ₊₁ⁿ σᵢ
```

> *"Truncation error bounded by discarded singular values."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=LtzmeSMBTW)

**Results:**

| Model | CIFAR-10 Accuracy | State Dimension |
|-------|-------------------|-----------------|
| **CompreSSM** | **84.4%** | dim 57 |
| Baseline | 78.2% | dim 57 |

> *"CIFAR10: 84.4% at dim 57 vs baseline 78.2% at same dimension."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=LtzmeSMBTW)

**HSV Property:**

> *"HSV ordering stabilizes early in training; bottom-r dimensions stay negligible."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=LtzmeSMBTW)

---

### SDLoRA / SDT (Sparse Dimension Tuning)

> *"Selectively updates only important channels and hidden states in SSM modules while applying LoRA to linear projections."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

> *"Fine-tuning Win encompasses the expressivity of fine-tuning WB, WC, WΔ,↑."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

**Performance:**

| Method | GLUE Score | Notes |
|--------|------------|-------|
| **SDLoRA** | **81.05** | Similar param count to LoRA |
| LoRA | 80.79 | Baseline |

---

## State Expansion

### Mamba-2 State Space Duality (SSD)

> *"Mamba-2 connects SSMs to structured matrices via State Space Duality (SSD). Preserves efficient FLOP counts as SSMs while dramatically speeding up training using matmuls."*
> — [Tri Dao Blog](https://tridao.me/blog/2024/mamba2-part3-algorithm/)

### Mamba-3 MIMO

> *"Adds parallel work inside each update without increasing decode latency."*
> — [Mamba Release Notes](https://github.com/state-spaces/mamba/releases)

Mamba-3's MIMO (Multi-Input, Multi-Output) expands B/C matrices for more expressive state dynamics.

### d_state Parameter

| Version | d_state | Trade-off |
|---------|---------|-----------|
| Mamba-1 | 16 | Baseline |
| Mamba-2 | Larger (SSD) | Faster training, slightly less expressivity per state |
| Mamba-3 | Optimized | Best overall efficiency |

---

## Efficient State Passing

### State-offset Tuning (ACL 2025)

> *"Adds learnable state-offset h' directly to hidden state at every timestep: ŷ_t = y_t + C_th'."*
> — [arXiv:2503.03499](https://arxiv.org/abs/2503.03499)

| Variant | Parameters | Effect |
|---------|-----------|--------|
| State-offset (h) | D×H | Larger expressivity |
| State-offset (y) | D only | 0.01% of parameters |

> *"Consistent impact at every timestep vs prefix-tuning which diminishes over time."*
> — [arXiv:2503.03499](https://arxiv.org/abs/2503.03499)

---

## Hybrid Architectures for State Management

### SAmBA (Samba) — Mamba + Sliding Window Attention

> *"Efficiently extrapolates to 256K context length with perfect memory recall on Passkey Retrieval."*
> — [OpenReview](https://openreview.net/forum?id=bIlnpVM4bc)

> *"3.73× higher throughput compared to Transformers with grouped-query attention for 128K prompts."*
> — [OpenReview](https://openreview.net/forum?id=bIlnpVM4bc)

### Hymba — Parallel Attention + SSM

> *"Attention heads provide high-resolution recall, while SSM heads facilitate efficient context summarization."*
> — [arXiv:2411.13676](https://arxiv.org/abs/2411.13676)

> *"~20× smaller KV cache and 3× higher throughput."*
> — [arXiv:2411.13676](https://arxiv.org/abs/2411.13676)

**Key Innovation:**

> *"Meta tokens (128 fixed tokens) bootstrap hidden state AND mitigate attention sinks."*
> — [arXiv:2411.13676](https://arxiv.org/abs/2411.13676)

### Jamba — Transformer + Mamba + MoE

> *"Effective context length of 256K tokens."*
> — [AI21 Blog](https://www.ai21.com/blog/announcing-jamba/)

> *"Hybrid structure renders 12B active parameters more efficient than Transformer-only equivalent."*
> — [AI21 Blog](https://www.ai21.com/blog/announcing-jamba/)

---

## Key Findings

| Finding | Source | Quote |
|---------|--------|-------|
| Prompt-based PEFT fails on SSMs | ICML 2025 | "Maximum expressiveness via prefix-tuning = tuning initial state h₀ only" |
| LoRA on projections > LoRA on SSM | ICML 2025 | "LoRA on linear projections outperforms LoRA on SSM modules" |
| SSMs need hybrid for recall | arXiv:2507.12442 | "Pure Mamba/Mamba-2 lag on associative recall/in-context learning" |
| SSM state compression is principled | ICLR 2026 | "HSV ordering stabilizes early in training; bottom-r dimensions stay negligible" |
| SSM state compression works | ICLR 2026 | "SSMs trained large and shrunk outperform directly trained small models" |

---

## Summary Table: Optimization Strategies

| Strategy | Method | Key Benefit |
|----------|--------|-------------|
| **State Compression** | CompreSSM (HSV truncation) | Principled, theory-backed compression during training |
| **State Compression** | SDT (Sparse Dimension Tuning) | Selective channel/state updates |
| **State Expansion** | d_state parameter (Mamba-2/3) | More expressive representations |
| **State Expansion** | MIMO (Mamba-3) | Parallel work without decode latency |
| **Efficient Passing** | State-offset tuning | Consistent timestep influence |
| **Hybrid Architecture** | SAmBA/Hymba/Jamba | SSM (compressed memory) + Attention (perfect recall) |

---

## Sources

| Type | Citation |
|------|----------|
| **ICLR 2026** | [CompreSSM / MambaPEFT](https://openreview.net/forum?id=LtzmeSMBTW) |
| **Paper** | [arXiv:2410.09016 — PEFT for SSMs (ICML 2025)](https://arxiv.org/abs/2410.09016) |
| **Paper** | [arXiv:2503.03499 — State-offset Tuning (ACL 2025)](https://arxiv.org/abs/2503.03499) |
| **Paper** | [arXiv:2411.13676 — Hymba](https://arxiv.org/abs/2411.13676) |
| **Paper** | [OpenReview — SAmBA/Samba](https://openreview.net/forum?id=bIlnpVM4bc) |
| **Paper** | [arXiv:2403.19887 — Jamba](https://arxiv.org/abs/2403.19887) |
| **Blog** | [Tri Dao Mamba-2 Blog](https://tridao.me/blog/2024/mamba2-part3-algorithm/) |
| **GitHub** | [state-spaces/mamba](https://github.com/state-spaces/mamba) |

---

## Related Topics

- [Mamba-3](./Mamba-3.md) — SSM architecture with MIMO and complex-valued states
- [Jamba](./Jamba.md) — Hybrid Transformer-Mamba with MoE
