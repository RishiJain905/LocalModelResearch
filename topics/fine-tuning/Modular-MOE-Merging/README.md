# Modular-MOE-Merging

## 📌 Overview

Methods for training, merging, and routing Mixture-of-Experts (MoE) modules in large language models. Covers modular post-training pipelines, expert merging techniques, and router calibration strategies.

> *"Rather than training a single model on all data sequentially (which causes catastrophic forgetting), modular MoE merging trains separate experts per domain and composes them into a unified MoE model."*
> — [Ai2 BAR Blog](https://allenai.org/blog/bar)

---

## Topics

| Topic | Description |
|-------|-------------|
| [BAR](./BAR.md) | Branch-Adapt-Route — AI2's modular post-training pipeline with 3-stage expert merging |
| [Expert-Adaption](./Expert-Adaption.md) | Methods for training, adapting, and merging expert modules in MoE |
| [Router-Tuning](./Router-Tuning.md) | Router calibration after expert merging — training strategies and key findings |

---

## Taxonomy Matrix

| Method | Origin | Training After Merge? | Key Innovation |
|--------|--------|----------------------|----------------|
| **BAR** | AI2 (2025) | Yes (5% SFT sample) | 3-stage modular pipeline, progressive unfreezing |
| **BTX** | Meta (2024) | Yes (full fine-tune) | Embarrassingly parallel expert training, attention averaging |
| **MergeME** | Academic (2025) | Optional (perplexity routing) | TIES/DARE merging, no fine-tune option |
| **PERFT** | Academic (2025) | Yes (PEFT) | PEFT experts with independent routing |
| **MoECondenser** | ICLR 2026 | Yes (SFT) | Router biases + condenser experts |
| **SMEAR** | Academic (2023) | Yes | Soft merging via weighted average |

---

## Key Concepts

### Parameter Interference

> *"State-of-the-art MoE merging methods only work with homogeneous model architectures and rely on simple unweighted averaging to merge expert layers, which does not address parameter interference."*
> — [MergeME arXiv:2502.00997](https://arxiv.org/html/2502.00997v2)

### Expert Specialization vs Load Balancing

> *"The conflict between expert specialization (encouraging distinct expert representations) and routing uniformity (load balancing). Load balancing losses encourage uniform routing, but this conflicts with expert specialization."*
> — [Expert Specialization arXiv:2505.22323](https://arxiv.org/html/2505.22323v3)

### Why Progressive Unfreezing for Post-Training

> *"Pre-training primarily updates knowledge representations (well captured by FFNs), while post-training requires adapting broader behaviors, including attention patterns and handling new tokens."*
> — [Ai2 BAR Blog](https://allenai.org/blog/bar)

---

## Performance Comparison

### OLMo 2 7B — 19 Benchmarks

| Model | Overall | Math | Code | Safety |
|-------|---------|------|------|--------|
| **BAR 5×7B** | **49.1** | **56.2** | **49.9** | **94.0** |
| Full retraining (ceiling) | 50.5 | 55.9 | 59.6 | 90.4 |
| BTX 5×7B | 46.7 | 62.1 | 32.1 | 93.9 |
| Model merging w/ mid-train | 6.5 | 0.3 | 0.4 | 9.1 |

---

## Related Topics

- [Reasoning-r1](../Reasoning-r1/) — Reasoning-focused post-training (GRPO, RLVR, GIFT, CoT)
- [Acceleration-Frameworks](../Acceleration-Frameworks/) — Fine-tuning acceleration (Unsloth, Axolotl, QAT, FSDP2)
