# QLoRA: Quantized Low-Rank Adaptation

## 📌 Overview

**QLoRA** (Quantized Low-Rank Adaptation) combines 4-bit quantization with LoRA adapters to enable fine-tuning of large language models on consumer-grade GPUs. It reduces memory enough to finetune a 65B parameter model on a single 48GB GPU while preserving full 16-bit fine-tuning task performance.

> *"QLoRA backpropagates gradients through a frozen, 4-bit quantized pretrained language model into Low Rank Adapters (LoRA)."*
> — [arXiv:2305.14314](https://arxiv.org/abs/2305.14314)

---

## Three Key Innovations

### 1. 4-bit NormalFloat (NF4) Quantization

> *"A new data type that is information-theoretically optimal for normally distributed weights (which is the case for neural network weights). NF4 uses 16 quantization bins with equal proportions."*
> — [HuggingFace Blog](https://huggingface.co/blog/4bit-transformers-bitsandbytes)

Weights are normalized to [-1, 1] before quantization.

### 2. Double Quantization

> *"Reduces the average memory footprint by quantizing the quantization constants. Saves approximately **0.37 bits per parameter** — roughly **3 GB for an 8B model** and **26 GB for a 70B model**."*
> — [MaximF](https://www.maximofn.com/en/qlora/)

Quantization constants (FP32 per-block scaling factors) are themselves quantized to FP8.

### 3. Paged Optimizers

> *"Uses NVIDIA unified memory to handle memory spikes during gradient checkpointing — automatically pages data between CPU RAM and GPU, preventing out-of-memory errors."*
> — [arXiv:2305.14314](https://ar5iv.labs.arxiv.org/html/2305.14314)

---

## Memory Requirements

### QLoRA vs Full Fine-tuning

| Model Size | Full Fine-tuning | QLoRA | Min Consumer GPU |
|------------|-----------------|-------|------------------|
| 7B | 100-120 GB | **~5-6 GB** | RTX 3060 (8GB) |
| 13B | 200+ GB | **~10 GB** | RTX 4060 Ti (16GB) |
| 33B | 400+ GB | **~21 GB** | RTX 3090/4090 (24GB) |
| 65B | 780 GB | **~41-48 GB** | Single A100 80GB or 2x RTX 4090 |

> *"Fine-tuning needs 3–4× more VRAM than inference — full fine-tuning of 7B requires 80GB+; QLoRA reduces this to 8–10GB."*
> — [Craftrigs](https://craftrigs.com/guides/fine-tuning-7b-llm-consumer-gpu-unsloth-lora/)

### Sweet Spot Hardware

| GPU | VRAM | Use Case |
|-----|------|----------|
| **RTX 4090 24GB** | 24GB | 7B comfortable (batch 4-8), 13B (batch 2-4) |
| **RTX 3090 24GB** | 24GB | Same VRAM, slower compute, best value |
| **RTX 4080 Super** | 16GB | 7B only |
| **RTX 4060 Ti** | 16GB | Budget, 7B with patience |

---

## Benchmark Results: Guanaco Models

QLoRA fine-tuned LLaMA models on OASST1 dataset:

| Model | Size | Elo Rating | Vicuna Benchmark (% of ChatGPT) |
|-------|------|------------|----------------------------------|
| GPT-4 | — | 1348 | — |
| **Guanaco 65B** | 41 GB | 1022 | **99.3%** |
| **Guanaco 33B** | 21 GB | 992 | **97.8%** |
| Vicuna 13B | 26 GB | 974 | 94.9% |
| Guanaco 13B | 10 GB | 916 | 90.4% |
| Guanaco 7B | 6 GB | 879 | 87.0% |
| ChatGPT | — | 966 | 100% |

> *"Guanaco 65B achieves 99.3% of ChatGPT's performance while needing only 24 hours of finetuning on a single GPU."*
> — [arXiv:2305.14314](https://ar5iv.labs.arxiv.org/html/2305.14314)

---

## Critical: LoRA Must Target ALL Linear Layers

> *"Default LoRA (query/value only) does NOT match 16-bit performance. LoRA must be applied to **all linear transformer block layers**."*
> — [arXiv:2305.14314](https://ar5iv.labs.arxiv.org/html/2305.14314)

**Recommended config:**
- `target_modules="all-linear"` (PEFT)
- LoRA `r`: 16-64
- LoRA `α`: 2× rank
- LoRA dropout: 0.05 (7B/13B), 0.0 (33B/65B)

---

## Data Quality >> Dataset Size

> *"A 9K sample dataset (OASST1) outperformed a 450K sample dataset (FLAN v2) on chatbot performance."*
> — [arXiv:2305.14314](https://ar5iv.labs.arxiv.org/html/2305.14314)

QLoRA demonstrated that high-quality, task-specific datasets of modest size outperform larger, generic datasets for instruction tuning.

---

## Implementation

```python
from transformers import AutoModelForCausalLM, BitsAndBytesConfig
import torch

quantization_config = BitsAndBytesConfig(
    load_in_4bit=True,
    bnb_4bit_quant_type="nf4",
    bnb_4bit_compute_dtype=torch.bfloat16,
    bnb_4bit_use_double_quant=True,
)

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-2-7b-hf",
    quantization_config=quantization_config,
    device_map="auto"
)
```

> *"QLoRA reduces memory usage enough to finetune a 65B parameter model on a single 48GB GPU."*
> — [arXiv:2305.14314](https://arxiv.org/abs/2305.14314)

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2305.14314 — QLoRA](https://arxiv.org/abs/2305.14314) |
| **Paper (ar5iv)** | [QLoRA HTML version](https://ar5iv.labs.arxiv.org/html/2305.14314) |
| **Blog** | [HuggingFace — 4-bit transformers](https://huggingface.co/blog/4bit-transformers-bitsandbytes) |
| **Guide** | [Craftrigs — Fine-tuning 7B on consumer GPU](https://craftrigs.com/guides/fine-tuning-7b-llm-consumer-gpu-unsloth-lora/) |
| **Blog** | [The Decoder — Guanaco](https://the-decoder.com/guanaco-is-a-chatgpt-competitor-trained-on-a-single-gpu-in-one-day/) |

---

## Related Topics

- [LoRA](./LoRA.md) — Base LoRA mechanism
- [DoRA](./DoRA.md) — Weight-Decomposed LoRA
