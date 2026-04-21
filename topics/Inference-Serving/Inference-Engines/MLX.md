# MLX (Apple Silicon)

> **GitHub:** [ml-explore/mlx](https://github.com/ml-explore/mlx) — 25k stars  
> **MLX LM:** [ml-explore/mlx-lm](https://github.com/ml-explore/mlx-lm) — 4.7k stars  
> **Docs:** [ml-explore.github.io/mlx](https://ml-explore.github.io/mlx/)  
> **Papers:** [Production-Grade LLM Inference on Apple Silicon arXiv:2511.05502](https://arxiv.org/abs/2511.05502)

---

## Overview

> *"MLX is Apple's open-source array framework for machine learning on Apple Silicon, designed for efficient inference and training on unified memory architecture."* — [Apple MLX GitHub](https://github.com/ml-explore/mlx)

**MLX** is Apple's answer to CUDA — an open-source ML framework for Apple Silicon with unified CPU/GPU memory, lazy computation, and a NumPy-like API. Built on **Metal** for GPU acceleration, it excels at local LLM inference with the **mlx-lm** package.

---

## Core Design Principles

| Principle | Description |
|-----------|-------------|
| **Unified Memory** | CPU and GPU share the same memory pool — no data transfer between devices |
| **NumPy-like API** | Familiar usage patterns, follows PyTorch conventions |
| **Lazy computation** | Arrays only materialize when needed |
| **Multi-device** | Operations run on CPU or GPU without explicit data movement |

---

## MLX LLM (mlx-lm)

```bash
pip install mlx-lm

# Text generation
mlx_lm.generate --prompt "How tall is Mt Everest?"

# Interactive chat
mlx_lm.chat

# Quantize and upload
mlx_lm.convert --model mistralai/Mistral-7B-Instruct-v0.3 -q --upload-repo mlx-community/my-4bit-mistral
```

### Python API

```python
from mlx_lm import load, generate

model, tokenizer = load("mlx-community/Mistral-7B-Instruct-v0.3-4bit")
response = generate(model, tokenizer, prompt="Write a story about Einstein")
```

---

## M5 Neural Accelerators

> *"M5 Neural Accelerators provide up to 4× speedup for TTFT vs M4."* — [Apple ML Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

| Model | TTFT Speedup (M5 vs M4) | Generation Speedup |
|-------|------------------------|-------------------|
| Qwen3-1.7B-MLX-bf16 | 3.57× | 1.27× |
| Qwen3-8B-MLX-bf16 | 3.62× | 1.24× |
| Qwen3-8B-MLX-4bit | 3.97× | 1.24× |
| Qwen3-14B-MLX-4bit | 4.06× | 1.19× |

---

## Performance vs Other Backends

> *"MLX achieves highest sustained generation throughput among Apple Silicon frameworks."* — [arXiv:2511.05502](https://arxiv.org/abs/2511.05502)

### vs llama.cpp

| Aspect | MLX | llama.cpp |
|--------|-----|-----------|
| **Prompt processing** | Faster | Slower |
| **Token generation (long context)** | ~50% slower | Faster |
| **Memory model** | Unified memory | Discrete |

### vs CUDA (NVIDIA)

| Aspect | MLX | CUDA |
|--------|-----|------|
| **Absolute speed** | Lower for massive models | Higher |
| **Energy efficiency** | Superior | Lower |
| **VRAM capacity** | Unified memory (exceeds single GPU VRAM) | Limited to GPU VRAM |

### Framework Comparison

| Framework | Best For | Quantization | KV Cache | Strengths |
|-----------|----------|--------------|----------|-----------|
| **MLX** | Mac users | 4-8 bit, BF16 | Rotating KV, prompt cache | Best raw throughput on Apple Silicon |
| **llama.cpp** | Lightweight single-stream | GGUF (all formats) | Sliding window | Efficient, portable |
| **Ollama** | Developer ergonomics | GGUF | Prefix reuse | One-command deploy |
| **MLC-LLM** | Mobile/cross-platform | AWQ, GPTQ, FP8 | Paged KV | iOS/Android |
| **PyTorch MPS** | PyTorch compatibility | INT4/INT8, BF16 | Minimal | Baseline GPU backend |

---

## Supported Models & Quantization

**HuggingFace:** [huggingface.co/mlx-community](https://huggingface.co/mlx-community) — 4,545 pre-converted models

| Format | Description |
|--------|-------------|
| **4-bit** | Most aggressive compression |
| **6-bit** | Balanced quality/efficiency |
| **8-bit** | Good balance, recommended |
| **BF16** | Full precision |

### GGUF Support

> *"MLX can read most GGUF quantization formats directly (Q4_0, Q4_1, Q8_0). Unsupported formats cast to float16."* — [MLX Examples](https://github.com/ml-explore/mlx-examples/blob/main/llms/gguf_llm/README.md)

---

## LoRA Fine-Tuning

```bash
# Fine-tune with LoRA
python lora.py --model mistralai/Mistral-7B-v0.1 --train --iters 600

# With quantized model (QLoRA)
python lora.py --model <quantized_model> --train --iters 600

# Fuse and upload
python fuse.py --upload-name My-4-bit-model
```

> *~475 tokens/sec on M2 Ultra for LoRA training. QLoRA enabled automatically when using quantized base models.*

---

## Swift API for iOS

> *"MLX Swift enables on-device LLM integration in iOS apps."* — [ml-explore/mlx-swift](https://github.com/ml-explore/mlx-swift)

Requires physical device (Metal GPU not available in Simulator).

---

## GitHub Repos

| Repo | Stars | Description |
|------|-------|-------------|
| [ml-explore/mlx](https://github.com/ml-explore/mlx) | 25k | Main MLX framework |
| [ml-explore/mlx-lm](https://github.com/ml-explore/mlx-lm) | 4.7k | LLM inference package |
| [ml-explore/mlx-examples](https://github.com/ml-explore/mlx-examples) | — | LoRA, Stable Diffusion, Whisper, GGUF |
| [ml-explore/mlx-swift](https://github.com/ml-explore/mlx-swift) | — | Swift iOS integration |

---

## Documentation

| Resource | URL |
|----------|-----|
| **MLX Docs** | [ml-explore.github.io/mlx](https://ml-explore.github.io/mlx/) |
| **MLX LM** | [github.com/ml-explore/mlx-lm](https://github.com/ml-explore/mlx-lm) |
| **HuggingFace Hub** | [huggingface.co/docs/hub/mlx](https://huggingface.co/docs/hub/mlx) |
| **Apple ML Research (M5)** | [machinelearning.apple.com/research/exploring-llms-mlx-m5](https://machinelearning.apple.com/research/exploring-llms-mlx-m5) |
| **WWDC 2025** | [developer.apple.com/videos/play/wwdc2025/298/](https://developer.apple.com/videos/play/wwdc2025/298/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Apple Silicon users wanting best local LLM throughput |
| **Key advantage** | Unified memory, M5 Neural Accelerators (4× TTFT), simplest Mac LLM setup |
| **Backend** | Metal GPU |
| **Quantization** | 4-bit to BF16, GGUF-compatible |
| **Fine-tuning** | LoRA/QLoRA built-in |

---

*Last updated: 2026-04-21*
