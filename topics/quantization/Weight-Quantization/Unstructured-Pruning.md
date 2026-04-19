# Unstructured Pruning

> **Papers:** [Deep Compression](https://arxiv.org/abs/1510.00149) · [Lottery Ticket Hypothesis](https://arxiv.org/abs/1803.03635) · [SparseGPT](https://arxiv.org/abs/2301.00774) · [Wanda](https://arxiv.org/abs/2306.11695) · [DSnoT](https://openreview.net/forum?id=1ndDmZdT4g)  
> **GitHub:** [locuslab/wanda](https://github.com/locuslab/wanda) · [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt) · [BaiTheBest/SparseLLM](https://github.com/BaiTheBest/SparseLLM) · [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner)  
> **Blogs:** [ApX Magnitude Pruning](https://apxml.com/courses/llm-compression-acceleration/chapter-3-sophisticated-pruning-methodologies/magnitude-based-pruning) · [LLM Pruning Guide](https://datamagiclab.com/llm-pruning-a-comprehensive-guide-to-model-compression/) · [Lottery Ticket Explained](https://sebastianraschka.com/faq/docs/lottery-ticket.html)

---

## Overview

> *"Dense, randomly-initialized, feed-forward networks contain subnetworks ('winning tickets') that—when trained in isolation—reach test accuracy comparable to the original network in a similar number of iterations."* — Frankle & Carbin, [ICLR 2019](https://arxiv.org/abs/1803.03635)

Unstructured pruning removes individual weights anywhere in the network without respecting any structural pattern. This creates sparse networks where specific weights are set to zero based on an importance criterion.

| Aspect | Details |
|--------|---------|
| **Method** | Individual weights set to zero based on importance metric |
| **Sparsity Level** | Can achieve 50–90%+ sparsity |
| **Hardware Efficiency** | Poor on standard GPUs — requires specialized sparse kernels |
| **Accuracy** | Degrades at very high sparsity without fine-tuning |
| **Fine-tuning Need** | Essential for recovery at high sparsity |

---

## Key Methods

### 1. Magnitude Pruning (2015)

> **Paper:** [Deep Compression](https://arxiv.org/abs/1510.00149) (Han et al., ICLR 2016)  
> **GitHub:** [songhan/Deep-Compression-AlexNet](https://github.com/songhan/Deep-Compression-AlexNet)

**TL;DR:** Remove weights with smallest absolute values — the simplest and most foundational pruning method.

**Procedure:**
1. Calculate absolute magnitude `|w|` for every weight
2. For target sparsity `S`, find the `(S×100)`-th percentile threshold
3. Set all weights below threshold to zero
4. Optionally fine-tune to recover performance

> *"Deep Compression: Compressing Deep Neural Networks with Pruning, Trained Quantization and Huffman Coding."* — Han et al.

| Aspect | Details |
|--------|---------|
| **Complexity** | O(1) — simply sort magnitudes |
| **Accuracy at 50%** | Severely degrades (LLaMA-2-7B: 5.12 → 14.89 perplexity) |
| **Key Limitation** | Magnitude ≠ Importance; fails beyond ~10% sparsity |

### 2. Iterative Magnitude Pruning (IMP)

A gradual approach that mitigates accuracy drop through cycles:

1. **Prune** — Remove 5–10% of currently active weights with lowest magnitudes
2. **Fine-tune** — Retrain pruned model for limited epochs
3. **Repeat** — Continue until desired sparsity achieved

> *"Winning tickets are less than 10–20% of the size of fully-connected and convolutional architectures."* — Frankle & Carbin

### 3. SparseGPT (ICML 2023)

> **Paper:** [arXiv:2301.00774](https://arxiv.org/abs/2301.00774) · **GitHub:** [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt)

**TL;DR:** First method to show large-scale LLMs can be pruned to 50%+ sparsity in one-shot without retraining.

> *"We show for the first time that large-scale generative pretrained transformer (GPT) family models can be compressed to high sparsity via weight pruning in one-shot, without any retraining, at low loss of accuracy."*

**Method:** Uses weight freezing interpretation with greedy column-wise reconstruction based on Optimal Brain Surgeon (OBS) framework. Updates remaining weights to compensate for pruned ones.

**Results on LLaMA-2 (50% unstructured sparsity):**

| Model | Dense | Magnitude | SparseGPT | Wanda |
|-------|-------|-----------|-----------|-------|
| LLaMA-2-7B | 5.12 | 14.89 | 6.51 | **6.42** |
| LLaMA-2-13B | 4.57 | 6.37 | 5.63 | **5.56** |

### 4. Wanda — Weight and Activation Pruning (ICLR 2024)

> **Paper:** [arXiv:2306.11695](https://arxiv.org/abs/2306.11695) · **GitHub:** [locuslab/wanda](https://github.com/locuslab/wanda) · **Project Page:** [eric-mingjie.github.io/wanda](https://eric-mingjie.github.io/wanda/home.html)

**TL;DR:** Simple one-shot pruning using `metric = |W| × ||X||₂` (per output column). No retraining required. Outperforms magnitude pruning dramatically.

> *"Wanda significantly outperforms the established baseline of magnitude pruning and performs competitively against recent method involving intensive weight update."*

**Core Implementation:**
```python
def prune(W, X, s):
    metric = W.abs() * X.norm(p=2, dim=0)
    _, sorted_idx = torch.sort(metric, dim=1)
    pruned_idx = sorted_idx[:,:int(C_in * s)]
    W.scatter_(dim=1, index=pruned_idx, src=0)
    return W
```

**Why Per-Output Matters:**

| Comparison Group | LLaMA-7B 50% Perplexity |
|-----------------|------------------------|
| Layer-wise (magnitude) | 17.29 |
| Per-output (magnitude) | 8.86 |
| **Per-output (Wanda)** | **7.26** |

Even magnitude pruning improves dramatically with per-output comparison — this behavior is **unique to LLMs**, not seen in image classifiers.

### 5. DSnoT — Dynamic Sparse No Training (ICLR 2024)

> **Paper:** [OpenReview](https://openreview.net/forum?id=1ndDmZdT4g)

> *"DSNT minimizes the reconstruction error between the dense and sparse LLMs, in the fashion of performing iterative..."*

**Significance:** Addresses the practical challenge of fine-tuning sparse LLMs without expensive backpropagation.

---

## Sparsity Patterns

| Pattern | Description | Hardware Support |
|---------|-------------|-----------------|
| **Unstructured** | Individual weights set to zero anywhere | ❌ Poor on GPUs |
| **Semi-Structured (N:M)** | 2:4 sparsity: 2 zeros per every 4 weights (50%) | ✅ NVIDIA Ampere sparse tensor cores (~2× speedup) |
| **Structured** | Remove entire neurons/heads/layers | ✅ Dense matrix operations |

---

## Pruning Workflow

```
ORIGINAL DENSE LLM
        │
        ▼
IMPORTANCE SCORING & RANKING
(Magnitude, Wanda, Taylor, Hessian-based)
        │
        ▼
PRUNING MASK CREATION
(Global or Layer-wise thresholding)
        │
        ▼
ONE-SHOT PRUNING
(Set weights to zero via mask)
        │
   ┌────┴────┐
   │         │
   ▼         ▼
DIRECT USE  POST-TRAINING
(One-shot)  (LoRA/Full FT)
```

---

## Common Benchmarks

| Benchmark | What it Measures |
|-----------|-----------------|
| **WikiText-2/PTB** | Perplexity |
| **BoolQ, PIQA, HellaSwag, WinoGrande, ARC** | Common sense reasoning (7 tasks from EleutherAI LM eval harness) |
| **MMLU** | Massive Multitask Language Understanding |
| **Zero-shot Accuracy** | Average across multiple tasks |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **Deep Compression** | 2015 | ICLR 2016 | [arXiv](https://arxiv.org/abs/1510.00149) | Foundational magnitude pruning + quantization + Huffman coding |
| **Lottery Ticket Hypothesis** | 2018 | ICLR 2019 | [arXiv](https://arxiv.org/abs/1803.03635) | Dense networks contain sparse subnetworks that match performance |
| **SparseGPT** | 2023 | ICML 2023 | [arXiv](https://arxiv.org/abs/2301.00774) | First one-shot 50%+ pruning for large LLMs using OBS framework |
| **Wanda** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2306.11695) | Simple magnitude×activation pruning, 300× faster than SparseGPT |
| **LLM-Pruner** | 2023 | NeurIPS 2023 | [arXiv](https://arxiv.org/abs/2305.11627) | Structural pruning framework for LLMs (not pure unstructured) |
| **SliceGPT** | 2024 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2401.15024) | Computational-level pruning via PCA |
| **SparseLLM** | 2024 | NeurIPS 2024 | [arXiv](https://arxiv.org/abs/2402.17946) | Global unstructured + semi-structured pruning framework |
| **DSnoT** | 2024 | ICLR 2024 | [OpenReview](https://openreview.net/forum?id=1ndDmZdT4g) | Training-free fine-tuning for sparse LLMs |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [locuslab/wanda](https://github.com/locuslab/wanda) | Wanda implementation (ICLR 2024) |
| [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt) | SparseGPT one-shot pruning for GPT-family models |
| [horseee/LLM-Pruner](https://github.com/horseee/LLM-Pruner) | Structural pruning for LLMs (NeurIPS 2023) |
| [BaiTheBest/SparseLLM](https://github.com/BaiTheBest/SparseLLM) | Global unstructured + semi-structured pruning |
| [songhan/Deep-Compression-AlexNet](https://github.com/songhan/Deep-Compression-AlexNet) | Original Deep Compression implementation |
| [horseee/Awesome-Efficient-LLM](https://github.com/horseee/Awesome-Efficient-LLM) | Comprehensive efficient LLM paper list |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Magnitude-Based Pruning** | ApX | [Link](https://apxml.com/courses/llm-compression-acceleration/chapter-3-sophisticated-pruning-methodologies/magnitude-based-pruning) |
| **LLM Pruning: A Comprehensive Guide** | DataMagic Lab | [Link](https://datamagiclab.com/llm-pruning-a-comprehensive-guide-to-model-compression/) |
| **Techniques for Efficient Inference of LLMs** | Medium/MantisNLP | [Link](https://medium.com/mantisnlp/techniques-for-efficient-inference-of-llms-ii-iv-5324f3dad69c) |
| **Lottery Ticket Hypothesis Explained** | Sebastian Raschka | [Link](https://sebastianraschka.com/faq/docs/lottery-ticket.html) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Reduces model size and theoretical FLOPs by removing individual unimportant weights |
| **Key innovation** | One-shot pruning at 50%+ sparsity with minimal accuracy loss (SparseGPT, Wanda) |
| **Best for** | Scenarios where specialized sparse kernels are available; achieving maximum sparsity |
| **Key insight** | LLMs have effective sparse sub-networks discoverable via magnitude×activation metrics |
| **Hardware challenge** | Irregular sparsity doesn't map well to GPU/TPU — semi-structured (2:4) preferred for actual speedups |

---

*Last updated: 2026-04-19*
