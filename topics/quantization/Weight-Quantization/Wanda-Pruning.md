# Wanda — Weight and Activation Pruning

> **Paper:** [arXiv:2306.11695](https://arxiv.org/abs/2306.11695) (Mingjie Sun*, Zhuang Liu*, Anna Bair, J. Zico Kolter — CMU, Meta AI, Bosch)  
> **Conference:** ICLR 2024  
> **GitHub:** [locuslab/wanda](https://github.com/locuslab/wanda)  
> **Project Page:** [eric-mingjie.github.io/wanda](https://eric-mingjie.github.io/wanda/home.html)  
> **Follow-ups:** [Wanda++](https://arxiv.org/html/2503.04992v4) · [M-Wanda](https://arxiv.org/abs/2505.21171)

---

## Overview

> *"Wanda significantly outperforms the established baseline of magnitude pruning and performs competitively against recent method involving intensive weight update."* — Sun et al., [ICLR 2024](https://arxiv.org/abs/2306.11695)

Wanda (Weight and Activation pruning) is a simple and effective **one-shot pruning method** for large language models that requires **no retraining** and prunes weights per-output using the product of weight magnitudes and input activation norms.

| Aspect | Details |
|--------|---------|
| **Type** | Unstructured pruning |
| **Calibration Data** | Required (small sample of tokens) |
| **Weight Updates** | None — one-shot |
| **Complexity** | O(d²) — ~300× faster than SparseGPT |
| **Speed (7B)** | 0.54s vs SparseGPT's 203.1s |

---

## Core Method

### The Wanda Metric

Wanda prunes weights using:

$$S_{ij} = |W_{ij}| \cdot \|X_j\|_2$$

Where:
- $|W_{ij}|$ = absolute weight value
- $\|X_j\|_2$ = ℓ₂ norm of input activation across N×L tokens

> *"Motivated by the recent observation of emergent large magnitude features in LLMs, our approach prunes weights with the smallest magnitudes multiplied by the corresponding input activations, on a per-output basis."*

### Key Implementation

```python
def prune(W, X, s):
    metric = W.abs() * X.norm(p=2, dim=0)
    _, sorted_idx = torch.sort(metric, dim=1)
    pruned_idx = sorted_idx[:,:int(C_in * s)]
    W.scatter_(dim=1, index=pruned_idx, src=0)
    return W
```

### Per-Output Comparison: The Critical Insight

Even **magnitude-only** pruning improves dramatically when comparing per-output instead of layer-wise. This behavior is **unique to LLMs** — not seen in image classifiers.

| Comparison Group | LLaMA-7B @ 50% Perplexity |
|-----------------|---------------------------|
| Layer-wise (magnitude) | 17.29 |
| Per-output (magnitude) | 8.86 |
| **Per-output (Wanda)** | **7.26** |

### Why Wanda Works Better Than Magnitude Alone

1. **Activation Magnitudes Reflect Importance**: Large activations indicate important features; weights producing them should be retained
2. **Per-Output Competition**: By comparing within each output column, Wanda avoids comparing across structurally different parts of the network
3. **LLM-Specific Behavior**: Emergent large magnitude features in LLMs mean activation norms vary significantly per output

---

## Benchmarks

### WikiText Perplexity (Lower = Better)

| Sparsity | Method | LLaMA-7B | LLaMA-65B | LLaMA-2-70B |
|----------|--------|----------|-----------|-------------|
| Dense | — | 5.68 | 3.56 | 3.12 |
| 50% unstructured | Magnitude | 17.29 | 5.90 | 4.98 |
| 50% unstructured | SparseGPT | 7.22 | 4.57 | 3.98 |
| **50% unstructured** | **Wanda** | **7.26** | **4.57** | **3.98** |

### Zero-Shot Accuracy (Higher = Better)

| Method | LLaMA-7B | LLaMA-65B | LLaMA-2-70B |
|--------|----------|-----------|-------------|
| Dense | 59.99% | 66.97% | 67.08% |
| Magnitude (50%) | 46.94% | 62.74% | 60.93% |
| SparseGPT (50%) | 54.94% | 66.30% | 67.28% |
| **Wanda (50%)** | **54.21%** | **66.67%** | **67.03%** |

> *"At 50% unstructured sparsity, LLaMA-65B and LLaMA-2-70B **match their dense counterparts** in zero-shot accuracy."*

### Wanda vs SparseGPT: Speed Comparison

| Aspect | Magnitude | SparseGPT | Wanda |
|--------|-----------|-----------|-------|
| Weight Update | ✗ | ✓ | ✗ |
| Calibration Data | ✗ | ✓ | ✓ |
| Complexity | O(1) | O(d³) | O(d²) |
| **Pruning Speed (7B)** | — | 203.1s | **0.54s** |
| **Speedup** | — | 1× | **~300×** |

### Theoretical Connection to SparseGPT

Wanda's metric equals SparseGPT's when Hessian dampening λ=0 and only diagonal elements are retained.

---

## Follow-up Work

### Wanda++ — Regional Gradients (2025)

> **Paper:** [arXiv:2503.04992v4](https://arxiv.org/html/2503.04992v4)  
> **Authors:** Yang & Zhen et al. (UC Santa Barbara & Amazon AGI)

> *"Wanda++: Pruning Large Language Models via Regional Gradients"*

**Key Innovation:**
- Uses decoder-block-level regional gradients + regional optimization
- Up to **32% perplexity reduction** over Wanda (OpenLLaMA-3B, 2:4 sparsity)

**Trade-off:** More compute than Wanda but faster than full gradient methods

### M-Wanda — Multilingual LLMs (EMNLP 2025)

> **Paper:** [arXiv:2505.21171](https://arxiv.org/abs/2505.21171)  
> **Authors:** Rochelle Choenni, Ivan Titov (University of Amsterdam, Edinburgh)

> *"M-Wanda: Improving One-Shot Pruning for Multilingual LLMs"*

**Key Innovations:**
- Correlation-weighted layerwise sparsity allocation
- Cross-lingual activation variance
- Activation probability filtering

**Result:** 5–12% perplexity improvement over Wanda across 6 multilingual models

---

## Usage

### Installation & Basic Usage

```bash
git clone https://github.com/locuslab/wanda.git
cd wanda
python -m lib.prune \
    --model meta-llama/Llama-2-7b-hf \
    --prune_method wanda \
    --sparsity_ratio 0.5 \
    --calibration_samples 128
```

### Wanda with SparseGPT Comparison

```python
from lib.prune import prune_magnitude, prune_wanda, prune_sparsegpt

# Magnitude baseline
prune_magnitude(model, tokenizer, sparsity=0.5)

# Wanda (fast, no weight updates)
prune_wanda(model, tokenizer, sparsity=0.5)

# SparseGPT (slow, with weight updates)
prune_sparsegpt(model, tokenizer, sparsity=0.5)
```

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **Wanda (original)** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2306.11695) | Simple magnitude×activation one-shot pruning |
| **Wanda++** | 2025 | — | [arXiv](https://arxiv.org/html/2503.04992v4) | Regional gradients for improved 2:4 pruning |
| **M-Wanda** | 2025 | EMNLP 2025 | [arXiv](https://arxiv.org/abs/2505.21171) | Multilingual LLM adaptation |

---

## GitHub Repository

| Component | Path | Description |
|-----------|------|-------------|
| Core pruning | [lib/prune.py](https://github.com/locuslab/wanda) | `prune_magnitude()`, `prune_wanda()`, `prune_sparsegpt()`, `prune_ablate()` |
| Project page | [eric-mingjie.github.io/wanda](https://eric-mingjie.github.io/wanda/home.html) | Interactive demos and detailed results |

**Repository structure:**
```
wanda/
├── lib/
│   └── prune.py         # Core pruning implementations
├── run.sh               # Example scripts
└── README.md
```

---

## Key Quotes

> *"Wanda consistently finds effective pruned networks across various sparsity levels, without any weight updates."*

> *"Our analysis shows that the pruning configuration of Wanda delivers the best result among existing pruning metrics."*

> *"At 50% unstructured sparsity, LLaMA-65B and LLaMA-2-70B match their dense counterparts in zero-shot accuracy."*

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | One-shot unstructured pruning without expensive weight updates or retraining |
| **Key innovation** | Per-output comparison of magnitude×activation metric — simple yet highly effective |
| **Best for** | Fast LLM pruning when weight updates are too expensive; matches SparseGPT at 300× speed |
| **Limitation** | Still unstructured — hardware acceleration requires N:M semi-structured patterns |
| **Critical insight** | Per-output comparison matters enormously for LLMs (17.29 → 7.26 perplexity) |

---

*Last updated: 2026-04-19*
