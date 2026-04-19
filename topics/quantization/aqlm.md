# AQLM: Additive Quantization of Language Models

> **Original Paper:** [arXiv:2401.06118](https://arxiv.org/abs/2401.06118) — ICML 2024  
> **Authors:** Vage Egiazarian, Andrei Panferov, Denis Kuznedelev, Elias Frantar, Artem Babenko, Dan Alistarh  
> **Official Repo:** [Vahe1994/AQLM](https://github.com/Vahe1994/AQLM)

---

## Overview

> *"AQLM is a weight-only post-training quantization (PTQ) algorithm that sets a new state-of-the-art for the 2 bit-per-parameter range. AQLM represents groups of 8-16 weights as a sum of multiple vector codes."*

> *"AQLM is the first scheme that is Pareto optimal in terms of accuracy-vs-model-size when compressing to less than 3 bits per parameter, and significantly improves upon all known schemes in the extreme compression (2bit) regime."*

---

## Original Paper

| | |
|---|---|
| **Title** | Extreme Compression of Large Language Models via Additive Quantization |
| **Venue** | ICML 2024 |
| **arXiv** | [https://arxiv.org/abs/2401.06118](https://arxiv.org/abs/2401.06118) |
| **PDF** | [https://arxiv.org/pdf/2401.06118.pdf](https://arxiv.org/pdf/2401.06118.pdf) |
| **Follow-up** | PV-Tuning: Beyond Straight-Through Estimation — NeurIPS 2024 (Oral) |

---

## GitHub Repository

| | |
|---|---|
| **Repo** | [https://github.com/Vahe1994/AQLM](https://github.com/Vahe1994/AQLM) |
| **Latest Release** | v1.1.7 (April 2025) |

> *"Official PyTorch implementation for Extreme Compression of Large Language Models via Additive Quantization and PV-Tuning: Beyond Straight-Through Estimation for Extreme LLM Compression"*

---

## How AQLM Works: Multi-Codebook Additive Quantization

### Core Concept

AQLM extends **Additive Quantization (AQ)** — a classic Multi-Codebook Quantization (MCQ) method from information retrieval — to LLM weight compression.

### Weight Representation

1. Weight matrix rows are split into groups of **g consecutive elements** (typically 8-16)
2. Each group is represented as a **sum of M vectors** from M learned codebooks C₁,...,Cₘ
3. Encoding: each group uses **M·B bits** (codes) + codebook storage

### Algorithm: Three Phases

| Phase | Description | Method |
|-------|-------------|--------|
| **1: Beam Search** | Update discrete codes to minimize MSE | Reformulated as MAP inference in MRF; beam search |
| **2: Codebook Update** | Find optimal codebook vectors | Adam optimizer (100 steps, lr=10⁻⁴) |
| **3: Block Fine-tuning** | Joint optimization across transformer block | Fine-tune codebooks, scales, and non-quantized params while keeping codes frozen |

### Notation Scheme

Models use naming like `2Bit-1x16` or `2Bit-2x8`:

| Notation | Meaning | Accuracy | Speed |
|----------|---------|----------|-------|
| `1x16` | 1 codebook, 16-bit codes | Best | Slowest |
| `2x8` | 2 codebooks, 8-bit codes each | Good | ~2-3× faster |

---

## Performance Benchmarks

### Llama 2 Results at 2-bit Range

| Model | Method | Avg bits | Wiki2 PPL ↓ | C4 PPL ↓ | Avg Accuracy ↑ |
|-------|--------|----------|-------------|----------|----------------|
| 7B | FP16 | 16 | 5.12 | 6.63 | 62.35 |
| 7B | **AQLM** | **2.02** | **6.59** | **8.54** | **57.28** |
| 7B | QuIP# | 2.02 | 8.22 | 11.01 | 52.23 |
| 13B | **AQLM** | **1.97** | **5.60** | **7.49** | **61.32** |
| 13B | QuIP# | 2.01 | 6.06 | 8.07 | 57.55 |
| 70B | **AQLM** | **2.07** | **3.94** | **5.72** | **68.75** |
| 70B | QuIP# | 2.01 | 4.16 | 6.01 | 67.67 |

### Llama 2 Results at 3-bit Range

| Model | Method | Avg bits | Wiki2 PPL ↓ | C4 PPL ↓ | Avg Accuracy ↑ |
|-------|--------|----------|-------------|----------|----------------|
| 7B | **AQLM** | **3.04** | **5.46** | **7.08** | **60.88** |
| 7B | GPTQ | 3.00 | 8.06 | 10.61 | 53.08 |
| 7B | SpQR | 2.98 | 6.20 | 8.20 | 59.07 |
| 70B | **AQLM** | **3.01** | **3.36** | **5.17** | **69.86** |
| 70B | GPTQ | 3.00 | 4.40 | 6.26 | 65.41 |

> **Key finding:** AQLM at ~2.76 bits on 13B outperforms uncompressed 7B model.

### Inference Speed (GPU Nvidia RTX 3090)

| Config | 7B | 13B | 70B |
|--------|-----|------|------|
| FP16 baseline | 129 µs | 190 µs | 578 µs |
| AQLM (1×16-bit) | **1.31×** | **1.20×** | **1.20×** |
| AQLM (2×8-bit) | **1.57×** | **1.82×** | **3.05×** |

### CPU Speedup (Intel i9)

| Config | 7B | 13B | 70B |
|--------|-----|------|------|
| AQLM (4×8-bit) | **2.55×** | **3.02×** | **4.07×** |

---

## Framework Support

### Installation

```bash
pip install aqlm[gpu]   # GPU support
pip install aqlm[cpu]   # CPU support  
pip install aqlm[gpu,cpu]  # Both
```

**Requirements:** Python 3.10+

### Kernel Configurations

| Kernel | Codebooks | Bits | Notation | Fast GPU | Fast CPU |
|--------|-----------|------|----------|----------|----------|
| Triton | K | N | KxN | ✅ | ❌ |
| CUDA | 1 | 16 | 1x16 | ✅ | ❌ |
| CUDA | 2 | 8 | 2x8 | ✅ | ❌ |
| Numba | K | 8 | Kx8 | ❌ | ✅ |

---

## Usage

### Loading Pre-Quantized Models

```python
from transformers import AutoModelForCausalLM

# GPU inference
model = AutoModelForCausalLM.from_pretrained(
    "ISTA-DASLab/Llama-2-7b-AQLM-2Bit-1x16-hf",
    torch_dtype=torch.float16,
    device_map="auto"
)

# CPU inference
model = AutoModelForCausalLM.from_pretrained(
    "ISTA-DASLab/Llama-2-7b-AQLM-2Bit-1x16-hf",
    torch_dtype=torch.float32
)
```

### Quantizing Your Own Model

```bash
# Clone and setup
git clone https://github.com/Vahe1994/AQLM.git
cd AQLM
pip install -e .

# Quantize
python quantize.py meta-llama/Llama-2-7b-hf \
    ./path/to/saved/quantization \
    --num_codebooks 1 --nbits_per_codebook 16

# Convert to HuggingFace format
python convert_to_hf.py meta-llama/Llama-2-7b-hf \
    ./path/to/saved/quantization \
    ./converted-llama2-7b-hf --save_safetensors
```

### vLLM Serving

```python
from vllm import LLM

llm = LLM(model="ISTA-DASLab/Llama-2-7b-AQLM-2Bit-1x16-hf")
```

---

## Pre-Quantized Models on HuggingFace

Collection: [ISTA-DASLab](https://huggingface.co/ISTA-DASLab)

| Model | Scheme | WikiText-2 PPL | MMLU (5-shot) FP16→AQLM | Size (GB) |
|-------|--------|---------------|--------------------------|-----------|
| Llama-3-8b | 1x16 | — | 0.65→0.56 | 4.1 |
| Llama-3-70b | 1x16 | — | 0.79→0.75 | 21.9 |
| Mistral-7b | 1x16 | 5.40 | — | 2.5 |
| Mixtral-8x7b | 1x16 | 3.35 | — | 12.6 |
| Llama-2-7b | 1x16 | 5.92 | 0.46→0.39 | 2.4 |
| Llama-2-13b | 1x16 | 5.22 | 0.55→0.49 | 4.1 |
| Llama-2-70b | 1x16 | 3.83 | 0.69→0.65 | 18.8 |

---

## Colab Demo Notebooks

| Feature | Link |
|---------|------|
| Basic generation | [Colab](https://colab.research.google.com/github/Vahe1994/AQLM/blob/main/notebooks/colab_example.ipynb) |
| Streaming GPU/CPU | [Colab](https://colab.research.google.com/github/Vahe1994/AQLM/blob/main/notebooks/streaming_example.ipynb) |
| CUDA graphs (3× speedup) | [Colab](https://colab.research.google.com/github/Vahe1994/AQLM/blob/main/notebooks/aqlm_cuda_graph.ipynb) |
| PEFT fine-tuning | [Colab](https://colab.research.google.com/github/Vahe1994/AQLM/blob/main/notebooks/aqlm_2bit_training.ipynb) |
| vLLM serving | [Colab](https://colab.research.google.com/github/Vahe1994/AQLM/blob/main/notebooks/aqlm_vllm.ipynb) |

---

## Blog Posts & Technical Explanations

| Source | Link |
|--------|------|
| **Medium/TDS** | [The AQLM Quantization Algorithm, Explained](https://medium.com/data-science/the-aqlm-quantization-algorithm-explained-8cf33e4a783e) |
| **HuggingFace Docs** | [Transformers: AQLM](https://huggingface.co/docs/transformers/en/quantization/aqlm) |
| **vLLM Integration** | [GitHub Issue #3439](https://github.com/vllm-project/vllm/issues/3439) |

---

## Caveats

> *"AQLM quantization takes considerably longer to calibrate than simpler quantization methods such as GPTQ. This only impacts quantization time, not inference time."*

---

*Last updated: 2026-04-19*
