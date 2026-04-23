# Router Tuning in MoE

## 📌 Overview

Router tuning is the process of calibrating the expert-selection mechanism in a Mixture-of-Experts model after experts have been independently trained and merged. The router determines which expert processes each token — making router training a critical final step in modular MoE pipelines like BAR and BTX.

> *"After domain-specific experts are trained independently and combined into an MoE architecture, the router must be calibrated to learn optimal token-level expert selection patterns."*
> — [arXiv:2604.18473](https://arxiv.org/html/2604.18473v1)

---

## Router Architecture

### Standard MoE Router

A router is a **linear layer per transformer block** that outputs a probability distribution over experts:

```
g(W_l x) = SoftMax(TopK(W_l x))
```

| Component | Description |
|-----------|-------------|
| **W_l** | Trainable routing transformation (per layer) |
| **TopK** | Select top-k experts per token (k=2 default in BTX) |
| **SoftMax** | Normalize over selected experts |

> *"BTX uses top-k routing (k=2 default). Load balancing auxiliary loss: L_LB = αN Σ_i=1^N u_i p_i where α=0.01 default."*
> — [arXiv:2403.07816](https://arxiv.org/html/2403.07816v1)

---

## Key Methods

### BAR Router Training

> *"Stage 3: Router training — only router parameters trained (all other weights frozen). Uses 5% stratified SFT sample from all domains."*
> — [Ai2 Blog](https://allenai.org/blog/bar)

> *"Router training is inexpensive compared to expert training."*
> — [arXiv:2604.18473](https://arxiv.org/html/2604.18473v1)

### BTX Router Training

> *"MoE Finetuning: Train router on mixed domain data. Only new parameters are router transformations W_l (negligible size)."* — [arXiv:2403.07816](https://arxiv.org/html/2403.07816v1)

### Load Balancing in BTX

> *"Without LB: Math expert dominates all domains. With LB: Code expert 'comes back to life' and becomes dominant."*
> — [arXiv:2403.07816](https://arxiv.org/html/2403.07816v1)

| Configuration | Math Expert | Code Expert | Trade-off |
|---------------|-------------|-------------|-----------|
| **Without Load Balancing** | Dominates | Dies | Math wins, code loses |
| **With Load Balancing** | Active | Active | LB improves coding, slightly degrades math |

### MergeME Perplexity-Based Routing (No Fine-Tuning)

> *"Perplexity routing requires only inference inputs, no training data access."*
> — [MergeME arXiv:2502.00997](https://arxiv.org/html/2502.00997v2)

```
PPL(x_inf | θᵢ) = exp(-(1/t) × Σⱼ₌₁ᵗ log P(xⱼ | x<ⱼ, θᵢ))
α = SoftMax(top-K(1/PPL₁, ..., 1/PPLₗ))
```

> *"Novel heuristic requiring no post-merge fine-tuning."*
> — [MergeME](https://arxiv.org/html/2502.00997v2)

### PERFT: Parameter-Efficient Routed Fine-Tuning

> *"MoE actually demands a mixture of adaptation modules — properly routed PEFT experts unlock more expressive adaptation space."*
> — [arXiv:2508.02587](https://arxiv.org/html/2508.02587v1)

| Variant | Description | Best For |
|---------|-------------|----------|
| **Perft** | Full routing with new G̃(⋅) | Extreme parameter efficiency |
| **Perft-E** | Reuses pretrained G(⋅) | Large number of PEFT experts, scarce data |
| **Perft-D** | Dense (no routing), shared experts | — |
| **Perft-S** | Single shared expert | Baseline |

> *"Up to 17.2% improvement on commonsense, 12.3% on arithmetic over LoRA baseline."*
> — [arXiv:2508.02587](https://arxiv.org/html/2508.02587v1)

### MoECondenser: Router Biases

> *"SFT for MoE is difficult because router layers are fragile; certain 'super experts' activate far more frequently."*
> — [OpenReview ICLR 2026](https://openreview.net/forum?id=DxbLY3Fctc)

> *"Auxiliary loss-free MoE SFT framework combining router biases with shared condenser experts."*
> — [MoECondenser](https://openreview.net/forum?id=DxbLY3Fctc)

### MergeKit MoE Gate Initialization

> *"Gate initialization modes for mergekit MoE creation."*
> — [GitHub: arcee-ai/mergekit](https://github.com/arcee-ai/mergekit/blob/main/docs/moe.md)

| Mode | Description | Use Case |
|------|-------------|----------|
| **hidden** | Uses hidden state representations | Best quality, requires model evaluation |
| **cheap_embed** | Uses raw token embeddings | Low-resource hardware |
| **random** | Random initialization | For further training (sparse upcycling) |

---

## Summary: Router Tuning Methods

| Method | Training After Merge? | Router Trainable Params | Key Innovation |
|--------|----------------------|------------------------|----------------|
| **BTX** | Yes (full fine-tune) | Linear layer per block | Load balancing loss |
| **BAR** | Yes (SFT sample) | Linear layer per block | 3-stage modular pipeline |
| **MergeME** | Optional | Varies | Perplexity-based routing (no fine-tune needed) |
| **PERFT** | Yes (PEFT) | PEFT experts + routing | Independent routing for PEFT experts |
| **MoECondenser** | Yes (SFT) | Router biases | Condenser experts aggregate knowledge |

---

## Key Findings

1. **Router training is essential** after merging experts — even "retraining-free" methods like perplexity routing benefit from calibration

2. **Load balancing is critical** — without it, dominant experts (e.g., Math) overshadow others, causing dead experts

3. **Frozen backbone + train router** is standard — both BTX and BAR freeze all non-router parameters during router tuning

4. **Router initialization matters** — hidden state-based initialization outperforms embedding-based or random initialization

5. **PEFT + routing synergy** — PERFT shows combining PEFT modules with proper routing significantly outperforms treating MoE as dense

6. **Sparse activation efficiency** — total PEFT capacity matters more than activated parameters ratio; increasing sparsity does not severely impact performance

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2604.18473 — BAR](https://arxiv.org/abs/2604.18473) |
| **Paper** | [arXiv:2403.07816 — BTX](https://arxiv.org/abs/2403.07816) |
| **Paper** | [arXiv:2502.00997 — MergeME](https://arxiv.org/html/2502.00997v2) |
| **Paper** | [arXiv:2508.02587 — PERFT](https://arxiv.org/html/2508.02587v1) |
| **Paper** | [arXiv:2411.08212 — PERFT v1](https://arxiv.org/html/2411.08212v1) |
| **Paper** | [OpenReview — MoECondenser](https://openreview.net/forum?id=DxbLY3Fctc) |
| **Docs** | [GitHub: mergekit MoE](https://github.com/arcee-ai/mergekit/blob/main/docs/moe.md) |

---

## Related Topics

- [BAR](./BAR.md) — Full BAR pipeline (includes router tuning)
- [Expert-Adaption](./Expert-Adaption.md) — Expert training and merging methods
