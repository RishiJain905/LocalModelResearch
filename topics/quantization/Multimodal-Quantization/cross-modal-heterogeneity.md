# Cross-Modal Heterogeneity in Multimodal LLM Quantization

> **Key Papers:** MBQ ([arXiv:2412.19509](https://arxiv.org/abs/2412.19509), CVPR 2025) · MASQuant ([arXiv:2603.04800](https://arxiv.org/abs/2603.04800)) · QSLAW ([arXiv:2408.03735](https://arxiv.org/abs/2408.03735)) · Modality-Aware Quantization ([OpenReview](https://openreview.net/pdf?id=n0pSH3hOJA))  
> **GitHub:** [thu-nics/MBQ](https://github.com/thu-nics/MBQ) · [alibaba/EfficientAI](https://github.com/alibaba/EfficientAI)  
> **Blog:** [Red Hat: Faster Vision-Language Models with Quantization](https://developers.redhat.com/articles/2025/04/01/enable-faster-vision-language-models-quantization)

---

## Overview

> *"The average absolute gradient value of language token features is over 10× larger than that of vision token features. Language tokens might impact the loss function more than 10× as much as vision tokens."* — MBQ (arXiv:2412.19509)

**Cross-modal heterogeneity** is the observation that vision and language modalities have fundamentally different **quantization sensitivity profiles**. This challenges the assumption that standard LLM quantization techniques transfer directly to multimodal models.

---

## The Visual Dominance Hypothesis (Actually: Language Is More Sensitive)

### Counter-Intuitive Finding

The key discovery from MBQ and subsequent research:

| Token Type | Gradient Magnitude | Quantization Sensitivity |
|------------|-------------------|-------------------------|
| **Language tokens** | 10–22× larger | **HIGH** — small errors cause large loss impact |
| **Vision tokens** | 1× baseline | **LOW** — high redundancy, fault-tolerant |

> *"Visual token activations reach ~±40 while textual tokens peak at ~±0.2 — a two-order-of-magnitude disparity. Despite this, semantic importance of language tokens makes them more quantization-sensitive."* — Modality-Aware Quantization (ICLR 2026 under review)

### Why Not Visual Dominance?

| Aspect | Reason |
|--------|--------|
| **Vision has higher magnitude** | True — activations 10–100× larger than text |
| **Vision is NOT more sensitive** | Redundancy absorbs quantization errors |
| **Language is more sensitive** | Semantic content requires precise representation |

---

## Activation Magnitude Disparity

| Observation | Detail |
|------------|--------|
| **Visual token activations** | ~±40 (large dynamic range) |
| **Textual token activations** | ~±0.2 (small, focused) |
| **Ratio** | 100–200× larger for vision |

This makes standard **SmoothQuant catastrophic for non-text modalities**:
- SmoothQuant migrates difficulty to weights — designed for balanced modalities
- Audio WER with SmoothQuant: **3.9 → 77.4** (20× degradation)
- MASQuant maintains: **3.6** (barely affected)

---

## Key Methods

### 1. MBQ: Modality-Balanced Quantization (CVPR 2025)

> **arXiv:** [2412.19509](https://arxiv.org/abs/2412.19509) · **GitHub:** [thu-nics/MBQ](https://github.com/thu-nics/MBQ)  
> **Authors:** Tsinghua-NICS Lab

**Key contribution:** Uses gradients of SFT loss w.r.t. vision and language token features as sensitivity indicators to balance quantization across modalities.

> *"Treating vision and language tokens equally during calibration over-emphasizes insensitive vision tokens, causing significant accuracy loss. MBQ uses gradient-based weighting to rebalance."*

| Result | Improvement |
|--------|------------|
| **W3 setting** | +4.4% over SOTA baselines |
| **W4A8 setting** | +11.6% over SOTA baselines |

#### MBQ Results on Qwen2-VL-7B

| Method | T.VQA↑ | D.VQA↑ | OCRBench↑ | MME↑ |
|--------|--------|--------|-----------|------|
| **BF16** | 84.43 | 93.87 | 842 | 2319 |
| Quarot | 71.44 | 83.96 | 670 | 1911 |
| **MBQ** | **84.32** | **93.58** | **824** | **2255** |

---

### 2. MASQuant: Modality-Aware Smoothing Quantization (2025)

> **arXiv:** [2603.04800](https://arxiv.org/abs/2603.04800) · **GitHub:** [alibaba/EfficientAI](https://github.com/alibaba/EfficientAI)

**Key contribution:** Learns **separate per-modality smoothing factors**. SmoothQuant's unified smoothing over-smooths non-dominant modalities.

| Method | Libri WER↓ | OmniBench↑ |
|--------|-----------|------------|
| **FP16** | 3.9 | 43.8 |
| **SmoothQuant W4A8** | 77.4 | 27.3 |
| **MBQ W4A8** | 9.5 | 36.7 |
| **MASQuant W4A8** | **3.6** | **41.4** |

---

### 3. QSLAW: Quantization-Aware Scale Learning (ACM MM 2024)

> **arXiv:** [2408.03735](https://arxiv.org/abs/2408.03735) · **GitHub:** [xjjxmu/QSLAW](https://github.com/xjjxmu/QSLAW)

**Key contribution:** Activation outliers markedly increase in multimodal inputs. NF4 overlooks these outliers, causing accumulated quantization errors.

| Innovation | Description |
|-----------|-------------|
| **Group-wise learnable scales** | Per-channel/group scale factors |
| **Multimodal warmup** | Blend linguistic + multimodal samples |

> *"QSLAW outperforms full-precision LLaVA-13B (91.04% vs 90.92%) with only 16.83% of LoRA's parameters."*

---

### 4. VLMQ: Token Saliency-Driven PTQ (2025)

> **arXiv:** [2508.03351](https://arxiv.org/abs/2508.03351)

**Key contribution:** Visual over-representation (excessive redundant vision tokens) and modality gap degrade quantization.

| Finding | Detail |
|---------|--------|
| **Redundant vision tokens** | Vision dominates activation space |
| **Solution** | Down-weight 50% of vision tokens |
| **Result** | DocVQA +0.39% improvement |

---

## Practical Deployment Strategy

### Per-Component Precision Allocation

| Component | Precision | Reason |
|-----------|-----------|--------|
| **ViT encoder** | W4A8 | Compute-bound; can use aggressive quantization |
| **VLM decode stage** | W3 weight-only | Memory-bound (GEMV); W3 is sweet spot |
| **VLM prefill** | W4A8 | Long inputs benefit from activation quantization |
| **Language backbone** | W4–W6, W8A8 | Highest sensitivity — most bits needed |

---

## Framework Support

### vLLM Multimodal Quantization

> **Docs:** [vllm.ai/projects/llm-compressor/en/0.8.1/examples/multimodal_vision/](https://docs.vllm.ai/projects/llm-compressor/en/0.8.1/examples/multimodal_vision/)

```python
from llm_compressor import parse_recursive
from modifiers import GPTQModifier, SmoothQuantModifier

# Sequential targets for different components
parse_recursive(
    model,
    sequential_targets=["llm", "vision"],
    modifiers=[GPTQModifier(precision="int4")]
)
```

### TensorRT-LLM

> **GitHub:** [NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM)

Supports LLaVA, Qwen-VL, InternVL2 with quantization (FP8, W4A8 AWQ, W4A16 AWQ).

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **MBQ** | 2025 | CVPR | [2412.19509](https://arxiv.org/abs/2412.19509) | Gradient-based modality balancing |
| **MASQuant** | 2025 | — | [2603.04800](https://arxiv.org/abs/2603.04800) | Per-modality smoothing factors |
| **QSLAW** | 2024 | ACM MM | [2408.03735](https://arxiv.org/abs/2408.03735) | Learnable scales + multimodal warmup |
| **VLMQ** | 2025 | — | [2508.03351](https://arxiv.org/abs/2508.03351) | Token saliency + redundancy elimination |
| **Modality-Aware** | 2026 | ICLR (review) | [OpenReview](https://openreview.net/pdf?id=n0pSH3hOJA) | Visual dominance, adaptive 3-stage pipeline |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [thu-nics/MBQ](https://github.com/thu-nics/MBQ) | CVPR 2025: Modality-Balanced Quantization |
| [alibaba/EfficientAI](https://github.com/alibaba/EfficientAI) | MASQuant, MQuant, E2VQG |
| [xjjxmu/QSLAW](https://github.com/xjjxmu/QSLAW) | ACM MM 2024: Quantization-aware scale learning |
| [NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) | NVIDIA's VLM inference with quantization |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **"Enable 3.5× Faster Vision-Language Models with Quantization"** | Red Hat Developers | [Link](https://developers.redhat.com/articles/2025/04/01/enable-faster-vision-language-models-quantization) |
| **"Quantizing Multimodal VLMs"** | vLLM Docs | [Link](https://docs.vllm.ai/projects/llm-compressor/en/0.8.1/examples/multimodal_vision/) |
| **"Multimodal Model Quantization Support"** | Red Hat | [Link](https://developers.redhat.com/articles/2025/02/19/multimodal-model-quantization-support-through-llm-compressor) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Key finding** | Language tokens are 10–22× MORE sensitive to quantization than vision tokens |
| **Why** | Language tokens have higher gradient impact; vision has redundancy |
| **Activation disparity** | Vision: ±40, Text: ±0.2 — 100–200× difference |
| **Standard smoothing fails** | SmoothQuant degrades audio from 3.9 → 77.4 WER |
| **Solution** | Per-modality sensitivity weighting (MBQ) or per-modality smoothing (MASQuant) |
| **Deployment** | ViT: W4A8 (compute-bound), VLM decode: W3 (memory-bound), LLM: W4–W6 |

---

*Last updated: 2026-04-19*
