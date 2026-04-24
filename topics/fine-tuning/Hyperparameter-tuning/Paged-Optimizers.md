# Paged Optimizers

## 📌 Overview

| Field | Value |
|-------|-------|
| **What** | Memory-management technique using CUDA unified memory to page optimizer states to CPU RAM |
| **Primary Use** | Train LLMs that exceed GPU VRAM on a single GPU |
| **Key Benefit** | 65B model: >780GB → <48GB GPU memory (94% reduction) |
| **Origin** | QLoRA paper (arXiv:2305.14314) by Tim Dettmers et al. |
| **Related Concept** | vLLM PagedAttention (SOSP 2023 Best Paper) — same paging idea applied to KV cache |

> "Paged optimizers work like regular CPU paging, which means that it only becomes active if you run out of GPU memory."

— [bitsandbytes documentation](https://huggingface.co/docs/bitsandbytes/explanations/optimizers)

---

## Definitions

### The Memory Problem

AdamW stores gradient, first moment, second moment = **12 bytes per parameter** at FP32.

For an 8B model: ~64 GB for optimizer states alone.

> "For each model parameter, AdamW requires the storage of two additional optimizer states in memory, each typically in float32 format. This translates to an extra 8 bytes per parameter, meaning that for a model with 8 billion parameters, such as Llama 3.1, roughly 64 GB of memory goes solely toward managing optimizer states."

— [Kaitchup Blog](https://kaitchup.substack.com/p/fine-tuning-llms-with-32-bit-8-bit)

### How Paged Optimizers Work

1. **Quantize**: Optimizer states quantized from FP32 → 8-bit
2. **Offload**: Bulk of optimizer states reside in pinned CPU memory
3. **Page In**: Only states needed for current mini-batch are transferred from CPU RAM → GPU VRAM
4. **Compute**: Optimizer update runs on GPU with paged-in states
5. **Write Back**: Updated states written back (paged out) to CPU RAM

> "Uses NVIDIA unified memory to manage memory spikes from gradient checkpointing, preventing out-of-memory errors during long-sequence training."

— [QLoRA paper, arXiv:2305.14314](https://arxiv.org/abs/2305.14314)

---

## Key Sources

### QLoRA Paper (arXiv:2305.14314)

| Field | Value |
|-------|-------|
| Paper | [QLoRA: Efficient Finetuning of Quantized LLMs](https://arxiv.org/abs/2305.14314) |
| Authors | Tim Dettmers, Artidoro Pagnoni, Ari Holtzman, Luke Zettlemoyer (UWash) |
| Date | 2023 |

> "We present QLORA, an efficient finetuning approach that reduces memory usage enough to finetune a 65B parameter model on a single 48GB GPU while preserving full 16-bit finetuning task performance."

> "QLORA reduces the average memory requirements of finetuning a 65B parameter model from >780GB of GPU memory to <48GB without degrading the runtime or predictive performance compared to a 16-bit fully finetuned baseline."

### vLLM PagedAttention (SOSP 2023 Best Paper)

| Field | Value |
|-------|-------|
| Paper | [arXiv:2309.06180](https://arxiv.org/abs/2309.06180) |
| Authors | Woosuk Kwon, Zhuohan Li et al. (Berkeley, Stanford, UCSD) |
| Venue | SOSP 2023 — Best Paper Award |

> "vLLM improves the throughput of popular LLMs by 2-4x with the same level of latency compared to the state-of-the-art systems."

> "Existing systems waste 60%-80% of the KV-Cache (LLM memory), whereas vLLM achieves near-optimal memory usage with a mere waste of under 4%."

**Note**: PagedAttention targets inference (KV cache). The vLLM paper is the origin of the "paged" terminology for LLM memory optimization. bitsandbytes applies the same concept to training optimizer states.

### bitsandbytes Documentation

| Field | Value |
|-------|-------|
| URL | [huggingface.co/docs/bitsandbytes/explanations/optimizers](https://huggingface.co/docs/bitsandbytes/explanations/optimizers) |

> "Paged optimizers are built on top of the unified memory feature of CUDA. Unified memory provides a single memory space the GPU and CPU can easily access. While this feature is not supported by PyTorch, it has been added to bitsandbytes."

> "The unified memory feature is less efficient than regular asynchronous memory transfers, and you usually won't be able to get full PCIe memory bandwidth utilization."

### Consumer GPU Benchmarks (arXiv:2509.12229v1)

| URL | [arXiv:2509.12229v1](https://arxiv.org/html/2509.12229v1) |
|------|------|

> "BitsAndBytes, combined with paged optimizers, emerges as the most effective backend for fine-tuning on GPUs with <12 GB VRAM."

> "bf16 degrades efficiency on consumer hardware: bf16 was substantially slower (360 tok/s) than fp16 and increased energy per token, contrary to datacenter trends."

---

## Benchmark Results

### QLoRA Memory Reduction

| Model Size | Full Fine-tune | QLoRA + Paged Optimizer |
|-----------|--------------|------------------------|
| 7B | ~56GB | **6.9GB** |
| 13B | ~104GB | **11.3GB** |
| 33B | ~264GB | **24.7GB** |
| 65B | >780GB | **<48GB** |

### Consumer GPU Throughput (RTX 4060 8GB, Qwen2.5-1.5B-Instruct)

| Optimizer | Prec. | Batch | Seq Len | Tokens/sec | VRAM Peak |
|-----------|--------|-------|---------|-----------|----------|
| AdamW (torch) | fp16 | 1 | 512 | 500.3 | 6,234 MB |
| **PagedAdamW (8-bit)** | fp16 | 2 | 2048 | **628.1** | 8,062 MB |
| PagedAdamW (8-bit) | bf16 | 2 | 1024 | 360.2 | 7,949 MB |

**PagedAdamW achieves +25.6% throughput vs baseline on consumer hardware.**

### vLLM PagedAttention (Inference)

| Metric | Existing Systems | vLLM |
|--------|-----------------|------|
| Throughput improvement | baseline | **2-4x** |
| KV cache waste | 60–80% | **<4%** |

---

## Implementation

### bitsandbytes Classes

| Class | Description |
|-------|-------------|
| `PagedAdamW8bit` | 8-bit quantized + paged |
| `PagedAdamW32bit` | 32-bit + paged |
| `PagedAdamW` | Base paged AdamW |
| `AdamW8bit` | 8-bit quantized, non-paged |
| `AdamW32bit` | 32-bit, non-paged |

### Hugging Face Transformers

```python
TrainingArguments(
    optim="paged_adamw_8bit"
)
```

### QLoRA CLI

```bash
python qlora.py --model_name_or_path <model> --optim paged_adamw_32bit
```

---

## Important Constraints

### CUDA Unified Memory Required

> "Paged optimizers are built on top of the unified memory feature of CUDA... This feature is not supported by PyTorch and we added it to bitsandbytes."

— Tim Dettmers, GitHub Issue #962

**Platform support**: Only NVIDIA GPUs with CUDA unified memory. Not supported on AMD GPUs, Apple Silicon (MPS), or other backends.

### PCIe Bandwidth Overhead

> "For example, if you evict 1 GB of memory per forward-backward-optimizer loop, then you can expect about 50% of the PCIe bandwidth as time in the best case."

— Tim Dettmers, GitHub Issue #962

### Zero Overhead When Memory Fits

> "Compared to CPU offloading, a paged optimizer has zero overhead if all the memory fits onto the device and only some overhead if some of memory needs to be evicted."

— bitsandbytes docs

### bf16 Degradation on Consumer Hardware

On RTX 4060: bf16 PagedAdamW = 360 tok/s vs fp16 = 628 tok/s. Significant degradation without training stability benefits on consumer GPUs.

---

## Documentation Gaps

> "Hi Tim, I have just accidentally discovered that you added paged optimizers to this library - so very awesome! But there is absolutely zero documentation - would you consider adding at least a basic doc entry on these?"

— @stas00, GitHub Issue #962

Tim Dettmers never published a standalone paper on paged optimizers — the feature was only documented via the QLoRA paper and GitHub issues.

---

## Summary Data Points

| Metric | Value | Source |
|--------|-------|--------|
| 65B memory reduction | >780GB → <48GB (94%) | QLoRA paper |
| vLLM throughput improvement | 2–4x | SOSP 2023 |
| vLLM KV cache waste | <4% (vs 60–80%) | vLLM blog |
| PCIe bandwidth during paging | ~50% | bitsandbytes docs |
| PagedAdamW speedup (consumer) | +25.6% vs AdamW | arXiv:2509.12229v1 |
| Memory per param (AdamW FP32) | 8 bytes extra | Kaitchup blog |
| Paged optimizer activation | Only when GPU OOM | bitsandbytes docs |

---

## Related Topics

- [rsLoRA.md](./rsLoRA.md) — High-rank training enabled by paged optimizers
- [Rank-Alpha-Scaling.md](./Rank-Alpha-Scaling.md) — Hyperparameter scaling theory
- [QAT.md](../Acceleration-Frameworks/QAT.md) — Quantization-aware training

---

## References

- [QLoRA (arXiv:2305.14314)](https://arxiv.org/abs/2305.14314)
- [vLLM PagedAttention (arXiv:2309.06180)](https://arxiv.org/abs/2309.06180)
- [bitsandbytes optimizers docs](https://huggingface.co/docs/bitsandbytes/explanations/optimizers)
- [bitsandbytes GitHub #962](https://github.com/bitsandbytes-foundation/bitsandbytes/issues/962)
- [QLoRA repo](https://github.com/artidoro/qlora)
- [Consumer GPU benchmarks (arXiv:2509.12229v1)](https://arxiv.org/html/2509.12229v1)
- [vLLM blog](https://blog.vllm.ai/2023/06/20/vllm.html)
