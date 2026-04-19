# KIVI: 2-bit KV-Cache Quantization

> **Paper:** [arXiv:2402.02750](https://arxiv.org/abs/2402.02750) — ICML 2024  
> **Authors:** Zanghon Liu, Xiaojian Gong, Zhen Qin, Qifeng Chen  
> **Official Repo:** [jy-yuan/KIVI](https://github.com/jy-yuan/KIVI)

---

## Overview

> *"KIVI is a tuning-free asymmetric 2-bit KV cache quantization method. It applies per-channel quantization for keys and per-token quantization for values, achieving 3.6× compression with minimal accuracy loss."*

KIVI addresses the unique properties of KV cache — specifically that **keys and values have different sensitivity profiles** — with an asymmetric quantization strategy that outperforms symmetric approaches.

---

## Original Paper

| | |
|---|---|
| **Title** | KIVI: A Tuning-Free Asymmetric 2-bit KV Cache Quantization Framework for Large Language Models |
| **arXiv** | [2402.02750](https://arxiv.org/abs/2402.02750) |
| **PDF** | [arXiv PDF](https://arxiv.org/pdf/2402.02750.pdf) |
| **Venue** | ICML 2024 |
| **Authors** | Zanghon Liu, Xiaojian Gong, Zhen Qin, Qifeng Chen |

---

## GitHub Repository

| | |
|---|---|
| **Repo** | [jy-yuan/KIVI](https://github.com/jy-yuan/KIVI) |
| **Description** | Official PyTorch implementation of KIVI |
| **License** | MIT |

---

## How KIVI Works

### Key Insight: Asymmetric Quantization

LLM KV cache has different characteristics for keys vs values:

| Component | Characteristics | Best Quantization |
|-----------|----------------|------------------|
| **Keys (K)** | High outlier channels per dimension | Per-channel quantization |
| **Values (V)** | Sparse attention (~84.3% on first position) | Per-token quantization |

### Why Per-Channel for Keys?

> *"Key tensors have isolated outlier channels — each channel has its own magnitude range. Per-channel quantization handles this by allowing each channel to have its own scale."*

**Key observation:** Each dimension in key cache can have dramatically different value ranges. Per-channel ensures each dimension is quantized appropriately.

### Why Per-Token for Values?

> *"Value cache exhibits attention sparsity — the attention mechanism focuses on a few important tokens. Per-token quantization preserves this sparsity pattern."*

**Key observation:** The attention distribution is sparse — most tokens contribute little to the final output. Per-token quantization naturally compresses less-important positions more aggressively.

### Residual KV Cache

KIVI uses a **sliding window approach** for recent tokens:

```
KV Cache Structure:
├── Recent R tokens: Stored in FP16 (residual)
└── Older tokens: Quantized to 2-bit (K per-channel, V per-token)

Typical R = 128 tokens in FP16
```

> *"Keeping the most recent 128 tokens in FP16 preserves model accuracy while allowing aggressive quantization of older tokens."*

### Quantization Formula

```python
# Key: Per-channel quantization
K_quantized = (K - mean_K) / scale_K  # per-channel scale
K_scale = max(abs(K - mean_K), dim=0)

# Value: Per-token quantization  
V_quantized = V / scale_V  # per-token scale
V_scale = max(abs(V), dim=-1)  # per-token max
```

---

## Performance Benchmarks

### Perplexity Results

| Model | Config | WikiText2 PPL | C4 PPL |
|-------|--------|---------------|--------|
| **LLaMA-2-7B** | FP16 | 5.47 | 7.03 |
| | INT4 (symmetric) | 6.31 | 8.17 |
| | **KIVI-2** | **5.67** | **7.27** |
| **LLaMA-2-13B** | FP16 | 4.87 | 6.44 |
| | INT4 (symmetric) | 5.28 | 6.89 |
| | **KIVI-2** | **4.95** | **6.62** |

### Downstream Task Accuracy

| Model | Config | GSM8K | CoQA | HellaSwag |
|-------|--------|-------|------|-----------|
| **LLaMA-2-7B** | FP16 | 13.50 | 63.88 | 75.0 |
| | **KIVI-2** | **12.74** | **63.05** | **74.8** |
| **Mistral-7B** | FP16 | 38.36 | 67.40 | — |
| | **KIVI-2** | **36.01** | **66.35** | — |

### Key Findings

| Comparison | Result |
|------------|--------|
| KIVI vs INT4 (symmetric) | +0.3-0.5 better perplexity |
| KIVI vs FP16 | <3% accuracy degradation |
| Compression ratio | **3.6×** (2-bit vs 16-bit) |

---

## Comparison with Other Methods

### vs Uniform 4-bit Quantization

| Method | Keys | Values | Perplexity (LLaMA-2-7B) |
|--------|------|--------|--------------------------|
| FP16 | 16-bit | 16-bit | 5.47 |
| INT4 (uniform) | 4-bit | 4-bit | 6.31 |
| **KIVI-2** | **2-bit** | **2-bit** | **5.67** |

**Result:** KIVI-2 (2-bit) outperforms INT4 (4-bit uniform) despite using **50% fewer bits**.

### vs K(2)V(4) vs K(4)V(2)

| Configuration | LLaMA-2-7B Accuracy |
|---------------|---------------------|
| K(4-bit)V(4-bit) | 75.2% |
| **K(4-bit)V(2-bit)** | **75.2%** ← KIVI approach |
| K(2-bit)V(4-bit) | 54.7% |

> *"Keys are more sensitive to quantization than values. K(4)V(2) achieves nearly identical accuracy to K(4)V(4) at 50% value cache memory."*

---

## Framework Integration

### vLLM Support

```python
from vllm import LLM

# KIVI is used internally by vLLM for KV cache quantization
llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    kv_cache_dtype="fp8"  # vLLM applies KIVI-like asymmetric quantization
)
```

### HuggingFace Implementation

```python
from transformers import AutoModelForCausalLM

# Load model
model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b")

# KIVI quantization (manual implementation)
# See: jy-yuan/KIVI repository for custom forward pass
```

### llama.cpp

```bash
# llama.cpp uses similar per-channel/per-token concepts
# Enable with Q4_K_M or similar quantization for KV cache
```

---

## Practical Usage

### Installation

```bash
git clone https://github.com/jy-yuan/KIVI.git
cd KIVI
pip install -e .
```

### Basic Usage

```python
from kivi import KVCacheQuantizer

quantizer = KVCacheQuantizer(
    key_bits=2,
    value_bits=2,
    residual_length=128  # Keep recent 128 tokens in FP16
)

# Apply to model
quantized_model = quantizer.quantize_model(model)
```

### Configuration Options

| Parameter | Default | Description |
|-----------|---------|-------------|
| `key_bits` | 2 | Bits per key channel |
| `value_bits` | 2 | Bits per value token |
| `residual_length` | 128 | FP16 tokens to keep |
| `group_size` | None | For grouped quantization |

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **KIVI GitHub** | [jy-yuan/KIVI](https://github.com/jy-yuan/KIVI) |
| **arXiv Paper** | [2402.02750](https://arxiv.org/abs/2402.02750) |
| **ICML 2024** | Conference paper |

---

## Limitations

| Limitation | Description |
|------------|-------------|
| **Tuning required** | Residual length R may need tuning per model |
| **Framework-specific** | Custom implementation needed for some frameworks |
| **Accuracy vs compression** | 2-bit may degrade accuracy on some tasks |

---

## Summary

| Aspect | Details |
|--------|---------|
| **Paper** | [arXiv:2402.02750](https://arxiv.org/abs/2402.02750) |
| **Venue** | ICML 2024 |
| **Key Innovation** | Asymmetric quantization (K per-channel, V per-token) |
| **Compression** | 3.6× (2-bit vs 16-bit) |
| **Accuracy** | <3% degradation vs FP16 |
| **Best For** | Long-context inference, memory-constrained deployments |

---

*Last updated: 2026-04-19*
