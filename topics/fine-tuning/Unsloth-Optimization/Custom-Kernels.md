# Custom Kernels (for LLM Fine-tuning Optimization)

## 📌 Overview

| Field | Value |
|-------|-------|
| **What** | Custom Triton/CUDA kernels that fuse operations into single GPU launches |
| **Goal** | Reduce kernel launch overhead, eliminate HBM traffic, fuse optimizers |
| **Key Benefit** | 2-5x speedup, 30-90% VRAM reduction |
| **Key Repo** | [unslothai/unsloth](https://github.com/unslothai/unsloth) (56.5k stars) |

> "Fused Triton kernels are single-launch GPU implementations that merge multiple tensor operations — such as normalization, activation, embedding, and collective communication — into one pass. By eliminating repeated kernel launches and intermediate HBM traffic, they achieve up to 8x speedups and over 50% memory reduction in LLM training and inference."

— [Emergent Mind](https://www.emergentmind.com/topics/fused-triton-kernels)

---

## Definitions

### Key Concepts

| Concept | Definition |
|---------|------------|
| **Kernel Fusion** | Merging multiple GPU operations into a single kernel launch |
| **Row-wise Linear** | Processing each row of a weight matrix independently, parallelizing across batch × sequence |
| **Fused Optimizer (AdamW)** | Combining gradient accumulation and weight updates into one kernel |
| **Gradient Checkpointing** | Trading compute for memory by recomputing activations during backward pass |
| **Memory-Efficient Attention** | IO-aware attention (FlashAttention) tiling computation in SRAM |
| **Triton** | OpenAI's Python-first GPU kernel language that JIT-compiles to CUDA/ROCm |
| **RoPE** | Rotary Position Embedding using rotation matrices; Unsloth fuses Q+K rotation into one kernel |

### Row-Wise Linear Kernel

> "All kernels reshape I/O tensors to 2D matrices of shape (B×T, H) and parallelize per row."

— [Liger Kernel arXiv:2410.10989v2](https://arxiv.org/html/2410.10989v2)

### Fused AdamW

> "PyTorch AdamW issues 5+ separate CUDA kernels per step. Triton fuses all state updates into one — read w, g, m, v once; write w, m, v once."

— [anviit/triton-llm-kernels](https://github.com/anviit/triton-llm-kernels)

### Memory-Efficient Attention

> "FlashAttention eliminates the need to materialize S in global memory by introducing tiling."

— [Emergent Mind](https://www.emergentmind.com/topics/flash-attention-kernels)

---

## Key Sources

### Unsloth Blog — "3x Faster LLM Training with Kernels + Packing"

| Field | Value |
|-------|-------|
| URL | [unsloth.ai/docs/blog/3x-faster-training-packing](https://unsloth.ai/docs/blog/3x-faster-training-packing) |
| Author | Unsloth AI |

> "Unsloth now supports up to 5x faster (typically 3x) training with our new custom RoPE and MLP Triton kernels, plus our new smart auto packing."

> "Merged 2 Triton kernels into 1 and enabled variable length RoPE. 2.3x faster on long context lengths, 1.9x faster on shorter. Eliminated clones and contiguous transpose operations; RoPE is now fully inplace."

> "GPUs cannot process variable-length sequences, requiring padding with zeros. This causes waste... By combining multiple sequences into one tensor."

### Liger Kernel — LinkedIn Engineering (ICML 2025 Workshop)

| Field | Value |
|-------|-------|
| Paper | [arXiv:2410.10989v2](https://arxiv.org/html/2410.10989v2) |
| OpenReview | [OpenReview ID: 36SjAIT42G](https://openreview.net/forum?id=36SjAIT42G) |
| Authors | LinkedIn Engineering |
| Published | ICML 2025 Workshop CODEML, June 2025 |

> "With kernel optimization techniques like kernel operation fusing and input chunking, our kernels achieve on average 20% increase in training throughput and a 60% reduction in GPU memory compared with HuggingFace implementations."

> "The true potential for optimization lies in fusing operations at the GPU kernel level, which minimizes memory copying and maximizes parallel efficiency."

### Subhadip Mitra — "From 11% to 88% Peak Bandwidth: Writing Custom Triton Kernels"

| Field | Value |
|-------|-------|
| URL | [subhadipmitra.com/blog/2025/triton-kernels-llm-inference](https://subhadipmitra.com/blog/2025/triton-kernels-llm-inference/) |
| Date | June 2025 |

> "PyTorch RMSNorm achieves 11% of peak memory bandwidth. My Triton version hits 88%. That's an 8x speedup on a single operation."

> "Three reasons PyTorch is slow: (1) Kernel launch overhead ~5-10μs per launch. (2) Intermediate tensors materialize to GPU memory. (3) Generic implementations handle every edge case."

### Subhadip Mitra — "Beating CUDA with Triton: A Fused MoE Dispatch Kernel"

| Field | Value |
|-------|-------|
| URL | [subhadipmitra.com/blog/2026/fused-moe-dispatch-triton](https://subhadipmitra.com/blog/2026/fused-moe-dispatch-triton/) |
| Date | January 2026 |

> "Triton doesn't support 2D scalar indexing. `acc[m, :]` where `m` is a loop variable doesn't compile."

> "At inference-relevant batch sizes (1-128 tokens), the pure-Triton kernel is faster than Megablocks (Stanford's CUDA-optimized MoE library)."

### Towards Data Science — "Cutting LLM Memory by 84%"

| Field | Value |
|-------|-------|
| URL | [towardsdatascience.com/cutting-llm-memory-by-84-a-deep-dive-into-fused-kernels](https://towardsdatascience.com/cutting-llm-memory-by-84-a-deep-dive-into-fused-kernels/) |
| Date | 2024 |

> "The bottleneck isn't the weight matrix itself (~1 GB), but the intermediate logit tensor, which can exceed 80 GB of VRAM for large batches."

> "By fusing the linear projection and cross-entropy into a single kernel: (1) Eliminate the large intermediate logit tensor. (2) Keep intermediate values on-chip. (3) Reduce memory complexity from O(V) to O(1)."

---

## Repositories & Tools

### Unsloth (Main Fine-tuning Framework)

| Field | Value |
|-------|-------|
| Repository | [github.com/unslothai/unsloth](https://github.com/unslothai/unsloth) |
| Stars | 56.5k |
| License | Apache 2.0 |

Key kernels: Custom Triton RoPE (fused Q+K rotation, in-place), MLP (SwiGLU/GeGLU with int64 indexing), smart padding-free packing.

### Liger-Kernel (LinkedIn)

| Field | Value |
|-------|-------|
| Repository | [github.com/linkedin/Liger-Kernel](https://github.com/linkedin/Liger-Kernel) |
| License | Apache 2.0 |
| Docs | [linkedin.github.io/Liger-Kernel](https://linkedin.github.io/Liger-Kernel/) |

Supported kernels per model family:

| Model Family | Kernels |
|--------------|---------|
| Llama 2/3/4 | RoPE, RMSNorm, SwiGLU, CrossEntropyLoss, FusedLinearCrossEntropy |
| Gemma 1/2/3 | RoPE, RMSNorm, GeGLU, CrossEntropyLoss, FusedLinearCrossEntropy |
| Qwen2/2.5/3 | RoPE, RMSNorm, SwiGLU, CrossEntropyLoss, FusedLinearCrossEntropy |
| Mistral/Mixtral | RoPE, RMSNorm, SwiGLU, CrossEntropyLoss, FusedLinearCrossEntropy |

### Individual Triton Kernels (anviit)

| Repository | [github.com/anviit/triton-llm-kernels](https://github.com/anviit/triton-llm-kernels) |

| Kernel | Speedup vs PyTorch |
|--------|-------------------|
| FlashAttention | 2.52x |
| Fused AdamW | 3.45x |
| Bias+GELU | 14.65x |

> "The central thesis: modern LLM performance is not about FLOPs. It is about memory bandwidth."

— anviit/triton-llm-kernels

### FlashAttention (Dao-AILab)

| Repository | [github.com/dao-ailab/flash-attention](https://github.com/dao-ailab/flash-attention) |
| Authors | Tri Dao et al. |

- IO-aware exact attention algorithm
- Memory complexity: O(T) vs O(T²) for standard attention
- Versions: FA2 (Ampere/Ada), FA3 (Hopper/H100), FA4 (Blackwell)

---

## Benchmark Results

### Unsloth End-to-End

| Model | Speedup | VRAM Reduction |
|-------|---------|----------------|
| Qwen3.5 (4B) | 1.5x | 60% |
| gpt-oss (20B) | 2x | 70% |
| gpt-oss (20B) + GRPO | 2x | 80% |
| Llama 3.1 (8B) | 2x | 70% |
| Gemma 3 (4B) Vision | 1.7x | 60% |

### Smart Packing Speedup (Unsloth)

| Batch Size | Speedup |
|-----------|---------|
| 2 | +40% |
| 4 | +70% |
| 8 | +2x |
| 16 | +2.5x |
| 32 | +3x |

### Liger-Kernel Component Benchmarks (A100 80GB)

| Kernel | Speedup | Memory Reduction |
|--------|---------|----------------|
| RMSNorm | 7x | 3x |
| CrossEntropy | 3x | 5x |
| RoPE | 8x | 3x |
| GeGLU/SwiGLU | ~same speed | 1.6x |

### Liger-Kernel End-to-End Training (4× A100)

| Model | Batch | Throughput Gain | GPU Memory Reduction |
|-------|-------|----------------|---------------------|
| LLaMA-3-8B | 64 | +42.8% | -54.8% |
| Qwen2 | 48 | +25.5% | -56.8% |
| Gemma-7B | 48 | +11.9% | -51.8% |
| Mistral-7B | 128 | +27% | -21% |

### Triton vs PyTorch RMSNorm (Subhadip Mitra)

| Implementation | Latency | Bandwidth | % of Peak |
|----------------|---------|-----------|-----------|
| PyTorch | 0.30 ms | 168 GB/s | 11% |
| Triton | 0.04 ms | 1365 GB/s | 88% |

A100 theoretical peak: ~1555 GB/s

---

## Triton vs CUDA

### Why Triton

| Aspect | Triton | CUDA (CUTLASS, cuBLAS) |
|--------|--------|------------------------|
| **Abstraction** | Python-first, block-level | Low-level C++ template |
| **Portability** | JIT-compiles to CUDA/ROCm | GPU vendor-specific |
| **Iteration Speed** | Fast prototyping | Slow compile cycles |
| **Flexibility** | Easily customizable | Highly optimized but rigid |

> "Triton lets you write fused kernels that avoid all three problems [of PyTorch]."

— Subhadip Mitra

### When CUDA Still Wins

> "FlashAttention-4: 1613 TFLOPs/s, 2.7x faster than Triton, written in Python."

— [Reddit r/LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1s1yw23/flashattention4_1613_tflopss_27x_faster_than/)

FlashAttention-4 achieves 2.7x higher TFLOPS on Blackwell via warp-specialized 5-stage pipeline and software exponentials.

---

## Limitations

### Kernel Launch Overhead (Triton GitHub #2637)

> "The kernel executes around 80μs on GPU, however, it takes 220μs to launch."

Mitigation: Cached kernels, batching, warm-up runs.

### Compilation Time (Triton GitHub #8908)

> "The following triton kernel takes 117-151 seconds to compile depending on machine."

Mitigation: Use `tl.constexpr` for compile-time constants.

### Not Always Worth It for Small Models

> "Gradient checkpointing slows the train down enough that even the larger batch size does not allow us to train faster."

— [Giles Thomas](https://www.gilesthomas.com/2024/09/fine-tuning-9)

Custom kernels trade memory for compute — for small models or short sequences, overhead may not pay off.

### Single-GPU Focus

Unsloth is primarily single-GPU focused. For large-scale multi-GPU training, FSDP2 or DeepSpeed may be more appropriate.

---

## Summary Data Points

| Metric | Value | Source |
|--------|-------|--------|
| Unsloth training speedup | 2-5x | Unsloth Blog |
| Unsloth VRAM reduction | 30-90% | Unsloth Blog |
| Liger-Kernel throughput gain | ~20% avg | arXiv:2410.10989v2 |
| Liger-Kernel memory reduction | ~60% | Liger-Kernel GitHub |
| RMSNorm speedup (Triton vs PyTorch) | 7-8.1x | Emergent Mind / Mitra |
| RoPE speedup | 8x | Emergent Mind |
| CrossEntropy speedup | 3x | Emergent Mind |
| Fused AdamW speedup | 3.45x | anviit |
| Bias+GELU speedup | 14.65x | anviit |
| FlashAttention speedup vs naive | 2.52x | anviit |
| RMSNorm bandwidth | 88% (Triton) vs 11% (PyTorch) | Mitra 2025 |
| Kernel launch overhead | 5-10μs/launch | Triton GitHub #2637 |
| Compilation time | Up to 151s | Triton GitHub #8908 |
| FlashAttention-4 TFLOPS (Blackwell) | 1613 TFLOPs/s | Reddit |

---

## Related Topics

- [Unsloth-AI.md](./Unsloth-AI.md) — The company behind these kernels
- [QAT.md](./QAT.md) — Quantization-Aware Training
- [FSDP2.md](./FSDP2.md) — PyTorch per-parameter sharding for distributed fine-tuning
- [Axolotl.md](./Axolotl.md) — YAML-driven multi-GPU fine-tuning

---

## References

- [Emergent Mind — Fused Triton Kernels](https://www.emergentmind.com/topics/fused-triton-kernels)
- [Liger Kernel arXiv:2410.10989v2](https://arxiv.org/html/2410.10989v2)
- [Liger Kernel OpenReview](https://openreview.net/forum?id=36SjAIT42G)
- [Subhadip Mitra — Triton Kernels for LLM Inference](https://subhadipmitra.com/blog/2025/triton-kernels-llm-inference/)
- [Subhadip Mitra — Fused MoE Dispatch](https://subhadipmitra.com/blog/2026/fused-moe-dispatch-triton/)
- [Towards Data Science — Cutting LLM Memory by 84%](https://towardsdatascience.com/cutting-llm-memory-by-84-a-deep-dive-into-fused-kernels/)
- [Unsloth Blog — 3x Faster Training + Packing](https://unsloth.ai/docs/blog/3x-faster-training-packing)
