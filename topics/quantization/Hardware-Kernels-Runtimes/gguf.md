# GGUF: llama.cpp's Quantization Format

> **Main Repo:** [ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) · **Stars:** 104k+ · **License:** MIT  
> **Format Spec:** [huggingface.co/docs/hub/gguf](https://huggingface.co/docs/hub/gguf) · [Technical Guide](http://tlbflush.org/post/2025_02_17_gguf_weekend/)  
> **K-Quants Guide:** [Kaitchup Substack](https://kaitchup.substack.com/p/choosing-a-gguf-model-k-quants-i)  
> **I-Quants Guide:** [Kaitchup Substack](https://kaitchup.substack.com/p/the-i-quants-an-improved-version)

---

## Overview

> *"GGUF = GPT-Generated Unified Format — a binary format optimized for mmap, storing tensors and metadata in a single file. It is the de facto standard for CPU and Apple Silicon LLM inference."* — [HF GGUF docs](https://huggingface.co/docs/hub/gguf)

**GGUF** (GPT-Generated Unified Format) is llama.cpp's quantization format and inference engine. It is the most widely supported format for local LLM inference, especially on **CPU and Apple Silicon**.

**Key characteristics:**
- C/C++ with zero dependencies
- Apple Silicon first-class: ARM NEON, Accelerate, Metal backend
- CUDA, HIP (AMD), Vulkan, SYCL backends for GPU
- Supports 1.5-bit through 8-bit quantization
- 104k+ GitHub stars — largest LLM quantization ecosystem

---

## Quantization Types

### K-Quants (Default, Block-based)

K-quants use **double-quantization** (quantizing the quantization scales) and **asymmetric encoding** for better quality per bit than simple formats.

| Type | Bits/Weight | Size (7B) | Quality | Recommendation |
|------|------------|-----------|---------|---------------|
| **Q8_0** | 8-bit | ~7GB | ~99.5% | Near-lossless baseline |
| **Q6_K** | 6.56-bit | ~5.5GB | ~98% | "Almost lossless" |
| **Q5_K_M** | 5.5-bit | ~4.8GB | ~97% | Recommended high-quality |
| **Q5_K_S** | 5.5-bit | ~4.7GB | ~96% | Slightly smaller than M |
| **Q4_K_M** | 4.5-bit | ~4.1GB | ~95% | **Most popular, recommended default** |
| **Q4_K_S** | 4.5-bit | ~4.0GB | ~94% | Low RAM systems |
| **Q4_K_XS** | 4.25-bit | ~3.9GB | ~93% | Smaller than S |
| **Q3_K_M** | 3.44-bit | ~3.5GB | ~90% | Aggressive compression |
| **Q3_K_S** | 3.37-bit | ~3.4GB | ~89% | — |
| **Q2_K** | 2.63-bit | ~2.9GB | ~85% | Extreme compression |

**Suffix meanings:**
- `_S` = Small — mostly 4-bit, only sensitive tensors get higher
- `_M` = Medium — selective 5-6 bit for sensitive tensors
- `_L` = Large — more tensors at higher precision

### I-Quants (Importance-Matrix Quantized)

I-quants use an **importance matrix (imatrix)** to preserve precision where it matters most. Better quality per bit than K-quants, but require an imatrix for best results.

| Type | Bits/Weight | Quality | Notes |
|------|------------|---------|-------|
| **IQ4_XS** | 4.25 | Better than Q4_K_M | Best quality/size ratio |
| **IQ4_NL** | 4.5 | High | CPU-friendly, non-linear mapping |
| **IQ3_S** | 3.06 | Better than Q3_K | — |
| **IQ3_M** | 3.2 | Better than Q3_K | — |
| **IQ3_XXS** | 2.77 | Better than Q2_K | — |
| **IQ2_XXS** | 2.06 | Usable | Very low bit |
| **IQ2_XS** | 2.25 | Usable | Better than XXS |
| **IQ2_S** | 2.5 | Usable | — |
| **IQ2_M** | 2.8 | Good | — |
| **IQ1_S** | 1.56 | Experimental | 1.5-bit |
| **IQ1_M** | 1.75 | Experimental | 1.75-bit |

### Special Types

| Type | Bits/Weight | Notes |
|------|------------|-------|
| **TQ1_0** | ~1.6 | Ternary quantization, for very large models |
| **TQ2_0** | ~1.7 | Ternary quantization |
| **MXFP4** | 4 | Microscaling Block Floating Point (NVIDIA collaboration) |

### Quick Selection Guide

| Use Case | Recommended | Alternative |
|----------|-------------|-------------|
| **Best quality** | Q5_K_M | Q6_K |
| **Best balance** | Q4_K_M | IQ4_XS |
| **Low RAM** | Q4_K_S | Q3_K_M |
| **Extreme compression** | Q3_K_M | IQ3_S |
| **Very low RAM** | Q2_K | IQ2_M |

---

## Apple Silicon Benchmarks

> **Source:** [llama.cpp Discussion #4167](https://github.com/ggml-org/llama.cpp/discussions/4167)

### Token Generation Speed (tokens/second, Metal GPU, Q4_0)

| Chip | Bandwidth (GB/s) | Prompt Proc | Token Gen |
|------|-----------------|-------------|----------|
| **M1 Pro** | 200 | 266 t/s | 36 t/s |
| **M1 Max** | 400 | 530 t/s | 61 t/s |
| **M2** | 100 | 180 t/s | 22 t/s |
| **M2 Pro** | 200 | 294 t/s | 38 t/s |
| **M2 Max** | 400 | 537 t/s | 61 t/s |
| **M2 Ultra** | 800 | 1014 t/s | 89 t/s |
| **M3 Max** | 400 | 568 t/s | 57 t/s |
| **M3 Ultra** | 800 | 1073 t/s | 88 t/s |
| **M4 Max** | 546 | 886 t/s | 83 t/s |
| **M4 Max (40-core)** | 546 | 922 t/s | 92 t/s |
| **M4 Ultra** | 800+ | ~1092 t/s | ~88 t/s |

### Key Insights

> *"Token generation is memory-bandwidth bound — scales linearly with bandwidth. Prompt processing is compute-bound — F16 can be faster than quantized."*

| Observation | Detail |
|------------|--------|
| **Generation** | Memory-bandwidth bound → scales with bandwidth |
| **Prompt processing** | Compute-bound → F16 can be faster than quantized |
| **Cores** | Use only p-cores for CPU inference: `-t <p-core-count>` |
| **Bandwidth scaling** | Linear with bandwidth until hitting compute ceiling |

---

## GGUF vs Other Formats

### vs EXL2 (ExLlamaV2)

| Aspect | **GGUF** | **EXL2** |
|--------|---------|---------|
| **Throughput** | 16–18 tok/s | **28 tok/s** (~1.5×) |
| **VRAM (13B Q4)** | 7.2GB | 5.1GB |
| **Quality loss** | 2–4% | <1% (4.25bpw) |
| **Tool support** | Ollama, llama.cpp, LM Studio, GPT4All | ExLlamaV2 only |
| **Setup complexity** | Easiest | Hardest |
| **HF availability** | 95%+ of quantized models | ~5% |

### vs AWQ / GPTQ

| Aspect | **GGUF** | **AWQ** | **GPTQ** |
|--------|---------|--------|---------|
| **Quality** | Good | Best (~1–2% loss) | Good (~3–5% loss) |
| **Speed** | CPU-optimal | NVIDIA-optimal | NVIDIA-optimal |
| **Ecosystem** | Largest | Growing | Established |
| **Hardware** | CPU, Apple Silicon, NVIDIA | NVIDIA | NVIDIA |

> **Community finding:** EXL2 4-bit and imatrix IQ quants (GGUF) have "exactly the same" quality at same model size for Llama 3.

---

## Tools for GGUF Quantization

### llama.cpp CLI

```bash
# Convert HF model to GGUF FP16
python3 convert_hf_to_gguf.py ./models/mymodel/

# Quantize to Q4_K_M
./llama-quantize input-f16.gguf output-Q4_K_M.gguf Q4_K_M

# Generate importance matrix for IQ types
./llama-imatrix -m input.gguf -f corpus.txt -o imatrix.gguf

# Quantize with imatrix (recommended for IQ)
./llama-quantize --imatrix imatrix.gguf input.gguf output-IQ4_XS.gguf IQ4_XS
```

### Online (No Setup)

> **HF Spaces:** [huggingface.co/spaces/ggml-org/gguf-my-repo](https://huggingface.co/spaces/ggml-org/gguf-my-repo)

Quantize any HuggingFace model to GGUF without installing anything.

### Popular Curators

Models on HuggingFace pre-quantized to GGUF: **TheBloke**, **Bartowski**, **MaziyarPanahi**, **Unsloth**

### DASLab GGUF Toolkit

> **GitHub Discussion:** [llama.cpp/discussions/16035](https://github.com/ggml-org/llama.cpp/discussions/16035)

GPTQ + EvoPress → GGUF heterogeneous quantization.

---

## Backends

| Backend | Hardware | Notes |
|---------|----------|-------|
| **Metal** | Apple Silicon | First-class GPU support |
| **CUDA** | NVIDIA | Good support |
| **HIP** | AMD | ROCm support |
| **Vulkan** | General GPU | Cross-vendor |
| **SYCL** | Intel, AMD | Cross-vendor |
| **CPU (NEON)** | ARM | High performance |
| **CPU (BLAS)** | x86 | Accelerate framework |

---

## GitHub & Documentation

| Resource | URL |
|----------|-----|
| **llama.cpp GitHub** | [github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp) |
| **GGUF Format Spec** | [huggingface.co/docs/hub/gguf](https://huggingface.co/docs/hub/gguf) |
| **GGUF Technical Guide** | [tlbflush.org/post/2025_02_17_gguf_weekend/](http://tlbflush.org/post/2025_02_17_gguf_weekend/) |
| **Quantize Tool Docs** | [github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md](https://github.com/ggml-org/llama.cpp/blob/master/tools/quantize/README.md) |
| **Apple Silicon Benchmarks** | [github.com/ggml-org/llama.cpp/discussions/4167](https://github.com/ggml-org/llama.cpp/discussions/4167) |
| **K-Quants Guide** | [kaitchup.substack.com/p/choosing-a-gguf-model-k-quants-i](https://kaitchup.substack.com/p/choosing-a-gguf-model-k-quants-i) |
| **I-Quants Guide** | [kaitchup.substack.com/p/the-i-quants-an-improved-version](https://kaitchup.substack.com/p/the-i-quants-an-improved-version) |
| **HF Models (GGUF)** | [huggingface.co/models?library=gguf](https://huggingface.co/models?library=gguf) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Type** | Quantization format + inference engine |
| **Language** | C/C++ — zero dependencies |
| **Hardware** | CPU, Apple Silicon, NVIDIA, AMD, Vulkan, Intel |
| **Quantization types** | K-quants (Q2–Q8), I-quants (IQ), ternary |
| **Best quality** | Q5_K_M, Q6_K |
| **Best balance** | Q4_K_M, IQ4_XS |
| **Apple Silicon speed** | Up to ~92 tok/s on M4 Max (40-core) |
| **Best for** | CPU/Apple Silicon deployment, maximum compatibility, easiest setup |

---

*Last updated: 2026-04-19*
