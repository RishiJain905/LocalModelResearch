# MLC LLM

> **GitHub:** [mlc-ai/mlc-llm](https://github.com/mlc-ai/mlc-llm) · [mlc-ai/web-llm](https://github.com/mlc-ai/web-llm)  
> **Docs:** [llm.mlc.ai](https://llm.mlc.ai/docs/) · [Introduction](https://llm.mlc.ai/docs/get_started/introduction)  
> **Papers:** [Relax arXiv:2311.02103](https://arxiv.org/html/2311.02103v2) · [TensorIR arXiv:2207.04296](https://arxiv.org/abs/2207.04296) · [TVM OSDI 2018](https://arxiv.org/abs/1802.04799)

---

## Overview

> *"MLC LLM is a machine learning compiler and high-performance deployment engine for LLMs, built on Apache TVM Unity. It compiles and runs models via MLCEngine — a unified inference engine across all supported platforms."* — [MLC LLM Blog](https://blog.mlc.ai/2024/06/07/universal-LLM-deployment-engine-with-ML-compilation)

**MLC LLM** (Machine Learning Compilation for Large Language Models) uses **compilation-based inference** — compiling model-specific optimized kernels via Apache TVM — rather than hand-written kernels. This enables native deployment across CPU, GPU, WebGPU, WebAssembly, iOS, and Android from a single codebase.

---

## Supported Backends

| Platform | Backends |
|----------|----------|
| **NVIDIA GPU** | CUDA, Vulkan |
| **AMD GPU** | ROCm, Vulkan |
| **Apple GPU** | Metal |
| **Intel GPU** | Vulkan |
| **Web Browser** | WebGPU, WebAssembly |
| **iOS** | Metal |
| **Android** | OpenCL (Adreno/Mali) |
| **Linux/Windows** | Vulkan, CUDA |

---

## How It Works: TVM/MLC Compilation Flow

```
1. Model Definition → TVM nn.Module or HuggingFace import
         ↓
2. TVM Relax IR → Graph-level representation with symbolic shapes
         ↓
3. Quantization → Weight-only or weight-activation quantization
         ↓
4. Optimization Passes:
   - LegalizeOps (convert to TensorIR)
   - AnnotateTIROpPattern (tag/groups functions)
   - FoldConstant (pre-compute constants)
   - FuseOps/FuseTIR (kernel fusion)
         ↓
5. TensorIR → Loop-level tensor program abstraction (ASPLOS 2023)
         ↓
6. Code Generation → CUDA/Metal/Vulkan/WebGPU/OpenCL
         ↓
7. MLCEngine Runtime → OpenAI-compatible API (REST/Python/JS/iOS/Android)
```

> *"TensorIR: Automatic Tensorized Program Optimization" — [ASPLOS 2023](https://arxiv.org/abs/2207.04296)

---

## Quantization Formats

MLC LLM uses **group quantization** based on *The case for 4-bit precision* (arXiv:2212.09720) and *LUT-GEMM* (arXiv:2206.09557).

**Format code:** `qAfB(_id)` where `A` = weight bits, `B` = activation bits

| Format | Description |
|--------|-------------|
| `q0f16`, `q0f32` | No quantization (FP16/FP32) |
| `q3f16_1` | 3-bit weight-only |
| `q4f16_1` | 4-bit weight-only |
| `q4f32_1` | 4-bit weight-only (FP32 activations) |
| `q4f16_awq` | AWQ quantization |
| `e4m3_e4m3_f16` | FP8 weight-activation (CUDA only) |
| `e5m2_e5m2_f16` | FP8 weight-activation (CUDA only) |

---

## Dynamic Shape Handling

MLC LLM handles dynamic sequence lengths via **TVM Relax** — a graph-level IR with **first-class symbolic shapes**:

> *"Symbolic variables (e.g., `_vars["n"]`) represent unknown dimensions, enabling global tracking of shape relations across operators and static analysis before concrete shapes are known."* — [Relax Paper](https://arxiv.org/html/2311.02103v2)

This is critical for autoregressive decoding where KV-cache length varies at runtime.

---

## Performance vs llama.cpp

| Metric | MLC LLM | llama.cpp |
|--------|---------|-----------|
| **Decode speed (Jetson)** | ~3× faster | Baseline |
| **Overall (RTX 3090 Ti)** | ~51 tok/s | ~46 tok/s (Ollama) |
| **Approach** | Compiler optimization | Hand-crafted kernels |

> *"MLC compiles model-specific optimized kernels; llama.cpp uses hand-written kernels for each operation."* — MLC Blog

---

## Performance Benchmarks

| Model | Format | Hardware | Throughput |
|-------|--------|---------|------------|
| Llama2-70B | 4-bit | 2× RTX 4090 | 34.5 tok/sec |
| CodeLlama-34B | 4-bit | 2× RTX 4090 | 64 tok/sec |
| Llama2-70B | 4-bit | AMD 7900 XTX | ~85% of RTX 4090 |

> *"Scales to 8× A10G GPUs for larger deployments."* — MLC Blog

---

## Key Papers

| Paper | Year | URL | Key Contribution |
|-------|------|-----|-----------------|
| **Relax: Composable Abstractions for End-to-End Dynamic ML** | 2023 | [arXiv](https://arxiv.org/html/2311.02103v2) | TVM Relax IR for symbolic shapes |
| **TensorIR: Automatic Tensorized Program Optimization** | 2023 | [arXiv](https://arxiv.org/abs/2207.04296) | ASPLOS 2023 — loop-level tensor optimization |
| **TVM: Automated End-to-End Optimizing Compiler** | 2018 | [arXiv](https://arxiv.org/abs/1802.04799) | OSDI 2018 — TVM foundation |
| **Production-Grade Local LLM Inference on Apple Silicon** | 2025 | [arXiv:2511.05502](https://arxiv.org/abs/2511.05502) | MLC on Apple Silicon benchmarks |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [mlc-ai/mlc-llm](https://github.com/mlc-ai/mlc-llm) | Main MLC LLM — compilation-based LLM inference |
| [mlc-ai/web-llm](https://github.com/mlc-ai/web-llm) | WebGPU/WebAssembly LLM in browser |
| [mlc-ai/mlc-llm/tree/main/python/tvm** | TVM Unity integration |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Universal LLM Deployment Engine with ML Compilation** | MLC Blog | [Link](https://blog.mlc.ai/2024/06/07/universal-LLM-deployment-engine-with-ML-compilation) |
| **Multi-GPU Inference on NVIDIA/AMD GPUs** | MLC Blog | [Link](https://blog.mlc.ai/2023/10/19/Scalable-Language-Model-Inference-on-Multiple-NVDIA-AMD-GPUs) |
| **Bringing LLMs to Consumer Devices** | MLC Blog | [Link](https://blog.mlc.ai/2023/05/22/bringing-open-large-language-models-to-consumer-devices) |
| **MLC LLM Documentation** | llm.mlc.ai | [Link](https://llm.mlc.ai/docs/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Cross-platform deployment (WebGPU, WASM, iOS, Android, Metal), compilation optimization |
| **Key advantage** | Model-specific kernel compilation via TVM, dynamic shapes, widest platform support |
| **vs llama.cpp** | ~3× faster decode on embedded; compiler-generated vs hand-written kernels |
| **Quantization** | q4f16_1, q3f16_1, q4f16_awq, FP8 (CUDA) |

---

*Last updated: 2026-04-21*
