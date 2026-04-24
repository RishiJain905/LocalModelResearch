# Axolotl

> *"Axolotl is a free and open-source tool designed to streamline post-training and fine-tuning for the latest large language models (LLMs)."* — [Axolotl Docs](https://docs.axolotl.ai/)

## Contents

1. [Overview](#overview)
2. [Key Sources](#key-sources)
3. [GitHub Repository](#github-repository)
4. [Supported Models](#supported-models)
5. [Fine-tuning Methods](#fine-tuning-methods)
6. [Training Backends & Optimizations](#training-backends--optimizations)
7. [Performance Benchmarks](#performance-benchmarks)
8. [Configuration Example (FSDP2)](#configuration-example-fsdp2)
9. [Notable Users](#notable-users)
10. [Limitations & Debates](#limitations--debates)
11. [Related Topics](#related-topics)

---

## Overview

Axolotl is a YAML-configuration-driven fine-tuning framework that supports a wide range of LLMs, training methods, and distributed backends. It is community-driven and designed for both single-GPU and multi-GPU/multi-node training.

| Metric | Value |
|--------|-------|
| **License** | Apache 2.0 |
| **Contributors** | 170+ |
| **Discord Members** | 500+ |
| **Approach** | YAML-first, single config file |
| **Notable Users** | Teknium/Nous Research, Modal, Replicate, OpenPipe |

> *"Axolotl ships with built-in documentation optimized for AI coding agents... Docs bundled with pip package (no repo clone needed)"* — [Axolotl Docs](https://docs.axolotl.ai/)

---

## Key Sources

| Resource | URL |
|----------|-----|
| **GitHub** | [axolotl-ai-cloud/axolotl](https://github.com/axolotl-ai-cloud/axolotl) |
| **Documentation** | [docs.axolotl.ai](https://docs.axolotl.ai/) |
| **Blog (Substack)** | [axolotlai.substack.com](https://axolotlai.substack.com/) |
| **Cookbook** | [axolotl-ai-cloud/axolotl-cookbook](https://github.com/axolotl-ai-cloud/axolotl-cookbook) |

---

## GitHub Repository

| Repository | Description |
|-----------|-------------|
| [axolotl-ai-cloud/axolotl](https://github.com/axolotl-ai-cloud/axolotl) | Main framework — YAML-driven fine-tuning |
| [axolotl-ai-cloud/axolotl-cookbook](https://github.com/axolotl-ai-cloud/axolotl-cookbook) | Community training recipes and examples |

---

## Supported Models

> *"GPT-OSS, LLaMA, Mistral, Mixtral, Pythia, and many more HF Hub models"* — [Axolotl Docs](https://docs.axolotl.ai/)

**Mistral Family:**
- Ministral 3, Magistral, Mistral Small 3.1/3.2, Mistral 7B, Voxtral

**Llama:**
- Llama 4, Llama 2

**Qwen:**
- Qwen 3, Qwen 3 Next

**Vision:**
- InternVL 3.5, Ministral 3 Vision, Magistral Vision

**Other:**
- Kimi Linear, Plano Orchestrator, OLMo 3, Arcee AFM, Gemma 3n, Devstral, Apertus, GPT-OSS, Seed-OSS, Phi, SmolVLM 2, Granite 4, Liquid Foundation Models 2, Hunyuan, Jamba, Orpheus

---

## Fine-tuning Methods

| Method | Description |
|--------|-------------|
| **Full Fine-tuning** | Update all model weights |
| **LoRA / QLoRA** | Low-rank adapter fine-tuning with 4-bit quantization |
| **GPTQ** | Post-training quantization |
| **QAT** | Quantization-Aware Training |
| **Preference Tuning** | DPO, IPO, KTO, ORPO |
| **RL** | GRPO, GDPO (via TRL library) |
| **Reward Modelling** | RM/PRM |

> *"Fine-tuning Methods: Full fine-tuning, LoRA, QLoRA, GPTQ, QAT, Preference Tuning (DPO, IPO, KTO, ORPO), RL (GRPO, GDPO), Reward Modelling (RM/PRM)"* — [Axolotl Docs](https://docs.axolotl.ai/)

---

## Training Backends & Optimizations

| Category | Details |
|----------|---------|
| **Distributed Backends** | FSDP1, FSDP2, DeepSpeed ZeRO (1-3), Accelerate, Torchrun, Ray Train |
| **Performance Optimizations** | Multipacking, Flash Attention 2/3/4, Xformers, Flex Attention, SageAttention, Liger Kernel, Cut Cross Entropy, ScatterMoE, Sequence Parallelism |
| **Quantization** | 4-bit NF4/QLoRA, 8-bit, GPTQ, QAT, MoE Expert Quantization |
| **RLHF** | DPO, IPO, ORPO, KTO, GRPO, GDPO, SimPO, EBFT, NeMo Gym (via TRL) |
| **Multimodal** | LLaMA-Vision, Qwen2-VL, Pixtral, LLaVA, SmolVLM2, GLM-4.6V, InternVL 3.5, Gemma 3n, Voxtral (audio) |

---

## Performance Benchmarks

> *"Full fine-tuning of Llama-3.1 70B across 8× H100 GPUs with Axolotl: **18 hours** on 50K examples"* — [Spheron Blog](https://blog.spheron.network/fine-tuning-on-multiple-gpus-with-axolotl)

> *"QLoRA fine-tuning of 70B with Axolotl: **6.1 hours** (vs Unsloth's 4.2 hours on single GPU, but Axolotl scales better multi-GPU)"* — [Community Comparison](https://blog.spheron.network/fine-tuning-on-multiple-gpus-with-axolotl)

> *"Axolotl claims **3-5× faster** training vs alternatives with new product updates"* — [Axolotl Blog](https://axolotlai.substack.com/)

**Cookbook Reference Training:**
- Llama 3.1 405B fine-tuning on Lambda 1-Click Cluster — [Axolotl Cookbook](https://github.com/axolotl-ai-cloud/axolotl-cookbook)

---

## Configuration Example (FSDP2)

```yaml
fsdp_version: 2
fsdp_config:
  offload_params: false
  cpu_ram_efficient_loading: true
  auto_wrap_policy: TRANSFORMER_BASED_WRAP
  transformer_layer_cls_to_wrap: Qwen3DecoderLayer
  state_dict_type: FULL_STATE_DICT
  reshard_after_forward: true
```

> *"FSDP2 is recommended for new users. FSDP1 is deprecated and will be removed in a future release."* — [Axolotl Multi-GPU Docs](https://docs.axolotl.ai/docs/multi-gpu.html)

---

## Notable Users

> *"Axolotl is used by research companies, model builders, Gen AI platforms, and practitioners like **Teknium/Nous Research, Modal, Replicate, OpenPipe**"* — [axolotl.ai](https://axolotl.ai)

---

## Limitations & Debates

- **YAML complexity:** Deep configurations can become verbose for complex multi-GPU setups
- **FSDP2 still maturing:** Some model-specific issues (e.g., Qwen-3 FSDP2 failures reported in [GitHub Issue #3056](https://github.com/axolotl-ai-cloud/axolotl/issues/3056))
- **Documentation gaps:** Some advanced features lack detailed examples
- **Multi-node:** More complex setup than single-node compared to managed solutions

---

## Related Topics

- [Unsloth](./Unsloth.md) — Single-GPU optimized memory-efficient fine-tuning
- [QAT](./QAT.md) — Quantization-Aware Training for low bit-width accuracy recovery
- [FSDP2](./FSDP2.md) — PyTorch's recommended distributed sharding strategy
