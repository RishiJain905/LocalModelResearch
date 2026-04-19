# FP8 Quantization: E4M3 and E5M2

> **Standards:** NVIDIA FP8 Spec / [OCP MX Specification](https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf)  
> **Key Papers:** [arXiv:2309.17355](https://arxiv.org/abs/2309.17355) (FP8 for LLMs), [arXiv:2309.12424](https://arxiv.org/abs/2309.12424) (FP8-LLM)

---

## Overview

FP8 (8-bit floating point) quantization provides a hardware-accelerated path to reduce LLM memory footprint and increase throughput with minimal accuracy loss. Two formats dominate: **E4M3** (4-bit exponent, 3-bit mantissa) and **E5M2** (5-bit exponent, 2-bit mantissa).

> *"FP8 reduces activation memory by 50% and improves throughput by 1.2× vs FP16 with <1% accuracy degradation."*

---

## FP8 Formats

### E4M3 (4-bit Exponent, 3-bit Mantissa)

| Property | Value |
|----------|-------|
| **Total bits** | 8 |
| **Exponent bits** | 4 |
| **Mantissa bits** | 3 |
| **Exponent bias** | 7 |
| **Range** | ±448 (max), ±0.015625 (min positive) |
| **Precision** | ~3-4 decimal digits |
| **Use case** | Weights and activations |

**Representable values:** ~89 distinct values including denormals

### E5M2 (5-bit Exponent, 2-bit Mantissa)

| Property | Value |
|----------|-------|
| **Total bits** | 8 |
| **Exponent bits** | 5 |
| **Mantissa bits** | 2 |
| **Exponent bias** | 15 |
| **Range** | ±57344 (max), ±0.00006 (min positive) |
| **Precision** | ~2 decimal digits |
| **Use case** | Gradients, activations (high dynamic range) |

**Supports:** Inf, NaN (unlike E4M3)

### E4M3 vs E5M2 Comparison

| Aspect | E4M3 | E5M2 |
|--------|------|------|
| **Dynamic range** | Narrower (±448) | Wider (±57344) |
| **Precision** | Higher (~3-4 digits) | Lower (~2 digits) |
| **Inf/NaN support** | ❌ No | ✅ Yes |
| **Typical use** | Weights, quantized models | Gradients, activations |
| **Hardware** | Blackwell, AMD MI300+ | Blackwell, AMD MI300+ |

---

## Research Papers

| Paper | Venue | arXiv | Key Contribution |
|-------|-------|-------|------------------|
| **FP8-LLM** | — | [2309.12424](https://arxiv.org/abs/2309.12424) | First comprehensive FP8 LLM study |
| **FP8 for LLMs** | — | [2309.17355](https://arxiv.org/abs/2309.17355) | E4M3 for weights, E5M2 for activations |
| **Training LLM with FP8** | — | — | NVIDIA research on FP8 training |

---

## Implementation in vLLM

### FP8 Quantization (Dynamic)

```python
from vllm import LLM

# Dynamic FP8 (easiest - no calibration needed)
llm = LLM(
    model="meta-llama/Llama-3-8B-Instruct",
    dtype="fp8"
)
```

### Weight-Only FP8 Quantization

```python
from vllm import LLM
from vllm.model_executor.quantization_utils import FP8Config

# Weight-only FP8 (calibration recommended)
llm = LLM(
    model="meta-llama/Llama-3-8B-Instruct",
    quantization="fp8",
    quantization_config=FP8Config(
        method="weight_only",
        calibration_data=calib_dataset
    )
)
```

### HuggingFace Transformers

```python
from transformers import AutoModelForCausalLM, AutoConfig

# Load with FP8 dtype
config = AutoConfig.from_pretrained("meta-llama/Llama-3-8B")
config.quantization_config = {"dtype": "fp8"}

model = AutoModelForCausalLM.from_pretrained(
    "meta-llama/Llama-3-8B-Instruct",
    torch_dtype="fp8",
    device_map="auto"
)
```

---

## Performance Benchmarks

### Memory & Throughput (from NVIDIA)

| Precision | Memory | Throughput | Speedup vs FP16 |
|-----------|--------|------------|-----------------|
| FP16 | 100% | 1× | baseline |
| FP8 | ~50% | ~1.2× | **1.2×** |

### Accuracy (from FP8-LLM paper)

| Model | FP16 | FP8 | Degradation |
|-------|------|-----|-------------|
| LLaMA-2-7B | 14.62 PPL | 14.89 PPL | +0.27 |
| LLaMA-2-13B | 12.43 PPL | 12.71 PPL | +0.28 |
| LLaMA-2-70B | 8.34 PPL | 8.52 PPL | +0.18 |

### vs INT8

| Method | Memory | Speed | Accuracy |
|--------|--------|-------|----------|
| INT8 | ~50% of FP16 | ~1.1× | Variable |
| **FP8** | ~50% of FP16 | **~1.2×** | **More consistent** |

---

## How FP8 Works

### Dynamic vs Static Quantization

**Dynamic Quantization:**
1. Activation scales computed on-the-fly during forward pass
2. No calibration data needed
3. Slightly lower quality but zero setup time

**Static Quantization:**
1. Activation scales computed during calibration with representative data
2. Better accuracy but requires calibration
3. Standard for production deployments

### Mixed-Precision Approach

Most FP8 implementations use **mixed precision**:

```
Forward pass:
├── Linear(Q) → Q = fp16/bf16 weights × fp8 activations
├── Linear(K) → K = fp16/bf16 weights × fp8 activations  
├── Linear(V) → V = fp16/bf16 weights × fp8 activations
└── Linear(O) → O = fp16/bf16 weights × fp8 activations

Where: E4M3 for weights, E5M2 for activations
```

---

## Framework Support

| Framework | FP8 Support | Notes |
|-----------|-------------|-------|
| **vLLM** | ✅ Native | `dtype="fp8"` or `quantization="fp8"` |
| **Transformers** | ✅ Limited | Via `torch_dtype="fp8"` |
| **TensorRT-LLM** | ✅ Full | Recommended for production |
| **llama.cpp** | ⚠️ Via GGUF | FP8 support in Q8_0/Q8_K |
| **AMD ROCm** | ✅ Native | MI300+ GPUs |

---

## Practical Usage

### NVIDIA Transformer Engine

```python
import transformer_engine.pytorch as te

# FP8 forward pass
with te.fp8_autocast(enabled=True):
    output = te.Linear(hidden_size, intermediate_size, input_tensor)
```

### Calibration for Static FP8

```python
from datasets import load_dataset

# Load calibration data
calib_data = load_dataset("wikitext", split="train[:1000]")
calib_data = calib_data.map(lambda x: tokenizer(x["text"]))

# Compute activation scales
quantizer = FP8DynamicQuantizer()
scales = quantizer.calibrate(calib_data)
```

---

## Comparison with INT8

| Aspect | FP8 | INT8 |
|--------|-----|------|
| **Dynamic range** | Wider (FP format) | Narrower (needs scaling) |
| **Precision** | Higher (8-bit FP) | Lower (8-bit integer) |
| **Hardware support** | Blackwell, MI300+ | Universal |
| **Accuracy** | Better preservation | Good for most tasks |
| **Speed** | Faster (native ops) | Variable |

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **NVIDIA FP8 Documentation** | [CUDA Math API](https://docs.nvidia.com/cuda/cuda-math-api/cuda_math_api/struct____nv__fp8__e4m3.html) |
| **FP8-LLM Paper** | [arXiv:2309.12424](https://arxiv.org/abs/2309.12424) |
| **vLLM FP8 Docs** | [vllm fp8](https://docs.vllm.ai/en/stable/quantization/fp8.html) |
| **AMD FP8** | [ROCm docs](https://rocm.docs.amd.com/) |

---

## Limitations

- **Hardware required:** FP8 needs Blackwell (B200+) or AMD MI300+
- **Limited precision:** 8 bits still loses some accuracy vs FP16
- **Not for training:** Primarily inference-focused (gradients still need higher precision)

---

*Last updated: 2026-04-19*
