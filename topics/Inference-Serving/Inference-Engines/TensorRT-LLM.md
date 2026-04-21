# TensorRT-LLM

> **GitHub:** [NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) — 13.4k stars  
> **Docs:** [nvidia.github.io/TensorRT-LLM](https://nvidia.github.io/TensorRT-LLM/)  
> **Papers:** [DWDP arXiv:2604.01621](https://arxiv.org/html/2604.01621v1)

---

## Overview

> *"TensorRT-LLM provides an easy-to-use Python API for defining, building, and running optimized large language model inference."* — [NVIDIA TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM)

TensorRT-LLM is NVIDIA's **production-grade LLM inference library** built on the TensorRT deep learning compiler with custom CUDA kernels, Inflight Batching, and deep Hopper/Blackwell optimization. It excels at **single-request throughput** but scales differently from vLLM at high concurrency.

---

## Architecture

### Key Difference from Generic TensorRT

| Aspect | Generic TensorRT | TensorRT-LLM |
|--------|----------------|--------------|
| **Focus** | CNNs, vision | LLMs only |
| **Attention** | Auto-discovered | Custom gpt_attention plugin |
| **Batching** | Static graphs | **Inflight Batching (IFB)** |
| **KV Cache** | Not supported | **Paged KV Cache** |
| **Quantization** | INT8/FP16 | **FP8, FP4, AWQ, GPTQ** |
| **Model defs** | User-defined | **Pre-built**: Llama, Mistral, GPT, Qwen, DeepSeek |

### Inflight Batching (IFB)

> *"Dynamically manages request execution, processing context and generation phases together for maximum GPU utilization."* — [NVIDIA Docs](https://nvidia.github.io/TensorRT-LLM/)

IFB keeps all GPUs busy by mixing ready requests at every iteration — context (prefill) and generation (decode) phases processed together rather than in separate batches.

### Plugin Architecture

Custom fused kernels (FlashAttention-like) that cannot be auto-discovered by the compiler:

- `gpt_attention` — custom attention kernel
- `reshape妈` — fused reshape + softmax patterns
- Custom GEMM patterns for quantization

---

## FP8 & Quantization

> *"FP8 can double performance and halve memory consumption vs FP16."* — [NVIDIA TensorRT-LLM](https://nvidia.github.io/TensorRT-LLM/features/quantization.html)

| Format | Hardware | Description |
|--------|----------|-------------|
| **FP4** | Blackwell (SM100) | Lowest precision |
| **FP8 E4M3** | Hopper, Blackwell | High precision (H100, B200) |
| **FP8 E5M2** | Hopper, Blackwell | Wide range |
| **INT8** | All TensorRT | Compute quantization |
| **W4A16** | All TensorRT | 4-bit weight, 16-bit activation |
| **W4A8** | All TensorRT | 4-bit weight, 8-bit activation |
| **AWQ/GPTQ** | All TensorRT | Per-group scaling + zero-offset |

---

## Multi-GPU Tensor Parallelism

| Strategy | Description |
|---------|-------------|
| **TP** | Tensor Parallel — shard weights across GPUs |
| **PP** | Pipeline Parallel — stages across nodes |
| **EP** | Expert Parallel — full experts per GPU (MoE) |
| **Wide-EP** | Expert replication + load balancing for large-scale MoE |
| **CP** | Context Parallel — context sharded across GPUs |
| **DP** | Data Parallel — replicate for throughput |
| **DWDP** | Distributed Weight Data Parallelism for NVL72 systems |

> *"Wide-EP is the advanced solution for large-scale MoE deployments with expert replication and load balancing."* — [NVIDIA Blog](https://nvidia.github.io/TensorRT-LLM/features/parallel-strategy.html)

---

## Benchmark Performance

### H100 vs A100

> *"H100 FP8 delivers 4.6× max throughput and 4.4× faster 1st token latency vs A100 FP16."* — [NVIDIA Blog](https://nvidia.github.io/TensorRT-LLM/blogs/H100vsA100.html)

| Model | Hardware | Format | Throughput |
|-------|----------|--------|-----------|
| GPT-J 6B | H100 FP8 | Batch 64 | 10,907 tok/s |
| DeepSeek-R1 | B200 | FP4 | **World record inference** |
| Llama 4 Maverick | B200 | BF16 | **>1,000 TPS/User** |

### vs vLLM and SGLang (GPT-OSS-120B, 2× H100)

> **Source:** [Clarifai Blog](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b)

| Concurrency | TRT-LLM Throughput | vLLM Throughput | SGLang Throughput |
|-------------|-------------------|-----------------|-------------------|
| **1** | **243 tok/s** | 187 tok/s | 231 tok/s |
| 10 | 867 tok/s | 863 tok/s | **988 tok/s** |
| 50 | 2,163 tok/s | 2,212 tok/s | **3,109 tok/s** |
| 100 | 1,943 tok/s | **4,742 tok/s** | 3,222 tok/s |

| Concurrency | TRT-LLM TTFT | vLLM TTFT | SGLang TTFT |
|-------------|-------------|-----------|-------------|
| 1 | 0.177s | **0.053s** | 0.125s |
| 10 | 2.50s | 1.91s | **1.16s** |
| 100 | 5.47s | **1.87s** | 8.99s |

**Key takeaway:** TensorRT-LLM wins at **single-request throughput** but degrades at high concurrency. **On B200**, consistently outperforms both vLLM and SGLang due to deeper Blackwell optimization.

---

## Production Integration

### Triton Backend

> *"Triton inflight_batcher_llm is the C++ runtime for batching and KV cache management."* — [NVIDIA](https://github.com/triton-inference-server/tensorrtllm_backend)

```bash
# Triton + TensorRT-LLM production stack
github.com/triton-inference-server/tensorrtllm_backend
```

### AutoDeploy

Automated optimization pipeline: PyTorch model → TensorRT engine with minimal configuration.

---

## Key Technical Blogs

| Title | URL |
|-------|-----|
| **DWDP: Distributed Weight Data Parallelism for NVL72** | [arXiv](https://arxiv.org/html/2604.01621v1) |
| **Optimizing MoE Communication with One-Sided AlltoAll** | [NVIDIA Blog](https://nvidia.github.io/TensorRT-LLM/blogs/scaling-moe.html) |
| **Sparse Attention in TensorRT LLM** | [NVIDIA Blog](https://nvidia.github.io/TensorRT-LLM/blogs/sparse-attention.html) |
| **H100 vs A100 Performance** | [NVIDIA Blog](https://nvidia.github.io/TensorRT-LLM/blogs/H100vsA100.html) |

---

## Papers Table

| Paper | Year | URL | Key Contribution |
|-------|------|-----|-----------------|
| **DWDP: Distributed Weight Data Parallelism** | 2026 | [arXiv](https://arxiv.org/html/2604.01621v1) | NVL72-scale inference optimization |
| **FlashAttention** | 2023 | [arXiv](https://arxiv.org/abs/2307.08691) | Referenced as attention kernel foundation |
| **FP8 Formats** | 2022 | [arXiv](https://arxiv.org/abs/2209.05433) | FP8 E4M3/E5M2 specification |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) | Main repo — 13.4k stars, Apache 2.0 |
| [triton-inference-server/tensorrtllm_backend](https://github.com/triton-inference-server/tensorrtllm_backend) | Triton backend for production serving |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Single-request throughput, production NVIDIA deployment, Blackwell (B200) optimization |
| **Key innovation** | Inflight Batching, custom gpt_attention plugin, FP8/FP4 quantization |
| **Strength** | Best single-user latency; wins on B200 hardware |
| **Weakness** | High concurrency scaling inferior to vLLM |
| **Quantization** | FP8, FP4, W4A16, W4A8, AWQ, GPTQ |
| **Multi-GPU** | TP, PP, EP, Wide-EP, CP, DWDP |

---

*Last updated: 2026-04-21*
