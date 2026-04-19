# SmoothQuant: Activation Quantization via Outlier Migration

> **Paper:** [arXiv:2211.10438](https://arxiv.org/abs/2211.10438) (v7, March 2024) · [ICML 2023](https://proceedings.mlr.press/v202/xiao23c.html)  
> **GitHub:** [mit-han-lab/smoothquant](https://github.com/mit-han-lab/smoothquant) (official)  
> **Follow-up:** [arXiv:2312.03788](https://arxiv.org/abs/2312.03788) (SmoothQuant+)  
> **Blog:** [MIT-IBM Watson AI Lab](https://mitibmwatsonailab.mit.edu/research/blog/smoothquant-accurate-and-efficient-post-training-quantization-for-large-language-models/)  
> **Integrations:** [vLLM (llm-compressor)](https://docs.vllm.ai/projects/llm-compressor/en/0.8.0/reference/llmcompressor/modifiers/smoothquant/) · [LMDeploy](https://lmdeploy.readthedocs.io/en/latest/quantization/w8a8.html) · [AMD Quark](https://quark.docs.amd.com/latest/pytorch/smoothquant.html) · [Intel Neural Compressor](https://github.com/onnx/neural-compressor/blob/main/docs/smooth_quant.md)

---

## Overview

> *"SmoothQuant enables W8A8 quantization — 8-bit weights and 8-bit activations — with negligible accuracy loss, achieving up to 1.56× speedup and 2× memory reduction."* — Xiao et al., ICML 2023

**SmoothQuant** is a **training-free, post-training quantization (PTQ)** method that solves the core problem preventing activation quantization: **per-channel outliers** in LLM activations. These outliers force activation quantization to use a high bit precision or suffer severe accuracy loss.

**Core insight:** Migrate the quantization difficulty from activations to weights using a mathematically principled **migration factor**.

---

## The Problem: Why Activation Quantization Is Hard

LLM activations contain **per-channel outliers** — certain channels have values up to **100× larger** than other channels. This makes per-token quantization ineffective:

```
Standard per-token quantization:
- Scale = max(|X_token|) → large range per channel
- Most channels under-utilize their precision
- Accuracy drops significantly
```

### Per-Channel Outlier Example (OPT-175B)

| Channel | Max Activation | Relative Scale |
|---------|--------------|----------------|
| Outlier channel | ~100 | 100× |
| Median channel | ~1 | baseline |
| Small channel | ~0.1 | 0.1× |

---

## The Solution: Migration Factor

**Key equation:**

```
S_migration[j] = max(|X_j|) / max(|W_j|)^(1-α)

Y = (X · diag(s)⁻¹) · (diag(s) · W)
```

Where:
- `X` = input activations
- `W` = weight matrix
- `s` = per-channel migration factor
- `α` ∈ [0,1] = migration strength (user-controlled)

### How It Works

| Step | Operation | Effect |
|------|-----------|--------|
| **1. Compute migration factor** | `s_j = max(|X_j|)^(1-α) / max(|W_j|)^(1-α)` | Channels with large activations get smaller scales |
| **2. Pre-scale activations** | `X' = X · diag(s)⁻¹` | Activations become uniform across channels |
| **3. Pre-scale weights** | `W' = diag(s) · W` | Weights absorb the migration |
| **4. Quantize both** | W8A8 via standard per-tensor quantization | No outliers in activations |
| **5. Absorb during inference** | Merged into adjacent operations | Zero overhead |

> *"By migrating the quantization difficulty from activations to weights, we enable INT8 quantization of both weights and activations without accuracy loss."* — [MIT-IBM Blog](https://mitibmwatsonailab.mit.edu/research/blog/smoothquant-accurate-and-efficient-post-training-quantization-for-large-language-models/)

---

## Theoretical Foundation

### Quantization Error Analysis

For a given channel `j`, after migration:

```
Original: Q(X_j) ≈ X_j / max(|X_j|)     → large dynamic range
Smoothed: Q(X'_j) ≈ X'_j / max(|X'_j|)  → uniform across channels
```

The migration factor `α` controls the trade-off:

| α | Effect |
|---|--------|
| **α = 0** | No migration — original quantization |
| **α = 1** | Full migration — all difficulty moved to weights |
| **α ∈ (0,1)** | Balanced — typically α = 0.5 |

### Why It Doesn't Hurt Accuracy

> *"The migration does not change the mathematical equivalence of the operation — it merely reorganizes which tensor carries the quantization difficulty. The output Y is mathematically identical whether computed in FP16 or INT8."*

The key property: `Y = XW = (X·s⁻¹)(s·W)`, which is mathematically equivalent.

---

## Performance Results

### Speedup & Memory (from original paper)

| Model | Original Speedup | Memory Reduction | 
|-------|----------------|-----------------|
| **MT-NLG 530B** | 1.56× | 2× |
| **OPT-175B** | 1.4× | 1.9× |
| **BLOOM-176B** | 1.35× | 1.85× |

> **Result:** Enables serving **530B MT-NLG within a single node** using W8A8 quantization.

### Accuracy (Perplexity on WikiText-2)

| Model | FP16 | W8A8 (SmoothQuant) | Δ |
|-------|------|-------------------|---| 
| **OPT-125M** | 27.07 | 27.31 | +0.24 |
| **OPT-1.3B** | 14.97 | 15.05 | +0.08 |
| **OPT-13B** | 12.14 | 12.22 | +0.08 |
| **LLaMA-2-7B** | 5.87 | 5.89 | +0.02 |

> *"Negligible perplexity degradation across OPT, BLOOM, GLM, Llama-2/3, Falcon, Mistral, and Mixtral."* — [arXiv:2211.10438](https://arxiv.org/abs/2211.10438)

### vLLM Benchmark (SmoothQuant + W4A16)

| Setting | Throughput Improvement |
|---------|----------------------|
| **A100 + LLaMA-2-7B** | 1.73× more tokens/sec vs FP16 |
| **A100 + Vicuna-33B** | 1.96× more tokens/sec vs FP16 |

---

## SmoothQuant+

> **Paper:** [arXiv:2312.03788](https://arxiv.org/abs/2312.03788) — "SmoothQuant+: Accurate and Efficient 4-bit Post-Training Weight Quantization for LLM" (December 2023)

Extends SmoothQuant to **W4A8** and **W4** quantization using **group-wise smoothing** — divides channels into groups with separate migration factors per group.

### vs Original SmoothQuant

| Aspect | SmoothQuant | SmoothQuant+ |
|--------|------------|--------------|
| **Weight bits** | 8 | 4 |
| **Activation bits** | 8 | 8 |
| **Key innovation** | Per-channel migration | Group-wise migration |
| **Integration** | vLLM (llm-compressor) | vLLM (llm-compressor) |

---

## Related Work: Handling Activation Spikes

### QFeM & QFeP (NeurIPS 2024)

> **OpenReview:** [openreview.net/forum?id=t8ch1OCvHh](https://openreview.net/forum?id=t8ch1OCvHh)  
> **Authors:** Jaewoo Yang, Hayun Kim, Younghoon Kim

Proposes **Quantization-free Module (QFeM)** and **Quantization-free Prefix (QFeP)** to isolate activation spikes in **GLU-based LLMs** (LLaMA-2/3, Mistral, Mixtral).

> *"Complements SmoothQuant by handling residual activation spikes that SmoothQuant does not fully resolve in GLU variants."*

| Method | Target | Approach |
|--------|--------|----------|
| **QFeM** | FFN/MLP modules | Keep outlier channels in FP16 |
| **QFeP** | Attention prefix | Isolate prefix computation |
| **Both** | GLU-based models | Residual spike isolation |

### SpinQuant (ICLR 2025) — Learned Rotations

> **arXiv:** [2405.16406](https://arxiv.org/abs/2405.16406) · **GitHub:** [facebookresearch/SpinQuant](https://github.com/facebookresearch/SpinQuant)

Uses **Cayley optimization** to learn rotation matrices that minimize quantization error, rather than using fixed random rotations:

> *"SpinQuant reduces the accuracy gap to 2.9 points for LLaMA-2 7B W4A4KV4, compared to 25+ points with SmoothQuant."*

---

## Usage & Implementation

### Python API (from official repo)

```python
from smoothquant.smooth import smooth_lm
from smoothquant.quantization import DynamicActQuantizedLinear

# Apply SmoothQuant to a model
model = smooth_lm(model, alpha=0.5)  # alpha ∈ [0, 1]

# Use quantized linear layers
layer = DynamicActQuantizedLinear(in_features, out_features)
```

### LMDeploy Usage

```python
# LMDeploy supports SmoothQuant out of the box
lmdeploy serve api_server model_name \
  --quant-policy 1  # enables W8A8 SmoothQuant
```

### vLLM / llm-compressor

```python
from llm_compressor import parse_recursive
from llm_compressor.modifiers.smoothquant import SmoothQuantModifier

# Apply SmoothQuant + weight quantization
parse_recursive(
    model,
    modifiers=[
        SmoothQuantModifier(alpha=0.5),
        # ... weight quantization modifiers
    ]
)
```

---

## Framework Support

| Framework | SmoothQuant Support | URL |
|-----------|-------------------|-----|
| **vLLM** | ✅ Via llm-compressor | [docs.vllm.ai](https://docs.vllm.ai/projects/llm-compressor/en/0.8.0/reference/llmcompressor/modifiers/smoothquant/) |
| **LMDeploy** | ✅ Built-in | [lmdeploy.readthedocs.io](https://lmdeploy.readthedocs.io/en/latest/quantization/w8a8.html) |
| **AMD Quark** | ✅ Built-in | [quark.docs.amd.com](https://quark.docs.amd.com/latest/pytorch/smoothquant.html) |
| **Intel Neural Compressor** | ✅ Built-in | [github.com/onnx/neural-compressor](https://github.com/onnx/neural-compressor/blob/main/docs/smooth_quant.md) |
| **HuggingFace** | 🔄 Manual | Apply via `smooth_lm()` then use Transformers |
| **llama.cpp** | ❓ | Not directly supported (GGML) |

---

## GitHub Repositories

| Repo | Description | Language |
|------|-------------|----------|
| [mit-han-lab/smoothquant](https://github.com/mit-han-lab/smoothquant) | **Official implementation** — ICML 2023. Includes act_scales for Llama/Mistral/Mixtral/Falcon/OPT/BLOOM, real INT8 demo | Python |
| [AniZpZ/AutoSmoothQuant](https://github.com/AniZpZ/AutoSmoothQuant) | User-friendly wrapper — supports Llama-2/3, Mixtral, OPT, Baichuan | Python |
| [usstq/ov.smoothquant](https://github.com/usstq/ov.smoothquant) | OpenVINO implementation | Python |
| [facebookresearch/SpinQuant](https://github.com/facebookresearch/SpinQuant) | Learned rotation matrices — ICLR 2025, W4A4KV4 support | Python |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **"SmoothQuant: Accurate and Efficient Post-Training Quantization for LLMs"** | MIT-IBM Watson AI Lab | [Link](https://mitibmwatsonailab.mit.edu/research/blog/smoothquant-accurate-and-efficient-post-training-quantization-for-large-language-models/) |
| **"SmoothQuant Explained"** | Substack (Aakash Varma) | [Link](https://aakashvarma.substack.com/p/smoothquant) |
| **"SmoothQuant — Accurate and Efficient PTQ for LLMs"** | Medium (Poorna Ravuri) | [Link](https://medium.com/@poorna.ravuri/smoothquant-accurate-and-efficient-post-training-quantization-for-large-language-models-8d823f71d6c0) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Per-channel activation outliers prevent INT8 activation quantization |
| **Core idea** | Migrate quantization difficulty from activations to weights via `Y = (X·s⁻¹)(s·W)` |
| **Key parameter** | `α` — controls migration strength (0.5 is typical) |
| **Result** | W8A8 with negligible accuracy loss, 1.56× speedup, 2× memory reduction |
| **Extensions** | SmoothQuant+ (W4), QFeM/QFeP (GLU spike handling), SpinQuant (learned rotations) |
| **Training required** | None — fully post-training |
| **Best for** | Serving LLMs with W8A8 on INT8-capable hardware (A100, H100, AMD) |

---

*Last updated: 2026-04-19*
