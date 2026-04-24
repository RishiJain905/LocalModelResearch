# Vision-LoRA

## 📌 Overview

**Vision-LoRA** refers to applying Low-Rank Adaptation (LoRA) to vision encoders and vision-specific components in multimodal large language models (MLLMs). It covers approaches that use LoRA to adapt vision-specific functions — including replacing vision encoders entirely with LoRA layers inside LLMs, and selectively targeting vision-related layers.

> *"Most current efficient multimodal fine-tuning methods are directly borrowed from LLMs, often neglecting the intrinsic differences of multimodal scenarios."*
> — [MokA Paper](https://arxiv.org/abs/2506.05191)

---

## Key Methods

### VoRA (Vision as LoRA) — ByteDance 2025

**Replaces** the vision encoder entirely with LoRA layers inside the LLM.

> *"VoRA applies trainable LoRA layers to LLMs, which encode the new modality (vision) while preserving the language knowledge of the original LLM by freezing its parameters."*
> — [VoRA GitHub](https://github.com/Hon-Wong/VoRA)

**Key Innovation:** High-rank LoRA (rank 1024) is critical — without LoRA, training causes catastrophic forgetting. LoRA layers can be merged after training, incurring near-zero computational overhead.

| Detail | Value |
|--------|-------|
| Paper | [arXiv:2503.20680](https://arxiv.org/abs/2503.20680) |
| GitHub | [Hon-Wong/VoRA](https://github.com/Hon-Wong/VoRA) |

---

### VFL-LoRA (Vision Function Layer LoRA) — NeurIPS 2025

Selectively applies LoRA to **specific decoder layers** that handle specific visual functions.

> *"When LoRA training is selectively applied to VFLs whose functions align with the training data, VFL-LoRA not only outperforms full-LoRA but also prevents out-of-domain function forgetting."*
> — [VFL-LoRA](https://github.com/ChengShiest/Vision-Function-Layer)

**Discovery:** Visual functions (counting, grounding, OCR, recognition) are handled by specific decoder layers (2-3 layers each), with a consistent hierarchical pattern across models.

**Results:**

| Method | Parameters | Out-of-Domain Gen |
|--------|-----------|-------------------|
| Full-LoRA | 309M | 74.3% |
| **VFL-LoRA** | **155M** | **75.0%** |

> *"50% fewer parameters with better out-of-domain generalization."*
> — [VFL-LoRA GitHub](https://github.com/ChengShiest/Vision-Function-Layer)

| Detail | Value |
|--------|-------|
| Paper | [arXiv:2509.24791](https://arxiv.org/abs/2509.24791) |
| GitHub | [ChengShiest/Vision-Function-Layer](https://github.com/ChengShiest/Vision-Function-Layer) |

---

### MokA (Multimodal Low-Rank Adaptation) — NeurIPS 2025

Modality-specific LoRA for multimodal scenarios.

> *"Modality-specific matrices (A) for unimodal adaptation + shared matrix (B) for cross-modal alignment + task-centric cross-attention."*
> — [MokA](https://gepu-lab.github.io/MokA)

**Results on visual-text benchmarks:**

| Method | MMEpercu Score |
|--------|---------------|
| Standard LoRA | 908.52 |
| **MokA** | **1292.37** |

| Detail | Value |
|--------|-------|
| Paper | [arXiv:2506.05191](https://arxiv.org/abs/2506.05191) |
| Project | [gepu-lab/MokA](https://gepu-lab.github.io/MokA) |

---

### PEFT-MLLM — ACL 2024

Empirical study on PEFT techniques for Multimodal LLMs.

> *"Evaluates different PEFT techniques across multiple vision-language tasks and datasets."*
> — [PEFT-MLLM GitHub](https://github.com/alenai97/peft-mllm)

| Detail | Value |
|--------|-------|
| GitHub | [alenai97/peft-mllm](https://github.com/alenai97/peft-mllm) |

---

## Practical Implementations

### Vision LoRA in Existing Frameworks

| Repository | Description |
|-----------|-------------|
| **VLM-Finetune** | Fine-tuning Qwen2-VL with `--vision_lora` option |
| **Llama3.2-Vision-Finetune** | Meta Llama3.2-Vision with LoRA including vision tower |
| **multimodal-peft-finetuning** | PaliGemma fine-tuning with QLoRA |
| **MOSS-VL** | Supports LoRA with vision encoder independently controllable |

---

## Common Vision Encoder LoRA Target Modules

| Module Type | Example Paths |
|-------------|--------------|
| Transformer layers | `vision_encoder.transformer.layers` |
| ViT blocks | `vision_encoder.blocks` |
| Resblocks | `visual.transformer.resblocks` |
| Encoder layers | `vision_model.encoder.layers` |
| Attention projections | `q_proj`, `k_proj`, `v_proj`, `out_proj` |
| MLP layers | `fc1`, `fc2` |

---

## Vision-LoRA vs Standard LoRA

| Aspect | Standard LoRA (LLaM) | Vision-LoRA |
|--------|---------------------|-------------|
| Target | Language model layers | Vision encoder / vision tower |
| Rank | 8-64 typical | 8-1024 (VoRA uses 1024) |
| Key Risk | Language forgetting | Vision function overlap |
| Merging | Mergeable | Mergeable (VoRA) |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2503.20680 — VoRA](https://arxiv.org/abs/2503.20680) |
| **Paper** | [arXiv:2509.24791 — VFL-LoRA](https://arxiv.org/abs/2509.24791) |
| **Paper** | [arXiv:2506.05191 — MokA](https://arxiv.org/abs/2506.05191) |
| **GitHub** | [Hon-Wong/VoRA](https://github.com/Hon-Wong/VoRA) |
| **GitHub** | [ChengShiest/Vision-Function-Layer](https://github.com/ChengShiest/Vision-Function-Layer) |
| **GitHub** | [alenai97/peft-mllm](https://github.com/alenai97/peft-mllm) |

---

## Related Topics

- [VLM.md](./VLM.md) — Vision-Language Model architectures
- [Projector-Alignment.md](./Projector-Alignment.md) — Visual projector/connector alignment
- [Vision-Encoding.md](./Vision-Encoding.md) — Vision encoding strategies
