# Cross-Modal Benchmarking and Sensitivity Analysis for Multimodal Quantization

> **Key Papers:** PTQ-Bench ([arXiv:2502.13178](https://arxiv.org/abs/2502.13178)) · MBQ ([arXiv:2412.19509](https://arxiv.org/abs/2412.19509)) · VLMQ ([arXiv:2508.03351](https://arxiv.org/abs/2508.03351)) · LUQ ([arXiv:2509.23729](https://arxiv.org/abs/2509.23729))  
> **GitHub:** [zjq0455/PTQ-Bench](https://github.com/zjq0455/PTQ-Bench) · [thu-nics/MBQ](https://github.com/thu-nics/MBQ) · [gautomdas/mmq](https://github.com/gautomdas/mmq)  
> **Survey:** [HKUST-LongGroup/Awesome-MLLM-Benchmarks](https://github.com/HKUST-LongGroup/Awesome-MLLM-Benchmarks)

---

## Overview

> *"Even the largest models at 2-bit perform worse than the smallest models at 4-bit. Extremely low-bit PTQ for large models needs reexamination."* — PTQ-Bench (arXiv:2502.13178)

Evaluating multimodal LLM quantization requires **cross-modal benchmarks** that test both vision and language capabilities, and **sensitivity analysis** to identify which components/tokens need higher precision.

---

## PTQ-Bench: Unified PTQ Evaluation

> **arXiv:** [2502.13178](https://arxiv.org/abs/2502.13178) · **GitHub:** [zjq0455/PTQ-Bench](https://github.com/zjq0455/PTQ-Bench)

**Key contribution:** Comprehensive taxonomy and unified evaluation of PTQ methods across LLaVA and VILA models.

> *"PTQ-Bench covers LLaMA1/2/3, Mixtral, DeepSeekMoE, Mamba — evaluating GPTQ, AWQ, OmniQuant, QuIP across multiple precision levels."*

### Supported Methods

| Method | Type | Notes |
|--------|------|-------|
| **GPTQ** | Weight-only | Per-channel quantization |
| **AWQ** | Weight-only | Activation-aware |
| **OmniQuant** | Weight + activation | Learnable smoothing |
| **QuIP** | Weight-only | Non-uniform quantization |

---

## Sensitivity Analysis

### Gradient Magnitude Analysis

| Paper | Finding |
|-------|---------|
| **MBQ** (CVPR 2025) | Text tokens have **22.4× higher average gradient magnitude** than vision tokens |
| **VEQ** (2026) | Text-to-vision gradient ratio spikes to **15×** in extreme cases (MoE VLMs) |
| **VLMQ** (2025) | Visual over-representation + modality gap; gradient-driven importance factor G |

### Layer-Wise Sensitivity

> *"Multimodal tokens exhibit higher statistical variance and entropy than text tokens across all layers."* — LUQ (arXiv:2509.23729)

| Layer Type | Sensitivity | Notes |
|-----------|------------|-------|
| **Early layers** | High | Processing raw visual features |
| **Middle layers** | Variable | Depends on model depth |
| **End layers** | High | Output generation |
| **Low-entropy layers** | Low | Simpler, more compressible functions |

### LUQ: Entropy-Based Layer Selection

> **arXiv:** [2509.23729](https://arxiv.org/abs/2509.23729) (TMLR under review)

| Finding | Detail |
|---------|--------|
| **Method** | Binary search over layer selection for ultra-low bits |
| **Result** | Pareto-optimal precision allocation |
| **Critical** | Standard 2-bit GPTQ/AWQ → **complete model collapse (0%)** |

---

## Evaluation Datasets

### MME: Multimodal Evaluation Benchmark

> **arXiv:** [2306.13394](https://arxiv.org/abs/2306.13394) · **NeurIPS DB 2025 Spotlight**

| Feature | Detail |
|---------|--------|
| **Subtasks** | 14 comprehensive tasks |
| **Coverage** | Perception + cognition |
| **Purpose** | Overall MLLM capability assessment |

### MMMU: Massive Multi-discipline Multimodal Understanding

> **GitHub:** [MMMU-Benchmark/MMMU](https://github.com/MMMU-Benchmark/MMMU)

| Feature | Detail |
|---------|--------|
| **Questions** | 11.5K across 30 subjects |
| **Disciplines** | 6 major areas |
| **Level** | College-level reasoning |
| **Pro version** | MMMU-Pro — filters text-only questions |

### MMVP: Multimodal Visual Patterns

> **HuggingFace:** [MMVP/MMVP](https://huggingface.co/datasets/MMVP/MMVP) · **GitHub:** [tsb0601/MMVP](https://github.com/tsb0601/MMVP)

| Feature | Detail |
|---------|--------|
| **Purpose** | Identify CLIP-blind pairs |
| **Patterns** | 9 visual patterns |
| **Focus** | Tests visual grounding failures |

### SEED-Bench / SEED-Bench-2

| Feature | Detail |
|---------|--------|
| **Dimensions** | 27 hierarchical capability dimensions |
| **Purpose** | Comprehensive capability evaluation |

### Additional Benchmarks

| Benchmark | Focus | URL |
|----------|-------|-----|
| **MMBench** | Multi-modal evaluation | — |
| **MathVista** | Math reasoning in visual contexts | — |
| **TouchStone** | VLM evaluation via LMs | — |
| **LLMC+** | AAAI 2025 — VLM compression (6 quantization algos) | — |

---

## MBQ Sensitivity Study Results

### MBQ: Gradient-Based Modality Sensitivity

| Token Type | Gradient Magnitude | Relative Sensitivity |
|-----------|-------------------|---------------------|
| **Language tokens** | 10–22× baseline | **HIGH** |
| **Vision tokens** | 1× baseline | **LOW** |

### VLMQ: Token-Level Redundancy

| Finding | Detail |
|---------|--------|
| **Visual over-representation** | 50% of vision tokens are redundant |
| **Solution** | Down-weighting 50% of vision tokens |
| **Result** | DocVQA +0.39% improvement |

---

## Mixed-Precision Benchmark: MMQ

> **GitHub:** [gautomdas/mmq](https://github.com/gautomdas/mmq)

**Pareto-Optimal Quantization Strategies for Multimodal LLMs.**

### Component Importance (from MMQ paper)

| Model | Task | Method | ViT Importance | Q-Former | LLM Importance |
|-------|------|--------|---------------|----------|----------------|
| **BLIP-2** | COCO Caption | GPTQ | 50.4% | 2.0% | 47.6% |
| **BLIP-2** | COCO Caption | AWQ | 17.1% | 2.5% | **80.5%** |
| **LLaVA** | GQA | GPTQ | 20.87% | — | 79.13% |
| **LLaVA** | GQA | AWQ | 5.38% | — | **94.62%** |

---

## Benchmarking Frameworks

### PTQ-Bench Architecture

```
Input: Quantized MLLM
    ↓
Evaluation Pipeline
    ├── Vision benchmarks (MME, MMVP, MMMU)
    ├── Language benchmarks (VQAv2, GQA)
    └── Overall score
    ↓
Output: Per-method, per-precision results
```

### vLLM LLM Compressor

> **Docs:** [vllm.ai/projects/llm-compressor/](https://docs.vllm.ai/projects/llm-compressor/en/0.8.1/examples/multimodal_vision/)

```python
from llm_compressor import parse_recursive
from modifiers import GPTQModifier, SmoothQuantModifier

# Multimodal sequential targeting
parse_recursive(
    model,
    sequential_targets=["vision", "projector", "llm"],
    modifiers=[GPTQModifier(precision="int4")]
)
```

---

## Key Findings from Benchmarking

### 1. Modality Disparity is Quantifiable

> *"Text tokens are significantly more quantization-sensitive than vision tokens (gradient ratio 15-22× higher), yet vision tokens dominate in quantity (200+ vision vs ~5 text tokens)."* — VEQ (arXiv:2602.01037)

### 2. Layer-Wise Variation Matters

Sensitivity varies significantly across layers:
- Some layers tolerate ultra-low bit quantization
- Binary search over layer selection creates Pareto-optimal precision allocation

### 3. MoE Expert Heterogeneity

In MoE VLMs:
- Experts show non-uniform token distribution
- Modality-specific expert specializations exist
- Routing bias means preserving decisive expert precision is critical

### 4. Performance Ceiling

| Observation | Implication |
|------------|-------------|
| 2-bit large models < 4-bit small models | Extremely low-bit needs rethinking |
| PTQ-Bench finding | Scale doesn't solve quantization quality |

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **PTQ-Bench** | 2025 | — | [2502.13178](https://arxiv.org/abs/2502.13178) | Unified PTQ evaluation framework |
| **MBQ** | 2025 | CVPR | [2412.19509](https://arxiv.org/abs/2412.19509) | Gradient-based modality balancing |
| **VLMQ** | 2025 | — | [2508.03351](https://arxiv.org/abs/2508.03351) | Token saliency-driven PTQ |
| **LUQ** | 2025 | TMLR | [2509.23729](https://arxiv.org/abs/2509.23729) | Layer-wise entropy for ultra-low bit |
| **VEQ** | 2026 | — | [2602.01037](https://arxiv.org/abs/2602.01037) | MoE VLMs: dual-aware modality+expert heterogeneity |
| **QSLAW** | 2024 | ACM MM | [2408.03735](https://arxiv.org/abs/2408.03735) | Multimodal warmup + learnable scales |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [zjq0455/PTQ-Bench](https://github.com/zjq0455/PTQ-Bench) | Unified PTQ evaluation for LLMs + Multimodal |
| [gautomdas/mmq](https://github.com/gautomdas/mmq) | Pareto-optimal mixed-precision study |
| [thu-nics/MBQ](https://github.com/thu-nics/MBQ) | CVPR 2025: gradient-based sensitivity |
| [HKUST-LongGroup/Awesome-MLLM-Benchmarks](https://github.com/HKUST-LongGroup/Awesome-MLLM-Benchmarks) | 50+ benchmarks cataloged |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Key datasets** | MME, MMMU, MMVP, SEED-Bench |
| **Gradient disparity** | Text 15–22× more sensitive than vision |
| **Layer variation** | Entropy-guided selection enables ultra-low bit |
| **MoE heterogeneity** | Expert specialization requires dual-aware approach |
| **Performance ceiling** | 2-bit large < 4-bit small — rethink ultra-low strategies |
| **Framework** | PTQ-Bench unifies evaluation across methods |

---

*Last updated: 2026-04-19*
