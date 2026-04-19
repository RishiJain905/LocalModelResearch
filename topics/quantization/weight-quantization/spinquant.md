# SpinQuant: LLM Quantization with Learned Rotations

> **Paper:** [arXiv:2405.16406](https://arxiv.org/abs/2405.16406) — ICLR 2025  
> **Authors:** Zechun Liu, Changsheng Zhao, Igor Fedorov, Bilge Soran, Dhruv Choudhary, Raghuraman Krishnamoorthi, Vikas Chandra, Yuandong Tian, Tijmen Blankevoort (Meta FAIR)  
> **Official Repo:** [facebookresearch/SpinQuant](https://github.com/facebookresearch/SpinQuant)

---

## Overview

> *"SpinQuant narrows the accuracy gap of W4A4KV4 quantization with full precision to merely **2.9 points** for the LLaMA-2 7B model on zero-shot reasoning tasks, surpassing LLM-QAT by 19.1 points and SmoothQuant by 25.0 points."*

> *"We find that some random rotations lead to much better quantization than others, with an up to **13 points difference** in downstream zero-shot reasoning performance."*

---

## Original Paper

| | |
|---|---|
| **Title** | SpinQuant: LLM Quantization with Learned Rotations |
| **arXiv** | [https://arxiv.org/abs/2405.16406](https://arxiv.org/abs/2405.16406) |
| **Venue** | ICLR 2025 |
| **PDF** | [https://arxiv.org/pdf/2405.16406](https://arxiv.org/pdf/2405.16406) |
| **OpenReview** | [https://openreview.net/forum?id=ogO6DGE6FZ](https://openreview.net/forum?id=ogO6DGE6FZ) |

---

## GitHub Repository

| | |
|---|---|
| **Repo** | [https://github.com/facebookresearch/SpinQuant](https://github.com/facebookresearch/SpinQuant) |
| **License** | CC-BY-NC 4.0 |
| **Stars** | 387+ | **Forks** | 82+ |

---

## How It Works

### Core Problem: Activation Outliers

LLM activations contain **extreme outlier values** that stretch quantization ranges, reducing effective precision for the majority of values.

**Key observations:**
- Random rotations reduce outliers (kurtosis drops from >200 to ~3, Gaussian-like)
- Different random rotations produce **up to 13 points difference** in zero-shot reasoning accuracy
- **Learned rotations** outperform random rotations significantly

### Key Innovation: 4 Rotationally Invariant Points

SpinQuant identifies **4 rotationally invariant points** in Transformer architectures where rotations don't alter full-precision output:

| Rotation | Location | Shape | Purpose |
|----------|----------|-------|---------|
| **R1** | Residual stream (D_token × D_token) | Full matrix | Reduces activation outliers |
| **R2** | Attention block per layer (D_head × D_head) | Per-head | Reduces V-cache/out-projection outliers |
| **R3** | Online Hadamard after softmax | Online | KV-cache outlier suppression |
| **R4** | Online Hadamard inside MLP | Online | Down-projection activation outlier suppression |

### Two Variants

| Variant | Rotations | Forward Pass Mod | Kernel Support | Notes |
|---------|-----------|-----------------|----------------|-------|
| **SpinQuant_no_had** | R1 + R2 | Fused into weights | ✅ No kernel needed | Simpler, less aggressive |
| **SpinQuant_had** | R1 + R2 + R3 + R4 | Online Hadamard | ❌ Kernel needed | For extreme W4A4KV4 quantization |

### Cayley Optimization on Stiefel Manifold

SpinQuant uses **Cayley SGD** to learn orthonormal rotation matrices R1, R2:

```
arg min_{R ∈ M} L_Q(R1, R2 | W, X)
where M = Stiefel manifold
```

**Update rule:**
```
R' = ΔR(Y)R := (I - α/2 · Y)^{-1}(I + α/2 · Y)R
where Y = Ĝ - Ĝ^T
```

- Guarantees orthonormality (R'^T R' = I) without costly projections
- **Training setup:** 800 WikiText-2 samples, 100 iterations, LR 1.5 with linear decay
- **Optimization time:** ~13-30 min for 1B-8B models, ~3.5 hrs for 70B

### Outlier Suppression Mechanism

> *"Rotations 'smear' activation outliers across dimensions, making distributions more uniform. This reduces dominance of outliers, allowing low-bit formats to represent entire vectors more accurately."*

**SNR Improvement:**
| Configuration | SNR |
|---------------|-----|
| No rotation | -2.9 dB |
| Random rotation | +0.9 dB (+3.8 dB improvement) |
| Learned rotation | **+6.8 dB** (+5.9 dB additional) |

---

## Performance Benchmarks

### W4A4KV4 Results (Zero-Shot / WikiText2 Perplexity)

| Model | FP | SpinQuant | QuaRot | LLM-QAT | SmoothQuant |
|-------|-----|-----------|--------|---------|------------|
| LLaMA-2 7B | 66.9 | **64.0** | 62.5 | 44.9 | 39.0 |
| LLaMA-2 13B | 68.3 | **66.9** | 66.2 | — | 40.5 |
| LLaMA-2 70B | 72.9 | **71.2** | 70.3 | — | 55.9 |
| LLaMA-3 8B | 69.6 | **65.2** | 63.3 | 43.2 | 38.7 |
| LLaMA-3 70B | 74.5 | **69.3** | 65.1 | — | 52.4 |

> *"For LLaMA-3 8B models that are hard to quantize, SpinQuant reduces the gap to full precision by up to **45.1%** relative to QuaRot."*

### Learned vs Random Rotation

| Model | Metric | Random Hadamard | SpinQuant (Learned) | Gain |
|-------|--------|----------------|---------------------|------|
| Mistral-7B | W4A4 accuracy | 52.4 | **68.6** | +16.2 pts |
| LLaMA-3 8B | W4A4 accuracy | 63.9 | **65.5** | +1.6 pts |

### Comparison with QuaRot

| Aspect | SpinQuant | QuaRot |
|--------|-----------|--------|
| Online Hadamard per block | **2** | 4 |
| LLaMA-3 8B W4A4 gap to FP | **~4.4 pts** | ~6.3 pts |
| Relative improvement vs QuaRot | — | **45.1%** narrower gap |

### SNR Results

| Method | SNR (dB) | Notes |
|--------|----------|-------|
| No rotation | -2.9 | Baseline |
| Random rotation | +0.9 | +3.8 dB |
| **Learned rotation** | **+6.8** | +9.7 dB total |

---

## Practical Usage

### Workflow

```bash
# Step 1: Optimize rotation matrix
bash scripts/10_optimize_rotation.sh $model_name $w_bit $a_bit $kv_bit

# For 70B models (FSDP)
bash scripts/11_optimize_rotation_fsdp.sh $model_name

# Step 2: Evaluate PTQ with optimized rotation
bash scripts/2_eval_ptq.sh $model_name $w_bit $a_bit $kv_bit \
    --optimized_rotation_path <path>
```

### Key Arguments

| Argument | Description |
|----------|-------------|
| `--w_bits` | Weight bit width |
| `--a_bits` | Activation bit width |
| `--k_bits` | Key cache bit width |
| `--v_bits` | Value cache bit width |
| `--w_clip` | Enable weight clipping |
| `--a_asym`, `--k_asym`, `--v_asym` | Asymmetric quantization |
| `--optimized_rotation_path` | Path to pre-trained rotation matrices |

### Pre-trained Rotation Matrices

Download from: [Google Drive folder](https://drive.google.com/drive/folders/1R2zix4qeXBjcmgnJN1rny93cguJ4rEE8?usp=sharing)

### ExecuTorch Export

```bash
# Optimize rotation for ExecuTorch
bash scripts/31_optimize_rotation_executorch.sh $model_name

# Evaluate PTQ with ExecuTorch
bash scripts/32_eval_ptq_executorch.sh $model_name
```

**Colab notebook:** [https://colab.research.google.com/gist/zxdmike/abbb2c9b0d1fd1f4ed8cdae8c02180f4](https://colab.research.google.com/gist/zxdmike/abbb2c9b0d1fd1f4ed8cdae8c02180f4)

### On-Device Deployment (ExecuTorch)

**Pre-quantized models on HuggingFace:**
- [executorch-community/Llama-3.2-1B-Instruct-SpinQuant_INT4_EO8-ET](https://huggingface.co/executorch-community/Llama-3.2-1B-Instruct-SpinQuant_INT4_EO8-ET)
- [executorch-community/Llama-3.2-3B-Instruct-SpinQuant_INT4_EO8-ET](https://huggingface.co/executorch-community/Llama-3.2-3B-Instruct-SpinQuant_INT4_EO8-ET)

**Android benchmarks:**
| Metric | Improvement |
|--------|-------------|
| Decode speed | **2.6× faster** |
| Prefill speed | **4.3× faster** |
| Model size | **54% smaller** |

vs BF16 baseline.

---

## Framework Support

| Framework | Support | Notes |
|-----------|---------|-------|
| **ExecuTorch** | ✅ Native | Mobile/edge deployment |
| **AMD Quark** | ✅ Docs | [Rotation pre-processing](https://quark.docs.amd.com/latest/pytorch/tutorial_rotation.html) |
| **vLLM** | Planned | Not yet integrated |
| **llama.cpp** | ❌ | Not supported |

---

## Comparison with Other Methods

| Method | Approach | W4A4KV4 Quality | Notes |
|--------|----------|-----------------|-------|
| **SpinQuant** | Learned rotations | **Best** (2.9 pt gap) | ICLR 2025 |
| **QuaRot** | Random Hadamard | Good (4.4 pt gap on LLaMA-3 8B) | Concurrent work |
| **LLM-QAT** | Quantization-aware training | 19.1 pts behind SpinQuant | Requires training |
| **SmoothQuant** | Per-channel smoothing | 25.0 pts behind SpinQuant | Columbia approach |

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **ICLR 2025 Poster** | [https://iclr.cc/virtual/2025/poster/28338](https://iclr.cc/virtual/2025/poster/28338) |
| **OpenReview** | [https://openreview.net/forum?id=ogO6DGE6FZ](https://openreview.net/forum?id=ogO6DGE6FZ) |
| **Literature Review** | [Moonlight Review](https://www.themoonlight.io/en/review/spinquant-llm-quantization-with-learned-rotations) |
| **YouTube** | [Simple explanation](https://www.youtube.com/watch?v=qCMNjJ535n8) |
| **AMD Quark docs** | [Rotation pre-processing](https://quark.docs.amd.com/latest/pytorch/tutorial_rotation.html) |

---

## Summary

| Aspect | Details |
|--------|---------|
| **Paper** | [arXiv:2405.16406](https://arxiv.org/abs/2405.16406) |
| **Venue** | ICLR 2025 |
| **Key Innovation** | Learned rotation matrices at 4 rotationally invariant points |
| **Best Result** | W4A4KV4: only 2.9 pt gap to FP16 (vs 25 pt gap for SmoothQuant) |
| **Optimization Time** | ~13-30 min for 1B-8B, ~3.5 hrs for 70B |
| **Use Case** | Extreme low-bit quantization (W4A4KV4), mobile/edge via ExecuTorch |

---

*Last updated: 2026-04-19*
