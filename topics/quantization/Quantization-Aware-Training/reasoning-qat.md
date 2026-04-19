# Reasoning-QAT: Quantization-Aware Training for Reasoning Models

> **Papers:** [arXiv:2601.14888](https://arxiv.org/abs/2601.14888) (ICLR 2026) · [arXiv:2505.11574](https://arxiv.org/abs/2505.11574) (ICLR 2026) · [arXiv:2501.03035](https://arxiv.org/abs/2501.03035) · [arXiv:2504.04823](https://arxiv.org/abs/2504.04823) (COLM 2025)  
> **GitHub:** [ruikangliu/Quantized-Reasoning-Models](https://github.com/ruikangliu/Quantized-Reasoning-Models) · [yasu0001/ReasoningQAT](https://github.com/yasu0001/ReasoningQAT)  
> **Related:** [ICLR 2026 Poster](https://iclr.cc/virtual/2026/poster/10010985) · [NeurIPS 2025](https://neurips.cc/virtual/2025/126555)

---

## Overview

> *"Quantization disproportionately elevates method/execution errors — not conceptual mistakes. Failures cascade from the first faulty step of chain-of-thought reasoning."* — Li et al., arXiv:2501.03035

Reasoning models (DeepSeek-R1, QwQ-32B, etc.) generate long chain-of-thought sequences — small per-token quantization errors **accumulate and cascade**, making them significantly more vulnerable to quantization degradation than standard LLMs.

---

## Why Reasoning Models Are Different

| Property | Standard LLM | Reasoning Model |
|----------|-------------|-----------------|
| **Output length** | Short (512–2K tokens) | Long (8K–128K tokens) |
| **Error accumulation** | Minimal | Severe — errors cascade |
| **Training method** | SFT | RL (DeepSeek-R1) or Distillation (R1-Distill) |
| **Quantization sensitivity** | Moderate | **Very high** — especially on hard tasks |
| **Calibration data** | Generic text | Domain-specific reasoning traces |

### Quantization Degradation on Hard Tasks

| Task | FP16 Baseline | W4A16 | Degradation |
|------|--------------|-------|-------------|
| AIME-120 (hard math) | ~60% | ~20% | **−40%** |
| MATH-500 | ~90% | ~75% | **−15%** |
| GSM8K (simple math) | ~95% | ~92% | **−3%** |

> Quantization degradation scales with task difficulty — harder tasks suffer up to **4× greater degradation** than simpler ones. (Liu et al., COLM 2025)

### RL-based vs Distilled Models

| Model Type | Example | Quantization Vulnerability |
|------------|---------|--------------------------|
| **RL-trained** | QwQ-32B | **More vulnerable** — reward signals amplify with quantization noise |
| **Distillation-based** | DeepSeek-R1-Distill (1.5B–70B) | More robust — inherits teacher behavior |

---

## Key Papers

### 1. "What Makes Low-Bit QAT Work for Reasoning LLMs? A Systematic Study" (ICLR 2026)

> **arXiv:** [2601.14888](https://arxiv.org/abs/2601.14888) | **HTML:** [arxiv.org/html/2601.14888v1](https://arxiv.org/html/2601.14888v1)  
> **Authors:** Keyu Lv, Manyi Zhang, Xiaobo Jia et al. (Tsinghua, Huawei, NUS)

**TL;DR:** Systematic study of QAT for reasoning models under aggressive low-bit (2–3 bit) settings. Identifies four critical factors:

#### Four Key Findings

| Finding | Description | Impact |
|---------|-------------|--------|
| **KD >> SFT** | Knowledge Distillation preserves uncertainty structure; SFT causes 2× more degradation | +46% improvement with KD warm-up |
| **PTQ Init** | GPTQ/FlatQuant initialization dramatically improves accuracy vs naive RTN | Faster convergence, better final quality |
| **Cold-Start RL** | Direct RL on heavily quantized models collapses; KD warm-up is essential | Without it: complete training failure |
| **Domain Alignment** | PTQ calibration data must align with QAT training data domain | +9.81% improvement on 1.5B model |

#### Proposed: Reasoning-QAT Pipeline

```
Stage 1: PTQ Initialization (GPTQ/FlatQuant)
    ↓
Stage 2: Knowledge Distillation Warm-up (KD on reasoning data)
    ↓
Stage 3: Cold-Start RL Fine-tuning
```

#### Results

| Model | Setting | GPTQ | Reasoning-QAT | Improvement |
|-------|---------|------|---------------|-------------|
| R1-Qwen-1.5B | W3G128 | 38.07% | **43.26%** | **+5.19%** |
| Qwen3-0.6B | W3G128 | — | **44.53%** on MATH-500 | — |

---

### 2. "Towards Quantization-Aware Training for Ultra-Low-Bit Reasoning LLMs" (ICLR 2026)

> **arXiv:** [2505.11574](https://arxiv.org/abs/2505.11574) | **OpenReview:** [openreview.net/forum?id=Azsd2qyK6C](https://openreview.net/forum?id=Azsd2qyK6C) | **GitHub:** [yasu0001/ReasoningQAT](https://github.com/yasu0001/ReasoningQAT)  
> **ICLR 2026 Poster:** [iclr.cc/virtual/2026/poster/10010985](https://iclr.cc/virtual/2026/poster/10010985)  
> **Authors:** Yasuyuki Okoshi, Hikari Otsuka, Daichi Fujiki, Masato Motomura

**TL;DR:** Two-stage QAT pipeline for ultra-low-bit (2-bit) reasoning models. Outperforms PTQ baselines by **50.45%** on average and beats BitNet-2B4T with fewer than 1B tokens.

#### Two-Stage Design

| Stage | Method | Purpose |
|-------|--------|---------|
| **Stage 1** | Mixed-domain calibration | Preserves abilities across domains |
| **Stage 2** | Teacher-guided reward-rectification loss | Restores reasoning capability |

#### Results

- **2-bit Qwen3-8B** outperforms all PTQ baselines by **50.45%** average
- Outperforms **BitNet-2B4T** by ~2% with fewer than 1B parameters

---

### 3. "Quantization Hurts Reasoning? An Empirical Study on Quantized Reasoning Models" (COLM 2025)

> **arXiv:** [2504.04823](https://arxiv.org/abs/2504.04823) | **HTML:** [arxiv.org/html/2504.04823v2](https://arxiv.org/html/2504.04823v2)  
> **GitHub:** [ruikangliu/Quantized-Reasoning-Models](https://github.com/ruikangliu/Quantized-Reasoning-Models)  
> **HuggingFace:** [DeepSeek-R1-Distill Quantized Collection](https://huggingface.co/collections/ruikangliu/deepseek-r1-distill-quantized-68357b2a87b1a76137ad20d0)  
> **Authors:** Ruikang Liu, Yuxuan Sun, Manyi Zhang, Haoli Bai et al. | **Venue:** COLM 2025

**TL;DR:** First comprehensive empirical study on quantized reasoning models. Evaluated DeepSeek-R1-Distill (1.5B–70B) and QwQ-32B across math, science, and code benchmarks.

#### Key Findings

| Setting | Safety Threshold | Notes |
|---------|----------------|-------|
| **W8A8KV8** | ✅ Lossless | ≤1% drop across all model sizes |
| **W4A16** | ✅ Near-lossless | On 14B+ models only |
| **W4A4KV4** | ⚠️ Risky | Significant degradation, especially on small models |
| **W4A16KV4** | ⚠️ Moderate risk | Depends on model size |

#### Calibration Data Domain Matters

> Using **reasoning-domain calibration data** instead of pre-training data for GPTQ improves 1.5B model by **+9.81%** on reasoning tasks.

---

### 4. "Quantization Meets Reasoning: Where Does Degradation Occur?" (ICLR 2026)

> **arXiv:** [2501.03035](https://arxiv.org/abs/2501.03035) | **Updated:** [2505.11574](https://arxiv.org/abs/2505.11574)  
> **OpenReview:** [openreview.net/forum?id=So3hbnEGYV](https://openreview.net/forum?id=So3hbnEGYV)  
> **Authors:** Zhen Li, Yupeng Su, Songmiao Wang et al.

**TL;DR:** First systematic study on *where* degradation occurs along chain-of-thought steps. Key insight: failures cascade from the **first faulty step**.

#### Key Findings

| Observation | Detail |
|-------------|--------|
| **Degradation type** | PTQ disproportionately elevates *method/execution errors* vs conceptual mistakes |
| **Cascading failure** | Errors don't just accumulate — they compound; first wrong step ruins all subsequent reasoning |
| **Numerical vs planning** | Quantization affects numerical computation differently than reasoning planning |

#### "Silver Bullet" Intervention

> As few as **332 curated examples** and **3–5 minutes** of single-GPU compute can recover 4-bit weight math reasoning to near full-precision levels.

---

### 5. "The Impact of Quantization on Large Reasoning Models" (NeurIPS 2025)

> **Venue:** NeurIPS 2025 | **URL:** [neurips.cc/virtual/2025/126555](https://neurips.cc/virtual/2025/126555)

**Key Finding:** Quantization-aware RL training **negatively impacted the learning process**, whereas PTQ and QLoRA led to greater stability. This aligns with cold-start RL findings above.

---

### 6. "I-LLM: Efficient Integer-Only Inference for Fully-Quantized Low-Bit LLMs" (ICLR 2025)

> **arXiv:** [2405.17849](https://arxiv.org/abs/2405.17849) | **OpenReview:** [openreview.net/forum?id=44pbCtAdLx](https://openreview.net/forum?id=44pbCtAdLx)  
> **Venue:** ICLR 2025

**Key Contribution:** Enables **fully integer-only inference** for LLMs — no floating-point operations. Proposes **Fully-Smooth Block-Reconstruction (FSBR)** for inter-channel variations and **DI-MatMul** for dynamic integer-only matrix multiplication.

---

### 7. "Low-Rank QAT for LLMs" (LR-QAT, ICLR 2025)

> **arXiv:** [2406.06385](https://arxiv.org/abs/2406.06385) | **OpenReview:** [openreview.net/forum?id=dIK0EfZFO9](https://openreview.net/forum?id=dIK0EfZFO9)

**Key Contribution:** Memory-efficient QAT inspired by LoRA. Reduces 7B LLM QAT memory from **>98GB to <21GB** without performance degradation. Uses low-rank auxiliary weights aware of quantization grid.

---

### 8. "EfficientQAT: Efficient Quantization-Aware Training for LLMs" (ACL 2025)

> **arXiv:** [2407.11062](https://arxiv.org/abs/2407.11062) | **Paper:** [aclanthology.org/2025.acl-long.498.pdf](https://aclanthology.org/2025.acl-long.498.pdf)  
> **GitHub:** [OpenGVLab/EfficientQAT](https://github.com/OpenGVLab/EfficientQAT) | **Venue:** ACL 2025

**Key Contribution:** Block-wise training of all parameters (Block-AP) + end-to-end training of quantization parameters (E2E-QP). Achieves significant improvements over existing uniform quantization methods.

---

## Lossless Quantization Thresholds for Reasoning

| Setting | Small Models (<7B) | Medium (7B–14B) | Large (14B+) |
|---------|-------------------|-----------------|--------------|
| **W8A8KV8** | ✅ Lossless | ✅ Lossless | ✅ Lossless |
| **W4A16** | ⚠️ Some degradation | ✅ Near-lossless | ✅ Near-lossless |
| **W4A4KV4** | ❌ Severe degradation | ⚠️ Moderate | ⚠️ Moderate |
| **W4A16KV4** | ⚠️ Moderate | ✅ Near-lossless | ✅ Near-lossless |

> **Rule of thumb:** W4A4KV4 requires FlatQuant or similar outlier suppression for reasoning models to work.

---

## Best Practices for Reasoning-QAT

### ✅ Do This

| Practice | Why |
|----------|-----|
| Use **Knowledge Distillation** as QAT objective, not SFT | KD preserves uncertainty structure; SFT causes 2× more degradation |
| Initialize with **PTQ** (GPTQ/FlatQuant) | Dramatically improves accuracy and convergence speed |
| Always do **KD cold-start before RL** | Direct RL on heavily quantized models collapses |
| Use **reasoning-domain calibration data** aligned with QAT training data | Domain alignment improves by +9.81% |
| For ultra-low-bit (<4-bit): mixed-domain calibration + teacher-guided fine-tuning | Okoshi et al. (ICLR 2026) shows +50.45% improvement |

### ❌ Don't Do This

| Mistake | Consequence |
|---------|-------------|
| Apply RL directly to heavily quantized models | Complete training failure |
| Use generic pre-training data for calibration | Reasoning tasks degrade significantly |
| Use SFT as QAT objective | 2× more degradation vs KD |
| Expect W4A4KV4 to work without outlier suppression | Severe reasoning degradation |

---

## GitHub Repositories

| Repo | Paper | Description |
|------|-------|-------------|
| [ruikangliu/Quantized-Reasoning-Models](https://github.com/ruikangliu/Quantized-Reasoning-Models) | COLM 2025 | Quantized DeepSeek-R1-Distill models + evaluation code |
| [yasu0001/ReasoningQAT](https://github.com/yasu0001/ReasoningQAT) | ICLR 2026 | Two-stage ultra-low-bit QAT for reasoning models |
| [facebookresearch/LLM-QAT](https://github.com/facebookresearch/LLM-QAT) | ICLR 2024 | Original Meta LLM-QAT (general, not reasoning-specific) |
| [OpenGVLab/EfficientQAT](https://github.com/OpenGVLab/EfficientQAT) | ACL 2025 | Block-AP + E2E-QP for efficient QAT |
| [Kai-Liu001/Awesome-Model-Quantization](https://github.com/Kai-Liu001/Awesome-Model-Quantization) | — | Comprehensive quantization paper list |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **"What Makes Low-Bit QAT Work for Reasoning LLMs" — Full Review** | themoonlight.io | [Link](https://www.themoonlight.io/en/review/what-makes-low-bit-quantization-aware-training-work-for-reasoning-llms-a-systematic-study) |
| **"Deployment-ready reasoning with quantized DeepSeek-R1 models"** | Red Hat Developers | [Link](https://developers.redhat.com/articles/2025/03/03/deployment-ready-reasoning-quantized-deepseek-r1-models) |
| **"Quantization-Aware Training for LLMs: How lightweight models are becoming mainstream"** | Medium | [Link](https://medium.com/@rickboelen98/quantization-aware-training-for-llms-how-these-lightweight-models-are-becoming-mainstream-6cea4a67576c) |
| **"Understanding Reasoning LLMs"** | Sebastian Raschka | [Link](https://magazine.sebastianraschka.com/p/understanding-reasoning-llms) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core Problem** | Reasoning models are significantly more vulnerable to quantization than standard LLMs due to error accumulation in long CoT sequences |
| **Key Vulnerability** | W4A4KV4 — significant degradation; W8A8KV8 — lossless across all sizes |
| **Critical Insight** | KD >> SFT as QAT objective; cold-start RL is essential before applying RL to quantized reasoning models |
| **Recovery** | 332 curated examples + 3–5 min fine-tuning can recover 4-bit weight reasoning to near-FP16 levels |
| **Best Practice** | PTQ init → KD warm-up → Cold-start RL → Domain-aligned calibration data |
| **Open Problems** | Ultra-low-bit (<3-bit) reasoning, W4A4KV4 robustness, RL-QAT stability |

---

*Last updated: 2026-04-19*
