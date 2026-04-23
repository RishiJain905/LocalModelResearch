# DoRA: Weight-Decomposed Low-Rank Adaptation

## 📌 Overview

**DoRA** (Weight-Decomposed Low-Rank Adaptation) decomposes pre-trained weights into two components — magnitude (m) and direction — adapting them separately. It fine-tunes the magnitude directly while using LoRA for direction, mimicking full fine-tuning's independent magnitude/direction optimization with better parameter efficiency than standard LoRA.

> *"DoRA uses LoRA for the directional component while introducing a learnable magnitude vector, achieving better learning capacity than LoRA."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-dora-a-high-performing-alternative-to-lora-for-fine-tuning/)

---

## Core Mechanism

### Weight Decomposition

| Component | Adaptation Method |
|-----------|------------------|
| **Magnitude (m)** | Fine-tuned directly |
| **Direction (ΔW)** | Adapted via LoRA |

This decomposition mirrors how full fine-tuning works — DoRA treats them independently rather than coupling them through a single scaling factor (α/r).

---

## Performance vs LoRA

### Common Sense Reasoning (Llama models)

| Model | LoRA | DoRA | Improvement |
|-------|------|------|-------------|
| Llama 7B | baseline | **+3.7** | +3.7 |
| Llama 13B | baseline | **+1.0** | +1.0 |
| Llama 2 7B | baseline | **+2.9** | +2.9 |
| Llama 3 8B | baseline | **+4.4** | +4.4 |

### Vision-Language Tasks

| Task | VL-BART (LoRA) | VL-BART (DoRA) | Improvement |
|------|----------------|----------------|-------------|
| Image-Text Understanding | baseline | **+0.9** | +0.9 |
| Video-Text Understanding | baseline | **+1.9** | +1.9 |
| Visual Instruction Tuning | baseline | **+0.6** | +0.6 |

> *"DoRA with rank=r often outperforms LoRA with rank=2r."*
> — [GitHub Issue #1692](https://github.com/huggingface/peft/issues/1692)

---

## Consumer VRAM Considerations

### Memory Overhead vs LoRA

> *"DoRA uses more VRAM than LoRA during training — approximately 2× activation memory in some cases."*
> — [GitHub PEFT](https://github.com/huggingface/peft/issues/1692)

**Profiling on ResNet-50:**

| Method | Total VRAM | Activation Memory |
|--------|-----------|-------------------|
| Full Fine-tune (FFT) | 1682 MiB | — |
| LoRA | 2725 MiB | 2618 MiB |
| **DoRA** | **4088 MiB** | **3963 MiB** |

### FP32 Upcasting Issue

> *"magnitude is in fp32, so the input vector x is upcast to fp32 when it gets returned as result_dora... causes OOM on large models."*
> — [GitHub Issue #1692](https://github.com/huggingface/peft/issues/1692)

> *"If both MLP and attention layers are added to target_modules, that fp32 output then causes the dequantized weight matrix to get upcast to fp32."*
> — [GitHub Issue #1692](https://github.com/huggingface/peft/issues/1692)

**Workaround:** Custom Triton kernel reduced VRAM by ~1,974 MB and training time by 20%.

---

## QDoRA: Memory-Efficient Variant

Combines DoRA with 4-bit NF4 quantization (same as QLoRA).

### Results on Orca-Math

**10k samples:**

| Method | Exact Match |
|--------|-------------|
| Full Fine-tune | 0.182 |
| QLoRA | 0.098 |
| **QDoRA** | **0.176** |

**100k samples:**

| Method | Exact Match |
|--------|-------------|
| Full Fine-tune | 0.26 |
| QLoRA | 0.118 |
| **QDoRA** | **0.312** |

> *"QDoRA closes the gap between full fine-tuning and quantized PEFT."*
> — [Answer.AI Blog](https://www.answer.ai/posts/2024-04-26-fsdp-qdora-llama3.html)

---

## Training Time Overhead

> *"DoRA is ~2-3× slower per step than LoRA due to extra L2 normalization and magnitude computations."*
> — [HuggingFace Forums](https://discuss.huggingface.co/)

> *"Configuration with lora_dropout=0.05 slows DoRA further vs 0.0."*
> — [HuggingFace Forums](https://discuss.huggingface.co/)

---

## Implementation

```python
from peft import LoraConfig, get_peft_model

config = LoraConfig(
    use_dora=True,  # Enable DoRA
    r=8,
    lora_alpha=16,
    target_modules=["q_proj", "k_proj", "v_proj", "o_proj", 
                    "gate_proj", "up_proj", "down_proj"],
    lora_dropout=0.05
)
```

> *"lora_dropout=0.05 vs 0.0 affects DoRA optimization path (0.0 is faster)."*
> — [GitHub PEFT Issues](https://github.com/huggingface/peft/issues/1692)

---

## Zero Inference Overhead

> *"DoRA can be merged back into pretrained weights before deployment — no inference overhead."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-dora-a-high-performing-alternative-to-lora-for-fine-tuning/)

---

## Key Trade-offs

| Aspect | LoRA | DoRA |
|--------|------|------|
| **VRAM during training** | Lower | ~1.5-2× higher |
| **Training time/step** | Baseline | ~2-3× slower |
| **Performance (rank=r)** | Baseline | Better than LoRA rank=2r |
| **Inference overhead** | None | None (mergeable) |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2402.09353 — DoRA (ICML 2024 Oral)](https://arxiv.org/abs/2402.09353) |
| **Blog** | [NVIDIA Developer Blog — DoRA](https://developer.nvidia.com/blog/introducing-dora-a-high-performing-alternative-to-lora-for-fine-tuning/) |
| **GitHub** | [NVlabs/DoRA](https://github.com/NVlabs/DoRA) |
| **Blog** | [Answer.AI — FSDP QDoRA](https://www.answer.ai/posts/2024-04-26-fsdp-qdora-llama3.html) |
| **GitHub** | [huggingface/peft Issue #1692](https://github.com/huggingface/peft/issues/1692) |

---

## Related Topics

- [LoRA](./LoRA.md) — Base LoRA mechanism
- [QLoRA](./QLoRA.md) — Quantized LoRA for low VRAM
