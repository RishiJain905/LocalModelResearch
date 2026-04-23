# HIP-SDK

## 📌 Overview

**HIP SDK** (Heterogeneous-compute Interface for Portability) is AMD's development toolkit providing APIs, tooling, and libraries for creating portable, high-performance GPU applications. HIP is the AMD equivalent of NVIDIA's CUDA — enabling a single codebase to run on both AMD and NVIDIA GPUs.

> *"HIP (Heterogeneous-compute Interface for Portability) is a C++ runtime API and kernel language that enables developers to create platform-independent GPU programs running on both AMD and NVIDIA GPUs."*
> — [GPUOpen](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-hipify-readme/)

---

## HIP vs CUDA

| Aspect | CUDA | HIP |
|--------|------|-----|
| Vendor | NVIDIA | AMD |
| Syntax | `cudaMalloc`, `cudaFree` | `hipMalloc`, `hipFree` |
| Portability | NVIDIA only | AMD + NVIDIA |
| Base | NVIDIA proprietary | Open source (ROCm) |

> *"Syntax closely mirrors NVIDIA CUDA — replace `cuda` with `hip` for basic translation."*
> — [GPUOpen](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-hipify-readme/)

---

## Core Components

| Component | Description |
|-----------|-------------|
| **AMD GPU Driver** | Required for hardware acceleration |
| **HIP Runtime** | C++ runtime API for GPU programming |
| **HIP Libraries** | Math libraries (BLAS, FFT) and primitives |

> *"The HIP SDK consists of the AMD GPU Driver, HIP runtime, and HIP Libraries. These three parts are distributed in the HIP SDK installer."*
> — [ROCm Windows Docs](https://rocm.docs.amd.com/projects/install-on-windows/en/latest/)

---

## HIPIFY — CUDA to HIP Conversion

AMD provides two tools to automatically translate CUDA code to HIP:

### hipify-clang (Recommended)

> *"hipify-clang is a Clang-based tool for translating NVIDIA CUDA sources into HIP sources. It translates CUDA source into an Abstract Syntax Tree (AST)."*
> — [ROCm HIPIFY Docs](https://rocm.docs.amd.com/projects/HIPIFY/en/latest/hipify-clang.html)

```bash
hipify-clang --cuda-path=/your-path/to/cuda \
  -I /your-path/to/cuda/include \
  -o /your-path/to/desired-output-dir/vectorAdd_hip.cpp vectorAdd.cu
```

Handles complex constructs, kernel launches, and API calls. Supports Clang options (`-I`, `-D`, `--cuda-path`).

### hipify-perl

> *"hipify-perl is useful for simple CUDA programs, but offers less error detection."*
> — [ROCm HIPIFY Docs](https://rocm.docs.amd.com/projects/HIPIFY/en/docs-6.4.2/)

### Limitations

> *"HIPIFY does not automatically convert all CUDA code into HIP code seamlessly... CUDA libraries, or third-party libraries that have no HIP equivalent cannot be translated."*
> — [ROCm HIPIFY](https://rocm.docs.amd.com/projects/HIPIFY/en/docs-6.4.2/)

---

## ROCm Components: Linux vs Windows

| Component | Linux | Windows |
|-----------|-------|---------|
| Compiler | hipcc/amdclang++ | hipcc/clang++ |
| Debugger | rocGDB | ROCm Debugger |
| Profiler | ROCProfiler | Radeon GPU Profiler |
| Porting Tools | HIPIFY | HIPIFY |
| Runtime | HIP (open source) | HIP (closed source) |
| Math Libraries | ✅ | ✅ |
| AI Libraries (MIOpen, MIGraphX) | ✅ | ❌ |
| Communication Libraries | ✅ | ❌ |
| PyTorch/TensorFlow | ✅ | ❌ |

> *"AI Libraries (MIOpen, MIGraphX) and Communication Libraries are not available on Windows HIP SDK."*
> — [ROCm Windows](https://rocm.docs.amd.com/projects/install-on-windows/en/develop/conceptual/component-support.html)

---

## PyTorch and HIP

PyTorch for HIP **intentionally reuses `torch.cuda` interfaces** for easy porting:

```python
torch.cuda.is_available()   # Works for both CUDA and HIP
torch.cuda.memory_allocated()   # Same interface
torch.cuda.empty_cache()   # Same interface
```

> *"PyTorch for HIP intentionally reuses the existing `torch.cuda` interfaces. Whether you are using PyTorch for CUDA or HIP, the result of calling..."*
> — [PyTorch HIP Notes](https://docs.pytorch.org/docs/stable/notes/hip.html)

### ROCm-Specific Limitations

- **TF32 (TensorFloat-32)** is NOT supported on ROCm
- Distributed backends only support `"nccl"` and `"gloo"` on ROCm
- hipBLAS workspaces: Default 32 MiB, MI300 and newer: 128 MiB

---

## Unsloth on AMD via HIP/ROCm

```bash
uv pip install unsloth[amd]
```

> *"Unsloth enables fine-tuning of large language models on AMD hardware with significant performance gains: Up to 2x faster fine-tuning, ~70% less memory usage compared to standard methods."*
> — [Unsloth AMD](https://unsloth.ai/docs/get-started/install/amd)

### Known Issues

**Module Not Initialized Error:**

> *"User encountering `Module not initialized` error from `hip_global.cpp:158` when attempting to fine-tune using Unsloth on AMD GPU with ROCm."*
> — [Unsloth GitHub Issue #3526](https://github.com/unslothai/unsloth/issues/3526)

**4-bit Quantization Disabled:**

> *"Unsloth: AMD currently is not stable with 4bit bitsandbytes. Disabling for now."*
> — [Unsloth GitHub Issue #3526](https://github.com/unslothai/unsloth/issues/3526)

**Flash Attention 2 Fallback:**

> *"Flash Attention 2 is not available on AMD. Unsloth automatically falls back to Xformers, which provides equivalent performance on ROCm."*
> — [Unsloth AMD](https://unsloth.ai/docs/get-started/install/amd)

---

## AMD AI Developer Hub — Unsloth Tutorial

> *"This tutorial demonstrates how to fine-tune the Llama-3.1 8B large language model (LLM) on AMD ROCm GPUs by leveraging Unsloth."*
> — [AMD AI Developer Hub](https://rocm.docs.amd.com/projects/ai-developer-hub/en/v3.1/notebooks/fine_tune/unsloth_Llama3_1_8B_GRPO.html)

**Prerequisites:**
- Ubuntu 22.04
- AMD Instinct MI300X GPU (or compatible)
- ROCm 6.3

---

## Key Resources

| Resource | URL |
|----------|-----|
| **AMD HIP SDK Download** | [amd.com/en/developer/resources/rocm-hub/hip-sdk](https://www.amd.com/en/developer/resources/rocm-hub/hip-sdk.html) |
| **HIPIFY Docs** | [rocm.docs.amd.com/projects/HIPIFY](https://rocm.docs.amd.com/projects/HIPIFY/en/docs-6.4.2/) |
| **HIP Language GitHub** | [github.com/ROCm/hip](https://github.com/ROCm/hip) |
| **Unsloth AMD Install** | [unsloth.ai/docs/get-started/install/amd](https://unsloth.ai/docs/get-started/install/amd) |
| **PyTorch HIP Semantics** | [docs.pytorch.org/stable/notes/hip](https://docs.pytorch.org/docs/stable/notes/hip.html) |

---

## Sources

| Type | Citation |
|------|----------|
| **Docs** | [ROCm HIP SDK Windows](https://rocm.docs.amd.com/projects/install-on-windows/en/latest/) |
| **Docs** | [GPUOpen HIP Notes](https://gpuopen.com/learn/amd-lab-notes/amd-lab-notes-hipify-readme/) |
| **Docs** | [HIPIFY Documentation](https://rocm.docs.amd.com/projects/HIPIFY/en/latest/hipify-clang.html) |
| **Docs** | [PyTorch HIP Notes](https://docs.pytorch.org/docs/stable/notes/hip.html) |
| **Tutorial** | [AMD AI Dev Hub — Unsloth Llama](https://rocm.docs.amd.com/projects/ai-developer-hub/en/v3.1/notebooks/fine_tune/unsloth_Llama3_1_8B_GRPO.html) |

---

## Related Topics

- [ROCm-7x](./ROCm-7x.md) — ROCm 7.x GPU compatibility and installation
- [bitsandbytes-amd](./bitsandbytes-amd.md) — Quantization on AMD ROCm
