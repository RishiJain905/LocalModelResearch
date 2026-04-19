# GPTQ: Generative Pre-trained Transformer Quantization

> **Original Paper:** [arXiv:2210.17323](https://arxiv.org/abs/2210.17323) — ICLR 2023  
> **Authors:** Elias Frantar, Saleh Ashkboos, Torsten Hoefler, Dan Alistarh  
> **Official Repo:** [IST-DASLab/gptq](https://github.com/IST-DASLab/gptq)

---

## Overview

GPTQ was the first post-training quantization (PTQ) method to achieve **viable 3-4 bit quantization** for LLMs with hundreds of billions of parameters.

> *"GPTQ can quantize GPT models with 175 billion parameters in approximately four GPU hours, reducing the bit-width down to 3 or 4 bits per weight, with negligible accuracy degradation relative to the uncompressed baseline."*

---

## Original Paper

| | |
|---|---|
| **Title** | GPTQ: Accurate Post-Training Quantization for Generative Pre-trained Transformers |
| **Venue** | ICLR 2023 |
| **arXiv** | [https://arxiv.org/abs/2210.17323](https://arxiv.org/abs/2210.17323) |
| **PDF** | [https://arxiv.org/pdf/2210.17323](https://arxiv.org/pdf/2210.17323) |
| **Code** | [https://github.com/IST-DASLab/gptq](https://github.com/IST-DASLab/gptq) |

---

## GitHub Repositories

| Repository | Stars | Status | Notes |
|---|---|---|---|
| [IST-DASLab/gptq](https://github.com/IST-DASLab/gptq) | ⭐ | Official | Original implementation by paper authors |
| [AutoGPTQ/AutoGPTQ](https://github.com/AutoGPTQ/AutoGPTQ) | 5k | ⚠️ Archived | **Deprecated** — recommends GPTQModel as successor |
| [ModelCloud/GPTQModel](https://github.com/ModelCloud/GPTQModel) | ⭐ | Active | Supports 70+ models, multiple backends (vLLM, SGLang, HF) |
| [IST-DASLab/gptq-gguf-toolkit](https://github.com/IST-DASLab/gptq-gguf-toolkit) | ⭐ | Active | Direct GPTQ → GGUF export |

> *"Prior post-training methods only remain accurate at 8 bits... GPTQ is the first to show that extremely accurate language models with hundreds of billions of parameters can be quantized to 3-4 bits/component."*

---

## Performance Benchmarks

### WikiText2 Perplexity — LLaMA Family (Lower is Better)

| Model | FP16 | 4bit-RTN | **4bit-GPTQ** | 3bit-RTN | **3bit-GPTQ** |
|-------|------|----------|---------------|----------|---------------|
| LLaMA-7B | 5.68 | 6.29 | **6.09** | 25.54 | **8.07** |
| LLaMA-13B | 5.09 | 5.53 | **5.36** | 11.40 | **6.63** |
| LLaMA-30B | 4.10 | 4.54 | **4.45** | 14.89 | **5.69** |
| LLaMA-65B | 3.53 | 3.92 | **3.84** | 10.59 | **5.04** |

### OPT-175B Key Result
- **FP16:** 8.34
- **GPTQ 4-bit:** 8.37 (virtually no degradation)

### Speedups

| Platform | Speedup vs FP16 |
|----------|-----------------|
| A100 GPUs | **~3.25×** |
| A6000 GPUs | **~4.5×** |

---

## Comparison with Other Methods

| Method | Key Difference | Inference Speed | Memory Savings | Notes |
|--------|----------------|-----------------|---------------|-------|
| **GPTQ** | Post-training reconstruction error minimization | ✅ Faster (3-4×) | ~4x | Best for GPU inference |
| **bitsandbytes 8-bit** | Mixed-precision with outlier handling | ❌ Slower | Good | Requires full model in RAM first |
| **NF4 (bitsandbytes 4-bit)** | 4-bit NormalFloat data type | ❌ Slower | ~4x | Best for QLoRA fine-tuning |
| **AWQ** | Salient weight protection | Medium | ~4x | Better quality at 4-bit on instruction tasks |
| **GGML/GGUF** | CPU-friendly, different quantization scheme | CPU-optimized | Good | Best for CPU-only setups |

---

## Supported Models

**70+ models** via GPTQModel including:
- LLaMA 1/2/3/4
- Qwen, DeepSeek, GLM, Gemma
- Mistral/Mixtral, Phi
- Cohere, Falcon, OPT, BLOOM
- ChatGLM

---

## Usage

### Basic Quantization (GPTQModel)

```bash
pip install gptqmodel
```

```python
from gptqmodel import GPTQModel

model = GPTQModel.load(
    "model_name_or_path",
    quantization_config={"bits": 4, "group_size": 128}
)
model.generate("Your prompt here")
```

### vLLM Inference

```python
from vllm import LLM, SamplingParams

llm = LLM(model="TheBloke/Llama-2-7B-Chat-GPTQ", quantization="gptq")
```

---

## Key Algorithmic Innovation

GPTQ improves on OBQ (Optimal Brain Quantization) by:

1. **Batch quantization** — processes multiple columns together for GPU efficiency
2. **Lazy batch updates** — defers weight updates to reduce computational overhead  
3. **Arbitrary group sizes** — flexible 32/64/128/1024 parameter grouping

---

## External Resources

- **Hugging Face Blog:** [Introduction to Quantization](https://huggingface.co/blog/merve/quantization)
- **Maxime Labonne:** [4-bit LLM Quantization with GPTQ](https://mlabonne.github.io/blog/posts/4_bit_Quantization_with_GPTQ.html)
- **Towards Data Science:** [4-bit Quantization with GPTQ](https://towardsdatascience.com/4-bit-quantization-with-gptq-36b0f4f02c34/)
- **Aakash Varma:** [The GPTQ Algorithm: Enhancing OBQ for Large Language Models](https://aakashvarma.substack.com/p/gptq)
- **Abstract Algorithms:** [GPTQ vs AWQ vs NF4 Comparison](https://www.abstractalgorithms.dev/gptq-vs-awq-vs-nf4-choosing-the-right-llm-quantization-pipeline)

---

*Last updated: 2026-04-19*
