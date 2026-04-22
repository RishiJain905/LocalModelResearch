# GEMM and MatMul Kernel Optimization

> *The fundamental building blocks of AI matrix computation — covering GEMM theory, CUDA/ROCm/oneDNN/MLX kernel optimization, Flash Attention, and benchmark data across NVIDIA, AMD, Intel, and Apple Silicon.*

---

## Table of Contents

1. [Definitions & Core Concepts](#1-definitions--core-concepts)
2. [GPU Memory Hierarchy](#2-gpu-memory-hierarchy)
3. [NVIDIA CUDA Stack](#3-nvidia-cuda-stack)
4. [AMD ROCm Stack](#4-amd-rocm-stack)
5. [Intel oneMKL & oneDNN](#5-intel-onemkl--onednn)
6. [Apple Silicon: Accelerate & MLX](#6-apple-silicon-accelerate--mlx)
7. [Flash Attention](#7-flash-attention)
8. [Kernel Fusion](#8-kernel-fusion)
9. [Automatic Kernel Generation](#9-automatic-kernel-generation)
10. [Benchmarks & Performance Data](#10-benchmarks--performance-data)
11. [Key Sources & Quotes](#11-key-sources--quotes)
12. [Summary Table](#12-summary-table)

---

## 1. Definitions & Core Concepts

### GEMM (General Matrix Multiply)

> *"GEMM is a common algorithm in linear algebra, machine learning, statistics, and many other domains. It computes the general matrix multiplication C = AB, where A, B, C are matrices."*
> — [Spatial-lang.org](https://spatial-lang.org/gemm)

> *"Dense matrix multiplications appear in the feed forward network layers and within the attention mechanism itself (for projecting queries, keys, and values)."*
> — [apxml.com — Optimized Kernels for LLM Layers](https://apxml.com/courses/llm-compression-acceleration/chapter-6-hardware-acceleration-systems-optimization/optimized-kernels-llm-layers)

### cuBLAS Definition

> *"To use the cuBLAS API, the application must allocate the required matrices and vectors in the GPU memory space, fill them with data, call the sequence of desired cuBLAS functions, and then upload the results from the GPU memory space back to the host."*
> — [NVIDIA cuBLAS Documentation](https://docs.nvidia.com/cuda/cublas/)

### Memory Layout: Row-Major vs Column-Major

> *"cuBLAS, the CUDA library for linear algebra, requires that the first matrix be in row-major format, and the second matrix be in column-major format, as this is the layout that is most efficient for coalesced memory accesses."*
> — [Justin Smallwood](https://jsmallwood.net/2025/08/13/row-major-vs-column-major-memory-layouts/)

### NHWC vs NCHW Memory Access

> *"When we represent the tensors in NHWC format the access locations are contiguous and sure to be a cache hit. For the first time accessing leads to a cache miss and a transaction from DRAM fetching 32/128 bytes of data. While accessing the next element it will be a cache hit saving a transaction."*
> — [Medium — NHWC vs NCHW](https://medium.com/@deepika_writes/nhwc-vs-nchw-a-memory-access-perspective-on-gpus-4e79bd3b1b54)

---

## 2. GPU Memory Hierarchy

| Memory Type | Location | Latency | Bandwidth | Capacity | Scope |
|------------|----------|---------|-----------|----------|-------|
| **Register** | On-chip | ~1 cycle | Highest | 255 per thread | Thread |
| **Shared Memory** | On-chip | ~5 cycles | ~19 TB/s | 48–164 KB/SM | Block |
| **L1 Cache** | On-chip | ~28 cycles | ~19 TB/s | 128 KB/SM | SM |
| **L2 Cache** | Off-chip | ~193 cycles | ~7 TB/s | Tens of MB | Device |
| **Global (HBM)** | HBM | ~600 cycles | ~2 TB/s | Tens of GB | Device |

> *"The tensor cores consume data far faster than memory can supply it. The goal of GEMM optimization is figuring out how to copy numbers fast enough to keep tensor cores busy."*
> — [SIGARCH — Efficient GEMM Kernel Designs with Pipelining](https://www.sigarch.org/efficient-gemm-kernel-designs-with-pipelining/)

### Warp Divergence

> *"A warp is a group of 32 threads that execute in lockstep under SIMT. When a conditional branch splits threads within a warp onto different paths, both paths execute serially — this is called warp divergence."*
> — [YoungJu Dev Blog — CUDA GPU Programming](https://www.youngju.dev/blog/gpu-cuda/2026-03-17-cuda-gpu-programming-advanced-guide.en)

### TMA (Tensor Memory Accelerator)

> *"Compute capability 9.0 (The NVIDIA Hopper GPU architecture) extended the asynchronous execution features with the Tensor Memory Accelerator (TMA) unit, which can transfer large blocks of data and multidimensional tensors from global memory to shared memory and vice versa."*
> — [NVIDIA CUDA Programming Guide](https://docs.nvidia.com/cuda/cuda-programming-guide/03-advanced/advanced-kernel-programming.html?hl=en-US)

---

## 3. NVIDIA CUDA Stack

### cuBLAS

| Routine | Use Case |
|---------|----------|
| **cublasGemmEx** | General matrix multiply with flexible data types |
| **cublasLt** | Low-level GEMM with more control over heuristics |
| **cublasSgemm** | FP32 matrix multiply |
| **cublasDgemm** | FP64 matrix multiply |
| **cublasHgemm** | FP16 matrix multiply |

### CUTLASS

> *"CUTLASS is a collection of CUDA C++ template abstractions for implementing high-performance matrix-multiplication (GEMM) at all levels and scales within CUDA."*
> — [NVIDIA CUTLASS GitHub](https://github.com/NVIDIA/cutlass)

**Key features:**
- Templates for all precision types (FP16, BF16, TF32, FP64, INT8, FP8)
- Persistent kernels for Hopper (Ping-Pong GEMM)
- WGMMA (Warp Group Matrix Multiply Accumulate) support
- Python bindings for rapid experimentation

### CUTLASS Ping-Pong GEMM

> *"Ping-Pong is one of the fastest matmul (GEMM) kernel architectures available for the Hopper GPU architecture (H100)."*
> — [PyTorch Blog — CUTLASS Ping-Pong GEMM](https://pytorch.org/blog/cutlass-ping-pong-gemm-kernel/)

> *"When the H100 (Hopper) GPU was launched, Nvidia billed it as the first truly asynchronous GPU."*
> — [PyTorch Blog — CUTLASS Ping-Pong GEMM](https://pytorch.org/blog/cutlass-ping-pong-gemm-kernel/)

**Ping-Pong architecture splits work across two warp groups:**
- **Producer warp group**: Loads data via TMA into shared memory
- **Consumer warp group**: Performs matmul on data in shared memory
- Both groups operate concurrently, overlapping load and compute

### Transformer Engine

- FP8 training support via `.transformer_engine`
- Automatic mixed precision with BF16/FP8
- Integrates with DeepSpeed and Megatron

---

## 4. AMD ROCm Stack

### hipBLASLt

> *"hipBLASLt is a library that provides GEMM operations with flexible APIs and extends functionality beyond the traditional BLAS library. The library adds flexibility for matrix data layouts, input types, and compute types and for choosing the algorithmic implementations and heuristics through parameter programmability."*
> — [AMD hipBLASLt](https://rocm.docs.amd.com/projects/hipBLASLt/en/latest/what-is-hipBLASLt.html)

### ROCm GEMM Blog

> *"Matrix multiplication underlies critical computational pathways in AI. General Matrix Multiplication (GEMM) operations serve as performance-critical kernels in neural network architectures — from fully connected layers to convolutions and transformer attention mechanisms."*
> — [AMD ROCm Blog — GEMM Kernel Optimization](https://rocm.blogs.amd.com/artificial-intelligence/gemm_blog/README.html)

**Four tuning techniques from AMD:**
1. **Pre-Tuned Docker**: Use AMD's pre-configured containers
2. **PyTorch TunableOp**: Automatic kernel selection at runtime
3. **rocblas-gemm-tune**: Command-line tuning utility
4. **hipblaslt-bench**: Benchmarking tool for kernel selection

> *"In experiments, this approach yielded over 20% performance improvement in GEMM operations."*
> — [AMD ROCm Blog](https://rocm.blogs.amd.com/artificial-intelligence/gemm_blog/README.html)

### Tensile

- Kernel generation and tuning for AMD GPUs
- Produces optimized assembly kernels for each unique problem size
- Replaces hand-tuned kernels with auto-tuned ones

---

## 5. Intel oneMKL & oneDNN

### AMX (Advanced Matrix Extensions)

> *"oneMKL for instance directly leverages Intel® Advanced Matrix Extensions (Intel® AMX) to optimize matrix multiply computations for BF16 and INT8 data types."*
> — [Intel — oneMKL Performance Benchmarks](https://www.intel.com/content/www/us/en/developer/articles/technical/performance-benchmarks-onemkl-on-xeon-processors.html)

> *"The AMX-FP16 support being introduced there."*
> — [Phoronix — Intel Xeon AMX Review](https://www.phoronix.com/review/intel-xeon-amx/2)

**Intel AMX Available ISA Levels:**

| ISA Level | Data Type | Use Case |
|-----------|-----------|----------|
| **AVX512_CORE_AMX** | BF16, INT8 | AI inference/training |
| **AVX512_CORE_FP16** | FP16 | AI inference |
| **AVX512_CORE_BF16** | BF16 | AI training |
| **AVX512_CORE_VNNI** | INT8 | AI inference |

### oneDNN 3.0

> *"oneDNN 3.0: Released with improved Sapphire Rapids performance and early Granite Rapids support including with the AMX-FP16 support being introduced there."*
> — [Phoronix](https://www.phoronix.com/review/intel-xeon-amx/2)

### Xeon CPU Cores for AI

| CPU | Cores | AMX Support | oneDNN Optimization |
|-----|-------|-------------|-------------------|
| **Xeon Platinum 8490H** | 120 (dual socket) | AMX-BF16/INT8 | oneDNN 3.0 |
| **Xeon Max 9480** | 56 + HBM | AMX-BF16 | oneDNN + HBM |
| **Xeon w9-3595X** | 56 (Workstation) | AMX-BF16/FP16 | oneDNN |

---

## 6. Apple Silicon: Accelerate & MLX

### Accelerate Framework

> *"The Accelerate framework provides high-performance, energy-efficient computation on the CPU by leveraging its vector-processing capability. It performs optimized large-scale mathematical computations and image calculations for: Machine learning, Data compression, Signal processing."*
> — [Apple Developer — Accelerate](https://developer.apple.com/accelerate/)

**Key components:**

| Component | Focus | URL |
|-----------|-------|-----|
| **vDSP** | Vector/signal processing, FFT, convolution | [Apple Docs](https://developer.apple.com/documentation/accelerate/vdsp) |
| **BLAS** | Basic Linear Algebra Subprograms | [Apple Docs](https://developer.apple.com/documentation/accelerate/blas) |
| **BNNS** | Batched Neural Network Subroutines | [Apple Docs](https://developer.apple.com/documentation/accelerate/bnns-library/) |
| **LAPACK** | Linear algebra package | [Apple Docs](https://developer.apple.com/documentation/accelerate/lapack) |

### BNNS (Batched Neural Network Subroutines)

> *"The BNNS functions enable you to perform forward and backward passes of neural networks on CPU, using inference and training functions."*
> — [Apple BNNS](https://developer.apple.com/documentation/accelerate/bnns-library/)

### MLX

> *"MLX is an array framework for machine learning on Apple silicon, brought to you by Apple machine learning research."*
> — [Apple MLX GitHub](https://github.com/ml-explore/mlx)

> *"The GPU Neural Accelerators shine with MLX on ML workloads involving large matrix multiplications, yielding up to 4x speedup compared to a M4 baseline for time-to-first-token in language model inference."*
> — [Apple ML Research — Exploring LLMs with MLX and M5](https://machinelearning.apple.com/research/exploring-llms-mlx-m5)

**MLX key features:**
- Lazy computation (operations not executed until needed)
- Unified memory model (arrays shared across CPU/GPU/Neural Engine)
- NumPy-compatible API
- Automatic differentiation for training

---

## 7. Flash Attention

### FlashAttention (2022)

> *"We propose FlashAttention, an IO-aware exact attention algorithm that uses tiling to reduce the number of memory reads/writes between GPU high bandwidth memory (HBM) and GPU on-chip SRAM."*
> — [NeurIPS 2022 — FlashAttention Paper](https://openreview.net/forum?id=H4DqfPSibmx)

**Authors:** Tri Dao, Daniel Y. Fu, Stefano Ermon, Atri Rudra, Christopher Ré

**Results:**
- 3× speedup on GPT-2 (seq length 1K)
- 2.4× speedup on Long Range Arena (seq length 1K–4K)

### FlashAttention-2 (2024)

> *"FlashAttention-2 reaches 50-73% of theoretical max throughput on A100 GPUs, achieving up to 225 TFLOPs/s per A100 GPU (72% model FLOPs utilization) in end-to-end GPT training."*
> — [FlashAttention-2 Paper](https://tridao.me/publications/flash2/flash2.pdf)

> *"The matmul throughput can be up to 16× higher than non-matmul throughput on A100 GPUs (312 TFLOPs/s matmul vs 19.5 TFLOPs/s non-matmul FP32)."*
> — [FlashAttention-2 Paper](https://tridao.me/publications/flash2/flash2.pdf)

### FlashAttention-3 (Hopper FP8)

> *"FlashAttention-3 incorporates key techniques to achieve 1.5–2.0x faster performance than FlashAttention-2 with FP16, up to 740 TFLOPS. With FP8, FlashAttention-3 reaches up to 1.2 PFLOPS."*
> — [PyTorch Blog — FlashAttention-3](https://pytorch.org/blog/flashattention-3/)

### FlashAttention Performance Comparison (A100)

| Configuration | PyTorch | FlashAttention | FlashAttention-2 |
|---------------|---------|----------------|------------------|
| No causal, d=64, seq=8k | 86 TFLOPs/s | 110 TFLOPs/s | **176 TFLOPs/s** |
| No causal, d=128, seq=8k | OOM | 72 TFLOPs/s | **203 TFLOPs/s** |
| Causal, d=64, seq=8k | 92 TFLOPs/s | 97 TFLOPs/s | **165 TFLOPs/s** |
| Causal, d=128, seq=8k | OOM | 80 TFLOPs/s | **189 TFLOPs/s** |

### FlashAttention-3 Performance (H100)

| Configuration | Performance |
|---------------|-------------|
| FP16 forward pass | ~540–570 TFLOPS (vs 350 in FA2) |
| FP16 maximum | 740 TFLOPS (~75% of H100 theoretical max) |
| FP8 | Up to 1.2 PFLOPS |
| FP8 error | 2.6× smaller than baseline FP8 attention |

---

## 8. Kernel Fusion

### Fusion Patterns

| Pattern | Description | Example |
|---------|-------------|---------|
| **GEMM + Activation** | Fuse matmul with ReLU, GELU, SiLU, Swish | GEMM → GELU fused |
| **Attention fusion** | Combine softmax with scaling and masking | Scale × Softmax fused |
| **LayerNorm fusion** | Fuse add, mean, variance, rsqrt, div, multiply | Single CUDA kernel |
| **Residual add + LayerNorm** | Combine elementwise add with normalization | f(x) = LayerNorm(x + sublayer(x)) |

### CUTLASS Kernel Fusion

- Fuses elementwise operations into GEMM kernels
- Supports custom activation functions via `cutlass::epilogue::`
- Reduces memory traffic by eliminating intermediate writes

### ONNX Runtime Fusion

> *"ONNX Runtime applies graph-level optimizations including node fusion, constant folding, and operator fusion to reduce computational overhead."*
> — [ONNX Runtime](https://onnxruntime.ai/docs/)

---

## 9. Automatic Kernel Generation

### TVM (Apache TVM)

- Machine learning compiler for CPU/GPU/NPU
- Auto-tuning with TVMC (TVM Compiler)
- Relay IR for graph-level optimization
- Supports NVIDIA, AMD, Intel, ARM, RISC-V

### XLA (Accelerated Linear Algebra)

> *"XLA (Accelerated Linear Algebra) is a domain-specific, hardware-agnostic compiler. Uses whole-program analysis to fuse operators and optimize memory layouts."*
> — [Google Developers Blog](https://developers.googleblog.com/building-production-ai-on-google-cloud-tpus-with-jax/)

### CUTLASS Template Instantiation

- Explicit template parameters for problem size, tile dimensions, stages
- Warp-level tile shape selection
- Datatype and math operator selection

### FlexFlow

> *"FlexFlow is a DNN framework for automatic parallelization strategy discovery — targeting data, model, and pipeline parallelism across dimensions."*
> — [Carnegie Mellon FlexFlow](https://catalyst.cs.cmu.edu/projects/flexflow.html)

---

## 10. Benchmarks & Performance Data

### NVIDIA H100 GPU Performance

| Precision | H100 SXM5 | H100 PCIe | Use Case |
|-----------|-----------|-----------|----------|
| **FP8 Tensor** | ~989 TFLOPS | ~850 TFLOPS | LLMs, Transformers |
| **FP16 Tensor** | ~495 TFLOPS | ~204.9 TFLOPS | AI Training |
| **BF16 Tensor** | ~495 TFLOPS | ~204.9 TFLOPS | AI Training |
| **TF32** | 1 petaflop (matrix) | ~83 TFLOPS | Zero-code AI acceleration |
| **FP64** | 60 TFLOPS | ~25.61 TFLOPS | HPC, Scientific |

> *"The H100 delivers exceptional FLOPS performance across different precisions: up to 989 TFLOPS for FP8 Tensor operations, 495 TFLOPS for FP16, and 60 TFLOPS for FP64 computing. The H100 features a theoretical peak performance exceeding 1 ExaFLOP of AI compute power using FP8 precision."*
> — [Jarvis Labs AI FAQ](https://jarvislabs.ai/ai-faqs/what-is-the-flops-performance-of-the-nvidia-h100-gpu)

### NVIDIA A100 GPU Performance

| Specification | A100 80GB | A100 40GB |
|--------------|-----------|-----------|
| **FP16 Tensor FLOPS** | 312 TFLOPS | 312 TFLOPS (up to 624 with sparsity) |
| **Memory Bandwidth** | 2 TB/s | 1.6 TB/s |

### Training Speedup with H100 FP8 vs A100

| Model Size | 8x A100 BF16 | 8x H100 BF16 | 8x H100 FP8 | FP8 Speedup |
|------------|--------------|--------------|-------------|-------------|
| 1B | Baseline | ~2.1× | ~2.7× | ~1.3× |
| 7B | Baseline | ~2.2× | ~3.0× | ~1.4× |
| 13B | Baseline | ~2.1× | ~2.6× | ~1.2× |

### Memory Bandwidth vs Compute Ratio

| GPU | TFLOPS (FP16) | Bandwidth | Ratio (Lower = More Balanced) |
|-----|---------------|-----------|-------------------------------|
| RTX 4090 | 165 | 1,008 GB/s | 0.164 |
| H100 SXM | 1,000 | 3,350 GB/s | 0.298 |

---

## 11. Key Sources & Quotes

| Topic | Source | URL | Key Quote |
|-------|--------|-----|-----------|
| GEMM Definition | Spatial-lang.org | https://spatial-lang.org/gemm | *"GEMM computes C = AB, where A, B, C are matrices"* |
| GEMM in Neural Networks | apxml.com | https://apxml.com/courses/llm-compression-acceleration/chapter-6-hardware-acceleration-systems-optimization/optimized-kernels-llm-layers | *"Dense matrix multiplications appear in feed forward layers and attention"* |
| Memory Layouts | Justin Smallwood | https://jsmallwood.net/2025/08/13/row-major-vs-column-major-memory-layouts/ | *"cuBLAS requires row-major then column-major for coalesced access"* |
| NHWC Memory | Medium | https://medium.com/@deepika_writes/nhwc-vs-nchw-a-memory-access-perspective-on-gpus-4e79bd3b1b54 | *"NHWC format provides contiguous cache-hit access"* |
| CUDA Memory | YoungJu Dev Blog | https://www.youngju.dev/blog/gpu-cuda/2026-03-17-cuda-gpu-programming-advanced-guide.en | *"Warp is a group of 32 threads in lockstep SIMT"* |
| TMA | NVIDIA CUDA Guide | https://docs.nvidia.com/cuda/cuda-programming-guide/03-advanced/advanced-kernel-programming.html?hl=en-US | *"Hopper TMA transfers large blocks of data asynchronously"* |
| FlashAttention | NeurIPS 2022 | https://openreview.net/forum?id=H4DqfPSibmx | *"IO-aware exact attention using tiling"* |
| FlashAttention-2 | Tri Dao | https://tridao.me/publications/flash2/flash2.pdf | *"225 TFLOPs/s per A100 GPU (72% model FLOPs utilization)"* |
| FlashAttention-3 | PyTorch Blog | https://pytorch.org/blog/flashattention-3/ | *"Up to 1.2 PFLOPS with FP8"* |
| CUTLASS | NVIDIA | https://github.com/NVIDIA/cutlass | *"CUDA C++ templates for high-performance GEMM"* |
| Ping-Pong GEMM | PyTorch Blog | https://pytorch.org/blog/cutlass-ping-pong-gemm-kernel/ | *"One of the fastest kernels on Hopper architecture"* |
| GEMM Pipelining | SIGARCH | https://www.sigarch.org/efficient-gemm-kernel-designs-with-pipelining/ | *"Goal is to copy numbers fast enough to keep tensor cores busy"* |
| AMD GEMM | AMD ROCm Blog | https://rocm.blogs.amd.com/artificial-intelligence/gemm_blog/README.html | *"Over 20% performance improvement with tuning"* |
| hipBLASLt | AMD | https://rocm.docs.amd.com/projects/hipBLASLt/en/latest/what-is-hipBLASLt.html | *"Flexible GEMM APIs beyond traditional BLAS"* |
| Intel AMX | Intel | https://www.intel.com/content/www/us/en/developer/articles/technical/performance-benchmarks-onemkl-on-xeon-processors.html | *"oneMKL leverages AMX for BF16 and INT8"* |
| Apple Accelerate | Apple | https://developer.apple.com/accelerate/ | *"High-performance CPU computation leveraging vector-processing"* |
| Apple MLX | Apple ML Research | https://machinelearning.apple.com/research/exploring-llms-mlx-m5 | *"4x speedup vs M4 for LLM inference time-to-first-token"* |
| XLA | Google | https://developers.googleblog.com/building-production-ai-on-google-cloud-tpus-with-jax/ | *"Domain-specific, hardware-agnostic compiler"* |
| H100 Specs | Jarvis Labs | https://jarvislabs.ai/ai-faqs/what-is-the-flops-performance-of-the-nvidia-h100-gpu | *"989 TFLOPS FP8, 495 TFLOPS FP16, 1 ExaFLOP FP8"* |

---

## 12. Summary Table

| Aspect | NVIDIA | AMD | Intel | Apple |
|--------|--------|-----|-------|-------|
| **Primary Library** | cuBLAS, CUTLASS | hipBLASLt, rocBLAS | oneMKL, oneDNN | Accelerate, MLX |
| **Key GPU Feature** | TMA, WGMMA, Tensor Cores | CDNA 3, Tensile | AMX (BF16, INT8) | Neural Engine |
| **Auto-Tuning** | CUTLASS templates | Tensile | oneDNN heuristics | MLX lazy eval |
| **Flash Attention** | FA2, FA3, FA4 | Via ROCm | Via oneDNN | Via MLX |
| **FP8 Support** | H100 native | MI300X | Granite Rapids | Via MLX |
| **HBM Bandwidth** | 3.35 TB/s (H100) | 5.3 TB/s (MI300X) | HBM on Xeon Max | Unified memory |
| **Kernel Fusion** | CUTLASS, TRT | ROCm fusion | oneDNN fusion | MLX fusion |

### Key Performance Numbers

| GPU | FP16 Tensor TFLOPS | FP8 Tensor TFLOPS | Memory Bandwidth |
|-----|-------------------|------------------|------------------|
| **H100 SXM5** | 495 | 989 | 3.35 TB/s |
| **H100 PCIe** | 204.9 | 850 | 2.04 TB/s |
| **A100 80GB** | 312 | N/A | 2 TB/s |
| **MI300X** | 1,307 | 2,614 | 5.3 TB/s |
| **Apple M4 Max** | ~60 (GPU) | N/A | ~500 GB/s (unified) |

---

*Sources compiled from: NVIDIA CUDA/CUTLASS/Transformer Engine docs, AMD ROCm Blog, Intel Developer Articles, Apple ML Research, Tri Dao FlashAttention papers, PyTorch Blog, YoungJu Dev Blog, SIGARCH, Jarvis Labs AI FAQ, Google Developers Blog*
