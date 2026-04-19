# Weight Sharing / Recursive Layers — DeepSeek-V3 Style & Related

> **Key Papers:** [DeepSeek-V3](https://arxiv.org/abs/2412.19437) · [DeepSeek-V2 (MLA origin)](https://arxiv.org/abs/2405.04434) · [Relaxed Recursive Transformers ICLR 2025](https://arxiv.org/abs/2410.20672) · [Universal Transformers ICLR 2019](https://arxiv.org/abs/1807.03819) · [Mixture-of-Recursions arXiv:2507.10524](https://arxiv.org/abs/2507.10524) · [Cross-Layer SVD Sharing](https://arxiv.org/abs/2410.03765)  
> **GitHub:** [deepseek-ai/DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) · [ahmedtaha100/RRT](https://github.com/ahmedtaha100/RRT) · [raymin0223/mixture_of_recursions](https://github.com/raymin0223/mixture_of_recursions)  
> **Blogs:** [VITALab Summary](https://vitalab.github.io/article/2025/02/11/DeepSeekV3.html) · [Gonzo ML](https://gonzoml.substack.com/p/deepseek-v3-technical-details) · [Verda/SGLang MLA Blog](https://verda.com/blog/deepseek-sglang-multi-head-latent-attention) · [Ajith Prabhakar](https://ajithp.com/2024/10/29/recursive-transformers/) · [Glass Box Medicine](https://glassboxmedicine.com/2019/09/07/universal-transformers/)

---

## Overview

> ⚠️ **Critical Clarification:** DeepSeek-V3 does **NOT** share weights between transformer layers. All 61 transformer layers have **unique weights**. DeepSeek-V3's efficiency comes from **architectural innovations** (MLA, MoE, MTP), not layer sharing.

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

## Part 2: Relaxed Recursive Transformers (Google DeepMind, ICLR 2025)

> **Paper:** [arXiv:2410.20672](https://arxiv.org/abs/2410.20672) · **OpenReview:** [WwpYSOkkCt](https://openreview.net/forum?id=WwpYSOkkCt) · **ICLR Poster:** [29334](https://iclr.cc/virtual/2025/poster/29334) · **Google DeepMind:** [Publication 122290](https://deepmind.google/research/publications/122290/)  
> **GitHub:** [ahmedtaha100/RRT](https://github.com/ahmedtaha100/RRT) (JAX/Flax) · [avramdj/rrt-lora](https://github.com/avramdj/rrt-lora) (PyTorch community port)  
> **Authors:** Sangmin Bae, Adam Fisch, Hrayr Harutyunyan, Ziwei Ji, Seungyeon Kim, Tal Schuster (KAIST + Google DeepMind)

### Core Innovation

> *"We revisit 'layer tying' as form of parameter sharing in Transformers, and introduce novel methods for converting existing LLMs into smaller 'Recursive Transformers' that share parameters across layers, with minimal loss of performance... We show that our recursive models (e.g., recursive Gemma 1B) outperform both similar-sized vanilla pretrained models (such as TinyLlama 1.1B and Pythia 1B) and knowledge distillation baselines—and can even recover most of the performance of the original 'full-size' model."*

Converts an N-layer Transformer into a **single block of K unique layers repeated in a loop**, with layer-wise LoRA modules to recover capacity lost from strict tying.

**CYCLE Formula:**
```
h_t^ℓ = f(h_t^{ℓ-1}; Φ'_{((ℓ-1) mod L/B)+1})
```
Where B = number of looping blocks. All parameters tied including linear weights and RMSNorm.

**Example:** Gemma 2B (18 layers) → 2 blocks of 9 unique layers, each looped twice.

---

### Layer Initialization Methods

| Method | Description | Best For |
|--------|-------------|----------|
| **Stepwise** | Selects intermediate layers at intervals, keeps first/last fixed | Strict Recursive Transformers |
| **Average** | Averages weights across tied layers | Relaxed Recursive Transformers (with LoRA) |
| **Lower** | Uses first K layers directly | General fallback |
| **Truncated SVD** | Initializes LoRA from residual matrices between original and shared weights | Maximum recovery |

> *"SVD initialization provides up to **6.5 percentage points** improvement over zero initialization."*

---

### Relaxed Recursive Transformer with Layer-wise LoRA

Adds layer-specific LoRA modules to relax strict parameter sharing:

```
h_t^ℓ = f(h_t^{ℓ-1}; Φ'_{((ℓ-1) mod L/B)+1}, ΔΦ'_ℓ)
```

For transformation `h = W'x`:
```
h = W'x + ΔW'x = W'x + BAx
```
Where A ∈ ℝ^(r×k), B ∈ ℝ^(d×r), rank r controls relaxation degree:
- **Rank = 0** → pure Recursive Transformer
- **Rank = full** → recovers original vanilla Transformer

---

### Continuous Depth-wise Batching (CDB)

A new inference paradigm exploiting recursive structure. Since parameters are shared across loop iterations, different iterations for different samples can be batched together.

> *"We theoretically show that this can enable dynamic sample scheduling at various depths for significant throughput gains (for up to **2.76× speedup**)."*

Combined with **early-exiting (EE)**, tokens exit at the earliest loop where prediction matches final output.

---

### Benchmarks

#### Performance vs. Baselines

| Model | Params | Training Tokens | Avg Few-shot Accuracy |
|-------|--------|-----------------|----------------------|
| Gemma 2B (full) | 1.99B | 3T (original) | 61.7% |
| Gemma 2B (uptrained) | 1.99B | 60B | 58.6% |
| **Recursive Gemma 1B (Stepwise)** | 0.99B | 60B + KD | 54.9% |
| **Relaxed Gemma 1.6B (Avg, r=512)** | 1.60B | 60B + KD | **58.4%** ← matches full! |
| TinyLlama 1.1B | 0.97B | 105B | 43.3% |
| Pythia 1B | 0.81B | 300B | 48.8% |

**Key Achievements:**
- Recursive Gemma 1B outperforms TinyLlama 1.1B and Pythia 1B by **+13.5 percentage points**
- Relaxed Gemma (r=512) matches full Gemma 2B performance (58.4% vs 58.6%) with **fewer parameters**
- Achieved with only **2% of original training data** (60B vs 3T tokens)

#### Throughput Gains (Theoretical + Measured)

| Model | Batching | Accuracy | Throughput Speedup |
|-------|----------|----------|-------------------|
| Gemma 2B (vanilla) | None | 57.3% | ×1.00 |
| Gemma 2B | CSB | 57.3% | ×1.41 |
| **Recursive Gemma 1B** | **CDB+EE** | **54.0%** | **×2.66** |
| Relaxed Gemma 1.07B (r=64) | CDB+EE | 54.0% | ×2.00 |
| Relaxed Gemma 1.6B (r=512) | CDB+EE | 56.2% | ×1.59 |

#### Knowledge Distillation

- Teacher = full-size model uptrained on 15B tokens
- **Forward KL divergence** performed best among FKL, RKL, JSD, TVD
- Combined extended uptraining (60B) + KD: **+4.1 percentage points** improvement

---

## Part 3: Universal Transformers (Google, ICLR 2019)

> **Paper:** [arXiv:1807.03819](https://arxiv.org/abs/1807.03819) · **OpenReview:** [HyzdRiR9Y7](https://openreview.net/forum?id=HyzdRiR9Y7) · **Google Blog:** [Link](https://research.google/blog/moving-beyond-translation-with-the-universal-transformer/)  
> **Authors:** Mostafa Dehghani, Stephan Gouws, Oriol Vinyals, Jakob Uszkoreit, Łukasz Kaiser (Google Brain + University of Amsterdam)

### Core Innovation

> *"Instead of recurring over the individual symbols of sequences like RNNs, the Universal Transformer repeatedly revises its representations of all symbols in the sequence with each recurrent step."*

**Problem:** Standard Transformers fail to generalize on tasks requiring iterative/recursive processing (e.g., copying strings, logical inference beyond training lengths). They are also not computationally universal.

**Solution:** Combines parallelizability of Transformers with recurrent inductive bias. Instead of recurring over sequence positions like RNNs, it **recurs over depth**.

**Equivalence:** When running for a fixed number of steps, the Universal Transformer is equivalent to a multi-layer Transformer with **tied parameters** across all layers.

---

### Architecture

#### Encoder Computation (at step t):
```
Aᵗ = LayerNorm(Hᵗ⁻¹ + MultiHeadSelfAttention(Hᵗ⁻¹ + Pᵗ))
Hᵗ = LayerNorm(Aᵗ + Transition(Aᵗ))
```

Where:
- **Self-Attention:** Same as standard Transformer (multi-head attention with Q,K,V projections)
- **Transition function:** Either position-wise fully-connected layer (ReLU between affine transforms) OR separable convolution
- **Coordinate embeddings Pᵗ:** 2D embeddings combining position and time-step:
  - `Pᵗₚₒₛ,₂ⱼ = sin(pos/10000²ʲ/ᵈ) ⊕ sin(t/10000²ʲ/ᵈ)`
  - `Pᵗₚₒₛ,₂ⱼ₊₁ = cos(pos/10000²ʲ/ᵈ) ⊕ cos(t/10000²ʲ/ᵈ)`

---

### ACT — Adaptive Computation Time (Halting Mechanism)

Each position dynamically determines number of refinement steps via a "pondering" value:

> *"We further employ an adaptive computation time (ACT) mechanism to allow the model to dynamically adjust the number of times the representation of each position is revised."*

- Halting unit (sigmoidal) computes probability that computation should stop
- Once a position halts, its state is copied forward until all blocks halt or max steps reached
- Enables interpolation between fixed-depth Transformer and gated recurrent architecture

---

### Results

#### bAbI Question-Answering (State-of-the-Art):

| Model | 10K single | 10K joint | 1K single | 1K joint |
|-------|-----------|----------|-----------|----------|
| Transformer | 15.2 (10/20) | 22.1 (12/20) | 21.8 (5/20) | 26.8 (14/20) |
| **Universal Transformer** | **0.23 (0/20)** | **0.47 (0/20)** | 5.31 (5/20) | 8.50 (8/20) |
| **Adaptive UT** | **0.21 (0/20)** | **0.29 (0/20)** | **4.56 (3/20)** | **7.85 (5/20)** |

#### LAMBADA Language Modeling (State-of-the-Art):

| Model | LM Perplexity | RC Accuracy |
|-------|---------------|-------------|
| Transformer | 9725 | 0.3988 |
| LSTM | 5174 | 0.2007 |
| **Universal Transformer** | **319** | **0.5216** |
| **Adaptive UT** | **142** | **0.5625** |

#### Subject-Verb Agreement:
Universal Transformer (99.17%) significantly outperforms vanilla Transformer (96.16%)

---

### Turing Completeness

> *"With sufficient memory, the Universal Transformer is computationally universal (can simulate any Turing machine)."*

---

## Part 4: Follow-up Work

### Mixture-of-Recursions (MoR) — Google DeepMind + KAIST (arXiv:2507.10524)

> **Paper:** [arXiv:2507.10524](https://arxiv.org/abs/2507.10524) · **OpenReview:** [QuqsEIVWIG](https://openreview.net/forum?id=QuqsEIVWIG) · **GitHub:** [raymin0223/mixture_of_recursions](https://github.com/raymin0223/mixture_of_recursions)

Combines parameter sharing with adaptive token-level computation:
- Lightweight routers dynamically assign different recursion depths to individual tokens
- Focuses quadratic attention computation only among tokens still active at given recursion depth
- Selectively caches only key-value pairs for active tokens

### Sparse Universal Transformer (SUT) — 2023

> **Paper:** [arXiv:2310.07096](https://arxiv.org/abs/2310.07096)

Expresses both FFN and multi-head attention as sparse MoEs. Matches/exceeds BLEU scores of dense UT on WMT'14 EN-DE with **~50% less computation**.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **DeepSeek-V3** | 2024 | — | [arXiv](https://arxiv.org/abs/2412.19437) | MLA + MoE + MTP; 93.3% KV cache reduction |
| **DeepSeek-V2** | 2024 | — | [arXiv](https://arxiv.org/abs/2405.04434) | Origin of MLA, DeepSeekMoE, 42.5% cost savings |
| **Relaxed Recursive Transformers** | 2024 | ICLR 2025 | [arXiv](https://arxiv.org/abs/2410.20672) | N→K layers + LoRA recovery; 98.5% performance with half params; CDB for 2.76× speedup |
| **Universal Transformers** | 2018 | ICLR 2019 | [arXiv](https://arxiv.org/abs/1807.03819) | Weight tying + ACT dynamic halting; Turing complete |
| **Mixture-of-Recursions** | 2025 | — | [arXiv](2507.10524](https://arxiv.org/abs/2507.10524) | Token-level adaptive recursion depth + routers |
| **Cross-Layer SVD Sharing** | 2024 | — | [arXiv](https://arxiv.org/abs/2410.03765) | SVD-based cross-layer parameter sharing |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [deepseek-ai/DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) | Official (103k stars) |
| [deepseek-ai/DeepSeek-V2](https://github.com/deepseek-ai/DeepSeek-V2) | DeepSeek-V2 origin of MLA |
| [ahmedtaha100/RRT](https://github.com/ahmedtaha100/RRT) | Relaxed Recursive Transformers (JAX/Flax, official) |
| [avramdj/rrt-lora](https://github.com/avramdj/rrt-lora) | Community PyTorch port of RRT |
| [raymin0223/mixture_of_recursions](https://github.com/raymin0223/mixture_of_recursions) | MoR implementation |
| [andreamad8/Universal-Transformer-Pytorch](https://github.com/andreamad8/Universal-Transformer-Pytorch) | Universal Transformer PyTorch |
| [tensorflow/tensor2tensor](https://github.com/tensorflow/tensor2tensor) | Official TensorFlow/Tensor2Tensor UT |
| [deepseek-ai/DeepSeek-V3](https://huggingface.co/deepseek-ai/DeepSeek-V3) | HuggingFace model weights |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **DeepSeek-V3 Technical Report Breakdown** | VITALab | [Link](https://vitalab.github.io/article/2025/02/11/DeepSeekV3.html) |
| **DeepSeek-V3 Technical Details** | Gonzo ML | [Link](https://gonzoml.substack.com/p/deepseek-v3-technical-details) |
| **MLA Weight Absorption with Code** | Verda/SGLang | [Link](https://verda.com/blog/deepseek-sglang-multi-head-latent-attention) |
| **Build DeepSeek-V3 MLA from Scratch** | PyImageSearch | [Link](https://pyimagesearch.com/2026/03/16/build-deepseek-v3-multi-head-latent-attention-mla-architecture/) |
| **Relaxed Recursive Transformers Explained** | Ajith Prabhakar | [Link](https://ajithp.com/2024/10/29/recursive-transformers/) |
| **Universal Transformers Explained** | Glass Box Medicine | [Link](https://glassboxmedicine.com/2019/09/07/universal-transformers/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **DeepSeek-V3** | Does NOT share layer weights — uses MLA (93.3% KV cache reduction via low-rank compression), MoE (8/256 sparse experts), MTP (embedding/head sharing) |
| **Relaxed Recursive Transformers** | Converts N→K unique layers repeated N/K times + layer-wise LoRA (rank r); Truncated SVD init; CDB for 2.76× speedup; 98.5% recovery with half params |
| **Universal Transformers** | Strict weight tying across all layers + ACT dynamic halting; Turing complete; excels at iterative/recursive tasks |
| **Key insight** | DeepSeek-V3 efficiency is architectural (MLA's weight absorption), not from layer sharing |
| **Best for** | DeepSeek-V3 style: KV-cache-constrained inference; RRT: parameter efficiency with fine-tuning recovery; UT: theoretical universality, iterative reasoning |

---

*Last updated: 2026-04-19*
