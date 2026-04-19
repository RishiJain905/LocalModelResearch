# MultiModal MoE Quantization (VEQ)

> **Parent Topic:** [MoE Quantization](./README.md) | **Tags:** `moe` `quantization` `multimodal` `vision` `language` `VEQ`

---

## Table of Contents
1. [Overview](#overview)
2. [Key Papers](#key-papers)
3. [Architecture Approaches](#architecture-approaches)
4. [Methods](#methods)
5. [Benchmarks](#benchmarks)
6. [GitHub Repositories](#github-repositories)
7. [Related Topics](#related-topics)

---

## Overview

MultiModal MoE quantization addresses the challenge of quantizing models that handle both **text and vision** modalities using Mixture-of-Experts architecture. Vision tokens and language tokens have fundamentally different characteristics — vision features are more spatially correlated and have different entropy distributions — requiring **modality-specific quantization strategies**.

> *"VEQ proposes treating vision and language experts differently during quantization."*
> — [VEQ: Vision-Experts Quantization](https://github.com/VectorSpaceLab/VEQ)

---

## Key Papers

### VEQ — Vision-Experts Quantization
**"VEQ: Vision-Experts Quantization for Multimodal Large Language Models"**

- **GitHub:** https://github.com/VectorSpaceLab/VEQ
- **Organization:** VectorSpaceLab

**Key Approach:**
- VEQ applies modality-specific quantization strategies to preserve multimodal performance
- Vision tokens and language tokens have different characteristics:
  - **Vision features:** More spatially correlated, different entropy distributions
  - **Language features:** Different activation patterns, different outlier distributions
- The method uses different bit-widths for vision encoder vs language model components

---

### QMoE — MoE KV Cache Quantization
**"QMoE: Practical KV Cache Quantization for Mixture-of-Experts Models"**

- **arXiv:** https://arxiv.org/abs/2312.00079 (or similar)
- **Focus:** KV cache compression in MoE architectures
- **Key Result:** Sub-1% memory reduction for billion-scale MoE models

---

### MoEQuant — General MoE Quantization
**"MoEQuant: Enhancing Quantization for Mixture-of-Experts Large Language Models"**

- **arXiv:** https://arxiv.org/abs/2505.03804
- **Related Work:** Expert-specific quantization strategies applicable to multimodal

---

## Architecture Approaches

### MoE Architecture Types for Multimodal

| Architecture | Description | Organization |
|--------------|-------------|--------------|
| **LLaVA-MoE** | LLaVA with Mixture-of-Experts | Various |
| **VLMoE** | Vision-Language MoE | Google/ByteDance |
| **CaiT-MoE** | Class-Attention in Transformer with MoE | Meta |
| **MMCo** | Multimodal Mixture of Experts | Various |
| **MoE-LLaVA** | Mixture of Experts for Large Vision-Language Models | [arXiv](https://arxiv.org/) |

---

### Vision-Experts vs Language-Experts

**Key Difference:**

| Aspect | Vision Experts | Language Experts |
|--------|---------------|-----------------|
| Activation Pattern | Spatially structured, local correlations | Contextual, long-range dependencies |
| Entropy Distribution | Lower entropy (more redundancy) | Higher entropy (more diverse) |
| Outlier Frequency | Less frequent, more structured | More frequent, more scattered |
| Routing Behavior | May route based on spatial region | May route based on semantic content |

**VEQ's Insight:** Treating these differently during quantization yields better accuracy preservation.

---

## Methods

### Modality-Specific Quantization

| Method | Description |
|--------|-------------|
| **Per-Modality Precision** | Assign different bit-widths to vision vs language components |
| **Activation Smoothing** | Suppress modality-specific outliers before quantization |
| **Routing-Aware Bit Allocation** | Consider which experts handle which modality |
| **Cross-Modality Alignment** | Preserve alignment between vision and language representations |

---

### VEQ Quantization Pipeline

```
Input: Multimodal MoE Model
  ├── Vision Encoder → Vision Experts → Quantize (e.g., INT6)
  ├── Language Model → Language Experts → Quantize (e.g., INT4)
  └── Router → Routing-aware bit allocation
Output: Quantized Multimodal MoE
```

---

## Benchmarks

| Benchmark | Description | Used For |
|-----------|-------------|----------|
| **VQAv2** | Visual Question Answering v2 | V&L understanding |
| **GQA** | Graphical Question Answering | V&L reasoning |
| **POPE** | Object hallucination evaluation | Reliability |
| **MME** | Multimodal evaluation | Comprehensive |
| **MMBench** | Multi-modal benchmark | Holistic capability |
| **ImageNet** | Classification accuracy | Vision-only tasks |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [VEQ](https://github.com/VectorSpaceLab/VEQ) | Vision-Experts Quantization for Multimodal LLMs |
| [LLaVA-MoE](https://github.com/) | LLaVA with MoE architecture |
| [VLMoE](https://github.com/) | Vision-Language MoE from Google/ByteDance |
| [QMoE](https://github.com/jzhang38/QMoE) | General MoE quantization (applicable to multimodal) |
| [MoEQuant](https://github.com/chenzx921020/MoEQuant) | Expert-balanced sampling for MoE PTQ |

---

## Key Technical Considerations

### Why Multimodal MoE Quantization is Hard

1. **Different activation distributions:** Vision vs language have fundamentally different statistics
2. **Cross-modal routing:** Router must handle both modalities — balance differs per modality
3. **Alignment preservation:** Quantization must not break vision-language alignment
4. **Outlier patterns differ:** Vision outliers are spatially structured; language outliers are semantic

### VEQ's Solution

VEQ addresses these by:
1. **Treating experts as modality-aware:** Some experts specialize in vision, others in language
2. **Per-expert precision:** Assign bit-widths based on which modality the expert handles
3. **Calibration dataset balance:** Include both vision and language samples in calibration

---

## Related Topics

- [Affinity Guided Quantization](./Affinity-Guided-Quantization.md) — bit allocation for MoE
- [Attention Sink & Outlier Preservation](./Attention-Sink-Outlier-Preservation.md) — outlier handling differs per modality
- [Expert Merging & Pruning](./Expert-Merging-Pruning.md) — consolidating multimodal experts
- [Weight Sharing](../Non-Quant-Compression/Weight-Sharing-Recursive-Layers/DeepSeek-V3-Style-Sharing.md) — DeepSeek-V3's MLA for efficient KV handling

---

## Summary

| Aspect | Key Finding |
|--------|-------------|
| **Challenge** | Vision and language have different activation distributions |
| **VEQ Solution** | Modality-specific quantization for vision vs language experts |
| **Key Benefit** | Preserves multimodal alignment while compressing |
| **Benchmark Focus** | VQAv2, GQA, POPE, MME, MMBench |

---

*Last updated: 2026-04-19 | Sources: VEQ GitHub (VectorSpaceLab), arXiv, various multimodal MoE papers*
