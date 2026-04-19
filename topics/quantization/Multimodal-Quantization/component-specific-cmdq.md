# Component-Specific Strategies (CMDQ) for Multimodal LLM Quantization

> **Key Papers:** MBQ ([arXiv:2412.19509](https://arxiv.org/abs/2412.19509), CVPR 2025) · MQuant ([arXiv:2502.00425](https://arxiv.org/abs/2502.00425), ACM MM 2025) · LUQ ([arXiv:2509.23729](https://arxiv.org/abs/2509.23729), TMLR) · Q-VLM ([arXiv:2410.08119](https://arxiv.org/abs/2410.08119), NeurIPS 2024) · Best Practices ([arXiv:2601.15287](https://arxiv.org/abs/2601.15287))  
> **GitHub:** [thu-nics/MBQ](https://github.com/thu-nics/MBQ) · [StiphyJay/MQuant](https://github.com/StiphyJay/MQuant) · [changyuanwang17/qvlm](https://github.com/changyuanwang17/qvlm)

---

## Overview

> *"LLM is most sensitive to quantization (80–98% importance for reasoning tasks), yet has far fewer parameters than ViT in some architectures. The ViT shows comparable importance to LLM despite having fewer parameters."* — "Towards Understanding Best Practices for Quantization of VLMs" (arXiv:2601.15287)

**CMDQ (Component-wise Mixed-precision for Multimodal)** is the strategy of allocating different precision levels to the three main components of a Vision-Language model: the **Vision Encoder**, the **Projector** (Q-Former, MLP, etc.), and the **LLM Backbone**.

---

## The Three Components

### Architecture Overview

```
Input Image → [Vision Encoder] → Image Features
                            ↓
                    [Projector] → Visual Tokens
                            ↓
Text Prompt → [LLM Backbone] → "This is an image of..."
                     ↑
            Visual + Text Tokens
```

### Component Sensitivity

| Component | Sensitivity | Precision Recommendation | Notes |
|-----------|------------|------------------------|-------|
| **Vision Encoder (ViT)** | Medium | W4A8 | Compute-bound; redundancy tolerates aggressive quantization |
| **Projector** | Low-Medium | W4–W6 | Varies by architecture; Q-Former is fragile |
| **LLM Backbone** | **Highest** | W4–W6 weights, W8 activations | Dominates overall performance |

---

## Component Sensitivity Evidence

### From "Best Practices for Quantization of VLMs" (arXiv:2601.15287)

| Model | Task | Method | ViT Importance | Q-Former | LLM Importance |
|-------|------|--------|---------------|----------|----------------|
| **BLIP-2** | COCO Caption | GPTQ | 50.4% | 2.0% | 47.6% |
| **BLIP-2** | COCO Caption | AWQ | 17.1% | 2.5% | **80.5%** |
| **LLaVA** | GQA | GPTQ | 20.87% | — | 79.13% |
| **LLaVA** | GQA | AWQ | 5.38% | — | **94.62%** |
| **LLaVA** | VQAv2 | GPTQ | 27.85% | — | 72.15% |
| **LLaVA** | VQAv2 | AWQ | 6.11% | — | **93.89%** |

### Key Findings

| Insight | Detail |
|---------|--------|
| **LLM dominates** | 72–98% of sensitivity for reasoning tasks (VQA) |
| **ViT matters for perception** | More important for captioning than VQA |
| **Q-Former is fragile** | Low importance but breaks under wrong quantization |
| **AWQ concentrates on LLM** | 80–98% of importance on LLM |
| **GPTQ distributes evenly** | Better for captioning/retrieval tasks |

---

## Key Methods

### 1. MBQ: Modality-Balanced Quantization (CVPR 2025)

> **arXiv:** [2412.19509](https://arxiv.org/abs/2412.19509) · **GitHub:** [thu-nics/MBQ](https://github.com/thu-nics/MBQ)

**Key contribution:** Gradient-based sensitivity weighting for vision vs language tokens. Uses gradients of SFT loss to balance quantization.

| Result | Metric |
|--------|--------|
| **W3** | +4.4% over SOTA |
| **W4A8** | +11.6% over SOTA |

---

### 2. MQuant: Full Static Quantization for MLLMs (ACM MM 2025)

> **arXiv:** [2502.00425](https://arxiv.org/abs/2502.00425) · **GitHub:** [StiphyJay/MQuant](https://github.com/StiphyJay/MQuant)

**Key contribution:** Modality-Specific Static Quantization (MSQ) + Attention-Invariant Flexible Switching (AIFS).

| Feature | Description |
|---------|-------------|
| **MSQ** | Different quantization scales per modality |
| **AIFS** | Switches between modalities without attention degradation |

**Supported models:** Qwen-VL, InternVL2, MiniCPM-V, Qwen2-VL

---

### 3. LUQ: Layerwise Ultra-Low Bit Quantization (TMLR)

> **arXiv:** [2509.23729](https://arxiv.org/abs/2509.23729)

**Key contribution:** Layer-wise entropy-based selection for ultra-low bit (<4-bit) quantization.

| Finding | Detail |
|---------|--------|
| **Multimodal tokens** | Higher entropy than text tokens across all layers |
| **Entropy varies by layer** | Some layers tolerate ultra-low bit quantization |
| **Binary search** | Creates Pareto-optimal precision allocation |
| **Critical finding** | Standard 2-bit GPTQ/AWQ → **complete model collapse (0%)** |

#### LUQ Results (LLaVA-1.5)

| Method | Avg Bits | MME Per | TextVQA | Memory Savings |
|--------|----------|---------|---------|----------------|
| **FP16** | 16 | 1510 | 58.2 | — |
| GPTQ 4-bit | 4 | 1450 | 58.2 | baseline |
| **LUQ 16-layer** | **2.54** | **1365** | **53.4** | **40% vs 4-bit** |

---

### 4. Q-VLM: Cross-Layer Dependency Quantization (NeurIPS 2024)

> **arXiv:** [2410.08119](https://arxiv.org/abs/2410.08119) · **GitHub:** [changyuanwang17/qvlm](https://github.com/ChangyuanWang17/QVLM)

**Key contribution:** Cross-layer dependency mining for block-wise quantization search.

> *"W4A4 capable — addresses cross-layer dependencies that naive per-layer quantization misses."*

---

### 5. Fine-Grained PTQ for LVLMs (2025)

> **arXiv:** [2603.17809](https://arxiv.org/abs/2603.17809)

**Key contribution:** Token-level sensitivity via Quantization-aware Integrated Gradients.

---

## Deployment Strategy

### Recommended Precision by Component

| Component | Precision | Method | Notes |
|-----------|-----------|--------|-------|
| **Vision Encoder** | W4A8 | AWQ or GPTQ | ViT redundancy tolerates aggression |
| **Projector (MLP)** | W4–W6 | GPTQ | Less sensitive than LLM |
| **Projector (Q-Former)** | W6–W8 | GPTQ | More fragile; avoid compensation-based |
| **LLM Backbone** | W4–W6 | AWQ | Highest sensitivity; most bits |
| **LLM Activations** | W8 | SmoothQuant | If needed |

### What NOT to Do

| Mistake | Consequence |
|---------|-------------|
| Quantize projector with GPTQ compensation | Complete projector failure |
| Use uniform precision across all components | Suboptimal quality |
| Apply SmoothQuant to Q-Former | Catastrophic degradation |

---

## Framework Support

### vLLM Multimodal

```python
# Sequential targeting of different components
parse_recursive(
    model,
    sequential_targets=["vision", "projector", "llm"],
    modifiers=[
        GPTQModifier(precision="int4"),  # applies per component
    ]
)
```

### LLM Compressor

> **Docs:** [vllm.ai/projects/llm-compressor/](https://docs.vllm.ai/projects/llm-compressor/en/0.8.1/examples/multimodal_vision/)

Supports `sequential_targets` for per-component quantization.

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **MBQ** | 2025 | CVPR | [2412.19509](https://arxiv.org/abs/2412.19509) | Gradient-based modality balancing |
| **MQuant** | 2025 | ACM MM | [2502.00425](https://arxiv.org/abs/2502.00425) | MSQ + AIFS for MLLMs |
| **LUQ** | 2025 | TMLR | [2509.23729](https://arxiv.org/abs/2509.23729) | Layer-wise entropy for ultra-low bit |
| **Q-VLM** | 2024 | NeurIPS | [2410.08119](https://arxiv.org/abs/2410.08119) | Cross-layer dependency mining |
| **Best Practices** | 2025 | — | [2601.15287](https://arxiv.org/abs/2601.15287) | Component sensitivity study |
| **Fine-Grained PTQ** | 2025 | — | [2603.17809](https://arxiv.org/abs/2603.17809) | Token-level sensitivity via Q-IG |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [thu-nics/MBQ](https://github.com/thu-nics/MBQ) | CVPR 2025: Supports InternVL2, LLaVA-OV, LLaVA, Qwen2-VL |
| [StiphyJay/MQuant](https://github.com/StiphyJay/MQuant) | ACM MM 2025: Qwen-VL, InternVL2, MiniCPM-V, Qwen2-VL |
| [changyuanwang17/qvlm](https://github.com/ChangyuanWang17/QVLM) | NeurIPS 2024: W4A4 capable |
| [gautomdas/mmq](https://github.com/gautomdas/mmq) | Pareto-optimal mixed-precision study |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Three components** | Vision Encoder, Projector, LLM Backbone |
| **Most sensitive** | LLM backbone (72–98% importance for VQA) |
| **ViT sensitivity** | Medium — lower than LLM for reasoning tasks |
| **Projector** | Q-Former fragile; avoid compensation-based methods |
| **AWQ vs GPTQ** | AWQ concentrates on LLM; GPTQ distributes evenly |
| **Ultra-low bit** | LUQ: entropy-based layer selection avoids collapse |
| **Key insight** | Component-specific >> uniform precision |

---

*Last updated: 2026-04-19*
