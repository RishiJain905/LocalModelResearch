# Qualcomm Snapdragon X2 Elite

> **Qualcomm Snapdragon X2 Elite** is the second-generation Snapdragon X series processor for Windows Copilot+ PCs, announced September 2025 with availability in H1 2026. It features the **Hexagon NPU 6** delivering **80 TOPS**, the **Adreno X2 GPU**, and **Oryon CPU** cores — targeting the heterogeneous NPU+GPU+CPU compute landscape for on-device AI.

---

## Table of Contents

1. [Definitions & Specifications](#1-definitions--specifications)
2. [Key Sources & Quotes](#2-key-sources--quotes)
3. [Benchmarks & Performance Data](#3-benchmarks--performance-data)
4. [Blog Posts & Articles](#4-blog-posts--articles)
5. [Comparison with Alternatives](#5-comparison-with-alternatives)
6. [Summary Table](#6-summary-table)

---

## 1. Definitions & Specifications

### What is Snapdragon X2 Elite?

> *"Snapdragon X2 Elite strengthens our leadership in the PC industry, providing legendary leaps in performance, AI processing and battery life to enable the experiences that consumers deserve."*
> — Kedar Kondap, Senior Vice President and General Manager, Compute and Gaming, Qualcomm Technologies, Inc.
> — [Qualcomm Press Release, September 24, 2025](https://www.qualcomm.com/news/releases/2025/09/new-snapdragon-x2-elite-extreme-and-snapdragon-x2-elite-are-the-)

> *"The platform boasts up to 31% faster performance at ISO power and requires up to 43% less power than the previous generation."*
> — [Qualcomm Press Release, September 24, 2025](https://www.qualcomm.com/news/releases/2025/09/new-snapdragon-x2-elite-extreme-and-snapdragon-x2-elite-are-the-)

### Snapdragon X2 Elite Family Specifications

| Specification | X2 Elite Extreme (X2E-96-100) | X2 Elite (X2E-88-100) | X2 Elite (X2E-80-100) | X2 Plus (X2P-64-100) |
|---------------|-------------------------------|------------------------|------------------------|---------------------|
| **CPU Cores** | 18 (12 Prime + 6 Performance) | 18 (12 Prime + 6 Performance) | 12 (6 Prime + 6 Performance) | 10 (6P+4E) |
| **Max Boost Clock** | 5.0 GHz | 4.7 GHz | 4.7 GHz | 4.0 GHz |
| **All-Core Freq** | 4.4 GHz | 4.0 GHz | 4.0 GHz | — |
| **Total Cache** | 53 MB | 53 MB | 34 MB | 34 MB |
| **GPU** | Adreno X2-90 | Adreno X2-90 | Adreno X2-85 | Adreno X2-45 |
| **NPU TOPS** | 80 | 80 | 80 | 80 |
| **Memory** | LPDDR5x-9523, 192-bit, 228 GB/s | LPDDR5x-9523, 192-bit, 228 GB/s | LPDDR5x, 128-bit, 152 GB/s | 128 GB/s |
| **Max Memory** | 128+ GB | 128+ GB | 128 GB | — |
| **Process Node** | 3nm | 3nm | 3nm | 3nm |

### Snapdragon X2 Plus (CES 2026)

- **X2P-64-100:** 10-core (6P+4E), 34MB cache, 80 TOPS NPU, up to 4.0 GHz
- **X2P-42-100:** 6-core, 22MB cache, 80 TOPS NPU

### Hexagon NPU 6 Architecture

> *"The Hexagon NPU, designed from the ground up for accelerating AI inference at low power, features the industry's most advanced NPU architecture—evolving along with the development of new AI use cases, models, and requirements."*
> — [Qualcomm - Hexagon NPU](https://www.qualcomm.com/processors/hexagon)

> *"The Hexagon NPU fuses together the scalar, vector, and tensor accelerators for better performance and power efficiency."*
> — [Qualcomm - Hexagon NPU](https://www.qualcomm.com/processors/hexagon)

The Hexagon NPU executes three instruction sets:

| Instruction Set | Purpose | Data Types |
|-----------------|---------|-------------|
| **Scalar** | Control flow and general purpose | Int: 8,16,32,64 / Float: 32,64 |
| **Vector** | General purpose data-parallel compute | Int: 8,16,32 / Float: 16,32 |
| **Tensor** | Matrix multiply, convolutional layers | Int: 4,8,16 / Float: 16 |

> *"Processor executing 3 instruction sets: Scalar: For control flow and general purpose. Vector: General purpose data-parallel compute. Tensor: Matrix multiply, convolutional layers."*
> — [Hot Chips 2023 - Qualcomm Hexagon Tensor Processor](https://www.hc2023.hotchips.org/assets/program/conference/day2/ML%20Inference/HC2023%20Qualcomm%20Hexagon%20NPU.pdf)

**Hexagon NPU 6 improvements over previous generation:**

| Metric | Improvement |
|--------|-------------|
| **Throughput** | +143% (via 12 threads, 4-wide VLIW per thread) |
| **Bus bandwidth** | +127% faster (8 parallel threads) |
| **Vector processing** | +143% faster (FP8 and BF16 support) |
| **Matrix unit** | +78% faster (independent power rail) |
| **Power efficiency** | 43% reduction at ISO power |

### Heterogeneous Compute Components

Snapdragon X2 integrates four compute engines:

| Component | Role |
|----------|------|
| **Hexagon NPU** | Dedicated AI acceleration (80 TOPS) |
| **Adreno GPU** | Graphics and GPU-compute workloads |
| **Oryon CPU** | General computation |
| **Qualcomm Sensing Hub** | Additional micro NPU for background tasks |

---

## 2. Key Sources & Quotes

### Source 1: Qualcomm Press Release — Snapdragon X2 Elite Launch

- **URL:** https://www.qualcomm.com/news/releases/2025/09/new-snapdragon-x2-elite-extreme-and-snapdragon-x2-elite-are-the-
- **Date:** September 24, 2025
- **Author:** Qualcomm Technologies, Inc.

> *"Snapdragon X2 Elite strengthens our leadership in the PC industry, providing legendary leaps in performance, AI processing and battery life to enable the experiences that consumers deserve."*
> — Kedar Kondap, Senior Vice President and General Manager, Compute and Gaming

> *"The platform boasts up to 31% faster performance at ISO power and requires up to 43% less power than the previous generation."*

### Source 2: Qualcomm Hexagon NPU Official Page

- **URL:** https://www.qualcomm.com/processors/hexagon

> *"The Hexagon NPU, designed from the ground up for accelerating AI inference at low power, features the industry's most advanced NPU architecture—evolving along with the development of new AI use cases, models, and requirements."*

> *"The Hexagon NPU fuses together the scalar, vector, and tensor accelerators for better performance and power efficiency."*

### Source 3: Hot Chips 2023 — Qualcomm Hexagon Tensor Processor

- **URL:** https://www.hc2023.hotchips.org/assets/program/conference/day2/ML%20Inference/HC2023%20Qualcomm%20Hexagon%20NPU.pdf

**Vector unit specs:**

> *"Width: 1 Kb = 128B = 64H = 32W. Resources: 6 VLIW slots (4 compute, 1 load, 1 store). Registers: 32 vector registers + 4 vector predicate registers."*

### Source 4: Windows Central — Snapdragon X2 Elite vs X Elite

- **URL:** https://www.windowscentral.com/hardware/qualcomm/snapdragon-x2-elite-vs-x-elite

> *"Qualcomm has bumped the NPU up to a whopping 80 TOPS across all the new X2 Elite chips."*

### Source 5: PCMag — "I Tested Qualcomm's Snapdragon X2 Elite Extreme"

- **URL:** https://www.pcmag.com/news/i-tested-qualcomm-snapdragon-x2-elite-extreme-18-core-power-hits-hard

> *"Qualcomm has officially moved past the 'alternative' phase and into a 'serious challenger' position."*

> *"Over 800-point jump in single-core performance and a staggering 6,000-point-plus increase in multi-core performance."*

> *"The critical finding that AMD and Intel chips lose approximately 50% of their performance when unplugged is perhaps the most consequential data point in these reviews."*

### Source 6: Tom's Guide — Apple M5 vs Intel Panther Lake vs Snapdragon X2

- **URL:** https://www.tomsguide.com/computing/apple-m5-vs-intel-vs-amd-vs-snapdragon-x2-which-chip-wins

**Winner Table:**

| Category | Winner |
|----------|--------|
| Single-Core CPU | Apple M5 |
| Multi-Core CPU | Apple M5 Max |
| Graphics | Apple M5 Max |
| AI Performance | Snapdragon X2 |
| Battery Life | Apple M5 Pro |

---

## 3. Benchmarks & Performance Data

### NPU / AI Performance

| Benchmark | Snapdragon X2 Elite Extreme | Intel Core Ultra 9 285H | Apple M4 |
|-----------|---------------------------|------------------------|----------|
| Procyon AI Computer Vision | 5.7x faster than Intel | Baseline | — |
| Geekbench AI | **88,615** | ~55,000 | ~57,000 |
| vs Apple M4 | 95% faster | — | Baseline |

### CPU Performance

| Benchmark | X2 Elite Extreme | Intel Core Ultra X9 388H | Advantage |
|-----------|-------------------|--------------------------|-----------|
| Geekbench 6 Single-Core | 3,807 | 3,066 | **+24%** |
| Geekbench 6 Multi-Core | 23,198 | 17,924 | **+29%** |

### Tom's Guide Comprehensive Benchmark Data

**Geekbench 6 CPU:**

| Chip | Single-Core | Multi-Core |
|------|-------------|------------|
| M5 Max | 4,338 | 29,430 |
| M5 Pro | 4,306 | 28,586 |
| M5 | 4,288 | 17,926 |
| **Snapdragon X2 Elite Extreme** | **4,070** | **23,407** |
| Intel Core Ultra X9 388H | 3,031 | 17,283 |
| AMD Ryzen AI Max+ 395 | 2,932 | 18,407 |

**Geekbench AI (NPU) Performance:**

| Chip | AI Score |
|------|----------|
| **Snapdragon X2 Elite Extreme** | **88,615** |
| Snapdragon X2 Elite (18-core) | 87,814 |
| M5 Pro | 57,420 |
| Intel Core Ultra X9 388H | 56,829 |
| AMD Ryzen AI Max+ 395 | 17,854 |

> *"Qualcomm dominates AI with 88K+ scores—55% ahead of Apple's M5 (~57K)."*
> — Tom's Guide

### Battery Performance (Critical Finding)

> *"The critical finding that AMD and Intel chips lose approximately 50% of their performance when unplugged is perhaps the most consequential data point in these reviews."*
> — [PCMag](https://www.pcmag.com/news/i-tested-qualcomm-snapdragon-x2-elite-extreme-18-core-power-hits-hard)

### Hexagon NPU 6 Performance Gains (vs Previous Generation)

| Metric | Improvement |
|--------|-------------|
| **NPU TOPS** | 80 (up from 45) — 78% increase |
| **Scalar throughput** | +143% |
| **Vector processing** | +143% |
| **Matrix unit** | +78% |
| **Power efficiency** | 43% reduction at ISO power |

---

## 4. Blog Posts & Articles

| Title | URL | Author | Date |
|-------|-----|--------|------|
| I Tested Qualcomm's Snapdragon X2 Elite Extreme | https://www.pcmag.com/news/i-tested-qualcomm-snapdragon-x2-elite-extreme-18-core-power-hits-hard | PCMag | 2026 |
| Apple M5 vs. Intel Panther Lake vs. Snapdragon X2 — Benchmark Showdown | https://www.tomsguide.com/computing/apple-m5-vs-intel-vs-amd-vs-snapdragon-x2-which-chip-wins | Tony Polanco | 2026 |
| Snapdragon X2 Elite Review | https://tech-insider.org/qualcomm-snapdragon-x2-elite-review-benchmarks-2026/ | Tech Insider | 2026 |
| Snapdragon X2 Elite vs X Elite | https://www.windowscentral.com/hardware/qualcomm/snapdragon-x2-elite-vs-x-elite | Windows Central | 2025 |

---

## 5. Comparison with Alternatives

### vs Apple M5 Neural Engine

| Feature | Snapdragon X2 Elite | Apple M5 |
|---------|--------------------|----------|
| **Neural Engine** | Hexagon NPU 6 (80 TOPS) | 16-core Neural Engine |
| **Peak GPU AI Compute** | — | over 4× faster than M4 |
| **Geekbench AI** | 88,615 | 57,420 |
| **Memory Bandwidth** | 228 GB/s (X2E-96) | Higher on Pro/Max |

> *"M5 Pro: 4× faster LLM prompt processing vs M4 Pro."*
> — Apple Newsroom

### vs Intel Panther Lake

| Feature | Snapdragon X2 Elite | Intel Panther Lake |
|---------|--------------------|-------------------|
| **Architecture** | ARM | x86-64 |
| **AI TOPS** | 80+ | 172-180 |
| **NPU TOPS** | 80 | 50 |
| **Geekbench AI** | 88,615 | 56,829 |
| **Gaming** | Lower | **10-20% faster** |
| **Battery (sustained performance)** | **Maintains performance** | Drops ~50% unplugged |

### vs AMD Strix Halo (Ryzen AI Max)

| Feature | Snapdragon X2 Elite | AMD Strix Halo |
|---------|--------------------|----------------|
| **AI TOPS** | 80 | ~50 |
| **Memory Bandwidth** | 228 GB/s | LPDDR5x-8000 |
| **Max Memory** | 128+ GB | 128 GB |
| **Battery Life** | ~20 hrs | 11 hrs |
| **NPU Architecture** | Hexagon 6 | XDNA 2 |

---

## 6. Summary Table

### Snapdragon X2 Elite Family Specifications

| Feature | X2E-96-100 (Extreme) | X2E-88-100 | X2E-80-100 | X2P-64-100 (Plus) |
|---------|---------------------|------------|------------|-------------------|
| **Cores** | 18 (12P+6E) | 18 (12P+6E) | 12 (6P+6E) | 10 (6P+4E) |
| **Max Clock** | 5.0 GHz | 4.7 GHz | 4.7 GHz | 4.0 GHz |
| **Cache** | 53 MB | 53 MB | 34 MB | 34 MB |
| **GPU** | X2-90 | X2-90 | X2-85 | X2-45 |
| **NPU TOPS** | 80 | 80 | 80 | 80 |
| **Memory BW** | 228 GB/s | 228 GB/s | 152 GB/s | 128 GB/s |
| **Process** | 3nm | 3nm | 3nm | 3nm |

### Competitive Position

| Metric | Snapdragon X2 Elite | Apple M5 | Intel Panther Lake | AMD Strix Halo |
|--------|--------------------|----------|-------------------|----------------|
| **NPU TOPS** | 80 | ~57 (est.) | ~48 (est.) | ~50 |
| **Single-core Geekbench 6** | 4,070 | 4,288+ | 3,031 | 2,932 |
| **Multi-core Geekbench 6** | 23,407 | 29,430 | 17,283 | 18,407 |
| **Battery Life** | ~20 hrs | 21+ hrs | 14-20 hrs | 11 hrs |
| **Geekbench AI** | **88,615** | 57,420 | 56,829 | 17,854 |

### Windows AI Ecosystem

| Component | Description |
|----------|-------------|
| **ONNX Runtime** | QNN Execution Provider enables hardware-accelerated inference on Hexagon NPU |
| **Windows ML** | General availability, binds apps to best available silicon (NPU/GPU/CPU) |
| **Prism Emulation** | Updated to support AVX/AVX2 extensions, enables x86 app execution on ARM |
| **Qualcomm AI Stack** | ONNX RT with Execution Provider → Qualcomm AI Engine Direct SDK → Hexagon NPU |

---

*Sources compiled from: Qualcomm Press Release, Qualcomm Hexagon NPU Official Page, Hot Chips 2023, PCMag, Tom's Guide, Windows Central, Tech Insider Organization*
