# AutoRound

> **Original Paper:** [arXiv:2309.05516](https://arxiv.org/abs/2309.05516) (EMNLP 2024 Findings)  
> **Follow-up:** [SignRound V2: arXiv:2512.04746](https://arxiv.org/abs/2512.04746)  
> **Authors:** Intel Research  
> **Official Repo:** [intel/auto-round](https://github.com/intel/auto-round)

---

## Overview

> *"AutoRound is an advanced quantization toolkit designed for Large Language Models (LLMs) and Vision-Language Models (VLMs). It achieves high accuracy at ultra-low bit widths (2–4 bits) with minimal tuning by leveraging sign-gradient descent and providing broad hardware compatibility."*

---

## Original Paper

| | |
|---|---|
| **Title** | Optimize Weight Rounding via Signed Gradient Descent for the Quantization of LLMs |
| **Venue** | EMNLP 2024 Findings |
| **arXiv** | [https://arxiv.org/abs/2309.05516](https://arxiv.org/abs/2309.05516) |
| **Follow-up Paper** | [SignRound V2: Closing the Performance Gap in Extremely Low-Bit PTQ](https://arxiv.org/abs/2512.04746) |

---

## GitHub Repository

| | |
|---|---|
| **Repo** | [https://github.com/intel/auto-round](https://github.com/intel/auto-round) |
| **License** | Custom |

---

## How It Works: SignSGD Rounding

AutoRound uses **Signed Gradient Descent (SignSGD)** to optimize weight rounding and clipping ranges — **no training required**.

### Three Trainable Parameters per Quantized Tensor

| Parameter | Purpose |
|-----------|---------|
| **V** | Rounding offset/adjustment |
| **α** | Learned clipping range control |
| **β** | Learned clipping range control |

### Quantization Formulation

```python
# With trainable V
W̃ = s * clip(⌊W/s + zp + V⌉, n, m)

# With trainable clipping (α, β)
s = (max(W)*α - min(W)*β) / (2^bit - 1)
```

### Block-wise Output Reconstruction

```python
min_{α,β,V} ||WX - W̃X||_F^2
```

### Why SignSGD Over Adam?

1. **Large solution regions** — only threshold matters, gradient magnitude doesn't need convergence
2. **Bounded solution space** — V ∈ [-0.5, 0.5], α/β ∈ [0,1]
3. **Intuitive step sizing** — lr=5e-3, 200 steps → max adjustment = 0.5
4. **Lightweight** — fewer memory/compute resources than Adam

> *"SignRound achieved absolute average accuracy improvements ranging from **6.91% to 33.22%** at 2 bits, as measured by the average zero-shot accuracy across 11 tasks."*

### No Training Required

- Uses **block-wise output reconstruction** as the training objective
- Processes decoder layers sequentially with calibration data (128–512 samples)
- Only **200 steps** for convergence (light mode: 50 steps)
- Quantizing a 72B model takes **37 minutes on an A100 GPU** in light mode

---

## Performance Benchmarks

### Win Rates vs Other Methods (11-task average accuracy)

| vs. Method | Win Rate |
|------------|----------|
| GPTQ | **30/32** |
| AWQ | **27/32** |
| OmniQuant | **29/32** |

### 2-Bit Results (W2G128) — Largest Gains

| Method | Mistral-7B | V2-7B | V2-13B | V2-70B |
|--------|------------|-------|--------|--------|
| RTN | 30.52 | 29.94 | 33.51 | 38.14 |
| GPTQ | 39.61 | 35.37 | 42.46 | 28.47 |
| AWQ | 30.06 | 30.10 | 32.16 | 32.23 |
| OmniQuant | 32.17 | 40.74 | 46.55 | 51.31 |
| **SignRound** | **52.71** | **48.64** | **53.46** | **61.69** |

### INT4 (W4G128) Average Accuracy

| Model | 16-bit | AutoRound | GPTQ | AWQ |
|-------|--------|-----------|------|-----|
| Mistral-7B | 63.30 | **62.87** | 62.45 | 62.48 |
| V2-7B | 57.98 | **57.97** | 57.28 | 57.52 |
| V2-13B | 61.42 | 60.90 | 60.97 | 61.03 |
| V2-70B | 66.12 | **66.41** | 66.21 | 66.28 |

> *"At INT2 precision, it outperforms popular baselines by up to **2.1× higher in relative accuracy**."* — [HuggingFace Blog](https://huggingface.co/blog/autoround)

---

## Supported Devices & Backends

| Device | Support |
|--------|---------|
| **CPU** | ✅ Intel Xeon, Intel Arc |
| **Intel GPU** | ✅ Gaudi AI accelerators, Intel Data Center GPUs, Intel Arc B-Series |
| **CUDA** | ✅ NVIDIA GPUs |
| **HPU** | ✅ Habana |

### Inference Frameworks

| Framework | Support |
|-----------|---------|
| **vLLM** | ✅ Via `compressed-tensors` |
| **SGLang** | ✅ Native (v0.5.4.post2+) |
| **Transformers** | ✅ Via auto-round format |

### Export Formats

| Format | Bits | Best For |
|--------|------|----------|
| **auto_round** | 2,3,4,8 | CPU, Intel GPU, CUDA, HPU; mixed-precision |
| **gguf** | q*_k variants | CPU |
| **auto_gptq** | 2,3,4,8 | CUDA (symmetric) |
| **auto_awq** | 4 only | CUDA (asymmetric 4-bit) |
| **llm_compressor** | NVFP4, MXFP4, MXFP8 | vLLM |

### Quantization Schemes

- W4A16, W2A16, W3A16, W8A16
- MXFP4, MXFP8, NVFP4, FP8
- Mixed-bit (per-layer precision search)
- LM-head quantization

### Supported Models

**LLMs:** Qwen, LLaMA, DeepSeek, Mistral, Gemma, Falcon, Phi, Mixtral, GLM, ChatGLM

**VLMs:** 10+ vision-language models including Mistral-Small-3.1, Gemma3, Qwen2-VL

---

## Usage

### Python API

```python
from auto_round import AutoRound

autoround = AutoRound("meta-llama/Llama-3.2-1B-Instruct", scheme="W4A16")
autoround.quantize_and_save("./output", format="auto_round")
```

### CLI

```bash
auto-round \
    --model Qwen/Qwen2-VL-2B-Instruct \
    --bits 4 \
    --group_size 128 \
    --format "auto_round" \
    --output_dir ./tmp_autoround
```

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **Hugging Face Blog** | [https://huggingface.co/blog/autoround](https://huggingface.co/blog/autoround) |
| **LMSYS Blog** | [https://lmsys.org/blog/2025-11-13-AutoRound/](https://lmsys.org/blog/2025-11-13-AutoRound/) |
| **vLLM/LLM Compressor Docs** | [https://docs.vllm.ai/projects/llm-compressor/en/latest/examples/autoround/](https://docs.vllm.ai/projects/llm-compressor/en/latest/examples/autoround/) |
| **Red Hat Developer Blog** | [https://developers.redhat.com/articles/2025/12/09/advancing-low-bit-quantization-llms-autoround-x-llm-compressor](https://developers.redhat.com/articles/2025/12/09/advancing-low-bit-quantization-llms-autoround-x-llm-compressor) |
| **Intel Analytics Blog** | [https://medium.com/intel-analytics-software/accelerating-vllm-and-sglang-deployment-using-autoround-45fdc0b2683e](https://medium.com/intel-analytics-software/accelerating-vllm-and-sglang-deployment-using-autoround-45fdc0b2683e) |

---

*Last updated: 2026-04-19*
