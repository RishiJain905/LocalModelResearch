# Projector-Alignment

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Visual Projector / Modality Connector |
| **Role** | Maps frozen vision encoder outputs into LLM embedding space |
| **Key Insight** | Bridge component in MLLMs that aligns vision features to text space |
| **Critical Debate** | Does the projector actually "align" — or does the LLM learn attributes independently? |

> "In Multimodal Large Language Models (MLLMs), a visual projector plays a crucial role in bridging pre-trained vision encoders with LLMs."

— [Honeybee (CVPR 2024)](https://arxiv.org/abs/2312.06742)

---

## Definitions

### What is a Projector?

A **visual projector** (also called modality connector) is the trainable bridge component in an MLLM that maps frozen vision encoder outputs into the LLM's embedding space.

### Projector Types

| Type | Description | Trainable Params |
|------|-------------|-----------------|
| **Linear Projector** | Single FC layer (LLaVA v1.0) | 1-5M |
| **MLP Projector** | 2-layer MLP with GELU (LLaVA v1.5+) | 10-20M |
| **Q-Former** | Transformer-based query module with 32 learned queries (BLIP-2) | 188M |
| **Perceiver Resampler** | Cross-attention based (Flamingo) | 20-100M |
| **C/D-Abstractor** | CNN/deformable attention locality-enhanced (Honeybee) | Variable |
| **TokenPacker** | Coarse-to-fine visual token packing | — |

---

## Key Sources

### BLIP-2 (arXiv:2301.12597)

| Field | Value |
|-------|-------|
| Paper | [BLIP-2: Bootstrapping Language-Image Pre-Training](https://arxiv.org/abs/2301.12597) |
| Code | [Salesforce/LAVIS](https://github.com/salesforce/LAVIS) |
| Authors | Junnan Li et al. (Salesforce Research), January 2023 |
| Trainable Params | 188M (Q-Former only) |

> "BLIP-2 achieves state-of-the-art results... outperforming Flamingo-80B by 8.7% on zero-shot VQAv2 with 54x fewer trainable parameters."

### LLaVA Improved Baselines (arXiv:2310.03744, CVPR 2024)

| Field | Value |
|-------|-------|
| Paper | [LLaVA: Improved Baselines](https://arxiv.org/abs/2310.03744) |
| Code | [haotian-liu/LLaVA](https://github.com/haotian-liu/LLaVA) |
| Authors | Haotian Liu et al. |

> "With simple modifications to LLaVA, namely, using CLIP-ViT-L-336px with an MLP projection... we achieve state-of-the-art results."

> "The vision-language cross-modal connector in LLaVA is surprisingly powerful."

### Honeybee (CVPR 2024)

| Field | Value |
|-------|-------|
| Paper | [Honeybee: Locality-Enhanced Universal Vision-Language Encode](https://arxiv.org/abs/2312.06742) |
| Code | [kakaobrain/honeybee](https://github.com/kakaobrain/honeybee) |
| Authors | Junbum Cha et al. (Kakao Brain) |

> "We propose a novel projector design that is both flexible and locality-enhanced."

### Prismatic VLMs (ICML 2024)

| Field | Value |
|-------|-------|
| Paper | [Prismatic VLMs](https://arxiv.org/abs/2402.07865) |
| Code | [TRI-ML/prismatic-vlms](https://github.com/TRI-ML/prismatic-vlms) |
| Authors | Siddharth Karamcheti et al. (MIT TRI-ML) |

> "Including the explicit projector pretraining stage is unnecessary, with single-stage training improving aggregate performance."

> "Full finetuning of the ViT significantly degrades performance."

### TokenPacker (arXiv:2407.02392)

| Field | Value |
|-------|-------|
| Paper | [TokenPacker](https://arxiv.org/abs/2407.02392) |
| Code | [CircleRadon/TokenPacker](https://github.com/CircleRadon/TokenPacker) |

> "TokenPacker with Vicuna-13B achieves 70.6% on VQAv2, 521 on OCRBench, and 70.0% on DocVQA."

### LVP — Language-guided Visual Projector (ICLR 2025)

| Field | Value |
|-------|-------|
| Venue | [ICLR 2025](https://openreview.net/forum?id=PxBzxO02Ef) |

> "LLaVA1.5-LVP with Qwen2.5-7B obtains 72.4% accuracy on VQAv2."

---

## ACL 2024 Controversy: Does the Projector Actually Align?

| Field | Value |
|-------|-------|
| Paper | [Projection in MLLMs](https://aclanthology.org/2024.acl-short.60.pdf) |
| Code | [claws-lab/projection-in-MLLMs](https://github.com/claws-lab/projection-in-MLLMs) |
| Authors | Gaurav Verma et al. |

> "The domain-specific visual attributes are modeled by the LLM, even when only the projection is fine-tuned."

> "As the MLLM gains domain-specific capabilities, the domain-specific richness of the post-projection representation consistently worsens."

**Implication**: The projector may NOT truly "align" visual attributes to text space. The LLM itself may be modeling the attributes independently.

---

## Repositories & Tools

| Repository | URL | Description |
|------------|-----|-------------|
| LLaVA | [GitHub](https://github.com/haotian-liu/LLaVA) | MLP projector; visual instruction tuning |
| LAVIS/BLIP-2 | [GitHub](https://github.com/salesforce/LAVIS) | Q-Former implementation |
| Honeybee | [GitHub](https://github.com/kakaobrain/honeybee) | C/D-Abstractor projectors |
| TokenPacker | [GitHub](https://github.com/CircleRadon/TokenPacker) | Coarse-to-fine packing |
| Prismatic VLMs | [GitHub](https://github.com/TRI-ML/prismatic-vlms) | Single-stage training findings |
| MiniGPT-4 | [GitHub](https://github.com/Vision-CAIR/MiniGPT-4) | Lightweight VLM framework |

---

## Benchmark Results

### Honeybee: Projector Comparison

| Projector Type | MMB | SEED-II | MMEP | MME |
|----------------|-----|---------|------|-----|
| Linear | 67.1 | 65.1 | 1556.5 | 70.0 |
| 2-layer MLP | 68.3 | 64.5 | 1557.2 | 70.2 |
| Resampler | 66.0 | 57.0 | 1389.6 | 64.2 |
| **C-Abstractor** | **69.2** | **64.2** | **1568.2** | **70.6** |

C-Abstractor outperforms all other projector types on MMBench.

---

## Debates & Design Choices

### Q-Former vs MLP Projector

| Aspect | Q-Former | MLP |
|--------|----------|-----|
| Token output | Fixed (32 queries) | All tokens from ViT |
| Parameter efficiency | Better (188M) | Worse (10-20M) |
| Token compression | Excellent | None |
| Fine-grained details | Loses some | Preserves all |
| Benchmark performance | Lower | Higher (LLaVA-1.5) |

> "It's a trade-off between the effectiveness of the QFormer and MLP projector. QFormer is good for token compression, while MLP uses all the tokens."

— InternLM-XComposer

### The Design Triangle (Honeybee)

| Projector | Flexibility | Locality |
|-----------|-------------|----------|
| Linear | No | Yes |
| Abstractor | Yes | No |
| **Locality-enhanced** | **Yes** | **Yes** |

### Two-Stage vs Single-Stage Training

- **Two-stage** (BLIP-2, LLaVA): Projector pretraining → LLM fine-tuning
- **Single-stage** (Prismatic VLMs): Joint training from scratch — may be sufficient

### Frozen vs Fine-tuned Vision Encoder

> "Full finetuning of ViT significantly degrades performance."

— Prismatic VLMs

---

## Historical Milestones

| Year | Paper/Model | Projector Type |
|------|------------|----------------|
| 2022 | Flamingo | Perceiver Resampler |
| 2023 | BLIP-2 | Q-Former |
| 2023 | LLaVA | Linear → MLP |
| 2024 | LLaVA-1.5/1.6 | MLP |
| 2024 | Honeybee | C/D-Abstractor |
| 2024 | TokenPacker | Coarse-to-fine |
| 2025 | LVP | Language-guided |

---

## Related Topics

- [VLM](./VLM.md) — Vision-Language Models that use projectors
- [Vision-Encoding](./Vision-Encoding.md) — Vision encoders that projectors connect to
- [BLIP-2](../model-cards/BLIP-2.md) — Q-Former introduction paper
- [LLaVA](../model-cards/LLaVA.md) — MLP projector breakthrough

---

## References

- [BLIP-2 (arXiv:2301.12597)](https://arxiv.org/abs/2301.12597)
- [LLaVA Improved Baselines (arXiv:2310.03744)](https://arxiv.org/abs/2310.03744)
- [Honeybee (arXiv:2312.06742)](https://arxiv.org/abs/2312.06742)
- [Prismatic VLMs (arXiv:2402.07865)](https://arxiv.org/abs/2402.07865)
- [TokenPacker (arXiv:2407.02392)](https://arxiv.org/abs/2407.02392)
- [ACL 2024 — Projection in MLLMs](https://aclanthology.org/2024.acl-short.60.pdf)
