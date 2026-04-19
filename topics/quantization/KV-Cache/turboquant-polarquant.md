# TurboQuant & PolarQuant

> **Paper:** [TurboQuant: Online Vector Quantization with Near-optimal Distortion Rate (ICLR 2026)](https://arxiv.org/abs/2504.19874)  
> **PolarQuant:** [PolarQuant: Online Vector Quantization (AISTATS 2026)](https://arxiv.org/pdf/2502.02617)  
> **QJL:** [Quantized Johnson-Lindenstrauss (AAAI 2025)](https://dl.acm.org/doi/10.1609/aaai.v39i24.34773)  
> **Blog:** [Google Research Blog](https://research.google/blog/turboquant)  
> **Code:** [0xSero/turboquant](https://github.com/0xSero/turboquant) · [sharpner/turboquant-mlx](https://github.com/sharpner/turboquant-mlx) · [turboquant.net (independent analysis)](https://turboquant.net/)

---

## Overview

TurboQuant is a **training-free, data-oblivious** online vector quantization framework from Google Research (released March 2026, ICLR 2026) that achieves near-information-theoretic-optimal compression. The key claim: **3-bit KV cache compression with zero measurable accuracy loss** — no retraining, no fine-tuning.

The name is shared with an older Tencent project (TurboTransformers), which is a **different thing entirely**. This entry covers only the Google Research 2026 work.

> *"TurboQuant is not just another compression trick. It is an online quantization framework that gets close to the information-theoretic limit while staying data-oblivious and accelerator-friendly."* — turboquant.net

---

## How It Works: Two-Stage Design

**TurboQuant = PolarQuant (main compression) + QJL (residual correction)**

### Stage 1: PolarQuant (AISTATS 2026)

PolarQuant applies a **random orthogonal rotation** to each vector before quantization:

1. Generate a random orthogonal matrix `Π` via QR decomposition of a Gaussian matrix
2. Rotate the vector: `y = Π @ x`
3. Group the `d` dimensions into `d/2` pairs
4. Convert each pair into **radius + angle** (polar coordinates)
5. Apply **recursive polar transforms** on the radii
6. The radii follow a **Beta distribution** — highly concentrated near extremes, easy to quantize
7. Quantize **only the angles** using Lloyd-Max optimal scalar quantization

The key insight is that after random rotation, coordinates follow a known, concentrated distribution that is near-optimal for scalar quantization:

```
f_X(x) = Γ(d/2) / (√π · Γ((d-1)/2)) × (1 - x²)^((d-3)/2)   where x ∈ [-1, 1]
```

**Advantages of PolarQuant:**
- No per-block full-precision constants needed
- Near-lossless beyond 4.2× compression
- Supports optimal Lloyd-Max scalar quantizers
- Data-oblivious (no calibration data required)

### Stage 2: QJL — Quantized Johnson-Lindenstrauss (AAAI 2025)

Even PolarQuant alone leaves residual error. QJL handles this with a **1-bit unbiased inner-product estimator**:

- High-precision query vectors pair with the simplified stored data during attention scoring
- The estimator is **unbiased**: E[estimated inner product] = true inner product
- Removes normalization overhead entirely

### Combined Result

| Component | What it does |
|-----------|-------------|
| **PolarQuant** | Compresses rotated vectors to b bits via Lloyd-Max on angles |
| **QJL** | Handles the 1-bit residual for unbiased attention scoring |
| **Together** | Achieves ~6× KV cache memory reduction with zero accuracy loss |

---

## Theoretical Foundation

### Distortion Lower Bounds

- **MSE lower bound:** D_MSE ≥ 1/4^b
- **Inner-product lower bound:** D_prod ≥ (||y||² / d) · 1/4^b

Classical approaches (e.g., Product Quantization) remain noticeably above these bounds. TurboQuant approaches them.

### Distortion Formulas

```
MSE:         D_MSE = E[||x - x̂||²]
Inner product: D_prod = E[|⟨y,x⟩ - ⟨y,x̂⟩|²]
```

### Theoretical Guarantees (Theorems 1–3)

| Theorem | Claim | Verified |
|---------|-------|---------|
| Theorem 1 | MSE distortion bounds | ✓ Within bounds for unit-norm vectors |
| Theorem 2 | Unbiasedness of combined estimator | ✓ Relative bias < 0.1% |
| Theorem 3 | Distortion scales as 1/4^b | ✓ 2-bit=0.70×, 3-bit=0.82×, 4-bit=0.97× of bound |

---

## KV Cache Problem & Solution

### The Memory Problem

```
memory ≈ 2 × L × d × 2 bytes (FP16) per layer
128K context + 7B model = tens of GB of KV cache
KV cache share of total memory: 80%+
```

### What TurboQuant Changes

| Property | Before | After |
|----------|--------|-------|
| KV cache bits per value | 16 (FP16) | **3** (near-lossless) |
| Memory usage | 100% | **~17%** |
| Attention speed (H100, 4-bit) | baseline | **8× faster** |
| LongBench quality | 50.06 | **50.06** (identical) |
| Model retraining | required | **none** |

---

## Benchmark Results

### KV Cache Compression (LongBench)

| Benchmark | TurboQuant 3.5-bit | TurboQuant 2.5-bit | Full Cache |
|-----------|-------------------|-------------------|------------|
| **LongBench** | 50.06 | 49.44 | 50.06 |
| **Needle In A Haystack** | 100% | 99.8% | 100% |
| **ZeroSCROLLS** | best | near-best | baseline |
| **RULER** | best | near-best | baseline |
| **L-Eval** | best | near-best | baseline |

### Vector Search (GloVe d=200)

| Method | Recall | Indexing Time |
|--------|--------|---------------|
| **TurboQuant** | highest | ≈ 0 (no training) |
| RabbiQ | lower | middle |
| PQ | lower | long |

### Comparison vs Alternatives

| Method | Needs Training | Unbiased | Compression | Speedup |
|--------|--------------|----------|-------------|---------|
| **TurboQuant** | No | Yes | 6×+ | 8× |
| KIVI | Calibration | No | 4× | 4× |
| SnapKV | Fine-tuning | No | 2-4× | 2-4× |
| DuQuant | Calibration | Partial | 4× | 4× |

### Real-World: RTX 5090 — Qwen3.5-27B-AWQ (4-bit weights, TP=1)

| Metric | Baseline (bf16 KV) | TurboQuant (3b key / 2b val) |
|--------|-------------------|-------------------------------|
| Prefill tok/s (30k ctx) | 1,804 | **1,907** (+5.7%) |
| Decode tok/s (30k ctx) | 1.264 | **1.303** (+3.1%) |
| KV cache freed | — | **30.0 GB** |
| Max token capacity | 457,072 | **914,144** (2×) |
| Peak activation memory | 644.6 MB | **599.2 MB** (-7%) |

### Real-World: 8× RTX 3090 — Qwen3.5-35B-A3B MoE (TP=8)

- **Context extension:** 1.41M → 2.04M tokens (1.45× capacity)
- **Freed VRAM:** Supports 3 additional concurrent 131k-context requests per GPU
- **Needle retrieval:** 5/5 needles at all context lengths (512–131k tokens)
- **Multi-fact coherence:** 3/3 retrieved

### Compression Quality Breakdown

| Component | cos_sim | Notes |
|-----------|---------|-------|
| Key compression (3-bit) | **1.000000** | Near-lossless |
| Key compression (4-bit) | **1.000000** | Near-lossless |
| Value quantization (2-bit) | 0.940 | **Bottleneck** for quality |
| Value quantization (4-bit) | 0.997 | Recommended for quality-sensitive |
| Combined (3b key + 2b val) | 0.940 | Value quant dominates degradation |

> **Key insight:** Value quantization is the quality bottleneck, not key quantization. For quality-sensitive workloads, use 4-bit value quantization.

---

## Implementation

### Algorithm Sketch

```python
import numpy as np

# Step 1: Precompute Lloyd-Max centroids for Beta distribution
centroids = lloyd_max_quantizer(distribution="beta", bits=b)

# Step 2: Generate random orthogonal rotation matrix (once, offline)
G = np.random.randn(d, d)
Pi, _ = np.linalg.qr(G)  # columns are orthogonal

# Step 3: TurboQuant compression
def quant_turbo(x, Pi, centroids):
    y = Pi @ x                           # random rotation
    idx = find_nearest_centroid(y, centroids)  # Lloyd-Max quantization
    return idx

def dequant_turbo(idx, Pi, centroids):
    y = centroids[idx]
    x = Pi.T @ y                        # inverse rotation
    return x

# Step 4: During attention — use QJL for inner products
k_quant = quant_turbo(k, Pi, centroids)
v_quant = quant_turbo(v, Pi, centroids)
# QJL handles ⟨query, k_quant⟩ and ⟨k_quant, v_quant⟩ lookups
```

### Architecture (from 0xSero/turboquant)

```
codebook.py       # Lloyd-Max optimal scalar quantizer for Beta distribution
codebook/         # Pre-generated codebook files (d=128/256, bits 2/3/4)
rotation.py       # Random orthogonal rotation + QJL projection matrices
quantizer.py      # TurboQuantMSE + TurboQuantProd (Algorithms 1 & 2)
kv_cache.py       # KV cache manager with value bit-packing
capture.py        # Modular KV capture hooks for attention layers
store.py          # Compressed KV store (quantize + append + flat cache)
score.py          # Attention scoring from compressed keys
integration/vllm.py  # vLLM adapter (monkey-patch, free_kv_cache, hybrid decode)
triton_kernels.py # 3 fused Triton kernels for decode attention
validate_paper.py # 9 tests validating Theorems 1-3
audit_claims.py   # Adversarial audit of all claims
```

---

## Framework Integration

| Framework | Status | Notes |
|-----------|--------|-------|
| **vLLM** | ✅ In v0.18.0 | `integration/vllm.py` adapter |
| **llama.cpp** | 🔄 Discussion [#20969](https://github.com/ggml-org/llama.cpp/discussions/20969) | Tracking integration |
| **MLX** | ✅ [sharpner/turboquant-mlx](https://github.com/sharpner/turboquant-mlx) | Apple Silicon, ~5.5× compression |

### vLLM Usage

```python
# Via 0xSero/turboquant integration
from turboquant.integration.vllm import TurboQuantVLLM

# Monkey-patch vLLM to use TurboQuant for KV cache
# No configuration changes needed — drops in as replacement
```

### Hugging Face / Transformers

```python
# PolarQuant idea applied to Hugging Face models
from turboquant import TurboQuantizer

quantizer = TurboQuantizer(
    bits=3,           # 3-bit for keys, 2-bit for values
    rotation="random"  # data-oblivious
)

# KV cache is automatically compressed
```

---

## Deployment Notes

### Hardware Guidance

| Hardware | Recommendation | Expected Speedup |
|----------|---------------|-----------------|
| H100 / A100 | Ideal for 4-bit mode | 8× attention |
| RTX 5090 | Dense model, 3-bit KV | 2× capacity |
| RTX 3090 | MoE with TP=8 | 1.45× context extension |
| Edge (phone) | Software-only, 3-bit | 32K+ context feasible |

### Mixed-Precision Strategy

> **TurboQuant for KV cache + INT4 for weights = maximum total compression**

This combination achieves the best memory/quality tradeoff for production deployments.

### Practical Risks & Mitigations

| Risk | Mitigation |
|------|------------|
| Random rotation overhead | Pre-generate and reuse matrices offline — don't rebuild online |
| Residual norm storage | One FP16 scalar per block — negligible overhead (~0.1%) |
| Random-seed handling | Must use fixed seeds for reproducibility; poor handling could introduce small bias in extreme edge cases |
| Value quantization quality | Use 4-bit for values if quality-sensitive; 2-bit is the bottleneck |

---

## Timeline & Current Status

| Status | Details |
|--------|---------|
| **Paper** | Published (ICLR 2026, arXiv:2504.19874) |
| **Code** | Open-source available ([0xSero/turboquant](https://github.com/0xSero/turboquant), GPL-3.0) |
| **vLLM** | Integrated in v0.18.0 |
| **llama.cpp** | Under discussion |
| **MLX** | Reproduction available |

> ⚠️ The GitHub project `cg94301/turboquant` is **unrelated** — it's a trading strategy, not this KV-cache algorithm.

---

## Relationship: TurboQuant vs PolarQuant vs QJL

| Paper | Venue | Role in TurboQuant |
|-------|-------|-------------------|
| **PolarQuant** | AISTATS 2026 | Main compression — random rotation + polar transform + Lloyd-Max quantization on angles |
| **QJL** | AAAI 2025 | Residual correction — 1-bit unbiased inner-product estimator |
| **TurboQuant** | ICLR 2026 | Combined framework = PolarQuant + QJL, plus full KV cache integration |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Origin** | Google Research, March 2026 |
| **Paper** | arXiv:2504.19874 (ICLR 2026) |
| **Key Innovation** | Online vector quantization approaching information-theoretic limits, data-oblivious, accelerator-friendly |
| **Core Components** | PolarQuant (compression) + QJL (residual correction) |
| **KV Cache Compression** | 3-bit per value with zero accuracy loss |
| **Memory Reduction** | 6×+ |
| **Attention Speedup** | 8× (H100, 4-bit mode) |
| **Training Required** | None |
| **Open Source** | [0xSero/turboquant](https://github.com/0xSero/turboquant) |
| **Framework Support** | vLLM 0.18.0+, MLX, llama.cpp (in progress) |
| **Best For** | Long-context LLM inference, edge deployment, vector search at scale |

---

*Last updated: 2026-04-19*
