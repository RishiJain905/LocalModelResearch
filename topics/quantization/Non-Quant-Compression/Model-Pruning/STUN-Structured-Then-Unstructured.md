# STUN — Structured-Then-Unstructured Pruning

> **Papers:** [STUN](https://arxiv.org/abs/2409.06211) · [SparseGPT](https://arxiv.org/abs/2301.00774) · [OWL](https://arxiv.org/html/2310.05175v4) · [MaskLLM](https://arxiv.org/abs/2409.17481) · [DaSS](https://arxiv.org/html/2405.01943v1) · [MoE-Pruner](https://arxiv.org/abs/2410.12013)  
> **GitHub:** [thnkinbtfly/STUN](https://github.com/thnkinbtfly/STUN) · [NVlabs/MaskLLM](https://github.com/NVlabs/MaskLLM) · [luuyin/OWL](https://github.com/luuyin/OWL) · [Paramathic/patch](https://github.com/Paramathic/patch)  
> **Blogs:** [NVIDIA — Accelerating Inference with Sparsity](https://developer.nvidia.com/blog/accelerating-inference-with-sparsity-using-ampere-and-tensorrt/)

---

## Overview

> *"We propose STUN, the first method to combine structured and unstructured pruning, outperforming both approaches."* — Lee et al., [ACL 2025](https://arxiv.org/abs/2409.06211)

STUN is the **first method combining structured (expert) pruning followed by unstructured pruning** that outperforms both approaches individually — a counterintuitive finding since unstructured pruning's solution space subsumes structured pruning's.

| Aspect | Details |
|--------|---------|
| **Approach** | Structured (expert) pruning → Unstructured (weight) pruning |
| **Key Insight** | Expert pruning preserves weight kurtosis, maintaining robustness to subsequent unstructured pruning |
| **Scalability** | O(1) expert pruning via behavioral similarity clustering |
| **Performance** | Nearly no loss at 40% sparsity on Snowflake Arctic (480B) |

---

## Why STUN Works: Kurtosis Preservation

**Unstructured pruning removes near-zero weights** → shifts weight distribution toward bimodal → **lowers kurtosis** → reduces margin for further pruning

**Expert (structured) pruning removes entire experts** → remaining weights still follow Gaussian → **kurtosis preserved**

This means the order matters: structured-first preserves the distribution characteristics that unstructured pruning needs to remain effective.

---

## Key Methods

### 1. STUN — Structured-Then-Unstructured (ACL 2025)

> **Paper:** [arXiv:2409.06211](https://arxiv.org/abs/2409.06211) · **Code:** [github.com/thnkinbtfly/STUN](https://github.com/thnkinbtfly/STUN)  
> **Venue:** ACL 2025 Main Conference  
> **Authors:** Jaeseong Lee, Seung-won Hwang, Aurick Qiao, Daniel Campos, Zhewei Yao, Yuxiong He (Snowflake AI Research, Seoul National University)

**O(1) Expert Pruning Algorithm:**

Prior work (Lu et al., 2024) requires **O(k^n/√n)** forward passes — for Snowflake Arctic (128 experts) this equals ~2.4×10²⁵ forward passes per layer, completely infeasible.

**STUN's Three-Step Reduction:**
1. **Probabilistic Reformulation (→ O(n))**: Reframe combinatorial search as maximizing joint probability, leverage latent expert clusters
2. **Behavioral Similarity Clustering**: `b_{i,j} = -λ₁||W_i - W_j||_F + λ₂a_{i,j}` using weight-based and coactivation statistics
3. **1st-Order Taylor Approximation + Selective Reconstruction (→ O(1))**: Avoids OOM-inducing Hessian computation

**Experimental Results:**

*Snowflake Arctic (480B, 128 experts) at 40% sparsity:*

| Method | GSM8K | ARC-c | Avg |
|--------|-------|-------|-----|
| Unpruned | 70.74 | 56.91 | 64.86 |
| **STUN (w/ OWL)** | **70.28** | **57.68** | **64.75** |
| OWL alone | 63.76 | 56.74 | 63.47 |
| **STUN (w/ Wanda)** | **69.60** | **57.25** | **64.81** |
| Wanda alone | 64.59 | 57.00 | 63.32 |

**Key Result:** STUN achieves nearly no loss at 40% sparsity with only **1 H100 and 2 hours** on Snowflake Arctic.

*Mixtral-8x7B at 65% sparsity:*

| Method | GSM8K | Avg |
|--------|-------|-----|
| **STUN (w/ OWL)** | **25.09** | **60.34** |
| OWL alone | 1.29 | 45.20 |

### 2. OWL — Outlier Weighed Layerwise Sparsity

> **Paper:** [arXiv:2310.05175v4](https://arxiv.org/html/2310.05175v4) · **Code:** [github.com/luuyin/OWL](https://github.com/luuyin/OWL)

> *"OWL exhibits a remarkable performance gain, surpassing the state-of-the-art Wanda and SparseGPT by **61.22 and 6.80 perplexity** at a high sparsity level of 70%, respectively."*

**Key Discovery:** Strong correlation between pruning efficacy and outlier retention. LLMs exhibit activation outliers (features with magnitudes up to 20× larger) crucial for model performance.

**Layerwise Outlier Distribution (LOD)** follows a "U" shape: high outlier ratios at beginning/end layers, monotonic descent in middle. OWL assigns non-uniform sparsity ratios based on LOD.

### 3. MaskLLM — Learnable N:M Semi-Structured Sparsity (NeurIPS 2024)

> **Paper:** [arXiv:2409.17481](https://arxiv.org/abs/2409.17481) · **Code:** [github.com/NVlabs/MaskLLM](https://github.com/NVlabs/MaskLLM)

> *"Leading approaches achieve perplexity (PPL) of **10+** on Wikitext vs. dense model's **5.12**, but MaskLLM achieves **6.72 PPL** solely by learning masks with frozen weights."*

**Method:** Models N:M mask selection as a learnable distribution via Gumbel Softmax sampling, enabling end-to-end training on large-scale datasets.

**Results at 2:4 sparsity:**

| Model | Dense PPL | SparseGPT | Wanda | **MaskLLM** |
|-------|-----------|-----------|-------|-------------|
| LLaMA-2 7B | 5.12 | 10.42 | 11.29 | **6.72** |
| LLaMA-2 13B | 4.57 | 8.20 | 8.47 | **5.85** |

### 4. DaSS — Dependency-Aware Semi-Structured Sparsity

> **Paper:** [arXiv:2405.01943](https://arxiv.org/html/2405.01943v1)

> *"DaSS surpasses state-of-the-art LLM pruning methods like SparseGPT and Wanda in achieving hardware-friendly N:M sparsity patterns."*

**Key Finding:** Contrary to traditional beliefs, outliers play a diminished role in input projections of GLU-based MLPs. DaSS balances unstructured flexibility with dependency-based structured consistency.

### 5. PATCH — Tile-Level Hybrid Sparsity

> **Paper:** [arXiv:2509.23410](https://arxiv.org/abs/2509.23410) · **Code:** [github.com/Paramathic/patch](https://github.com/Paramathic/patch)

> *"PATCH partitions weight matrices into tiles, assigning each tile to be either dense or 2:4 sparse via a learnable mask selection mechanism... achieving higher accuracy, hardware efficiency, and speedups."*

**Key Innovation:** Hybrid sparsity framework enabling continuous sparsity ratio between 0% and 50%, bridging unstructured and 2:4 semi-structured approaches.

### 6. MoE-Pruner — Expert-Level MoE Pruning

> **Paper:** [arXiv:2410.12013](https://arxiv.org/abs/2410.12013)

> *"We propose MoE-Pruner, a method that prunes weights with the smallest magnitudes multiplied by the corresponding input activations and router weights."*

**Key Feature:** Incorporates MoE router weight information to identify unimportant weights in expert layers, combined with expert-wise knowledge distillation for performance recovery.

---

## Hardware: N:M Semi-Structured Sparsity

### NVIDIA Ampere Sparse Tensor Cores

> *"Accelerating Inference with Sparsity Using the NVIDIA Ampere Architecture and NVIDIA TensorRT"* — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/accelerating-inference-with-sparsity-using-ampere-and-tensorrt/)

- **2:4 fine-grained structured sparsity**: Every 4 consecutive elements has exactly 2 non-zero values
- Up to **2× speedup** on A100 GPUs with sparse tensor cores
- Three-step sparse retraining workflow maintains baseline accuracy

| Pattern | Hardware Support | Typical Speedup |
|---------|------------------|-----------------|
| **Unstructured** | Requires specialized kernels | Theoretical, often not realized |
| **2:4 Semi-Structured** | NVIDIA Ampere+ sparse tensor cores | ~2× on A100 |
| **4:8** | NVIDIA Ampere+ (extended) | ~1.5–2× |

---

## Pruning Performance Comparison

### STUN vs. Other MoE Pruning Methods

| Method | Inter-Expert | Intra-Expert | Complexity |
|--------|--------------|--------------|------------|
| Lu et al. (2024) | ✓ | ✗ | O(k^n/√n) — infeasible |
| LLM-Pruner, Wanda, OWL | ✗ | ✓ | Layer-wise |
| **STUN (Ours)** | **✓** | **✓** | **O(1)** |

### 70% Sparsity Results (LLaMA-V1)

| Method | LLaMA-7B | LLaMA-13B | LLaMA-30B |
|--------|----------|-----------|-----------|
| Dense | 62.40 | 65.61 | 67.91 |
| Wanda | 39.39 | 41.25 | 54.66 |
| **OWL + Wanda** | **46.47** | **48.88** | **55.53** |
| SparseGPT | 44.93 | 48.38 | 55.78 |
| **OWL + SparseGPT** | **48.03** | **50.85** | **57.11** |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **STUN** | 2025 | ACL 2025 | [arXiv](https://arxiv.org/abs/2409.06211) | First structured→unstructured hybrid pruning, O(1) expert pruning |
| **OWL** | 2023 | — | [arXiv](https://arxiv.org/html/2310.05175v4) | Outlier-weighted layerwise sparsity, U-shaped LOD |
| **MaskLLM** | 2024 | NeurIPS 2024 | [arXiv](https://arxiv.org/abs/2409.17481) | Learnable N:M masks via Gumbel Softmax |
| **DaSS** | 2024 | — | [arXiv](https://arxiv.org/html/2405.01943v1) | Dependency-aware semi-structured sparsity |
| **PATCH** | 2025 | — | [arXiv](https://arxiv.org/abs/2509.23410) | Tile-level hybrid sparsity 0–50% |
| **MoE-Pruner** | 2024 | — | [arXiv](https://arxiv.org/abs/2410.12013) | Router-weighted expert pruning + KD |
| **AST** | 2024 | AAAI 2025 | [arXiv](https://arxiv.org/abs/2407.20584) | Adaptive sparse training, 1.12% zero-shot gap |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [thnkinbtfly/STUN](https://github.com/thnkinbtfly/STUN) | Official STUN implementation |
| [NVlabs/MaskLLM](https://github.com/NVlabs/MaskLLM) | Learnable N:M semi-structured sparsity |
| [luuyin/OWL](https://github.com/luuyin/OWL) | Outlier weighed layerwise sparsity |
| [IST-DASLab/SparseGPT](https://github.com/IST-DASLab/SparseGPT) | One-shot GPT pruning (baseline for STUN) |
| [ModelTC/Wanda](https://github.com/ModelTC/Wanda) | Weight and activation pruning (baseline for STUN) |
| [Paramathic/patch](https://github.com/Paramathic/patch) | Tile-level hybrid sparsity |
| [horseee/Awesome-Efficient-LLM](https://github.com/horseee/Awesome-Efficient-LLM/blob/main/pruning.md) | Comprehensive pruning resources list |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Accelerating Inference with Sparsity Using NVIDIA Ampere and TensorRT** | NVIDIA Developer Blog | [Link](https://developer.nvidia.com/blog/accelerating-inference-with-sparsity-using-ampere-and-tensorrt/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Combines benefits of structured (hardware efficiency) and unstructured (high sparsity) pruning |
| **Key innovation** | Kurtosis preservation through structured-first ordering; O(1) expert pruning via behavioral clustering |
| **Best for** | MoE models (STUN), hybrid N:M patterns, anyone needing structured+unstructured tradeoffs |
| **Why order matters** | Structured pruning preserves weight distribution; unstructured destroys it |

---

*Last updated: 2026-04-19*
