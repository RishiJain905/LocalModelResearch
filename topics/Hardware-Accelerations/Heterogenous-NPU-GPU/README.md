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
