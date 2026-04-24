# BAR: Branch-Adapt-Route

## 📌 Overview

**BAR** (Branch-Adapt-Route) is a modular post-training framework developed by [AI2 (Allen Institute for AI)](https://allenai.org) that trains independent domain experts separately and merges them into a unified [Mixture-of-Experts (MoE)](https://en.wikipedia.org/wiki/Mixture_of_experts) model. It addresses catastrophic forgetting by isolating domain pipelines during training, then composing them via an MoE architecture with a lightweight router training stage.

> *"Rather than training a single model on all data sequentially (which causes catastrophic forgetting), BAR trains separate experts per domain and composes them into a unified MoE model."*
> — [Ai2 Blog](https://allenai.org/blog/bar)

---

## Definition

### Formal Definition

BAR stands for **Branch-Adapt-Route** — a three-stage modular post-training pipeline:

| Stage | Name | Description |
|-------|------|-------------|
| **Stage 1** | Branch | Train independent domain experts (each as a 2-expert MoE: frozen "anchor" + trainable domain expert) |
| **Stage 2** | Adapt | Merge all domain experts + anchor into a single (k+1)-expert MoE |
| **Stage 3** | Route | Train only router parameters on 5% stratified SFT sample |

> *"Train separately, merge together: modular post-training with mixture-of-experts."*
> — [arXiv:2604.18473](https://arxiv.org/abs/2604.18473)

---

## Core Technical Details

### Why Progressive Unfreezing Matters for Post-Training

> *"Pre-training primarily updates knowledge representations (well captured by FFNs), while post-training requires adapting broader behaviors, including attention patterns and handling new tokens, e.g., `<|thinking|>`."*
> — [Ai2 Blog](https://allenai.org/blog/bar)

- **Pretraining (FlexOlmo):** Primarily updates knowledge → FFN layers → can freeze shared params
- **Post-training (BAR):** Requires behavioral shifts (new tokens, attention patterns) → must progressively unfreeze shared layers (embeddings → LM head → all shared params during RLVR)

### Three-Stage Pipeline

#### Stage 1: Independent Expert Training

Each domain expert is trained as a **2-expert MoE**:

| Component | Role |
|-----------|------|
| **Anchor expert** | Frozen pretrained model weights |
| **Domain expert** | Trainable parameters for specific domain (math, code, safety, tool use) |

Shared parameters are progressively unfrozen: embeddings → LM head → all shared params during RLVR.

#### Stage 2: Expert Merging

All k domain experts + 1 anchor are merged into a single **(k+1)-expert MoE**:

> *"Diverged shared parameters are averaged across models."*
> — [arXiv:2604.18473](https://arxiv.org/abs/2604.18473)

#### Stage 3: Router Training

> *"Only router parameters trained on 5% stratified SFT sample (all other weights frozen)."*
> — [Ai2 Blog](https://allenai.org/blog/bar)

Router is a simple linear layer per transformer block that outputs a probability distribution over all (k+1) experts.

---

## Performance Results

### OLMo 2 7B — 19 Benchmarks

| Model | Overall | Math | Code | Safety |
|-------|---------|------|------|--------|
| **BAR 5×7B** (proposed) | **49.1** | **56.2** | **49.9** | **94.0** |
| Full retraining (ceiling) | 50.5 | 55.9 | 59.6 | 90.4 |
| Retrain post-only | 47.8 | 48.7 | 43.6 | 92.8 |
| BTX 5×7B | 46.7 | 62.1 | 32.1 | 93.9 |
| Model merging w/ mid-train | 6.5 | 0.3 | 0.4 | 9.1 |

### Key Findings

> *"BAR outperforms retraining-post-only by +1.3 overall, +7.8 math, +6.3 code."*
> — [Ai2 Blog](https://allenai.org/blog/bar)

> *"Mid-training weight averaging causes catastrophic failure (6.5 overall). This demonstrates that naively merging models mid-training is catastrophic."*
> — [arXiv:2604.18473](https://arxiv.org/abs/2604.18473)

> *"Independent expert upgrades (Code v1→v2) achieve +16.5 gain without affecting other domains."*
> — [Ai2 Blog](https://allenai.org/blog/bar)

---

## Structural Advantages

| Advantage | Description |
|-----------|-------------|
| **Linear cost scaling** | Adding/updating domains costs O(n) vs O(n²) for monolithic retraining |
| **No catastrophic forgetting** | Each domain pipeline is fully isolated during training |
| **Distributed development** | Different teams can contribute separate experts independently |
| **Independent expert upgrades** | Update one expert without retraining others |

---

## Sources

| Type | Citation |
|------|----------|
| **Blog** | [Ai2 Blog — BAR](https://allenai.org/blog/bar) |
| **Paper** | [arXiv:2604.18473 — Train Separately, Merge Together](https://arxiv.org/abs/2604.18473) |
| **Paper (HTML)** | [arXiv HTML version](https://arxiv.org/html/2604.18473v1) |
| **Review** | [The Moonlight — Literature Review](https://www.themoonlight.io/en/review/train-separately-merge-together-modular-post-training-with-mixture-of-experts) |
| **News** | [24 AI News](https://24-ai.news/en/vijest/2026-04-21/allen-institute-bar-modular-moe) |

---

## Related Topics

- [Router-Tuning](./Router-Tuning.md) — Router training after expert merging
- [Expert-Adaption](./Expert-Adaption.md) — Expert training and adaptation methods in MoE
- [BTX](./BTX.md) — Branch-Train-MiX (alternative modular MoE approach)
