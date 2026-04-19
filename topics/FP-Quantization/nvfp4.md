# NVIDIA NVFP4 Quantization Format

> **Key Paper:** [FP4 All the Way (NeurIPS 2025)](https://arxiv.org/abs/2505.19115)  
> **Developer Blog:** [Introducing NVFP4](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)  
> **Implementation:** [llm-compressor](https://github.com/vllm-project/llm-compressor), [TensorRT Model Optimizer](https://github.com/NVIDIA/TensorRT-Model-Optimizer)

---

## Overview

> *"By combining fine-grained FP8 scaling per 16-value block with a global FP32 scale for the entire tensor, NVFP4 reduces rounding errors, preserves important details, and allows the models to run in 4-bit precision with far less accuracy loss than earlier FP4 formats."*

**NVFP4** (NVIDIA FP4) is a hardware-accelerated 4-bit floating-point quantization format introduced with the **NVIDIA Blackwell GPU architecture** (B200, B300, GB200, GB300, DGX Spark, RTX 5090).

### Key Advantages over INT4

| Advantage | Description |
|-----------|-------------|
| **Native FP4 compute** | Direct tensor core operations — no dequantization needed |
| **2.35× higher throughput** | vs INT4 on Blackwell GPUs |
| **Better outlier handling** | Per-block scaling adapts to local value distributions |
| **Hardware accelerated** | Native Blackwell GPU support |

---

## Format Specification

### FP4 (E2M1) Element Format

```
FP4 value: 1 sign bit + 2 exponent bits + 1 mantissa bit = 4 bits total
```

**Representable values:** ±{0, 0.5, 1, 1.5, 2, 3, 4, 6}

| Value | Sign | Exponent | Mantissa |
|-------|------|----------|----------|
| 0 | 0 | 00 | 0 |
| 0.5 | 0 | 01 | 0 |
| 1 | 0 | 10 | 0 |
| 1.5 | 0 | 10 | 1 |
| 2 | 0 | 11 | 0 |
| 3 | 0 | 11 | 1 |
| 4 | 1 | 00 | 0 |
| 6 | 1 | 00 | 1 |

**Exponent bias:** 1 (non-standard)

> *"This encoding does not support Inf/NaN."* — [CUDA Math API](https://docs.nvidia.com/cuda/cuda-math-api/cuda_math_api/struct____nv__fp4__e2m1.html)

### No Denormal/Subnormal Support

NVFP4 uses **saturation mode** (`__NV_SATFINITE`) — out-of-range denormals are flushed to nearest representable value, not gradually underflowed.

### Block-wise Scaling Architecture

NVFP4 uses **two-level hierarchical scaling**:

**Level 1 — Per-Block FP8 E4M3 Scale (Micro-scaling):**
- Block size: **16 elements** (vs MXFP4's 32)
- Each block shares one FP8 E4M3 scale factor
- E4M3: 4 exponent bits + 3 mantissa bits (bias=7)
- Range: ±448 (supports fractional/non-power-of-two scaling)

**Level 2 — Per-Tensor FP32 Scale (Global normalization):**
- Single FP32 scale factor for entire tensor
- Enables wide dynamic ranges across blocks

**Quantization formula:**
```
x̄ᵢ = QuantFP4(α · Δⱼ · xᵢ)
```
Where α = FP32 global scale, Δⱼ = FP8 E4M3 block scale

**Dequantization:**
```
x̂ᵢ = x̄ᵢ · α · Δⱼ
```

---

## NVFP4 vs MXFP4 vs INT4

| Property | FP4 (E2M1) | MXFP4 | NVFP4 | INT4 |
|----------|------------|-------|-------|------|
| **Block Size** | N/A | 32 | **16** | Variable |
| **Scale Format** | N/A | E8M0 (PoT) | **E4M3** (fractional) | Per-channel |
| **Bits/element** | 4 | 4.25 | 4.5 | 4 |
| **Hardware Support** | No | AMD MI355X, NVIDIA B200 | **NVIDIA Blackwell** | All GPUs |
| **Native Compute** | No | Yes | **Yes** | No (dequantize) |
| **Outlier Handling** | Poor | Moderate | **Good** | Good |
| **E8M0 MSE** | — | 0.72 | **0.08** | — |

**Key innovation of NVFP4 over MXFP4:**
> *"E4M3 adjusts its scaling factor to each small block of values, allowing for finer fit across wider ranges of inputs. That additional flexibility is what translates to a lower overall rounding error."*

---

## Research Papers

### 1. FP4 All the Way: Fully Quantized Training of LLMs

| | |
|---|---|
| **Venue** | NeurIPS 2025 Spotlight |
| **arXiv** | [2505.19115](https://arxiv.org/abs/2505.19115) |
| **OpenReview** | [kuzye4EPLR](https://openreview.net/forum?id=kuzye4EPLR) |

**First fully quantized training (FQT)** of LLMs using predominantly FP4 precision for weights, activations, AND gradients.

> *"Our analysis shows that the NVFP4 format, where each block of 16 FP4 values (E2M1) shares a scale represented in E4M3, provides optimal results."*

**Key findings:**
- Block sizes below 16 offer diminishing returns
- Stochastic rounding (SR) required on neural gradients
- Critical threshold: When gradient std falls below √3 × σ_q, FP4 becomes ineffective

**Results:** FP4 + QAF matches/exceeds BF16 on zero-shot tasks (45.75 avg acc vs 45.63 BF16)

### 2. Four Over Six (4/6): Adaptive Block Scaling

| | |
|---|---|
| **arXiv** | [2512.02010](https://arxiv.org/html/2512.02010v3) |
| **GitHub** | [mit-han-lab/fouroversix](https://github.com/mit-han-lab/fouroversix) |

Addresses NVFP4 error for near-maximal values by choosing scale M=4 or M=6 per block.

> *"When the values [10, 20, 30, 40] are quantized to NVFP4, 30 is scaled to 30/6.5=4.62, and then rounded down to 4... If this block is instead scaled such that the largest value is 4, 30 would instead be scaled to 30/10=3, which can be represented in FP4 with no error."*

**Results:**
| Method | Effect |
|--------|--------|
| AWQ + 4/6 | **19.9%** closer to BF16 perplexity |
| SmoothQuant + 4/6 | **5.3%** closer |
| GPTQ + 4/6 | -34.6% (incompatible) |

### 3. RaZeR: Redundant Zero Remapping

| | |
|---|---|
| **arXiv** | [2501.04052](https://arxiv.org/abs/2501.04052) |

Exploits redundancies in NVFP4 to add extra quantization values (+5 and -5) at no memory cost.

**Results:**
| Method | W4-A16 Perplexity | W4-A4 Perplexity |
|--------|-------------------|------------------|
| NVFP4 | 9.61 | 9.83 |
| FourOverSix | 9.59 | 9.78 |
| **RaZeR** | **9.52** | **9.68** |

- 34.6% perplexity loss reduction vs NVFP4

### 4. Bridging the Gap for Microscaling FP4

| | |
|---|---|
| **arXiv** | [2509.23202](https://arxiv.org/html/2509.23202v2) |

Introduces **MR-GPTQ** algorithm for NVFP4.

> *"NVFP4's small group size (G=16) neutralizes Hadamard rotations — absmax scaling already preserves top element perfectly."*

**Performance:**
| GPU | Layer-wise Speedup | End-to-end Speedup |
|-----|-------------------|-------------------|
| B200 | ~3.6× | ~2.2× |
| RTX 5090 | ~6× | ~4× |

---

## vLLM Implementation

### Installation

```bash
git clone https://github.com/vllm-project/llm-compressor.git
cd llm-compressor
pip install -e .
```

### Quantization with LLM Compressor

```python
from llmcompressor import oneshot
from llmcompressor.modifiers.quantization import QuantizationModifier
from transformers import AutoTokenizer, AutoModelForCausalLM

# 1) Load model
MODEL_ID = "meta-llama/Meta-Llama-3-8B-Instruct"
model = AutoModelForCausalLM.from_pretrained(MODEL_ID, dtype="auto")
tokenizer = AutoTokenizer.from_pretrained(MODEL_ID)

# 2) Prepare calibration data
from datasets import load_dataset
NUM_CALIBRATION_SAMPLES = 512
MAX_SEQUENCE_LENGTH = 2048
ds = load_dataset("HuggingFaceH4/ultrachat_200k", split=f"train_sft[:{NUM_CALIBRATION_SAMPLES}]")
ds = ds.shuffle(seed=42)
ds = ds.map(lambda x: {"text": tokenizer.apply_chat_template(x["messages"], tokenize=False)})
ds = ds.map(lambda x: tokenizer(x["text"], padding=False, max_length=MAX_SEQUENCE_LENGTH, truncation=True, add_special_tokens=False), remove_columns=ds.column_names)

# 3) Apply NVFP4 quantization
recipe = QuantizationModifier(targets="Linear", scheme="NVFP4", ignore=["lm_head"])
oneshot(
    model=model,
    dataset=ds,
    recipe=recipe,
    max_seq_length=MAX_SEQUENCE_LENGTH,
    num_calibration_samples=NUM_CALIBRATION_SAMPLES,
)

# 4) Save compressed model
SAVE_DIR = MODEL_ID.rstrip("/").split("/")[-1] + "-NVFP4"
model.save_pretrained(SAVE_DIR, save_compressed=True)
tokenizer.save_pretrained(SAVE_DIR)
```

### Weight-Only (NVFP4A16)

```python
# No calibration data needed
recipe = QuantizationModifier(targets="Linear", scheme="NVFP4A16", ignore=["lm_head"])
oneshot(model=model, recipe=recipe)
```

> ⚠️ **Warning:** NVFP4A16 dequantizes weights at runtime — most performance benefits are lost. Use full NVFP4 (W4A4).

### vLLM Inference

```python
from vllm import LLM

# Requires Blackwell GPU
llm = LLM(model="meta-llama/Meta-Llama-3-8B-Instruct-NVFP4", tensor_parallel_size=1)
```

> ⚠️ On non-Blackwell hardware, activation quantization is disabled automatically.

---

## Performance Benchmarks

### DeepSeek-R1-0528 (FP8 → NVFP4 via PTQ)

| Benchmark | FP8 | NVFP4 | Delta |
|-----------|-----|-------|-------|
| MMLU-PRO | 85% | 84% | -1% |
| GPQA Diamond | 81% | 80% | -1% |
| HLE | 15% | 14% | -1% |
| LIVECODEBENCH | 77% | 76% | -1% |
| SCICODE | 40% | 40% | 0% |
| Math-500 | 98% | 98% | 0% |
| **AIME 2024** | 89% | **91%** | **+2%** |

> *"≤1% accuracy degradation on most tasks; AIME 2024 shows NVFP4 2% better than FP8."*

### Llama-3.1-8B-Instruct Accuracy (Emulated W4A4)

| Format | Method | MMLU-CoT | GSM8k | Avg Recovery |
|--------|--------|----------|-------|--------------|
| Baseline | FP16 | 72.76 | 85.06 | 100% |
| INT4 | RTN+HT | 68.30 | 79.61 | 94.71% |
| **NVFP4** | **GPTQ** | **68.85** | **82.60** | **95.92%** |
| MXFP4 | GPTQ | 63.49 | 68.46 | 89.47% |

> *"NVFP4 provides the best accuracy, with INT4 second, and MXFP4 third."*

### Throughput Comparison (RTX 5090)

| Format | Throughput | vs FP16 |
|--------|------------|---------|
| FP16 (baseline) | 3,192 tok/s | 1× |
| INT4 (AWQ/GPTQ) | ~3,200 tok/s | ~1× |
| **NVFP4** | **4,495 tok/s** | **2.35×** |

> *"NVFP4 models are 2.35× faster than INT4 models, thanks to Blackwell GPU acceleration."*

### Memory Savings

| Comparison | Reduction |
|------------|-----------|
| vs FP16 | **~3.5×** |
| vs FP8 | **~1.8×** |

A 70B model in FP16 requires ~140GB; NVFP4 reduces to ~40GB.

---

## Practical Usage

### Prerequisites

- **Hardware:** NVIDIA Blackwell GPU (B200, B300, GB200, GB300, DGX Spark, RTX 5090)
- **Software:** vLLM ≥0.8.0, LLM Compressor, or TensorRT Model Optimizer

### TensorRT Model Optimizer (Production)

```python
from model_optimizer import quantize

quantize(
    model="meta-llama/Llama-3-70B-Instruct",
    scheme="nvfp4",
    calibration_data=calib_data,
    output_dir="./llama3-70b-nvfp4"
)
```

### Pre-Quantized Models on HuggingFace

| Model | Quantizer | Link |
|-------|-----------|------|
| nvidia/DeepSeek-R1-0528-NVFP4 | TensorRT Model Optimizer v0.31.0 | [HF](https://huggingface.co/nvidia/DeepSeek-R1-0528-NVFP4) |
| nvidia/DeepSeek-R1-NVFP4 | TensorRT Model Optimizer v0.23.0 | [HF](https://huggingface.co/nvidia/DeepSeek-R1-NVFP4) |
| Meta-Llama-3-8B-Instruct-NVFP4 | LLM Compressor | Research |

---

## Known Issues

1. **FlashInfer crashes with NVFP4** — Workaround: `pip uninstall flashinfer-python`
2. **Non-Blackwell GPUs** — Falls back to weight-only quantization (NVFP4A16)
3. **Standard pip vLLM** — Must compile from source for FP4 support on some configurations

---

## References

- [NVIDIA Developer Blog: Introducing NVFP4](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)
- [LLM Compressor NVFP4 Docs](https://docs.vllm.ai/projects/llm-compressor/en/latest/examples/quantization_w4a4_fp4/)
- [FP4 All the Way (arXiv:2505.19115)](https://arxiv.org/abs/2505.19115)
- [Four Over Six (arXiv:2512.02010)](https://arxiv.org/html/2512.02010v3)
- [RaZeR (arXiv:2501.04052)](https://arxiv.org/abs/2501.04052)
- [Bridging the Gap (arXiv:2509.23202)](https://arxiv.org/html/2509.23202v2)
- [CUDA Math API: __nv_fp4_e2m1](https://docs.nvidia.com/cuda/cuda-math-api/cuda_math_api/struct____nv__fp4__e2m1.html)
- [TensorRT Model Optimizer](https://github.com/NVIDIA/TensorRT-Model-Optimizer)

---

*Last updated: 2026-04-19*
