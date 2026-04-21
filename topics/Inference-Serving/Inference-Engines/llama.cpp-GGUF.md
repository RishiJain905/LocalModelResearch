# llama.cpp & GGUF

> **GitHub:** [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) — 105k stars  
> **Papers:** [Which Quantization arXiv:2601.14277](https://arxiv.org/abs/2601.14277) · [Mind the Gap arXiv:2505.23786](https://arxiv.org/abs/2505.23786)  
> **Docs:** [GGUF Spec](https://github.com/ggerganov/ggml/blob/master/docs/gguf.md) · [HuggingFace Guide](https://huggingface.co/docs/hub/gguf) · [llama.cpp Wiki](https://github.com/ggml-org/llama.cpp/wiki)

---

## Overview

> *"llama.cpp is plain C/C++ with zero external dependencies, designed for consumer hardware with first-class Apple Silicon support."* — [llama.cpp README](https://github.com/ggml-org/llama.cpp)

**llama.cpp** is a plain C/C++ inference engine with no external dependencies, supporting GGUF model files across CPU, GPU (CUDA, Metal, Vulkan, OpenCL, HIP, ROCm), and even WebGPU. It is the dominant format for local/edge LLM deployment.

**GGUF (GPT-Generated Unified Format)** is a self-contained binary format encoding both tensors and standardized metadata (tokenizer, architecture config). It evolved from GGML in August 2023 and supports memory-mapped execution (`mmap`) for fast loading.

---

## Supported Backends

| Backend | Target |
|---------|--------|
| **CUDA** | NVIDIA GPUs |
| **HIP** | AMD GPUs |
| **Metal** | Apple Silicon |
| **Vulkan** | Cross-vendor GPU |
| **SYCL** | Intel/NVIDIA GPUs |
| **OpenCL** | Qualcomm Adreno, Intel GPUs |
| **OpenVINO** | Intel CPUs, GPUs, NPUs |
| **MUSA** | Moore Threads GPUs |
| **CANN** | Ascend NPUs |
| **BLAS** | CPU acceleration |
| **WebGPU** | In progress |
| **RPC** | Distributed inference |

> *"CUDA Graphs now enabled by default for batch_size=1 on NVIDIA GPUs — up to 1.2× speedup on H100."* — [NVIDIA Blog](https://developer.nvidia.com/blog/optimizing-llama-cpp-ai-inference-with-cuda-graphs/)

---

## Quantization Methods

### K-Quant (Recommended)

| Type | Size (7B) | PPL Change | Notes |
|------|-----------|------------|-------|
| **Q2_K** | 2.63GB | +0.6717 | Lowest quality, extreme compression |
| Q3_K_S | 2.75GB | +0.5551 | Very small, high quality loss |
| Q3_K_M | 3.07GB | +0.2496 | Good for severe constraints |
| **Q4_K_M** | 3.80GB | +0.0532 | **Recommended — best balance** |
| Q4_K_S | 3.59GB | +0.0992 | Slightly smaller, more quality loss |
| Q5_K_S | 4.33GB | +0.0400 | Low quality loss |
| **Q5_K_M** | 4.45GB | +0.0122 | **Recommended — higher quality** |
| Q6_K | 5.15GB | +0.0008 | Near-original quality |
| Q8_0 | 6.70GB | +0.0004 | Best quality, largest size |

> *"Always prefer K-quant over legacy formats (Q4_0, Q4_1, Q5_0, Q5_1)."* — [llama.cpp quantize README](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md)

### I-Quant (Importance-Based)

| Type | Description |
|------|-------------|
| IQ2_XXS, IQ2_XS, IQ2_S | 2-bit importance-weighted |
| IQ3_XXS, IQ3_S, IQ3_M | 3-bit importance-weighted |
| IQ4_NL, IQ4_XS | 4-bit non-linear |

### How K-Quantization Works

> *"Weights split into blocks (typically 32-64 elements). Each block has its own scale AND zero point. Non-uniform quantization: more levels near zero where weights cluster."* — [Medium](https://medium.com/@paul.ilvez/demystifying-llm-quantization-suffixes-what-q4-k-m-q8-0-and-q6-k-really-mean-0ec2770f17d3)

---

## Importance Matrix (imatrix) Calibration

> *"Calibration data derived by running a model over representative text — records which weights/activations matter most for output quality."* — [llama.cpp imatrix docs](https://github.com/ggml-org/llama.cpp/blob/master/examples/imatrix/README.md)

```bash
# Compute importance matrix
./llama-imatrix -m model.gguf -f calibration_data.txt -o imatrix.gguf

# Use during quantization
./llama-quantize --imatrix imatrix.gguf input.gguf output.gguf Q4_K_M
```

> *"Highly recommended for quantization levels below Q5_K_M."*

---

## Multi-GPU Support

> *"llama.cpp has LIMITED multi-GPU tensor parallelism — only layer offloading and CPU offloading."* — [ahmadosman.com](https://www.ahmadosman.com/blog/do-not-use-llama-cpp-or-ollama-on-multi-gpus-setups-use-vllm-or-exllamav2/)

| Setup | Speedup |
|-------|---------|
| 2× RTX 4090 (PCIe 4.0) | ~1.4-1.5× |
| 2× RTX 4090 (NVLink) | ~1.6-1.7× |

**Recommendation:** Use **vLLM** or **ExLlamaV2** for proper tensor parallelism.

---

## Performance: llama.cpp vs vLLM

> *"llama.cpp is the undisputed champion for edge optimization, distributed inference, and GGUF execution."* — [DecodesFuture](https://developers.redhat.com/articles/2025/09/30/vllm-or-llamacpp-choosing-right-llm-inference-engine-your-use-case)

| Metric | vLLM | llama.cpp |
|--------|------|-----------|
| **Multi-user throughput** | 35× higher at peak load | Flat, single-user focused |
| **TTFT at high concurrency** | Stable, near-flat | Exponential growth |
| **ITL at high concurrency** | Lower and stable | Lower at high load |
| **Best for** | Production multi-user | Single-user, edge, batch |

> *"vLLM's PagedAttention manages KV cache by partitioning into non-contiguous blocks, reducing memory consumption by 19-27%."* — [Red Hat Developer](https://developers.redhat.com/articles/2025/09/30/vllm-or-llamacpp-choosing-right-llm-inference-engine-your-use-case)

---

## Papers Table

| Paper | Year | URL | Key Contribution |
|-------|------|-----|-----------------|
| **Which Quantization Should I Use?** | 2025 | [arXiv:2601.14277](https://arxiv.org/abs/2601.14277) | Unified evaluation of llama.cpp quantization on Llama-3.1-8B-Instruct |
| **Mind the Gap: Practical Attack on GGUF Quantization** | 2025 | [arXiv:2505.23786](https://arxiv.org/abs/2505.23786) | Security analysis of GGUF quantization |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) | Main repo — 105k stars, C/C++, zero dependencies |
| [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) | Georgi Gerganov's original fork |
| [ggerganov/ggml](https://github.com/ggerganov/ggml) | GGML tensor library (precursor to ggml-org) |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Optimizing llama.cpp with CUDA Graphs** | NVIDIA Blog | [Link](https://developer.nvidia.com/blog/optimizing-llama-cpp-ai-inference-with-cuda-graphs/) |
| **Understanding the GGUF Format** | Medium | [Link](https://medium.com/@vimalkansal/understanding-the-gguf-format-a-comprehensive-guide-67de48848256) |
| **Demystifying LLMs: Quantization Suffixes** | Medium | [Link](https://medium.com/@paul.ilvez/demystifying-llm-quantization-suffixes-what-q4-k-m-q8-0-and-q6-k-really-mean-0ec2770f17d3) |
| **GGUF Format Deep Dive** | mbrenndoerfer | [Link](https://mbrenndoerfer.com/writing/gguf-format-quantized-llm-storage-inference) |
| **GGUF Weekend Deep Dive** | tlbflush | [Link](http://tlbflush.org/post/2025_02_17_gguf_weekend/) |
| **vLLM or llama.cpp: Choosing the Right LLM Inference Engine** | Red Hat Developer | [Link](https://developers.redhat.com/articles/2025/09/30/vllm-or-llamacpp-choosing-right-llm-inference-engine-your-use-case) |
| **Do NOT Use llama.cpp on Multi-GPU Setups** | ahmadosman.com | [Link](https://www.ahmadosman.com/blog/do-not-use-llama-cpp-or-ollama-on-multi-gpus-setups-use-vllm-or-exllamav2/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Single-user, edge, CPU+GPU offload, portable apps, Mac/Windows |
| **Best quantization** | Q4_K_M (balance) or Q5_K_M (quality) |
| **Multi-GPU?** | Use vLLM or ExLlamaV2 instead |
| **imatrix** | Use for < Q5_K_M quantization |
| **Key advantage** | Zero dependencies, widest hardware support, GGUF ecosystem |

---

*Last updated: 2026-04-21*
