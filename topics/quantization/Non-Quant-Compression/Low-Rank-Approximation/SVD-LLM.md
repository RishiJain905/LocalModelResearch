# SVD-LLM — Truncation-Aware Singular Value Decomposition

> **Papers:** [SVD-LLM (ICLR 2025)](https://arxiv.org/abs/2403.07378) · [SVD-LLM V2 (NAACL 2025)](https://arxiv.org/abs/2503.12340) · [Dobi-SVD (ICLR 2025)](https://arxiv.org/abs/2502.02723) · [AdaSVD](https://arxiv.org/abs/2502.01403) · [ASVD](https://github.com/hahnyuan/ASVD4LLM)  
> **GitHub:** [AIoT-MLSys-Lab/SVD-LLM](https://github.com/AIoT-MLSys-Lab/SVD-LLM) · [ZHITENGLI/AdaSVD](https://github.com/ZHITENGLI/AdaSVD) · [wangqinsi1/Dobi-SVD](https://github.com/wangqinsi1/Dobi-SVD)  
> **Blogs:** [Hugging Face — SVD Training](https://huggingface.co/blog/fractalego/svd-training)

---

## Overview

> *"Singular Value Decomposition (SVD) offers a promising solution for LLM compression. However, state-of-the-art SVD-based LLM compression methods have two key limitations: truncating smaller singular values may lead to higher compression loss, and the lack of update on the compressed weights after SVD truncation."* — SVD-LLM [Paper](https://arxiv.org/abs/2403.07378)

SVD-LLM applies low-rank matrix factorization to compress LLM weight matrices by decomposing them into products of smaller matrices, reducing parameter count and computational cost.

| Aspect | Details |
|--------|---------|
| **Method** | Truncated SVD with truncation-aware data whitening |
| **Key Innovation** | Direct mapping: compression loss = truncated singular value |
| **Fine-tuning** | Sequential LoRA on decomposed matrices |
| **Supported Models** | LLaMA, LLaMA-2, LLaMA-3, Mistral, OPT, Vicuna |

---

## Key Methods

### 1. SVD-LLM — Truncation-Aware SVD (ICLR 2025)

> **Paper:** [arXiv:2403.07378](https://arxiv.org/abs/2403.07378) · **Code:** [AIoT-MLSys-Lab/SVD-LLM](https://github.com/AIoT-MLSys-Lab/SVD-LLM)  
> **Authors:** Xin Wang, Yu Zheng, Zhongwei Wan, Mi Zhang (OSU / Michigan State)

**Core Problem:** Existing SVD methods have two limitations:
1. Truncating smaller singular values may lead to *higher* compression loss (no direct mapping)
2. No parameter update after truncation

**Two Key Contributions:**

1. **Truncation-Aware Data Whitening:** Enforces whitened activation S⁻¹X to be orthonormal via Cholesky decomposition, establishing direct mapping: `Lᵢ = σᵢ` (compression loss equals truncated singular value)

2. **Sequential Low-Rank Approximation:** Fine-tunes decomposed matrices W'ᵤ and W'ᵥ separately with LoRA to compensate for accuracy degradation

**Pipeline (3 Steps):**
```
1. Truncation-Aware Data Whitening + SVD Compression
   W' = U × Trunc.(Σ) × Vᵀ × S⁻¹
2. Sequential LoRA Fine-tuning
   Fine-tune W'ᵤ then W'ᵥ separately
3. Optional: GPTQ Integration
   Combine with quantization for further compression
```

**Benchmark Results (LLaMA-7B, WikiText-2 PPL):**

| Ratio | SVD | FWSVD | ASVD | **SVD-LLM** |
|-------|-----|-------|------|-------------|
| 20% | 20061 | 1727 | 11.14 | **7.73** |
| 40% | 52489 | 18156 | 1407 | **9.27** |
| 60% | 105474 | 32194 | 57057 | **15.00** |
| 80% | 687291 | 96872 | 80425 | **31.79** |

---

### 2. SVD-LLM V2 — Heterogeneous Compression (NAACL 2025)

> **Paper:** [arXiv:2503.12340](https://arxiv.org/abs/2503.12340) · Same repo

**Core Problem:** SVD-LLM suffers from:
1. Homogeneous compression ratios across layers
2. Numerical instability from Cholesky decomposition

**Two Key Innovations:**

1. **Heterogeneous Compression Ratio Allocation:** Groups matrices by type (Q, K, G, U), computes theoretical minimum truncation loss, allocates ratios per matrix

2. **Loss-Optimized Weight Truncation:** Two-round SVD replaces Cholesky — achieves theoretical minimum truncation loss without positive-definite requirement

> *"Different weight matrices have different redundancy levels. Layer 1 query matrix has much lower truncation loss than Layer 27 in LLaMA-3 8B."*

**Benchmark Results:**
- LLaMA-3 8B WikiText-2 PPL: 11.82 → **8.01** (↓32%)
- LLaMA-7B WikiText-2 PPL: 7.94 → **7.12** (↓10%)
- Inference speedup: up to **2.71×** at 80% compression

---

### 3. Dobi-SVD — Differentiable SVD (ICLR 2025)

> **Paper:** [arXiv:2502.02723](https://arxiv.org/abs/2502.02723) · **Code:** [wangqinsi1/Dobi-SVD](https://github.com/wangqinsi1/Dobi-SVD)  
> **Authors:** Qinsi Wang et al. (Duke, UC Berkeley)

**Core Innovation:** First SVD method achieving high-ratio compression (40%) while maintaining competitive performance. Key insight: **truncate activations, not weights**.

**Results:**
- 78.7% improvement over prior SOTA at 40% compression on LLaMA-7B
- **12.4× speedup** on 12GB Titan Xp GPUs
- LLaMA-7B WikiText2 PPL: 9.95 at 40% ratio (vs SVD-LLM's 53.74)

---

### 4. AdaSVD — Adaptive SVD

> **Paper:** [arXiv:2502.01403](https://arxiv.org/abs/2502.01403) · **Code:** [ZHITENGLI/AdaSVD](https://github.com/ZHITENGLI/AdaSVD)  
> **Authors:** Zhiteng Li et al. (SJTU, ETH Zürich)

**Two Key Methods:**
1. **adaComp:** Alternately updates singular matrices U and V⊤ to compensate for truncation errors using Moore-Penrose pseudoinverse
2. **adaCR:** Adaptive layer-specific compression ratios based on cosine similarity importance

**Benchmark Results (LLaMA2-7B, 60% compression):**
- AdaSVD: 50.33 PPL vs SVD-LLM: 89.90 PPL (↓44%)

---

## Related: SVD-Based Fine-Tuning

### SVD-Training (LoRA-style)

> **Blog:** [Hugging Face — SVD Training](https://huggingface.co/blog/fractalego/svd-training) · **PyPI:** `svd-training`

Similar to LoRA but ~100× fewer trainable parameters. Only trains singular values Σ; U and V remain frozen.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **SVD-LLM** | 2024 | ICLR 2025 | [arXiv](https://arxiv.org/abs/2403.07378) | Truncation-aware whitening + sequential LoRA |
| **SVD-LLM V2** | 2025 | NAACL 2025 | [arXiv](https://arxiv.org/abs/2503.12340) | Heterogeneous ratios, loss-optimized truncation |
| **Dobi-SVD** | 2025 | ICLR 2025 | [arXiv](https://arxiv.org/abs/2502.02723) | Truncate activations not weights, 12.4× GPU speedup |
| **AdaSVD** | 2025 | — | [arXiv](https://arxiv.org/abs/2502.01403) | Adaptive compression ratios + pseudoinverse update |
| **ASVD** | — | — | [GitHub](https://github.com/hahnyuan/ASVD4LLM) | Activation-aware SVD, sensitivity-based rank selection |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [AIoT-MLSys-Lab/SVD-LLM](https://github.com/AIoT-MLSys-Lab/SVD-LLM) | SVD-LLM & V2 implementations |
| [ZHITENGLI/AdaSVD](https://github.com/ZHITENGLI/AdaSVD) | Adaptive SVD compression |
| [wangqinsi1/Dobi-SVD](https://github.com/wangqinsi1/Dobi-SVD) | Differentiable SVD |
| [hahnyuan/ASVD4LLM](https://github.com/hahnyuan/ASVD4LLM) | Activation-aware SVD |
| [fractalego/svd-training](https://github.com/fractalego/svd-training) | SVD-based fine-tuning (PyPI: `svd-training`) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Compresses LLM weight matrices via low-rank factorization with minimal perplexity degradation |
| **Key innovation** | Truncation-aware whitening establishes direct mapping between singular values and compression loss |
| **Best for** | Memory/compute reduction; works with LoRA fine-tuning and GPTQ quantization |
| **SVD-LLM V2 insight** | Different layers have different redundancy — heterogeneous ratios outperform uniform |

---

*Last updated: 2026-04-19*
