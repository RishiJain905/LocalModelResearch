# Adaptive KV-Cache Quantization

> **Key Papers:** [Don't Waste Bits! (arXiv:2604.04722)](https://arxiv.org/html/2604.04722v1), [QAQ (arXiv:2403.04643)](https://arxiv.org/html/2403.04643), [KVQuant (arXiv:2401.18079)](https://arxiv.org/html/2401.18079v5), [KVTuner (ICML 2025)](https://openreview.net/forum?id=zDwipF6h06)

---

## Overview

Adaptive KV-Cache quantization dynamically adjusts precision based on token importance, layer sensitivity, and cache pressure — achieving better quality than uniform quantization at the same memory budget.

> *"Optimal bit-width assignment satisfies: b*(x) = f(I(x)), where f maps token importance to quantization class."* — Don't Waste Bits!

---

## Key Papers

| Paper | Venue | arXiv | Key Contribution |
|-------|-------|-------|------------------|
| **Don't Waste Bits!** | arXiv 2026 | [2604.04722](https://arxiv.org/html/2604.04722v1) | Token-wise adaptive precision via MLP controller |
| **QAQ: Quality Adaptive Quantization** | arXiv 2024 | [2403.04643](https://arxiv.org/html/2403.04643) | ~10× compression with <1% accuracy loss |
| **KVQuant** | NeurIPS 2024 | [2401.18079](https://arxiv.org/html/2401.18079v5) | 3-bit with <0.1 perplexity degradation; 10M context |
| **KVTuner** | ICML 2025 | [OpenReview](https://openreview.net/forum?id=zDwipF6h06) | Layer-wise mixed-precision via offline search |
| **ZipCache** | NeurIPS 2024 | [Link](https://neurips.cc/virtual/2024/poster/96563) | Salient token identification + quantization |
| **More for Keys, Less for Values** | arXiv 2025 | [2502.15075](https://arxiv.org/html/2502.15075v1) | K(4-bit)V(2-bit) nearly matches K(4-bit)V(4-bit) |

---

## GitHub Repositories

| Project | Repo | Description |
|---------|------|-------------|
| **Don't Waste Bits!** | [SayedPedramHaeri/Dont-Waste-Bits](https://github.com/SayedPedramHaeri/Dont-Waste-Bits) | MLP controller for token-wise precision {2,4,8,16}-bit |
| **QAQ** | [ClubieDong/QAQ-KVCacheQuantization](https://github.com/ClubieDong/QAQ-KVCacheQuantization) | Quality-adaptive quantization with outlier handling |
| **KVQuant** | [squeezeailab/kvquant](https://github.com/squeezeailab/kvquant) | Per-channel Key, Pre-RoPE quantization, non-uniform types |
| **KVTuner** | [cmd2001/KVTuner](https://github.com/cmd2001/KVTuner) | Offline layer-wise precision search |
| **Quantized Pruning** | [zhzihao/QPruningKV](https://github.com/zhzihao/QPruningKV) | Unified pruning + quantization framework |

---

## How Adaptive KV-Cache Works

### Core Principle: Token Importance

| Feature | Formula | Intuition |
|---------|---------|-----------|
| **Entropy** H_t | H_t = −Σ p_t(v) log p_t(v) | High entropy → sensitive to compression |
| **Rarity** R_t | R_t = −log((c(x_t)+1) / (N+𝓥_obs+1)) | Infrequent tokens carry more semantic info |
| **Attention Variance** V_t | V_t = (1/h) Σ Var(A_i^(L)) | Sharp/even attention = structural sensitivity |
| **Confidence** C_t | Token-level confidence | Model certainty in prediction |

### Layer-Wise Strategies

#### KVTuner (ICML 2025)

> *"We theoretically analyze the inherent correlation of layer-wise transformer attention patterns to KV cache quantization errors and study why key cache is generally more important than value cache."*

- Offline searches optimal precision pairs per layer
- Uses intra-layer KV precision pair pruning and inter-layer clustering
- Achieves **3.25-bit nearly lossless** for Llama-3.1-8B-Instruct

### Key vs Value Asymmetry

| Finding | Evidence |
|---------|----------|
| Key matrices have higher norm values | ‖K‖_F ≫ ‖V‖_F |
| K(4-bit)V(2-bit) accuracy | **75.2%** |
| K(2-bit)V(4-bit) accuracy | 54.7% |
| Spectral norm error | ‖K − K̃‖²_F ≫ ‖V − Ṽ‖²_F at equal bits |

> *"K(4-bit)V(2-bit) nearly matches K(4-bit)V(4-bit) at 50% memory savings on KV cache."*

---

## Performance Benchmarks

### QAQ Compression Ratios (LLaMA 2)

| Model | Task | Compression (<1% acc drop) | vs Scissorhands |
|-------|------|---------------------------|-----------------|
| LLaMA 2-7B | HellaSwag | **7.477×** | 1.6-1.8× improvement |
| LLaMA 2-13B | HellaSwag | **8.394×** | — |

### KIVI-2 Results

| Model | Config | GSM8K | CoQA |
|-------|--------|-------|------|
| Llama-2-7B | 16-bit | 13.50 | 63.88 |
| | KIVI-2 | **12.74** | **63.05** |
| Mistral-7B | 16-bit | 38.36 | 67.40 |
| | KIVI-2 | **36.01** | **66.35** |

### Quantized Pruning Key Finding

> *"Storing 4× tokens in 4-bit precision outperforms storing 1× tokens in 16-bit precision across various KV cache budgets."*

| Method | LongBench (2048 tokens) | NIAH |
|--------|-------------------------|------|
| 16-bit | 42.4 | 100% |
| 4-bit | 41.0 | 100% |
| 2-bit | 13.4 | 27.6% |

### vLLM/TensorRT-LLM Comparison (SqueezeBits)

| Configuration | Decode-heavy Throughput |
|---------------|------------------------|
| TRT-LLM FP8 | **1.45× improvement** |
| vLLM FP8 | Marginal (FlashAttention-2 incompatible) |
| MMLU accuracy | ~0.68 (identical across all) |

---

## Framework Integration

### vLLM FP8 KV Cache

```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="meta-llama/Llama-2-7b-chat-hf",
    kv_cache_dtype="fp8",  # or "fp8_e4m3" or "fp8_e5m2"
    calculate_kv_scales=True
)
```

### KV Cache dtype Options

| kv_cache_dtype | Format | Platform |
|----------------|--------|----------|
| `"fp8"` / `"fp8_e4m3"` | E4M3 (higher precision) | CUDA 11.8+, ROCm |
| `"fp8_e5m2"` | E5M2 (wider range) | CUDA 11.8+ |

### LLM Compressor Calibration

```python
from llmcompressor.transformers import oneshot

recipe = """
quant_stage:
  quant_modifiers:
    QuantizationModifier:
      kv_cache_scheme:
        num_bits: 8
        type: float
        strategy: tensor
"""
oneshot(model=model, dataset=calib_data, recipe=recipe)
model.save_pretrained("model-FP8-KV")
```

### llama.cpp Per-Head Adaptive

> *"Per-head entropy-adaptive bit allocation improves quality by +8% BLEU at the same compression ratio."*

```markdown
# Per-head KV cache types:
- Sink heads (lowest entropy, ~2%): keep at f16 or q8_0
- Standard heads: q4_0
- High-entropy heads: higher precision
```

### TensorRT-LLM

```bash
# FP8 KV cache + FP8 attention
trtllm-build --use_fp8_context_fmha=True model.onnx
```

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **SqueezeBits Blog** | [vLLM vs TensorRT-LLM: KV Cache Quantization](https://blog.squeezebits.com/vllm-vs-tensorrtllm-8-kv-cache-quantization-35079) |
| **vLLM Docs** | [Quantized KV Cache](https://docs.vllm.ai/en/v0.7.1/features/quantization/quantized_kvcache.html) |
| **NVIDIA Blog** | [NVFP4 KV Cache](https://developer.nvidia.com/blog/optimizing-inference-for-long-context-and-large-batch-sizes-with-nvfp4-kv-cache/) |
| **Modular** | [Five Eras of KVCache](https://www.modular.com/blog/the-five-eras-of-kvcache) |
| **Zansara Dev** | [KV Cache Optimizations Ep. 2](https://www.zansara.dev/posts/2025-10-27-kv-caching-optimizations-token-level/) |

---

## Key Takeaways

1. **Adaptive > Static**: Token-wise precision allocation consistently outperforms uniform quantization
2. **Keys need more bits than Values**: K(4)V(2) nearly matches K(4)V(4) at 50% memory savings
3. **Layer-wise tuning**: KVTuner achieves 3.25-bit nearly lossless via offline search
4. **8-bit is practical**: Minimal accuracy impact, 2× cache capacity
5. **vLLM currently limited**: FlashAttention-2 doesn't support FP8 KV; use XFormers/FlashInfer
6. **Quantized pruning**: More tokens + lower precision > fewer tokens + higher precision for long-context

---

*Last updated: 2026-04-19*
