# P-KD-Q Pipeline — Post-training Quantization + Knowledge Distillation

> **Key Papers:** [P-KD-Q Systematic Study (arXiv:2511.19495)](https://arxiv.org/abs/2511.19495) · [Prune-Quantize-Distill (arXiv:2604.04988)](https://arxiv.org/html/2604.04988v1) · [SPQ (arXiv:2602.18420)](https://arxiv.org/abs/2602.18420) · [FITCompress (arXiv:2302.07612)](https://arxiv.org/abs/2302.07612) · [Oh! We Freeze (ICLR 2024 Workshop)](https://arxiv.org/abs/2403.18159)  
> **GitHub:** [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner) · [microsoft/LMOps/minillm](https://github.com/microsoft/LMOps/tree/main/minillm) · [neuralmagic/sparseml](https://github.com/neuralmagic/sparseml)  
> **Surveys:** [LLM Compression Survey (arXiv:2308.07633)](https://arxiv.org/abs/2308.07633)

---

## Overview

> *"The sequence **Pruning → Knowledge Distillation → Quantization (P-KD-Q)** yields the best balance, achieving a **3.68× compression ratio** while preserving strong instruction-following and language understanding capabilities."* — [P-KD-Q Systematic Study](https://arxiv.org/abs/2511.19495)

Compound compression pipelines combine multiple compression techniques (pruning, quantization, knowledge distillation) in an optimal ordering. Research shows that **order matters significantly** — applying quantization too early causes catastrophic performance collapse.

| Aspect | Details |
|--------|---------|
| **Method** | Combine pruning + KD + quantization in optimal sequence |
| **Key Finding** | P → KD → Q is optimal ordering for LLMs |
| **Compression Ratio** | Up to 3.68× with preserved capabilities |
| **Critical** | Quantization must be last step |

---

## Key Methods

### 1. P-KD-Q — Systematic Study of Compression Ordering (LREC 2026)

> **Paper:** [arXiv:2511.19495](https://arxiv.org/abs/2511.19495) · **PDF:** [arxiv.org/pdf/2511.19495](https://arxiv.org/pdf/2511.19495)

**Core Finding:**
> *"Quantization provides the greatest standalone compression, while pruning introduces moderate quality degradation. Early quantization causes **catastrophic performance collapse**."*

**Pipeline:**
1. **Pruning** — Structured pruning (30% FFN layers)
2. **Knowledge Distillation** — Dual-objective loss (α=0.3, T=4.0)
3. **Quantization** — NF4 post-training quantization via BitsAndBytes

**Results on Qwen2.5-3B:**

| Configuration | Compression | G-Eval |
|--------------|-------------|--------|
| Base | 1× | 0.830 |
| **P-KD-Q** | **3.68×** | **0.733** |

**Key Insight:** "Quantization must be the final step due to inference-only constraint — no gradient computation possible after NF4."

### 2. Prune-Quantize-Distill Pipeline (arXiv:2604.04988)

> **Paper:** [arXiv:2604.04988](https://arxiv.org/html/2604.04988v1)

**Key Quote:**
> *"INT8 QAT provides the dominant runtime benefit, while pruning mainly acts as a capacity-reduction pre-conditioner that improves the robustness of subsequent low-precision optimization; KD, applied last, recovers accuracy within the already constrained sparse INT8 regime."*

**Optimal Order:** Prune → QAT → KD

**Stage Roles:**
| Stage | Role |
|-------|------|
| **Pruning** | Reduces accumulated quantization perturbation noise: $E\|\|\epsilon\|\|^2 \leq \Delta^2/12 \times \|\|W(p)\|\|_0$ |
| **QAT** | Provides actual CPU speedup via standard integer backends (e.g., fbgemm) |
| **KD** | "Reshapes the loss landscape inside the constrained feasible region" |

**Results (ResNet-18):**

| Configuration | Accuracy | Size | Latency |
|---------------|----------|-------|---------|
| Baseline | 78.37% | 42.65 MB | 2.45ms |
| **Hybrid P-QD** | **79.62%** | **6.74 MB** | **1.00ms** |

### 3. SPQ — SVD + Pruning + Quantization (LREC 2026)

> **Paper:** [arXiv:2602.18420](https://arxiv.org/abs/2602.18420) · **PDF:** [arxiv.org/pdf/2602.18420]

> *"SPQ is the first to combine SVD on attention layers, activation-based pruning on MLPs, and linear quantization across all layers in a modular, layer-aware pipeline."*

**Three-Stage Modular Pipeline:**

| Stage | Method | Target |
|-------|--------|--------|
| 1 | SVD on attention layers | Reduce embedding dimension |
| 2 | Activation-based pruning on MLPs | Remove redundant neurons |
| 3 | Linear quantization | Reduce bit-width |

**Results:**

| Model | MEM Before | MEM After | Ratio | PPL Before | PPL After |
|-------|-----------|-----------|-------|------------|-----------|
| LLaMA-3.2-1B | 5.99 GB | 2.27 GB | 62% | 7.88 | 8.62 |
| LLaMA-3.2-3B | 14.42 GB | 4.77 GB | 67% | 6.86 | 6.39 |
| Mistral-7B | 28.95 GB | 7.52 GB | 74% | 5.15 | 5.52 |

### 4. FITCompress — Joint Fisher Information Pruning + Quantization

> **Paper:** [arXiv:2302.07612](https://arxiv.org/abs/2302.07612)

> *"FITCompress optimally selects a combination of pruning mask and mixed-precision quantization configuration for a given pre-trained model and compression constraint."*

**Key Innovation:** **Joint optimization** rather than sequential application — uses Fisher Information Metric and path planning through compression space.

### 5. Oh! We Freeze — KD-Aware Quantization Fine-Tuning (ICLR 2024 Workshop)

> **Paper:** [arXiv:2403.18159](https://arxiv.org/abs/2403.18159)

> *"In this study, we propose a light-weight quantization aware fine tuning technique using knowledge distillation (KD-QAT) to improve the performance of 4-bit."*

**Method:** Ov-freeze technique to stabilize KD-QAT process.

---

## Compression Ordering Principles

### Why Order Matters

From systematic ablation studies:

```
┌─────────────────────────────────────────────────────────────────────┐
│  OPTIMAL ORDER:  Pruning → Knowledge Distillation → Quantization    │
│                                                                     │
│  1. Pruning first     → Reduces capacity, stabilizes optimization   │
│  2. KD second         → Transfers knowledge before Q noise          │
│  3. Quantization last → Minimal info loss; inference-only constraint│
└─────────────────────────────────────────────────────────────────────┘
```

### Catastrophic Collapse Risk

| Order | Risk | Result |
|-------|------|--------|
| Q → KD → P | ❌ Catastrophic | ~53× perplexity degradation |
| Q → P → KD | ❌ Catastrophic | ~53× perplexity degradation |
| P → Q → KD | ⚠️ Moderate | Quality degradation |
| **P → KD → Q** | ✅ Safe | **Optimal balance** |

### Why Quantization Must Be Last

> *"Quantization must be the final step due to inference-only constraint — no gradient computation possible after NF4."*

Quantization converts the model to an inference-only format. KD requires gradient flow, so KD must complete before quantization.

---

## Taxonomic Context

```
Model Compression for LLMs
├── Quantization (PTQ/QAT)
│   ├── Weight-only (AWQ, GPTQ, SpQR)
│   ├── Weight-Activation (SmoothQuant)
│   └── NF4/BF16 (BitsAndBytes)
├── Pruning
│   ├── Unstructured (magnitude, Wanda, SparseGPT)
│   ├── Structured (LLM-Pruner, Sheared LLaMA)
│   └── Semi-Structured (N:M sparsity)
├── Knowledge Distillation
│   ├── Logit Distillation (MiniLLM, GKD)
│   └── Feature Distillation (MiniLM, CKA-matching)
└── Low-Rank Factorization
    ├── SVD-LLM
    ├── OBD-LLM
    └── RSI
```

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **P-KD-Q Systematic Study** | 2025 | LREC 2026 | [arXiv](https://arxiv.org/abs/2511.19495) | Optimal ordering: P→KD→Q; 3.68× compression |
| **Prune-Quantize-Distill** | 2026 | — | [arXiv](https://arxiv.org/html/2604.04988v1) | INT8 QAT dominant; prune as pre-conditioner |
| **SPQ** | 2026 | LREC 2026 | [arXiv](https://arxiv.org/abs/2602.18420) | SVD+P+Quant layer-aware modular pipeline |
| **FITCompress** | 2023 | — | [arXiv](https://arxiv.org/abs/2302.07612) | Joint Fisher-based pruning + mixed-precision Q |
| **Oh! We Freeze** | 2024 | ICLR Workshop | [arXiv](https://arxiv.org/abs/2403.18159) | KD-aware quantization fine-tuning |
| **LLM Compression Survey** | 2023 | — | [arXiv](https://arxiv.org/abs/2308.07633) | Comprehensive taxonomy of LLM compression |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner) | LLM structural pruning (NeurIPS 2023) |
| [microsoft/LMOps/minillm](https://github.com/microsoft/LMOps/tree/main/minillm) | Reverse KL distillation for LLMs (ICLR 2024) |
| [neuralmagic/sparseml](https://github.com/neuralmagic/sparseml) | Pruning, quantization, and distillation pipeline |
| [HuangOwen/Awesome-LLM-Compression](https://github.com/HuangOwen/Awesome-LLM-Compression) | 1.8k stars — curated LLM compression list |
| [ModelTC/LightCompress](https://github.com/ModelTC/LightCompress) | LLMC benchmark toolkit (EMNLP 2024) |
| [pprp/Awesome-LLM-Quantization](https://github.com/pprp/awesome-llm-quantization) | Comprehensive quantization resources |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Optimal ordering of multiple compression techniques to maximize compression while preserving quality |
| **Key finding** | P → KD → Q is optimal; Q-P or Q-KD order causes catastrophic collapse (~53× perplexity) |
| **Best ratio** | 3.68× compression with strong capability preservation (Qwen2.5-3B) |
| **Why it works** | Pruning reduces capacity → stabilizes KD → Q final (no gradients needed) |
| **Key insight** | Quantization must always be last due to inference-only constraint |

---

*Last updated: 2026-04-19*
