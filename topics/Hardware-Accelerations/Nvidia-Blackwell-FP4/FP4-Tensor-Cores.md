# FP4 Tensor Cores

> *"NVFP4 is an innovative 4-bit floating-point format introduced with the NVIDIA Blackwell GPU architecture, purpose-built to help developers scale inference more efficiently with best accuracy at ultra-low precision."* — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

## Contents

1. [NVFP4 Format Specification](#nvfp4-format-specification)
2. [Key Sources & Papers](#key-sources--papers)
3. [Repositories & Tools](#repositories--tools)
4. [Pre-Quantized Models](#pre-quantized-models)
5. [Benchmarks & Performance](#benchmarks--performance)
6. [Debates & Trade-offs](#debates--trade-offs)
7. [Key Findings](#key-findings)

---

## NVFP4 Format Specification

### E2M1 Format

| Component | Bits | Description |
|-----------|------|-------------|
| Sign | 1 | Positive/negative |
| Exponent | 2 | Bias of 1 |
| Mantissa | 1 | Fraction bit |
| **Total** | **4 bits** | |

> *"NVFP4 is an innovative 4-bit floating-point format introduced with the NVIDIA Blackwell GPU architecture."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

**Representable values:** `{0, ±0.5, ±1, ±1.5, ±2, ±3, ±4, ±6}` (8 positive values, ~−6 to +6 range)

### Dual-Scaling Architecture

> *"Instead of assigning one scaling factor to an entire tensor, the tensor is divided into small blocks (e.g., 32 elements), each with its own shared 8-bit scale exponent."*
> — [Verda Blog](https://verda.com/blog/nvfp4-nvidia-blackwell-intro)

NVFP4 uses two-level scaling:
1. **Micro-block scaling (FP8 E4M3):** Per 16-value block
2. **Global tensor scaling (FP32):** Per entire tensor

### 5th-Generation Tensor Core Specifications

| Spec | B200 (Datacenter) | RTX 5090 (GeForce) |
|------|-------------------|---------------------|
| Architecture | Blackwell | Blackwell |
| SMs | 148 | 170 |
| Tensor Cores | 5th Gen | 5th Gen |
| Peak FP4 Tensor TOPS | 18,000 sparse | 1,676/3,352 sparse |
| Memory | 192 GB HBM3e | 32 GB GDDR7 |
| Transistors | 208 billion (dual-die) | 92.2 billion |

> *"5th Generation Tensor Cores — New FP4 support, double FP8 throughput"*
> — [NVIDIA RTX Blackwell GPU Architecture Whitepaper](https://images.nvidia.com/aem-dam/Solutions/geforce/blackwell/nvidia-rtx-blackwell-gpu-architecture.pdf)

### FP4 Format Comparison

| Feature | FP4 (E2M1) | MXFP4 | NVFP4 |
|---------|-----------|-------|-------|
| Structure | 4 bits + software scaling | 4 bits + power-of-two scale per 32 values | 4 bits + FP8 scale per 16 values |
| Hardware Scaling | No | Yes | **Yes** |
| Accuracy Risk | High | Medium | **Lowest** |
| Block Size | N/A | 32 | **16** |
| Scale Format | N/A | E8M0 | **E4M3** |

> *"Blackwell was the first architecture from NVIDIA to natively support FP4 formats."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/nvfp4-trains-with-precision-of-16-bit-and-speed-and-efficiency-of-4-bit/)

---

## Key Sources & Papers

### NVIDIA Developer Blog Posts

**Introducing NVFP4 for Efficient and Accurate Low-Precision Inference**
> *"NVFP4 solves the accuracy-vs-precision tradeoff through two-level scaling (E4M3 per block + FP32 per tensor)"*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/), June 24, 2025

**NVFP4 Trains with Precision of 16-Bit and Speed and Efficiency of 4-Bit**
> *"Blackwell was the first architecture from NVIDIA to natively support FP4 formats. The massive FP4 FLOPs throughput on GB200 and GB300..."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/nvfp4-trains-with-precision-of-16-bit-and-speed-and-efficiency-of-4-bit/), August 25, 2025

**NVIDIA Blackwell Delivers World-Record DeepSeek-R1 Inference Performance**
> *"A single NVIDIA DGX system with eight NVIDIA Blackwell GPUs can achieve over 250 tokens per second per user or a maximum throughput of over 30,000 tokens per second on the massive, state-of-the-art 671 billion parameter DeepSeek-R1 model."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/nvidia-blackwell-delivers-world-record-deepseek-r1-inference-performance/), March 18, 2025

### Academic Papers

**RaZeR: Pushing the Limits of NVFP4 Quantization with Redundant Zero Remapping**
> *"RaZeR extends the standard NVFP4 format by remapping the redundant FP4 zero as an additional, special quantization value. By carefully selecting the set of allowed special values, each NVFP4 block can be quantized with basic FP4 values and a useful special value, thereby reducing per-block quantization error."*
> — [arXiv:2501.04052](https://arxiv.org/html/2501.04052v2) — Cornell University (abdelfattah-lab)

**Microbenchmarking NVIDIA's Blackwell Architecture**
> *"Our systematic evaluation of dense/sparse GEMM, transformer inference, and training workloads demonstrates that B200's tensor core enhancements achieve 1.85× ResNet-50 and 1.55× GPT-1.3B mixed-precision training throughput with 32% better energy efficiency than H200."*
> — [arXiv:2512.02189](https://arxiv.org/html/2512.02189v2)

**Towards Efficient Pre-training: Exploring FP4 Precision in Large Language Models**
> — [arXiv:2502.11458](https://arxiv.org/abs/2502.11458)

**Optimizing Large Language Model Training Using FP4 Quantization**
> — [arXiv:2501.17116](https://arxiv.org/abs/2501.17116)

**Elucidating the Design Space of FP4 Training**
> — [arXiv:2509.17791](https://arxiv.org/abs/2509.17791)

---

## Repositories & Tools

| Tool | URL | Purpose |
|------|-----|---------|
| **NVIDIA TensorRT Model Optimizer** | [github.com/NVIDIA/TensorRT-Model-Optimizer](https://github.com/NVIDIA/TensorRT-Model-Optimizer/tree/main) | PTQ, QAT, non-LLM models, ONNX export |
| **LLM Compressor** | [github.com/vllm-project/llm-compressor](https://github.com/vllm-project/llm-compressor) | PTQ, QAT workflows for vLLM |
| **TensorRT-LLM** | [github.com/NVIDIA/TensorRT-LLM](https://github.com/NVIDIA/TensorRT-LLM) | ✅ NVFP4 deployment support |
| **Intel Neural Compressor** | [github.com/intel/neural-compressor](https://github.com/intel/neural-compressor/blob/master/docs/source/PT_NVFP4Quant.md) | PTQ for NVFP4 |
| **CUTLASS** | [github.com/NVIDIA/cutlass](https://github.com/NVIDIA/cutlass) | 98% relative peak performance for Tensor Core ops; FP4 support |
| **NVFP4-RaZeR** | [github.com/abdelfattah-lab/NVFP4-RaZeR](https://github.com/abdelfattah-lab/NVFP4-RaZeR) | Cornell research; remaps redundant FP4 zero as special quantization value |

> *"Neural Compressor supports post-training quantization to NVFP4, providing recipes and APIs for users to quantize LLMs easily."*
> — [Intel Neural Compressor](https://github.com/intel/neural-compressor/blob/master/docs/source/PT_NVFP4Quant.md)

---

## Pre-Quantized Models

| Model | URL |
|-------|-----|
| DeepSeek-R1-NVFP4 | [huggingface.co/nvidia/DeepSeek-R1-NVFP4](https://huggingface.co/nvidia/DeepSeek-R1-NVFP4) |
| Llama-3.1-405B-Instruct-FP4 | [huggingface.co/nvidia/Llama-3.1-405B-Instruct-FP4](https://huggingface.co/nvidia/Llama-3.1-405B-Instruct-FP4) |
| Llama-4-Scout-17B-16E-Instruct-NVFP4 | [huggingface.co/nvidia/Llama-4-Scout-17B-16E-Instruct-NVFP4](https://huggingface.co/nvidia/Llama-4-Scout-17B-16E-Instruct-NVFP4) |
| FLUX.1-dev-onnx | [huggingface.co/black-forest-labs/FLUX.1-dev-onnx](https://huggingface.co/black-forest-labs/FLUX.1-dev-onnx) |

---

## Benchmarks & Performance

### MLPerf Inference Results

| GPU | Precision | Tokens/sec | Source |
|-----|-----------|------------|--------|
| H100 SXM (8-GPU) | FP8 | ~24,525 | MLPerf v4.1 |
| H200 SXM (8-GPU) | FP8 | ~34,988 | MLPerf v5.0 |
| **B200 (8-GPU)** | **FP4** | **~98,858** | **MLPerf v5.0** |
| **B200 (8-GPU)** | **FP4** | **~102,725** | **MLPerf v5.1** |

> — [Spheron](https://www.spheron.network/blog/fp4-quantization-blackwell-gpu-cost/)

### DeepSeek-R1 Accuracy (FP8 vs FP4)

| Dataset | FP8 | FP4 | Δ |
|---------|-----|-----|---|
| MMLU | 90.8% | 90.7% | −0.1% |
| GSM8K | 96.3% | 96.1% | −0.2% |
| AIME 2024 | 80.0% | 80.0% | 0% |
| GPQA Diamond | 69.7% | 69.2% | −0.5% |
| MATH-500 | 95.4% | 94.2% | −1.2% |

> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/nvidia-blackwell-delivers-world-record-deepseek-r1-inference-performance/)

### GPU Generation Performance Evolution

| GPU | Dense (PF) | Sparse (PF) | Smallest Format |
|-----|-----------|-------------|-----------------|
| A100 | 0.3 | 0.6 | FP16 |
| H100 | 1.9 | 3.9 | FP8 |
| B200 | 9 | 18 | FP4 |
| B300 | 13 | 18 | FP4 |
| GB200 | 10 | 20 | FP4 |
| GB300 | 15 | 20 | **NVFP4** |

> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

### Energy Efficiency

| GPU Generation | J/Token |
|----------------|---------|
| Kepler (2014) | 42,000 |
| Pascal | 17,460 |
| Volta | 1,200 |
| Ampere | 150 |
| Hopper | 10 |
| **Blackwell** | **0.4** |
| **Blackwell Ultra** | **0.2** |

> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

---

## Debates & Trade-offs

### Accuracy vs. Throughput

> *"QAT achieves lossless FP4 quantization compared to BF16 baseline"*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/)

NVFP4 maintains ≤1% accuracy degradation in most benchmarks. Some degradation in reasoning tasks (MATH-500: −1.2%, Llama 3.3 70B GSM8K: −2.7%). QAT can recover accuracy to lossless levels.

### NVFP4 vs. MXFP8

> *"For a mere 1.3-second 'tax' over NVFP4, MXFP8 stays remarkably closer to the 16-bit BF16 original."*
> — [Creepybits](https://zanno.se/blackwell-mxfp8-nvfp4/)

**Performance (Flux.1 image generation):**
| Precision | Time |
|-----------|------|
| BF16 | 16.93 sec |
| MXFP8 | 8.96 sec |
| NVFP4 | 7.62 sec |

### Hardware Dependency

> *"FP4 is native to Blackwell GPUs only — NVIDIA B200, B300, RTX 5090, and RTX PRO 6000."*
> — [Spheron](https://www.spheron.network/blog/fp4-quantization-blackwell-gpu-cost/)

### Algorithmic Work Required

> *"The amount of algorithmic work we have been doing to ensure that we preserve accuracy as we go to that reduced precision has been a substantial amount of work, and it's ongoing work."*
> — Dave Salvator, NVIDIA Director of Accelerated Computing Products — [Constellation Research](https://www.constellationr.com/blog-news/insights/nvidia-highlights-algorithmic-research-it-moves-fp4)

---

## Key Findings

- **NVFP4 (E2M1)** — 4-bit floating point with dual-scaling (FP8 block scales + FP32 tensor scales) enables near-lossless quantization
- **18,000 FP4 TOPS** (sparse) on B200 datacenter GPU — 2x FP8 throughput
- **≤1% accuracy degradation** in most benchmarks; QAT recovers to lossless
- **MLPerf v5.1**: B200 FP4 achieves ~102,725 tokens/sec (8-GPU) — 2.9x faster than H200 FP8
- **Tools**: TensorRT Model Optimizer, LLM Compressor, TensorRT-LLM, Intel Neural Compressor all support NVFP4
- **Hardware-locked**: FP4 requires Blackwell (B200/B300/RTX 5090) — not accessible on Hopper

---

## Related Topics

- [CG-HBM](./CG-HBM.md) — HBM memory bandwidth on Blackwell (8 TB/s HBM3e)
- [DGX-Spark](./DGX-Spark.md) — Personal AI supercomputer with GB10 Blackwell chip
