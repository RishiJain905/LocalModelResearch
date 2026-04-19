# RSI — Randomized Subspace Iteration for Low-Rank Approximation

> **Foundational:** [Halko et al. 2011 — SIAM Review](https://epubs.siam.org/doi/10.1137/090771806) · [Martinsson et al. 2013 — SIMAX](https://epubs.siam.org/doi/10.1137/130938700)  
> **LLM Application:** [RSI for LLMs (arXiv:2604.02659)](https://arxiv.org/abs/2604.02659) · [GaLore++ (arXiv:2412.19820)](https://arxiv.org/abs/2412.19820) · [PRAC (arXiv:2602.23111)](https://arxiv.org/abs/2602.23111)  
> **GitHub:** [jiaweizzhao/GaLoRe](https://github.com/jiaweizzhao/galore) · [stefan-zobel/rSVD](https://github.com/stefan-zobel/rSVD)

---

## Overview

> *"RSVD is inadequate, and we propose randomized subspace iteration (RSI) as a more effective alternative."* — Pourkamali-Anaraki, [arXiv:2604.02659](https://arxiv.org/abs/2604.02659)

Randomized Subspace Iteration (RSI) is a **randomized numerical linear algebra** technique for efficient low-rank matrix approximation. For LLMs specifically, standard randomized SVD (RSVD) fails when singular value spectra decay slowly — RSI with power iterations (q>1) amplifies spectral separation to recover accuracy.

| Aspect | Details |
|--------|---------|
| **Method** | Randomized SVD with power iterations |
| **Key Innovation** | Power iterations (q>1) amplify large singular values vs. small ones |
| **Speedup** | 76× faster than exact SVD for VGG19 at k=200 |
| **Complementary to** | LoRA — compress backbone first, then adapt |

---

## Core Algorithm

### RSI Algorithm (from arXiv:2604.02659)

```
Require: Weight matrix W ∈ ℝ^{C×D}, target rank k, iteration count q ≥ 1
1: Draw random matrix Ω ∈ ℝ^{D×k}, set Y = Ω
2: for t = 1, ..., q do
3:   X = WY
4:   [X, _] = qr(X)
5:   Y = WᵀX
6: end for
7: [Û, Ŝ, Ṽ] = svd(Yᵀ)
8: Ǔ = XÛ
9: return Ǔ, Ŝ, Ṽ
```

### Why Power Iterations Matter

For **q=1**, RSI reduces to standard RSVD. For **q>1**, the algorithm amplifies large singular values by factors of **σ^{2q-1}** while suppressing smaller ones.

- **Slow decay spectrum** (common in LLMs): RSVD (q=1) fails because singular values don't separate fast enough
- **RSI q=4**: Recovers ViT-B/32 accuracy from 16.82% → **77.15%** Top-1 at α=0.4 compression

### RSVD vs RSI Comparison

| Scenario | RSVD (q=1) | RSI (q>1) |
|----------|-------------|------------|
| Fast decaying spectrum | ✓ Works | ✓ Works |
| Slow decaying spectrum (LLMs) | ✗ Fails | ✓ Amplifies separation |
| Computational cost | O(dk) | O(q·d·k) |

---

## LLM-Specific Applications

### 1. RSI for LLMs — Low-Rank Compression (arXiv:2604.02659)

> **Paper:** [arXiv:2604.02659](https://arxiv.org/abs/2604.02659)  
> **Author:** Farhad Pourkamali-Anaraki, University of Colorado Denver

**Core Problem:** Standard randomized SVD fails for LLMs with slowly decaying singular value spectra.

**Theoretical Contribution:** Proved softmax perturbation bound — prediction errors bounded by spectral approximation error ‖W−Ŵ‖₂

**Key Quote:**
> *"RSVD is inadequate, and we propose randomized subspace iteration (RSI) as a more effective alternative"*

**Benchmarks (ViT-B/32, α=0.4 compression):**

| Method | Top-1 Accuracy |
|--------|---------------|
| RSVD (q=1) | 16.82% |
| **RSI (q=4)** | **77.15%** |

**Speedup:** 76× faster than exact SVD at k=200 for VGG19 classifier layer.

---

### 2. GaLore++ — Cross-Head Projection + RSI (arXiv:2412.19820)

> **Paper:** [arXiv:2412.19820](https://arxiv.org/abs/2412.19820) · **OpenReview:** [Link](https://openreview.net/forum?id=n8MNWHfhTO)  
> **Authors:** Xutao Liao et al. (Tsinghua, Salesforce AI Research)

**Core Problem:** SVD consumes >80% of GaLore fine-tuning time; multi-head attention accounts for ~half.

**Method:** Combines:
- Cross-head projection sharing
- **RSI for fast SVD**
- Sparsely coded residuals

> *"We employ randomized subspace iteration to achieve fast SVD"*

**Results:**
- ~4× speedup over vanilla GaLore
- 70% total time reduction
- Superior performance on GSM8k/MAWPS arithmetic reasoning and E2E NLG

---

### 3. PRAC — Principal-Random Subspace for Activation Compression (arXiv:2602.23111)

> **Paper:** [arXiv:2602.23111](https://arxiv.org/abs/2602.23111)

**Core Problem:** Activations are 89% of memory in large-batch LLM training (LLaMA-1B: 84.17GB of 94.5GB).

**Method:** Decomposes activations into:
- **Principal subspace** (SVD) — captures main variance
- **Random orthogonal complement** — unbiased reconstruction with minimum variance

> *"PRAC is the first [method] to theoretically bridge fast convergence and activation compression"*

**Results:** 27–36% memory reduction with negligible perplexity degradation; compatible with Muon, Adam-mini optimizers.

---

## Relationship to LoRA

> *"Our approach is complementary to low-rank adaptation methods such as LoRA. While LoRA reduces trainable parameters during fine-tuning by constraining updates to a low-rank subspace, RSI can be applied to compress backbone weights prior to or during adaptation."* — RSI paper

**Proposed combination:** RSI for backbone compression → LoRA-style adaptation on compressed weights.

---

## Related: Randomized Numerical Linear Algebra

### Foundational Papers

| Paper | Venue | URL | Contribution |
|-------|-------|-----|-------------|
| **Halko et al. 2011** | SIAM Review 53(2) | [Link](https://epubs.siam.org/doi/10.1137/090771806) | Modular framework for randomized SVD and subspace iteration |
| **Martinsson et al. 2013** | SIMAX 36(3) | [Link](https://epubs.siam.org/doi/10.1137/130938700) | Error analysis for randomized algorithms within subspace iteration |

### Alternative: Trion (ICLR 2026)

> **Paper:** [arXiv:2505.17967](https://arxiv.org/abs/2505.17967) · **OpenReview:** [Link](https://openreview.net/forum?id=TkHjRwbMNl)

FFT-based dynamic subspace selection replacing SVD/QR with DCT for faster computation.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **Halko et al. 2011** | 2011 | SIAM Review | [Link](https://epubs.siam.org/doi/10.1137/090771806) | Foundational randomized NLA framework |
| **Martinsson et al. 2013** | 2013 | SIMAX | [Link](https://epubs.siam.org/doi/10.1137/130938700) | Error analysis for randomized subspace iteration |
| **RSI for LLMs** | 2026 | — | [arXiv](https://arxiv.org/abs/2604.02659) | RSI applied to LLM low-rank compression |
| **GaLore++** | 2024 | — | [arXiv](https://arxiv.org/abs/2412.19820) | Cross-head projection + RSI for fast GaLore |
| **PRAC** | 2025 | — | [arXiv](https://arxiv.org/abs/2602.23111) | Principal-random subspace for activation compression |
| **Trion** | 2025 | ICLR 2026 | [arXiv](https://arxiv.org/abs/2505.17967) | FFT-based dynamic subspace selection |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [jiaweizzhao/GaLore](https://github.com/jiaweizzhao/galore) | GaLore — memory-efficient LLM training via gradient low-rank projection |
| [stefan-zobel/rSVD](https://github.com/stefan-zobel/rSVD) | Randomized SVD per Halko et al. 2011 |
| [gwgundersen/randomized-svd](https://github.com/gwgundersen/randomized-svd) | Python randomized SVD |
| [JuliaLinearAlgebra/RandomizedLinAlg.jl](https://github.com/JuliaLinearAlgebra/RandomizedLinAlg.jl) | Julia randomized NLA package |
| [wangqinsi1/Dobi-SVD](https://github.com/wangqinsi1/Dobi-SVD) | Differentiable SVD for LLM compression |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Fast low-rank approximation for LLMs where singular values decay slowly |
| **Key innovation** | Power iterations (q>1) amplify spectral separation — RSVD fails where RSI succeeds |
| **Speedup** | 76× faster than exact SVD; 4× faster than vanilla GaLore |
| **Best for** | Pre-compression before LoRA fine-tuning; activation compression for memory-efficient training |
| **Complementary to** | LoRA (RSI compresses backbone; LoRA adapts) |

---

*Last updated: 2026-04-19*
