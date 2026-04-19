# OBD-LLM — Optimal Brain Decomposition for LLMs

> **Paper:** [arXiv:2604.00821](https://arxiv.org/abs/2604.00821) · **PDF:** [arxiv.org/pdf/2604.00821](https://arxiv.org/pdf/2604.00821)  
> **Related:** [OBC (NeurIPS 2022)](https://github.com/IST-DASLab/OBC) · [LLM Surgeon (ICLR 2024)](https://github.com/Qualcomm-AI-research/llm-surgeon)  
> **Historical:** [OBD LeCun 1989](https://proceedings.neurips.cc/paper/1989/file/6c9882bbac1c7093bd25041881277658-Paper.pdf) · [OBS Hassibi & Stork 1992](https://www.babak.caltech.edu/pubs/conferences/00298572.pdf)

---

## Overview

> *"We propose Optimal Brain Decomposition for LLM (OBD-LLM), a principled decomposition technique that minimizes the decomposition error in the global task loss space."* — Li et al., [arXiv:2604.00821](https://arxiv.org/abs/2604.00821)

OBD-LLM is a **loss-aware low-rank decomposition method** that uses second-order Hessian information via Kronecker-factorization to achieve bi-directional whitening (input activation + output gradient). It achieves ~20–40% better results than SVD-LLM.

| Aspect | Details |
|--------|---------|
| **Method** | Kronecker-factored Hessian bi-directional whitening |
| **Key Innovation** | Decomposition considers both input and output information of each layer |
| **Result** | ~20–40% better than SVD-LLM |
| **Approach** | Closed-form solution for optimal decomposition |

---

## Core Method

### The Problem with Prior Work

Prior SVD-based methods (ASVD, SVD-LLM) only whiten using **input activation statistics**, ignoring **output gradient information**, leading to suboptimal decomposition.

### OBD-LLM Solution

Minimizes global task loss change via second-order Taylor expansion:

$$\Delta L(w) \approx (w - \hat{w})H_w(w - \hat{w})^\top$$

Where $H_w$ is the Hessian. Uses K-FAC approximation:

$$H_w \approx C_x \otimes C_g \quad \text{(input covariance } \otimes \text{ output gradient covariance)}$$

**Bi-directional whitening:** $\tilde{W} = L_g^\top W L_x$ (Cholesky factors of both covariances)

**Low-rank factors:**
- $B = L_g^{(-\top)} U_{:,:r} \Sigma^{1/2}_{:r,:r}$
- $A = \Sigma^{1/2}_{:r,:r} V^\top_{:r,:r} L_x^{(-1)}$

> *"Through a rigorous Kronecker-factorization of the Hessian, we show that the decomposition needs to consider both input and output information of the layer, and achieves much better decomposition results compared to input only method."*

### Key Theoretical Result

> *"OBD-LLM is a closed-form solution for the optimal decomposition of weights in the language model."*

---

## Benchmarks

### WikiText-2 Perplexity (Lower = Better)

| Model | Method | 10% | 20% | 30% | 40% |
|-------|--------|-----|-----|-----|-----|
| LLaMA3-8B | SVD-LLM | 21.23 | 49.88 | 160.3 | 679.8 |
| **OBD-LLM** | — | **15.73** | **32.92** | **70.83** | **167.3** |
| LLaMA2-7B | SVD-LLM | 9.42 | 12.86 | 22.76 | 53.16 |
| **OBD-LLM** | — | **8.42** | **11.50** | **18.83** | **36.39** |

### Downstream Task Accuracy (20% compression, 8 tasks average)

| Model | Full Rank | SVD-LLM | OBD-LLM |
|-------|-----------|---------|---------|
| LLaMA3-8B | 65.41 | 52.69 | **54.23** |
| LLaMA2-7B | 62.44 | 51.88 | **54.48** |

### Implementation Details

| Detail | Value |
|--------|-------|
| **Calibration data** | 128 sequences × 2048 tokens from C4 |
| **Covariance collection time** | ~3 minutes |
| **K-FAC validity** | ρ (activation-gradient correlation) ≤ 0.1 for all layers — confirms independence assumption |

---

## Historical Context

### Original Optimal Brain Damage (OBD) — LeCun et al., 1989

> **Paper:** [NeurIPS 1989](https://proceedings.neurips.cc/paper/1989/file/6c9882bbac1c7093bd25041881277658-Paper.pdf)

Introduced second-order information ( Hessian) for network pruning. Estimated importance of weights using second derivatives of the loss function.

### Optimal Brain Surgeon (OBS) — Hassibi & Stork, 1992

> **Paper:** [Caltech](https://www.babak.caltech.edu/pubs/conferences/00298572.pdf)

Improved upon OBD — removes wrong weights less often and provides exact weight update formulas. Foundation for modern OBS-based compression.

---

## Related Work

### OBC — Optimal Brain Compression (NeurIPS 2022)

> **Paper:** [arXiv:2208.11580](https://arxiv.org/pdf/2208.11580) · **Code:** [IST-DASLab/OBC](https://github.com/IST-DASLab/OBC)  
> **Authors:** Elias Frantar, Sidak Pal Singh, Dan Alistarh

Framework for post-training quantization and pruning using exact OBS. Foundation for SparseGPT.

### LLM Surgeon (ICLR 2024)

> **Paper:** [arXiv:2312.17244](https://arxiv.org/abs/2312.17244) · **Code:** [Qualcomm-AI-research/llm-surgeon](https://github.com/Qualcomm-AI-research/llm-surgeon)

Scales Kronecker-factored curvature approximations to LLMs for structured pruning.

### L-OBS — Layer-wise Optimal Brain Surgeon

> **Code:** [csyhhu/L-OBS](https://github.com/csyhhu/L-OBS)

Layer-wise application of OBS for LLM compression.

### SoLoRA — LoRA Meets Second-Order Optimization (ICLR 2026)

> **Paper:** [OpenReview](https://openreview.net/forum?id=3IGUspVNtk)

Low-rank fine-tuning with adaptive metric incorporating Hessian information.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **OBD (original)** | 1989 | NeurIPS | [PDF](https://proceedings.neurips.cc/paper/1989/file/6c9882bbac1c7093bd25041881277658-Paper.pdf) | Foundational second-order pruning |
| **OBS** | 1992 | — | [PDF](https://www.babak.caltech.edu/pubs/conferences/00298572.pdf) | Exact weight updates, less wrong pruning |
| **OBC** | 2022 | NeurIPS 2022 | [arXiv](https://arxiv.org/pdf/2208.11580) | Post-training quantization + pruning via OBS |
| **LLM Surgeon** | 2024 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2312.17244) | K-FAC at LLM scale for structured pruning |
| **OBD-LLM** | 2026 | — | [arXiv](https://arxiv.org/abs/2604.00821) | Bi-directional K-FAC whitening for low-rank decomposition |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [IST-DASLab/OBC](https://github.com/IST-DASLab/OBC) | Optimal Brain Compression (quantization + pruning) |
| [Qualcomm-AI-research/llm-surgeon](https://github.com/Qualcomm-AI-research/llm-surgeon) | LLM Surgeon — K-FAC at LLM scale |
| [csyhhu/L-OBS](https://github.com/csyhhu/L-OBS) | Layer-wise Optimal Brain Surgeon |
| [HuangOwen/Awesome-LLM-Compression](https://github.com/HuangOwen/Awesome-LLM-Compression) | Curated LLM compression resources |
| [horseee/Awesome-Efficient-LLM](https://github.com/horseee/Awesome-Efficient-LLM/blob/main/low_rank_decomposition.md) | Low-rank decomposition resources |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Loss-aware low-rank decomposition that considers both input activations AND output gradients |
| **Key innovation** | Bi-directional K-FAC Hessian whitening — closed-form optimal decomposition |
| **Improvement** | ~20–40% better perplexity than SVD-LLM at equivalent compression |
| **Why better than SVD-LLM** | SVD-LLM only considers input whitening; OBD-LLM considers both directions |
| **Limitation** | Still a preprint (April 2026); no official GitHub implementation found |

---

*Last updated: 2026-04-19*
