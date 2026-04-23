# Projector-Tuning

## 📌 Overview

**Projector-Tuning** (also called connector/projection tuning) is a PEFT strategy for multimodal LLMs where only the **vision-language projector/connector** is trained, while both the **vision encoder** and **LLM** remain frozen. The projector maps visual features into the LLM's embedding space.

> *"By keeping both the vision encoder and language model frozen during initial training, LLaVA minimizes the number of trainable parameters to just the projection layer."*
> — [LLaVA Architecture](https://mbrenndoerfer.com/writing/llava-architecture-visual-instruction-tuning)

---

## Fine-Tuning Strategy Comparison

| Strategy | Vision Encoder | Projector | LLM | Use Case |
|----------|---------------|-----------|-----|----------|
| **Projector-only** | Frozen | Train | Frozen | Low-cost bootstrapping |
| **Projector + LLM** | Frozen | Train | Train | Visual instruction tuning |
| **Full Fine-tuning** | Train | Train | Train | Maximum performance |

> *"Projector-only tuning enables multimodal capability with minimal compute (can run on single GPU)."*
> — [Medium: Projectors and Fine-Tuning in VLMs](https://medium.com/@zdj0712/introduction-to-multimodal-learning-part-3-projectors-and-fine-tuning-strategies-in-vlms-71feaceb698d)

---

## Projector Architectures

### LLaVA: Linear → MLP

| Version | Projector | Description |
|---------|-----------|-------------|
| LLaVA (original) | Linear projection | Single linear layer |
| LLaVA-1.5 | **MLP** | 2-layer MLP with ReLU activation |

> *"LLaVA-1.5 uses an MLP projector with 2 layers (activated by ReLU) instead of a simple linear layer."*
> — [LLaVA Paper](https://static.hliu.cc/files/llava/improved_llava.pdf)

---

### Honeybee: C-Abstractor & D-Abstractor — NeurIPS 2023

**Locality-enhanced projectors** addressing two key properties:
- **Flexibility**: Ability to manage number of visual tokens
- **Locality preservation**: Maintaining local context from visual features

| Abstractor | Type | Method |
|-----------|------|--------|
| **C-Abstractor** | Convolutional | ResNet blocks with adaptive average pooling |
| **D-Abstractor** | Deformable | Enhanced locality via deformable attention |

> *"Honeybee achieves 73.6% on MMBench vs previous SoTA of 67.7%, with 3× faster inference."*
> — [Honeybee Paper](https://arxiv.org/html/2312.06742v2)

| Detail | Value |
|--------|-------|
| Paper | [arXiv:2312.06742](https://arxiv.org/abs/2312.06742) |

---

### ProDiaL: Projector-Targeted Diagonal-Centric — CVPR 2025

First study to analyze projectors as key components for transfer learning in Mamba architecture.

> *"Projectors constitute ~65% of model parameters and are predominant contributors to transfer learning."*
> — [ProDiaL Paper](http://openaccess.thecvf.com/content/CVPR2025/papers/Ham_Parameter_Efficient_Mamba_Tuning_via_Projector-targeted_Diagonal-centric_Linear_Transformation_CVPR_2025_paper.pdf)

**Key Finding:** Fine-tuned projectors can be approximated via near-diagonal linear transformation.

> *"Diagonal entries accumulate large gradients; off-diagonal entries show minor changes."*
> — [ProDiaL Paper](http://openaccess.thecvf.com/content/CVPR2025/papers/Ham_Parameter_Efficient_Mamba_Tuning_via_Projector-targeted_Diagonal-centric_Linear_Transformation_CVPR_2025_paper.pdf)

**Method:**
```
W' = s·W·D_b + ε
```
- `D_b` = block-diagonal matrix (trains diagonal entries)
- `ε` = off-diagonal matrix (trained via LoRA)
- `s` = learnable scaling parameter

**Results:**

| Method | Mamba LLM (Avg) | Params |
|--------|-----------------|--------|
| Full-FT | 43.43% | 130M |
| LoRA | 43.71% | 2.36M |
| **ProDiaL** | **44.06%** | **2.42M** |

---

### Dense Connector — NeurIPS 2024

Leverages **multi-layer visual features** instead of just final layer features.

| Connector | Method | Token Reduction |
|-----------|--------|-----------------|
| STI | Sparse Token Integration | — |
| SCI | Sparse Channel Integration | — |
| **DCI** ✓ | Dense Channel Integration | **75% fewer tokens** |

> *"Dense Connector achieves performance comparable to LLaVA-v1.5 with only **25% of visual tokens** and **3× faster inference**."*
> — [Dense Connector Paper](https://proceedings.neurips.cc/paper_files/paper/2024/file/3a10c46572628d58cb44fb705f25cbbf-Paper-Conference.pdf)

---

### MoReS: Modality Representation-Steering — ACL 2025

**Representation steering** approach instead of weight modification.

> *"MoReS achieves **287 to 1,150 times fewer trainable parameters** than LoRA while maintaining comparable performance."*
> — [MoReS Paper](https://arxiv.org/html/2412.12359v1)

| Method | Parameters (3B model) |
|--------|----------------------|
| Full Fine-tuning | 2.78B |
| LoRA | 188.74M |
| **MoReS-H** | **0.655M** |

> *"Addresses modality imbalance where textual information dominates attention distribution."*
> — [MoReS Paper](https://arxiv.org/html/2412.12359v1)

| Detail | Value |
|--------|-------|
| Paper | [arXiv:2412.12359](https://arxiv.org/abs/2412.12359) |

---

### Delta-LLaVA: DeltaProjection — WACV 2026

**Low-rank approach** to align multi-level vision features into compact subspace.

> *"Delta-LLaVA drastically reduces visual token budget while enhancing reasoning performance."*
> — [Delta-LLaVA Paper](https://arxiv.org/html/2512.18910v1)

Key innovation: Combines low-rank DeltaProjection with specialization layers (EMHSA + Transformer block).

---

## Projector Evolution Timeline

```
Linear (LLaVA)
  ↓
MLP (LLaVA-1.5)
  ↓
Convolutional/Deformable (Honeybee C/D-Abstractor)
  ↓
Multi-layer aggregation (Dense Connector)
  ↓
Low-rank diagonal (ProDiaL)
  ↓
Representation steering (MoReS)
  ↓
Low-rank Delta (Delta-LLaVA)
```

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [LLaVA Improved](https://static.hliu.cc/files/llava/improved_llava.pdf) |
| **Paper** | [arXiv:2312.06742 — Honeybee](https://arxiv.org/abs/2312.06742) |
| **Paper** | [CVPR 2025 — ProDiaL](http://openaccess.thecvf.com/content/CVPR2025/papers/Ham_Parameter_Efficient_Mamba_Tuning_via_Projector-targeted_Diagonal-centric_Linear_Transformation_CVPR_2025_paper.pdf) |
| **Paper** | [NeurIPS 2024 — Dense Connector](https://proceedings.neurips.cc/paper_files/paper/2024/file/3a10c46572628d58cb44fb705f25cbbf-Paper-Conference.pdf) |
| **Paper** | [arXiv:2412.12359 — MoReS](https://arxiv.org/abs/2412.12359) |
| **Paper** | [arXiv:2512.18910 — Delta-LLaVA](https://arxiv.org/abs/2512.18910) |

---

## Related Topics

- [VLM.md](./VLM.md) — Vision-Language Model architectures
- [Vision-Encoding.md](./Vision-Encoding.md) — Vision encoding strategies
- [Vision-LoRA.md](./Vision-LoRA.md) — LoRA for vision components
