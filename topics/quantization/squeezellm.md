# SqueezeLLM: Dense-and-Sparse Quantization

> **Original Paper:** [arXiv:2306.07629](https://arxiv.org/abs/2306.07629) — ICML 2024  
> **Official Repo:** [SqueezeAILab/SqueezeLLM](https://github.com/SqueezeAILab/SqueezeLLM)

---

## Overview

> *"SqueezeLLM is a post-training quantization framework that incorporates a new method called Dense-and-Sparse Quantization to enable efficient LLM serving."*

> *"The Squeeze variant of the Vicuna models can be served within 6 GB of memory and reach 2% higher MMLU than the baseline model in FP16 with an even 2× larger memory footprint."*

---

## Original Paper

| | |
|---|---|
| **Title** | SqueezeLLM: Dense-and-Sparse Quantization |
| **arXiv** | [https://arxiv.org/abs/2306.07629](https://arxiv.org/abs/2306.07629) |
| **Conference** | ICML 2024 |
| **Full HTML** | [ar5iv.labs.arxiv.org/html/2306.07629](https://ar5iv.labs.arxiv.org/html/2306.07629) |

---

## GitHub Repository

| | |
|---|---|
| **Repo** | [https://github.com/SqueezeAILab/SqueezeLLM](https://github.com/SqueezeAILab/SqueezeLLM) |
| **License** | MIT |
| **Gradient Repo** | [SqueezeLLM-gradients](https://github.com/kssteven418/SqueezeLLM-gradients) — Fisher sensitivity score calculation |

---

## How It Works

### Core Insight: Memory-Bound Inference

> *"The main bottleneck for generative inference with LLMs is **memory bandwidth, rather than compute**, specifically for single batch inference."*

- Generative LLM inference has **extremely low arithmetic intensity** (matrix-vector ops)
- Each weight load processes only a single token — no amortization across batches
- Example: A5000 GPU has **222 TFLOPS compute** vs **768 GB/s memory bandwidth** — a **290× disparity**
- Latency decreases **linearly** as bit precision is reduced → confirms memory-bound bottleneck

### Technique 1: Sensitivity-Based Non-Uniform Quantization

**Why non-uniform?** Two problems with uniform quantization:
1. LLM weight distributions are **clearly non-uniform**
2. Integer arithmetic advantage of uniform quantization doesn't help memory-bound inference (arithmetic is FP16 anyway)

**Method:** Weighted k-means clustering using second-order (Hessian) information:

```python
# Standard k-means objective
Q(w)* = arg min_Q ‖W − WQ‖²₂

# Sensitivity-weighted objective (via Taylor expansion)
Q(w)* = arg min_Q (W − WQ)ᵀH(W − WQ)

# Approximated using Fisher information matrix (diagonal approximation)
Q(w)* ≃ arg min_Q Σᵢ Fᵢᵢ(wᵢ − Q(wᵢ))²
```

- **Fisher info** computed from gradients over 100 random samples
- Centroids are **pulled closer to sensitive weight values**
- **Result:** LLaMA-7B 3-bit perplexity on C4 drops from **28.26** (RTN uniform) → **7.75**

### Technique 2: Dense-and-Sparse Decomposition

**Observation:** ~99.9% of weights concentrated in ~10% of the full range. Outliers force large quantization ranges.

**Method:** Decompose W = D + S
- **D (Dense):** Weights within thresholds [T_min, T_max] → quantized normally
- **S (Sparse):** Outlier values + highly sensitive values → stored in FP16 via CSR format

**Sparsity configuration:** 0.45% total = 0.05% sensitive + 0.4% outlier values

### Kernel: Lookup-Based Inference

> *"Dense kernel: LUT-based 3/4-bit CUDA kernels for matrix-vector multiplication. Compressed weights store 3/4-bit indices → LUT entries contain FP16 centroid values. Dequantize piece-by-piece to minimize memory bandwidth. All arithmetic in FP16."*

> *"Sparse kernel: CSR-format sparse matrix-vector multiplication with balanced hybrid kernel — assigns equal nonzeros per thread (~10 nonzeros/thread) rather than one thread per row."*

---

## Performance Benchmarks

### Perplexity Comparison (LLaMA Models, C4 Dataset)

| Model | Method | Avg Bits | C4 PPL | WikiText2 PPL | Speedup | Mem (GB) |
|-------|--------|----------|--------|---------------|---------|----------|
| **7B** | FP16 Baseline | 16 | 7.08 | 5.68 | 1× | 12.7 |
| | GPTQ (3-bit) | 3 | 9.55 | 7.55 | 2.3× | 2.9 |
| | **SqueezeLLM dense (3-bit)** | **3.02** | **7.75** | **6.32** | **2.1×** | **2.9** |
| | GPTQ g128 (3-bit) | 3.24 | 7.89 | 6.27 | **0.2×** | 3.0 |
| | AWQ g128 (3-bit) | 3.24 | 7.90 | 6.44 | 2.0× | 3.0 |
| | **SqueezeLLM 0.45% (3-bit)** | **3.24** | **7.56** | **6.13** | **1.9×** | **3.1** |
| | **SqueezeLLM 0.45% (4-bit)** | **4.27** | **7.18** | **5.77** | **1.7×** | **4.0** |
| **13B** | **SqueezeLLM 0.45% (3-bit)** | **3.24** | **6.92** | **5.45** | **2.2×** | **5.8** |
| **30B** | **SqueezeLLM 0.45% (3-bit)** | **3.25** | **6.23** | **4.44** | — | — |
| **65B** | **SqueezeLLM 0.45% (3-bit)** | **3.24** | **5.84** | **3.88** | — | **28.0** |

*Source: [ar5iv paper, Table 1](https://ar5iv.labs.arxiv.org/html/2306.07629)*

### Key Findings

> *"In the case of 3-bit quantization, SqueezeLLM outperforms both GPTQ and AWQ with comparable model sizes."*

> *"3-bit SqueezeLLM reduces perplexity gap from FP16 by up to **2.1×** vs SOTA methods at same memory"*

### Critical GPTQ Grouping Latency Issue

> *"GPTQ with activation ordering + grouping (g128) causes severe latency degradation: LLaMA-7B 3-bit: **0.2× speedup** (effectively slower than FP16). LLaMA-65B 3-bit: **117.8s** latency vs. SqLLM's **8.8s** (13× slower)."*

**Cause:** Permutation causes elements in same channel to need different scaling factors → distributed memory accesses → poor GPU memory coalescing

### Instruction-Following (Vicuna, MMLU Zero-Shot)

| Model | Method | MMLU |
|-------|--------|------|
| Vicuna-7B | FP16 | 39.1% |
| Vicuna-7B | SqueezeLLM 4-bit (0.45%) | **39.4%** |
| Vicuna-13B | SqueezeLLM 4-bit (0.45%) | **2× smaller memory, 2% higher accuracy** |

---

## Practical Usage

### Installation

```bash
conda create --name sqllm python=3.9 -y
conda activate sqllm
git clone https://github.com/SqueezeAILab/SqueezeLLM
cd SqueezeLLM
pip install -e .
cd squeezellm
python setup_cuda.py install
```

### Quantization Steps

**Step 1: Calculate Fisher Sensitivity Scores**
```bash
git clone https://github.com/kssteven418/SqueezeLLM-gradients.git
cd SqueezeLLM-gradients && pip install -e . && pip install -r requirements.txt
pip install accelerate
CUDA_VISIBLE_DEVICES=0 python SqueezeLLM-gradients/run.py \
    --output_dir llama-gradients --model_name meta-llama/Llama-2-7b-hf
```

**Step 2: Chunk Models (Layer-Level Splitting)**
```bash
python SqueezeLLM/quantization/chunk_models.py \
    --model meta-llama/Llama-2-7b-hf --output llama-chunks --model_type llama
python SqueezeLLM/quantization/chunk_models.py \
    --model llama-gradients --output llama-gradients-chunks --model_type llama
```

**Step 3: Find Outlier Parameters**
```bash
python SqueezeLLM/quantization/generate_outlier_config.py \
    --model llama-chunks --range 1.8 --output llama-chunks-outlierconfig/
```

**Step 4: K-Means Clustering for Non-Uniform Quantization**
```bash
python SqueezeLLM/quantization/nuq.py \
    --bit 4 --model_type llama --model llama-chunks \
    --gradient llama-gradients-chunks --output llama-LUT-4bit/ \
    --outlier_config llama-chunks-outlierconfig/outlier_config_o0.53.json \
    --sensitivity 0.05
```

**Step 5: Rebuild Quantized Model**
```bash
python SqueezeLLM/quantization/pack.py \
    --model meta-llama/Llama-2-7b-hf --wbits 4 --folder llama-LUT-4bit/ \
    --save llama-squeezed --include_sparse --balance
```

### Running Quantized Models

**Benchmarking (C4 dataset):**
```bash
CUDA_VISIBLE_DEVICES=0 python llama.py {model_path} c4 \
    --wbits 3 --load sq-llama-7b-w3-s0.pt --benchmark 128 --check --torch_profile
```

**Perplexity Evaluation:**
```bash
CUDA_VISIBLE_DEVICES=0 python llama.py {model_path} c4 \
    --wbits 3 --load sq-llama-7b-w3-s0.pt --eval
```

### Pre-Quantized Models (HuggingFace)

Available under `squeeze-ai-lab/` organization:

| Model | Bits | Sparsity | HuggingFace |
|-------|------|----------|-------------|
| sq-llama-2-7b-w4-s0 | 4 | 0% | [link](https://huggingface.co/squeeze-ai-lab/sq-llama-2-7b-w4-s0) |
| sq-llama-2-13b-w4-s0 | 4 | 0% | [link](https://huggingface.co/squeeze-ai-lab/sq-llama-2-13b-w4-s0) |
| sq-mistral-7b-w4-s0 | 4 | 0% | [link](https://huggingface.co/squeeze-ai-lab/sq-mistral-7b-w4-s0) |
| sq-vicuna-7b-w3-s45 | 3 | 0.45% | [link](https://huggingface.co/squeeze-ai-lab/sq-vicuna-7b-w3-s45) |

---

## vLLM Integration

> *"SqueezeLLM now supported in official vLLM framework."*

**Note:** Tokens per second is significantly slower than AWQ and GPTQ when using vLLM.

---

## Supported Models

| Family | Models | Bits | Sparsity Options |
|--------|--------|------|------------------|
| **LLaMA v1** | 7B, 13B, 30B, 65B | 3, 4 | 0%, 0.05%, 0.45% |
| **LLaMA-2** | 7B, 13B | 3, 4 | 0% (dense only) |
| **Mistral** | 7B, 7B-instruct | 3, 4 | 0% (dense only) |
| **Vicuna v1.1** | 7B, 13B | 3, 4 | 0%, 0.45% |
| **Vicuna v1.3** | 7B, 13B | 3, 4 | 0% (dense only) |
| **XGen** | 7B-8k-Base, 7B-8k-Inst | 3, 4 | 0%, 0.45% |
| **OPT** | 1.3B, 2.7B, 6.7B, 13B, 30B | 3, 4 | 0%, 0.50% |

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **Picovoice Blog** | [Understanding SqueezeLLM Compression](https://picovoice.ai/blog/understanding-squeezellm/) |
| **Intel Developer Case Study** | [Enable Efficient LLM Inference with SqueezeLLM](https://www.intel.com/content/www/us/en/developer/articles/case-study/ucberkeley-optimizes-llm-inference-with-squeezellm.html) |
| **The Kaitchup** | [SqueezeLLM: Better 3-bit and 4-bit Quantization for LLMs](https://kaitchup.substack.com/p/squeezellm-better-3-bit-and-4-bit) |
| **LinkedIn (Author)** | [Amir Gholami's Announcement](https://www.linkedin.com/posts/a-gholami_introducing-squeezellm-a-post-training-quantization-activity-7074927641220329472-kVgR) |

---

## Limitations

> *"Hugging Face Transformers does NOT work — causes OOM errors when casting weights."*

**Workarounds:**
1. Adapt code from `llama.py` (benchmarking file in repo)
2. Use **vLLM** (officially supported)

---

*Last updated: 2026-04-19*
