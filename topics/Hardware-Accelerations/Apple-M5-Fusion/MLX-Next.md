# MLX Next

> *"MLX is an array framework for efficient and flexible machine learning on Apple silicon, brought to you by Apple machine learning research."* — [Apple ML Research](https://github.com/ml-explore/mlx)

## Contents

1. [What is MLX?](#what-is-mlx)
2. [Key Specifications](#key-specifications)
3. [Repositories & Tools](#repositories--tools)
4. [Benchmarks on M5](#benchmarks-on-m5)
5. [Third-Party Benchmarks](#third-party-benchmarks)
6. [Hugging Face Ecosystem](#hugging-face-ecosystem)
7. [Blog Posts & Articles](#blog-posts--articles)
8. [Summary Table](#summary-table)

---

## What is MLX?

> *"MLX is an **open source array framework** that is efficient, flexible, and highly tuned for Apple silicon. You can use MLX for a wide variety of applications ranging from numerical simulations and scientific computing to machine learning."*
> — [Apple Machine Learning Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

> *"MLX makes it easy to generate text with or fine tune of large language models on Apple silicon devices."*
> — [Apple Machine Learning Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

### Design Influences

> *"Design Inspiration: NumPy, PyTorch, Jax, ArrayFire"*
> — [GitHub ml-explore/mlx](https://github.com/ml-explore/mlx)

### Unified Memory Architecture

> *"Apple silicon has a unified memory architecture. The CPU and GPU have direct access to the same memory pool. MLX is designed to take advantage of that."*
> *"A notable difference from MLX and other frameworks is the unified memory model. Arrays in MLX live in shared memory. Operations on MLX arrays can be performed on any of the supported device types without transferring data."*
> — [MLX Documentation](https://ml-explore.github.io/mlx/build/html/usage/unified_memory.html)

### Core Design Principles

| Feature | Description |
|---------|-------------|
| **Familiar APIs** | Python API closely follows NumPy; C++ and Swift APIs fully featured |
| **Lazy Computation** | Arrays are only materialized when needed |
| **Dynamic Graph Construction** | Changing shapes doesn't trigger slow compilations |
| **Composable Function Transformations** | Automatic differentiation, vectorization, computation graph optimization |
| **Multi-Device Support** | CPU and GPU operations |

---

## Key Specifications

### System Requirements

> *"MLX requires `macOS 26.2` or later to take advantage of M5's Neural Accelerators enhanced performance."*
> — [Apple ML Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

### Multi-Language Support

| Language | API |
|----------|-----|
| Python | `pip install mlx` |
| Swift | `pip install mlx` + Swift |
| C | MLX C API |
| C++ | Full MLX C++ API |

> *"MLX is an array framework for machine learning on Apple silicon, brought to you by Apple machine learning research."*
> — [Apple Open Source](https://opensource.apple.com/projects/mlx)

### Core Contributors

**Original Authors:**
- Awni Hannun
- Jagrit Digani
- Angelos Katharopoulos
- Ronan Collobert

---

## Repositories & Tools

### Core MLX

| Item | Value |
|------|-------|
| **Repository** | [github.com/ml-explore/mlx](https://github.com/ml-explore/mlx) |
| **Stars** | ~19k |
| **License** | BSD |

**Installation:**
```bash
pip install mlx
```

### MLX LM (LLM Inference Package)

| Item | Value |
|------|-------|
| **Repository** | [github.com/ml-explore/mlx-lm](https://github.com/ml-explore/mlx-lm) |
| **Stars** | ~4.7k |

**Features:**
- Run most LLMs from Hugging Face
- Native quantization support
- Quick model conversion (`mlx_lm.convert`)
- LoRA and QLoRA fine-tuning
- Streaming generation
- Multiple sampling methods

**Quick Start:**
```bash
pip install mlx-lm
mlx_lm.generate --model mlx-community/Qwen3-4B-Instruct-2507-4bit --prompt "hello"
```

### MLX Examples

| Item | Value |
|------|-------|
| **Repository** | [github.com/ml-explore/mlx-examples](https://github.com/ml-explore/mlx-examples) |
| **Stars** | ~6k |

**Includes:** LLaMA, Stable Diffusion, Whisper, Segment Anything Model, LoRA fine-tuning, GGUF format support.

### MLX Swift

| Item | Value |
|------|-------|
| **Repository** | [github.com/ml-explore/mlx-swift](https://github.com/ml-explore/mlx-swift) |
| **Stars** | ~1.7k |

> *"MLX Swift expands MLX to the Swift language, making experimentation on Apple silicon easier for ML researchers."*
> — [Swift.org](https://swift.org/blog/mlx-swift/)

### WWDC25 Resources

| Resource | URL |
|----------|-----|
| Get started with MLX for Apple silicon | [developer.apple.com/videos/play/wwdc2025/315/](https://developer.apple.com/videos/play/wwdc2025/315/) |
| Explore LLMs on Apple silicon with MLX | [developer.apple.com/videos/play/wwdc2025/298/](https://developer.apple.com/videos/play/wwdc2025/298/) |

### Third-Party Tools

| Tool | URL | Description |
|------|-----|-------------|
| **llm-mlx** | [github.com/simonw/llm-mlx](https://github.com/simonw/llm-mlx) | Simon Willison's LLM library plugin for MLX |
| **mlx-my-repo** | [huggingface.co/spaces/mlx-community/mlx-my-repo](https://huggingface.co/spaces/mlx-community/mlx-my-repo) | Web-based model conversion Space |

---

## Benchmarks on M5

### Official Apple ML Research Benchmarks

**Configuration:** MacBook Pro M5 (24GB) vs MacBook Pro M4 (24GB), macOS 26.2 beta

**Memory Bandwidth Comparison:**
| Component | M4 | M5 | Improvement |
|-----------|-----|-----|-------------|
| Memory Bandwidth | 120 GB/s | 153 GB/s | **+28%** |

### Time To First Token (TTFT) — M5 vs M4

| Model | Precision | TTFT Speedup | Generation Speedup | Memory |
|-------|-----------|-------------|-------------------|--------|
| Qwen3-1.7B-MLX-bf16 | BF16 | **3.57x** | 1.27x | 4.40 GB |
| Qwen3-8B-MLX-bf16 | BF16 | **3.62x** | 1.24x | 17.46 GB |
| Qwen3-8B-MLX-4bit | 4-bit | **3.97x** | 1.24x | 5.61 GB |
| Qwen3-14B-MLX-4bit | 4-bit | **4.06x** | 1.19x | 9.16 GB |
| gpt-oss-20b-MXFP4-Q4 | MXFP4 | **3.33x** | 1.24x | 12.08 GB |
| Qwen3-30B-A3B-MLX-4bit | 4-bit (MoE) | **3.52x** | 1.25x | 17.31 GB |

> *"The GPU Neural Accelerators shine with MLX on ML workloads involving large matrix multiplications, yielding up to 4x speedup compared to a M4 baseline for time-to-first-token in language model inference."*
> — [Apple ML Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

### Image Generation Performance

| Model | Improvement vs M4 |
|-------|------------------|
| FLUX-dev-4bit (12B params) | **3.8x faster** |

> *"In LLM inference, generating the first token is compute-bound, and takes full advantage of the Neural Accelerators. The M5 pushes the time-to-first-token generation under 10 seconds for a dense 14B architecture, and under 3 seconds for a 30B MoE."*
> — [Apple ML Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

---

## Third-Party Benchmarks

### M4 Pro vs M5 Pro — Tokens per Second

| Configuration | Model | Quantization | Context | Tokens/sec |
|---------------|-------|--------------|---------|------------|
| M4 Pro 24GB | Qwen 2.5 14B | 4-bit | 8K | 52-58 |
| M4 Pro 48GB | Qwen 2.5 32B | 4-bit | 8K | 32-38 |
| **M5 Pro 48GB** | Qwen 2.5 14B | 4-bit | 32K | **58-65** |
| **M5 Pro 48GB** | Qwen 2.5 32B | 4-bit | 8K | **42-50** |
| M5 Pro 64GB | Llama 3.3 70B | 4-bit | 8K | **18-24** |

> *"The 32B model on M5 Pro 48GB delivering 42-50 tokens/sec at 8K context is faster than typical cloud API response times under load—with zero per-token cost and no external data exposure."*
> — [Contra Collective](https://contracollective.com/blog/m4-m5-pro-local-ai-inference-mlx-2026)

> *"The 64GB M5 Pro is the first MacBook Pro or Mac Studio configuration that runs Llama 3.3 70B at 4-bit with comfortable headroom."*
> — [Contra Collective](https://contracollective.com/blog/m4-m5-pro-local-ai-inference-mlx-2026)

### Prefill Stage Performance (Third-Party)

> *"Looking at Max's benchmarks with Qwen3 8B and a ~20,000-token prompt, there is indeed a **3.65x speedup** in tokens/sec in the prefill stage – jumping from 158.2 tok/s to a remarkable 578.7 tok/s."*
> — [MacStories](https://www.macstories.net/stories/ipad-pro-m5-neural-benchmarks-mlx/)

---

## Hugging Face Ecosystem

### MLX Community Hub

| Metric | Value |
|--------|-------|
| **Models** | 4,545+ |
| **Team Members** | 4,394 |
| **Collections** | 128 |
| **Datasets** | 35 |

> [huggingface.co/mlx-community](https://huggingface.co/mlx-community)

### Featured Model Series on MLX

**MiniMax-M2.6 Series:**
- `mlx-community/MiniMax-M2.6-4bit`
- `mlx-community/MiniMax-M2.6-5bit`
- `mlx-community/MiniMax-M2.6-6bit`
- `mlx-community/MiniMax-M2.6-8bit`

**Qwen3.6-35B-A3B Variants:**
- `mlx-community/Qwen3.6-35B-A3B-nvfp4`
- `mlx-community/Qwen3.6-35B-A3B-mxfp8`
- `mlx-community/Qwen3.6-35B-A3B-mxfp4`
- `mlx-community/Qwen3.6-35B-A3B-4bit`
- `mlx-community/Qwen3.6-35B-A3B-8bit`
- `mlx-community/Qwen3.6-35B-A3B-6bit`

**Other Notable Models:**
- `mlx-community/gemma-4-31b-it-4bit`
- `mlx-community/Llama-3.3-70B-Instruct-4bit`
- `mlx-community/Qwen3.5-9B-MLX-4bit`
- `mlx-community/Llama-3.2-3B-Instruct-4bit`

### Model Conversion

```bash
# Convert and upload to Hugging Face
mlx_lm.convert --model Qwen/Qwen3-4B-Instruct-2507 -q --upload-repo mlx-community/Qwen3-4B-Instruct-2507-4bit
```

### Supported Architectures

> *MLX-LM supports popular LLM architectures including: LLaMA, Phi-2, Mistral, Qwen*
> — [Hugging Face MLX Documentation](https://huggingface.co/docs/hub/mlx)

---

## Blog Posts & Articles

### Ollama MLX Support (March 2026)

> *"Ollama on Apple silicon is now built on top of Apple's machine learning framework, MLX, to take advantage of its unified memory architecture. This results in a large speedup of Ollama on all Apple Silicon devices."*
> *"On Apple's M5, M5 Pro and M5 Max chips, Ollama leverages the new GPU Neural Accelerators to accelerate both time to first token (TTFT) and generation speed (tokens per second)."*
> — [Ollama Blog](https://ollama.com/blog/mlx), March 30, 2026

**Ollama MLX Performance (Qwen3.5-35B-A3B):**
- Prefill: 1,851 token/s
- Decode: 134 token/s (int4 quantization)

### Ars Technica Coverage

> *"Ollama, a runtime system for operating large language models on a local computer, has introduced support for Apple's open source MLX framework for machine learning."*
> — [Ars Technica](https://arstechnica.com/apple/2026/03/running-local-models-on-macs-gets-faster-with-ollamas-mlx-support/)

### MLX + EXO Performance Guide

> *"MLX does not attempt to be a deployment framework like Core ML. Instead, it complements the Apple ecosystem by giving researchers and engineers a tool to prototype, iterate, and run state-of-the-art models efficiently on Macs."*
> — [Petronella Tech](https://petronellatech.com/blog/mlx-exo-unlocking-apple-silicon-s-ml-performance/)

### 9to5Mac Coverage

> *"Apple showed how much faster the M5 runs local LLMs compared to the M4."*
> — [9to5Mac](https://9to5mac.com/2025/11/20/apple-shows-how-much-faster-the-m5-runs-local-llms-compared-to-the-m4/)

---

## Summary Table

### Performance Summary (M5 vs M4 via MLX)

| Metric | Improvement | Notes |
|--------|-------------|-------|
| Time To First Token (TTFT) | Up to **4x faster** | Compute-bound; leverages Neural Accelerators |
| Token Generation Speed | **19-27% faster** | Memory-bandwidth-bound (153 vs 120 GB/s) |
| Memory Bandwidth | **+28%** | 120 → 153 GB/s |
| FLUX-dev-4bit Image Gen | **3.8x faster** | 12B parameter model |

### MLX Package Ecosystem

| Package | Purpose |
|---------|---------|
| `mlx` | Core array framework |
| `mlx-lm` | LLM inference and fine-tuning |
| `mlx-examples` | Example implementations |
| `mlx-swift` | Swift API |
| `mlx-c` | C API |

### Memory Capacity on MacBook Pro 24GB

| Model | Precision | Memory Footprint |
|-------|-----------|------------------|
| 8B | BF16 | ~18 GB |
| 30B MoE | 4-bit | ~18 GB |
| 14B | 4-bit | ~9 GB |

### Key Hardware Requirements

> *"MLX requires `macOS 26.2` or later to take advantage of M5's Neural Accelerators enhanced performance."*
> — [Apple ML Research](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

---

## Key Findings

- **MLX is Apple's open-source array framework** (~19k GitHub stars) for ML on Apple Silicon with Python, Swift, C, C++ APIs
- **M5 Neural Accelerators: up to 4x TTFT speedup** vs M4 through MLX, 19-27% generation speedup
- **Ollama now uses MLX** as backend (v0.19+) — 1,851 token/s prefill on Qwen3.5-35B-A3B
- **4,545+ models on Hugging Face MLX Community** including MiniMax-M2.6, Qwen, Llama, Gemma
- **LoRA/QLoRA fine-tuning** natively supported via `mlx_lm.lora`
- **Metal 4 Tensor APIs** allow direct Neural Accelerator programming
- **Unified memory model**: arrays live in shared memory; CPU/GPU operations without data transfer

---

## Related Topics

- [M5-Neural-Accelerators](./M5-Neural-Accelerators.md) — Hardware architecture enabling MLX performance
- [MLX-Next](./Nvidia-Blackwell-FP4/FP4-Tensor-Cores.md) — NVIDIA's competing FP4 precision
- [DGX-Spark](./Nvidia-Blackwell-FP4/DGX-Spark.md) — NVIDIA personal AI supercomputer comparison
