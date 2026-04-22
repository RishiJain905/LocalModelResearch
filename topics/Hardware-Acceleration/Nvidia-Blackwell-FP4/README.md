# Nvidia-Blackwell-FP4

> *NVIDIA Blackwell is the architecture succeeding Hopper, featuring native FP4 Tensor Cores, HBM3e memory at 8 TB/s, and a dual-die design with 208 billion transistors.* — Compiled from NVIDIA documentation

## Overview

This folder covers NVIDIA's **Blackwell architecture** and its **FP4 precision support** — the first GPU architecture to natively support 4-bit floating-point inference through 5th-generation Tensor Cores.

**Key highlights:**
- **NVFP4**: Native 4-bit floating-point format with dual-scaling (E4M3 per-block + FP32 per-tensor)
- **18,000 FP4 TOPS** (sparse) on B200 datacenter GPU
- **8 TB/s HBM3e memory bandwidth** — 2.4x vs Hopper H100
- **DGX Spark**: Personal AI supercomputer at $4,699 with FP4 support

---

## Subtopics

| Document | Description |
|----------|-------------|
| [FP4-Tensor-Cores](./FP4-Tensor-Cores.md) | NVFP4 format, dual-scaling, benchmarks, tools |
| [CG-HBM](./CG-HBM.md) | HBM3e memory subsystem (8 TB/s), CoWoS-L packaging |
| [DGX-Spark](./DGX-Spark.md) | Personal AI supercomputer ($4,699, GB10 chip) |

---

## Architecture Summary

### Blackwell Product Line

| Product | GPU | FP4 TOPS | Memory | Bandwidth | Price |
|---------|-----|----------|--------|-----------|-------|
| **B200** (datacenter) | 208B transistor dual-die | 18,000 sparse | 192 GB HBM3e | 8 TB/s | ~$30-40k/GPU |
| **B300** (Ultra) | — | — | 288 GB HBM3e | 8 TB/s | — |
| **GB200 NVL72** | 72× B200 | 1.44 exaflops | 13.4 TB | 576 TB/s | Rack-scale |
| **GB10** (DGX Spark) | Single chip | ~1,000 | 128 GB LPDDR5x | 273 GB/s | $4,699 |
| **RTX 5090** (consumer) | 92.2B transistor | 1,676 sparse | 32 GB GDDR7 | — | ~$1,999 |

### NVFP4 Format (E2M1)

| Component | Bits |
|-----------|------|
| Sign | 1 |
| Exponent | 2 (bias of 1) |
| Mantissa | 1 |
| **Total** | **4 bits** |

**Dual-scaling**: FP8 (E4M3) per 16-value block + FP32 per tensor

### GPU Performance Evolution

| GPU | FP4 (sparse) | vs H100 |
|-----|-------------|---------|
| H100 | — (FP8 max) | baseline |
| B200 | 18,000 TOPS | 4.7x |
| B300 | 18,000 TOPS | 4.7x |
| GB300 | 20,000 TOPS | 5.3x |

### Memory Bandwidth Evolution

| GPU | Bandwidth | vs H100 |
|-----|-----------|---------|
| H100 | 3.35 TB/s | baseline |
| B200 | **8 TB/s** | **2.4x** |
| B300 | **8 TB/s** | **2.4x** |
| Rubin (R100) | 22 TB/s (est) | 6.6x |

---

## Key Sources

| Source | URL |
|--------|-----|
| NVIDIA Developer Blog — NVFP4 Introduction | [developer.nvidia.com/blog/introducing-nvfp4](https://developer.nvidia.com/blog/introducing-nvfp4-for-efficient-and-accurate-low-precision-inference/) |
| NVIDIA Developer Blog — NVFP4 Training | [developer.nvidia.com/blog/nvfp4-trains](https://developer.nvidia.com/blog/nvfp4-trains-with-precision-of-16-bit-and-speed-and-efficiency-of-4-bit/) |
| NVIDIA Developer Blog — DeepSeek-R1 | [developer.nvidia.com/blog/deepseek-r1](https://developer.nvidia.com/blog/nvidia-blackwell-delivers-world-record-deepseek-r1-inference-performance/) |
| NVIDIA Blackwell Architecture Brief | [cdn.prod.website-files.com/.../nvidia-blackwell-architecture-technical-brief.pdf](https://cdn.prod.website-files.com/61dda201f29b7efc52c5fbaf/6602ea9d0ce8cb73fb6de87f_nvidia-blackwell-architecture-technical-brief.pdf) |
| RTX Blackwell Whitepaper | [images.nvidia.com/.../nvidia-rtx-blackwell-gpu-architecture.pdf](https://images.nvidia.com/aem-dam/Solutions/geforce/blackwell/nvidia-rtx-blackwell-gpu-architecture.pdf) |
| DGX Spark Hardware Overview | [docs.nvidia.com/dgx/dgx-spark/hardware.html](https://docs.nvidia.com/dgx/dgx-spark/hardware.html) |
| TSMC CoWoS | [3dfabric.tsmc.com/.../cowos.htm](https://3dfabric.tsmc.com/english/dedicatedFoundry/technology/cowos.htm) |

---

## Related Topics

- [FP4-Tensor-Cores](./FP4-Tensor-Cores.md) — Native FP4 precision
- [CG-HBM](./CG-HBM.md) — HBM3e memory subsystem
- [DGX-Spark](./DGX-Spark.md) — Personal AI supercomputer
