# Triton-Based Quantization: GPU Kernel Programming for LLMs

> **Triton Official:** [github.com/triton-lang/triton](https://github.com/triton-lang/triton)  
> **GPTQ Triton:** [fpgaminer/GPTQ-triton](https://github.com/fpgaminer/GPTQ-triton)  
> **LLM Kernels:** [bassrehab/triton-kernels](https://github.com/bassrehab/triton-kernels) · [dropbox/gemlite](https://github.com/dropbox/gemlite)  
> **Papers:** [arXiv:2402.00025](https://arxiv.org/html/2402.00025v2) (SplitK) · [arXiv:2511.11581](https://arxiv.org/html/2511.11581v1) (Triton Attention)

---

## Overview

> *"Triton enables researchers to write fast CUDA-quality kernels in Python, achieving 80–95% of hand-tuned CUDA performance with a fraction of the development time."* — OpenAI Triton

**Triton** is OpenAI's DSL (Domain-Specific Language) for GPU kernel programming. Rather than writing CUDA in C++/PTX, you write Python-like code that Triton compiles to optimized GPU kernels. For LLM quantization, Triton kernels handle the **dequantization + matmul fusion** critical for quantized inference performance.

**Key advantage:** Same source code runs on **NVIDIA, AMD, and Intel GPUs** — no hardware-specific rewrites.

---

## Triton Implementations of Quantization Methods

### GPTQ-Triton

> **GitHub:** [fpgaminer/GPTQ-triton](https://github.com/fpgaminer/GPTQ-triton)

Full GPTQ inference implementation in Triton, based on GPTQ-for-LLaMa:

```python
# Fuses dequantization into kernel — no separate unpack step
# Benchmark on RTX 3090:
# GPTQ Triton: 1.70 it/s
# CUDA baseline: 0.11 it/s
# → 15× faster generation throughput
```

| Result | Details |
|--------|---------|
| **Speedup** | 15× faster vs CUDA baseline on generation |
| **Quality** | 5.44 wikitext2 ppl (matches CUDA) |
| **Memory** | Same as CUDA |

### PyTorch Blog: Accelerating Triton GPTQ Kernels

> **Blog:** [pytorch.org/blog/accelerating-triton/](https://pytorch.org/blog/accelerating-triton/)

Step-by-step optimization of the GPTQ Triton kernel:

| Optimization | Result |
|-------------|--------|
| **L2 tile swizzling** | Reduced memory contention |
| **Vectorized loads** | Better memory coalescing |
| **Warp stall reduction** | Higher GPU utilization |
| **Core kernel speedup** | **3× faster** |
| **Full AutoGPTQ speedup** | **6× faster** (275μs → 47μs) |

### General LLM Kernels (bassrehab)

> **GitHub:** [bassrehab/triton-kernels](https://github.com/bassrehab/triton-kernels)

Fused kernels for common LLM operations:

| Kernel | Speedup vs PyTorch |
|--------|-------------------|
| **Fused RMSNorm** | **8.1×** (11% → 88% peak bandwidth) |
| **SwiGLU activation** | **1.6×** |
| **INT8 GEMM (W8A16)** | ~1× speedup but **2× memory savings** |
| **Fused MoE** | Up to **9.1×** |

Includes **roofline analysis** for each kernel.

### GemLite: Low-Bit Matmul

> **GitHub:** [dropbox/gemlite](https://github.com/dropbox/gemlite)

Low-bit matrix multiplication for LLMs:

> *"GemLite achieves 7–8× prefill speedup and 3–6× decode speedup vs TorchAO default kernels, competitive with RedHat FP8/NVFP4 on RTX PRO 6000."*

| Precision | Prefill | Decode |
|---------|---------|--------|
| **8-bit** | 7–8× vs TorchAO | 3–6× vs TorchAO |
| **4-bit** | 7–8× | 3–6× |
| **2-bit** | 7–8× | 3–6× |

Integrates via **TorchAO** and **HQQ**.

---

## Performance: Triton vs CUDA

### Summary

| Source | Result |
|--------|--------|
| [EmergentMind](https://www.emergentmind.com/topics/triton-and-cuda-kernels) | Triton achieves **90–105% of hand-tuned CUDA** performance |
| SplitK Paper | **65% speedup on A100**, **124% on H100** vs DP block tiling |
| bassrehab kernels | RMSNorm 8.1×, INT8 GEMM ~1× but 2× memory savings |
| fpgaminer/GPTQ-triton | **15× faster** on generation vs CUDA baseline |
| PyTorch Blog | 3× core, 6× full AutoGPTQ with optimization |
| GemLite | 7–8× prefill, 3–6× decode vs TorchAO |
| ML-Triton (2025) | **95–105% of FlashAttention3** with same source on NVIDIA + AMD |

### SplitK: W4A16 Fused Kernel (Meta)

> **Paper:** [arXiv:2402.00025](https://arxiv.org/html/2402.00025v2)

SplitK work decomposition for fused dequantization + GEMM:

```
Method: SplitK — splits work across K dimension to increase parallelism
Hardware: A100 and H100
Result: 65% average speedup on A100, 124% on H100 (peak 295%)
```

---

## vLLM Integration

### Quantization Support in vLLM

> **Docs:** [docs.vllm.ai/en/latest/features/quantization/](https://docs.vllm.ai/en/latest/features/quantization/)

vLLM supports all major quantization formats. Triton is used as the **default backend for AMD GPUs** and for several quantization kernels:

| Format | Kernel Type |
|--------|-------------|
| **W4A16** | Triton GEMM (`triton_w4a16_gemm_kernel`) |
| **FP8** | Triton scaled_mm |
| **AWQ** | Via llm-compressor + Triton |
| **GPTQ** | Via Marlin (CUDA) or Triton |
| **Marlin** | CUDA (INT4 Tensor Core) |

### Built-in Triton Kernels

| Kernel | Path |
|--------|------|
| **W4A16 GEMM** | `vllm/model_executor/kernels/linear/mixed_precision/triton_w4a16/` |
| **FP8 scaled_mm** | `vllm/model_executor/kernels/linear/mixed_precision/scaled_mm/` |
| **Quantization ops** | Various Triton-based quantization/dequantization |

### Triton Attention Kernel (IBM/vLLM)

> **Paper:** [arXiv:2511.11581](https://arxiv.org/html/2511.11581v1) — "The Anatomy of a Triton Attention Kernel"

Cross-platform paged attention in Triton:
- **95–105% of FlashAttention3** performance
- Same source code on **NVIDIA and AMD** GPUs
- Now **default in vLLM for AMD** GPUs
- Addresses Triton backend limitations for attention

### Triton MoE Integration

> **vLLM Issue #16294:** [github.com/vllm-project/vllm/issues/16294](https://github.com/vllm-project/vllm/issues/16294)

Tracks integration of Triton's new MoE kernel into vLLM.

---

## NVIDIA Triton Inference Server

### vLLM Backend

> **Docs:** [docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/vllm_backend/README.html](https://docs.nvidia.com/deeplearning/triton-inference-server/user-guide/docs/vllm_backend/README.html)

Triton Inference Server supports **vLLM as a backend** starting from version 23.10:

```
Triton Server
├── Model A → vLLM backend
├── Model B → TensorRT-LLM backend
└── Model C → ONNX backend
```

Multi-model serving with vLLM's optimized LLM inference.

### AMD ROCm + Triton + vLLM

> **Blog:** [rocm.blogs.amd.com/artificial-intelligence/triton_server_vllm/README.html](https://rocm.blogs.amd.com/artificial-intelligence/triton_server_vllm/README.html)

Guide for running Microsoft/phi-2, Mistral-7B, Llama-3-8B on AMD GPUs via Triton Server + vLLM.

---

## Papers & Technical References

| Paper | Key Contribution |
|-------|-----------------|
| **[SplitK W4A16](https://arxiv.org/html/2402.00025v2)** (Meta, 2024) | SplitK fused dequantization + GEMM. 65% A100, 124% H100 speedup |
| **[Triton Attention Anatomy](https://arxiv.org/html/2511.11581v1)** (IBM/vLLM, 2025) | Cross-platform paged attention. 95–105% of FlashAttention3 |
| **[Towards Automated Kernel Generation](https://arxiv.org/pdf/2601.15727)** (2026) | Survey: Triton, CUTLASS, FlashAttention, FlagGems, Liger-Kernel, GemLite |
| **[Automatic Triton Programming with RL](https://arxiv.org/html/2507.05687v1)** (2025) | GRPO algorithm for LLM-based Triton kernel generation |
| **[RL for Triton Kernels](https://arxiv.org/abs/2602.05885)** (2025) | KernelGym framework for RL-based optimization |
| **[PyTorch Blog](https://pytorch.org/blog/accelerating-triton/)** (2024) | 6× AutoGPTQ speedup via Triton kernel optimization |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [triton-lang/triton](https://github.com/triton-lang/triton) | Official OpenAI Triton — Python DSL for GPU kernels |
| [fpgaminer/GPTQ-triton](https://github.com/fpgaminer/GPTQ-triton) | GPTQ in Triton — 15× faster generation vs CUDA |
| [bassrehab/triton-kernels](https://github.com/bassrehab/triton-kernels) | Fused LLM kernels — RMSNorm 8.1×, MoE 9.1× |
| [dropbox/gemlite](https://github.com/dropbox/gemlite) | Low-bit matmul — 7–8× prefill, 3–6× decode vs TorchAO |
| [Cornell-RelaxML/QuIP](https://github.com/Cornell-RelaxML/QuIP) | QTIP/QuIP# — not Triton-specific, but CUDA baseline |
| [vllm-project/vllm](https://github.com/vllm-project/vllm) | vLLM — uses Triton for AMD backend and W4A16/FP8 kernels |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **What it is** | OpenAI's Python DSL for writing GPU kernels that compile to CUDA/HIP equivalent |
| **Key advantage** | Same code runs on NVIDIA, AMD, Intel — no hardware rewrites |
| **Performance** | 80–95% of hand-tuned CUDA for most kernels; 90–105% for optimized ones |
| **GPTQ Triton** | 15× faster than naive CUDA; 6× with PyTorch blog optimizations |
| **SplitK** | 65% A100 speedup, 124% H100 speedup for W4A16 |
| **Triton Attention** | 95–105% of FlashAttention3, cross-platform |
| **vLLM use** | Default AMD backend; W4A16 and FP8 Triton kernels |
| **AWQ/QuIP** | Not Triton-specific; supported in vLLM/TorchAO backends |

---

*Last updated: 2026-04-19*
