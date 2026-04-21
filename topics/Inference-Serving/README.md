# Inference-Serving

> Inference engines and serving infrastructure for LLM deployment — covering KV cache optimization, distributed orchestration, sub-quadratic runtimes, kernel-level optimizations, and kernel-first backends.

---

## Topics

| Topic | Description |
|-------|-------------|
| [Inference-Engines](./Inference-Engines/) | All inference engine research docs |

---

## Inference-Engines Docs

### High Throughput / Concurrency (Throughput-First)

| Topic | Description |
|-------|-------------|
| [vLLM](./Inference-Engines/vLLM.md) | PagedAttention, continuous batching, v1 re-architecture, 4,742 tok/s @ 100 concurrency |
| [SGLang](./Inference-Engines/SGLang.md) | RadixAttention, 6.4× throughput, HiCache, 3,109 tok/s @ 50 concurrency |
| [TensorRT-LLM](./Inference-Engines/TensorRT-LLM.md) | NVIDIA Inflight Batching, FP8/FP4, Blackwell optimization, best single-request throughput |

### Kernel-First Backends

| Topic | Description |
|-------|-------------|
| [llama.cpp-GGUF](./Inference-Engines/llama.cpp-GGUF.md) | Plain C/C++, GGUF format, Q4_K_M/Q5_K_M, imatrix, 10 backends, zero dependencies |
| [ExLlamaV2-EXL2](./Inference-Engines/ExLlamaV2-EXL2.md) | EXL2 mixed-precision (2-8 fractional bpw), 40-70% faster than llama.cpp, tensor parallelism |
| [MLC-LLM](./Inference-Engines/MLC-LLM.md) | TVM compilation, WebGPU/WASM/iOS/Android/Metal, dynamic shapes, 3× faster on embedded |

### KV Cache & Orchestration

| Topic | Description |
|-------|-------------|
| [Automatic-Prefix-Caching-RadixAttention](./Inference-Engines/Automatic-Prefix-Caching-RadixAttention.md) | APC and RadixAttention for KV cache prefix reuse |
| [Distributed-KV-Cache-Orchestration](./Inference-Engines/Distributed-KV-Cache-Orchestration.md) | llm-d, Mooncake, LMCache, PD disaggregation |

### Hardware Native / Edge Runtimes

| Topic | Description |
|-------|-------------|
| [MLX](./Inference-Engines/MLX.md) | Apple Silicon unified memory, Metal backend, M5 Neural Accelerators, mlx-lm |
| [NPU-Native-Runtimes](./Inference-Engines/NPU-Native-Runtimes.md) | OpenVINO (Intel NPU), Qualcomm Hexagon QNN, edge NPU benchmarks |
| [Transformers.js](./Inference-Engines/Transformers.js.md) | Browser/WebGPU/WASM LLM, ONNX-based, v4 C++ runtime, privacy-first |

### Local Abstraction & Management Layers

| Topic | Description |
|-------|-------------|
| [Ollama](./Inference-Engines/Ollama.md) | Go/llama.cpp wrapper, zero-config CLI, OpenAI-compatible API, auto hardware detection |
| [LMStudio](./Inference-Engines/LMStudio.md) | Desktop GUI + CLI, EXL2/GGUF/MLX, in-app HF search, llmster headless deploy |

### Sub-Quadratic Runtimes

| Topic | Description |
|-------|-------------|
| [Sub-Quadratic-Hybrid-Runtimes-SSMs](./Inference-Engines/Sub-Quadratic-Hybrid-Runtimes-SSMs.md) | Mamba, RWKV, RetNet, xLSTM, hybrid SSM-Transformer |
| [Kernel-Level-Optimizations-Sparse-MLA](./Inference-Engines/Kernel-Level-Optimizations-Sparse-MLA.md) | DeepSeek MLA (57× KV compression), FlashMLA, DSA sparse attention |

---

## Quick Reference: Throughput Engines

| Engine | Best For | Throughput @ 100 | Single-User Speed |
|--------|----------|------------------|-------------------|
| **vLLM** | High concurrency, data center | **4,742 tok/s** | Fast TTFT |
| **SGLang** | Moderate concurrency, few-shot | 3,222 tok/s | Good |
| **TensorRT-LLM** | Single-request, Blackwell | 1,943 tok/s | **Fastest per-request** |

## Quick Reference: Kernel-First Backends

| Engine | Format | Best For | Multi-GPU |
|--------|--------|----------|-----------|
| **llama.cpp** | GGUF | Edge, CPU offload, Mac/Windows | ❌ |
| **ExLlamaV2** | EXL2 | Consumer NVIDIA, max speed | ✅ |
| **MLC LLM** | TVM-compiled | WebGPU, WASM, iOS, Android | ✅ |

---

*Last updated: 2026-04-21*
