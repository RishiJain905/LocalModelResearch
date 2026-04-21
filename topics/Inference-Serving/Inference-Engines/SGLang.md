# SGLang

> **GitHub:** [sgl-project/sglang](https://github.com/sgl-project/sglang) — LMSYS org  
> **Paper:** [SGLang arXiv:2312.07104](https://arxiv.org/abs/2312.07104) (NeurIPS 2024)  
> **Docs:** [docs.sglang.io](https://docs.sglang.io/) · [LMSYS Blog](https://lmsys.org/blog/)

---

## Overview

> *"SGLang achieves up to 6.4× higher throughput compared to state-of-the-art inference systems on various large language and multi-modal models."* — [SGLang Paper](https://arxiv.org/abs/2312.07104)

SGLang is an open-source inference framework from **LMSYS** (LMSYS org, non-profit) running on **400,000+ GPUs**, generating **trillions of tokens daily**. It combines RadixAttention, compressed finite state machines, and continuous batching for high-throughput structured LLM programs.

---

## Core Architecture

### RadixAttention (Prefix Caching)

> *"Automatically detects and reuses common prompt prefixes via a radix tree."* — [SGLang NeurIPS Paper](https://arxiv.org/abs/2312.07104)

KV caches stored in a **radix tree** (space-efficient prefix tree) with LRU eviction. Achieves dramatically higher cache hit rates than vLLM's block-based approach:

| Workload | vLLM PagedAttention | SGLang RadixAttention |
|----------|---------------------|------------------------|
| Few-Shot Learning | 15-25% | **85-95%** |
| Multi-Turn Chat | 10-20% | **75-90%** |
| Code Analysis | 5-15% | **60-80%** |

### Compressed Finite State Machines

Faster constrained decoding via FSM compression — reduces JSON/regex decoding overhead.

### Chunked Prefill

- Breaks long prompts (10K+ tokens) into 512-token chunks
- Prevents long prompts from blocking other requests

---

## 2025 Major Updates

| Feature | Description |
|---------|-------------|
| **DeepSeek-V3 Optimization** | FlashAttention-3, Dynamic MLA-to-MHA switching, DeepGEMM FP8 |
| **HiCache (Hierarchical KV Caching)** | GPU/CPU/Storage tiering — up to 6× throughput improvement, 84% TTFT reduction |
| **DeepGEMM** | FP8 matrix multiplication for MoE models |
| **DeepSeek-V3.2 Day-0** | Native Sparse Attention (DSA) with 128K context |
| **NVIDIA Model Optimizer** | Native integration — up to 2× per GPU throughput (W4AFp8) |
| **Deterministic Inference** | Batch-invariant kernels, 100% reproducible RL on Qwen3-8B |
| **Reinforcement Learning** | slime framework (Megatron-LM + SGLang); powers GLM 4.5/4.6 training |
| **Model Gateway** | Multi-model fleet management, health checks, load balancing |
| **OME (Kubernetes Operator)** | Enterprise-grade deployment orchestration |
| **macOS support** | Native macOS deployment |
| **SGLang-Diffusion** | Diffusion model support (Nov 2025) |

> *"HiCache hierarchical KV caching: up to 6× throughput improvement, 84% TTFT reduction via GPU/CPU/Storage tiering."* — [LMSYS Blog](https://lmsys.org/blog/2025-09-10-sglang-hicache/)

---

## FP8 & Quantization

| Mode | Description |
|------|-------------|
| **FP4/FP8/INT4** | Low-precision formats |
| **AWQ/GPTQ** | Weight-only quantization |
| **FP8 W8A8** | Weight and activation |
| **W4A8** | 4-bit weight, 8-bit activation |
| **DeepGEMM FP8** | Kernel fusion for MoE |

> *"NVIDIA Model Optimizer: up to 2× per GPU throughput comparing NVFP4 and FP8 vs FP8 baseline."* — [LMSYS Blog](https://lmsys.org/blog/2025-12-02-modelopt-quantization/)

---

## Benchmark Performance

### GPT-OSS-120B, 2× H100

> **Source:** [Clarifai Blog](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b)

| Concurrency | SGLang Throughput | vLLM Throughput | TRT-LLM Throughput |
|-------------|-------------------|-----------------|-------------------|
| 1 | 231 tok/s | 187 tok/s | **243 tok/s** |
| 10 | **988 tok/s** | 863 tok/s | 867 tok/s |
| **50** | **3,109 tok/s** | 2,212 tok/s | 2,163 tok/s |
| 100 | 3,222 tok/s | **4,742 tok/s** | 1,943 tok/s |

**Key takeaway:** SGLang excels at **moderate concurrency (50 requests)** — best throughput at that level.

---

## Multi-Modal Support

- Vision models (LLaVA and similar VLM architectures)
- Video understanding with temporal reasoning
- Multi-modal encoders with DP (Data Parallelism) optimization
- CUDA graphs for multi-modal encoder optimization

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **SGLang: Efficient Execution of Structured LLM Programs** | 2024 | NeurIPS | [arXiv](https://arxiv.org/abs/2312.07104) | RadixAttention, FSM decoding, 6.4× throughput gain |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [sgl-project/sglang](https://github.com/sgl-project/sglang) | Main repo — 11,793+ commits, 400K+ GPUs |
| [sgl-project/sglang/releases](https://github.com/sgl-project/sglang/releases) | 49 releases |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Fast and Expressive LLM Inference with RadixAttention** | LMSYS | [Link](https://lmsys.org/blog/2024-01-17-sglang/) |
| **Large-Scale EP Deployment (96 H100)** | LMSYS | [Link](https://lmsys.org/blog/2025-05-05-large-scale-ep/) |
| **HiCache Hierarchical KV Caching** | LMSYS | [Link](https://lmsys.org/blog/2025-09-10-sglang-hicache/) |
| **Deterministic Inference** | LMSYS | [Link](https://lmsys.org/blog/2025-09-22-sglang-deterministic/) |
| **DeepSeek-V3.2 Day-0 with Sparse Attention** | LMSYS | [Link](https://lmsys.org/blog/2025-09-29-deepseek-V32/) |
| **NVIDIA Model Optimizer Integration** | LMSYS | [Link](https://lmsys.org/blog/2025-12-02-modelopt-quantization/) |
| **SGLang-Diffusion: Two Months In** | LMSYS | [Link](https://lmsys.org/blog/2026-01-16-sglang-diffusion/) |
| **Comparing SGLang, vLLM, TensorRT-LLM** | Clarifai Blog | [Link](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Moderate concurrency (50 requests), few-shot/multi-turn workloads, structured generation |
| **Key innovation** | RadixAttention — token-level prefix tree for KV cache reuse |
| **Strength** | Best throughput at 50 concurrent requests; 85-95% cache hit on few-shot |
| **Quantization** | FP8, W4A8, AWQ, GPTQ, NVIDIA Model Optimizer |
| **Multi-GPU** | Expert Parallel (EP), tensor parallelism, HiCache tiered KV |

---

*Last updated: 2026-04-21*
