# TurboQuant & PolarQuant

> **Related:** [TurboTransformers](https://github.com/turbo-transformers/turbo-transformers) — Tencent's high-performance inference engine  
> **PolarQuant:** Part of broader research on mixed-precision quantization strategies

---

## Overview

TurboQuant and PolarQuant are quantization techniques developed in the context of **TurboTransformers** — Tencent's high-performance transformer inference engine. These methods focus on efficient KV-cache compression and mixed-precision strategies for long-context LLM inference.

> *"TurboTransformers achieves 2-3× speedup over HuggingFace baseline on various NLP tasks."*

---

## TurboTransformers

### GitHub Repository

| | |
|---|---|
| **Repo** | [tencent/turbo-transformers](https://github.com/turbo-transformers/turbo-transformers) |
| **Organization** | Tencent |
| **License** | Apache 2.0 |

### Key Features

- **JIT compilation** for dynamic shape inputs
- **Memory-efficient KV cache** via block-wise quantization
- **Mixed-precision support** — different bit widths for different layers
- **CPU/GPU hybrid inference** — offloads to CPU when GPU memory is tight

---

## PolarQuant

### Concept

**PolarQuant** refers to a quantization strategy that uses **mixed-precision allocation** based on sensitivity analysis. It extends the insight that not all layers or tokens are equally important for model quality.

### Key Principles

| Principle | Description |
|-----------|-------------|
| **Layer sensitivity** | Some layers are more sensitive to quantization — allocate more bits |
| **Token importance** | Recent/relevant tokens matter more for KV cache |
| **Dynamic adaptation** | Adjust precision based on cache pressure |

### Relation to KV-Cache Quantization

PolarQuant-inspired techniques are used in:
- **KVTuner** — offline layer-wise precision search
- **Don't Waste Bits!** — token-wise adaptive precision

---

## Research Context

### Related Papers

| Paper | arXiv | Key Contribution |
|-------|-------|------------------|
| **TurboTransformers** | — | High-performance transformer inference engine |
| **Mixed-Precision Quantization** | Various | Layer-wise bit allocation strategies |

### Connection to Other Methods

TurboQuant/PolarQuant concepts influence modern KV-cache methods:

| Modern Method | Technique from TurboQuant/PolarQuant |
|--------------|-------------------------------------|
| **KIVI** | Asymmetric K/V quantization |
| **KVTuner** | Layer-wise precision search |
| **Don't Waste Bits!** | Token-wise adaptive precision |

---

## Technical Details

### Block-wise Quantization

```
Standard quantization:     [W1, W2, W3, ..., Wn] → single scale factor
Block-wise quantization:   [W1..W64] → scale1, [W65..W128] → scale2, ...
```

**Advantage:** Reduces quantization error by adapting to local value distributions

### Mixed-Precision Strategy

```python
# Example: Layer-wise precision allocation
layer_config = {
    "embeddings": "fp16",      # Most sensitive
    "early_layers": "int8",     # Medium sensitivity
    "middle_layers": "int4",   # Less sensitive
    "late_layers": "int8",     # Medium sensitivity
    "lm_head": "fp16"          # Critical for output
}
```

---

## Performance Characteristics

### Memory vs Quality Trade-off

| Configuration | Memory | Quality | Use Case |
|--------------|--------|---------|----------|
| FP16 KV-cache | 100% | 100% | Baseline |
| INT8 KV-cache | 50% | ~99% | General |
| INT4 KV-cache | 25% | ~95-98% | Memory-constrained |
| Mixed-precision | Variable | Optimized | Production |

### Speedup Factors

- **Block-wise quantization** reduces dequantization overhead
- **Mixed-precision** avoids over-compressing sensitive layers
- **Dynamic adaptation** handles varying sequence lengths

---

## Framework Integration

### TurboTransformers Usage

```python
from turbo_transformers import TurboTransformer

# Load model with mixed-precision
model = TurboTransformer.from_pretrained(
    "model_name",
    mixed_precision=True,
    kv_cache_bits=8
)

# Generate
output = model.generate(input_ids)
```

### Applying to Modern Frameworks

The block-wise and mixed-precision concepts from TurboQuant/PolarQuant are implemented in:

| Framework | Implementation |
|-----------|---------------|
| **vLLM** | `kv_cache_dtype="fp8"` with calibration |
| **llama.cpp** | Per-head adaptive quantization |
| **KVTuner** | Offline layer-wise precision search |

---

## Blog Posts & References

| Source | Link |
|--------|------|
| **TurboTransformers GitHub** | [github.com/turbo-transformers](https://github.com/turbo-transformers/turbo-transformers) |
| **Tencent AI Blog** | Original TurboTransformers announcement |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Origin** | Tencent TurboTransformers project |
| **Key Innovation** | Block-wise + mixed-precision quantization |
| **Modern Evolution** | Influenced KIVI, KVTuner, Don't Waste Bits! |
| **Best For** | Long-context inference, memory-constrained部署 |

---

*Last updated: 2026-04-19*
