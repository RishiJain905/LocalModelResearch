# Hardware-Accelerations

> *Hardware acceleration refers to the use of specialized hardware (GPUs, TPUs, NPUs, FPGAs, ASICs) to speed up AI/ML inference and training workloads beyond what general-purpose CPUs can achieve.*

## Overview

This folder covers hardware acceleration technologies for AI/ML workloads — including GPU inference optimization, TPU systems, NPU implementations, FPGA-based acceleration, and emerging custom silicon for LLMs.

---

## Subtopics

| Folder | Description |
|--------|-------------|
| [Nvidia-Blackwell-FP4/](./Nvidia-Blackwell-FP4/) | NVIDIA Blackwell architecture with native FP4 Tensor Cores, HBM3e memory |
| [Apple-M5-Fusion/](./Apple-M5-Fusion/) | Apple M5 Neural Accelerators, Fusion Architecture, MLX framework |
| [Heterogenous-NPU-GPU/](./Heterogenous-NPU-GPU/) | Intel Panther Lake, Snapdragon X2, OpenVINO — heterogeneous NPU+GPU compute |

---

## Nvidia-Blackwell-FP4 Subtopics

| Document | Description |
|----------|-------------|
| [FP4-Tensor-Cores](./Nvidia-Blackwell-FP4/FP4-Tensor-Cores.md) | NVFP4 format, 18,000 FP4 TOPS, near-lossless quantization |
| [CG-HBM](./Nvidia-Blackwell-FP4/CG-HBM.md) | 8 TB/s HBM3e memory, CoWoS-L packaging |
| [DGX-Spark](./Nvidia-Blackwell-FP4/DGX-Spark.md) | Personal AI supercomputer, $4,699, GB10 chip |

---

## Heterogenous-NPU-GPU Subtopics

| Document | Description |
|----------|-------------|
| [Intel-Panther-Lake](./Heterogenous-NPU-GPU/Intel-Panther-Lake.md) | Core Ultra Series 3, Intel 18A, NPU 5, Xe3 GPU, 180 TOPS |
| [Snapdragon-X2](./Heterogenous-NPU-GPU/Snapdragon-X2.md) | Hexagon NPU 6, 80 TOPS, Oryon CPU, Adreno X2 |
| [OpenVINO](./Heterogenous-NPU-GPU/OpenVINO.md) | Heterogeneous inference framework, HETERO plugin |
| [HBM4](./Heterogenous-NPU-GPU/HBM4.md) | JEDEC JESD270-4, 2TB/s+ bandwidth, 2048-bit interface |
| [CXL-3.0-Memory-Pooling](./Heterogenous-NPU-GPU/CXL-3.0-Memory-Pooling.md) | PCIe 6.0, 64GT/s, memory pooling, KV cache acceleration |
| [LPDDR5X](./Heterogenous-NPU-GPU/LPDDR5X.md) | JESD209-5B/C, 10.7Gbps, 25% power efficiency gain |

---

## Apple-M5-Fusion Subtopics

| Document | Description |
|----------|-------------|
| [M5-Neural-Accelerators](./Apple-M5-Fusion/M5-Neural-Accelerators.md) | Neural Accelerator architecture, >4x AI compute, Fusion dies |
| [MLX-Next](./Apple-M5-Fusion/MLX-Next.md) | Apple MLX framework, 4,545+ HF models, Ollama integration |
