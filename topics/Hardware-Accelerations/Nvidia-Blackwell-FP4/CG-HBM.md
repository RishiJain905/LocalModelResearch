# CG-HBM (HBM Memory Subsystem)

> *"The dominant limiter in AI training is memory bandwidth and communication, not raw FLOPS."* — [Rambus](https://www.chipestimate.com/Breakthrough-to-Greater-Bandwidth-with-HBM4-Memory/Rambus/Technical-Article/2025/10/28)

## Contents

1. [Definitions & HBM Overview](#definitions--hbm-overview)
2. [Blackwell HBM3e Specifications](#blackwell-hbm3e-specifications)
3. [CoWoS Packaging Technology](#cowos-packaging-technology)
4. [HBM Generations Comparison](#hbm-generations-comparison)
5. [Benchmarks & Performance Data](#benchmarks--performance-data)
6. [Blog Posts & Articles](#blog-posts--articles)
7. [Summary Table](#summary-table)

---

> **Note:** The exact term *"CG-HBM (Graphics Cross Stack HBM)"* was not found in any authoritative NVIDIA documentation or academic source. This document covers the actual Blackwell memory subsystem — HBM3e bandwidth, CoWoS-L packaging, and die-to-die interconnect.

---

## Definitions & HBM Overview

### What is HBM?

> *"High Bandwidth Memory (HBM) is a high-performance 2.5D/3D memory architecture introduced in 2013. The core innovation is a wide 1024-bit data path running at relatively slow data rates for high bandwidth at low power."*
> — [Rambus](https://www.chipestimate.com/Breakthrough-to-Greater-Bandwidth-with-HBM4-Memory/Rambus/Technical-Article/2025/10/28)

### Memory-Bound vs Compute-Bound

> *"LLM inference repeatedly reads model weights and the KV cache from HBM and writes back the new key–value, and if bandwidth is insufficient, compute units sit idle."*
> *"For most LLM decode workloads the arithmetic intensity is low, so throughput is proportional to available memory bandwidth rather than peak FLOPS."*
> — [The Booher's Blog](https://theboohers.org/memory-bound-inference-why-high-bandwidth-memory-not-flops-sets-the-pace-in-ai-and-what-that-means-for-blackwell-vs-tpus/)

---

## Blackwell HBM3e Specifications

| Specification | Blackwell (B200) | Blackwell Ultra (B300) |
|---------------|-------------------|------------------------|
| **Memory Type** | HBM3e | HBM3e |
| **Capacity** | 192 GB | 288 GB |
| **Bandwidth** | 8 TB/s | 8 TB/s |
| **HBM Scaling vs H100** | 3.6x | 3.6x |
| **Configuration** | 8 × 12-Hi stacks, 16 × 512-bit controllers | 8 × 12-Hi stacks |
| **L2 Cache** | 128 MB | 192 MB |

> *"The memory subsystem delivers complete model residence for 300B+ parameter models without offloading."*
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/inside-nvidia-blackwell-ultra-the-chip-powering-the-ai-factory-era/)

### Die-to-Die Interconnect (NV-HBI)

> *"10 TB/s die-to-die bandwidth"* — NV-HBI (NVIDIA High-Bandwidth Interface) connecting the two Blackwell compute dies
> — [NVIDIA Developer Blog](https://developer.nvidia.com/blog/inside-nvidia-blackwell-ultra-the-chip-powering-the-ai-factory-era/)

### Microbenchmark Result

> *"TMEM sustained 8 TB/s memory bandwidth matching HBM3e peak performance, achieving a 2.1× improvement over conventional ld.global paths which plateau at 3.8 TB/s due to L1/L2 traversal overhead."*
> — [arXiv:2512.02189](https://arxiv.org/html/2512.02189v1)

---

## CoWoS Packaging Technology

### TSMC CoWoS Overview

> *"CoWoS® (Chip on Wafer on Substrate) is a family of advanced packaging technologies enabling heterogeneous integration for ultra-high performance computing applications including AI and supercomputing."*
> — [TSMC 3DFabric](https://3dfabric.tsmc.com/english/dedicatedFoundry/technology/cowos.htm)

### CoWoS-L (Blackwell Generation)

> *"CoWoS-L extends CoWoS-S by combining it with LSI (Local Silicon Interconnect) technology and silicon bridge stitching to break the 2500 mm² limit. Aimed at large-scale AI systems like NVIDIA Blackwell (GB200)."*
> — [AmiNext](https://www.aminext.blog/en/post/tsmc-cowos-s-r-l-differences)

> *"As we move into Blackwell, we will use largely CoWoS-L. We will also transition the CoWoS-S capacity to CoWoS-L."*
> — Jensen Huang, NVIDIA CEO — [Tom's Hardware](https://www.tomshardware.com/tech-industry/nvidia-shifts-to-cowos-l-packaging-for-blackwell-gpu-production-ramp-up)

### CoWoS-S vs CoWoS-L

| Attribute | CoWoS-S | CoWoS-L |
|-----------|---------|---------|
| **Interposer** | Silicon Interposer + TSV | Large stitched interposer with LSI |
| **Max Size** | ~2,500 mm² | Over 3,000 mm² |
| **Bandwidth** | High | Ultra-High |
| **Example Products** | NVIDIA H100, AMD MI300 | NVIDIA GB200 (Blackwell) |

> *"CoWoS-L uses 'local silicon interconnect (LSI) bridges' and 'organic interposer' that acts as a redistribution layer (RDL)"*
> — [Tom's Hardware](https://www.tomshardware.com/tech-industry/nvidia-shifts-to-cowos-l-packaging-for-blackwell-gpu-production-ramp-up)

---

## HBM Generations Comparison

| Generation | Data Rate | Interface Width | Bandwidth/Stack | Notes |
|------------|-----------|-----------------|-----------------|-------|
| HBM | 1 Gb/s | 1024-bit | — | Introduced 2013 |
| HBM2 | 2 Gb/s | 1024-bit | 256 GB/s | Used in V100, P100 |
| HBM3 | 6.4 Gb/s | 1024-bit | ~819 GB/s | Used in H100 |
| HBM3e | 9.6 Gb/s | 1024-bit | ~1.15 TB/s | Used in H200, B200, MI300X |
| **HBM4** | 8 Gb/s (std), 10 Gb/s (extended) | **2048-bit** | **>2 TB/s** | JEDEC finalized April 2025 |

> *"JEDEC released the official HBM4 specification in April 2025, doubling the interface width to 2,048 bits and enabling up to 2 terabytes per second bandwidth per stack."*
> — [Tom's Hardware](https://www.tomshardware.com/pc-components/ram/jedec-finalizes-hbm4-memory-standard-with-major-bandwidth-and-efficiency-upgrades)

> *"SK Hynix leads mass production with 12-high die stacks delivering 1.2+ terabytes per second bandwidth while remaining backward compatible with HBM3 controllers."*
> — [Introl](https://introl.com/blog/hbm-evolution-hbm3-hbm3e-hbm4-memory-ai-gpu-2025)

---

## Benchmarks & Performance Data

### NVIDIA GPU Memory Bandwidth Evolution

| GPU | Architecture | Memory | Bandwidth | Year |
|-----|-------------|--------|-----------|------|
| A100 | Ampere | HBM2e | 2.0 TB/s | 2020 |
| H100 | Hopper | HBM3 | 3.35 TB/s | 2022 |
| H200 | Hopper | HBM3e | 4.8 TB/s | 2023 |
| **B200** | Blackwell | HBM3e | **8 TB/s** | 2024 |
| **B300** | Blackwell Ultra | HBM3e | **8 TB/s** | 2025 |
| **R100** | Rubin | HBM4 | **22 TB/s** (est) | H2 2026 |

> — [Spheron](https://www.spheron.network/blog/nvidia-rubin-vs-blackwell-vs-hopper/)

### Blackwell vs Hopper — Key Memory Differences

| Feature | Hopper (H100) | Blackwell (B200) | Change |
|---------|---------------|------------------|--------|
| **Transistors** | 80 Billion | 208 Billion | 2.6x |
| **Memory Capacity** | 80 GB | 192 GB | 2.4x |
| **Memory Bandwidth** | 3.35 TB/s | 8 TB/s | **~2.4x** |
| **NVLink Bandwidth** | 900 GB/s | 1.8 TB/s | 2x |
| **FP4 Support** | No | Yes (native) | New |

> *"The H200 is an H100 with better memory. That single change makes a meaningful difference for memory-bandwidth-bound workloads."*
> — [Spheron](https://www.spheron.network/blog/nvidia-rubin-vs-blackwell-vs-hopper/)

### GB200 NVL72 System Memory

| Specification | Value |
|---------------|-------|
| **Total GPU Memory** | 13.4 TB HBM3e |
| **HBM3e Memory Bandwidth (system)** | 576 TB/s |
| **NVLink Switch Bandwidth (all-to-all)** | 130 TB/s |
| **Max model (FP16)** | ~671B parameters |

> — [Spheron](https://www.spheron.network/blog/nvidia-gb200-nvl72-guide/)

---

## Blog Posts & Articles

### Rubin vs Blackwell vs Hopper Comparison

> *"The H200 is an H100 with better memory. That single change makes a meaningful difference for memory-bandwidth-bound workloads like 70B model serving."*
> — [Spheron](https://www.spheron.network/blog/nvidia-rubin-vs-blackwell-vs-hopper/)

### HBM Evolution: HBM3 to HBM4

> *"JEDEC released the official HBM4 specification in April 2025, doubling the interface width to 2,048 bits and enabling up to 2 terabytes per second bandwidth per stack."*
> — [Introl](https://introl.com/blog/hbm-evolution-hbm3-hbm3e-hbm4-memory-ai-gpu-2025)

### Memory-Bound Inference Analysis

> *"LLM inference repeatedly reads model weights and the KV cache from HBM and writes back the new key–value, and if bandwidth is insufficient, compute units sit idle."*
> — [The Booher's Blog](https://theboohers.org/memory-bound-inference-why-high-bandwidth-memory-not-flops-sets-the-pace-in-ai-and-what-that-means-for-blackwell-vs-tpus/)

---

## Summary Table

### Blackwell Memory Specs

| Metric | B200 | B300 | GB200 NVL72 |
|--------|------|------|-------------|
| **Memory Type** | HBM3e | HBM3e | HBM3e |
| **Capacity** | 192 GB | 288 GB | 13.4 TB |
| **Bandwidth** | 8 TB/s | 8 TB/s | 576 TB/s |
| **vs H100 Bandwidth** | 3.6x | 3.6x | — |
| **L2 Cache** | 128 MB | 192 MB | — |
| **Die-to-Die (NV-HBI)** | 10 TB/s | 10 TB/s | — |

### HBM Generations

| Gen | Data Rate | Width | Bandwidth/Stack | Used By |
|-----|-----------|-------|-----------------|---------|
| HBM | 1 Gb/s | 1024-bit | — | P100, V100 |
| HBM2 | 2 Gb/s | 1024-bit | 256 GB/s | V100, A100 |
| HBM3 | 6.4 Gb/s | 1024-bit | ~819 GB/s | H100 |
| HBM3e | 9.6 Gb/s | 1024-bit | ~1.15 TB/s | H200, B200, MI300X |
| **HBM4** | 8–10 Gb/s | **2048-bit** | **>2 TB/s** | Rubin (H2 2026) |

---

## Key Findings

- **No "CG-HBM" terminology** found in authoritative sources — actual terms are GPU-to-HBM bandwidth, HBM subsystem, or memory bandwidth
- **Blackwell delivers 8 TB/s HBM3e bandwidth** — a 2.4x improvement over Hopper H100's 3.35 TB/s
- **CoWoS-L** is the Blackwell packaging technology — enabling dual-die design with 10 TB/s NV-HBI die-to-die interconnect
- **HBM4 specification finalized** (April 2025) — 2048-bit interface, >2 TB/s per stack, targeted for Rubin in H2 2026
- **LLM inference is memory-bandwidth bound** — not compute-bound, making HBM bandwidth the critical metric
- **GB200 NVL72**: 576 TB/s system memory bandwidth, can fit ~671B parameter models in FP16

---

## Related Topics

- [FP4-Tensor-Cores](./FP4-Tensor-Cores.md) — Native FP4 precision on Blackwell Tensor Cores
- [DGX-Spark](./DGX-Spark.md) — Personal AI supercomputer with GB10 Blackwell chip
