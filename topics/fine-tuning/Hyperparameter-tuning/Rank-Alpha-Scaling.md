# Rank-Alpha Scaling

## 📌 Overview

| Field | Value |
|-------|-------|
| **Topic** | How LoRA rank (r) and alpha (α) interact and scale |
| **Core Question** | How should scaling factor γᵣ behave as rank increases? |
| **Key Papers** | LoRA (Hu et al.), rsLoRA (Kalajdzievski), LoRA+ (Hayou et al.) |
| **Key Concept** | α must be ≥ r for effective updates (α/r ≥ 1 constraint) |

> "Rank determines the capacity and size of a LoRA adapter, while alpha determines the strength of its contribution."

— [Sebastian Raschka](https://sebastianraschka.com/faq/docs/lora-rank-alpha.html)

---

## Definitions

### Core LoRA Formulation

```
h = (W₀ + γᵣBA)x
```

Where:
- B ∈ ℝ^(d_out × r), A ∈ ℝ^(r × d_in) — trainable low-rank matrices
- r = rank (inner dimension of low-rank decomposition)
- γᵣ = scaling factor linking rank and adaptation strength
- α = hyperparameter controlling scaling multiplier

### Standard Scaling: α/r

```
γᵣ = α / r
h = W₀x + (α/r)BAx
```

> "In the repo code, the LoRA contribution is scaled by alpha / rank, which helps make runs across different ranks easier to compare without constantly retuning everything else."

— [Sebastian Raschka](https://sebastianraschka.com/faq/docs/lora-rank-alpha.html)

### rsLoRA Scaling: α/√r

> "Unless γᵣ ∈ Θ(1/√r), learning becomes unstable or collapsing for sufficiently large r."

— [Tenyx, arXiv:2312.03732](https://arxiv.org/abs/2312.03732)

### MIUB Scaling Law (arXiv:2501.03152)

> "Higher MIUB = stronger dependency on base LLM (weaker domain adaptation). Lower MIUB = LoRA has learned more generalized, new knowledge."

### μA Framework (arXiv:2602.06204)

Maximal-Update Adaptation provides rank-invariant learning rate scaling:

| Initialization | α Setting | LR Scaling | Transfers to Full FT? |
|--------------|-----------|-----------|---------------------|
| Init[A] (A random, B=0) | 1 | η ∝ r^(-1/2) | No |
| Init[B] (B random, A=0) | 1 | η ∝ n^(-1) | **Yes** |

---

## Key Sources

### Academic Papers

| Paper | arXiv | Key Contribution |
|-------|-------|-----------------|
| LoRA: Low-Rank Adaptation of Large LLMs | [2106.09685](https://arxiv.org/abs/2106.09685) | Original α/r scaling formulation |
| rsLoRA: Rank Stabilization | [2312.03732](https://arxiv.org/abs/2312.03732) | Proved α/√r is optimal |
| MIUB Scaling Law | [2501.03152](https://arxiv.org/abs/2501.03152) | Mutual information upper bound for LoRA scaling |
| μA Framework | [2602.06204](https://arxiv.org/abs/2602.06204) | Rank-invariant LR that transfers to full fine-tuning |
| LoRA+ | [2402.12354](https://arxiv.org/abs/2402.12354) | Different learning rates for A and B matrices |

### Blog Posts

> "Think of alpha as a volume knob for the adapter. The rank determines the richness of what the adapter can say, and alpha determines how loudly it says it."

— [Michael Brenndoerfer](https://mbrenndoerfer.com/writing/lora-hyperparameters-rank-alpha-target-modules)

> "This saturation of performance with very low rank is not primarily due to a very low intrinsic dimensionality of the learning manifold, but rather, it stems from the stunted learning of LoRA outside of very low adapter ranks."

— [Hugging Face Blog](https://huggingface.co/blog/damjan-k/rslora)

### John Schulman: LoRA Without Regret

| Source | [Thinking Machines Lab](https://thinkingmachines.ai/blog/lora/) |
|--------|------|

> "Attention-only LoRA (rank 256, 0.25B params) underperforms MLP-only LoRA (rank 128, 0.24B params) despite having comparable trainable parameters."

**Key insight**: MLP/MoE layers are more critical than attention-only for LoRA effectiveness.

> "When configured correctly, LoRA matches Full Fine-Tuning (FullFT) in both sample efficiency and ultimate performance across most post-training scenarios."

---

## Scaling Factor Comparison

| Scaling Law | Formula | Paper |
|------------|---------|-------|
| Standard LoRA | γᵣ = α/r | Hu et al. (2021) |
| rsLoRA | γᵣ = α/√r | Kalajdzievski (2023) |
| MIUB | MIUB(N,R,D) = A(N₀/N)^α + B(R₀/R)^β + C(D₀/D)^γ | arXiv:2501.03152 |

---

## Rank Guidelines

| Rank Range | Use Case |
|-----------|----------|
| 1–4 | RL tasks, very simple tasks |
| 4–8 | Simple classification, sentiment analysis |
| 16–32 | Sweet spot for most instruction tuning |
| 64–128 | Complex behavior changes, multilingual |
| 256–2048 | rsLoRA enables effective high-rank training |

---

## Alpha Configuration Patterns

| Configuration | Effective Scaling | Use Case |
|-------------|------------------|----------|
| α = r | 1.0 | Default neutral baseline |
| α = 2r | 2.0 | Faster/stronger adaptation |
| α = r/2 | 0.5 | Conservative, preserves pre-training |
| Fixed α (8 or 16) | Varies with rank | Simplifies tuning; recommended for rsLoRA |

**Critical constraint**: α must be ≥ r for effective updates (α/r ≥ 1).

---

## Key Experimental Findings

| Finding | Source | Details |
|---------|--------|---------|
| Higher rank → lower MIUB (better adaptation) | arXiv:2501.03152 | Tested ranks 32, 128, 512 |
| rsLoRA gradient norms stable across all ranks | arXiv:2312.03732 | Ranks 4–2048 tested |
| LoRA matches FFT at rank 1 for RL | Schulman (2025) | Information-theoretic argument |
| MLP layers more important than attention | Schulman (2025) | MLP-only rank 128 ≈ All-layer rank 256 |
| Optimal LR approximately rank-invariant | Schulman (2025) | With Init[B] initialization |

---

## Debates

### Low Rank Sufficiency Controversy

Original LoRA paper (Hu et al., 2021): "very low ranks suffice."

rsLoRA counter-argument: This conclusion was an artifact of incorrect α/r scaling causing gradient collapse at higher ranks.

### rsLoRA Value Debate

> "If you have already determined a good alpha value, there is IMHO no reason to run more hyper-parameter searches with rslora enabled."

— BenjaminBossan (PEFT Maintainer)

### Attention-Only vs All-Layer LoRA

MLP/MoE layers are critical — attention-only LoRA underperforms comparable-parameter MLP-only LoRA.

---

## Related Topics

- [rsLoRA.md](./rsLoRA.md) — Rank-stabilized scaling implementation
- [Paged-Optimizers.md](./Paged-Optimizers.md) — Memory optimization for high-rank training
- [LoRA](../PEFT-Consumer-VRAM/LoRA.md) — Base LoRA documentation

---

## References

- [LoRA (arXiv:2106.09685)](https://arxiv.org/abs/2106.09685)
- [rsLoRA (arXiv:2312.03732)](https://arxiv.org/abs/2312.03732)
- [MIUB Scaling Law (arXiv:2501.03152)](https://arxiv.org/abs/2501.03152)
- [μA Framework (arXiv:2602.06204)](https://arxiv.org/abs/2602.06204)
- [LoRA+ (arXiv:2402.12354)](https://arxiv.org/abs/2402.12354)
- [Sebastian Raschka — LoRA rank/alpha FAQ](https://sebastianraschka.com/faq/docs/lora-rank-alpha.html)
- [Hugging Face Blog — rsLoRA](https://huggingface.co/blog/damjan-k/rslora)
- [Schulman — LoRA Without Regret](https://thinkingmachines.ai/blog/lora/)
