# ExLlamaV2 & EXL2

> **GitHub:** [turboderp-org/exllamav2](https://github.com/turboderp-org/exllamav2) — 4.5k stars (archived)  
> **Successor:** [turboderp-org/exllamav3](https://github.com/turboderp-org/exllamav3) (EXL3 format)  
> **Docs:** [Convert Guide](https://github.com/turboderp-org/exllamav2/blob/master/doc/convert.md) · [Dynamic Generator](https://github.com/turboderp-org/exllamav2/blob/master/doc/dynamic.md)

---

## Overview

> *"EXL2 is a mixed-precision quantization format that allows fractional bits-per-weight (2-8 bits) with measurement-based per-layer optimization."* — [ExLlamaV2 GitHub](https://github.com/turboderp-org/exllamav2)

**ExLlamaV2** is a high-performance quantized inference library for consumer NVIDIA GPUs, using the **EXL2** format. It focuses on speed at low VRAM through weight-only quantization with per-layer measurement-based bit allocation.

> *"ExLlamaV2 is now archived. ExLlamaV3 with EXL3 format (based on QTIP) is the active project."*

---

## What is EXL2?

EXL2 (ExLlamaV2 Format) is a **mixed-precision quantization format** with key characteristics:

| Feature | Details |
|---------|---------|
| **Bit flexibility** | 2-8 fractional bpw (2.5, 3.5, 4.5, 5.5, etc.) |
| **Mixed precision** | Different layers can use different precision |
| **Measurement-based** | Tests multiple quantization parameters per layer against calibration data |
| **Weight-only** | W4A16: 4-bit weights, 16-bit activations |
| **File format** | Uses SafeTensors; same structure as source HF model |

### How EXL2 Differs from GGUF/QLoRA

| Aspect | EXL2 | GGUF | QLoRA |
|--------|------|------|-------|
| **Bit flexibility** | 2-8 fractional bpw | Fixed formats (Q4_K_M, Q6_K) | 4-bit only |
| **Mixed precision** | Per-layer optimization | Fixed per model | LoRA adapters on quantized base |
| **Framework** | ExLlamaV2 only | llama.cpp, Ollama, etc. | Training + inference |
| **Quality at same size** | Best-in-class | Good | Good for fine-tuning |
| **LoRA at inference** | ❌ Not supported | ❌ | ✅ Supported |

> *"MLA achieves superior expressive power with the same KV cache overhead as GQA."* — [TransMLA Paper](https://arxiv.org/abs/2502.07864) (EXL2 comparison context)

---

## Weight-Only Quantization

EXL2 is **weight-only quantization (W4A16)**:

> *"LLM inference bottlenecks on weight streaming from VRAM, not computation. Smaller weights = faster streaming."* — ExLlamaV2 docs

- **Memory-bandwidth bound:** Decode speed is limited by how fast weights can be loaded, not compute
- **Calibration required:** Measures quantization impact by comparing base vs quantized outputs on calibration data

---

## Tensor Parallelism (Multi-GPU)

> *"Added in 2024. `--gpu_split auto` auto-splits across all visible devices."* — ExLlamaV2 docs

| Aspect | Details |
|---------|---------|
| **Multi-GPU support** | ✅ Yes |
| **Configuration | `--gpu_split auto` |
| **vs llama.cpp** | llama.cpp only supports layer offloading, NOT true tensor parallelism |
| **Best use case** | Limited VRAM but not critically so |

> *"For maximum throughput: vLLM. For efficient quantized multi-GPU: ExLlamaV2."* — Community benchmarks

---

## MoE (Mixture-of-Experts) Support

> *"Supported since v0.0.11 (December 2023)."* — ExLlamaV2 changelog

| Model | Format | Notes |
|-------|--------|-------|
| Mixtral 8x7B | EXL2 4-bit, 5-bit | Available on HuggingFace |
| **Caution** | GPTQ-style MoE quantization | ~15% perplexity increase at low bpw |

---

## Performance Benchmarks

### Official ExLlamaV2 Performance (tokens/sec)

| Model | Mode | Size | RTX 3090Ti | RTX 4090 |
|-------|------|------|------------|----------|
| Llama | GPTQ | 7B | 181 t/s | 205 t/s |
| Llama | GPTQ | 13B | 110 t/s | 114 t/s |
| Llama | GPTQ | 33B | 44 t/s | 48 t/s |
| Llama2 | EXL2 3.0 bpw | 7B | 217 t/s | 257 t/s |
| Llama2 | EXL2 4.0 bpw | 7B | 185 t/s | 211 t/s |
| Llama2 | EXL2 2.5 bpw | 70B | 33 t/s | 38 t/s |
| TinyLlama | EXL2 3.0 bpw | 1.1B | 656 t/s | 770 t/s |

### vs llama.cpp (RTX 4090)

> *"EXL2 is 40-70% faster than llama.cpp for same model at same quantization."* — Community benchmarks

### Perplexity Comparison (lower is better, Llama 2 13B)

| Format | Perplexity |
|--------|------------|
| EXL2 4.650b | **4.32136** (best quality) |
| EXL2 4.000b | 4.37648 |
| Q4_K_M.gguf | 4.33326 |
| GPTQ 4-bit 128g | 4.35793 |

---

## VRAM Requirements

| Model | Format | VRAM |
|-------|--------|------|
| 7B | EXL2 4-bit | ~4-6 GB |
| 13B | EXL2 4-bit | ~8-10 GB |
| 70B | EXL2 2.5-bit | ~19-24 GB |
| Mixtral 8x7B | EXL2 5-bit | ~33 GB (A100) |

---

## Key Technical Features

| Feature | Details |
|---------|---------|
| **Paged attention** | Via Flash Attention 2.5.7+ |
| **Dynamic generator** | Continuous batching, prompt caching, KV cache deduplication |
| **Q4 KV cache** | 4-bit quantized KV cache for extended context |
| **Speculative decoding** | Uses smaller draft model to accelerate larger target |
| **AMD ROCm** | Community-maintained port |

---

## When to Use Each

| Engine | Best For |
|--------|----------|
| **llama.cpp** | CPU offloading, Mac/Windows, diverse hardware, portability |
| **ExLlamaV2** | Consumer NVIDIA GPU, maximum speed, EXL2 quantization |
| **vLLM** | Multi-GPU data center, high concurrency, API serving |

> *"EXL2 format is NOT natively supported in vLLM — vLLM supports AWQ, GPTQ, INT4, but not ExLlamaV2's EXL2 kernels."* — vLLM docs

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [turboderp-org/exllamav2](https://github.com/turboderp-org/exllamav2) | Archived — EXL2 format, 4.5k stars |
| [turboderp-org/exllamav3](https://github.com/turboderp-org/exllamav3) | Active — EXL3 format based on QTIP, HF Transformers plugin |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **GPTQ vs AWQ vs EXL2 vs llama.cpp** | Oobabooga Blog | [Link](https://oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/) |
| **ExLlamaV2: The Fastest Library to Run LLMs** | Towards Data Science | [Link](https://towardsdatascience.com/exllamav2-the-fastest-library-to-run-llms-32aeda294d26/) |
| **Quantization Methods Compared** | ai.rs | [Link](https://ai.rs/ai-developer/quantization-methods-compared/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Consumer NVIDIA GPU, maximum speed at low VRAM |
| **Key advantage** | Per-layer measurement-based bit allocation, fractional bpw |
| **Multi-GPU?** | ✅ Yes (tensor parallelism) |
| **LoRA at inference?** | ❌ No |
| **Successor** | ExLlamaV3 with EXL3 format |

---

*Last updated: 2026-04-21*
