# Knowledge Distillation for Quantization (QAD)

> **Papers:** [arXiv:1911.12491](https://arxiv.org/abs/1911.12491) (QKD) · [arXiv:2407.11062](https://arxiv.org/abs/2407.11062) (EfficientQAT, ACL 2025) · [arXiv:2403.11106](https://arxiv.org/abs/2403.11106) (SQAKD, AISTATS 2024) · [arXiv:2601.20088](https://arxiv.org/abs/2601.20088) (NVIDIA QAD)  
> **GitHub:** [OpenGVLab/EfficientQAT](https://github.com/OpenGVLab/EfficientQAT) · [kaiqi123/SQAKD](https://github.com/kaiqi123/SQAKD) · [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm) · [huawei-noam/TernaryBERT](https://github.com/huawei-noam/Pretrained-Language-Model/tree/master/TernaryBERT)  
> **Surveys:** [arXiv:2308.04268](https://arxiv.org/abs/2308.04268) (Teacher-Student Survey) · [MIT TACL](https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00704/125482/A-Survey-on-Model-Compression-for-Large-Language) (LLM Compression Survey)

---

## Overview

> *"Knowledge distillation enables a quantized student model to inherit the behavior of a full-precision teacher, effectively compensating for information loss during quantization."* — QKD (arXiv:1911.12491)

**Quantization-Aware Distillation (QAD)** combines knowledge distillation with quantization training. The core idea: use a **full-precision teacher** to guide a **quantized student**, allowing the student to recover quality that would otherwise be lost to quantization error.

---

## Core Concept: Teacher-Student for Quantization

```
┌─────────────────┐        ┌─────────────────┐
│   FP16 Teacher  │        │  Quantized Student │
│   (reference)   │   KD   │  (b-bit weights)   │
└─────────────────┘        └─────────────────┘
     logits/activations  →  mimics teacher
     soft targets         →  preserves dark knowledge
```

### Loss Function

Typical QAD loss combines:

| Component | Formula | Purpose |
|-----------|---------|---------|
| **KD Loss** | KL-divergence between teacher/student softmax | Transfer dark knowledge |
| **Task Loss** | Cross-entropy on labeled data | Maintain task performance |
| **Quantization Loss** | MSE between full and quantized representations | Penalize quantization gap |

---

## Key Methods

### 1. QKD: Quantization-Aware Knowledge Distillation (2019)

> **Paper:** [arXiv:1911.12491](https://arxiv.org/abs/1911.12491)  
> **Venue:** — | **Authors:** Yushi Kanai, Yasuhiro Fujita, Shinji Kimura

**TL;DR:** Original QKD paper — coordinates quantization and KD in three phases. The teacher is FP32, the student is quantized (e.g., INT8). Uses three-phase training:

| Phase | What Happens |
|-------|-------------|
| **Phase 1** | Train full-precision teacher |
| **Phase 2** | Generate soft targets using teacher |
| **Phase 3** | Train quantized student using both hard labels + soft targets |

---

### 2. TernaryBERT: Distillation-aware Ultra-low Bit BERT (EMNLP 2020)

> **Paper:** [arXiv:2009.12812](https://arxiv.org/abs/2009.12812)  
> **GitHub:** [huawei-noam/TernaryBERT](https://github.com/huawei-noam/Pretrained-Language-Model/tree/master/TernaryBERT)  
> **Venue:** EMNLP 2020 | **Authors:** Huawei Noah

**TL;DR:** Weight ternarization (+1, 0, −1) + distillation loss. Uses approximation-based AND loss-aware ternarization jointly optimized with task loss.

#### Architecture

```
BERT teacher (FP32)
        ↓ distillation
TernaryBERT student (ternary weights)
        ↓
Task loss + Distillation loss + Ternarization loss (jointly optimized)
```

---

### 3. TinyBERT: Distilling BERT for NLU (EMNLP 2020)

> **Paper:** [arXiv:1909.10351](https://arxiv.org/abs/1909.10351)  
> **GitHub:** [thunlp/TinyBERT](https://github.com/thunlp/TinyBERT)  
> **Venue:** EMNLP 2020 | **Authors:** Xiaoqi Jiao, Yichun Yin et al. (THUNLP, Huawei)

**TL;DR:** Two-stage distillation: (1) general pre-training distillation, (2) task-specific fine-tuning distillation. Not quantization-specific, but foundational for model compression via KD.

> **Note:** TinyBERT focuses on model缩小 (shrinking dimensions) rather than quantization. However, it established the two-stage KD paradigm later adopted by QAD methods.

#### Key Insight: Two-Stage KD

| Stage | Data | Purpose |
|-------|------|---------|
| **General KD** | Large corpus | Transfer general language knowledge |
| **Task KD** | Task-specific data | Transfer task-specific capabilities |

---

### 4. KDLSQ-BERT: Learned Step Size Quantization + KD (2021)

> **Paper:** [arXiv:2101.05938](https://arxiv.org/abs/2101.05938)  
> **Venue:** — | **Authors:** Yuncheng Jiang et al.

**TL;DR:** Combines **learned step size quantization (LSQ)** with knowledge distillation. The step size (quantization scale) is learned via gradient descent jointly with model weights, guided by KD loss from the teacher.

---

### 5. LLM-QAT: Data-Free QAT for LLMs (ICLR 2024)

> **Paper:** [arXiv:2305.17888](https://arxiv.org/abs/2305.17888) | **GitHub:** [facebookresearch/LLM-QAT](https://github.com/facebookresearch/LLM-QAT)  
> **Venue:** ICLR 2024 | **Authors:** Yanjing Wu et al. (Meta)

**TL;DR:** Data-free QAT — uses the **pre-trained model itself** to generate synthetic distillation data instead of requiring a labeled dataset. The quantized student learns from the FP32 teacher's behavior on synthetic generations.

> *"LLM-QAT enables quantization-aware training without any data collection or labeling, by using the pre-trained model to generate representative synthetic data."*

---

### 6. SQAKD: Self-Supervised Quantization-Aware KD (AISTATS 2024)

> **Paper:** [arXiv:2403.11106](https://arxiv.org/abs/2403.11106) | **GitHub:** [kaiqi123/SQAKD](https://github.com/kaiqi123/SQAKD)  
> **Venue:** AISTATS 2024 | **Authors:** Kai Qi et al.

**TL;DR:** Self-supervised QAT that co-optimizes **KL-divergence (KD) loss** and **discretization error** simultaneously. **No labeled data required.**

#### Key Innovation

| Traditional QAD | SQAKD |
|----------------|-------|
| Requires labeled data | Self-supervised — no labels needed |
| KD loss only | KD loss + discretization error jointly |
| Two-phase (calibrate → distill) | End-to-end joint optimization |

```python
# SQAKD loss = KL_Loss + λ * discretization_error
loss = kl_divergence(teacher_softmax, student_softmax) + λ * discretization_error
```

---

### 7. EfficientQAT: Efficient QAT for LLMs (ACL 2025)

> **Paper:** [arXiv:2407.11062](https://arxiv.org/abs/2407.11062) | **Paper PDF:** [aclanthology.org/2025.acl-long.498.pdf](https://aclanthology.org/2025.acl-long.498.pdf)  
> **GitHub:** [OpenGVLab/EfficientQAT](https://github.com/OpenGVLab/EfficientQAT)  
> **Venue:** ACL 2025 Main | **Authors:** Yusheng Han et al. (OpenGVLab)

**TL;DR:** Two key innovations: **Block-AP** (block-wise training of all parameters) and **E2E-QP** (end-to-end training of quantization parameters). Achieves **3.5–6.5× compression** with strong performance.

#### Block-AP vs Traditional QAT

| Aspect | Traditional QAT | EfficientQAT (Block-AP) |
|--------|----------------|-------------------------|
| **Parameters trained** | Only quantization params | ALL parameters in block |
| **Memory footprint** | High — full model in FP32 | Reduced — block-wise FP32 |
| **Training stability** | Less stable | More stable |
| **Final quality** | Good | Significantly better |

#### E2E-QP: End-to-End Quantization Parameters

Instead of using statistical calibration for quantization parameters (scales/zeros), EfficientQAT **learns them end-to-end** via gradient descent:

```python
# E2E-QP: quantization parameters are trainable
scale = nn.Parameter(torch.tensor([1.0]))  # learned, not computed
zero = nn.Parameter(torch.tensor([0.0]))  # learned, not computed
```

---

### 8. QAD for NVFP4: NVIDIA Technical Report (2025)

> **Paper:** [arXiv:2601.20088](https://arxiv.org/abs/2601.20088)  
> **Technical Report PDF:** [research.nvidia.com/labs/nemotron/files/NVFP4-QAD-Report.pdf](https://research.nvidia.com/labs/nemotron/files/NVFP4-QAD-Report.pdf)  
> **Authors:** NVIDIA

**TL;DR:** NVIDIA's QAD method for recovering accuracy of **NVFP4-quantized LLMs** using KL divergence. Works with SFT, RL, and merged models.

#### Method

| Step | Description |
|------|-------------|
| **1. Quantize** | Apply NVFP4 quantization to LLM |
| **2. Distill** | Use KL divergence to align quantized student with full-precision reference |
| **3. Recover** | Works across SFT/RL/merged model stages |

---

### 9. QD-LLM: Quantum Knowledge Distillation for LLMs (2025)

> **Paper:** [arXiv:2505.13205](https://arxiv.org/abs/2505.13205)  
> **Venue:** — | **Authors:** —

**TL;DR:** Explores quantum-inspired knowledge distillation for compressing LLMs with quantization constraints.

---

### 10. Muon-Optimized Distillation and Quantization (2025)

> **Paper:** [arXiv:2601.09865](https://arxiv.org/abs/2601.09865)  
> **Venue:** — | **Authors:** —

**TL;DR:** Combines Muon optimization with distillation and quantization for improved LLM deployment.

---

### 11. ZeroQuant: Post-Training Quantization + Low Rank Compensation (2022–2023)

> **Paper:** [2206.01861](https://arxiv.org/abs/2206.01861) (ZeroQuant) · [2303.08302](https://arxiv.org/abs/2303.08302) (ZeroQuant-V2)  
> **Venue:** — | **Authors:** (Microsoft)

**ZeroQuant-V2** adds **low-rank compensation** to ZeroQuant's post-training quantization:

> Adds a low-rank matrix B such that: `W_quantized = W_orig + B @ A` where B, A are low-rank and trained to compensate quantization error.

---

### 12. "Oh! We Freeze" — KD-QAT via Signal Propagation Analysis (ICLR 2024 Workshop)

> **Paper:** [arXiv:2403.18159](https://arxiv.org/abs/2403.18159)  
> **Venue:** ICLR 2024 Workshop | **Authors:** —

**TL;DR:** Lightweight QAT using signal propagation analysis to determine which layers to freeze during KD. Improves 4-bit quantized KD quality without full training.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **QKD** | 2019 | — | [arXiv](https://arxiv.org/abs/1911.12491) | Original QKD framework |
| **TernaryBERT** | 2020 | EMNLP | [arXiv](https://arxiv.org/abs/2009.12812) | Weight ternarization + KD |
| **TinyBERT** | 2020 | EMNLP | [arXiv](https://arxiv.org/abs/1909.10351) | Two-stage BERT distillation |
| **KDLSQ-BERT** | 2021 | — | [arXiv](https://arxiv.org/abs/2101.05938) | Learned step size + KD |
| **ZeroQuant** | 2022 | — | [arXiv](https://arxiv.org/abs/2206.01861) | PTQ + efficient kernel |
| **ZeroQuant-V2** | 2023 | — | [arXiv](https://arxiv.org/abs/2303.08302) | + Low-rank compensation |
| **LLM-QAT** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2305.17888) | Data-free QAT for LLMs |
| **"Oh! We Freeze"** | 2024 | ICLR W | [arXiv](https://arxiv.org/abs/2403.18159) | Signal propagation KD-QAT |
| **SQAKD** | 2024 | AISTATS | [arXiv](https://arxiv.org/abs/2403.11106) | Self-supervised QAD |
| **EfficientQAT** | 2024 | ACL 2025 | [arXiv](https://arxiv.org/abs/2407.11062) | Block-AP + E2E-QP |
| **QAD/NVFP4** | 2025 | — | [arXiv](https://arxiv.org/abs/2601.20088) | NVIDIA KL-based QAD |
| **QD-LLM** | 2025 | — | [arXiv](https://arxiv.org/abs/2505.13205) | Quantum-inspired KD |
| **Muon+QAD** | 2025 | — | [arXiv](https://arxiv.org/abs/2601.09865) | Muon optimization + QAD |

---

## GitHub Repositories

| Repo | Description | Stars |
|------|-------------|-------|
| [OpenGVLab/EfficientQAT](https://github.com/OpenGVLab/EfficientQAT) | ACL 2025: Block-AP + E2E-QP for LLMs. Model zoo with prequantized models. Supports Llama-2/3, Mistral | — |
| [kaiqi123/SQAKD](https://github.com/kaiqi123/SQAKD) | AISTATS 2024: Self-supervised QAT, no labeled data needed | — |
| [facebookresearch/LLM-QAT](https://github.com/facebookresearch/LLM-QAT) | ICLR 2024: Data-free QAT for LLMs | — |
| [thunlp/TinyBERT](https://github.com/thunlp/TinyBERT) | EMNLP 2020: Two-stage BERT distillation | — |
| [huawei-noam/TernaryBERT](https://github.com/huawei-noam/Pretrained-Language-Model/tree/master/TernaryBERT) | EMNLP 2020: Ternary weight BERT + distillation | — |
| [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm) | MiniLM: Deep self-attention distillation | — |
| [microsoft/LMOps/minillm](https://github.com/microsoft/LMOps/blob/main/minillm/README.md) | MiniLLM: KD of large language models | — |
| [aiha-lab/qllm-infer](https://github.com/aiha-lab/qllm-infer) | Quantization framework with KD support | — |
| [FLHonker/Awesome-Knowledge-Distillation](https://github.com/FLHonker/Awesome-Knowledge-Distillation) | Awesome list with QKD, TernaryBERT, many KD papers | — |
| [Tebmer/Awesome-Knowledge-Distillation-of-LLMs](https://github.com/Tebmer/Awesome-Knowledge-Distillation-of-LLMs) | Survey of KD techniques for LLMs | — |
| [Kai-Liu001/Awesome-Model-Quantization](https://github.com/Kai-Liu001/Awesome-Model-Quantization) | Comprehensive quantization papers with code | — |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **NVIDIA NVFP4 QAD Technical Report** | NVIDIA Research | [PDF](https://research.nvidia.com/labs/nemotron/files/NVFP4-QAD-Report.pdf) |
| **"TernaryBERT: Quantization Meets Distillation"** | Towards Data Science | [Link](https://towardsdatascience.com/ternarybert-quantization-meets-distillation-1b902ac31bd6/) |
| **"Quantization and Knowledge Distillation — Making Giant Models Fit in Your Pocket"** | Medium | [Link](https://medium.com/@kimminwoo190/week-final-quantization-and-knowledge-distillation-making-giant-models-fit-in-your-pocket-6b17d8fdf916) |
| **"Quantization-Aware Training for LLMs"** | Medium | [Link](https://medium.com/@rickboelen98/quantization-aware-training-for-llms-how-these-lightweight-models-are-becoming-mainstream-6cea4a67576c) |
| **"Knowledge Distillation for LLM Inference"** | Plain English / AI | [Link](https://ai.plainenglish.io/knowledge-distillation-for-llm-inference-compressing-large-models-for-production-7fd201419412) |
| **"Edge 459: Quantization Plus Distillation"** | TheSequence | [Link](https://thesequence.substack.com/p/edge-459-quantization-plus-distillation) |
| **"Model Compression, Quantization & Distillation"** | Medium | [Link](https://medium.com/@akankshasinha247/model-compression-quantization-distillation-b5006cf41546) |

---

## Surveys

| Title | URL |
|-------|-----|
| **Teacher-Student Architecture for Knowledge Distillation: A Survey** (arXiv:2308.04268) | [Link](https://arxiv.org/abs/2308.04268) |
| **A Survey on Model Compression for LLMs** (MIT TACL) | [Link](https://direct.mit.edu/tacl/article/doi/10.1162/tacl_a_00704/125482/A-Survey-on-Model-Compression-for-Large-Language) |
| **A survey of model compression techniques: past, present, and future** (Frontiers 2025) | [Link](https://www.frontiersin.org/journals/robotics-and-ai/articles/10.3389/frobt.2025.1518965/full) |

---

## How QAD Relates to QAT

| Aspect | QAT (Quantization-Aware Training) | QAD (Quantization-Aware Distillation) |
|--------|----------------------------------|----------------------------------------|
| **Supervision** | Task labels (hard labels) | Teacher model (soft labels) |
| **Data requirement** | Labeled dataset | FP32 teacher + dataset |
| **Typical use** | Training from scratch | Compressing a trained model |
| **Information source** | Ground truth labels | Dark knowledge from teacher |
| **Best for** | When labels are available | When teacher is better than labels |

> **Note:** QAT and QAD are complementary — QAT can be combined with KD loss (as in EfficientQAT), making them less mutually exclusive than they first appear.

---

## Framework Integration

| Framework | QAD Support | Notes |
|-----------|-------------|-------|
| **llama.cpp** | 🔄 | GGML No. 15 tracked [here](https://github.com/ggml-org/llama.cpp/discussions/20969) |
| **vLLM** | 🔄 | TQ integration tracked in [TQ repo](https://github.com/0xSero/turboquant) |
| **HuggingFace** | ✅ | PEFT supports QAT-style training; Transformers supports LLM-QAT |
| **llamafolio** | ❓ | No known QAD support |
| **MLX** | 🔄 | turboquant-mlx available (TurboQuant, not QAD-specific) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core idea** | Use FP32 teacher to guide quantized student, recovering lost quality |
| **Key methods** | QKD (2019) → TernaryBERT (2020) → LLM-QAT (2023) → SQAKD (2024) → EfficientQAT (2025) |
| **EfficientQAT** | Block-AP + E2E-QP: train all parameters block-wise, learn quantization params end-to-end |
| **LLM-QAT** | Data-free — generates synthetic distillation data from pre-trained model |
| **SQAKD** | Self-supervised — no labeled data needed, co-optimizes KD + discretization error |
| **Best for** | When a strong teacher exists; when labeled data is scarce (LLM-QAT); ultra-low-bit compression |
| **Open problems** | End-to-end QAT+KD training efficiency, memory footprint reduction for large models |

---

*Last updated: 2026-04-19*
