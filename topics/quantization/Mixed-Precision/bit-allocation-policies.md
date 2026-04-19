# Bit-Allocation Policies: Mixed-Precision Quantization for LLMs

> **Core Methods:** BAQ ([arXiv:2506.05664](https://arxiv.org/abs/2506.05664)) · RAMP ([arXiv:2603.17891](https://arxiv.org/abs/2603.17891)) · IMPQ ([arXiv:2509.15455](https://arxiv.org/html/2509.15455v1)) · SliM-LLM ([arXiv:2405.14917](https://arxiv.org/abs/2405.14917)) · AMQ ([arXiv:2509.12019](https://arxiv.org/html/2509.12019v1))  
> **GitHub:** [CSU-ModelCompression/BAQ](https://github.com/CSU-ModelCompression/BAQ) · [AIdevsmartdata/ramp-quant](https://github.com/AIdevsmartdata/ramp-quant) · [dlwns147/amq](https://github.com/dlwns147/amq)  
> **Surveys:** [arXiv:2510.16805](https://arxiv.org/html/2510.16805v1) · [pprp/awesome-llm-quantization](https://github.com/pprp/awesome-llm-quantization)

---

## Overview

> *"Not all layers are created equal. Some layers can survive 2-bit quantization while others need 8-bit. Bit-allocation policies answer the question: which layer gets how many bits?"* — BAQ (arXiv:2506.05664)

The core insight of mixed-precision quantization: **allocating different bit-widths to different layers** based on their sensitivity yields better quality than using a uniform bit-width for all layers.

**Why uniform quantization fails:**
```
Layer 1 (sensitive): 8-bit → wastefully high precision
Layer 15 (robust):  2-bit → unacceptably low quality
→ Same total bits, worse overall quality than mixed precision
```

---

## Why Mixed Precision Matters

### Per-Layer Sensitivity Varies Dramatically

| Layer Type | Sensitivity | Can Survive |
|------------|------------|-------------|
| **Embedding layers** | Very high | FP16 or INT8 only |
| **Early transformer layers** | High | INT8–INT4 |
| **Middle transformer layers** | Medium | INT4–INT2 |
| **Attention projection layers** | Medium-high | INT4–INT8 |
| **Output LM head** | Very high | FP16 only |
| **KV cache (recent tokens)** | Very high | FP16 or INT8 only |

### Quality vs Bit Budget (Illustrative)

| Strategy | Bits | Quality (pplx) |
|----------|------|---------------|
| All 4-bit | 4 | baseline |
| All 8-bit | 8 | +quality, +memory |
| **Mixed: some 2-bit, some 8-bit** | ~4 avg | **Better than both** |

---

## Core Methods

### 1. BAQ: Bit Allocation Quantization (2025)

> **arXiv:** [2506.05664](https://arxiv.org/abs/2506.05664) · **GitHub:** [CSU-ModelCompression/BAQ](https://github.com/CSU-ModelCompression/BAQ)

**Key contribution:** Formulates layer-wise bit allocation as a **convex optimization problem** with a **closed-form solution** using Hessian-based sensitivity.

> *"We prove that the optimal bit allocation can be computed in closed form — no search required. The Hessian diagonal provides all the sensitivity information needed."*

| Aspect | Details |
|--------|---------|
| **Method** | Hessian-based sensitivity → closed-form bit allocation |
| **Computation** | Negligible overhead vs standard PTQ |
| **Result** | Optimal mixed-precision assignment without search |
| **Supported formats** | INT2/INT4/INT8 per layer |

---

### 2. RAMP: Reinforcement Adaptive Mixed Precision (2024)

> **arXiv:** [2603.17891](https://arxiv.org/abs/2603.17891) · **GitHub:** [AIdevsmartdata/ramp-quant](https://github.com/AIdevsmartdata/ramp-quant)

**Key contribution:** Off-policy **Soft Actor-Critic (SAC)** framework that learns per-layer bit-width assignments to minimize perplexity under a global bit budget.

> *"RAMP uses 11-dimensional state capturing activation statistics, weight properties, and structural features to make bit-allocation decisions."*

| Aspect | Details |
|--------|---------|
| **Method** | Reinforcement learning (SAC) |
| **State space** | 11D — activation stats, weight properties, structural features |
| **Action space** | Per-layer bit-width selection |
| **Objective** | Minimize perplexity under bit budget |
| **GGUF support** | ✅ Yes |

---

### 3. IMPQ: Interaction-Aware Layerwise Mixed Precision (2025)

> **arXiv:** [2509.15455](https://arxiv.org/html/2509.15455v1)

**Key contribution:** Introduces **SPQE (Shapley-based Progressive Quantization Estimation)** — accounts for **inter-layer interactions** rather than treating layers independently.

> *"Traditional methods treat each layer in isolation. IMPQ models how quantization errors cascade and interact across layers using game-theoretic Shapley values."*

| Aspect | Details |
|--------|---------|
| **Method** | Shapley-based progressive quantization estimation |
| **Optimization** | Binary quadratic for bit allocation |
| **Key innovation** | Inter-layer interaction modeling |
| **Result** | 20–80% perplexity reduction over baselines at <4 bits average |

---

### 4. SliM-LLM: Salience-Driven Mixed-Precision (ICML 2025)

> **arXiv:** [2405.14917](https://arxiv.org/abs/2405.14917)

**Key contribution:** Group-wise mixed-precision allocation based on **salience distribution** across layers.

| Component | Description |
|-----------|-------------|
| **SBA** | Salience-Determined Bit Allocation |
| **SQC** | Salience-Weighted Quantizer Calibration |
| **Partitioning** | Hardware-friendly structured grouping |

> *"Salience measures how much each layer contributes to the final output quality. By allocating more bits to high-salience layers, we maximize quality under a bit budget."*

---

### 5. AMQ: AutoML for Mixed-Precision (EMNLP 2025)

> **arXiv:** [2509.12019](https://arxiv.org/html/2509.12019v1) · **GitHub:** [dlwns147/amq](https://github.com/dlwns147/amq)

**Key contribution:** **Multi-objective optimization** framework for automated bit-width selection — optimizes both model quality and inference efficiency simultaneously.

> *"AMQ uses automated search to find the best bit allocation, treating it as a multi-objective optimization problem rather than a simple constrained one."*

---

### 6. LLM-MQ: Mixed-Precision Quantization (NeurIPS 2023)

> **PDF:** [LLM-MQ paper](https://raw.githubusercontent.com/wln20/wln20.github.io/master/files/LLM_MQ.pdf) · **GitHub:** (related)

**Key contribution:** Sensitivity-based precision allocation using **first-order information** + quantization error. Includes sparse outlier protection for low-precision layers and CUDA kernels for mixed-precision GEMV.

| Aspect | Details |
|--------|---------|
| **Method** | First-order sensitivity + quantization error |
| **Outlier protection** | Sparse outlier handling for low-bit layers |
| **Hardware** | Custom CUDA kernels for mixed-precision GEMV |

---

### 7. CPTQuant: Correlation-Based Mixed Precision (2024)

> **arXiv:** [2412.03599](https://arxiv.org/abs/2412.03599)

**Key contribution:** Three complementary approaches for mixed-precision allocation:

| Method | Description |
|--------|-------------|
| **CMPQ** | Correlation-based precision allocation |
| **PMPQ** | Pruning-based mixed precision |
| **TDMPQ** | Taylor decomposition-based |

---

### 8. Atom: Low-Bit Quantization for LLM Serving (MLSys 2024)

> **arXiv:** [2310.19102](https://arxiv.org/abs/2310.19102) · **GitHub:** [efeslab/Atom](https://github.com/efeslab/Atom)

**Key contribution:** Mixed-precision + fine-grained group quantization + dynamic activation quantization.

| Metric | Result |
|--------|--------|
| **Throughput improvement** | 7.73× over FP16 |
| **Approach** | Per-layer mixed precision + per-group weight quantization |

---

### 9. ResQ: Mixed-Precision with Low-Rank Residuals (ICLR 2025)

> **arXiv:** [2412.14363](https://arxiv.org/html/2412.14363v2)

**Key contribution:** PCA-based low-rank subspace identification. Keeps 1/8 channels in 8-bit, rest in 4-bit. Uses random rotation for outlier suppression.

> *"ResQ reduces perplexity by 33% compared to SpinQuant by identifying which channels are most sensitive and protecting them with higher precision."*

---

## KV-Cache Specific Bit Allocation

### KVTuner: Layerwise Mixed-Precision KV Cache (ICML 2025)

> **arXiv:** [2502.04420](https://arxiv.org/abs/2502.04420) · **GitHub:** [cmd2001/KVTuner](https://github.com/cmd2001/KVTuner)

**Key contribution:** Adapts per-layer K/V quantization precision pairs via multi-objective optimization.

> *"KVTuner automatically finds the optimal per-layer K and V precision — no manual tuning. For Llama-3.1-8B it finds 3.25-bit mixed precision KV cache."*

| Model | KV Cache Precision |
|-------|-------------------|
| **Llama-3.1-8B** | 3.25-bit mixed precision |
| **Qwen2.5-7B** | 4.0-bit mixed precision |

---

### MixKVQ: Query-Aware Mixed-Precision KV Cache (2025)

> **arXiv:** [2512.19206](https://arxiv.org/html/2512.19206v1)

**Key contribution:** Query-aware algorithm that preserves **critical key channels at higher precision** based on intrinsic quantization difficulty + query relevance.

> *"Not all key channels matter equally for all queries. MixKVQ allocates higher precision to key channels that are most relevant to the current query."*

---

### Don't Waste Bits! (2025)

> **arXiv:** [2604.04722](https://arxiv.org/html/2604.04722v1)

**Key contribution:** **Token-wise precision allocation** via a lightweight controller network that predicts precision assignments per token.

> *"A small neural network learns to predict which tokens need higher precision and which can be compressed more aggressively."*

---

### AsymKV: Layer-Wise Asymmetric KV Quantization (COLING 2025)

> **arXiv:** [2410.13212](https://arxiv.org/abs/2410.13212)

**Key contribution:** Asymmetric K (per-channel) vs V (per-token) quantization. Enables **1-bit KV cache** via layer-wise asymmetric configs.

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **LLM-MQ** | 2023 | NeurIPS | [PDF](https://raw.githubusercontent.com/wln20/wln20.github.io/master/files/LLM_MQ.pdf) | First-order sensitivity + custom CUDA kernels |
| **QuIP** | 2023 | NeurIPS | [link](https://neurips.cc/virtual/2023/poster/69982) | 2-bit via incoherence processing |
| **Atom** | 2024 | MLSys | [2310.19102](https://arxiv.org/abs/2310.19102) | 7.73× throughput, mixed + group + dynamic |
| **SliM-LLM** | 2024 | ICML 2025 | [2405.14917](https://arxiv.org/abs/2405.14917) | Salience-driven mixed precision |
| **RAMP** | 2024 | arXiv | [2603.17891](https://arxiv.org/abs/2603.17891) | RL (SAC) for bit allocation |
| **AMQ** | 2025 | EMNLP 2025 | [2509.12019](https://arxiv.org/html/2509.12019v1) | AutoML multi-objective search |
| **IMPQ** | 2025 | arXiv | [2509.15455](https://arxiv.org/html/2509.15455v1) | Shapley inter-layer interactions |
| **BAQ** | 2025 | arXiv | [2506.05664](https://arxiv.org/abs/2506.05664) | Closed-form Hessian-based allocation |
| **ResQ** | 2025 | ICLR 2025 | [2412.14363](https://arxiv.org/html/2412.14363v2) | Low-rank residual + PCA subspace |

---

## GitHub Repositories

| Repo | Description | Stars |
|------|-------------|-------|
| [CSU-ModelCompression/BAQ](https://github.com/CSU-ModelCompression/BAQ) | Closed-form Hessian-based bit allocation | — |
| [AIdevsmartdata/ramp-quant](https://github.com/AIdevsmartdata/ramp-quant) | RAMP implementation for GGUF models | — |
| [dlwns147/amq](https://github.com/dlwns147/amq) | AMQ AutoML mixed-precision | — |
| [Cornell-RelaxML/QuIP](https://github.com/Cornell-RelaxML/QuIP) | 2-bit via incoherence processing | — |
| [Cornell-RelaxML/qtip](https://github.com/Cornell-RelaxML/qtip) | QTIP trellis-coded quantization | — |
| [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) | AWQ — saliency-aware weight quantization | 3.5k |
| [efeslab/Atom](https://github.com/efeslab/Atom) | Low-bit serving with mixed precision | — |
| [cmd2001/KVTuner](https://github.com/cmd2001/KVTuner) | KV cache mixed precision search | — |
| [pprp/Awesome-LLM-Quantization](https://github.com/pprp/awesome-llm-quantization) | Comprehensive quantization paper list | — |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **"Mixed-Precision Quantization for LLMs"** | HuggingFace Blog | [Link](https://huggingface.co/blog/royswastik/llm-quantization) |
| **"Precision to Quantization Guide"** | DeepInfra | [Link](https://deepinfra.com/blog/precision-to-quantization-faster-cheaper-llms) |
| **"Mixed Precision Quantization Survey"** | AussieAI | [Link](https://www.aussieai.com/research/mixed-precision-quantization) |
| **"LLM Quantization: AWQ, GPTQ, SmoothQuant"** | BentoML | [Link](https://bentoml.com/llm/getting-started/llm-quantization) |
| **"Visual Guide to Quantization"** | Maarten Grootendorst | [Link](https://maartengrootendorst.com/blog/quantization/) |
| **"Per-Layer Quantization Sensitivity"** | Maarten Grootendorst | [Link](https://maartengrootendorst.com/blog/quantization/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core problem** | Uniform bit allocation is suboptimal — layers have different sensitivity |
| **Key methods** | Hessian-based (BAQ), RL-based (RAMP), Shapley-based (IMPQ), salience-based (SliM-LLM), AutoML (AMQ) |
| **KV cache** | KVTuner (ICML 2025), MixKVQ, Don't Waste Bits! — per-layer/token KV precision |
| **Best for** | Maximizing quality under a fixed memory/budget constraint |
| **Common finding** | ~1% of weights are "salient" and need higher precision (AWQ insight) |

---

*Last updated: 2026-04-19*
