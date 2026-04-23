# Unsloth

> *"Unsloth lets you train and run AI models on your own own local hardware. Train 500+ models ~2x faster with ~70% less VRAM (no accuracy loss)."* — [Unsloth Docs](https://unsloth.ai/docs)

## Contents

1. [Overview](#overview)
2. [Key Papers & Blog Posts](#key-papers--blog-posts)
3. [GitHub Repositories](#github-repositories)
4. [Technical Approach](#technical-approach)
5. [Benchmarks & Performance Claims](#benchmarks--performance-claims)
6. [Supported Models](#supported-models)
7. [QAT Integration](#qat-integration)
8. [Reinforcement Learning (GRPO)](#reinforcement-learning-grpo)
9. [Installation](#installation)
10. [Limitations & Debates](#limitations--debates)
11. [Related Topics](#related-topics)

---

## Overview

Unsloth is an open-source framework for memory-efficient and fast fine-tuning of LLMs. It uses custom Triton kernels, smart sequence packing, and 4-bit/16-bit LoRA/QLoRA to achieve 2× faster training with ~70% less VRAM compared to full fine-tuning.

| Metric | Value |
|--------|-------|
| **GitHub Stars** | ⭐ 56.5k |
| **Forks** | 4.7k |
| **Contributors** | 170 |
| **License** | Apache 2.0 (core) / AGPL-3.0 (Studio UI) |
| **Supported Models** | 500+ |
| **Speedup** | ~2× faster |
| **VRAM Reduction** | ~70% less |

> *"Unsloth wraps Transformers APIs and patches internal methods for speed. `FastLanguageModel.from_pretrained` loads config with AutoConfig.from_pretrained(). It then loads a base model with AutoModelForCausalLM.from_pretrained(). Before loading, Unsloth patches attention, decoder layer, and rotary embedding classes inside a Transformers model."* — [Hugging Face Docs](https://huggingface.co/docs/transformers/community_integrations/unsloth)

---

## Key Papers & Blog Posts

### Official Blog Posts

**3× Faster Training with Smart Packing**
> *"GPUs cannot process variable-length sequences, requiring padding with zeros. This causes waste... By combining multiple sequences into one tensor: Token Usage = 2×batchsize×L + 2×batchsize×S"* — [Unsloth Blog](https://unsloth.ai/docs/blog/3x-faster-training-packing)

**Quantization-Aware Training (QAT)**
> *"In collaboration with PyTorch, we're introducing QAT (Quantization-Aware Training) in Unsloth to enable trainable quantization that recovers as much accuracy as possible. QAT recovers up to 70% of lost accuracy."* — [Unsloth Docs](https://unsloth.ai/docs/blog/quantization-aware-training-qat)

**Reinforcement Learning GRPO**
> *"Due to Unsloth's unique weight sharing and custom kernels, Unsloth makes VLM RL 1.5–2× faster, use 90% less VRAM, and enables 10× longer context."* — [Unsloth Substack](https://unslothai.substack.com/p/unsloths-new-reinforcement-learning)

> *"We've enhanced the entire GRPO process, making it use 80% less VRAM than Hugging Face + FA2."* — [Unsloth Blog](https://unsloth.ai/blog/r1-reasoning)

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [unslothai/unsloth](https://github.com/unslothai/unsloth) | ⭐ 56.5k | Main framework — memory-efficient fine-tuning |
| [unslothai/unsloth-studio](https://github.com/unslothai/unsloth-studio) | — | Unified web UI for training and running open models |

---

## Technical Approach

### Custom Triton Kernels

> *"Unsloth now supports up to 5× faster (typically 3x) training with our new custom RoPE and MLP Triton kernels, plus our new smart auto packing."* — [Unsloth Blog](https://unsloth.ai/docs/blog/3x-faster-training-packing)

**Fused QK RoPE Kernel:**
- Merged Q and K into 1 fused Triton kernel
- 2.3× faster on longer context lengths, 1.9× faster on shorter
- RoPE is fully in-place, reducing GPU memory
- Eliminated all `clone()` and `contiguous transpose` operations

### Smart Packing (Padding-Free)

| Batch Size | Speedup (Packed) |
|------------|------------------|
| 2 | +40% |
| 4 | +70% |
| 8 | +2× |
| 16 | +2.5× |
| 32 | +3× |

### 4-bit / 16-bit LoRA Fine-tuning

> *"Instead of optimizing all model weights (e.g., 70B numbers for Llama 70B), thin matrices A and B are added to each weight layer, and only ~1% of weights are optimized."* — [Unsloth Docs](https://unsloth.ai/docs/get-started/fine-tuning-llms-guide)

- **LoRA** = 16-bit unquantized
- **QLoRA** = 4-bit quantization (75% memory savings)

---

## Benchmarks & Performance Claims

| Model | Speedup | VRAM Reduction |
|-------|---------|----------------|
| Qwen3.5 (4B) | 1.5× faster | 60% less |
| gpt-oss (20B) | 2× faster | 70% less |
| gpt-oss (20B) GRPO | 2× faster | 80% less |
| Llama 3.1 (8B) Alpaca | 2× faster | 70% less |
| Gemma 3 (4B) Vision | 1.7× faster | 60% less |

> *"Train 500+ models ~2x faster with ~70% less VRAM (no accuracy loss)."* — [Unsloth Docs](https://unsloth.ai/docs)

> *Praise from Andrej Karpathy* — widely cited in Unsloth marketing materials

---

## Supported Models

> *"Unsloth supports inference and training for **500+ models**: vision, TTS, embedding, RL."* — [Unsloth Docs](https://unsloth.ai/docs)

**Featured Model Families:**
- **Google Gemma:** Gemma 4, Gemma 3, Gemma 2, MedGemma, FunctionGemma
- **Qwen:** Qwen3, Qwen3.5, Qwen2.5-VL, QwQ-32B, Qwen3-Coder
- **Llama:** Llama 4, Llama 3.3, Llama 3.2, Llama 3.1, Llama 2, CodeLlama
- **DeepSeek:** DeepSeek-V3, DeepSeek-R1, distill variants
- **Mistral:** Mistral Small, Mistral Large 3, Mixtral, Pixtral
- **GLM:** GLM-5, GLM-4 series
- **Phi:** Phi-4, Phi-3

> Full model catalog: [unsloth.ai/docs/get-started/unsloth-model-catalog](https://unsloth.ai/docs/get-started/unsloth-model-catalog)

---

## QAT Integration

Unsloth implements Quantization-Aware Training (QAT) in collaboration with PyTorch:

> *"QAT recovers up to 70% of lost accuracy."* — [Unsloth Docs](https://unsloth.ai/docs/blog/quantization-aware-training-qat)

**Supported Schemes:**
- INT4
- FP8-INT4
- FP8-FP8
- INT8-INT4

See also: [QAT.md](./QAT.md) for broader QAT coverage.

---

## Reinforcement Learning (GRPO)

> *"We've enhanced the entire GRPO process, making it use 80% less VRAM than Hugging Face + FA2."* — [Unsloth Blog](https://unsloth.ai/blog/r1-reasoning)

> *"Due to Unsloth's unique weight sharing and custom kernels, Unsloth makes VLM RL 1.5–2× faster, use 90% less VRAM, and enables 10× longer context."* — [Unsloth Substack](https://unslothai.substack.com/p/unsloths-new-reinforcement-learning)

---

## Installation

**Linux / macOS / WSL:**
```bash
curl -fsSL https://unsloth.ai/install.sh | sh
```

**Windows PowerShell:**
```powershell
irm https://unsloth.ai/install.ps1 | iex
```

**Docker:**
```bash
docker pull unsloth/unsloth
```

---

## Limitations & Debates

- **Single-GPU optimized:** Unsloth excels on single-GPU setups; multi-GPU scaling less developed than FSDP2 or DeepSpeed
- **Model scope:** Primarily focused on causal LLMs; other architectures less tested
- **Commercial UI separation:** Studio UI is AGPL-3.0, not Apache 2.0 like the core library
- **PEFT only (default):** Full fine-tuning support is limited compared to frameworks like Axolotl

---

## Related Topics

- [Axolotl](./Axolotl.md) — YAML-driven multi-GPU fine-tuning with FSDP2 support
- [QAT](./QAT.md) — Quantization-Aware Training for recovering accuracy at low bit-widths
- [FSDP2](./FSDP2.md) — PyTorch's per-parameter sharding strategy for distributed fine-tuning
