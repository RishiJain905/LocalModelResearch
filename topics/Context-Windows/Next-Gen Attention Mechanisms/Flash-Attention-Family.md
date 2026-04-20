# Flash Attention Family

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **GitHub:** [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) ⭐23.4K · [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) ⭐12.6K · [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) ⭐5.4K

---

## Overview

Flash Attention is a family of **IO-aware exact attention** algorithms that are both faster and more memory-efficient than standard attention. All versions compute exact attention (no approximation) while achieving O(N) memory instead of O(N²).

---

## Flash Attention (v1)

### "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"

> **arXiv:** [2205.14135](https://arxiv.org/abs/2205.14135) | **GitHub:** [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) ⭐23.4K

> *"Reduces attention complexity from O(N²) memory to O(N) and is 2-4× faster."* — [Dao et al., 2022]

### Key Innovation

Standard attention materializes the full N×N attention matrix (O(N²) memory), then applies softmax. Flash Attention:

1. **Blockwise computation** — Compute attention in tiles that fit in SRAM
2. **Online softmax** — Compute softmax incrementally without full matrix
3. **IO-aware** — Minimize HBM (GPU memory) ↔ SRAM transfers

### Speedup

| Metric | Standard | Flash Attention | Speedup |
|--------|----------|-----------------|---------|
| Memory | O(N²) | O(N) | 4-16× reduction |
| Runtime | 1.0× | 2-4× | 2-4× faster |
| Accuracy | Exact | Exact | Identical |

---

## FlashAttention-2

> **From:** Dao-AILab/flash-attention

Improvements over v1:
- **Better parallelism** — Row parallelism over sequence length
- **Better GPU utilization** — Improved warp scheduling
- **~2× speedup** over v1

---

## FlashAttention-3 (Latest)

> **From:** Dao-AILab/flash-attention (in development)

Further optimizations for **H100 GPUs**:
- **Tensor core integration** — TMA (Tensor Memory Accelerator) support
- **Asynchronous data movement** — Overlaps computation and communication
- **~1.5-2× speedup** over v2 on H100

---

## FlashMLA (DeepSeek)

### "FlashMLA: Efficient Multi-head Latent Attention Kernels"

> **GitHub:** [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) ⭐12.6K · **Language:** C++ · **Organization:** DeepSeek-AI

Production-optimized attention kernels for **Multi-head Latent Attention (MLA)** — DeepSeek's memory-efficient attention variant.

### Key Features

| Feature | Description |
|---------|-------------|
| **MLA optimization** | Tuned for DeepSeek's MHA variant |
| **Variable-length support** | Efficient for ragged batching |
| **Hopper support** | Tensor Memory Accelerator on H100 |
| **Production-ready** | Used in DeepSeek-V3 inference |

---

## FlashInfer

### "FlashInfer: Efficient and Customizable Attention Engine for LLM Inference Serving"

> **arXiv:** [2501.01005](https://arxiv.org/abs/2501.01005) | **GitHub:** [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) ⭐5.4K

> *Efficient and Customizable Attention Engine for LLM Inference Serving.*

### Key Features

| Feature | Description |
|---------|-------------|
| **Customizable kernels** | Easy to extend for new attention variants |
| **JIT compilation** | Dynamic kernel generation |
| **Multiple attention types** | Grouped-query, multi-head, multi-query |
| **CUDA kernels** | Production-optimized |

### Topics

`attention` · `cuda` · `distributed-inference` · `gpu` · `jit`

---

## Comparison

| System | Stars | Focus | Use Case |
|--------|-------|-------|----------|
| [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) | ⭐23.4K | Training & inference | Research, foundation |
| [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) | ⭐12.6K | Inference optimization | Production DeepSeek |
| [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) | ⭐5.4K | Customizable kernels | Serving frameworks |

---

## See Also

- [README](./README.md) — Context Windows overview
- [PagedAttention](./PagedAttention-vLLM.md) — Combines with Flash Attention
- [StreamingLLM](./StreamingLLM-Attention-Sinks.md) — Works with Flash Attention
- [Ring Attention](./Ring-Attention.md) — Often implemented on top of Flash Attention
