# ZeroQuant: Microsoft Research Activation Quantization

> **ZeroQuant:** [arXiv:2206.01861](https://arxiv.org/abs/2206.01861) (NeurIPS 2022)  
> **ZeroQuant-V2:** [arXiv:2303.08302](https://arxiv.org/abs/2303.08302) (Low-Rank Compensation)  
> **ZeroQuant-FP:** [arXiv:2307.09782](https://arxiv.org/abs/2307.09782) (FP8/FP4 activation formats)  
> **GitHub:** [microsoft/DeepSpeed](https://github.com/microsoft/DeepSpeed) (integrated in DeepSpeed Compression)  
> **Blog:** [Microsoft Research Blog](https://www.microsoft.com/en-us/research/publication/zeroquant-efficient-and-affordable-post-training-quantization-for-large-scale-transformers/) · [DeepSpeed Compression Blog](https://www.microsoft.com/en-us/research/blog/deepspeed-compression-a-composable-library-for-extreme-compression-and-zero-cost-quantization/)

---

## Overview

> *"ZeroQuant enables INT8 weight and activation quantization with up to 5.19×/4.16× speedup over FP16 inference, with negligible accuracy loss."* — Yao et al., NeurIPS 2022

**ZeroQuant** from Microsoft Research is a **post-training quantization (PTQ)** framework targeting both **weight quantization** and **activation quantization** in transformer-based LLMs. The name "ZeroQuant" reflects its goal: zero-overhead quantization without retraining.

**Note:** ZeroQuant is **not** the same as "ZeroShot" (a computer vision/NLP concept) or "FlowZero" (a video synthesis method — unrelated).

---

## Original ZeroQuant (NeurIPS 2022)

> **arXiv:** [2206.01861](https://arxiv.org/abs/2206.01861) · **PDF:** [arxiv.org/pdf/2206.01861](https://arxiv.org/pdf/2206.01861)  
> **Presented:** NeurIPS 2022 · **Authors:** Zhewei Yao, Sourabh K. et al. (Microsoft Research)

### Core Innovations

| Innovation | Description |
|------------|-------------|
| **Post-training INT8** | No retraining needed — applied directly to pre-trained models |
| **Weight + Activation** | Both weights AND activations quantized to INT8 |
| **Hardware kernels** | Highly optimized INT8 kernels for transformer layers |
| **Layer-by-Layer KD (LKD)** | Enables INT4 weight quantization by combining with knowledge distillation |
| **5.19× speedup** | Measured on BERT and GPT-style models |

### Speedup Results

| Model | Speedup vs FP16 |
|-------|----------------|
| **BERT-base** | 5.19× |
| **GPT-NeoX-20B** | 4.16× |
| **GPT-J-6B** | 3.4× |

> *"Combined with LKD (Layer-by-Layer Knowledge Distillation), ZeroQuant enables INT4 weight + INT8 activation quantization while maintaining accuracy."*

### How ZeroQuant Works

1. **Per-tensor or per-channel** quantization for weights
2. **Per-token** quantization for activations (groups tokens, not channels)
3. **Statistical calibration** using 128–512 calibration samples
4. **Fused kernels** to minimize dequantization overhead

```python
# ZeroQuant quantization scheme
weight_quant:  per-tensor or per-channel INT8
activation_quant: per-token INT8 (token-grouped)
calibration: 128-512 samples, using min/max or MSE
```

### Limitations Addressed by V2

> *"The original ZeroQuant paper showed that for small models (<1B parameters), simple RTN quantization works well. But larger models need more sophisticated approaches."* — [ZeroQuant-V2 arXiv:2303.08302](https://arxiv.org/abs/2303.08302)

---

## ZeroQuant-V2: Low-Rank Compensation (2023)

> **arXiv:** [2303.08302](https://arxiv.org/abs/2303.08302) · **PDF:** [arxiv.org/pdf/2303.08302](https://arxiv.org/pdf/2303.08302)  
> **Title:** "Exploring Post-training Quantization in LLMs from Comprehensive Study to Low Rank Compensation"  
> **Authors:** Zhewei Yao, et al. (Microsoft Research)

### Key Innovation: Low-Rank Compensation (LoRC)

Adds a **low-rank matrix decomposition** to compensate for quantization error:

```
W_quantized = W_orig + B @ A
```

Where:
- `W_orig` = original quantized weights
- `B @ A` = low-rank correction matrices (trainable)
- `rank(B @ A) << rank(W_orig)` — minimal parameter overhead

> *"LoRC exploits the observation that quantization error has low-rank structure — we can correct it with a small low-rank matrix rather than full-precision weights."*

### Size-Dependent Recommendations from ZeroQuant-V2

| Model Size | Weight Quant | Activation Quant | Method |
|------------|-------------|-----------------|--------|
| **< 1B** | W8 (RTN) | W8A8 (RTN) | Round-to-nearest sufficient |
| **1B – 10B** | W8 (per-row) | W8A8 (fine-grained) | Per-row + group-wise |
| **> 10B** | W4 (group-wise) | W8A8 | LoRC + fine-grained |

### ZeroQuant-V2 Results

| Model | Setting | Accuracy (PTQ) | Accuracy (+ LoRC) | Δ |
|-------|---------|---------------|-------------------|---|
| **OPT-13B** | W4A8 | 23.1 | **24.8** | +1.7 |
| **BLOOM-7B** | W4A8 | 22.5 | **24.1** | +1.6 |
| **GPT-J-6B** | W4A8 | 24.1 | **25.3** | +1.2 |

---

## ZeroQuant-FP: Floating-Point Quantization (2023)

> **arXiv:** [2307.09782](https://arxiv.org/abs/2307.09782) · **ar5iv HTML:** [ar5iv.labs.arxiv.org/html/2307.09782](https://ar5iv.labs.arxiv.org/html/2307.09782)  
> **Title:** "ZeroQuant-FP: A Leap Forward in LLMs Post-Training W4A8 Quantization Using Floating-Point Formats"

### Key Finding

> *"FP8 activation consistently outperforms INT8 for LLMs. FP4 shows advantages over INT4 in certain cases due to its ability to represent values near zero more accurately."*

| Format | Weight | Activation | Use Case |
|--------|--------|-----------|----------|
| **FP8 (E4M3)** | ✅ | ✅ | High-precision activation quantization |
| **FP8 (E5M2)** | ✅ | ✅ | Wider range, lower precision |
| **FP4 (E2M1)** | ✅ | ❌ | Limited but effective for some weights |
| **INT8** | ✅ | ✅ | Standard — used in original ZeroQuant |

### ZeroQuant-FP + LoRC

Combines floating-point formats with Low-Rank Compensation:

```
W4A8-FP: Weights in FP4/INT4, Activations in FP8
LoRC applied to both to mitigate quantization error
```

---

## DeepSpeed Compression Integration

ZeroQuant is integrated into **DeepSpeed Compression** — Microsoft's composable library for extreme model compression.

> **Tutorial:** [deepspeed.ai/tutorials/model-compression/](https://www.deepspeed.ai/tutorials/model-compression/)  
> **GitHub:** [github.com/microsoft/DeepSpeed](https://github.com/microsoft/DeepSpeed)

### DeepSpeed Compression Components

| Component | Description |
|-----------|-------------|
| **ZeroQuant** | INT8 weight + activation quantization |
| **ZeroQuant-V2 (LoRC)** | + Low-rank compensation for W4A8 |
| **ZeroQuant-FP** | + Floating-point format support |
| **Layer-by-Layer KD (LKD)** | Knowledge distillation for INT4 weight quantization |
| **Hardware kernels** | Optimized CUDA kernels for INT8/INT4 |

### Usage in DeepSpeed

```python
from deepspeed.compression import compress_model
from deepspeed.compression.basics import zeroquant_dtype, zeroquantV2_dtype

# ZeroQuant (INT8)
compress_model(model, 
               dtype=zeroquant_dtype(),
               layer_name='linear')

# ZeroQuant-V2 with LoRC (INT4 + INT8)
compress_model(model,
               dtype=zeroquantV2_dtype(),
               layer_name='linear',
               LoRC=True)
```

### ⚠️ Important Note

> The **original ZeroQuant inference engine** was never publicly released as a standalone project. It was integrated directly into DeepSpeed. See: [DeepSpeed Issue #2207](https://github.com/microsoft/DeepSpeed/issues/2207)

A **third-party reimplementation** is available: [kunalkumar168/ZeroQuant](https://github.com/kunalkumar168/ZeroQuant)

---

## Layer-by-Layer Knowledge Distillation (LKD)

Combined with knowledge distillation to enable **INT4 weight quantization**:

```
Teacher: FP16 full model
Student: INT4 weight + INT8 activation

LKD: Distill layer-by-layer, not end-to-end
→ Each layer learns from corresponding teacher layer
→ Reduces gradient mismatch between FP and quantized
```

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **ZeroQuant** | 2022 | NeurIPS | [2206.01861](https://arxiv.org/abs/2206.01861) | Original INT8 W+A PTQ, 5.19× speedup |
| **ZeroQuant-V2** | 2023 | — | [2303.08302](https://arxiv.org/abs/2303.08302) | Low-Rank Compensation (LoRC), W4A8 |
| **ZeroQuant-FP** | 2023 | — | [2307.09782](https://arxiv.org/abs/2307.09782) | FP8/FP4 floating-point formats |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [microsoft/DeepSpeed](https://github.com/microsoft/DeepSpeed) | Official — ZeroQuant integrated in DeepSpeed Compression |
| [kunalkumar168/ZeroQuant](https://github.com/kunalkumar168/ZeroQuant) | Third-party standalone reimplementation of original ZeroQuant |
| [microsoft/DeepSpeedCompression](https://github.com/microsoft/DeepSpeed) | Same as DeepSpeed — compression tutorial at [deepspeed.ai/tutorials/model-compression/](https://www.deepspeed.ai/tutorials/model-compression/) |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **"ZeroQuant: Efficient and Affordable Post-Training Quantization for Large-Scale Transformers"** | Microsoft Research | [Link](https://www.microsoft.com/en-us/research/publication/zeroquant-efficient-and-affordable-post-training-quantization-for-large-scale-transformers/) |
| **"DeepSpeed Compression: A Composable Library for Extreme Compression and Zero-Cost Quantization"** | Microsoft Research Blog | [Link](https://www.microsoft.com/en-us/research/blog/deepspeed-compression-a-composable-library-for-extreme-compression-and-zero-cost-quantization/) |
| **"ZeroQuant Explained"** | Medium (Raymond Chang) | [Link](https://medium.com/@raymondchang_24290/microsoft-and-their-new-zeroquant-14ddb121e04b) |

---

## ZeroQuant vs SmoothQuant

| Aspect | **ZeroQuant** | **SmoothQuant** |
|--------|-------------|---------------|
| **Approach** | Per-token activation quantization + LoRC correction | Migration factor to move outlier difficulty to weights |
| **Weight quantization** | W4, W8 | W8, W4 (via SmoothQuant+) |
| **Activation quantization** | INT8, FP8 | INT8 |
| **Training required** | None (PTQ) | None (PTQ) |
| **Error correction** | Low-Rank Compensation (LoRC) | None — migration is exact |
| **Speedup** | 5.19× (BERT), 4.16× (GPT-NeoX-20B) | 1.56× (MT-NLG 530B) |
| **Integration** | DeepSpeed Compression | vLLM, LMDeploy, AMD Quark, Intel Neural Compressor |
| **Key advantage** | LoRC handles quantization error analytically | No weight loading overhead (migration absorbed) |

> *"ZeroQuant uses LoRC to *correct* quantization error after the fact. SmoothQuant *prevents* the error from occurring by mathematical reorganization. Both are complementary."*

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Origin** | Microsoft Research, NeurIPS 2022 |
| **Problem solved** | INT8 weight + activation quantization without retraining |
| **Key innovations** | Per-token activation quantization + hardware kernels + LKD for INT4 |
| **ZeroQuant-V2** | Added Low-Rank Compensation (LoRC) for W4A8 on large models |
| **ZeroQuant-FP** | Explored FP8/FP4 floating-point formats for activation quantization |
| **Best for** | Large-scale transformers (>10B) via DeepSpeed; INT4 weight + INT8 activation via LoRC |
| **Limitation** | Original standalone inference engine never released — only via DeepSpeed |
| **Training required** | None — fully post-training |

---

*Last updated: 2026-04-19*
