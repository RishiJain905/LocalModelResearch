# Inference-Serving

> Inference engines and serving infrastructure for LLM deployment — covering KV cache optimization, distributed orchestration, sub-quadratic runtimes, and kernel-level optimizations.

---

## Topics

| Topic | Description |
|-------|-------------|
| [Inference-Engines](./Inference-Engines/) | All research docs — APC/RadixAttention, Distributed KV-Cache, SSMs, Sparse MLA |

---

## Inference-Engines Docs

| Topic | Description |
|-------|-------------|
| [Automatic-Prefix-Caching-RadixAttention](./Inference-Engines/Automatic-Prefix-Caching-RadixAttention.md) | APC and RadixAttention for KV cache prefix reuse across requests |
| [Distributed-KV-Cache-Orchestration](./Inference-Engines/Distributed-KV-Cache-Orchestration.md) | Distributed KV-cache management, PD disaggregation, llm-d, Mooncake |
| [Sub-Quadratic-Hybrid-Runtimes-SSMs](./Inference-Engines/Sub-Quadratic-Hybrid-Runtimes-SSMs.md) | Mamba, Mamba-2, RWKV, RetNet, hybrid SSM-Transformer architectures |
| [Kernel-Level-Optimizations-Sparse-MLA](./Inference-Engines/Kernel-Level-Optimizations-Sparse-MLA.md) | DeepSeek MLA (Multi-head Latent Attention), FlashMLA kernels, DSA sparse attention |

---

## Key Themes

- **KV Cache Optimization** — Prefix caching (APC/RadixAttention), distributed orchestration (llm-d, Mooncake, LMCache), tiered memory
- **Sub-Quadratic Runtimes** — SSMs (Mamba family), linear recurrent models (RWKV, RetNet, xLSTM), hybrid architectures
- **Kernel Optimizations** — MLA low-rank compression, FlashMLA CUDA kernels, sparse attention (DSA/NSA)
- **Disaggregation** — Prefill/decode disaggregation (DistServe, TetriInfer, vLLM disagg), RDMA transfer

---

## Quick Reference

| System | Type | KV Cache Approach | Throughput Gain |
|--------|------|------------------|----------------|
| **vLLM + APC** | Serving | Hash-based block caching | 5-6× vs naive |
| **SGLang + RadixAttention** | Serving | Token-level radix tree | 6.4× vs vLLM |
| **llm-d** | Orchestration | Kubernetes-native KV-aware routing | Production scale |
| **Mooncake** | Production | KVCache-centric disaggregation | 525% vs coupled |
| **DeepSeek MLA** | Kernel | Low-rank KV compression | 57× vs MHA |
| **DSA (DeepSeek V3.2)** | Sparse | Lightning indexer top-k | 2× at 128K ctx |
| **FlashMLA** | Kernels | Optimized CUDA | 3000 GB/s |

---

*Last updated: 2026-04-21*
