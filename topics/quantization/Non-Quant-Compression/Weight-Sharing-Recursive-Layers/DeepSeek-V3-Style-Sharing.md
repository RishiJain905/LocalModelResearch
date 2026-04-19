# Weight Sharing / Recursive Layers — DeepSeek-V3 Style & Related

> **Key Papers:** [DeepSeek-V3 Technical Report](https://arxiv.org/abs/2412.19437) · [DeepSeek-V2 (MLA origin)](https://arxiv.org/abs/2405.04434) · [Relaxed Recursive Transformers ICLR 2025](https://arxiv.org/abs/2410.20672) · [Universal Transformers arXiv:1807.03819](https://arxiv.org/abs/1807.03819) · [Cross-Layer SVD Sharing arXiv:2410.03765](https://arxiv.org/abs/2410.03765)  
> **GitHub:** [deepseek-ai/DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) · [google-deepmind/relaxed-recursive-transformers](https://github.com/google-deepmind/relaxed-recursive-transformers)  
> **Blogs:** [VITALab Summary](https://vitalab.github.io/article/2025/02/11/DeepSeekV3.html) · [Gonzo ML](https://gonzoml.substack.com/p/deepseek-v3-technical-details) · [Verda/SGLang MLA Blog](https://verda.com/blog/deepseek-sglang-multi-head-latent-attention)

---

## Overview

> ⚠️ **Critical Clarification:** DeepSeek-V3 does **NOT** share weights between transformer layers. All 61 transformer layers have **unique weights**. DeepSeek-V3's efficiency comes from **architectural innovations** (MLA, MoE, MTP), not layer sharing.

This doc covers two related topics:
1. **DeepSeek-V3 Style Efficiency** — MLA (Multi-Head Latent Attention), MoE, MTP — architectural optimizations that reduce memory/compute
2. **True Weight Sharing** — Recursive/recursive layers that tie weights across transformer layers

| Aspect | DeepSeek-V3 (MLA/MoE/MTP) | True Layer Sharing |
|--------|---------------------------|-------------------|
| **Layer weights** | Unique per layer | Shared across layers |
| **Efficiency source** | Architectural: low-rank KV compression, sparse experts | Weight tying |
| **KV cache** | 93.3% reduction via MLA | Standard |
| **Activated params** | 8/256 experts per token | All layers active |

---

## Part 1: DeepSeek-V3 Efficiency Innovations

### 1. MLA — Multi-Head Latent Attention (DeepSeek-V2/V3)

> **Paper:** [DeepSeek-V2 arXiv:2405.04434](https://arxiv.org/abs/2405.04434) · **Blog:** [Verda/SGLang](https://verda.com/blog/deepseek-sglang-multi-head-latent-attention)

**Core Innovation:** Low-rank key-value joint compression — compresses K,V into a latent vector `c_t^{KV}` via down-projection `W_{DKV}`, reducing KV cache dramatically.

**KV Cache Comparison:**

| Attention Mechanism | KV Cache per Token | Capability |
|---------------------|-------------------|------------|
| MHA | `2*n_h*d_h*l` | Strong |
| GQA | `2*n_g*d_h*l` | Moderate |
| MQA | `2*d_h*l` | Weak |
| **MLA** | `(d_c + d_h^R)*l ≈ 9/2*d_h*l` | **Stronger than MHA** |

> *"Equipped with low-rank key-value joint compression, MLA achieves performance comparable to MHA while reducing KV cache by **93.3%**."*

**Weight Absorption Trick:** During inference, up-projection matrices `W_UK` and `W_UV` can be **absorbed** into `W_Q` and `W_O`, eliminating explicit K,V computation entirely.

```python
# Simplified absorption: W_Q_merged = W_Q @ W_UK
# No explicit K,V needs to be materialized
```

**Parameter Comparison:**

| Parameter | MLA | MHA (equivalent params) |
|-----------|-----|------------------------|
| #params/attention layer | **174M** | 470M |

---

### 2. DeepSeekMoE — Sparse Expert Mixture

> **From DeepSeek-V3 Technical Report**

- **1 shared expert** + **256 routed experts**, 8 activated per token
- **Auxiliary-loss-free load balancing:** Uses bias term added to affinity scores for routing only — dynamically adjusted to prevent expert collapse
- Achieves near-lossless performance with extreme sparsity

---

### 3. MTP — Multi-Token Prediction

> *"Predicts 2 tokens per position (current + next) sequentially."*

- Shares **embedding layer** and **output head** between main model and MTP modules
- Usable for **speculative decoding** — 85-90% acceptance rate, **1.8× TPS speedup**

---

### DeepSeek-V3 Benchmark Results

| Benchmark | DeepSeek-V3 | GPT-4o | Claude-3.5 |
|-----------|-------------|--------|------------|
| MMLU | **88.5** | 87.2 | 88.3 |
| MATH-500 | **90.2** | 74.6 | 78.3 |
| Arena-Hard | **85.5** | 80.4 | 85.2 |

**Training Cost:** 2.788M H800 GPU hours (~$5.576M total)

---

## Part 2: True Weight Sharing / Recursive Layers

### 1. Relaxed Recursive Transformers (Google DeepMind, ICLR 2025)

> **Paper:** [arXiv:2410.20672](https://arxiv.org/abs/2410.20672) · **GitHub:** [google-deepmind/relaxed-recursive-transformers](https://github.com/google-deepmind/relaxed-recursive-transformers)

**Core Innovation:** Converts N-layer model into K unique layers repeated N/K times, adds **layer-wise LoRA modules** to recover capacity lost from strict tying.

**Key Quote:**
> *"A relaxed Gemma model (Rank 512) recovers performance on par with the original Gemma 2B despite using fewer parameters and significantly less training data."*

**Tying Strategies:**

| Strategy | Description | Best For |
|----------|-------------|----------|
| **CYCLE** | Strict N/K repetition of K unique layers | Maximum sharing |
| **Stepwise** | Selects intermediate layers at intervals | Strict recursive |
| **Average** | Averages weight matrices of tied layers | Relaxed sharing |
| **Truncated SVD** | Initializes LoRA from residual matrices | Maximum recovery |

**Results:**
- Recursive Gemma 1B (with 15B tokens uptraining) outperforms TinyLlama 1.1B and Pythia 1B by **13.5 percentage points**
- Achieved **98.5%** of original Gemma 2B accuracy using **half the parameters**
- **2-3× throughput gains** via Continuous Depth-wise Batching

---

### 2. Universal Transformers (Google, 2018)

> **Paper:** [arXiv:1807.03819](https://arxiv.org/abs/1807.03819)

Early recursive transformer work — shares weights across all layers with position-wise recurrent modification.

**Key Idea:** Replaces fixed-depth stack of transformer layers with a stack that recycles the same parameters across depth, with a per-position dynamic halt mechanism.

---

### 3. Cross-Layer Parameter Sharing via SVD

> **Paper:** [arXiv:2410.03765](https://arxiv.org/abs/2410.03765)

Explores parameter sharing across different layers using SVD to achieve effective LLM compression.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **DeepSeek-V3** | 2024 | — | [arXiv](https://arxiv.org/abs/2412.19437) | MLA + MoE + MTP; 93.3% KV cache reduction |
| **DeepSeek-V2** | 2024 | — | [arXiv](https://arxiv.org/abs/2405.04434) | Origin of MLA, DeepSeekMoE, 42.5% cost savings |
| **Relaxed Recursive Transformers** | 2024 | ICLR 2025 | [arXiv](https://arxiv.org/abs/2410.20672) | Layer-wise LoRA + recursive layers; 98.5% recovery |
| **Universal Transformers** | 2018 | — | [arXiv](https://arxiv.org/abs/1807.03819) | Weight sharing across all layers with dynamic halting |
| **Cross-Layer SVD Sharing** | 2024 | — | [arXiv](2410.03765](https://arxiv.org/abs/2410.03765) | SVD-based cross-layer parameter sharing |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [deepseek-ai/DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) | Official implementation (103k stars) |
| [deepseek-ai/DeepSeek-V2](https://github.com/deepseek-ai/DeepSeek-V2) | DeepSeek-V2 origin of MLA |
| [google-deepmind/relaxed-recursive-transformers](https://github.com/google-deepmind/relaxed-recursive-transformers) | Official implementation of recursive + LoRA |
| [deepseek-ai/DeepSeek-V3](https://huggingface.co/deepseek-ai/DeepSeek-V3) | HuggingFace model weights (685B total) |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **DeepSeek-V3 Technical Report Breakdown** | VITALab | [Link](https://vitalab.github.io/article/2025/02/11/DeepSeekV3.html) |
| **DeepSeek-V3 Technical Details** | Gonzo ML | [Link](https://gonzoml.substack.com/p/deepseek-v3-technical-details) |
| **MLA Weight Absorption with Code** | Verda/SGLang | [Link](https://verda.com/blog/deepseek-sglang-multi-head-latent-attention) |
| **Build DeepSeek-V3 MLA from Scratch** | PyImageSearch | [Link](https://pyimagesearch.com/2026/03/16/build-deepseek-v3-multi-head-latent-attention-mla-architecture/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **DeepSeek-V3** | Does NOT share layer weights — uses MLA (93.3% KV cache reduction via low-rank compression), MoE (8/256 sparse experts), MTP (embedding/head sharing) |
| **True layer sharing** | Relaxed Recursive Transformers (Google DM): N→K unique layers + LoRA recovery; Universal Transformers: strict weight tying across all layers |
| **Key insight** | DeepSeek-V3 efficiency is architectural (MLA's weight absorption eliminates K,V materialization), not from layer sharing |
| **Best for** | DeepSeek-V3 style: KV-cache-constrained inference; True sharing: extreme parameter efficiency with fine-tuning recovery |

---

*Last updated: 2026-04-19*
