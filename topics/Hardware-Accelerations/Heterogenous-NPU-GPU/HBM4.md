# HBM4 (High Bandwidth Memory 4)

> **Note:** The original query specified "HGM4" — after extensive research, **HGM4 could not be confirmed as a real technology or standard**. The most likely intended topic is **HBM4 (High Bandwidth Memory 4)**, which is directly relevant to heterogeneous NPU+GPU compute as the memory backbone for next-generation AI accelerators. This document covers HBM4 per JEDEC JESD270-4 specification.

---

## Table of Contents

1. [Ambiguity Note](#ambiguity-note)
2. [Definitions & Specifications](#2-definitions--specifications)
3. [Key Sources & Quotes](#3-key-sources--quotes)
4. [Benchmarks & Performance Data](#4-benchmarks--performance-data)
5. [Blog Posts & Articles](#5-blog-posts--articles)
6. [Summary Table](#6-summary-table)

---

## 1. Ambiguity Note

**HGM4 was not found as a recognized hardware standard.** After searching across memory technology acronyms, GaN modules, ARM configurations, and Huawei Ascend product lines, no evidence supports "HGM4" as an established technology.

Given the context of **heterogeneous NPU+GPU architectures for AI**, the topic was interpreted as **HBM4 (High Bandwidth Memory 4)** — the JEDEC-standardized memory technology critical to modern AI accelerators where NPUs and GPUs share high-bandwidth memory infrastructure.

---

## 2. Definitions & Specifications

### HBM4 Overview

> *"The HBM4 DRAM is tightly coupled to the host compute die with a distributed interface. The interface is divided into independent channels. Each channel is completely independent of one another. Channels are not necessarily synchronous to each other. The HBM4 DRAM uses a wide-interface architecture to achieve high-speed, low power operation. Each channel interface maintains a 64 bit data bus operating at double data rate (DDR)."*
> — [JEDEC HBM4 DRAM Specification ( JESD270-4 )](https://www.jedec.org/standards-documents/docs/jesd270-4a)

> *"JEDEC has published the official HBM4 (High Bandwidth Memory 4) specification under document JESD270-4. This new memory standard targets the rapidly growing requirements of AI workloads, high-performance computing (HPC), and advanced data center environments."*
> — [Tom's Hardware - JEDEC Finalizes HBM4](https://www.tomshardware.com/pc-components/ram/jedec-finalizes-hbm4-memory-standard-with-major-bandwidth-and-efficiency-upgrades)

### Key Specifications

| Parameter | HBM4 Specification |
|-----------|-------------------|
| **Interface Width** | 2,048-bit (doubled from HBM3) |
| **Data Rate** | Up to 8 GT/s (official), 10+ GT/s achieved |
| **Bandwidth per Stack** | Up to 2 TB/s (JEDEC); 2.8–3.3 TB/s (vendor claims) |
| **Channels per Stack** | 32 independent channels |
| **Stack Configurations** | 4-Hi to 16-Hi |
| **DRAM Die Densities** | 24Gb or 32Gb |
| **Max Capacity** | 64 GB per stack |
| **Voltage (VDDQ)** | 0.7V, 0.75V, 0.8V, or 0.9V (vendor-specific) |
| **Voltage (VDDC)** | 1.0V or 1.05V |
| **Backward Compatibility** | Compatible with HBM3 controllers |

### Samsung HBM4

> *"Samsung HBM4 pushes beyond limits, delivering up to 3,300GB/s of bandwidth — approximately a 2.7× improvement over previous generations — and providing a powerful foundation for the evolution of AI systems."*
> — [Samsung Semiconductor HBM4](https://semiconductor.samsung.com/dram/hbm/hbm4/)

| Specification | Samsung HBM4 |
|--------------|-------------|
| **Bandwidth** | Up to 3,300 GB/s |
| **Data Rate** | Up to 13 Gbps |
| **I/O Pin Count** | 2,048 pins (doubled from 1,024) |
| **Energy Efficiency** | 40% higher |
| **Thermal Improvement** | 10% better vertical thermal resistance, 30% better heat dissipation |
| **Technologies** | 1c DRAM + 4nm foundry (base die) |

### Micron HBM4

> *"HBM4 capacity determines how much of the problem fits in memory, while its bandwidth determines how quickly the system can solve it. HBM4 bandwidth, over 2.8 TB/s, matters most for applications in AI and HPC that need to move terabytes of data at high speeds."*
> — [Micron HBM4](https://www.micron.com/products/memory/hbm/hbm4)

| Specification | Micron HBM4 |
|--------------|-------------|
| **Bandwidth** | >2.8 TB/s per stack |
| **Bus Width** | 2,048 I/O |
| **Speed** | >11.0 Gbps |
| **Power Efficiency** | 20% improvement (picojoules per bit) |

---

## 3. Key Sources & Quotes

### Source 1: JEDEC Official Specification

- **URL:** https://www.jedec.org/standards-documents/docs/jesd270-4a
- **Description:** Official HBM4 DRAM standard definition

### Source 2: Samsung Semiconductor — HBM4

- **URL:** https://semiconductor.samsung.com/dram/hbm/hbm4/
- **Organization:** Samsung Electronics

> *"Samsung HBM4 pushes beyond limits, delivering up to 3,300GB/s of bandwidth — approximately a 2.7× improvement over previous generations."*

### Source 3: Micron Technology — HBM4

- **URL:** https://www.micron.com/products/memory/hbm/hbm4
- **Organization:** Micron Technology

> *"HBM4 bandwidth, over 2.8 TB/s, matters most for applications in AI and HPC that need to move terabytes of data at high speeds."*

### Source 4: Tom's Hardware — JEDEC HBM4 Finalization

- **URL:** https://www.tomshardware.com/pc-components/ram/jedec-finalizes-hbm4-memory-standard-with-major-bandwidth-and-efficiency-upgrades

> *"Each HBM4 memory stack features a 2,048-bit interface that officially supports data transfer rates of up to 8 GT/s, although controllers from controller specialists like Rambus and HBM4 stacks from leading DRAM vendors already support speeds of 10 GT/s or higher."*

### Source 5: Supermicro Glossary — HBM4

- **URL:** https://www.supermicro.com/en/glossary/hbm4

> *"HBM4 is part of the evolving High Bandwidth Memory (HBM) family and is specifically optimized for use in high-performance computing environments such as data centers, artificial intelligence (AI), machine learning, and graphics-intensive applications."*

### Source 6: Wikipedia — High Bandwidth Memory

- **URL:** https://en.wikipedia.org/wiki/High_Bandwidth_Memory

> *"HBM4: April 2025, 8 Gb/s, 32×64 bit, 16 dies × 4GB = 64 GB, 2,048 GB/s"*

---

## 4. Benchmarks & Performance Data

### HBM Generation Timeline

| Generation | Year | Bandwidth/Stack | Key Application |
|------------|------|-----------------|----------------|
| HBM1 | 2015 | 128 GB/s | AMD Fiji GPU |
| HBM2 | 2016 | 256 GB/s | NVIDIA V100, P100 |
| HBM2E | 2020 | 460 GB/s | NVIDIA A100, AMD MI250X |
| HBM3 | 2023 | 819 GB/s | NVIDIA H100 |
| HBM3E | 2024 | 1.15 TB/s | NVIDIA H200, AMD MI300X |
| **HBM4** | **2026** | **2+ TB/s** | Next-gen AI accelerators |

### NVIDIA Vera Rubin vs AMD MI455X (HBM4)

> *"NVIDIA revised its Vera Rubin NVL72 AI server specifications at CES 2026, boosting HBM4 memory bandwidth by 10% to 22.2TB/sec to surpass AMD's upcoming Instinct MI455X AI accelerator."*
> — [TweakTown: NVIDIA HBM4 Bandwidth Upgrade](https://www.tweaktown.com/news/109820/nvidia-upgrades-vera-rubin-hbm4-bandwidth-by-10-percent-in-order-to-stay-ahead-of-amd-instinct-mi455x/index.html)

| Specification | NVIDIA Vera Rubin NVL72 | AMD Instinct MI455X |
|---------------|-------------------------|---------------------|
| **Memory Bandwidth** | 22.2 TB/sec | 19.6 TB/sec |
| **HBM4 Stack Config** | 8-Hi | 12-Hi |

### AI Training Performance Impact

> *"For memory-intensive workloads such as AI training and scientific simulations, HBM4 can deliver roughly 40–70% performance improvement compared with HBM3E."*
> — [AIChipLink: HBM3E vs HBM4](https://aichiplink.com/blog/HBM3E-vs-HBM4-Complete-Comparison-of-Next-Generation-High-Bandwidth-Memory_1053)

### HBM4 vs HBM3E Comparison

| Metric | HBM4 | HBM3E | Improvement |
|--------|-------|-------|-------------|
| **Interface Width** | 2,048-bit | 1,024-bit | 2× |
| **Channels per Stack** | 32 | 16 | 2× |
| **Bandwidth/Stack** | 2.0–3.3 TB/s | 1.15 TB/s | +75–187% |
| **Max Capacity** | 64 GB | 36 GB | +78% |
| **Data Rate/Pin** | 8–13 Gbps | 9.2 Gbps | Up to +41% |
| **Voltage** | 0.75–0.9V | 1.1V | -18–32% |
| **Power Efficiency** | Up to 40% better | Baseline | Significant |

### HBM4E (Enhanced) Specifications

> *"HBM4E vs HBM3E: 2.5× total bandwidth increase (1.2 TB/s → 3 TB/s), 1.7× more power efficient, 1.8× more area efficient."*
> — [Tom's Hardware - HBM Architecture Shakeup](https://www.tomshardware.com/pc-components/dram/hbm-undergoes-major-architectural-shakeup-as-tsmc-and-guc-detail-hbm4-hbm4e-and-c-hbm4e-3nm-base-dies-to-enable-2-5x-performance-boost-with-speeds-of-up-to-12-8gt-s-by-2027)

> *"12.8 GT/s achievable with advanced PHY (Cadence)."*

### C-HBM4E (Custom) Innovations

> *"C-HBM4E: First custom memory type integrated into HBM standard — marks major industry shift. HBM4E memory stack with custom base die while retaining standard JEDEC-compliant clock/electrical specs."*
> — [Tom's Hardware](https://www.tomshardware.com/pc-components/dram/hbm-undergoes-major-architectural-shakeup-as-tsmc-and-guc-detail-hbm4-hbm4e-and-c-hbm4e-3nm-base-dies-to-enable-2-5x-performance-boost-with-speeds-of-up-to-12-8gt-s-by-2027)

### Market Data

> *"SK Hynix leads the HBM market with 62% share in Q2 2025, followed by Micron at 21% and Samsung at 17%. The global HBM market will grow from $38 billion in 2025 to $58 billion in 2026."*
> — [Introl: HBM Evolution](https://introl.com/blog/hbm-evolution-hbm3-hbm3e-hbm4-memory-ai-gpu-2025)

---

## 5. Blog Posts & Articles

| Title | URL | Date |
|-------|-----|------|
| HBM evolution: from HBM3 to HBM4 and the AI memory war | https://introl.com/blog/hbm-evolution-hbm3-hbm3e-hbm4-memory-ai-gpu-2025 | February 11, 2026 |
| What Is HBM4? | https://www.supermicro.com/en/glossary/hbm4 | 2025-2026 |
| The Role of High Bandwidth Memory (HBM) in Advancing AI | https://medium.com/chip-design-insider/the-role-of-high-bandwidth-memory-hbm-in-advancing-ai-caffc63154e5 | 2025 |
| HBM Technology Leads the AI Era: Selection and Procurement Guide | https://trustcompo.com/blog/applications-of-HBM-in-AI | 2025 |
| HBM3E vs HBM4 Comparison | https://mozelectronics.com/tutorials/hbm3e-vs-hbm4-comparison/ | 2025 |
| How HBM4 Improves Efficiency in Mixed AI and HPC Workloads | https://eureka.patsnap.com/report-how-hbm4-improves-efficiency-in-mixed-ai-and-hpc-workloads | 2025 |

---

## 6. Summary Table

### Vendor Bandwidth Claims

| Vendor | HBM4 Bandwidth | Notes |
|--------|---------------|-------|
| **Samsung** | 3,300 GB/s | ~2.7× improvement over previous gen |
| **Micron** | >2.8 TB/s | 12-Hi stack |
| **SK Hynix** | 2.0+ TB/s | 12-Hi stack at 10 GT/s |
| **JEDEC Standard** | 2.0 TB/s | Official specification |

### AI Accelerator HBM4 Integration Timeline

| Company | Product | HBM4 Status | Expected |
|---------|---------|-------------|----------|
| **NVIDIA** | Vera Rubin | First with HBM4 | 2026 |
| **AMD** | Instinct MI455X | 12-Hi HBM4 | 2026 |
| **Intel** | Gaudi 4 (TBD) | Planning | 2026-2027 |
| **Huawei** | Ascend lineup | In-house HBM | 2026 |

### HBM4 At-a-Glance

| Metric | Value |
|--------|-------|
| **JEDEC Document** | JESD270-4 |
| **Finalization** | April 2025 |
| **Interface Width** | 2,048-bit |
| **Channels/Stack** | 32 |
| **Max Bandwidth/Stack** | 2.0–3.3 TB/s |
| **Max Capacity/Stack** | 64 GB |
| **Data Rate** | 8–13 Gbps |
| **Stack Configs** | 4-Hi to 16-Hi |
| **Voltage** | 0.75–0.9V VDDQ |
| **Key Improvement** | 40–70% AI training perf vs HBM3E |

---

*Note: "HGM4" was not a recognized standard — interpreted as HBM4. If a different technology was intended, please clarify.*

*Sources compiled from: JEDEC, Samsung Semiconductor, Micron Technology, Tom's Hardware, TweakTown, Supermicro, Wikipedia, AIChipLink, Introl Blog, Medium, Moz Electronics, TrustCompo, PatSnap*
