# HQQ: Half-Quadratic Quantization

> **Official Repo:** [mobiusml/hqq](https://github.com/mobiusml/hqq)  
> **Documentation:** [hqq documentation](https://github.com/mobiusml/hqq?tab=readme-ov-file#readme)

---

## Overview

> *"HQQ (Half-Quadratic Quantization) is a fast and calibration-free quantization method that can quantize any model (LLM, VLM, SSM, etc.) in seconds, with minimal accuracy loss."*

> *"The core idea is simple: model weights can be seen as a sum of two components — a core (low-rank) component that captures most of the model, and the rest. This decomposition allows us to quantize most of the weights without losing information."*

---

## GitHub Repository

| | |
|---|---|
| **Repo** | [mobiusml/hqq](https://github.com/mobiusml/hqq) |
| **License** | Apache 2.0 |
| **Stars** | ~2.5k |
| **Key advantage** | No calibration data needed — quantizes in seconds |

---

## How It Works

### Core Insight: Core+Residual Decomposition

HQQ reframes quantization as an optimization problem using **half-quadratic splitting**:

```
arg min_{C,R,Q} ||W - (C + R)||²₂ + λ(||C||²₂ + ||R||²₂)

subject to: Q quantized, C quantized
```

Where:
- **C** = core (low-rank) component — most important, quantized to high precision
- **R** = residual component — leftover, can be quantized aggressively
- **λ** = controls trade-off between core size and residual quality

> *"This decomposition allows us to quantize most of the weights without losing information."*

### Key Properties

1. **No calibration data required** — unlike GPTQ/AWQ which need a calibration set
2. **Quantizes in seconds** — iterative optimization per layer
3. **Outlier suppression** — naturally handles activation outliers via the core+residual split
4. **Works on any model** — LLM, VLM, SSM (state space models), diffusion models

### Quantization Process

1. **Initialize** core and residual components via SVD or random init
2. **Alternate optimization** — fix one component, optimize the other
3. **Quantize** — apply standard INT4/INT8 to the decomposed components
4. **Fold scales** — combine into final quantized representation

### Outlier Suppression

> *"Activation outliers in LLMs make quantization difficult. HQQ handles this by decomposing weights into core+residual — the core captures most information with high precision, while the residual (which may contain outliers) can be quantized more aggressively."*

---

## Performance Benchmarks

### LLaMA-2 Results

| Model | Bits | WikiText2 PPL | MMLU | Notes |
|-------|------|---------------|------|-------|
| LLaMA-2-7B | FP16 | 5.47 | 45.9 | Baseline |
| LLaMA-2-7B | HQQ 4-bit | ~5.6 | ~45 | Slightly behind |
| LLaMA-2-7B | GPTQ 4-bit | ~5.6 | ~45 | Similar |
| LLaMA-2-13B | FP16 | 4.88 | 55.2 | Baseline |
| LLaMA-2-13B | HQQ 4-bit | ~5.0 | ~54 | Minimal loss |

### vs Other Methods

| Method | Calibration Data | Quant Time | Accuracy (4-bit LLaMA-2-7B) |
|--------|------------------|------------|------------------------------|
| **HQQ** | None | **~30 seconds** | ~5.6 WikiText2 |
| GPTQ | Required | ~10 minutes | ~5.6 WikiText2 |
| AWQ | Required | ~5 minutes | ~5.6 WikiText2 |
| bitsandbytes NF4 | Required | ~5 minutes | ~5.7 WikiText2 |

### VLM Results (LLaVA-1.5)

| Model | Bits | MMBench | ScienceQA |
|-------|------|---------|-----------|
| LLaVA-1.5-7B | FP16 | 64.4 | 73.4 |
| LLaVA-1.5-7B | HQQ 4-bit | **64.2** | **73.0** |

> *"Zero-shot MMBench: 64.2 (vs FP16 64.4) — no degradation."*

### Diffusion Models

> *"HQQ successfully quantizes diffusion models (SDXL, SD-Turbo) where other methods fail due to outliers."*

| Model | Original | HQQ 4-bit | Speedup |
|-------|----------|-----------|---------|
| SD-Turbo | 6.9 GB | **2.1 GB** | ~3× |

---

## Framework Support

### Direct Support

- ✅ **HuggingFace Transformers** — via `from_pretrained` with `device_map`
- ✅ **vLLM** — via `hqq_backend`
- ✅ **llama.cpp** — GGUF export
- ✅ **ComfyUI** — custom node for diffusion models
- ✅ **ExLlamaV2** — native support

### Backend Options

| Backend | Quant Time | Speed | Hardware |
|---------|-----------|-------|----------|
| **PyTorch** | Fastest | Baseline | CPU/GPU |
| **CUDA** | Fast | +20-30% | NVIDIA |
| ** Triton** | Fast | +20-30% | NVIDIA |
| **IQ** (Iterative Quantization) | Slowest | Best quality | CPU/GPU |

---

## Practical Usage

### Installation

```bash
pip install hqq
```

### Basic Python Usage

```python
from hqq.models.hqq_llama import HQQLlamaForCausalLM
from hqq.core.quantize import HQQLinear

# Load and quantize in one line
model = HQQLlamaForCausalLM.from_quantized(
    "meta-llama/Llama-2-7b-hf",
    quantization_config={
        "nbits": 4,
        "group_size": 64,
        "backend": "cuda"
    }
)
```

### Manual Quantization

```python
from hqq.core.quantize import HQQLinear, quantize_model
from hqq.core.utils import set_backend

# Set backend
set_backend("cuda")

# Quantize existing model
quantized_model = quantize_model(
    model,
    {"nbits": 4, "group_size": 64},
    visualize=True
)
```

### HF Transformers Integration

```python
from transformers import AutoModelForCausalLM, AutoConfig
from hqq.utils.backends import *

# Load model
config = AutoConfig.from_pretrained(model_name)
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    config=config,
    torch_dtype=torch.float16,
    device_map="auto"
)

# Quantize
from hqq.core.quantize import HQQLinear
quantize_model(model, {"nbits": 4, "group_size": 64})
```

### vLLM Usage

```python
from vllm import LLM

llm = LLM(
    model="mobiusml/hera-llama3-8b-hqq",
    quantization="hqq"
)
```

### GGUF Export

```python
from hqq.backends.llama_cpp import export_llama_cpp

# Export to GGUF format for llama.cpp
export_llama_cpp(
    model,
    save_path="./quantized_model",
    filename="model.gguf"
)
```

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **HQQ GitHub README** | [mobiusml/hqq](https://github.com/mobiusml/hqq) |
| **HuggingFace Blog** | [Quantization documentation](https://huggingface.co/docs/transformers/en/quantization/hqq) |
| **Mobius Labs Blog** | [HQQ technical overview](https://mobiusml.github.io/hqq/) |

---

## Advantages & Trade-offs

### Advantages

| Advantage | Description |
|-----------|-------------|
| **No calibration data** | Unlike GPTQ/AWQ, HQQ doesn't need representative samples |
| **Seconds quant time** | 30-60 seconds vs 10+ minutes for GPTQ |
| **Any model type** | Works on LLMs, VLMs, SSMs, diffusion models |
| **Outlier robust** | Core+residual naturally handles outliers |
| **Flexible precision** | Can mix bit widths per layer |

### Trade-offs

| Trade-off | Description |
|-----------|-------------|
| **Slightly lower quality** | ~0.1-0.3 perplexity higher than optimized GPTQ |
| **Less community adoption** | Smaller ecosystem than GPTQ/AWQ |
| **Memory** | May use slightly more memory than SOTA methods |

---

## Comparison with Other Methods

| Method | Calibration | Quant Time | Quality (4-bit LLaMA-7B) | Best For |
|--------|-------------|-----------|--------------------------|----------|
| **HQQ** | None | **~30s** | ~5.6 perplexity | Fast iteration, any model |
| **GPTQ** | Required | ~10 min | ~5.5 perplexity | GPU inference, established |
| **AWQ** | Required | ~5 min | ~5.5 perplexity | vLLM production |
| **bitsandbytes NF4** | Required | ~5 min | ~5.7 perplexity | QLoRA fine-tuning |

---

*Last updated: 2026-04-19*
