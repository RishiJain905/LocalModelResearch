# Hyperparameter-tuning

## 📌 Overview

This section covers key hyperparameter strategies and optimization techniques for LoRA fine-tuning — including scaling factor theory (rsLoRA), rank/alpha interactions, and memory-efficient optimizers (paged AdamW).

---

## Subtopics

| File | Topic | Type |
|------|-------|------|
| [rsLoRA.md](./rsLoRA.md) | Rank-Stabilized LoRA | Scaling factor (α/√r) |
| [Rank-Alpha-Scaling.md](./Rank-Alpha-Scaling.md) | Rank-Alpha Scaling | Hyperparameter theory |
| [Paged-Optimizers.md](./Paged-Optimizers.md) | Paged Optimizers | Memory management |

---

## Summary Matrix

| Technique | Key Innovation | Key Metric |
|-----------|-------------|-----------|
| **rsLoRA** | α/r → α/√r scaling | r=2048 perplexity 1.836 vs r=4 at 1.863 |
| **Rank-Alpha Scaling** | Theory of α/r interactions | α ≥ r constraint; α=r neutral baseline |
| **Paged Optimizers** | CUDA unified memory paging | 65B: >780GB → <48GB GPU |

---

## Related Topics

- [PEFT-Consumer-VRAM](./PEFT-Consumer-VRAM/) — LoRA, QLoRA base implementations
- [Acceleration-Frameworks](./Acceleration-Frameworks/) — Unsloth, custom kernels
