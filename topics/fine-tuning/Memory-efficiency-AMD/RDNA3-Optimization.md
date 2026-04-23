# RDNA3 Optimization

## 📌 Overview

| Field | Value |
|-------|-------|
| **Architecture** | AMD RDNA3 (consumer) — gfx1100, gfx1101, gfx1102 |
| **GPUs** | RX 7900 XTX/XT, RX 7800 XT, RX 7700 XT, PRO W7900 |
| **Memory** | Up to 24GB GDDR6 on RX 7900 XTX |
| **ROCm Support** | Official since ROCm 5.7 (Ubuntu 22.04 only) |
| **Key Advantage** | 24GB VRAM at ~80-90% of RTX 4090 price |

> "For single batch inference performance, it can reach 80% of the speed of NVIDIA 4090."

— [MLC.ai Blog, August 2023](https://blog.mlc.ai/2023/08/09/Making-AMD-GPUs-competitive-for-LLM-inference)

---

## Definitions

### RDNA3 vs CDNA3

| Architecture | Market | GPUs | ROCm Support |
|-------------|--------|-----|-------------|
| **RDNA3** | Consumer | RX 7900 XTX, RX 7800 XT | Partial (gfx1100 only) |
| **CDNA3** | Data center | MI300X, MI250, MI210 | Full official |

### WMMA (Wave Matrix Multiply-Accumulate)

> "AMD adds WMMA support to GFX11 (RDNA3) architecture."

— [VideoCardz](https://videocardz.com/newz/amd-adds-wmma-wave-matrix-multiply-accumulate-support-to-gfx11-rdna3-architecture-amds-tensor-core)

AMD's equivalent of NVIDIA's Tensor Cores — enables 16×16×16 FP16/BF16 matrix operations via the `rocWMMA` library.

---

## ROCm Support Timeline

| Version | RDNA3 Consumer Support |
|---------|------------------------|
| ROCm 5.7 | Initial RX 7900 XTX support |
| ROCm 6.0 | PRO W7900/W7800 + RX 7900 GRE |
| ROCm 7.0 | vLLM first-class support |
| ROCm 7.1 | Broader tool support |
| ROCm 7.2 | RX 9070 XT (RDNA4) |

---

## Key Sources

### AMD ROCm Blog — LLM Inference Optimizations

| URL | [rocm.blogs.amd.com](https://rocm.blogs.amd.com/artificial-intelligence/llm-inference-optimize/README.html) |
|-----|------|

> "Techniques including PyTorch 2 compilation, Flash Attention v2, Paged Attention can deliver up to 3x acceleration."

> "Flash Attention v2 tiles partial queries into faster cache memory."

> "Paged Attention manages K-V cache using virtual memory to reduce fragmentation."

### AMD ROCm Blog — vLLM First-Class Support (January 2026)

| URL | [rocm.blogs.amd.com](https://rocm.blogs.amd.com/software-tools-optimization/vllm-omni/README.html) |
|-----|------|

> "AMD ROCm is now a first-class platform in the vLLM ecosystem."

> "In Nov 2025, 37% CI passing. Jan 2026, 93% passing."

### MLC-LLM — AMD Competitive for LLM Inference

| URL | [blog.mlc.ai](https://blog.mlc.ai/2023/08/09/Making-AMD-GPUs-competitive-for-LLM-inference) |
|-----|------|

> "For single batch inference performance, it can reach 80% of the speed of NVIDIA 4090."

**Benchmark**: Llama 2 7B/13B 4-bit on RX 7900 XTX:
- 80% of RTX 4090 performance
- 94% of RTX 3090 Ti performance

### ROCm Consumer GPU Guide (2026)

| URL | [kunalganglani.com](https://www.kunalganglani.com/blog/rocm-consumer-gpu-cuda-alternative-2026) |
|-----|------|

> "ROCm 7.2 now officially supports consumer Radeon GPUs — not just data center Instinct cards."

> "The RX 9070 XT is currently the safest consumer bet for explicit ROCm support."

> "Don't assume RDNA 3 means universal ROCm support. Check the specific LLVM target."

---

## Tools & Libraries

| Tool | RDNA3 Support | Notes |
|------|---------------|-------|
| [llama.cpp](https://github.com/ggerganov/llama.cpp) | ✅ Full | HIP/ROCm backend, Flash Attention |
| [vLLM](https://docs.vllm.ai/en/v0.5.1/getting_started/amd-installation.html) | ✅ v0.14.0+ | Official since January 2026 |
| PyTorch ROCm | ✅ | `pip install --index-url https://download.pytorch.org/whl/rocm6.2` |
| [MLC-LLM](https://blog.mlc.ai/2023/08/09/Making-AMD-GPUs-competitive-for-LLM-inference) | ✅ | 80% RTX 4090 performance |
| TensorFlow ROCm | ⚠️ Lagging | Limited optimization |
| JAX ROCm | ⚠️ Limited | Minimal RDNA3 support |

### ROCm Libraries

| Library | Purpose |
|---------|---------|
| hipBLAS / hipBLASLt | BLAS operations |
| rocWMMA | WMMA C++ API for wavefront matrix ops |
| flash-attention | ROCm FA v2 (v2.0.4) |
| Triton | FA backend for vLLM |
| Composable Kernel | GEMM operations |

---

## Benchmark Results

### llama.cpp on RX 7900 XTX (ROCm 7.1.1)

| Model | Test | FA=0 | FA=1 |
|-------|------|------|------|
| Llama 7B Q4_0 | pp512 | 3,487–3,550 t/s | 3,785–3,800 t/s |
| Llama 7B Q4_0 | tg128 | 118–130 t/s | 126–138 t/s |
| Llama 70B Q4_K_M (dual) | pp512 | 330 t/s | 341 t/s |
| Llama 70B Q4_K_M (dual) | tg128 | 13 t/s | 13.4 t/s |

Source: [1337hero/rx7900xtx-llama-bench-rocm](https://github.com/1337hero/rx7900xtx-llama-bench-rocm)

### Performance vs NVIDIA

| GPU | VRAM | vs RTX 4090 |
|-----|------|-------------|
| RX 7900 XTX | 24GB | 80–90% |

---

## LLVM Targets

| GPU | LLVM Target | Official Support |
|-----|-----------|----------------|
| RX 7900 XTX / 7900 XT | gfx1100 | ROCm 5.7+ ✅ |
| RX 7800 XT | gfx1102 | Community only ⚠️ |
| PRO W7900 | gfx1100 | ROCm 6.0+ ✅ |
| RX 9070 XT (RDNA4) | gfx1201 | ROCm 7.2+ ✅ |

> "Don't assume RDNA 3 means universal ROCm support. Check the specific LLVM target."

---

## Workarounds & Common Issues

| Issue | Solution |
|-------|---------|
| Secure Boot blocking amdgpu | Disable Secure Boot |
| Unsupported card | `HSA_OVERRIDE_GFX_VERSION=11.0.0` |
| Suspend/resume broken | Run `rocm-smi` loop on boot |
| BIOS freeze | Switch OC to Silent mode |
| FA not working on gfx1100 (vLLM) | `BUILD_FA=0` |
| Process hangs | `--enforce-eager` flag |

---

## Limitations

### OS Support
- **Primary**: Ubuntu 22.04.5 LTS, Ubuntu 24.04.3
- **Supported**: RHEL 8.10+, SLES 15 SP6+, Debian 12
- **Not supported**: Windows (use WSL2 instead)

### Multi-GPU
- ROCm 5.7: Preview only
- Requires identical PCIe lane width
- PCIe 3.0 Atomics required

### RX 7800 XT
> "Different LLVM target `gfx1102` — community support only"

---

## Summary Data Points

| Metric | Value |
|--------|-------|
| RX 7900 XTX inference speed | 80-90% of RTX 4090 |
| RX 7900 XTX VRAM | 24GB GDDR6 |
| vLLM CI passing (Jan 2026) | 93% |
| Llama 7B Q4_0 pp512 (FA=1) | 3,800 t/s |
| ROCm 7.2 support | RX 9070 XT (RDNA4) |
| Multi-GPU | Preview only (ROCm 5.7) |

---

## Related Topics

- [HSA_OVERRIDE_GFX.md](./HSA_OVERRIDE_GFX.md) — Environment variable for unsupported GPUs
- [Memory-efficiency-AMD](./README.md) — Parent folder overview

---

## References

- [AMD ROCm — LLM Inference Optimizations](https://rocm.blogs.amd.com/artificial-intelligence/llm-inference-optimize/README.html)
- [AMD ROCm — vLLM First-Class Support](https://rocm.blogs.amd.com/software-tools-optimization/vllm-omni/README.html)
- [MLC.ai — AMD Competitive for LLM Inference](https://blog.mlc.ai/2023/08/09/Making-AMD-GPUs-competitive-for-LLM-inference)
- [ROCm Compatibility Matrix](https://rocm.docs.amd.com/en/latest/compatibility/compatibility-matrix.html)
- [vLLM AMD Installation](https://docs.vllm.ai/en/v0.5.1/getting_started/amd-installation.html)
- [llama.cpp RX 7900 XTX Benchmarks](https://github.com/1337hero/rx7900xtx-llama-bench-rocm)
- [Are We gfx1100 Yet](https://are-we-gfx1100-yet.github.io/) — Community tracker
