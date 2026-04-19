# Compressed Tensors: Unified Compression Format for LLMs

> **GitHub:** [vllm-project/compressed-tensors](https://github.com/vllm-project/compressed-tensors) (transferred from [neuralmagic](https://github.com/neuralmagic/compressed-tensors))  
> **PyPI:** [pypi.org/project/compressed-tensors](https://pypi.org/project/compressed-tensors) (v0.15.0.1, Apr 2026)  
> **HuggingFace:** [huggingface.co/docs/transformers/quantization/compressed_tensors](https://huggingface.co/docs/transformers/quantization/compressed_tensors)  
> **Companion tool:** [vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor)  
> **Related:** [SparseGPT](https://arxiv.org/pdf/2301.00774) · [neuralmagic.com](https://neuralmagic.com)

---

## Overview

> *"As model compression becomes increasingly important for efficient deployment of LLMs, the landscape of quantization and compression techniques has become increasingly fragmented. Each method often comes with its own storage format and loading procedures... compressed-tensors addresses this by providing a single, extensible format."* — [PyPI description](https://pypi.org/project/compressed-tensors)

**Compressed Tensors** is a **unified checkpoint format** for compressed LLM weights — it can represent multiple compression schemes (quantization + sparsity) in a single extensible format. It was created by Neural Magic, acquired by Red Hat, then transferred to the vLLM organization in 2024–2025.

**Key differentiator from GGUF/AWQ/GPTQ:** GGUF, AWQ, and GPTQ are individual quantization methods/formats. Compressed Tensors is a **unifying format layer** that can store GPTQ, AWQ, SmoothQuant, FP8, INT8, and **SparseGPT**-pruned weights together.

---

## Compressed Tensors vs Other Formats

| Aspect | Compressed Tensors | GGUF | AWQ | GPTQ |
|--------|-------------------|------|-----|------|
| **Type** | Unified storage format | Standalone inference format | Quantization method | Quantization method |
| **Purpose** | Store multiple compression schemes | Storage + inference | Activation-aware weight quantization | PTQ algorithm |
| **Ecosystem** | vLLM-native, HF-compatible | llama.cpp | Various backends | Various backends |
| **Sparsity support** | ✅ **Yes** (SparseGPT, 2:4) | ❌ No native sparsity | ❌ No | ❌ No |
| **Multi-method unified** | ✅ Yes | ❌ No | ❌ No | ❌ No |
| **llama.cpp** | ❌ Incompatible | ✅ Native | ❌ | ❌ |

---

## Format Architecture

### Core Concept

```
compressed-tensors format
├── weights.safetensors (compressed)
│   ├── Layer1.weights: quantized + sparsity mask
│   ├── Layer2.weights: quantized + sparsity mask
│   └── ...
└── compression_config.json
    ├── compression_type: ["dense", "sparsity", "quantization"]
    ├── sparsity_scheme: {format, block_size, ...}
    ├── quantization_scheme: {format, bit_depth, ...}
    └── sparsity_format: "2:4" | "unstructured" | ...
```

### Supported Compression Schemes

| Scheme | Method | Notes |
|--------|--------|-------|
| **GPTQ** | Weight-only INT4/INT8 | Per-channel/group |
| **AWQ** | Activation-aware weight | Per-channel scaling |
| **SmoothQuant** | Per-channel migration | W8A8 |
| **FP8** | Floating-point 8-bit | E4M3/E5M2 |
| **INT8** | 8-bit integer | Weight + activation |
| **SparseGPT** | 2:4 structured sparsity | ICML 2023 |
| **Mixed** | Combine any above | Per-layer config |

---

## Integration

### vLLM (Primary)

```python
from vllm import LLM

# Load compressed-tensors checkpoint directly
llm = LLM("./model-compressed/")

# vLLM handles KV cache quantization schemes
# Uses CUTLASS kernels for INT8/FP8 activation quantization
```

**vLLM docs:** [docs.vllm.ai/en/latest/api/vllm/model_executor/layers/quantization/compressed_tensors/compressed_tensors/](https://docs.vllm.ai/en/latest/api/vllm/model_executor/layers/quantization/compressed_tensors/compressed_tensors/)

### HuggingFace Transformers

```python
from transformers import AutoModelForCausalLM, AutoConfig

# Load via HF Transformers
model = AutoModelForCausalLM.from_pretrained(
    "model-path",
    quantization_config="compressed_tensors"
)
```

**HF docs:** [huggingface.co/docs/transformers/quantization/compressed_tensors](https://huggingface.co/docs/transformers/quantization/compressed_tensors)

### llama.cpp Compatibility

> ⚠️ **Incompatible.** Compressed tensors (`.safetensors`) cannot be converted to GGUF via llama.cpp's `convert_hf_to_gguf.py`. This is by design — it's vLLM-native only.

---

## LLM Compressor: Creating Compressed Checkpoints

> **GitHub:** [vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor)

LLM Compressor is the companion tool for creating compressed-tensors checkpoints:

```python
from llmCompressor import parse_recursive
from llmCompressor.modifiers import (
    GPTQModifier,
    SmoothQuantModifier,
    SparesGPTModifier,
    FP8Modifier
)

# Apply quantization to model
parse_recursive(
    model,
    modifiers=[
        GPTQModifier(precision="int4"),
        SmoothQuantModifier(alpha=0.5),
    ]
)

# Save as compressed-tensors format
model.save_compressed("output-model/")
```

### Supported Modifiers

| Modifier | Description |
|----------|-------------|
| `GPTQModifier` | GPTQ weight quantization |
| `AWQModifier` | Activation-aware weight quantization |
| `SmoothQuantModifier` | SmoothQuant migration |
| `FP8Modifier` | FP8 quantization |
| `SparseGPTModifier` | 2:4 structured sparsity |
| `QuIPModifier` | QuIP/QuIP# incoherence processing |

---

## Performance Benchmarks

### W8A8 (INT8) on Llama 3.1 70B (4×A100)

| Method | Throughput | Details |
|--------|-----------|---------|
| **W8A8 (INT8)** | Up to **3×** | Weight + activation quantization |
| W4A16 (INT4 weight-only) | Up to 4× latency | Weight-only, limited for compute-bound |
| 2:4 Sparse (SparseGPT) | Up to 1.5× | Structured sparsity |

### Accuracy Preservation (W8A8 vs FP16)

| Benchmark | FP16 Baseline | W8A8 INT8 | Recovery |
|-----------|--------------|-----------|---------|
| MMLU (5-shot) | 83.88 | 83.65 | 99.7% |
| MMLU (CoT) | 85.74 | 85.41 | 99.6% |
| ARC Challenge | 93.26 | 93.26 | 100.0% |
| GSM-8K | 93.10 | 93.25 | 100.2% |
| **Average** | **83.89** | **83.96** | **100.2%** |

> **Key insight from Neural Magic/Red Hat:** Activation quantization (W8A8) maintains accuracy; weight-only quantization (W4A16) often fails to deliver speed improvements for compute-bound workloads.

---

## SparseGPT: The Sparsity Foundation

> **Paper:** [arXiv:2301.00774](https://arxiv.org/pdf/2301.00774) (ICML 2023)  
> **GitHub:** [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt)

**SparseGPT** is the one-shot pruning algorithm that Compressed Tensors uses for **2:4 structured sparsity**:

```
2:4 sparsity: Every 4 weights, 2 are zero
→ 50% sparsity with hardware-friendly pattern
→ 2:4 is the minimum pattern that works with existing Tensor Cores
```

| Aspect | Details |
|--------|---------|
| **Paper** | "SparseGPT: Massive Language Models Can be Accurately Pruned in One-shot" |
| **Sparsity type** | 2:4 structured (every 4 weights, 2 zero) |
| **Accuracy** | Near-lossless at 2:4 sparsity |
| **Speedup** | Up to 1.5× with Sparse-Marlin |
| **Integration** | `SparseGPTModifier` in llm-compressor |

---

## History & Organization

```
SparseGPT (ICML 2023, Frantar et al.)
    ↓
Neural Magic implements in LLM Compressor
    ↓
Compressed Tensors (storage format) + LLM Compressor (compression tool)
    ↓
Neural Magic acquired by Red Hat (2024)
    ↓
Development transferred to vLLM org (2024–2025)
    ↓
Now: vllm-project/compressed-tensors (active)
```

**Neural Magic** still offers enterprise distribution: **nm-vllm** — [neuralmagic.com](https://neuralmagic.com)

---

## Papers & References

| Resource | URL |
|----------|-----|
| **Main GitHub (vLLM)** | [github.com/vllm-project/compressed-tensors](https://github.com/vllm-project/compressed-tensors) |
| **Original GitHub (Neural Magic)** | [github.com/neuralmagic/compressed-tensors](https://github.com/neuralmagic/compressed-tensors) |
| **LLM Compressor** | [github.com/vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor) |
| **PyPI** | [pypi.org/project/compressed-tensors](https://pypi.org/project/compressed-tensors) |
| **HF Transformers Docs** | [huggingface.co/docs/transformers/quantization/compressed_tensors](https://huggingface.co/docs/transformers/quantization/compressed_tensors) |
| **SparseGPT Paper** | [arxiv.org/pdf/2301.00774](https://arxiv.org/pdf/2301.00774) |
| **SparseGPT Repo** | [github.com/IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt) |
| **Red Hat Article** | [developers.redhat.com/articles/2024/08/14/llm-compressor-here-faster-inference-vllm](https://developers.redhat.com/articles/2024/08/14/llm-compressor-here-faster-inference-vllm) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Type** | Unified compression checkpoint format (extends safetensors) |
| **Creator** | Neural Magic → Red Hat → vLLM org |
| **Key advantage** | Stores multiple compression schemes (quantization + sparsity) in one format |
| **Sparsity** | Native 2:4 via SparseGPT |
| **vLLM support** | ✅ Native, primary target |
| **llama.cpp** | ❌ Incompatible |
| **HF Transformers** | ✅ Via transformers integration |
| **Companion tool** | LLM Compressor for creating compressed checkpoints |

---

*Last updated: 2026-04-19*
