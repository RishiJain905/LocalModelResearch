# AMD RDNA3 Intrinsics for ML Training Optimization

> **Context:** This document is part of the k8s-elastic-training wiki under [training-infrastructure](../README.md). It provides structured research on AMD RDNA3 GPU intrinsics for ML training optimization, with source links and key quotes.

---

## Table of Contents

1. [AMD GPU Architectures Overview](#1-amd-gpu-architectures-overview)
2. [RDNA3 ISA and Instruction Set](#2-rdna3-isa-and-instruction-set)
3. [Wavefront/Warp-Level Programming](#3-wavefrontwarp-level-programming)
4. [Matrix Instructions: WMMA](#4-matrix-instructions-wmma)
5. [ROCm Intrinsics: hipBLAS and rocBLAS](#5-rocm-intrinsics-hipblas-and-rocblas)
6. [CDNA3 vs RDNA3 for ML Training](#6-cdna3-vs-rdna3-for-ml-training)
7. [MI300X and MI250X GPU Architecture](#7-mi300x-and-mi250x-gpu-architecture)
8. [ROCm Software Stack for Kubernetes](#8-rocm-software-stack-for-kubernetes)
9. [HIP vs OpenCL for AMD GPU Development](#9-hip-vs-opencl-for-amd-gpu-development)
10. [Key GitHub Repositories](#10-key-github-repositories)

---

## 1. AMD GPU Architectures Overview

AMD GPU architectures are divided into two distinct product lines:

| Architecture | Target | Use Case | Products |
|--------------|--------|----------|----------|
| **CDNA (Compute DNA)** | Data Center / AI / HPC | ML Training, HPC | MI100, MI200, MI250X, MI300X, MI325X, MI355X |
| **RDNA (Radeon DNA)** | Consumer / Gaming | Graphics, Light ML | RX 6000, RX 7000, RX 9000 series |

> *"There are two AMD GPU architectures that are relevant to machine learning: RDNA3 and CDNA. The RDNA3 architecture is used in AMD's Radeon consumer graphics cards, while CDNA is designed for datacenter accelerators like the MI300X."*
> — [GPU Poet - Can I Use AMD GPUs with AI & Machine Learning?](https://gpupoet.com/gpu/learn/ai/faq/amd-gpus-for-ai-machine-learning)

**Key Architectural Differences:**

| Feature | RDNA3 | CDNA3 |
|---------|-------|-------|
| **Primary Focus** | Gaming / Graphics | AI / ML / HPC |
| **Matrix Cores** | WMMA (Wave Matrix Multiply-Accumulate) | MFMA (Matrix Fused Multiply-Add) |
| **Wavefront Size** | 32 or 64 threads | 32 or 64 threads |
| **FP8 Support** | Not natively | Native FP8 matrix operations |
| **HBM Memory** | No (consumer GPUs) | Yes (192-256 GB) |

---

## 2. RDNA3 ISA and Instruction Set

### Official Documentation

> *"This document describes the instruction set and shader program accessible state for RDNA3 devices."*
> — [AMD RDNA3 ISA Reference Guide](https://docs.amd.com/v/u/en-US/rdna3-shader-instruction-set-architecture-feb-2023_0)

**Source:** [AMD RDNA3 ISA Reference Guide (PDF)](https://www.techpowerup.com/gpu-specs/docs/amd-rdna3-isa.pdf)

### Key ISA Features

RDNA3 (GFX11) introduced several key architectural changes:

- **Chiplet Design**: First AMD gaming GPU to use chiplet packaging
- **Infinity Cache**: Enhanced 96MB L3 cache (up from 128MB in RDNA2 but more efficient)
- **Matrix Cores**: Hardware-accelerated WMMA instructions for AI workloads
- **Dual-issue Instructions**: Improved instruction-level parallelism

> *"RDNA 3 GPUs can hit the same frequency as RDNA 2 GPUs while using half the power, or they can hit 1.3 times the frequency while using the same power."*
> — [Reddit - AMD RDNA 3 GPU Architecture Deep Dive](https://www.reddit.com/r/hardware/comments/yv0qxk/amd_rdna_3_gpu_architecture_deep_dive_the_ryzen/)

### ISA Documentation Sources

| Resource | URL | Description |
|----------|-----|-------------|
| **Official ISA Guide** | [docs.amd.com - RDNA3 Shader ISA](https://docs.amd.com/v/u/en-US/rdna3-shader-instruction-set-architecture-feb-2023_0) | Complete instruction set documentation |
| **PDF Version** | [TechPowerUp RDNA3 ISA PDF](https://www.techpowerup.com/gpu-specs/docs/amd-rdna3-isa.pdf) | Downloadable ISA reference |
| **Phoronix Announcement** | [AMD RDNA3 ISA Guide Published](https://www.phoronix.com/news/AMD-RDNA3-ISA-Guide) | News coverage of ISA release |
| **GPUOpen Announcement** | [gpuopen.com - RDNA3 ISA Guide](https://gpuopen.com/news/rdna3-isa-guide-now-available/) | Official AMD announcement |

---

## 3. Wavefront/Warp-Level Programming

### Wavefront Concept

In AMD GPUs, a **wavefront** is a group of threads that execute in lockstep — equivalent to NVIDIA's "warp". RDNA3 supports both **Wave32** (32 threads) and **Wave64** (64 threads) modes.

> *"The MFMA instructions in AMD CDNA GPUs operate on a per-wavefront basis, rather than on a per-lane (thread) basis: entries of the input and output matrices are distributed over the lanes of the wavefront's vector registers."*
> — [AMD Matrix Cores - GPUOpen](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-matrix-cores-readme/)

### Wavefront-Level Matrix Operations

Both RDNA3 (WMMA) and CDNA (MFMA) use wavefront-level programming where:

- **All threads in a wavefront collectively** perform matrix multiplication
- **Matrix entries are distributed** across vector registers (VGPRs) of the wavefront
- **Data sharing happens implicitly** within the wavefront

```
Wavefront (32 or 64 threads)
    ├── Thread 0: holds matrix element D[0,0]
    ├── Thread 1: holds matrix element D[0,1]
    └── ...
```

### WMMA Wave Modes

| Mode | Threads | Use Case |
|------|--------|----------|
| **Wave32** | 32 threads | Standard GEMM operations |
| **Wave64** | 64 threads | Larger tile operations |

> *"Unlike per-thread matrix multiplication, WMMA performs matrix multiplication cooperatively across an entire wavefront... This enables sharing input/output matrix data across lanes, optimizing VGPR usage and reducing memory traffic."*
> — [How to accelerate AI applications on RDNA 3 using WMMA](https://gpuopen.com/learn/wmma_on_rdna3/)

### Performance Comparison: RDNA 2 vs RDNA 3

| Data Type | RX 6950 XT (RDNA 2) FLOPS/clock/CU | RX 7900 XTX (RDNA 3) FLOPS/clock/CU |
|-----------|-----------------------------------|-------------------------------------|
| FP16 | 256 | 512 |
| BF16 | N/A | 512 |
| IU8 | 512 | 512 |
| IU4 | 1024 | 1024 |

> *"The WMMA instruction optimizes the scheduling of data movement and peak math operations with minimal VGPR access by providing source data reuse and intermediate destination data forwarding operations without interruption."*
> — Mike Mantor, Corporate Fellow and Chief GPU Architect

---

## 4. Matrix Instructions: WMMA

### RDNA3 WMMA Overview

**WMMA (Wave Matrix Multiply-Accumulate)** is the matrix instruction set for RDNA3 GPUs, introduced specifically to accelerate AI/ML inference.

> *"Wave Matrix Multiply Accumulate (WMMA) is a hardware feature on RDNA 3 GPUs that accelerates Generalized Matrix Multiplication (GEMM) operations. A single WMMA instruction coordinates 32 clocks of optimal work scheduling."*
> — [GPUOpen - WMMA on RDNA3](https://gpuopen.com/learn/wmma_on_rdna3/)

### Supported Data Types

| Data Type | Description | Use Case |
|-----------|-------------|----------|
| FP16 | 16-bit float | Training (online/offline) |
| BF16 | Brain float | Training (online/offline) |
| IU8 | 8-bit unsigned integer | Inference |
| IU4 | 4-bit unsigned integer | Inference |

### Tile Size

RDNA 3 supports only **16x16x16** tile size:

```
Matrix A: 16x16 (column-major)
Matrix B: 16x16 (row-major)  
Result D: 16x16 added with matrix C
```

### GEMM Operation

```
D = A*B + C
```

| Matrix | Dimensions | Storage Order |
|--------|------------|---------------|
| A | MxK | **Column-major** |
| B | KxN | **Row-major** |
| C | MxN | **Row-major** |
| D | MxN | **Row-major** |

### Compiler Intrinsic Syntax

```cpp
D_frag = __builtin_amdgcn_wmma__16x16x16__w<32 or 64>(A_frag, B_frag, C_frag, OPSEL)
```

**Parameters:**
- `A_frag`, `B_frag`: Input matrix fragments (16 elements each)
- `C_frag`: Accumulator fragment
- `D_frag`: Result fragment
- `w<32 or 64>`: Wave32 or Wave64 mode
- `OPSEL`: Controls VGPR storage location for 16-bit C/D

### All 12 WMMA Intrinsics

| Wave32 Intrinsic | Wave64 Intrinsic | A,B Format | C,D Format |
|------------------|-------------------|------------|------------|
| `__builtin_amdgcn_wmma_f32_16x16x16_f16_w32` | `w64` | FP16 | FP32 |
| `__builtin_amdgcn_wmma_f32_16x16x16_bf16_w32` | `w64` | BF16 | FP32 |
| `__builtin_amdgcn_wmma_f16_16x16x16_f16_w32` | `w64` | FP16 | FP16 |
| `__builtin_amdgcn_wmma_bf16_16x16x16_bf16_w32` | `w64` | BF16 | BF16 |
| `__builtin_amdgcn_wmma_i32_16x16x16_iu8_w32` | `w64` | IU8 | I32 |
| `__builtin_amdgcn_wmma_i32_16x16x16_iu4_w32` | `w64` | IU4 | I32 |

### Complete Code Example

```cpp
// Wave Matrix Multiply Accumulate (WMMA) using HIP compiler intrinsic
// 16x16x16 GEMM with FP16 inputs/outputs in wave32 mode

#include <hip/hip_runtime.h>
#include <stdio.h>

typedef _Float16 half16 __attribute__((ext_vector_type(16)));

__global__ void wmma_matmul(__half* a, __half* b, __half* c)
{
    const int lIdx = threadIdx.x;
    const int lane = lIdx % 16;

    half16 a_frag;  // 8 VGPRs for 16 FP16 elements
    half16 b_frag;  // 8 VGPRs for 16 FP16 elements
    half16 c_frag = {};  // Initialize to zero

    // Load matrix B (row-major) into b_frag
    for (int ele = 0; ele < 16; ++ele)
        b_frag[ele] = b[16*ele + lane];

    // Load matrix A (column-major) into a_frag
    for (int ele = 0; ele < 16; ++ele)
        a_frag[ele] = a[16 * lane + ele];

    // Call WMMA intrinsic with OPSEL = false
    c_frag = __builtin_amdgcn_wmma_f16_16x16x16_f16_w32(a_frag, b_frag, c_frag, false);

    // Store results
    for (int ele = 0; ele < 8; ++ele) {
        const int r = ele * 2 + (lIdx / 16);
        c[16 * r + lane] = c_frag[ele*2];
    }
}
```

### Matrix Fragment Storage Requirements

**A_frag and B_frag (per thread):**

| Data Type | VGPRs Required | Values Packed per VGPR |
|-----------|----------------|------------------------|
| FP16/BF16 | 8 | 2 |
| IU8 | 4 | 4 |
| IU4 | 2 | 8 |

**C_frag and D_frag (per thread):**

| Mode | VGPRs Required |
|------|----------------|
| Wave32 | 8 |
| Wave64 | 4 |

### Data Replication Requirement

> *"Wave32 mode: Lanes 0-15 must mirror lanes 16-31"*
> *"Wave64 mode: Lanes 0-15 must replicate to lanes 32-47 and 48-63"*
> — [GPUOpen - WMMA on RDNA3](https://gpuopen.com/learn/wmma_on_rdna3/)

---

## 5. ROCm Intrinsics: hipBLAS and rocBLAS

### hipBLAS Overview

> *"hipBLAS is a Basic Linear Algebra Subprograms (BLAS) marshaling library that supports multiple backends. It sits between the application and a 'worker' BLAS library, marshalling inputs into the backend library and marshalling results back to the application."*
> — [hipBLAS Documentation](https://rocm.docs.amd.com/projects/hipBLAS/en/docs-5.3.3/)

**Key Features:**
- BLAS marshaling library with multiple backends
- Supports both AMD (rocBLAS) and NVIDIA (cuBLAS) backends
- Provides portable interface for both CUDA and AMD hardware

### rocBLAS Overview

> *"rocBLAS is written in C++14 and HIP. It uses AMD's ROCm runtime to run on GPU devices. The rocBLAS API is a thin C99 API using the Hourglass Pattern."*
> — [rocBLAS Introduction](https://rocm.docs.amd.com/projects/rocBLAS/en/docs-5.0.2/intro.html)

### Library Comparison

| Library | Language | Target | Notes |
|---------|----------|--------|-------|
| **rocBLAS** | HIP/C++ | AMD GPUs only | Native AMD implementation |
| **hipBLAS** | C/C++ | AMD + NVIDIA | Wrapper library for portability |
| **cuBLAS** | CUDA/C | NVIDIA only | NVIDIA native implementation |

### Performance Notes

> *"hipBLAS exports an interface that does not require the client to change, regardless of the chosen backend. Currently, it supports rocBLAS and cuBLAS as backends."*
> — [hipBLAS Documentation](https://rocm.docs.amd.com/projects/hipBLAS/en/docs-5.3.3/)

### Documentation Links

| Resource | URL |
|----------|-----|
| **hipBLAS Documentation** | [rocm.docs.amd.com - hipBLAS](https://rocm.docs.amd.com/projects/hipBLAS/en/latest/) |
| **hipBLAS API Reference** | [rocm.docs.amd.com - hipBLAS API](https://rocm.docs.amd.com/projects/hipBLAS/en/docs-5.1.1/API_Reference_Guide.html) |
| **rocBLAS Introduction** | [rocm.docs.amd.com - rocBLAS Intro](https://rocm.docs.amd.com/projects/rocBLAS/en/docs-5.0.2/intro.html) |
| **rocBLAS API Reference** | [rocm.docs.amd.com - rocBLAS API](https://rocm.docs.amd.com/projects/rocBLAS/en/docs-5.1.1/API_Reference_Guide.html) |

---

## 6. CDNA3 vs RDNA3 for ML Training

### Architectural Differences

| Feature | CDNA3 (MI300X) | RDNA3 (RX 7900 XTX) |
|---------|----------------|---------------------|
| **Architecture Codename** | Aldebaran | Navi 31 |
| **Process Node** | 5nm + 6nm chiplet | 5nm + 6nm chiplet |
| **Compute Units** | 304 | 96 |
| **Stream Processors** | 19,456 | 6,144 |
| **Matrix Cores** | 1,216 | 96 (WMMA) |
| **Memory** | 192 GB HBM3 | 24 GB GDDR6 |
| **Memory Bandwidth** | 5.3 TB/s | 960 GB/s |
| **FP16 Matrix Performance** | 1,307 TFLOPS | ~82 TFLOPS |
| **FP8 Matrix Performance** | 2,615 TFLOPS | Not supported |
| **TDP** | 750W | 355W |

### ML Training Suitability

> *"The AMD Instinct MI300X discrete GPU is based on next-generation AMD CDNA 3 architecture, delivering leadership efficiency and performance for the most demanding AI and HPC applications."*
> — [AMD MI300X Data Sheet](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/data-sheets/amd-instinct-mi300x-data-sheet.pdf)

**For ML Training:**
- **CDNA3 is purpose-built** for ML training with native FP8, TF32, BF16 support
- **RDNA3 has limited ML capability** — WMMA is designed for inference acceleration
- **RDNA3 lacks native FP8** matrix operations

### Key CDNA3 Advantages for Training

1. **Native FP8 Support**: 2,615 TFLOPS peak FP8 performance
2. **TF32 Data Type**: 653.7 TFLOPS for easier FP32 migration
3. **4:2 Sparsity**: Doubles effective throughput
4. **Large HBM3 Memory**: 192 GB enables large model training
5. **Infinity Cache**: 256 MB shared cache reduces memory latency

### Performance Comparison (FP16)

| GPU | FP16 Matrix TFLOPS | Relative Performance |
|-----|-------------------|----------------------|
| MI300X | 1,307 | 16x |
| RX 7900 XTX | ~82 | Baseline |

---

## 7. MI300X and MI250X GPU Architecture

### MI300X Specifications

| Specification | MI300X | MI250X |
|--------------|--------|--------|
| **Architecture** | CDNA 3 | CDNA 2 |
| **Process Node** | 5nm/6nm | 6nm |
| **Compute Units** | 304 | 220 |
| **Stream Processors** | 19,456 | 14,080 |
| **Matrix Cores** | 1,216 | 880 |
| **Memory** | 192 GB HBM3 | 128 GB HBM2 |
| **Memory Bandwidth** | 5.3 TB/s | 3.3 TB/s |
| **FP64 Matrix** | 163.4 TFLOPS | 95.6 TFLOPS |
| **FP16 Matrix** | 1,307.4 TFLOPS | 383 TFLOPS |
| **FP8 Matrix** | 2,614.9 TFLOPS | Not supported |
| **TDP** | 750W | 560W |

> *"AMD Instinct MI300X offers 13.7x the peak AI/ML workload performance using FP8 with sparsity compared to prior AMD MI250X accelerators using FP16."*
> — [AMD CDNA3 White Paper](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-3-white-paper.pdf)

### CDNA3 Architecture Highlights

> *"AMD CDNA™ 3 architecture represents a fundamental leap in GPU design, leveraging advanced 3D packaging and chiplet technology to deliver the highest performance, efficiency, and programmability to date."*
> — [AMD CDNA3 White Paper](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-3-white-paper.pdf)

### Chiplet Architecture

| Chiplet Type | Function | Process Node |
|-------------|----------|--------------|
| **XCDs (Accelerator Complex Dies)** | Computational elements + lowest cache levels | TSMC 5nm |
| **IODs (I/O Dies)** | System infrastructure, Infinity Cache, HBM interface | TSMC 6nm |

### MI300X Chiplet Configuration

```
MI300X (8 XCDs + 1 IOD)
├── 8 × XCDs = 304 Compute Units
├── 1 × IOD with 256 MB Infinity Cache
└── 8 × HBM3 stacks (192 GB total)
```

### MFMA vs WMMA Instruction Comparison

| Feature | CDNA MFMA | RDNA3 WMMA |
|---------|-----------|------------|
| **Instruction Family** | V_MFMA_* | V_WMMA_* |
| **MNK Dimensions** | 4x4 to 32x32 | 16x16x16 only |
| **FP64 Support** | Yes | No |
| **FP8 Support** | Yes (native) | No |
| **FP16/BF16** | Yes | Yes |
| **Output Types** | FP32, FP64, FP16, BF16, INT8 | FP32, FP16, BF16, INT32 |

### MFMA Intrinsic Example (CDNA)

```cpp
// CDNA MFMA: 32x32x8 FP16 -> FP32
d = __builtin_amdgcn_mfma_f32_32x32x8f16(a, b, c, cbsz, abid, blgp)
```

### Documentation Links

| Resource | URL |
|----------|-----|
| **MI300X Data Sheet** | [amd.com - MI300X Data Sheet (PDF)](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/data-sheets/amd-instinct-mi300x-data-sheet.pdf) |
| **CDNA3 White Paper** | [amd.com - CDNA3 White Paper (PDF)](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-3-white-paper.pdf) |
| **MI300X vs MI250X Comparison** | [Flopper.io](https://flopper.io/compare/amd-mi300x-192gb-vs-amd-mi250x-128gb) |
| **Testing AMD's Giant MI300X** | [Chips and Cheese](https://chipsandcheese.com/p/testing-amds-giant-mi300x) |

---

## 8. ROCm Software Stack for Kubernetes

### AMD GPU Operator

> *"The AMD GPU Operator simplifies the deployment and management of AMD Instinct™ GPU accelerators within Kubernetes clusters."*
> — [ROCm GPU Operator GitHub](https://github.com/ROCm/gpu-operator)

### Architecture Overview

```
Kubernetes Cluster
├── AMD GPU Operator
│   ├── Device Plugin (GPU allocation)
│   ├── Node Labeler (GPU labeling)
│   ├── DCGM Exporter (metrics)
│   └── Driver Management
└── AMD ROCm Stack
    ├── ROCm Driver (kernel module)
    ├── HIP Runtime
    ├── rocBLAS / hipBLAS
    └── MI300X/MI250X Support
```

### Installation via Helm

```bash
# Add Jetstack repo for cert-manager
helm repo add jetstack https://charts.jetstack.io
helm repo update

# Install cert-manager
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set installCRDs=true

# Install AMD GPU Operator
helm install gpu-operator -n gpu-operator --create-namespace \
  oci://ghcr.io/rocm/gpu-operator/gpu-operator \
  --version 1.16.0
```

### Device Configuration

```yaml
# device-config.yaml
apiVersion: gpu.openshift.io/v1alpha1
kind: DeviceConfig
metadata:
  name: device-config
spec:
  devicePlugin:
    enabled: true
  nodeLabeler:
    enabled: true
  dcgmExporter:
    enabled: true
```

### Verification

```bash
# Verify node labeling
kubectl get nodes -o wide --show-labels | grep amd.com/gpu

# Show available GPUs
kubectl get devices.node.amd.com
```

### Production Stack Components

| Component | Purpose |
|-----------|---------|
| **Kubernetes + MicroK8s** | Orchestration framework |
| **AMD GPU Operator** | GPU integration |
| **Persistent Storage** | Model caching (PVC) |
| **vLLM** | Inference runtime |
| **MetalLB** | Load balancing |
| **Prometheus + Grafana** | Monitoring |

> *"This three-part blog series provides a production-ready foundation for orchestrating AI inference workloads on the AMD Instinct platform with Kubernetes."*
> — [AMD ROCm Blog - K8s Orchestration Part 1](https://rocm.blogs.amd.com/artificial-intelligence/k8s-orchestration-part1/README.html)

### Documentation Links

| Resource | URL |
|----------|-----|
| **GPU Operator GitHub** | [github.com/ROCm/gpu-operator](https://github.com/ROCm/gpu-operator) |
| **K8s Orchestration Part 1** | [rocm.blogs.amd.com](https://rocm.blogs.amd.com/artificial-intelligence/k8s-orchestration-part1/README.html) |
| **K8s Orchestration Part 2** | [rocm.blogs.amd.com](https://rocm.blogs.amd.com/artificial-intelligence/k8s-orchestration-part2/README.html) |
| **K8s Orchestration Part 3** | [rocm.blogs.amd.com](https://rocm.blogs.amd.com/artificial-intelligence/k8s-orchestration-part3/README.html) |
| **Deploy vLLM on K8s** | [Medium - Deploy vLLM with K8s over AMD ROCm](https://medium.com/@alexhe.amd/deploy-vllm-service-with-kubernetes-over-amd-rocm-gpu-27cd5321271a) |

---

## 9. HIP vs OpenCL for AMD GPU Development

### Overview

| Aspect | HIP | OpenCL |
|--------|-----|--------|
| **Full Name** | Heterogeneous-Compute Interface for Portability | Open Computing Language |
| **Language** | C++ dialect | C99 |
| **AMD Support** | Primary, native | Supported but legacy |
| **NVIDIA Support** | Yes (via HIPify) | No |
| **Portability** | CUDA ↔ AMD | Vendor-agnostic |
| **Performance** | Near-native | Variable |

> *"HIP is an API based on C++ that provides a runtime and kernel language for GPU programming and is the essential ROCm programming language."*
> — [ROCm Programming Guide](https://rocm.docs.amd.com/en/latest/how-to/programming_guide.html)

### HIP Advantages

1. **CUDA Compatibility**: Easy porting from CUDA code via HIPify
2. **C++ Syntax**: Modern language features, templates, classes
3. **Single Source**: Write once, run on AMD and NVIDIA
4. **Native Performance**: Direct access to AMD hardware features

> *"Porting from CUDA to HIP is significantly easier than from CUDA to OpenCL."*
> — [HIP FAQ](https://rocm.docs.amd.com/projects/HIP/en/docs-6.3.1/faq.html)

### OpenCL Characteristics

1. **Vendor Agnostic**: Works across AMD, NVIDIA, Intel, etc.
2. **Mature Ecosystem**: Long-standing standard
3. **Limited AMD Features**: May not expose latest AMD GPU features
4. **Verbose Syntax**: Less intuitive than HIP C++

> *"ROCm supports OpenCL for developers who prefer the OpenCL standard or are porting existing OpenCL applications."*
> — [ROCm Programming Guide](https://rocm.docs.amd.com/en/latest/how-to/programming_guide.html)

### Performance Comparison

> *"The performance difference can be removed by configuring HIP to use the same number of threads as OpenCL. Since this kernel is compute-bound, its performance is sensitive to the details of register allocation and instruction selection, which may differ between the OpenCL and HIP kernel compilers."*
> — [Futhark Blog - Comparing OpenCL, CUDA, and HIP](https://futhark-lang.org/blog/2024-07-17-opencl-cuda-hip.html)

### Recommendation

| Use Case | Recommended |
|----------|-------------|
| **New AMD GPU Development** | **HIP** |
| **CUDA to AMD Porting** | **HIP** (via HIPIFY) |
| **Cross-vendor (AMD + NVIDIA + Intel)** | OpenCL |
| **Legacy OpenCL Code** | Continue with OpenCL or migrate to HIP |

### Documentation Links

| Resource | URL |
|----------|-----|
| **HIP Programming Guide** | [rocm.docs.amd.com - HIP](https://rocm.docs.amd.com/en/latest/books/Programming_guide.html) |
| **HIP FAQ** | [rocm.docs.amd.com - HIP FAQ](https://rocm.docs.amd.com/projects/HIP/en/docs-6.3.1/faq.html) |
| **ROCm Programming Guide** | [rocm.docs.amd.com - Programming Guide](https://rocm.docs.amd.com/en/latest/how-to/programming_guide.html) |
| **HIP vs CUDA Comparison** | [ACM Paper](https://dl.acm.org/doi/pdf/10.1145/3677997.3678226) |

---

## 10. Key GitHub Repositories

### AMD Official Repositories

| Repository | Description | URL |
|------------|-------------|-----|
| **ROCm** | Main ROCm ecosystem | [github.com/ROCm/ROCm](https://github.com/ROCm/ROCm) |
| **GPU Operator** | Kubernetes GPU management | [github.com/ROCm/gpu-operator](https://github.com/ROCm/gpu-operator) |
| **hipBLAS** | BLAS marshaling library | [github.com/ROCm/hipBLAS](https://github.com/ROCm/hipBLAS) |
| **rocBLAS** | AMD BLAS implementation | [github.com/ROCm/rocBLAS](https://github.com/ROCm/rocBLAS) |
| **rocWMMA** | WMMA library for CDNA | [github.com/ROCm/rocWMMA](https://github.com/ROCm/rocWMMA) |
| **rocFFT** | FFT library | [github.com/ROCm/rocFFT](https://github.com/ROCm/rocFFT) |
| **rocSOLVER** | LAPACK routines | [github.com/ROCm/rocSOLVER](https://github.com/ROCm/rocSOLVER) |

### Developer Tools

| Repository | Description | URL |
|------------|-------------|-----|
| **AMD Matrix Instruction Calculator** | MFMA/WMMA analysis tool | [github.com/ROCm/amd_matrix_instruction_calculator](https://github.com/ROCm/amd_matrix_instruction_calculator) |
| **HIPIFY** | CUDA to HIP conversion | [github.com/ROCm/HIPIFY](https://github.com/ROCm/HIPIFY) |
| **hip-python** | Python HIP bindings | [github.com/ROCm/hip-python](https://github.com/ROCm/hip-python) |
| **hipfort** | Fortran HIP bindings | [github.com/ROCm/hipfort](https://github.com/ROCm/hipfort) |

### Community Repositories

| Repository | Description | URL |
|------------|-------------|-----|
| **ROCM WMMA Samples** | WMMA usage examples | [github.com/adelj88/rocm_wmma_samples](https://github.com/adelj88/rocm_wmma_samples) |
| **CDNA Matrix Cores Guide** | Salykova's tutorial | [salykova.github.io/matrix-cores-cdna](https://salykova.github.io/matrix-cores-cdna) |

---

## Related Documents

- [TDP Metrics](../green-ai-thermals/TDP-Metrics.md) — GPU power specifications for MI300X, H100, etc.
- [Dynamic Provisioning](./Dynamic-Provisioning.md) — Storage provisioning for GPU clusters
- [Karpenter](./karpenter.md) — Autoscaling GPU nodes
- [Kubeflow](./kubeflow.md) — ML platform on Kubernetes
- [Distributed Networking](../Hardware-Accelerators/distributed-networking/) — High-speed NICs for GPU clusters

---

## References

1. [AMD RDNA3 ISA Reference Guide](https://docs.amd.com/v/u/en-US/rdna3-shader-instruction-set-architecture-feb-2023_0)
2. [GPUOpen - WMMA on RDNA3](https://gpuopen.com/learn/wmma_on_rdna3/)
3. [AMD Matrix Cores - GPUOpen](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-matrix-cores-readme/)
4. [Matrix Core Programming on CDNA3/CDNA4](https://rocm.blogs.amd.com/software-tools-optimization/matrix-cores-cdna/README.html)
5. [AMD CDNA3 White Paper (PDF)](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/white-papers/amd-cdna-3-white-paper.pdf)
6. [AMD MI300X Data Sheet (PDF)](https://www.amd.com/content/dam/amd/en/documents/instinct-tech-docs/data-sheets/amd-instinct-mi300x-data-sheet.pdf)
7. [hipBLAS Documentation](https://rocm.docs.amd.com/projects/hipBLAS/en/latest/)
8. [rocBLAS Documentation](https://rocm.docs.amd.com/projects/rocBLAS/en/latest/)
9. [ROCm Programming Guide](https://rocm.docs.amd.com/en/latest/how-to/programming_guide.html)
10. [AMD GPU Operator GitHub](https://github.com/ROCm/gpu-operator)
11. [K8s Orchestration Part 1](https://rocm.blogs.amd.com/artificial-intelligence/k8s-orchestration-part1/README.html)
12. [AMD Matrix Instruction Calculator](https://github.com/ROCm/amd_matrix_instruction_calculator)
13. [GPU Poet - AMD GPUs for AI/ML](https://gpupoet.com/gpu/learn/ai/faq/amd-gpus-for-ai-machine-learning)
14. [ROCm 7.0 Announcement](https://www.phoronix.com/forums/forum/linux-graphics-x-org-drivers/open-source-amd-linux/1550326-amd-rocm-7-0-to-align-hip-c-even-more-closely-with-cuda)
15. [Futhark Blog - OpenCL vs CUDA vs HIP](https://futhark-lang.org/blog/2024-07-17-opencl-cuda-hip.html)

---

*Last updated: 2026-05-01*