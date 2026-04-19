# QuIP, QuIP#, and QTIP

> Cornell RelaxML Lab quantization methods. Progressive improvements: **QuIP** (NeurIPS 2023) → **QuIP#** (ICML 2024) → **QTIP** (NeurIPS 2024 Spotlight)

---

## Papers

| Method | Paper | Venue | arXiv |
|--------|-------|-------|-------|
| **QuIP** | QuIP: 2-Bit Quantization of LLMs With Guarantees | NeurIPS 2023 | [2307.13304](https://arxiv.org/abs/2307.13304) |
| **QuIP#** | QuIP#: Even Better LLM Quantization with Hadamard Incoherence | ICML 2024 | [2402.04396](https://arxiv.org/abs/2402.04396) |
| **QTIP** | QTIP: Quantization with Trellises and Incoherence Processing | NeurIPS 2024 Spotlight | [2406.11235](https://arxiv.org/abs/2406.11235) |

---

## GitHub Repositories

| Method | Repo | Status |
|--------|------|--------|
| **QuIP** | [Cornell-RelaxML/QuIP](https://github.com/Cornell-RelaxML/QuIP) | Legacy |
| **QuIP#** | [Cornell-RelaxML/quip-sharp](https://github.com/Cornell-RelaxML/quip-sharp) | ⚠️ No longer actively developed |
| **QTIP** | [Cornell-RelaxML/qtip](https://github.com/Cornell-RelaxML/qtip) | Active successor |
| **YAQA** (2025) | [Cornell-RelaxML/yaqa](https://github.com/Cornell-RelaxML/yaqa) | Latest from same lab |

---

## Overview

> *"QuIP is the first post-training quantization (PTQ) method to achieve viable 2-bit quantization for LLMs."*

All three methods share a core insight: quantization benefits from **incoherent** weight and Hessian matrices — weights being even in magnitude and important rounding directions being unaligned with coordinate axes.

---

## QuIP: Quantization with Incoherence Processing

### Core Algorithm

**Two-step approach:**

1. **LDLQ (Adaptive Rounding)** — minimizes quadratic proxy objective `ℓ(Ŵ) = tr((Ŵ−W)H(Ŵ−W)ᵀ)`
2. **Incoherence Processing** — multiply W and H by random orthogonal matrices to spread quantization error evenly

### Key Result

> *"All quantization methods—even nearest—are viable at two bits with our incoherence processing."* — QuIP paper

---

## QuIP#: Hadamard Incoherence + E₈ Lattice Codebooks

### Three Key Innovations

| Innovation | Description | Improvement |
|------------|-------------|-------------|
| **Randomized Hadamard Transform (RHT)** | O(n log n) incoherence vs QuIP's O(n√n) | Faster, no FP multiplies |
| **E₈ Lattice Codebooks** | 8-dim vector quantization, 2-bit codebook with 256 entries | 4× smaller than K-means |
| **Fine-tuning scheme** | Per-block layer-by-layer optimization | Captures inter-layer interactions |

### E₈ Lattice Codebook

```
E₈ = (ℤ⁸ ∪ (ℤ⁸ + ½)) ∩ {x | 𝟏ᵀx is even}
```

- Based on optimal 8D sphere packing (Viazovska, 2017)
- Only **256 entries** stored as 4-bit integers (vs fp16 for K-means)
- **16-bit codeword:** 8 bits (entry) + 7 bits (sign flips) + 1 bit (±¼ shift)

### RHT vs Kronecker Factorization

| Property | QuIP (Kronecker) | QuIP# (RHT) |
|---|---|---|
| Incoherence bound μ_W | A²·log(4Cmn/δ)² | 2·log(4mn/δ) |
| Runtime | Θ(n√n) | Θ(n log n) |
| FP multiplies | Required | **None** (entries ∈ {-1,+1}) |

### Benchmark: QuIP# vs Other 2-bit Methods

| Method | Llama 2-7B Wiki2 ↓ | Llama 2-70B Wiki2 ↓ |
|--------|-------------------|---------------------|
| FP16 | 5.47 | 3.32 |
| OmniQuant 2-bit | 37.4 | 7.81 |
| AQLM ~2-bit | 6.93 | 3.94 |
| **QuIP# 2-bit** | **6.66** | **4.16** |

### Throughput (RTX 6000 Ada)

| Method | Llama 2-7B | Llama 2-70B |
|--------|-----------|-------------|
| FP16 | 57.7 tok/s | OOM |
| AQLM 2-bit | 81.1 tok/s | 8.72 tok/s |
| **QuIP# 2-bit** | **176 tok/s** | **21.9 tok/s** |

---

## QTIP: Trellis-Coded Quantization

> *"QTIP uses trellis-coded quantization (TCQ) to achieve ultra-high-dimensional (>100) quantization while maintaining fast inference."*

### Core Problem Solved

**Vector quantization (VQ) requires exponential codebook size** — VQ dimension is effectively hardware-limited to ≤8.

**TCQ solution:** Replace exponential codebook with a stateful trellis structure where optimal reconstruction is found via **Viterbi algorithm** — O(2^L × T) time, **linear in sequence length**.

### The Bitshift Trellis

> *"The top (L−kV) bits of j equal the bottom (L−kV) bits of i. Decoding only requires bitshifting by kV bits — decoding can be parallelized."*

### Distortion Comparison (2-bit, i.i.d. Gaussian)

| Method | Dimension | Distortion (MSE) |
|--------|-----------|------------------|
| Scalar (Lloyd-Max) | 1 | 0.118 |
| QuIP# E8P (VQ) | 8 | 0.089 |
| **QTIP 1MAD/3INST (TCQ)** | **256** | **0.069** |
| D_R (lower bound) | ∞ | 0.063 |

> *"TCQ reduces the distortion gap between QuIP# and an optimal 2-bit quantizer by over 3×!"*

### QTIP Throughput (Same Hardware)

| Method | Llama 2-7B | Llama 2-70B |
|--------|-----------|-------------|
| **QuIP# 2-bit** | 186 tok/s | 22.2 tok/s |
| **QTIP 2-bit** | **188 tok/s** | **23.5 tok/s** |

*QTIP achieves **same throughput** as QuIP# with higher quality.*

---

## YAQA: Yet Another Quantization Algorithm (2025)

- **Paper:** [arXiv:2505.22988](https://arxiv.org/abs/2505.22988)
- **Code:** [Cornell-RelaxML/yaqa](https://github.com/Cornell-RelaxML/yaqa)

> *"YAQA reduces the KL divergence to the original model by a factor of **1/3 over LDLQ/GPTQ** across a wide range of models and quantizers."*

Compatible with QTIP's quantizer.

---

## llama.cpp Integration

- **QuIP# 2-bit:** Supported in llama.cpp ([Reddit announcement](https://www.reddit.com/r/LocalLLaMA/comments/19anqbc/llamacpp_now_supports_quip_2bit_quant_mixtral_in/))
- **QTIP:** Discussion open at [ggml-org/llama.cpp#10125](https://github.com/ggml-org/llama.cpp/discussions/10125)

### Third-Party Implementation

- [QuIP-for-all](https://github.com/chu-tianxiang/QuIP-for-all) — unverified but reportedly works with vLLM, gpt-fast

---

## Technical Comparison

### VQ vs TCQ

| Aspect | Cost | VQ (QuIP#) | TCQ (QTIP) |
|--------|------|-------------|-------------|
| Quantization time | O(2^{kd} d) | Limited to d≤8 | **Linear in d** |
| Codebook storage | O(2^{kd} d) | Large | **None** (bitshift) |
| L1 cache fit | — | ❌ Barely | ✅ Yes |

### Summary Table

| Method | Year | Venue | Key Technique | Best Performance |
|--------|------|-------|---------------|-------------------|
| **QuIP** | 2023 | NeurIPS | Incoherence + LDLQ | First viable 2-bit PTQ |
| **QuIP#** | 2024 | ICML | RHT + E₈ lattice VQ | 176 tok/s (7B), 22 tok/s (70B) |
| **QTIP** | 2024 | NeurIPS Spotlight | TCQ + bitshift trellis | Better quality, same speed |
| **YAQA** | 2025 | preprint | Kronecker-factored Hessian | 1/3 KL divergence vs GPTQ |

---

## Pre-Quantized Models

HuggingFace collection: [relaxml/qtip-quantized-models](https://huggingface.co/collections/relaxml/qtip-quantized-models-66fa253ad3186746f4b62803)

---

## Blog Posts

- **Together AI Blog:** [Even Better, Even Faster: Quantized LLMs with QTIP](https://www.together.ai/blog/even-better-even-faster-quantized-llms-with-qtip)

---

*Last updated: 2026-04-19*
