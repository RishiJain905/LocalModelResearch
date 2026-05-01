# hardware-compilers

## 📌 Overview

Hardware compilers and GPU programming tools for ML training — covering Triton, Mojo, and AMD RDNA3/CDNA3 intrinsics, along with the ROCm and CUDA software stacks that lower kernels to silicon.

## Subtopics

| File | Description |
|------|-------------|
| [Triton-3.5.md](./Triton-3.5.md) | Triton 3.5 Python DSL compiler — fused kernels, FlashAttention, FP8/MXFP8 support, GEMM optimization, TMA (Hopper), Blackwell FP16, vLLM integration, comparison to CUDA/CUTLASS |
| [Mojo-AI.md](./Mojo-AI.md) | Mojo language for AI — 6-35x vs Python, MLIR-based compilation, SIMD/vectorization, Apple Silicon Metal GPU support, Modular MAX engine for inference serving, comparison to CUDA/Triton |
| [RDNA3-Intrinsics.md](./RDNA3-Intrinsics.md) | AMD RDNA3 intrinsics — wavefront programming, WMMA matrix instructions, ROCm HIP, CDNA3 vs RDNA3 for ML, MI300X/MI250X architecture, AMD GPU Operator for Kubernetes |

---

## Related

- [Kubernetes Training Infrastructure](../) — Parent topic
- [AMD ROCm](https://rocm.docs.amd.com/) — AMD GPU compute stack
- [NVIDIA Triton](https://triton-lang.org/) — GPU kernel compiler
- [Modular Mojo](https://docs.modular.com/mojo/) — AI compiler language

*Last updated: 2026-04-30*
