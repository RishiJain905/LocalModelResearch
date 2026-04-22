# Heterogenous-NPU-GPU

> *Heterogeneous NPU+GPU architectures combine dedicated neural processing units with graphics processing units to distribute AI workloads — NPUs handle sustained low-power inference while GPUs accelerate parallel compute-heavy operations.*

---

## Overview

This folder covers the emerging landscape of **heterogeneous NPU+GPU compute** — where dedicated neural processing units work alongside (or instead of) traditional GPUs to deliver efficient AI acceleration across mobile PCs, edge devices, and data center SoCs.

Key themes:
- **Intel Panther Lake** — Intel 18A with NPU 5 + Xe3 GPU (up to 180 platform TOPS)
- **Qualcomm Snapdragon X2 Elite** — Hexagon NPU 6 (80 TOPS) + Adreno GPU + Oryon CPU
- **OpenVINO** — Intel's heterogeneous inference framework for CPU+GPU+NPU deployment

---

## Subtopics

| Document | Description |
|----------|-------------|
| [Intel-Panther-Lake.md](./Intel-Panther-Lake.md) | Core Ultra Series 3, Intel 18A, RibbonFET, NPU 5, Xe3 GPU, 180 TOPS |
| [Snapdragon-X2.md](./Snapdragon-X2.md) | Hexagon NPU 6, 80 TOPS, Oryon CPU, Adreno X2, Windows Copilot+ |
| [OpenVINO.md](./OpenVINO.md) | Heterogeneous inference framework, HETERO plugin, CPU/GPU/NPU |
| [HBM4.md](./HBM4.md) | JEDEC JESD270-4, 2TB/s+ bandwidth, 2048-bit interface, Samsung/Micron/SK Hynix |
| [CXL-3.0-Memory-Pooling.md](./CXL-3.0-Memory-Pooling.md) | PCIe 6.0, 64GT/s, memory pooling, 4,096 fabric nodes, KV cache acceleration |
| [LPDDR5X.md](./LPDDR5X.md) | JESD209-5B/C, 10.7Gbps, 25% power efficiency gain, on-device AI memory |

---

## Intel Panther Lake — Key Data Points

| Metric | Value |
|--------|-------|
| **Process Node** | Intel 18A (RibbonFET + PowerVia) |
| **Total AI TOPS** | Up to 180 (NPU: 50, GPU: 120, CPU: 10) |
| **GPU Architecture** | Xe3 "Celestial" (Arc B390, up to 12 Xe cores) |
| **NPU** | 5th-generation, 50 TOPS INT8 |
| **Gaming vs AMD** | +82% faster than Ryzen AI 9 HX 370 |
| **LLM Inference** | 4.3x faster vs AMD XDNA2 |
| **Battery (video)** | Up to 27 hours |

> *"Instead of pushing AI tasks onto the CPU or GPU directly, it will select which ones are more suitable for the NPU and then let the NPU handle them efficiently and continuously."*
> — ASUS USA Blog

---

## Qualcomm Snapdragon X2 — Key Data Points

| Metric | Value |
|--------|-------|
| **NPU** | Hexagon NPU 6 (80 TOPS) |
| **CPU** | Oryon (up to 18 cores, 5.0 GHz) |
| **GPU** | Adreno X2-90 |
| **Memory Bandwidth** | 228 GB/s |
| **Geekbench AI** | 88,615 (highest among all Copilot+ chips) |
| **Battery (sustained perf)** | Maintains performance on battery; x86 drops ~50% |

> *"Qualcomm has officially moved past the 'alternative' phase and into a 'serious challenger' position."*
> — PCMag

---

## OpenVINO — Key Data Points

| Metric | Value |
|--------|-------|
| **License** | Apache 2.0 |
| **Latest Release** | 2026.1.0 (April 7, 2026) |
| **Stars** | 10.1k+ |
| **NPU Speedup (CNN)** | 3-5× vs CPU (FP16) |
| **NPU Limitation** | Static shapes only |
| **HETERO Plugin** | ✅ CPU+GPU+NPU fallback |
| **Quantization** | INT8, INT4 via NNCF |

> *"Heterogeneous Execution enables running inference of one model across multiple devices simultaneously."*
> — docs.openvino.ai

---

## HBM4 — Key Data Points

| Metric | Value |
|--------|-------|
| **JEDEC Standard** | JESD270-4 (finalized April 2025) |
| **Interface Width** | 2,048-bit (2× HBM3E) |
| **Bandwidth/Stack** | 2.0–3.3 TB/s |
| **Channels/Stack** | 32 (2× HBM3E) |
| **Max Capacity** | 64 GB per stack |
| **Data Rate** | 8–13 Gbps |
| **AI Training vs HBM3E** | 40–70% faster |

> *"HGM4" was not found as a recognized standard — interpreted as HBM4.*

---

## CXL 3.0 Memory Pooling — Key Data Points

| Metric | Value |
|--------|-------|
| **Base Standard** | PCIe 6.0 |
| **Lane Speed** | 64 GT/s |
| **Bandwidth (x16)** | 256 GB/s |
| **Latency** | Zero added vs CXL 2.0 |
| **Max Fabric Nodes** | 4,096 |
| **LLM Inference Gain** | 21.9× KV cache throughput vs baseline |
| **RDMA Speedup** | 3.8× vs 200G RDMA |

> *"CXL Memory pool is the critically missing component in the systems for AI workloads."*
> — CXL Consortium

---

## LPDDR5X — Key Data Points

| Metric | Value |
|--------|-------|
| **JEDEC Standard** | JESD209-5B/C |
| **Max Data Rate** | 10.7 Gbps |
| **Bandwidth (x64)** | ~68–85.6 GB/s |
| **Power Efficiency** | ~25% better vs LPDDR5 |
| **Apple M5 Bandwidth** | 153 GB/s (M5), 307 GB/s (M5 Pro), 614 GB/s (M5 Max) |
| **AI Decode Gain** | >9% per speed grade increment |

> *"LPDDR5X is transcending mobile devices — expanding into PCs, accelerators, servers, and automobiles."*
> — Samsung Semiconductor

---

## Cross-Platform AI TOPS Comparison

| Chip | CPU TOPS | GPU TOPS | NPU TOPS | **Total** |
|------|----------|----------|----------|-----------|
| **Intel Core Ultra X9 388H** | ~12 | ~120 | 50 | **~180** |
| **Qualcomm X2 Elite Extreme** | — | — | 80 | **80+** |
| AMD Ryzen AI 9 HX 370 | ~15 | ~15 | 50 | ~80 |
| Apple M5 | — | — | ~57 | ~57 |

---

## Related Research

- **Nvidia-Blackwell-FP4** — Discrete GPU AI acceleration with native FP4 support
- **Apple-M5-Fusion** — Apple Silicon neural accelerators and MLX framework

---

*Folder structure: `topics/Hardware-Accelerations/Heterogenous-NPU-GPU/`*
