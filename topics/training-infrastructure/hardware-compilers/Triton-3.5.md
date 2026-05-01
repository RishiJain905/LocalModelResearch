# Triton 3.5 Compiler for GPU ML Training Kernels

## Table of Contents
1. [Overview](#overview)
2. [Triton Python DSL](#triton-python-dsl)
3. [Fused Kernel Generation](#fused-kernel-generation)
4. [FlashAttention Integration](#flashattention-integration)
5. [FP8 Support in Triton 3.5](#fp8-support-in-triton-35)
6. [GEMM Kernel Optimization](#gemm-kernel-optimization)
7. [Comparison to CUDA/CUTLASS](#comparison-to-cudacutlass)
8. [TMA (Tensor Memory Accelerator)](#tma-tensor-memory-accelerator)
9. [Hopper/Blackwell Architecture Support](#hopperblackwell-architecture-support)
10. [Triton for LLM Inference (vLLM Integration)](#triton-for-llm-inference-vllm-integration)
11. [Resources and References](#resources-and-references)

---

## Overview

> "Triton is an open-source language and compiler for writing highly efficient custom Deep-Learning primitives. It aims to provide higher productivity than CUDA and higher flexibility than other existing DSLs."
> — [Triton GitHub Repository](https://github.com/triton-lang/triton)

| Aspect | Details |
|--------|---------|
| **Current Version (as of Oct 2025)** | 3.5.0 (October 13, 2025) |
| **Repository** | [github.com/triton-lang/triton](https://github.com/triton-lang/triton) |
| **Documentation** | [triton-lang.org](https://triton-lang.org/) |
| **Primary Maintainer** | OpenAI (Philippe Tillet, creator) |
| **License** | MIT |

**Version History (Recent):**
| Version | Release Date | Key Features |
|---------|--------------|--------------|
| 3.6.0 | Jan 20, 2026 | GFX1250 (RDNA4) support, `tl.trans`, `tl.dot`, `dot_scaled` |
| 3.5.1 | Nov 11, 2025 | sm103 (GB300) bug fix |
| 3.5.0 | Oct 13, 2025 | `tl.device_assert`, Blackwell support, `PaddedSharedEncoding`, FP8 support |
| 3.4.0 | Jul 30, 2025 | Gluon framework overhaul, async TMA, `static_assert`, TensorDescriptor |

---

## Triton Python DSL

> "Triton is Not CUDA in Python — It's a Tiling DSL. The compiler handles all of that based on the tile-level operations you describe."
> — [Medium Article: Triton Is Not CUDA in Python](https://medium.com/@varuntej07/triton-is-not-cuda-in-python-its-a-tiling-dsl-c65c15ce3c46)

### Core Concept: Tile-Based Programming

> "We present Triton, a language and compiler centered around the concept of tile, i.e., statically shaped multi-dimensional sub-arrays."
> — [Triton MAPL 2019 Paper](https://www.eecs.harvard.edu/~htk/publication/2019-mapl-tillet-kung-cox.pdf)

### Key Language Features

**Installation:**
```bash
pip install triton  # Binary wheels for CPython 3.10-3.14
```

**Basic Kernel Structure:**
```python
import triton
import triton.language as tl

@triton.jit
def fused_softmax_kernel(output_ptr, input_ptr, input_row_stride, output_row_stride, n_cols, BLOCK_SIZE: tl.constexpr):
    # Each program handles one row
    row_idx = tl.program_id(0)
    row_start_ptr = input_ptr + row_idx * input_row_stride
    
    # Load row with multiple blocks if needed
    col_offsets = tl.arange(0, BLOCK_SIZE)
    input_ptrs = row_start_ptr + col_offsets
    mask = col_offsets < n_cols
    
    # Load row
    row = tl.load(input_ptrs, mask=mask, other=-float('inf'))
    
    # Compute softmax (fused)
    row_max = tl.max(row, axis=0)
    row = row - row_max  # Numerical stability
    numerator = tl.exp(row)
    denominator = tl.sum(numerator, axis=0)
    softmax_output = numerator / denominator
    
    # Store result
    output_row_start_ptr = output_ptr + row_idx * output_row_stride
    output_ptrs = output_row_start_ptr + col_offsets
    tl.store(output_ptrs, softmax_output, mask=mask)
```

### Triton vs Traditional GPU Programming

| Feature | Triton | CUDA/C++ |
|---------|--------|----------|
| **Programming Model** | Tile-based DSL | Thread-based |
| **Memory Management** | Compiler handles shared memory | Manual |
| **Thread Layout** | 128-thread groups (warps) | Explicit warp/thread |
| **Performance Tuning** | `@triton.autotune` | Manual PTX/SASS |
| **Learning Curve** | Python-first, simpler | Steep |

### Autotuning Example

```python
@triton.autotune(
    configs=[
        triton.Config({'BLOCK_SIZE': 1024}, num_warps=4),
        triton.Config({'BLOCK_SIZE': 2048}, num_warps=8),
        triton.Config({'BLOCK_SIZE': 4096}, num_warps=8),
    ],
    key=['n_cols'],  # Retune when n_cols changes
)
@triton.jit
def fused_softmax_kernel(...):
    # Kernel implementation
```

### Reference Papers
- [Triton: An Intermediate Language and Compiler for Tiled Neural Network Computations (MAPL 2019)](https://www.eecs.harvard.edu/~htk/publication/2019-mapl-tillet-kung-cox.pdf)
- [IBM Research Publication](https://research.ibm.com/publications/triton-an-intermediate-language-and-compiler-for-tiled-neural-network-computations)

---

## Fused Kernel Generation

> "Fused attention kernels eliminate these inefficiencies by fusing the operations of score computation, online softmax (normalization), and output projection into a single, tile-wise, pipelined routine."
> — [Emergentmind: Fused Attention Kernel](https://www.emergentmind.com/topics/fused-attention-kernel)

### What is Kernel Fusion?

Kernel fusion combines multiple operations into a single GPU kernel, eliminating memory traffic between operations. This is critical for:
- Reducing HBM (High Bandwidth Memory) traffic
- Enabling inline computations without materializing intermediates
- Achieving near-peak hardware utilization

### Triton Fusion Capabilities

| Operation Type | Fusion Support |
|----------------|----------------|
| Element-wise ops | ✅ Native |
| Reductions | ✅ Native |
| Softmax | ✅ Fused in single kernel |
| LayerNorm | ✅ Fused |
| Attention | ✅ FlashAttention integration |
| GEMM + Activation | ✅ Fused |

### Fused Softmax Example

```python
# Fuses: load → exp → sum → divide → store
@triton.jit
def fused_softmax_kernel(output_ptr, input_ptr, n_cols, BLOCK_SIZE: tl.constexpr):
    row_idx = tl.program_id(0)
    row_start_ptr = input_ptr + row_idx * n_cols
    col_offsets = tl.arange(0, BLOCK_SIZE)
    mask = col_offsets < n_cols
    
    row = tl.load(row_start_ptr + col_offsets, mask=mask, other=-float('inf'))
    row_max = tl.max(row, axis=0)
    row = row - row_max  # numerical stability
    numerator = tl.exp(row)
    denominator = tl.sum(numerator, axis=0)
    softmax_output = numerator / denominator
    
    output_ptrs = row_start_ptr + col_offsets
    tl.store(output_ptrs, softmax_output, mask=mask)
```

---

## FlashAttention Integration

> "This is a Triton implementation of the Flash Attention v2 algorithm from Tri Dao."
> — [Triton GitHub Tutorial](https://github.com/triton-lang/triton/blob/main/python/tutorials/06-fused-attention.py)

### Triton FlashAttention Features

| Feature | Description |
|---------|-------------|
| **Algorithm** | Flash Attention v2 (online softmax) |
| **Memory Complexity** | O(N) instead of O(N²) |
| **Hardware Support** | NVIDIA Hopper, Blackwell; AMD CDNA/RDNA |
| **Data Types** | FP16, FP8 (float8_e5m2) |
| **Attention Modes** | Causal (autoregressive), non-causal |

### FlashAttention Kernel Architecture

**Forward Pass Key Structure:**
```python
@triton.autotune(configs=list(filter(keep, configs)), 
                 key=["N_CTX", "HEAD_DIM", "FP8_OUTPUT", "warp_specialize"],
                 prune_configs_by={'early_config_prune': prune_invalid_configs})
@triton.jit
def _attn_fwd(sm_scale, M, ...):
    # Initialize accumulator, log-sum-exp, and max
    m_i = tl.zeros([BLOCK_M], dtype=tl.float32) - float("inf")
    l_i = tl.zeros([BLOCK_M], dtype=tl.float32) + 1.0
    acc = tl.zeros([BLOCK_M, HEAD_DIM], dtype=tl.float32)
    
    # Two stages: off-band and on-band for causal attention
    if STAGE & 1:
        acc, l_i, m_i = _attn_fwd_inner(...)
    if STAGE & 2:
        acc, l_i, m_i = _attn_fwd_inner(...)
```

### Supported Configurations

```python
HEAD_DIM: [16, 32, 64, 128, 256]
BLOCK_M: [64, 128]
BLOCK_N: [32, 64, 128]
NUM_STAGES: [1] for HIP, [2, 3, 4] for others
NUM_WARPS: [4, 8]
```

### Performance Characteristics

> "Every major fused kernel (FlashAttention-2, FlexAttention, Flashlight) achieves 2–4× speedup at scale, up to 1.22–2.04× end-to-end LLM throughput gains."
> — [Emergentmind: Fused Attention Kernel](https://www.emergentmind.com/topics/fused-attention-kernel)

### Repository References

- [Official Triton FlashAttention Tutorial](https://github.com/triton-lang/triton/blob/main/python/tutorials/06-fused-attention.py)
- [Dao-AILab/flash-attention](https://github.com/dao-ailab/flash-attention) - Original FlashAttention with Triton support for AMD GPUs

---

## FP8 Support in Triton 3.5

> "The NVIDIA Blackwell architecture introduces a new Tensor Core that improves throughput and energy efficiency for matrix multiplications, with Triton's compiler infrastructure automatically exploiting these capabilities for FP8 and FP16 GEMM operations."
> — [NVIDIA Technical Blog: OpenAI Triton on NVIDIA Blackwell](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)

### FP8 Data Types

| Format | Description | Hardware Support |
|--------|-------------|------------------|
| **FP8 (float8_e5m2)** | 5-bit exponent, 2-bit mantissa | Hopper, Blackwell |
| **FP8 (float8_e4m3)** | 4-bit exponent, 3-bit mantissa | Blackwell |
| **MXFP8** | Block-scaled FP8 (microscaling) | Blackwell |
| **MXFP4** | Block-scaled FP4 | Blackwell |

### Triton FP8 Support in 3.5.0

**Features added in Triton 3.5.0:**
- `tl.float8_e5m2` - Standard FP8 e5m2 format
- `tl.float8_e4m3` - Standard FP8 e4m3 format (Blackwell)
- MXFP8/MXFP4 block scaling support via `dot_scaled`
- `PaddedSharedEncoding` optimization for FP8 GEMMs

### FP8 GEMM Example

```python
@triton.autotune(configs=[
    triton.Config({'BLOCK_SIZE_M': 128, 'BLOCK_SIZE_N': 256, 'BLOCK_SIZE_K': 64, 'GROUP_SIZE_M': 8}, num_stages=3, num_warps=8),
    # ... more fp8 configs
])
@triton.jit
def matmul_kernel(a_ptr, b_ptr, c_ptr, ...):
    # Load FP8 tensors
    a = tl.load(a_ptrs, mask=offs_k[None, :] < K - k * BLOCK_SIZE_K, other=0.0, dtype=tl.float8_e5m2)
    b = tl.load(b_ptrs, mask=offs_k[:, None] < K - k * BLOCK_SIZE_K, other=0.0, dtype=tl.float8_e5m2)
    
    # FP8 dot product with accumulation
    accumulator = tl.dot(a, b, accumulator)
```

### Blackwell FP8 Performance

> "Figure 1 shows that Triton optimizations on NVIDIA Blackwell architecture bring hardware performance improvements to users in both FP16 and FP8 in this K sweep analysis for a typical generative AI size of GEMM kernel."
> — [NVIDIA Technical Blog](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)

### Block-Scaled Formats (MXFP8/MXFP4)

> "The formats supported in the tutorial are the OCP microscaling formats, including mxfp4 and mxfp8, as well as NVIDIA's nvfp4 format."
> — [Hacker News: New microscaling data formats supported in Blackwell and Triton](https://news.ycombinator.com/item?id=43006285)

---

## GEMM Kernel Optimization

> "The kernel that we will write will implement the following blocked algorithm to multiply a (M, K) by a (K, N) matrix."
> — [Triton Documentation: Matrix Multiplication Tutorial](https://triton-lang.org/main/getting-started/tutorials/03-matrix-multiplication.html)

### Blocked GEMM Algorithm

```python
# Do in parallel for m in range(0, M, BLOCK_SIZE_M):
# Do in parallel for n in range(0, N, BLOCK_SIZE_N):
    acc = zeros((BLOCK_SIZE_M, BLOCK_SIZE_N), dtype=float32)
    for k in range(0, K, BLOCK_SIZE_K):
        a = A[m: m + BLOCK_SIZE_M, k: k + BLOCK_SIZE_K]
        b = B[k: k + BLOCK_SIZE_K, n: n + BLOCK_SIZE_N]
        acc += dot(a, b)
    C[m: m + BLOCK_SIZE_M, n: n + BLOCK_SIZE_N] = acc
```

### Key Optimization Techniques

#### 1. L2 Cache Optimization (Grouped Ordering)

> "In a 9×9 block matrix: Row-major ordering requires loading 90 blocks to compute first 9 output blocks. Grouped ordering: Only 54 blocks. This can improve performance by >10% on some hardware."
> — [Triton Documentation](https://triton-lang.org/main/getting-started/tutorials/03-matrix-multiplication.html)

```python
GROUP_SIZE_M = 8
num_pid_in_group = GROUP_SIZE_M * num_pid_n
group_id = pid // num_pid_in_group
first_pid_m = group_id * GROUP_SIZE_M
group_size_m = min(num_pid_m - first_pid_m, GROUP_SIZE_M)
pid_m = first_pid_m + ((pid % num_pid_in_group) % group_size_m)
pid_n = (pid % num_pid_in_group) // group_size_m
```

#### 2. Pointer Arithmetic

```python
# Block pointer initialization for A and B
offs_am = (pid_m * BLOCK_SIZE_M + tl.arange(0, BLOCK_SIZE_M)) % M
offs_bn = (pid_n * BLOCK_SIZE_N + tl.arange(0, BLOCK_SIZE_N)) % N
offs_k = tl.arange(0, BLOCK_SIZE_K)

a_ptrs = a_ptr + (offs_am[:, None] * stride_am + offs_k[None, :] * stride_ak)
b_ptrs = b_ptr + (offs_k[:, None] * stride_bk + offs_bn[None, :] * stride_bn)
```

#### 3. Auto-Tuning Configuration

```python
@triton.autotune(
    configs=[
        triton.Config({'BLOCK_SIZE_M': 128, 'BLOCK_SIZE_N': 256, 'BLOCK_SIZE_K': 64, 'GROUP_SIZE_M': 8}, num_stages=3, num_warps=8),
        triton.Config({'BLOCK_SIZE_M': 64, 'BLOCK_SIZE_N': 256, 'BLOCK_SIZE_K': 32, 'GROUP_SIZE_M': 8}, num_stages=4, num_warps=4),
        triton.Config({'BLOCK_SIZE_M': 128, 'BLOCK_SIZE_N': 128, 'BLOCK_SIZE_K': 32, 'GROUP_SIZE_M': 8}, num_stages=4, num_warps=4),
        # ... more configs
    ],
    key=['M', 'N', 'K']
)
@triton.jit
def matmul_kernel(...):
    ...
```

### Persistent Matmul (Advanced)

> "The persistent matmul kernel uses the TMA descriptor API to efficiently map data to the 2D tile and minimize host-device synchronization."
> — [Triton Tutorial: Persistent Matmul](https://triton-lang.org/main/getting-started/tutorials/09-persistent-matmul.html)

Features:
- Tensor descriptor support for efficient memory access
- Cooperative clusters for multi-block cooperation
- TMA-based loading for optimal memory throughput

---

## Comparison to CUDA/CUTLASS

> "Triton is easier to program at the cost of not having as much control over how the GPU actually executes your code (in terms of PTX/SASS)."
> — [Reddit: Triton VS Cutlass VS Cuda](https://www.reddit.com/r/CUDA/comments/1e4jf2j/triton_vs_cutlass_vs_cuda/)

### Feature Comparison

| Feature | Triton | CUDA (native) | CUTLASS |
|---------|--------|---------------|---------|
| **Programming Language** | Python DSL | C++/CUDA C | C++/CUDA |
| **Abstraction Level** | Tile-level | Thread-level | Template-based |
| **Code Portability** | NVIDIA + AMD | NVIDIA only | NVIDIA only |
| **Learning Curve** | Low | High | Medium-High |
| **Customization** | Limited | Full | High |
| **Performance Tuning** | Autotune | Manual | Manual |
| **Flexibility** | DSL constraints | Unlimited | Template constraints |

### Performance Considerations

> "While the optimization now uses IMAD instead of HFMA2.MMA to move constants. When ptxas sees matrix operations (MAD/MMA): Instruction selection changes, and as a side effect, may increase register pressure."
> — [Henry Zhu: Maybe consider putting "cutlass" in your CUDA/Triton kernels](https://maknee.github.io/blog/2025/Maybe-Consider-Putting-Cutlass-In-Your-CUDA-Kernels/)

### When to Use Each

| Use Case | Recommended Tool |
|----------|-------------------|
| Quick prototyping | Triton |
| Custom fused kernels (attention, softmax) | Triton |
| Production-grade GEMM | CUTLASS or cuBLAS |
| Maximum hardware control | CUDA/CUTLASS |
| Multi-GPU workloads | Triton (with caveats) |

### CUTLASS Integration with Triton

> "For future research, we plan to improve upon these results, by working with the community to incorporate the CUTLASS architecture of TMA loads into Triton as well as investigating the Cooperative Kernel for FP8 GEMM."
> — [PyTorch Blog: Hopper TMA Unit for FP8 GEMMs](https://pytorch.org/blog/hopper-tma-unit/)

---

## TMA (Tensor Memory Accelerator)

> "The Hopper (H100) GPU architecture is NVIDIA's 'first truly asynchronous GPU,' featuring a new Tensor Memory Accelerator (TMA) hardware unit for bulk data movement between global and shared memory."
> — [PyTorch Blog: Hopper TMA Unit for FP8 GEMMs](https://pytorch.org/blog/hopper-tma-unit/)

### What is TMA?

TMA enables asynchronous, bi-directional transfers of 1D-5D tensors between GPU global and shared memory (GMEM ↔ SMEM), offloading address computation from CUDA cores to dedicated hardware.

### TMA Key Capabilities

| Feature | Description |
|---------|-------------|
| **Multicast** | Can transfer data to multiple SMs within a Thread Block Cluster |
| **Lightweight Invocation** | Single thread can issue large data movement instructions |
| **Direct GMEM→SMEM** | Avoids intermediate register usage |
| **Swizzling Support** | Lays out arriving data to avoid shared memory bank conflicts |
| **Reduction Operations** | Supports add/min/max and bitwise for SMEM→GMEM transfers |

### TMA Performance Impact

> "TMA-enabled FP8 GEMM kernels in Triton deliver **1.4-2.2x performance gains** over cuBLAS FP16 for small-to-medium problem sizes."
> — [PyTorch Blog](https://pytorch.org/blog/hopper-tma-unit/)

| Configuration | GMEM Throughput |
|--------------|-----------------|
| **Without TMA** | 910.22 GB/s |
| **With TMA** | 1.45 TB/s |

> "Result: **59% increase in GMEM throughput**"
> — [PyTorch Blog](https://pytorch.org/blog/hopper-tma-unit/)

### TMA Usage in Triton

**Host Code (Tensor Descriptor Creation):**
```python
desc_a = np.empty(TMA_SIZE, dtype=np.int8)
desc_b = np.empty(TMA_SIZE, dtype=np.int8)
desc_c = np.empty(TMA_SIZE, dtype=np.int8)

triton.runtime.driver.active.utils.fill_2d_tma_descriptor(
    a.data_ptr(), m, k, block_m, block_k, a.element_size(), desc_a
)
triton.runtime.driver.active.utils.fill_2d_tma_descriptor(
    b.data_ptr(), n, k, block_n, block_k, b.element_size(), desc_b
)
triton.runtime.driver.active.utils.fill_2d_tma_descriptor(
    c.data_ptr(), m, n, block_m, block_n, c.element_size(), desc_c
)

desc_a = torch.tensor(desc_a, device='cuda')
desc_b = torch.tensor(desc_b, device='cuda')
desc_c = torch.tensor(desc_c, device='cuda')
```

**Device Code:**
```python
# Load using descriptor
a = tl._experimental_descriptor_load(a_desc_ptr, [offs_am, offs_k], [block_m, block_k], tl.float8e4nv)
b = tl._experimental_descriptor_load(b_desc_ptr, [offs_bn, offs_k], [block_n, block_k], tl.float8e4nv)

# Store using descriptor
tl._experimental_descriptor_store(c_desc_ptr, accumulator, [offs_am, offs_bn])
```

### TMA vs Traditional Load (PTX Comparison)

**Without TMA (cp.async):**
```asm
add.s32 %r27, %r100, %r8;
selp.b32 %r30, %r102, 0, %p18;
@%p1 cp.async.cg.shared.global [ %r27 + 0 ], [ %rd20 + 0 ], 0x10, %r30;
```
> Requires ldmatrix instruction operating on register files

**With TMA (cp.async.bulk.tensor):**
```asm
@%p8 cp.async.bulk.tensor.2d.shared::cluster.global.mbarrier::complete_tx::bytes [%r24], [%rd26, {%r25,%r152}], [%r19];
```
> Data directly reused from shared memory; no ldmatrix needed

### TMA Integration Status

> "On Hopper with num_stages >= 2, Triton's software pipeliner converts make_block_ptr loads inside loops into TMA-based async copies (cp.async)."
> — [Triton GitHub Issue #9348](https://github.com/triton-lang/triton/issues/9348)

---

## Hopper/Blackwell Architecture Support

> "As a result of NVIDIA's ongoing collaboration with OpenAI, the Triton compiler now supports the NVIDIA Blackwell architecture."
> — [NVIDIA Technical Blog](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)

### Hopper Architecture (sm_90, H100/H800)

**Key Features:**
- Tensor Memory Accelerator (TMA)
- Warp specialization support
- FP8 support (float8_e5m2, float8_e4m3)
- Thread Block Clusters

**Triton Support:**
- TMA descriptor API via `tl.make_tensor_descriptor`
- Async TMA operations
- Warp specialization in FlashAttention

### Blackwell Architecture (sm_100/sm_120, B100/B200/GB300)

> "The Triton compiler now supports NVIDIA Blackwell architecture, enabling developers to leverage its advanced features through a Python-based compiler."
> — [NVIDIA Technical Blog](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)

**New in Blackwell:**
| Feature | Description |
|---------|-------------|
| **New Tensor Core** | Designed for improved throughput and energy efficiency |
| **MXFP8** | Block-scaled FP8 with native scaling in Tensor Core |
| **MXFP4** | New operating point, double hardware-accelerated performance of FP8 |
| **Enhanced MMA** | Extended Matrix Multiply-Accumulate pipelining |

### Blackwell Performance Gains

> "Up to 1.5x speedup over NVIDIA Hopper GPU architecture for FP16 attention... Performance gain delivered 'for free' with existing Triton flash attention implementations—**no code changes required**."
> — [NVIDIA Technical Blog](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)

**Flash Attention on Blackwell:**
- FP16: Up to 1.5x faster than Hopper
- FP8: Significant gains via new Tensor Core

### Version Support Timeline

| Triton Version | Blackwell Support | Hopper Support |
|---------------|-------------------|----------------|
| 3.5.0 | ✅ Full (sm_100/sm_120) | ✅ Full |
| 3.4.0 | ✅ Partial | ✅ Full |
| 3.3.x | ❌ | ✅ Full |

### Known Limitations

> "Layout and packing of sub-byte datatype formats like MXFP4 still require care by end users. Low utilization when GEMM_K is small—mitigated through manual sub-tiling."
> — [NVIDIA Technical Blog](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)

---

## Triton for LLM Inference (vLLM Integration)

> "The Triton Inference Server hosts a tutorial demonstrating how to quickly deploy a simple facebook/opt-125m model using vLLM."
> — [vLLM Documentation](https://docs.vllm.ai/en/stable/deployment/frameworks/triton/)

### vLLM + Triton Integration

| Component | Description |
|-----------|-------------|
| **Triton Inference Server** | NVIDIA's inference serving platform |
| **vLLM Backend** | High-throughput LLM inference engine |
| **OpenAI-Compatible Frontend** | Supports Chat Completion API |

### Deployment Architecture

```
Client (OpenAI API) → Triton Inference Server → vLLM Backend → GPU Kernel
```

### vLLM Triton Kernel Support

> "The `vllm.kernels.triton` module contains Triton kernel implementations for vLLM."
> — [vLLM API Documentation](https://docs.vllm.ai/en/latest/api/vllm/kernels/triton/)

**Available Kernels:**
| Kernel | Description |
|--------|-------------|
| `qkv_padded_fp8_quant` | Stride-aware FP8 quantization with head_dim padding for ViT attention |

### Deployment Tutorial

**Quick Start:**
1. Deploy vLLM model with Triton Inference Server
2. Access via OpenAI-compatible endpoints
3. Use Chat Completion API for inference

**Reference:** [Deploying a vLLM model in Triton](https://github.com/triton-inference-server/tutorials/blob/main/Quick_Deploy/vLLM/README.md)

### vLLM vs Triton for AI Deployment

> "In 2025, vLLM has become even more popular for its simplicity and speed in LLM-focused setups. It's great for startups or teams that want quick deployment."
> — [Medium: vLLM vs Triton for Smarter AI Deployment](https://medium.com/@tam.tamanna18/vllm-vs-triton-for-smarter-ai-deployment-02b61f898b33)

### Triton Inference Server Features

- **OpenAI Compatibility**: Named function calling, structured outputs
- **Multiple Backends**: vLLM, TensorRT-LLM
- **Model Repository**: Pre-built optimized models
- **Dynamic Batching**: For throughput optimization

---

## Resources and References

### Official Documentation
- **Triton Documentation**: [triton-lang.org](https://triton-lang.org/)
- **Triton GitHub**: [github.com/triton-lang/triton](https://github.com/triton-lang/triton)
- **Triton Releases**: [github.com/triton-lang/triton/releases](https://github.com/triton-lang/triton/releases)

### Installation
```bash
# Quick install
pip install triton

# From source
pip install pybind11
pip install -e .
```

### Tutorials
| Tutorial | Description |
|----------|-------------|
| [Matrix Multiplication](https://triton-lang.org/main/getting-started/tutorials/03-matrix-multiplication.html) | GEMM kernel optimization |
| [Fused Attention](https://triton-lang.org/main/getting-started/tutorials/06-fused-attention.html) | FlashAttention implementation |
| [Persistent Matmul](https://triton-lang.org/main/getting-started/tutorials/09-persistent-matmul.html) | Advanced GEMM with TMA |

### Technical Papers
- [Triton: An Intermediate Language and Compiler for Tiled Neural Network Computations (MAPL 2019)](https://www.eecs.harvard.edu/~htk/publication/2019-mapl-tillet-kung-cox.pdf)
- [FlashAttention Paper](https://arxiv.org/abs/2205.14135)

### NVIDIA Technical Blogs
- [OpenAI Triton on NVIDIA Blackwell](https://developer.nvidia.com/blog/openai-triton-on-nvidia-blackwell-boosts-ai-performance-and-programmability/)
- [Hopper TMA Unit for FP8 GEMMs](https://pytorch.org/blog/hopper-tma-unit/)

### Community
- [GPU MODE Discord](https://discord.com/invite/gpumode)
- [Triton GitHub Issues](https://github.com/triton-lang/triton/issues)

### vLLM Integration
- [vLLM Triton Documentation](https://docs.vllm.ai/en/stable/deployment/frameworks/triton/)
- [vLLM Triton Kernels API](https://docs.vllm.ai/en/latest/api/vllm/kernels/triton/)
- [Triton Inference Server Tutorials](https://github.com/triton-inference-server/tutorials)

---

## Key Contributors

| Name | Role | Background |
|------|------|-----------|
| **Philippe Tillet** | OpenAI (Triton creator) | Ph.D. Harvard 2020, started Triton 2018 |
| **Thomas Raoux** | OpenAI (GPU compilation) | Previously Google (MLIR/IREE), Intel |
| **Paweł Szczerbuk** | OpenAI (Triton compiler) | Previously Intel Graphics Compiler, Apple Metal |
| **Peter Bell** | OpenAI (Triton compiler) | Previously Quansight, PyTorch core maintainer |
| **Pradeep Ramani** | NVIDIA (Sr. deep learning architect) | 14+ years across GPU stack |
| **Jason Knight** | NVIDIA (Director of deep learning compilers) | Ph.D. ML/Computational Biology |