# LoRA: Low-Rank Adaptation

## 📌 Overview

**LoRA** (Low-Rank Adaptation) is a parameter-efficient fine-tuning (PEFT) technique that decomposes weight updates (ΔW) into low-rank matrices instead of updating full model weights. It freezes original weights and only trains adapter matrices W_A ∈ ℝ^(A×r) and W_B ∈ ℝ^(r×B).

> *"LoRA is based on the concept that **pretrained LLMs have a low 'intrinsic dimension'** when adapted to new tasks."*
> — [Sebastian Raschka, PhD](https://sebastianraschka.com/blog/2023/llm-finetuning-lora.html)

---

## Core Mechanism

### Weight Update Decomposition

| Original | LoRA Decomposed |
|----------|----------------|
| Update full W ∈ ℝ^(A×B) | Freeze W, train W_A ∈ ℝ^(A×r) + W_B ∈ ℝ^(r×B) |
| A×B parameters | (A+B)×r parameters |

**Example:** 768×768 matrix (589,824 params) with r=32 → 49,152 trainable params (**91% reduction**).

> *"GPT-3 checkpoint: 1.2 TB → 35 MB (**34,000× reduction**)."*
> — [Sebastian Raschka](https://sebastianraschka.com/blog/2023/llm-finetuning-lora.html)

---

## VRAM Requirements

### Fine-tuning VRAM Comparison

| Method | 7B | 13B | 70B | Notes |
|--------|-----|-----|-----|-------|
| Full fine-tuning (FP16) | ~70GB | ~125GB | ~672GB | Multiple GPUs required |
| **LoRA (FP16)** | **~15GB** | **~28GB** | **~146GB** | Single consumer GPU feasible |
| **QLoRA (8-bit)** | **~9GB** | **~17GB** | **~88GB** | Good for consumer GPUs |
| **QLoRA (4-bit)** | **~5GB** | **~9GB** | **~46GB** | Most memory-efficient |

> *"A 7B model requiring 14GB for weights needs approximately **28GB total** for LoRA fine-tuning (weights + gradients + optimizer states for adapters only), versus 100+ GB for full fine-tuning."*
> — [Introl](https://introl.com/blog/fine-tuning-infrastructure-lora-qlora-peft-scale-guide-2025)

### Consumer GPU Benchmarks

| GPU | VRAM | Best For |
|-----|------|----------|
| **RTX 4090** | 24GB | 7B comfortable, 13B viable |
| **RTX 3090** | 24GB | Strong alternative |
| **RTX 4080** | 16GB | 7B only |
| **RTX 4060** | 8GB | ~628 tok/s with QLoRA on 1.5B model |

> *"PagedAdamW yields **25% throughput improvement** on consumer GPUs."*
> — [arXiv:2509.12229](https://arxiv.org/html/2509.12229v1)

> *"fp16 preferred over bf16 on consumer GPUs (bf16 degrades efficiency ~43%)."*
> — [arXiv:2509.12229](https://arxiv.org/html/2509.12229v1)

---

## Hyperparameters

### Rank (r) Selection

| Rank Range | Best For |
|------------|----------|
| 4-8 | Simple classification, sentiment analysis |
| **16-32** | **Sweet spot for instruction tuning** |
| 64-128 | Complex tasks, multilingual adaptation |
| 256+ | Use rsLoRA instead |

### Alpha (α)

- Scaling factor: **α/r** determines adapter contribution strength
- Original paper: α = 2r (e.g., r=8, α=16)
- QLoRA paper: α at 50-25% of rank
- **Key insight: α/r ratio is what matters, not absolute values**

> *"Traditional LoRA fine-tuning **saturates at very low ranks** (4-32). Performance doesn't improve with increasing rank... This saturation is caused by **stunted learning** stemming from LoRA's scaling factor γ_r = α/r."*
> — [HuggingFace Blog](https://huggingface.co/blog/damjan-k/rslora)

### Target Modules

| Strategy | Parameters | Use Case |
|----------|-----------|----------|
| Q, V only | ~0.1% of model | Most tasks - start here |
| All attention (Q,K,V,O) | ~0.2% | Modify key computations |
| Attention + FFN | ~1% | Domain adaptation |

---

## rsLoRA (Rank-Stabilized LoRA)

> *"Solution: Divide adapters by √r instead of r to fix the saturation problem."*
> — [HuggingFace Blog](https://huggingface.co/blog/damjan-k/rslora)

```python
# PEFT implementation
LoraConfig(..., use_rslora=True)
```

**Results:**

| Method | MT-Bench Score | Rank |
|--------|---------------|------|
| rsLoRA r=256 | **8.09** | 256 |
| Standard LoRA r=256 | 7.96 | 256 |

> *"rsLoRA nearly doubles improvement over base model with negligible extra compute."*
> — [HuggingFace Blog](https://huggingface.co/blog/damjan-k/rslora)

---

## Practical Consumer GPU Guidance

| Model Size | Min VRAM | Recommended |
|------------|----------|-------------|
| 7B | 12-16GB | RTX 3060+ |
| 13B | 16-24GB | RTX 3090/4090 |
| 70B | 24GB+ (Q4) | Multi-GPU or A100 |

> *"Gradient checkpointing reduces VRAM 30-50% with 17-22% performance loss."*
> — [Modal VRAM Guide](https://modal.com/blog/how-much-vram-need-fine-tuning)

---

## Sources

| Type | Citation |
|------|----------|
| **Blog** | [Sebastian Raschka — LoRA Explained](https://sebastianraschka.com/blog/2023/llm-finetuning-lora.html) |
| **Paper** | [arXiv:2509.12229 — RTX 4060 Case Study](https://arxiv.org/html/2509.12229v1) |
| **Blog** | [HuggingFace — rsLoRA](https://huggingface.co/blog/damjan-k/rslora) |
| **Guide** | [Modal — VRAM Fine-tuning Guide](https://modal.com/blog/how-much-vram-need-fine-tuning) |
| **Guide** | [Introl — LoRA/PEFT at Scale](https://introl.com/blog/fine-tuning-infrastructure-lora-qlora-peft-scale-guide-2025) |

---

## Related Topics

- [QLoRA](./QLoRA.md) — Quantized LoRA for even lower VRAM
- [DoRA](./DoRA.md) — Weight-Decomposed LoRA alternative
