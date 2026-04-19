# MoE Quantization

> **Parent Topic:** [Quantization](../README.md) | **Category:** Quantization | **Tags:** `moe` `mixture-of-experts` `quantization`

---

## Overview

MoE (Mixture-of-Experts) quantization addresses the unique challenges of quantizing sparse mixture-of-experts models. Unlike dense LLMs, MoE models have dynamic routing, non-uniform expert activation patterns, and modality-specific expert behavior that require specialized quantization strategies.

---

## Topics

| Topic | Description |
|-------|-------------|
| [Inter-Intra Expert Imbalance](./Inter-Intra-Expert-Imbalance.md) | Token routing imbalance across and within experts |
| [Affinity Guided Quantization](./Affinity-Guided-Quantization.md) | Routing-aware mixed-precision bit allocation |
| [Expert Merging & Pruning](./Expert-Merging-Pruning.md) | Consolidating and reducing expert counts |
| [Attention Sink & Outlier Preservation](./Attention-Sink-Outlier-Preservation.md) | Preserving critical activation patterns during quantization |
| [MultiModal MoE — VEQ](./MultiModal-MOE-VEQ.md) | Quantizing vision-language MoE models |

---

## Key Papers

| Paper | Venue | Key Contribution |
|-------|-------|------------------|
| [MoEQuant](https://arxiv.org/abs/2505.03804) | ICML 2025 | Expert-Balanced Self-Sampling + Affinity-Guided Quantization |
| [MxMoE](https://arxiv.org/abs/2505.05799) | ICML 2025 | Mixed-precision with accuracy-performance co-design (3.4× speedup) |
| [EAQuant](https://arxiv.org/abs/2506.13329) | 2025 | Smoothing aggregation + routing consistency + expert-specific strategies |
| [GEMQ](https://openreview.net/forum?id=wAc718O8UM) | ICLR 2026 | Global expert-level mixed-precision for extreme low-bit |
| [ExpertQuant](https://openreview.net/forum?id=kPgLp47bJf) | ICLR 2026 | Rank-aware quantization preserving router decisions |
| [VEQ](https://github.com/VectorSpaceLab/VEQ) | — | Vision-Experts Quantization for Multimodal LLMs |
| [QMoE](https://github.com/jzhang38/QMoE) | — | Sub-1% memory reduction for billion-scale MoE |
| [DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) | — | Auxiliary-loss-free load balancing + MLA (93.3% KV cache reduction) |

---

## Key Challenges

1. **Inter-Expert Imbalance** — Tokens unevenly distributed among experts
2. **Intra-Expert Imbalance** — Non-uniform activation distributions within experts
3. **Expert Routing Collapse** — Some experts become functionally redundant
4. **Attention Sinks** — Certain tokens absorb excess attention, creating outliers
5. **Modality Differences** — Vision vs language experts have different characteristics
6. **Router Dynamics** — Quantization alters router behavior; bit allocation must account for this

---

## Methods Summary

| Category | Methods |
|----------|---------|
| **Calibration** | EBSS, Heavy-token guidance, Calibration-free allocation |
| **Bit Allocation** | Affinity-guided, Router-preserving, Global expert-level |
| **Outlier Handling** | Mixed-precision decomposition, NF4, Per-token/channel quantization |
| **Architecture** | MLA (DeepSeek), Expert merging, Expert pruning |

---

## GitHub Repositories

| Repo | Focus |
|------|-------|
| [MoEQuant](https://github.com/chenzx921020/MoEQuant) | ICML 2025 implementation |
| [MxMoE](https://github.com/cat538/MxMoE) | Mixed-precision MoE |
| [EAQuant](https://github.com/darren-fzq1/EAQuant) | Expert-aware PTQ |
| [VEQ](https://github.com/VectorSpaceLab/VEQ) | Vision-Experts quantization |
| [QMoE](https://github.com/jzhang38/QMoE) | Sub-1% MoE memory reduction |
| [DeepSeek-V3](https://github.com/deepseek-ai/DeepSeek-V3) | MLA + auxiliary-loss-free balancing |
| [QuantMoE-Bench](https://github.com/UNITES-Lab/moe-quantization) | Bit allocation benchmark |

---

## Related Topics

- [Weight Quantization](../Weight-Quantization/README.md) — Standard weight-only quantization
- [KV Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md) — KV cache compression techniques
- [Model Pruning](../Non-Quant-Compression/Model-Pruning/README.md) — General pruning techniques

---

*Last updated: 2026-04-19*
