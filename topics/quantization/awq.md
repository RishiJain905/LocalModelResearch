# AWQ: Activation-Aware Weight Quantization

> **Original Paper:** [arXiv:2306.00978](https://arxiv.org/abs/2306.00978) — MLSys 2024 **Best Paper Award**  
> **Authors:** Ji Lin, Jiaming Tang, Haotian Tang, Shang Yang, Wei-Ming Chen, Wei-Chen Wang, Guangxuan Xiao, Xingyu Dang, Chuang Gan, Song Han (MIT Han Lab)  
> **Official Repo:** [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq)

---

## Overview

> *"Protecting only 1% salient weights can greatly reduce quantization error."*

> *"AWQ does not rely on any backpropagation or reconstruction, so it can well preserve LLMs' generalization ability on different domains and modalities, without overfitting to the calibration set."*

---

## Original Paper

| | |
|---|---|
| **Title** | AWQ: Activation-aware Weight Quantization for LLM Compression and Acceleration |
| **Venue** | **MLSys 2024 — Best Paper Award** |
| **arXiv** | [https://arxiv.org/abs/2306.00978](https://arxiv.org/abs/2306.00978) |
| **Paper PDF** | [https://arxiv.org/pdf/2306.00978](https://arxiv.org/pdf/2306.00978) |
| **Website** | [hanlab.mit.edu/projects/awq](https://hanlab.mit.edu/projects/awq) |
| **DOI** | [10.48550/arXiv.2306.00978](https://doi.org/10.48550/arXiv.2306.00978) |

---

## GitHub Repositories

### Official Research Repo
| | |
|---|---|
| **Repo** | [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) |
| **License** | MIT |
| **Features** | AWQ search, TinyChat inference framework, CUDA kernels, VILA support, W4A16 quantization |

### AutoAWQ (Deprecated)

| | |
|---|---|
| **Repo** | [casper-hansen/AutoAWQ](https://github.com/casper-hansen/AutoAWQ) |
| **Status** | ⚠️ **Deprecated** — moved to llm-compressor |

> *"AutoAWQ is officially deprecated and will no longer be maintained."*

### Recommended: llm-compressor

| | |
|---|---|
| **Repo** | [vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor) |
| **Purpose** | Official continuation of AWQ quantization for vLLM |
| **Examples** | [llm-compressor/examples/awq](https://github.com/vllm-project/llm-compressor/tree/main/examples/awq) |

---

## How AWQ Works

AWQ is based on a key observation: **not all weights matter equally**. LLMs have outlier channels (salient weights ~1%) that dominate quantization error. AWQ protects these salient weights via per-channel scaling.

### Core Algorithm

1. **Measure activation magnitudes** on a calibration dataset (no backprop needed)
2. **Search for optimal per-channel scaling factors** that protect salient weights
3. **Quantize** all weights with the learned scaling

### Key Differences from GPTQ

| Aspect | AWQ | GPTQ |
|--------|-----|------|
| **Optimization** | Per-channel scaling based on activations | Layer-by-layer iterative optimization |
| **Calibration** | Only measures activation magnitude | Requires weight reconstruction |
| **Backpropagation** | Not required | Required |
| **Generalization** | Excellent (no overfitting) | May overfit calibration set |
| **Multi-modal support** | ✅ First work | ❌ Not supported |
| **Instruction-tuned LMs** | Excellent | Good |

---

## Performance Benchmarks

### WikiText Perplexity (Lower is Better)

| Model | FP16 | RTN | GPTQ | GPTQ-R | **AWQ** |
|-------|------|-----|------|--------|---------|
| Llama-2-7B (INT4) | 5.47 | 5.73 | 5.69 | 5.63 | **5.60** |
| Llama-2-13B (INT4) | 4.88 | 4.98 | 4.98 | 4.99 | **4.97** |
| Llama-2-70B (INT4) | 3.32 | 3.46 | 3.42 | 3.43 | **3.41** |

### Multi-Modal VLM (COCO Captioning CIDEr ↑)

| Method | 32-shot | Δ(32-shot) |
|--------|---------|------------|
| FP16 | 81.70 | — |
| RTN (INT4) | 77.13 | -4.57 |
| GPTQ (INT4) | 74.98 | -6.72 |
| **AWQ (INT4)** | **80.53** | **-1.17** |

*AWQ achieves near-lossless VLM quantization — first work to do so.*

### Speed (TinyChat vs FP16 HuggingFace)

| Platform | Model | Speedup vs FP16 |
|----------|-------|-----------------|
| RTX 4090 | LLaMA-3-8B | **2.7×** |
| Jetson Orin | LLaMA-3-8B | **2.9×** |

### Benchmark Comparison (Llama-2-13B)

| Model | Perplexity ↓ | VRAM (GB) | Size (GB) | Tokens/s |
|-------|-------------|-----------|-----------|----------|
| **AWQ-4bit-32g** | **4.325** | 10.567 | 7.624 | 39.47 |
| AWQ-4bit-128g | 4.348 | 9.623 | 6.915 | 40.61 |
| GPTQ-4bit-32g | 4.338 | 8.701 | 7.633 | 42.44 |
| GPTQ-4bit-128g | 4.358 | 7.935 | 6.924 | 51.91 |

*AWQ achieves lower perplexity but higher VRAM than GPTQ at same group size.*

---

## Framework Support

| Framework | AWQ Support | Status |
|-----------|-------------|--------|
| **vLLM** | ✅ Full | Recommended for production |
| **llm-compressor** | ✅ Full | Official quantization tool |
| **llama.cpp** | ⚠️ Limited | Use GGUF instead |
| **HuggingFace** | ✅ Via AutoAWQ | Deprecated |
| **TinyChat** | ✅ Official | Edge device optimization |

### Hardware Compatibility (vLLM)

| | Ampere | Ada | Hopper | AMD GPU | Intel GPU |
|---|---|---|---|---|---|
| **AWQ** | ✅ | ✅ | ✅ | ❌ | ✅ |

---

## Model Zoo

Pre-computed search results: [huggingface.co/datasets/mit-han-lab/awq-model-zoo](https://huggingface.co/datasets/mit-han-lab/awq-model-zoo)

| Model Family | Sizes | INT4-g128 | INT3-g128 |
|--------------|-------|-----------|-----------|
| DeepSeek-R1-Distill | 1.5B/7B/8B | ✅ | — |
| Qwen-2.5 | 7B/72B | ✅ | — |
| NVILA | 3B/8B | ✅ | — |
| VILA-1.5 | 3B/8B/13B/40B | ✅ | ✅ |
| Llama3 | 8B/70B | ✅ | ✅ |
| Llama2 | 7B/13B/70B | ✅ | ✅ |
| LLaMA | 7B/13B/30B/65B | ✅ | ✅ |
| OPT | 125m–30B | ✅ | ✅ |
| CodeLlama | 7B/13B/34B | ✅ | ✅ |

---

## Usage

### vLLM (Recommended for Production)

```python
from vllm import LLM, SamplingParams

llm = LLM(
    model="TheBloke/Llama-2-7b-Chat-AWQ",
    quantization="awq"
)
```

### llm-compressor Quantization

```bash
pip install llmcompressor
```

```python
from llmcompressor import oneshot
from transformers import AutoModelForCausalLM, AutoTokenizer

model = AutoModelForCausalLM.from_pretrained("meta-llama/Llama-2-7b-hf")
tokenizer = AutoTokenizer.from_pretrained("meta-llama/Llama-2-7b-hf")

# AWQ quantization
oneshot(
    model=model,
    tokenizer=tokenizer,
    recipe=oneshot.model_settings.weight_quantize(
        "AWQ", ignore=["lm_head"]
    ),
    calibration_tasks=["lm_eval_harness"],
    num_calibration_samples=512,
)
```

### HuggingFace AutoAWQ (Deprecated)

```python
from awq import AutoAWQForCausalLM

model = AutoAWQForCausalLM.from_pretrained("model_path")
```

---

## Blog Posts & Technical Explanations

- **Lei Mao's Blog:** [AWQ: Activation-Aware Weight Quantization](https://leimao.github.io/blog/AWQ-Activation-Aware-Weight-Quantization/)
- **MLSys Talk Video:** [YouTube - MLSys'24 Best Paper Talk](https://www.youtube.com/watch?v=dcINVsqxQgQ)
- **vLLM Documentation:** [AutoAWQ Integration](https://docs.vllm.ai/en/stable/features/quantization/auto_awq/)
- **Byte-Sized AI Blog:** [vLLM Quantization: AWQ](https://medium.com/byte-sized-ai/vllm-quantization-awq-activation-aware-weight-quantization-for-llm-compression-and-35894ffd6a9b)

---

*Last updated: 2026-04-19*
