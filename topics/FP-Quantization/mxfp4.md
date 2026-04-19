# MXFP4 (Microscaling FP4) Quantization

> **Standard:** [OCP MX Specification v1.0](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf)  
> **Key Papers:** [Microsoft FP4 Training (ICML 2025)](https://arxiv.org/abs/2501.17116), [Amazon MXFP4](https://github.com/amazon-science/mxfp4-llm), [Unveiling MXFP4](https://arxiv.org/abs/2603.08713)

---

## Overview

> *"MXFP4 (Microscaling FP4) is an OCP-standardized floating-point format designed for AI accelerators, providing a vendor-neutral alternative to NVIDIA's NVFP4."*

**MXFP4** is the **vendor-neutral standard** from the Open Compute Project (OCP), designed to run consistently across AMD, Intel, NVIDIA, and other AI accelerators. It differs from NVIDIA's NVFP4 in key ways — primarily the use of **power-of-two (E8M0) scaling** vs NVIDIA's fractional (E4M3) scaling.

---

## Format Specification

### FP4 (E2M1) Element Format

Same as NVFP4:
```
FP4 value: 1 sign bit + 2 exponent bits + 1 mantissa bit = 4 bits total
```

**Representable values:** ±{0, 0.5, 1, 1.5, 2, 3, 4, 6}

### Key Difference: E8M0 Scale (Power-of-Two)

| Format | Scale Type | Block Size | Flexibility |
|--------|------------|------------|-------------|
| **MXFP4** | E8M0 (power-of-two only) | 32 elements | Less flexible |
| **NVFP4** | E4M3 (fractional FP8) | 16 elements | More flexible |

**E8M0 formula:**
```
scale = 2^(shared_exponent - 127)
```

> *"MXFP4's PoT (power-of-two) block scaling has limited ability to reconstruct large values within blocks — quantization error increases sharply with magnitude."* — Block Rotation paper

### Storage Overhead

- **Block size:** 32 elements per scale
- **Scale storage:** 8 bits (E8M0) per 32 elements
- **Effective bits per element:** ~4.25 bits (vs NVFP4's ~4.5)

---

## MXFP4 vs NVFP4 vs INT4

| Aspect | MXFP4 | NVFP4 | INT4 (AWQ/GPTQ) |
|--------|-------|-------|-----------------|
| **Block size** | 32 | 16 | Variable |
| **Scale format** | E8M0 (PoT only) | E4M3 (fractional) | Per-channel |
| **Accuracy (base)** | Lower | Higher | Highest with rotation |
| **Hardware** | AMD MI355X, NVIDIA B200 | NVIDIA B200 | Universal |
| **Native compute** | Yes | Yes | No (dequantize) |
| **Scale precision MSE** | 0.72 | **0.08** | N/A |
| **Vendor support** | OCP standard | NVIDIA only | All |

**Critical finding:**
> *"Rotation-based methods—nearly universal in SOTA INT4 quantization—cause catastrophic performance collapse when applied to MXFP4."* — Block Rotation paper

---

## Research Papers

### 1. OCP MX Specification v1.0

| | |
|---|---|
| **Standard** | [OCP MX v1.0](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf) |
| **Define** | MXFP4 (E2M1 elements, 32-element blocks, E8M0 scale) |

### 2. Microsoft FP4 Training

| | |
|---|---|
| **Venue** | ICML 2025 |
| **arXiv** | [2501.17116](https://arxiv.org/abs/2501.17116) |
| **Key** | Differentiable Gradient Estimator + Outlier Clamping |

**First FP4 training framework** — trains LLMs in FP4 with near-BF16 accuracy.

**Training accuracy:**
| Model | BF16 | FP4 | Gap |
|-------|------|-----|-----|
| 1.3B | 2.49 | 2.55 | +0.06 |
| 7B | 2.07 | 2.17 | +0.10 |
| 13B | 1.88 | 1.97 | +0.09 |

### 3. Amazon MXFP4 LLM Training

| | |
|---|---|
| **Repo** | [amazon-science/mxfp4-llm](https://github.com/amazon-science/mxfp4-llm) |
| **Key** | Stochastic rounding + random Hadamard transform |

> *"Near-lossless training via stochastic rounding + random Hadamard transform."*

### 4. Unveiling MXFP4 Potential

| | |
|---|---|
| **arXiv** | [2603.08713](https://arxiv.org/abs/2603.08713) |
| **Key** | OAS + MBS techniques to close gap vs NVFP4 |

**Results (Llama 3.1-8B-Instruct):**
| Precision | MMLU-Pro | GSM8K | Average |
|-----------|----------|-------|---------|
| BF16 | 44.22 | 83.18 | 70.53 |
| MXFP4-OCP | 32.50 | 65.37 | 61.25 |
| MXFP4-MBS-H | 37.35 | 78.52 | 66.50 |
| NVFP4 | 38.83 | 77.12 | 67.02 |

### 5. Block Rotation (ICLR 2026)

| | |
|---|---|
| **arXiv** | [2511.04214](https://arxiv.org/abs/2511.04214) |
| **Key** | Block-wise rotation (BRQ) — global rotation fails on MXFP4 |

### 6. MX+ (MICRO'25)

| | |
|---|---|
| **DOI** | [10.1145/3725843.3756118](https://dl.acm.org/doi/10.1145/3725843.3756118) |
| **Key** | Repurposes block-max exponent as extra mantissa |

---

## Framework Support

### AMD Quark

```bash
python llm_ptq.py \
    --model meta-llama/Llama-3.3-70B-Instruct \
    --weight_dtype mxfp4 \
    --activation_dtype mxfp4 \
    --algorithm autosmoothquant \
    --output_dir ./output/Llama-3.3-70B-Instruct-mxfp4
```

### vLLM (via llm-compressor)

```python
from llmcompressor.modifiers.quantization import QuantizationModifier

recipe = QuantizationModifier(
    targets="Linear",
    scheme="MXFP4",
    ignore=["lm_head"]
)
oneshot(model=model, dataset=calib_data, recipe=recipe)
```

### Intel Neural Compressor

Supports MXFP4/MXFP8/INT4 quantization across Intel hardware.

### OpenAI GPT-OSS

> *"The models were post-trained with MXFP4 quantization of the MoE weights, making gpt-oss-120b run on a single 80GB GPU."*

---

## Practical Usage

### AMD MI355X Setup

```python
# Using AMD Quark
from quark import QuarkQuantizer

quantizer = QuarkQuantizer()
quantized_model = quantizer.quantize(
    model,
    weight_dtype="mxfp4",
    activation_dtype="mxfp4",
    algorithm="autosmoothquant"
)
```

### vLLM Inference

```python
from vllm import LLM

llm = LLM(model="./output/Llama-3.3-70B-Instruct-mxfp4")
```

### HuggingFace Model Hub

Pre-quantized models available from AMD and partners.

---

## Benchmark Comparisons

### Accuracy: MXFP4 vs NVFP4 vs INT4

| Precision | MMLU-Pro | GSM8K | Average |
|-----------|----------|-------|---------|
| BF16 | 44.22 | 83.18 | 70.53 |
| INT4 (AWQ) | ~40 | ~78 | ~66 |
| **NVFP4** | **38.83** | **77.12** | **67.02** |
| MXFP4-OCP | 32.50 | 65.37 | 61.25 |
| MXFP4-MBS-H | 37.35 | 78.52 | 66.50 |

### Training Results (Microsoft FP4)

| Model | BF16 Loss | FP4 Loss | Gap |
|-------|-----------|----------|-----|
| 1.3B | 2.49 | 2.55 | +0.06 |
| 7B | 2.07 | 2.17 | +0.10 |
| 13B | 1.88 | 1.97 | +0.09 |

### Hardware Comparison

| GPU | MXFP4 Support | NVFP4 Support |
|-----|--------------|---------------|
| AMD MI355X | ✅ Native | ❌ |
| NVIDIA B200 | ✅ | ✅ Native |
| NVIDIA RTX 5090 | ✅ | ✅ Native |
| NVIDIA H100 | ❌ | ❌ |

---

## Why MXFP4 Matters

### Vendor Neutrality

MXFP4 is the **OCP standard** — works across AMD, Intel, NVIDIA, and custom AI accelerators. NVFP4 is NVIDIA-only.

### When to Use MXFP4

| Scenario | Recommendation |
|----------|---------------|
| AMD MI300X/MI355X | ✅ Use MXFP4 |
| Multi-vendor deployment | ✅ Use MXFP4 |
| NVIDIA Blackwell | Consider NVFP4 (better accuracy) |
| Production INT4 | AWQ/GPTQ still best quality |

### The Rotation Problem

> *"Rotation-based methods—nearly universal in SOTA INT4 quantization—cause catastrophic performance collapse when applied to MXFP4."*

This means:
- AWQ's activation-aware weighting **cannot** be directly used with MXFP4
- GPTQ-style remapping **may not** work well
- Alternative methods (autosmoothquant, block rotation) needed

---

## References

- [OCP MX Specification v1.0](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf)
- [Microsoft FP4 Training (arXiv:2501.17116)](https://arxiv.org/abs/2501.17116)
- [Amazon MXFP4 LLM](https://github.com/amazon-science/mxfp4-llm)
- [Unveiling MXFP4 Potential (arXiv:2603.08713)](https://arxiv.org/abs/2603.08713)
- [Block Rotation (arXiv:2511.04214)](https://arxiv.org/abs/2511.04214)
- [MX+ (MICRO'25)](https://dl.acm.org/doi/10.1145/3725843.3756118)
- [AMD Quark Documentation](https://quark.docs.amd.com/)

---

*Last updated: 2026-04-19*
