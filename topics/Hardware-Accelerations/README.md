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

---

## Nvidia-Blackwell-FP4 Subtopics

| Document | Description |
|----------|-------------|
| [FP4-Tensor-Cores](./Nvidia-Blackwell-FP4/FP4-Tensor-Cores.md) | NVFP4 format, 18,000 FP4 TOPS, near-lossless quantization |
| [CG-HBM](./Nvidia-Blackwell-FP4/CG-HBM.md) | 8 TB/s HBM3e memory, CoWoS-L packaging |
| [DGX-Spark](./Nvidia-Blackwell-FP4/DGX-Spark.md) | Personal AI supercomputer, $4,699, GB10 chip |

---

## Apple-M5-Fusion Subtopics

| Document | Description |
|----------|-------------|
| [M5-Neural-Accelerators](./Apple-M5-Fusion/M5-Neural-Accelerators.md) | Neural Accelerator architecture, >4x AI compute, Fusion dies |
| [MLX-Next](./Apple-M5-Fusion/MLX-Next.md) | Apple MLX framework, 4,545+ HF models, Ollama integration |
