# EXL3: Extended Linear Learning Quantization

> **Also known as:** exllama2  
> **Official Repo:** [turboderp/exllamav2](https://github.com/turboderp/exllamav2)  
> **Documentation:** [exllama v2 README](https://github.com/turboderp/exllamav2)

---

## Overview

EXL3 (Extended Linear Learning) — implemented as **exllama v2** — is a high-performance GPTQ-based quantization method optimized for CUDA kernels with fused operations. It achieves excellent inference speeds through matrix fusion and carefully optimized dequantization kernels.

> *"EXL2 is designed specifically for speed — aiming for maximum inference throughput rather than minimum memory usage."*

---

## Key Repositories

| Repo | Description | Note |
|------|-------------|------|
| [turboderp/exllamav2](https://github.com/turboderp/exllamav2) | Main exllama v2 repo | Active development |
| [turboderp/exllama](https://github.com/turboderp/exllama) | Original exllama (v1) | Legacy |
| [IST-DASLab/gptq](https://github.com/IST-DASLab/gptq) | Base GPTQ algorithm | EXL3 builds on GPTQ |

---

## How EXL3 Works

EXL3 is a **GPTQ-based method** with several optimizations:

### Core Improvements over GPTQ

1. **Extended weight representations** — allows non-4-bit quantization (2, 3, 5, 6, 8 bits) with per-column optimal scale factors
2. **Fused dequantization kernels** — computes `x @ (w * scale)` in a single fused CUDA kernel
3. **Matrix fusion** — groups multiple small matrices into larger ones for better GPU utilization
4. **Dynamic activation ordering** — reorders columns by activation magnitude for better throughput

### Quantization Format

```
Extended (EXL2) format: Each weight column has its own scale factor
- Scales are computed to minimize reconstruction error
- Supports arbitrary bit widths per column
```

---

## Performance Benchmarks

### Memory & Speed (Llama 2 13B, RTX 4090)

| Format | Size | VRAM | Tokens/s |
|--------|------|------|----------|
| FP16 | 26.0 GB | 26.0 GB | ~42 |
| GPTQ 4-bit | 7.2 GB | 9.8 GB | ~55 |
| **EXL3 4-bit** | 7.2 GB | **~9.0 GB** | **~58-65** |

### Accuracy Comparison (perplexity, lower is better)

| Model | GPTQ 4-bit | EXL3 4-bit |
|-------|------------|------------|
| Llama-2-13B | 4.97 | ~4.96 |
| Llama-2-70B | 3.42 | ~3.40 |

*EXL3 typically matches or slightly outperforms GPTQ at same bit width due to extended precision options.*

### Speed Advantages

- **Fused kernels** eliminate separate dequantization overhead
- **Matrix fusion** improves tensor core utilization
- **Extended precision** (e.g., 6-bit) often faster than 4-bit with less accuracy loss

---

## Comparison: EXL3 vs GPTQ vs AWQ

| Aspect | EXL3 | GPTQ | AWQ |
|--------|------|------|-----|
| **Base algorithm** | GPTQ | OBQ | Activation-aware scaling |
| **Bit widths** | 2-8 arbitrary | 4 (primary) | 4 |
| **Kernel fusion** | ✅ Yes | ❌ No | ❌ No |
| **Per-column scales** | ✅ Optimal | ✅ Yes | ✅ Yes |
| **Speed** | **Fastest** | Fast | Medium |
| **Memory** | Very good | Good | Good |
| **Accuracy** | Excellent | Excellent | Excellent |

---

## Usage

### Installation

```bash
pip install exllamav2
```

### Quantize a Model

```bash
python convert.py \
    -ss /path/to/model \
    -od /path/to/output \
    -hb /path/to/calibration/data \
    -b 4 \
    -f
```

### Inference with exllama v2

```python
from exllamav2 import ExLlamaV2, ExLlamaV2Config, ExLlamaV2Cache

config = ExLlamaV2Config()
config.model_dir = "/path/to/model"
config.template = "llama"
config.max_seq_len = 4096

model = ExLlamaV2(config)
cache = ExLlamaV2Cache(model)

# Generate
generator = ExLlamaV2Sampler()
output = generator.generate(
    model,
    cache,
    "Your prompt here",
    max_new_tokens=200
)
```

### llama.cpp Integration

EXL3/EXL2 format is supported in **llama.cpp** as of late 2024:

```bash
# Use exllama v2 to export
python convert.py -ss model/ -od ./out/ -ft exl2
```

---

## Blog Posts & Technical Explanations

- **oobabooga blog:** [Benchmark results for exl2, gptq, awq](https://old.reddit.com/r/LocalLLaMA/comments/1b8wmxi/full_ benchmark_results_for_exl2_gptq_awq/)
- **r/LocalLLaMA discussions:** [Exllama v2 technical deep-dive](https://www.reddit.com/r/LocalLLaMA/search?q=exllama+v2)
- **HuggingFace blog:** [Quantization formats explained](https://huggingface.co/blog/quantization)

---

## Caveats

- EXL3 is **GPU-only** (CUDA)
- Quantization time can be longer than vanilla GPTQ due to extended search
- Best with **NVIDIA Ampere+ GPUs** (3090, 4090, A100, H100)

---

*Last updated: 2026-04-19*
