# Marlin & BitBlas: Optimized GPU Quantization Kernels

> **Marlin Paper:** [arXiv:2408.11743](https://arxiv.org/abs/2408.11743) · **GitHub:** [IST-DASLab/marlin](https://github.com/IST-DASLab/marlin) · [IST-DASLab/Sparse-Marlin](https://github.com/IST-DASLab/Sparse-Marlin)  
> **BitBlas:** [microsoft/BitBLAS](https://github.com/microsoft/BitBLAS) · [Quick Start](https://github.com/microsoft/BitBLAS/blob/main/docs/QuickStart.md)  
> **Integrations:** [vLLM Marlin](https://docs.vllm.ai/en/latest/api/vllm/model_executor/layers/quantization/gptq_marlin/) · [vLLM BitBLAS](https://docs.vllm.ai/en/v0.10.1/features/quantization/bitblas.html)

---

## Overview

> *"Marlin achieves near-ideal ~4× speedups for INT4×FP16 matmul at batch sizes up to 32 tokens — compared to prior work which only achieved this at batch=1-2."* — Frantar et al., arXiv:2408.11743

**Marlin** and **BitBlas** are two complementary GPU kernel libraries for efficient quantized LLM inference. Both target the **dequantization bottleneck** — the overhead of unpacking INT4/INT8 weights before matrix multiplication on Tensor Cores.

| Kernel | Origin | Primary Focus | Best For |
|--------|--------|--------------|----------|
| **Marlin** | IST-DASLab | FP16×INT4 batched inference | Autoregressive batched generation (batch 16-32) |
| **BitBlas** | Microsoft | General mixed-precision matmul | Flexible precision combos, weight-only operations |

---

## Marlin: Mixed-Precision Auto-Regressive Linear Kernel

> **Paper:** [arXiv:2408.11743](https://arxiv.org/abs/2408.11743) · **GitHub:** [github.com/IST-DASLab/marlin](https://github.com/IST-DASLab/marlin)  
> **Authors:** Elias Frantar (ISTA), Roberto L. Castro, Jiale Chen, Torsten Hoefler (ETH Zurich), Dan Alistarh (Neural Magic)  
> **Named after:** The fast fish — "named MARLIN for Mixed Auto-Regressive Linear INference"

### Core Innovation

Marlin solves the **autoregressive inference bottleneck** for INT4×FP16 matmul:

```
Problem: Dequantization overhead dominates at small batch sizes
- Prior kernels: ~2× speedup at batch=32, degrading at smaller batches
- Marlin: ~4× speedup sustained from batch=1 to batch=32
```

### Kernel Architecture

| Optimization | Description |
|-------------|-------------|
| **Async weight loads** | Overlaps HBM → GPU memory transfer with computation |
| **Circular shared memory queue** | P=4 pipeline depth: HBM → L2 → SHM → Registers → Tensor Cores |
| **Stream-K parallelization** | Optimal work distribution across GPU SMs |
| **XOR bank-conflict-free indexing** | Eliminates shared memory bank conflicts for A fragments |
| **Parallel dequantization** | Two INT4→FP16 in a single INT32 via modified binary manipulation |
| **Offline weight pre-processing** | Reshuffles 16×64 tiles into contiguous memory layout |

### Performance Results

#### Per-Layer Speedup (A100, groupsize=128)

| Batch Size | Speedup vs FP16 | Regime |
|-----------|----------------|--------|
| 1–16 | ~3.87× | Memory-bound, near-optimal |
| 16–32 | ~3.9× | Near-optimal |
| 64 | ~3.5× | Transitioning |
| 128 | ~1.5× | Compute-bound |

#### End-to-End (vLLM, Llama-2-7B, RTX A6000)

| Configuration | Speedup vs FP16 |
|--------------|-----------------|
| **Marlin** | 2.8× (batch=16) |
| **Sparse-Marlin** | 3.2× (batch=16, 2:4 sparsity) |
| **Sparse-Marlin TPOT** | 3.5× (at 10 QPS serving) |

---

## BitBlas: General Mixed-Precision BLAS for GPUs

> **GitHub:** [microsoft/BitBLAS](https://github.com/microsoft/BitBLAS) · [Quick Start](https://github.com/microsoft/BitBLAS/blob/main/docs/QuickStart.md)  
> **Based on:** Ladder (OSDI'24) — hardware-aware tensor transformation techniques

### Supported Precision Combinations

| A_dtype | W_dtype | Tested Platforms |
|---------|---------|----------------|
| **BF16** | UINT4/INT4, INT2, INT8, FP4, FP8, NF4, UINT1 | A100 (SM_80), A6000 (SM_86) |
| **FP16** | UINT4/INT4, INT2, INT8, FP4, FP8, NF4, UINT1 | V100, A100, A6000, RTX 4090 (SM_89) |
| **INT8** | INT8, UINT4/INT4, INT2, UINT1 | V100, A100, A6000, RTX 4090 |
| **FP8_E4M3/FP8_E5M2** | FP8 | RTX 4090 (SM_89) |
| **INT4** | INT4 | RTX 4090 (SM_89) |

### Python API

```python
import bitblas

# W_INT4 × A_FP16 → out_FP16
matmul = bitblas.Matmul(
    A_dtype=bitblas.float16,
    W_dtype=bitblas.int4,
    out_dtype=bitblas.float16,
    N=8192, K=4096
)
```

### Performance Claims

> *"BitBLAS INT4 dequantization outperforms bitsandbytes, faster-transformer, tensorrt-llm, vLLM, and Marlin"* — BitBLAS benchmarks on RTX 3090 (24GB) and A100 (80GB)

---

## Marlin vs BitBlas vs Prior Work

| Aspect | Marlin | BitBlas | bitsandbytes | FasterTransformer |
|--------|--------|---------|-------------|------------------|
| **Precision** | INT4/FP16 primary | INT1–INT8, FP4–FP16, BF16 | INT8/NF4 | INT8 |
| **Batch efficiency** | Near-ideal 4× at batch 1–32 | Varies | Degrades at small batch | Good |
| **Autoregressive focus** | ✅ Yes | ❌ General matmul | Partial | Partial |
| **Hardware** | NVIDIA Ampere+ | V100, A100, RTX 3090/4090 | Broad | NVIDIA |
| **Tensor Core usage** | ✅ Optimized INT4→TC | ✅ | ✅ | ✅ |

---

## Framework Integrations

### vLLM

```python
# Marlin
from vllm import LLM
llm = LLM(
    model="your-model",
    dtype=torch.float16,
    quantization="gptq_marlin"
)

# BitBLAS
llm = LLM(
    model="hxbgsyxh/llama-13b-4bit-g-1-bitblas",
    dtype=torch.bfloat16,
    trust_remote_code=True,
    quantization="bitblas"
)
```

### AutoGPTQ

```python
# Marlin support (v0.7.0+)
from auto_gptq import AutoGPTQForCausalLM
model = AutoGPTQForCausalLM.from_quantized(
    model_name_or_path,
    use_marlin=True  # enables Marlin kernel
)
```

### HuggingFace

```python
# Via optimum-gptq
from optimum.gptq import GPTQQuantizer
quantizer = GPTQQuantizer(use_marlin=True)
```

---

## Sparse-Marlin: Marlin + 2:4 Structured Sparsity

> **GitHub:** [IST-DASLab/Sparse-Marlin](https://github.com/IST-DASLab/Sparse-Marlin)

Marlin combined with **2:4 structured sparsity** (every 4 weights, 2 are zero):

```
Sparse-Marlin = Marlin kernel + 2:4 sparsity pattern
→ 2× weight compression + 4× INT4 speedup
→ Net: 8× effective speedup potential
```

---

## Papers & Technical References

| Resource | URL |
|----------|-----|
| **Marlin Paper (arXiv)** | [arxiv.org/abs/2408.11743](https://arxiv.org/abs/2408.11743) |
| **Marlin HTML** | [arxiv.org/html/2408.11743v1](https://arxiv.org/html/2408.11743v1) |
| **Marlin GitHub** | [github.com/IST-DASLab/marlin](https://github.com/IST-DASLab/marlin) |
| **Sparse-Marlin** | [github.com/IST-DASLab/Sparse-Marlin](https://github.com/IST-DASLab/Sparse-Marlin) |
| **BitBLAS GitHub** | [github.com/microsoft/BitBLAS](https://github.com/microsoft/BitBLAS) |
| **BitBLAS Benchmarks** | [github.com/microsoft/BitBLAS/blob/main/benchmark/README.md](https://github.com/microsoft/BitBLAS/blob/main/benchmark/README.md) |
| **vLLM Marlin Docs** | [docs.vllm.ai/en/latest/api/vllm/model_executor/layers/quantization/gptq_marlin/](https://docs.vllm.ai/en/latest/api/vllm/model_executor/layers/quantization/gptq_marlin/) |
| **vLLM BitBLAS Docs** | [docs.vllm.ai/en/v0.10.1/features/quantization/bitblas.html](https://docs.vllm.ai/en/v0.10.1/features/quantization/bitblas.html) |

---

## Summary

| Aspect | Marlin | BitBlas |
|--------|--------|---------|
| **Origin** | IST-DASLab | Microsoft |
| **Primary focus** | FP16×INT4 batched autoregressive inference | General mixed-precision matmul |
| **Speedup** | ~4× at batch 1–32 | Varies by config |
| **Precision support** | INT4/FP16 primary | INT1–INT8, FP4–FP16, BF16, NF4, FP8 |
| **vLLM support** | ✅ `quantization="gptq_marlin"` | ✅ `quantization="bitblas"` |
| **Autoregressive inference** | ✅ Best-in-class | ❌ General purpose |
| **Sparsity support** | ✅ Sparse-Marlin (2:4) | ❌ |

---

*Last updated: 2026-04-19*
