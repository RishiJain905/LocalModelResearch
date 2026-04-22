# Intel Panther Lake (Core Ultra Series 3)

> **Intel Panther Lake** is the codename for Intel's Core Ultra Series 3 mobile processors — the first consumer chip built on Intel's 18A process node with RibbonFET (GAA) and PowerVia (backside power delivery) technologies. It integrates up to 16 CPU cores, Xe3 GPU architecture, and a 5th-generation NPU delivering up to **180 platform TOPS**.

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

### What is Intel Panther Lake?

> *"Panther Lake is the codename for Core Ultra Series 3 mobile processors developed by Intel. The architecture was launched in January 2026 at CES 2026."*
> — [Wikipedia: Panther Lake (Microprocessor)](https://en.wikipedia.org/wiki/Panther_Lake_(microprocessor))

> *"Representing the most significant architectural leap for the company in a decade, Panther Lake is the first consumer lineup built on the highly anticipated Intel 18A process node."*
> — [FinancialContent - Intel Launches Core Ultra Series 3 at CES 2026](https://markets.financialcontent.com/stocks/article/tokenring-2026-1-30-intel-launches-core-ultra-series-3-panther-lake-at-ces-2026-the-18a-era-begins)

### Intel 18A Node — Two "Holy Grail" Technologies

**RibbonFET** — Intel's Gate-All-Around (GAA) transistor architecture replacing FinFET:

> *"RibbonFET is Intel's implementation of a Gate-All-Around (GAA) field-effect transistor, which uses ribbon-shaped channels surrounded by the gate."*
> — [XDA Developers - Intel Panther Lake CPU: Architecture, specs and a 2026 launch](https://www.xda-developers.com/intel-panther-lake-details/)

**PowerVia** — Backside power delivery technology.

### Xe3 "Celestial" GPU Architecture

> *"Xe3 is a continuous improvement of the Battlemage architectural lineage, not the previously expected Celestial architecture. Despite mapping to that codename, Intel classifies Xe3 GPUs under the Arc B-series umbrella because the capabilities presented to software are similar to Xe2 products."*
> — [Tom's Hardware - Intel's Xe3 graphics architecture breaks cover](https://www.tomshardware.com/pc-components/gpus/intels-xe3-graphics-architecture-breaks-cover-panther-lakes-12-xe-core-igpu-promises-50-percent-better-performance-than-lunar-lake)

### NPU 5 (5th-generation Neural Processing Unit)

> *"Delivering up to 180 platform TOPS, Panther Lake processors offer a substantial increase over Lunar Lake's 120 TOPS."*
> — [ASUS USA Blog - Intel Panther Lake vs Lunar Lake](https://www.asus.com/us/blog/intel-panther-lake-vs-lunar-lake-how-intel-core-ultra-series-3-processor-redefines-the-ai-pc-era/)

### Tile-Based Architecture

> *"The processor contains four tiles with a rectangular chip shape created using structural silicon 'Filler tiles.'"*
> — [TechPowerUp - Intel Core Ultra Series 3 "Panther Lake-H" Die Annotated](https://www.techpowerup.com/347141/intel-core-ultra-series-3-panther-lake-h-die-annotated)

| Tile | Foundry Node | Area |
|------|-------------|------|
| **Compute** | Intel 18A | 115 mm² |
| **Graphics** | TSMC N3E (12-core variant) | 55.18 mm² |
| **I/O** | TSMC N6 | 49.76 mm² |
| **Base (Interposer)** | Intel 22nm | — |

### Complete Specifications

| Specification | Panther Lake-H (Flagship) | Panther Lake-U |
|--------------|---------------------------|----------------|
| **Process Node** | Intel 18A (compute tile) | Intel 18A |
| **CPU Cores** | Up to 16 (4P+8E+4LP-E) | Up to 8 (4P+4LP-E) |
| **CPU Architecture** | Cougar Cove (P-cores) + Darkmont (E-cores) | Cougar Cove + Darkmont LP |
| **GPU Architecture** | Xe3 "Celestial" (Arc B390) | Xe3 (Intel Graphics) |
| **GPU Cores** | Up to 12 Xe cores | Up to 4 Xe cores |
| **NPU** | NPU 5 (50 TOPS INT8) | NPU 5 |
| **Total AI TOPS** | Up to 180 TOPS | ~72-77 TOPS |
| **Memory Support** | LPDDR5X-9600 MT/s | LPDDR5X-8533, DDR5-6400 |
| **GPU L2 Cache** | Up to 16 MB | Up to 4 MB |
| **PCIe Lanes** | 12 or 20 (config-dependent) | 12 |
| **Base TDP** | 25W | 25W |
| **Max Turbo TDP** | 80W | 55W |
| **Ray Tracing Units** | Up to 12 | 4 |
| **Manufacturing (GPU tile)** | TSMC N3E (12-core variant) | Intel 3 (4-core variant) |

> *"Intel is fully realizing the benefits of tile-based architecture here, so it can build you a CPU with as few as six CPU cores or as many as sixteen, just four Xe3-cores or up to twelve."*
> — [HotHardware - Intel Panther Lake Performance Unveiled](https://hothardware.com/news/intel-ces-2026-panther-lake-is-a-go)

---

## 2. Key Sources & Quotes

### Source 1: Intel Newsroom — "Intel Unveils Panther Lake Architecture"

- **URL:** https://newsroom.intel.com/client-computing/intel-unveils-panther-lake-architecture-first-ai-pc-platform-built-on-18a
- **Date:** October 9, 2025
- **Author:** Intel Corporation

> *"We are entering an exciting new era of computing, made possible by great leaps forward in semiconductor technology that will shape the future for decades to come."*

> *"Intel 18A is the most advanced semiconductor node developed and manufactured in the United States."*

> *"Delivering up to 180 Platform TOPS" with "50%+ faster CPU performance" and "50%+ faster graphics performance vs previous generation."*

### Source 2: Intel PDF — ITT 2025 Panther Lake Recap

- **URL:** https://cdrdv2-public.intel.com/866361/ITT_2025_Panther_Lake_Recap1.pdf
- **Author:** Intel Technology Tour 2025

> *">40% more TOPS/area"*
> *">50% more performance per watt"*
> *"1.5W reduction in power with HW staggered HDR"*
> *"Next-gen NPU 5" and "New Xe3 GPU"*

### Source 3: HotHardware — CES 2026 Launch Coverage

- **URL:** https://hothardware.com/news/intel-ces-2026-panther-lake-is-a-go
- **Date:** January 5, 2026
- **Author:** Zak Killian

> *"24% improved multi-threaded performance versus its own Arrow Lake processors (despite having fewer P-cores)"*

> *"50% better power efficiency than AMD's Ryzen AI 300 series"*

> *"73% better gaming performance against the competition"*

> *"4.3x faster LLM inference than AMD XDNA2 (Ryzen AI 9 HX 370)"*

**AI TOPS Breakdown:** *"GPU: 120 TOPS, NPU: 50 TOPS, CPU: 10 TOPS"* (Total: 180)

### Source 4: Wccftech — Intel Panther Lake 'Core Ultra 300' Roundup

- **URL:** https://wccftech.com/roundup/intel-panther-lake/
- **Date:** November 20, 2025

> *"If Intel manages to nail Panther Lake and then Clearwater Forest, 18A will see a great reception from the industry."*

### Source 5: Wikipedia — Panther Lake (Microprocessor)

- **URL:** https://en.wikipedia.org/wiki/Panther_Lake_(microprocessor)

> *"Widely praised for power-efficiency and integrated graphics performance, noted as a 'return to form' for Intel."*

---

## 3. Benchmarks & Performance Data

### CPU Performance

| Benchmark | Intel Core Ultra X9 388H | AMD Ryzen AI 9 HX 370 | Winner |
|-----------|--------------------------|-----------------------|--------|
| **Geekbench 6 Single-Core** | ~3,031 | ~2,932 | **Intel (+10.4%)** |
| **Geekbench 6 Multi-Core** | ~17,283 | ~18,407 | AMD |
| **Cinebench 2024 Single** | ~138 | ~127 | **Intel (+8.7%)** |
| **Cinebench 2024 Multi** | ~920 | ~960 | AMD |

> *"Intel dominates single-threaded performance; AMD leads in heavily multi-threaded workloads. Gap is narrow in both directions."*
> — [Tech Insider - Intel Panther Lake vs AMD Ryzen AI 400](https://tech-insider.org/intel-panther-lake-vs-amd-ryzen-ai-400-2026/)

### GPU / Gaming Performance

| Game | Settings | Intel Arc B390 FPS | AMD Radeon 890M FPS |
|------|----------|--------------------|--------------------|
| Cyberpunk 2077 | 1080p RT Medium, XeSS Balanced, FG 2x | **73.15** | ~40 |
| Forza Horizon 5 | 1080p High, XeSS Quality | **111** | ~70 |
| F1 25 | 1080p High, XeSS Balanced | **104** | ~65 |
| Doom The Dark Ages | 1080p High, XeSS Balanced, FG 2x | **97.8** | ~55 |

> *"Comparing the Core Ultra X9 388H against the Core Ultra 9 285H, the new part comes out 76% faster on average."*
> — [HotHardware](https://hothardware.com/news/intel-ces-2026-panther-lake-is-a-go)

> *"Intel Core Ultra X9 388H beats AMD Ryzen AI 9 HX 370 by 82% in average gaming performance."*
> — [HotHardware](https://hothardware.com/news/intel-ces-2026-panther-lake-is-a-go)

### AI / NPU Performance

| Platform | CPU TOPS | GPU TOPS | NPU TOPS | **Total TOPS** |
|----------|----------|----------|----------|----------------|
| **Intel Core Ultra X9 388H** | ~12 | ~120 | 50 | **~172-180** |
| AMD Ryzen AI 9 HX 370 | ~15 | ~15 | 50 | ~80 |
| Qualcomm Snapdragon X2 Elite | — | — | 80 | 80+ |

**Local LLM Inference (llama.cpp, 7B Q4_K_M model):**

| Platform | Prompt Eval | Generation |
|----------|-------------|------------|
| **Intel Core Ultra X9 388H** (Arc B390) | ~42 tokens/sec | ~28 tokens/sec |
| AMD Ryzen AI 9 HX 370 | ~31 tokens/sec | ~19 tokens/sec |
| **Intel Advantage** | **+35% faster** | **+47% faster** |

> *"Intel advantage: ~35-47% faster local LLM inference with GPU offload."*
> — [Tech Insider](https://tech-insider.org/intel-panther-lake-vs-amd-ryzen-ai-400-2026/)

**Geekbench AI Scores (Cross-Platform Comparison):**

| Chip | AI Score |
|------|----------|
| Snapdragon X2 Elite Extreme | **88,615** |
| M5 Pro | 57,420 |
| Intel Core Ultra X9 388H | 56,829 |
| AMD Ryzen AI Max+ 395 | 17,854 |

> *"Qualcomm dominates AI with 88K+ scores—55% ahead of Apple's M5 (~57K)."*
> — [Tom's Guide - Apple M5 vs Intel Panther Lake vs Snapdragon X2](https://www.tomsguide.com/computing/apple-m5-vs-intel-vs-amd-vs-snapdragon-x2-which-chip-wins)

### Power Efficiency

> *"Intel's claim of 27 hours of continuous video playback positions Panther Lake as the new 'Battery Life King.'"*
> — [FinancialContent](https://markets.financialcontent.com/stocks/article/tokenring-2026-1-30-intel-launches-core-ultra-series-3-panther-lake-at-ces-2026-the-18a-era-begins)

| Comparison | Power Reduction |
|------------|-----------------|
| vs Raptor Lake (video streaming) | **53% reduction** |
| vs Lunar Lake (average) | ~10-15% improvement |
| vs AMD Ryzen AI 7 365 | **78% reduction** |

### Xe3 Architecture Performance Gains

| Metric | Improvement |
|--------|-------------|
| vs Lunar Lake Xe2 (same power) | **>50% higher performance** |
| vs Arrow Lake-H (per watt) | **>40% better** |
| Depth writes (microbenchmark) | **7.4x faster** |
| High-register-pressure shaders | **3.1x faster** |
| Frame render time | **50% less time** (22.84ms vs Xe2) |

> *"The 12 Xe3 core PTL chip is going to be much nicer to game on than any LNL processor, and since they're pretty decent to begin with, I can't wait to get my hands on a Panther Lake laptop or handheld."*
> — [PC Gamer - Intel Panther Lake Xe3 graphics deep dive](https://www.pcgamer.com/hardware/graphics-cards/its-not-celestial-but-intels-new-xe3-gpu-architecture-looks-like-itll-be-ideal-for-the-next-generation-of-handheld-gaming-pcs/)

---

## 4. Blog Posts & Articles

| Title | URL | Author | Date |
|-------|-----|--------|------|
| Intel Panther Lake Performance Unveiled: Core Ultra Series 3 Roars To Life At CES | https://hothardware.com/news/intel-ces-2026-panther-lake-is-a-go | Zak Killian | January 5, 2026 |
| Intel Panther Lake Core Ultra review: Intel's best laptop CPU in a very long time | https://arstechnica.com/gadgets/2026/02/intel-panther-lake-core-ultra-review-intels-best-laptop-cpu-in-a-very-long-time/ | Andrew Cunningham | February 2026 |
| Apple M5 vs. Intel Panther Lake vs. Snapdragon X2 — Benchmark Showdown | https://www.tomsguide.com/computing/apple-m5-vs-intel-vs-amd-vs-snapdragon-x2-which-chip-wins | Tom's Guide | 2026 |
| Intel Panther Lake Xe3 graphics deep dive | https://www.pcgamer.com/hardware/graphics-cards/its-not-celestial-but-intels-new-xe3-gpu-architecture-looks-like-itll-be-ideal-for-the-next-generation-of-handheld-gaming-pcs/ | PC Gamer | October 2025 |
| Intel Panther Lake vs Lunar Lake: Copilot+ PC Leap | https://www.asus.com/us/blog/intel-panther-lake-vs-lunar-lake-how-intel-core-ultra-series-3-processor-redefines-the-ai-pc-era/ | ASUS USA | 2026 |
| Intel Rendering Toolkit & OpenVINO AI GPU Performance On Intel Panther Lake's Xe3 B390 | https://www.phoronix.com/review/intel-b390-openvino-render | Phoronix | 2026 |
| Intel Panther Lake vs AMD Ryzen AI 400: AI Laptop CPU Comparison 2026 | https://tech-insider.org/intel-panther-lake-vs-amd-ryzen-ai-400-2026/ | Tech Insider | 2026 |

**Ars Technica Key Quotes:**

> *"In Asus' Performance Mode, Panther Lake is around twice as fast as the Lunar Lake-based Core Ultra 7 258V and 80–90 percent faster than older 12th- and 13th-generation Core processors."*

> *"Panther Lake and the Core Ultra 3-series are Intel's most impressive laptop chips in at least half a decade."*

---

## 5. Comparison with Alternatives

### vs AMD Strix Point / Ryzen AI 300/400

| Feature | Intel Panther Lake | AMD Ryzen AI (Strix Point/Halo) |
|---------|-------------------|--------------------------------|
| **Process Node** | Intel 18A | TSMC 4nm FinFET |
| **CPU Cores** | Up to 16 (4P+8E+4LP-E) | Up to 12 (4 Zen 5 + 8 Zen 5c) |
| **Total AI TOPS** | ~172-180 TOPS | ~80 TOPS |
| **GPU Architecture** | Xe3 Arc B390 (12 cores) | RDNA 3.5 (16 CUs max) |
| **Gaming Performance** | **Win** (82% faster than Ryzen AI 9 HX 370) | Lower |
| **LLM Inference** | **Win** (+35-47% faster) | Lower |
| **Battery Life** | Similar | Similar |
| **vPro Support** | Yes | Limited |

### vs Apple M5 Series

| Feature | Intel Panther Lake | Apple M5 |
|---------|-------------------|----------|
| **Geekbench Single-Core** | ~3,031 | **~4,288-4,338** |
| **Geekbench Multi-Core** | ~17,283 | **~17,926-29,430** |
| **GPU (3DMark Solar Bay)** | 101-116 FPS | **178-268 FPS** |
| **Battery Life** | 14-20 hours | **17-21 hours** |
| **AI Performance (Geekbench)** | 55,000-57,000 | 57,000 |

> *"Apple's M5 chips help MacBooks deliver superior CPU and GPU performance, with epic battery life to boot."*
> — [Tom's Guide](https://www.tomsguide.com/computing/apple-m5-vs-intel-vs-amd-vs-snapdragon-x2-which-chip-wins)

### vs Qualcomm Snapdragon X2 Elite

| Feature | Intel Panther Lake | Qualcomm Snapdragon X2 Elite |
|---------|-------------------|------------------------------|
| **Architecture** | x86-64 | ARM |
| **AI TOPS** | 172-180 | 80+ |
| **NPU Performance** | 50 TOPS | 80 TOPS |
| **Geekbench AI** | ~56,000 | **~88,000** |
| **Ray Tracing (Solar Bay)** | 101-116 FPS | 88 FPS |
| **Gaming (Arc B390)** | **10-20% faster** | Lower |

> *"Qualcomm has officially moved past the 'alternative' phase and into a 'serious challenger' position."*
> — [PCMag - Qualcomm Snapdragon X2 Elite Extreme Review](https://me.pcmag.com/en/laptops/36468/move-over-m5-qualcomms-snapdragon-x2-elite-extreme-chip-crushes-apple-in-our-benchmark-testing)

### Heterogeneous Compute Positioning

> *"Instead of pushing AI tasks onto the CPU or GPU directly, it will select which ones are more suitable for the NPU and then let the NPU handle them efficiently and continuously."*
> — [ASUS USA Blog](https://www.asus.com/us/blog/intel-panther-lake-vs-lunar-lake-how-intel-core-ultra-series-3-processor-redefines-the-ai-pc-era/)

Intel positions Panther Lake as a **full heterogeneous AI compute platform**:

| Compute Unit | Role | TOPS |
|-------------|------|------|
| **CPU** | General compute, lightweight AI | 10 |
| **NPU 5** | Sustained, low-power AI workloads | 50 |
| **Xe3 GPU** | Parallel AI compute, graphics, gaming | 120 |

---

## 6. Summary Table

### Intel Panther Lake At-a-Glance

| Category | Data Point |
|----------|-----------|
| **Codename** | Panther Lake |
| **Brand** | Core Ultra Series 3 |
| **Launch Date** | January 5, 2026 (CES 2026) |
| **Process Node** | Intel 18A (first consumer chip on 18A) |
| **Transistor Tech** | RibbonFET (GAAFET) + PowerVia (backside power delivery) |
| **CPU Cores** | Up to 16 (4 Cougar Cove P-cores + 8 Darkmont E-cores + 4 LP E-cores) |
| **GPU Architecture** | Xe3 "Celestial" (Battlemage lineage) |
| **GPU Cores** | Up to 12 Xe cores |
| **GPU L2 Cache** | Up to 16 MB |
| **NPU Generation** | 5th-generation (NPU 5) |
| **NPU TOPS** | 50 TOPS INT8 |
| **GPU TOPS** | Up to 120 TOPS |
| **Total Platform TOPS** | Up to 180 TOPS |
| **Memory** | LPDDR5X-9600 MT/s (max), DDR5-7200 |
| **Base TDP** | 25W |
| **Max Turbo TDP** | 80W |
| **PCIe** | PCIe 5.0 (12 or 20 lanes) |
| **Wi-Fi** | Wi-Fi 7 (R2) |
| **Thunderbolt** | Thunderbolt 4 |
| **Manufacturing (GPU tile)** | TSMC N3E (12-core); Intel 3 (4-core) |

### Key Performance Claims (Intel Official)

| Metric | Improvement vs Prior Generation |
|--------|--------------------------------|
| Multi-threaded performance | +24% vs Arrow Lake |
| Power efficiency | +50% vs AMD Ryzen AI 300 |
| Gaming performance | +73% vs competition |
| GPU performance | +50%+ vs Lunar Lake Xe2 |
| GPU perf/watt | +40% vs Arrow Lake-H |
| NPU performance | 4.3x faster LLM inference vs AMD XDNA2 |

### AI Ecosystem Software Stack

| Component | Description |
|----------|-------------|
| **OpenVINO** | Intel's inference optimization toolkit for CPU/GPU/NPU deployment |
| **oneAPI** | Cross-industry API for heterogeneous computing |
| **Intel Compute Runtime** | OpenCL/SYCL support for Linux |
| **XeSS 3** | AI super-resolution with Multi-Frame Generation |
| **Copilot+ PC** | Meets Microsoft's AI certification requirements |

> *"Complementing the cross-vendor OpenCL benchmarks... benchmarks looking at how Intel graphics performance have evolved from Meteor Lake to Lunar Lake and now Panther Lake when looking at Intel's optimized software for their GPU compute environment using software from the Intel Rendering Toolkit (formerly known as the oneAPI Rendering Toolkit)."*
> — [Phoronix](https://www.phoronix.com/review/intel-b390-openvino-render)

### Market Availability

| Item | Date |
|------|------|
| **Pre-orders** | January 6, 2026 |
| **Shipping** | January 2026 |
| **System Designs** | 200+ designs planned |
| **Key OEMs** | ASUS Zenbook, MSI Prestige/Titan/Stealth, Samsung Galaxy Book6, Lenovo Yoga Slim 7i |

---

*Sources compiled from: Intel Newsroom, HotHardware, Tom's Hardware, Ars Technica, PC Gamer, ASUS USA, Phoronix, Tech Insider, Wikipedia, XDA Developers, Wccftech, FinancialContent*
