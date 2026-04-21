# Inference-Serving

> Inference engines and serving infrastructure for LLM deployment — covering KV cache optimization, distributed orchestration, sub-quadratic runtimes, kernel-level optimizations, and kernel-first backends.

---

## Topics

| Topic | Description |
|-------|-------------|
| [Inference-Engines](./Inference-Engines/) | All inference engine research docs |

---

## Inference-Engines Docs

| Topic | Description |
|-------|-------------|
| [llama.cpp-GGUF](./Inference-Engines/llama.cpp-GGUF.md) | Plain C/C++ inference, GGUF format, Q4_K_M/Q5_K_M quantization, imatrix calibration |
| [ExLlamaV2-EXL2](./Inference-Engines/ExLlamaV2-EXL2.md) | EXL2 mixed-precision quantization, 40-70% faster than llama.cpp, tensor parallelism |
| [MLC-LLM](./Inference-Engines/MLC-LLM.md) | Compilation-based inference via Apache TVM, WebGPU/WASM/iOS/Android, dynamic shapes |
| [Automatic-Prefix-Caching-RadixAttention](./Inference-Engines/Automatic-Prefix-Caching-RadixAttention.md) | APC and RadixAttention for KV cache prefix reuse |
| [Distributed-KV-Cache-Orchestration](./Inference-Engines/Distributed-KV-Cache-Orchestration.md) | llm-d, Mooncake, LMCache, PD disaggregation |
| [Sub-Quadratic-Hybrid-Runtimes-SSMs](./Inference-Engines/Sub-Quadratic-Hybrid-Runtimes-SSMs.md) | Mamba, RWKV, RetNet, xLSTM, hybrid SSM-Transformer |
| [Kernel-Level-Optimizations-Sparse-MLA](./Inference-Engines/Kernel-Level-Optimizations-Sparse-MLA.md) | DeepSeek MLA (57× KV compression), FlashMLA, DSA sparse attention |

---

## Key Themes

- **Kernel-First Backends** — llama.cpp (GGUF, widest hardware), ExLlamaV2 (EXL2, best quality/speed on consumer NVIDIA), MLC LLM (TVM compilation, cross-platform)
- **KV Cache Optimization** — Prefix caching (APC/RadixAttention), distributed orchestration (llm-d, Mooncake, LMCache), tiered memory
- **Sub-Quadratic Runtimes** — SSMs (Mamba family), linear recurrent models (RWKV, RetNet, xLSTM), hybrid architectures
- **Kernel Optimizations** — MLA low-rank compression, FlashMLA CUDA kernels, sparse attention (DSA/NSA)
- **Disaggregation** — Prefill/decode disaggregation (DistServe, TetriInfer, vLLM disagg), RDMA transfer

---

## Quick Reference: Kernel-First Backends

| Engine | Format | Best For | Multi-GPU | Key Advantage |
|--------|--------|----------|-----------|---------------|
| **llama.cpp** | GGUF | Edge, CPU offload, Mac/Windows, portability | ❌ | Zero dependencies, widest hardware |
| **ExLlamaV2** | EXL2 | Consumer NVIDIA, max speed | ✅ | 40-70% faster than llama.cpp, per-layer measurement |
| **MLC LLM** | TVM-compiled | WebGPU, WASM, iOS, Android, Metal | ✅ | Compilation optimization, dynamic shapes |
| **vLLM** | AWQ/GPTQ/FP8 | Data center, high concurrency | ✅ | PagedAttention, 35× throughput vs single-user |

---

## Quick Reference: KV Cache & Orchestration

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
