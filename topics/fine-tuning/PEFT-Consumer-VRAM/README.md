# PEFT-Consumer-VRAM

## 📌 Overview

Parameter-Efficient Fine-Tuning (PEFT) methods designed to run on consumer GPUs with limited VRAM. Covers LoRA, QLoRA, and DoRA — their memory requirements, hyperparameter guidance, and consumer GPU benchmarks.

> *"Fine-tuning needs 3–4× more VRAM than inference — full fine-tuning of 7B requires 80GB+; QLoRA reduces this to 8–10GB."*
> — [Craftrigs](https://craftrigs.com/guides/fine-tuning-7b-llm-consumer-gpu-unsloth-lora/)

---

## Topics

| Topic | Description |
|-------|-------------|
| [LoRA](./LoRA.md) | Low-Rank Adaptation — base mechanism, hyperparameters, rsLoRA |
| [QLoRA](./QLoRA.md) | Quantized LoRA — NF4, double quantization, Guanaco benchmarks |
| [DoRA](./DoRA.md) | Weight-Decomposed LoRA — magnitude + direction separation |

---

## VRAM Comparison Table

| Method | 7B | 13B | 33B | 65B | 70B |
|--------|-----|-----|-----|-----|-----|
| Full Fine-tune (FP16) | ~70GB | ~125GB | ~350GB | ~672GB | ~672GB |
| **LoRA (FP16)** | **~15GB** | **~28GB** | **~73GB** | **~146GB** | ~146GB |
| **QLoRA (8-bit)** | **~9GB** | **~17GB** | **~40GB** | **~88GB** | ~88GB |
| **QLoRA (4-bit)** | **~5-6GB** | **~10GB** | **~21GB** | **~41-48GB** | ~46GB |

---

## Consumer GPU Benchmarks

| GPU | VRAM | 7B LoRA | 7B QLoRA | Notes |
|-----|------|---------|----------|-------|
| RTX 4090 | 24GB | ✅ | ✅ | Comfortable, batch 4-8 |
| RTX 3090 | 24GB | ✅ | ✅ | Same VRAM, slower compute |
| RTX 4080 Super | 16GB | ✅ | ✅ | 7B only |
| RTX 4060 Ti | 16GB | ⚠️ | ✅ | Budget option |
| RTX 4060 | 8GB | ❌ | ✅ | ~628 tok/s on 1.5B |

> *"fp16 preferred over bf16 on consumer GPUs (bf16 degrades efficiency ~43%)."*
> — [arXiv:2509.12229](https://arxiv.org/html/2509.12229v1)

---

## Key Findings

### LoRA Hyperparameters

> *"α/r ratio is what matters, not absolute values."*
> — [Sebastian Raschka](https://sebastianraschka.com/blog/2023/llm-finetuning-lora.html)

| Rank | Sweet Spot |
|------|-----------|
| 4-8 | Simple classification |
| **16-32** | **Instruction tuning** |
| 64-128 | Complex/multilingual |

### rsLoRA Fixes Saturation

> *"Traditional LoRA fine-tuning saturates at very low ranks (4-32). rsLoRA r=256 achieved 8.09 MT-Bench vs standard LoRA r=256 at 7.96."*
> — [HuggingFace Blog](https://huggingface.co/blog/damjan-k/rslora)

### QLoRA Critical Setting

> *"Default LoRA (query/value only) does NOT match 16-bit performance. LoRA must be applied to all linear transformer block layers."*
> — [arXiv:2305.14314](https://ar5iv.labs.arxiv.org/html/2305.14314)

### Data Quality >> Dataset Size

> *"A 9K sample dataset (OASST1) outperformed a 450K sample dataset (FLAN v2) on chatbot performance."*
> — [arXiv:2305.14314](https://ar5iv.labs.arxiv.org/html/2305.14314)

---

## DoRA vs LoRA

| Aspect | LoRA | DoRA |
|--------|------|------|
| VRAM during training | Baseline | ~1.5-2× higher |
| Training time/step | Baseline | ~2-3× slower |
| Performance (rank=r) | Baseline | Better than LoRA rank=2r |
| Zero inference overhead | ✅ | ✅ |

> *"QDoRA closes the gap between full fine-tuning and quantized PEFT."*
> — [Answer.AI](https://www.answer.ai/posts/2024-04-26-fsdp-qdora-llama3.html)

---

## Related Topics

- [Acceleration-Frameworks](../Acceleration-Frameworks/) — Fine-tuning frameworks (Unsloth, Axolotl, QAT, FSDP2)
- [Modular-MOE-Merging](../Modular-MOE-Merging/) — Expert merging for MoE models
- [SSM-Hybrid-Tuning](../SSM-Hybrid-Tuning/) — SSM hybrid architectures
