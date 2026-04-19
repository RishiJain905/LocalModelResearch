# Affinity Guided Quantization for MoE

> **Parent Topic:** [MoE Quantization](./README.md) | **Tags:** `moe` `quantization` `affinity` `mixed-precision`

---

## Table of Contents
1. [Overview](#overview)
2. [Key Papers](#key-papers)
3. [Methods](#methods)
4. [Benchmarks](#benchmarks)
5. [GitHub Repositories](#github-repositories)
6. [Key Quotes](#key-quotes)
7. [Related Topics](#related-topics)

---

## Overview

Affinity Guided Quantization addresses the challenge of assigning appropriate precision levels to different experts based on their routing relationships and activation patterns. Unlike uniform quantization, AGQ incorporates **token-expert affinities** into the quantization process to accurately assess token-expert interactions.

> *"The paper identifies two primary challenges in MoE quantization: uneven distribution of calibration samples among experts, and token-expert affinity variations introduced by gating units."*
> — MoEQuant (ICML 2025)

---

## Key Papers

### MoEQuant — ICML 2025
**"MoEQuant: Enhancing Quantization for Mixture-of-Experts Large Language Models via Expert-Balanced Sampling and Affinity Guidance"**

- **arXiv:** https://arxiv.org/abs/2505.03804
- **arXiv HTML:** https://arxiv.org/html/2505.03804v1
- **GitHub:** https://github.com/chenzx921020/MoEQuant

**Core Techniques:**

1. **Expert-Balanced Self-Sampling (EBSS):** An efficient sampling method that constructs a calibration set with balanced expert distributions by leveraging cumulative probabilities of tokens and expert balance metrics as guiding factors.

2. **Affinity-Guided Quantization (AGQ):** Incorporates affinities between experts and samples into the quantization process to accurately assess token-expert interactions.

**Result:** With MoEQuant++, accuracy even **surpasses the full-precision model** in some cases.

---

### MxMoE — ICML 2025
**"MxMoE: Mixed-precision Quantization for MoE with Accuracy and Performance Co-Design"**

- **arXiv:** https://arxiv.org/abs/2505.05799
- **GitHub:** https://github.com/cat538/MxMoE

**Key Approach:** A mixed-precision optimization framework considering both algorithmic and system perspectives. It navigates design space defined by parameter sensitivity, expert activation dynamics, and hardware resources to derive efficient mixed-precision configurations.

**Benchmark:** Up to **3.4x speedup** over the original full-size model while outperforming existing methods in accuracy.

---

### EAQuant — 2025
**"EAQuant: Enhancing Post-Training Quantization for MoE Models"**

- **arXiv:** https://arxiv.org/abs/2506.13329
- **GitHub:** https://github.com/darren-fzq1/EAQuant

**Three Expert-Aware Innovations:**
1. Smoothing aggregation to suppress activation outliers
2. Routing consistency alignment
3. Expert-specific quantization strategies

---

### QuantMoE-Bench — 2024
**"QuantMoE-Bench: Examining Post-Training Quantization for Mixture-of-Experts"**

- **arXiv HTML:** https://arxiv.org/html/2406.08155v2
- **GitHub:** https://github.com/UNITES-Lab/moe-quantization

**Research Focus:** Systematic analysis of bit allocation strategies:

| Strategy | Description |
|----------|-------------|
| `+Attn` | Attention-aware adjustments |
| `+Freq` | Expert frequency-based bit allocation |
| `+FirstL` | Prioritization of early MoE layers |
| `+LinearOSP` | Importance-based (linear layer-wise) |
| `+LayerISP` | Importance-based (layer-wise) |

**Key Findings:** Expert usage frequency is an effective quantization heuristic; attention vs FFNN layer precision trade-offs; importance of first vs last MoE blocks; shared expert bit allocation needs.

---

### GEMQ — ICLR 2026 Submission
**"Towards Global Expert-Level Mixed-Precision Quantization for Mixture-of-Experts LLMs"**

- **OpenReview:** https://openreview.net/forum?id=wAc718O8UM
- **Authors:** Jianing Deng, Song Wang, Dongwei Wang, Zijie Liu, Tianlong Chen, Huanrui Yang, Jingtong Hu

**Key Limitations Identified:**
1. Expert importance estimated only locally within each MoE layer, failing to capture global importance
2. Expert quantization alters router dynamics, which is often overlooked

**Proposed Solution:** Global Expert-level Mixed-precision Quantization (GEMQ) for extreme low-bit quantization.

---

### ExpertQuant — ICLR 2026 Submission
**"Router Choice Matters: Rank-Aware Post-Training Quantization for MoE Models"**

- **OpenReview:** https://openreview.net/forum?id=kPgLp47bJf

**Key Insight:** Most errors arise as near-neighbor rank flips around top-k experts. Preserving router decisions of selected experts yields largest gains.

**Framework Components:**
1. Expert-Aware Scale: Accommodates heterogeneous activation ranges
2. Rank-Aware Jaccard Loss: Aligns top-k expert rank
3. Gap Hinge Loss: Preserves score margins between consecutive experts to suppress rank flipping

---

### AlphaQ — OpenReview
**"AlphaQ: Calibration-Free Bit Allocation for Mixture-of-Experts"**

- **OpenReview:** https://openreview.net/forum?id=rbE8Pxs8vx

**Approach:** Calibration-free bit allocation method inspired by heavy-token guidance. Addresses the problem of calibration sample distribution in MoE quantization.

---

### MoPEQ — 2025
**"Mixture of Mixed Precision Quantized Experts"**

- **arXiv:** https://arxiv.org/abs/2509.02512
- **IEEE:** https://ieeexplore.ieee.org/iel8/11373940/11374285/11376553.pdf

> **Note:** Mixed precision particularly well-suited for MoE due to sparse activation where only subset of experts activated.

---

### ACL 2025 Findings
**"Automated Fine-Grained MoE Quantization"**

- **Paper:** https://aclanthology.org/2025.findings-acl.1386.pdf

**Method:** Uses Fisher Information trace combined with condition number for sensitivity-based bit allocation:

```
Sensitivity Sc = α · (Kc · Vc) + (1 − α) · Oc
```

Where:
- `Kc` = condition number
- `Vc` = variance ratio
- `Oc` = outlier ratio

---

## Benchmarks

| Paper | Method | Speedup | Key Advantage |
|-------|--------|---------|--------------|
| MoEQuant | AGQ + EBSS | — | Surpasses FP16 in some cases |
| MxMoE | Mixed-precision co-design | 3.4x | Accuracy-performance co-design |
| EAQuant | Expert-aware PTQ | — | Routing consistency |
| GEMQ | Global expert-level | — | Extreme low-bit |
| ExpertQuant | Rank-aware | — | Router preservation |
| MoPEQ | Per-expert precision | — | Sparse activation exploitation |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [MoEQuant](https://github.com/chenzx921020/MoEQuant) | AGQ + EBSS implementation |
| [MxMoE](https://github.com/cat538/MxMoE) | Mixed-precision MoE |
| [EAQuant](https://github.com/darren-fzq1/EAQuant) | Expert-aware PTQ |
| [QuantMoE-Bench](https://github.com/UNITES-Lab/moe-quantization) | Bit allocation benchmark suite |

---

## Key Quotes

> *"Incorporates affinities between experts and samples into the quantization process to accurately assess token-expert interactions."*
> — MoEQuant

> *"Expert usage frequency is an effective quantization heuristic."*
> — QuantMoE-Bench

> *"Most errors arise as near-neighbor rank flips around top-k experts. Preserving router decisions of selected experts yields largest gains."*
> — ExpertQuant

---

## Related Topics

- [Inter-Intra Expert Imbalance](./Inter-Intra-Expert-Imbalance.md) — root causes of imbalance
- [Expert Merging & Pruning](./Expert-Merging-Pruning.md) — structural optimization
- [Attention Sink & Outlier Preservation](./Attention-Sink-Outlier-Preservation.md) — handling outliers

---

*Last updated: 2026-04-19 | Sources: ICML 2025, arXiv:2505.03804, arXiv:2505.05799, arXiv:2506.13329, OpenReview ICLR 2026*
