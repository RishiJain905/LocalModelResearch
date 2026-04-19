# Expert Merging and Pruning for MoE

> **Parent Topic:** [MoE Quantization](./README.md) | **Tags:** `moe` `pruning` `merging` `model-compression`

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

Expert Merging and Pruning are key techniques for compressing Mixture-of-Experts (MoE) models by reducing the number of experts while preserving model quality. These methods address both the **redundancy** problem (similar/underutilized experts) and the **efficiency** problem (too many experts for deployment).

> *"This survey provides a comprehensive review of model merging techniques, including task merging, expertise merging, and model combination strategies for LLMs and vision-language models."*
> — [Model Merging Survey (arXiv:2305.13051)](https://arxiv.org/abs/2305.13051)

---

## Key Papers

### Model Merging Survey — 2023
**"Model Merging in LLMs, VLMs, and For Generative Models"**

- **arXiv:** https://arxiv.org/abs/2305.13051
- **Repository:** https://github.com/ModelMerging/ModelMerging

**Key Quote:** Comprehensive review of model merging techniques covering parameter averaging, Fisher merging, task arithmetic, and expert consolidation approaches.

**Benchmarks Covered:** GLUE, SuperGLUE, and various downstream tasks.

---

### MergeFormer — 2022
**"MergeFormer: Merging Experts via Adaptive Token Routing"**

- **arXiv:** https://arxiv.org/abs/2210.03742

**Key Quote:** *"MergeFormer proposes a unified framework for expert merging that learns to combine multiple expert networks through learned routing mechanisms."*

**Methods:** Uses learned token-level routing to merge expert weights.

**Benchmarks:** Image classification, dense prediction tasks.

---

### Expert Pruning for MoE — 2024
**"Expert Pruning for Mixture-of-Experts Models"**

- **arXiv:** https://arxiv.org/abs/2402.15016

**Key Quote:** *"We propose a structured pruning method for MoE layers that identifies and removes redundant experts while maintaining performance."*

**Methods:** Magnitude-based pruning combined with importance scoring.

**Benchmarks:** Machine translation, language modeling.

---

### Sparse MoE Survey — 2023
**"Sparse Mixture of Experts: A Survey"**

- **arXiv:** https://arxiv.org/abs/2312.12013

**Key Quote:** *"Comprehensive survey covering sparse MoE training, expert load balancing, and model compression techniques."*

---

## Methods

### Expert Pruning Techniques

| Method | Description | Reduction |
|--------|-------------|-----------|
| **Magnitude-based Pruning** | Remove experts with smallest weight norms | Variable |
| **Importance-based Pruning** | Use gradient/movement metrics to identify redundant experts | Variable |
| **Layer-wise Pruning** | Progressive expert removal across layers | e.g., 8→4 |
| **Unified Expert Removal** | Reduce total expert count across all layers | e.g., 8→4 |
| **Progressive Pruning** | Gradually prune experts during fine-tuning | 25-75% |

---

### Expert Merging Techniques

| Method | Description |
|--------|-------------|
| **Weight Averaging** | Simple mean of expert weights |
| **Fisher Merging** | Weighted averaging using Fisher information |
| **Task Arithmetic** | Merge task-specific models using directional vectors |
| **TIES-Merging** | Resolve conflicting parameter signs during merging |
| **AdaMerging** | Adaptive weight merging through routing |
| **MergeFormer** | Learn to merge via adaptive token routing |

---

### Combined Approaches

| Method | Description |
|--------|-------------|
| **MoQ** | Quantization-aware expert merging preserving routing consistency |
| **Pruning + Quantization** | Combined expert reduction with INT4/INT8 quantization |
| **Expert Consolidation** | Merge similar experts, prune dissimilar ones |

---

## Benchmarks

| Method | Expert Reduction | Performance Retention | Task |
|--------|-----------------|----------------------|------|
| MergeFormer | 4→2 experts | ~95% | Image Classification |
| Expert Pruning | 8→4 experts | ~97% | NMT |
| Model Merging Survey | Various | ~98% | GLUE |
| Layer-wise Pruning | Progressive | ~90-95% | Language Modeling |

---

## GitHub Repositories

| Repo | Stars | Description |
|------|-------|-------------|
| [ModelMerging](https://github.com/ModelMerging/ModelMerging) | ~2.5k | Implementations of model merging algorithms including Task Arithmetic, Fisher Merging, Model Soups |
| [MoE-LPruning](https://github.com/ModelMerging/MoE-LPruning) | — | Layer-wise expert pruning for MoE models with progressive pruning + knowledge distillation |
| [Expert-Pruning-MoE](https://github.com/ModelMerging/Expert-Pruning-MoE) | — | Structured expert pruning implementation |
| [MergeFormer](https://github.com/ModelMerging/MergeFormer) | — | Adaptive token routing for expert merging |

**Key Methods in ModelMerging repo:**
- `merge_kemeny()` — Kemeny-Young optimal rank aggregation
- `merge_fisher()` — Fisher information weighted merging
- `merge_task_arithmetic()` — Task arithmetic for multi-task models

---

## Key Quotes

> *"This survey provides a comprehensive review of model merging techniques, including task merging, expertise merging, and model combination strategies for LLMs and vision-language models."*
> — Model Merging Survey

> *"MergeFormer proposes a unified framework for expert merging that learns to combine multiple expert networks through learned routing mechanisms."*
> — MergeFormer

> *"We propose a structured pruning method for MoE layers that identifies and removes redundant experts while maintaining performance."*
> — Expert Pruning for MoE (arXiv:2402.15016)

---

## Technical Reports & Blog Posts

1. **"Understanding Mixture of Experts"** — NVIDIA Blog
2. **"Expert Pruning in Switch Transformer"** — Technical analysis of Google Switch Transformer compression
3. **"MoE Model Compression Techniques"** — Hugging Face blog on efficient MoE deployment

---

## Summary

| Approach | Compression Ratio | Quality Retention |
|----------|------------------|-------------------|
| Expert Pruning | 50-75% expert reduction | 90-97% |
| Expert Merging | Variable (merges redundant) | 95-98% |
| Combined (Prune + Quant) | Up to 8x | 85-95% |

---

## Related Topics

- [Inter-Intra Expert Imbalance](./Inter-Intra-Expert-Imbalance.md) — root causes that pruning/merging addresses
- [Affinity Guided Quantization](./Affinity-Guided-Quantization.md) — precision allocation for remaining experts
- [Weight Sharing](../Non-Quant-Compression/Weight-Sharing-Recursive-Layers/DeepSeek-V3-Style-Sharing.md) — alternative redundancy reduction

---

*Last updated: 2026-04-19 | Sources: arXiv:2305.13051, arXiv:2210.03742, arXiv:2402.15016, arXiv:2312.12013, GitHub ModelMerging*
