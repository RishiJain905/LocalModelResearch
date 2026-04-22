# Vendor Software Stacks

> *AI/ML software ecosystems from major hardware vendors — covering the full stack from low-level kernels to high-level model deployment frameworks.*

---

## Table of Contents

1. [Definitions & Overview](#1-definitions--overview)
2. [NVIDIA Stack](#2-nvidia-stack)
3. [AMD Stack](#3-amd-stack)
4. [Intel Stack](#4-intel-stack)
5. [Qualcomm Stack](#5-qualcomm-stack)
6. [Apple Stack](#6-apple-stack)
7. [Google Stack](#7-google-stack)
8. [Key Sources & Quotes](#8-key-sources--quotes)
9. [Comparison Table](#9-comparison-table)
10. [Blog Posts & Tutorials](#10-blog-posts--tutorials)

---

## 1. Definitions & Overview

| Vendor | Stack Name | Primary Hardware | Model Formats | Open Source |
|--------|-----------|-----------------|---------------|-------------|
| **NVIDIA** | CUDA, cuDNN, TensorRT, Triton, DeepStream, DOCA | A100, H100, B200, Jetson | ONNX, TensorRT, PyTorch, TensorFlow | Partial |
| **AMD** | ROCm, MIGraphX, MIVisionX | MI300X, MI350, Radeon RX 7900 | ONNX, NNEF | Yes |
| **Intel** | oneAPI, OpenVINO | Xeon, Arc, Gaudi | ONNX, IR, TensorFlow | Yes |
| **Qualcomm** | AI Hub, QNN, Snapdragon SDK | Snapdragon X Elite, Hexagon DSP | ONNX | Partial |
| **Apple** | CoreML, MLX, MPS, Accelerate | Apple Silicon (M1–M5) | CoreML, ONNX (via conversion) | Partial |
| **Google** | JAX, XLA, TPU Runtime | Cloud TPU, Ironwood | JAX, TensorFlow | Yes |

---

## 2. NVIDIA Stack

### CUDA (Compute Unified Device Architecture)

> *"NVIDIA® CUDA® Deep Neural Network library (cuDNN) is a GPU-accelerated library of primitives for deep neural networks."*
> — [NVIDIA cuDNN](https://developer.nvidia.com/cudnn)

> *"The CUDA Toolkit from NVIDIA provides a development environment for creating high performance GPU-accelerated applications."*
> — [NVIDIA CUDA](https://developer.nvidia.com/cuda-toolkit)

**Core Components:**

| Component | Description |
|-----------|-------------|
| **CUDA** | Low-level parallel computing platform and programming model |
| **cuBLAS** | GPU-accelerated BLAS library for GEMM operations |
| **cuDNN** | Primitives for conv, pooling, normalization, attention |
| **TensorRT** | High-performance inference compiler and runtime |
| **Triton** | Open inference server for model deployment at scale |
| **DeepStream** | Streaming analytics toolkit for video/AI applications |
| **DOCA** | SDK for BlueField DPUs — "CUDA for the data center" |

### TensorRT

> *"NVIDIA® TensorRT™ is an ecosystem of APIs for high-performance deep learning inference. The TensorRT inference library provides a general-purpose AI compiler and an inference runtime that deliver low latency and high throughput for production applications."*
> — [NVIDIA TensorRT Getting Started](https://developer.nvidia.com/tensorrt-getting-started)

**TensorRT-LLM** builds on top of TensorRT with LLM-specific optimizations:
- In-flight batching
- Custom attention kernels
- FP8 quantization support

### Triton Inference Server

> *"NVIDIA Triton Inference Server is built to simplify the deployment of a model or a collection of models at scale in a production environment."*
> — [NVIDIA Blog — Optimizing and Serving with TensorRT and Triton](https://developer.nvidia.com/blog/optimizing-and-serving-models-with-nvidia-tensorrt-and-nvidia-triton/)

- Supports TensorFlow, PyTorch, ONNX, TensorRT models
- Enables model repository-based deployment
- Dynamic batching and concurrent model execution

### DeepStream

> *"DeepStream is a streaming analytic toolkit to build AI-powered applications that takes streaming data as input (USB/CSI camera, video files, RTSP streams) and uses AI and computer vision to generate insights from pixels."*
> — [NVIDIA DeepStream Overview](https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Overview.html)

- Built on GStreamer framework with 20+ hardware-accelerated plugins
- Used for video analytics, smart cities, retail analytics

### DOCA

> *"DOCA is to DPUs what CUDA is to GPUs."*
> — [NVIDIA DOCA Blog](https://developer.nvidia.com/blog/programming-the-entire-data-center-infrastructure-with-the-nvidia-doca-sdk/)

- Enables rapid creation of applications and services on BlueField DPUs
- Offloads, accelerates, and isolates data center workloads

---

## 3. AMD Stack

### ROCm (Radeon Open Compute)

> *"AMD ROCm™ is an open software stack offering a suite of optimizations for AI workloads and supporting the broader AI software ecosystem including open frameworks, models, and tools."*
> — [AMD ROCm](https://www.amd.com/en/products/software/rocm/ai.html)

**ROCm supports:** MI300X, MI350 Series, Radeon GPUs

### MIGraphX (ROCm Inference)

> *"MIGraphX is a graph compiler and inference engine for high performance machine learning model inference. It compiles trained models from end-to-end to optimize for inference performance on AMD hardware."*
> — [AMD MIGraphX](https://rocm.docs.amd.com/projects/AMDMIGraphX/en/latest/index.html)

### MIVisionX

> *"MIVisionX is a comprehensive machine intelligence and computer vision toolkit. MIVisionX includes AMD's implementation of Khronos OpenVX, as well as a Convolutional Neural Net model compiler and optimizer supporting the ONNX and Khronos NNEF exchange formats."*
> — [AMD MIVisionX](https://rocm.docs.amd.com/projects/MIVisionX/en/latest/)

---

## 4. Intel Stack

### oneAPI

> *"oneAPI includes a cross-architecture language: Data Parallel C++ (DPC++). DPC++ is an evolution of C++ that incorporates the SYCL language with extensions for Unified Shared Memory (USM), ordered queues and reductions."*
> — [oneAPI](https://oneapi.io/c-for-heterogeneous-programming-oneapi-dpc-and-onetbb/)

- Cross-platform heterogeneous parallel programming across CPUs, GPUs, FPGAs
- Open specification with multiple implementations

### OpenVINO

> *"OpenVINO™ toolkit for Linux* and Windows that is a comprehensive toolkit allowing to develop and deploy deep learning based solutions on Intel® platforms for native applications."*
> — [Intel](https://www.intel.com/content/www/us/en/developer/articles/technical/machine-learning-hw-accelerators-web-platform.html)

- Originally developed as a C++ AI inference solution for edge deployment
- Integrated with HuggingFace via Optimum-Intel
- Supports CPU, GPU, VPU, GNA, NPU

---

## 5. Qualcomm Stack

### Qualcomm AI Hub

> *"Qualcomm AI Hub, built on top of the Qualcomm AI Stack, is a developer-centric platform designed to simplify and accelerate the on-device AI applications."*
> — [Qualcomm AI](https://www.qualcomm.com/developer/artificial-intelligence)

- 175+ pre-optimized models for Snapdragon NPUs
- Enables model optimization, compilation, and deployment to edge devices

### QNN (Qualcomm Neural Network)

> *"Qualcomm AI Engine Direct API and the associated software stack provides all the constructs required by an application to construct, optimize, and execute neural networks on Qualcomm hardware."*
> — [QNN Overview](https://docs.qualcomm.com/nav/home/QNN_general_overview.html)

### Snapdragon SDK

> *"The Qualcomm Neural Processing SDK is designed to help developers run one or more neural network models trained in TensorFlow, PyTorch, Keras and ONNX on Snapdragon devices."*
> — [Qualcomm Neural Processing SDK](https://www.qualcomm.com/developer/software/neural-processing-sdk-for-ai)

---

## 6. Apple Stack

### CoreML

> *"Core ML provides a unified representation for all models. Your app uses Core ML APIs and user data to make predictions, and to train or fine-tune models, all on the user's device."*
> — [Apple CoreML](https://developer.apple.com/documentation/coreml)

- Automatic hardware selection (CPU, GPU, Neural Engine)
- Privacy-focused on-device ML

### MLX

> *"MLX is an array framework for machine learning on Apple silicon, brought to you by Apple machine learning research."*
> — [Apple MLX GitHub](https://github.com/ml-explore/mlx)

**Key features:**

| Feature | Description |
|---------|-------------|
| **NumPy-like API** | Familiar programming interface |
| **Lazy computation** | Operations are not executed until converted to arrays |
| **Unified memory model** | Arrays share memory across devices |
| **Automatic differentiation** | Built-in for model training |

> *"MLX offers 3x faster inference than PyTorch MPS for LLMs on Apple Silicon."*
> — [Medium — MPS or MLX](https://medium.com/@koypish/mps-or-mlx-for-domestic-ai-the-answer-will-surprise-you-df4b111de8a0)

### Metal Performance Shaders (MPS)

> *"The Metal Performance Shaders framework supports the following functionality: Apply high-performance filters to, and extract statistical and histogram data from images and compute workloads."*
> — [Apple MPS](https://developer.apple.com/documentation/metalperformanceshaders)

- GPU-accelerated primitives for image processing, linear algebra, ML

### Accelerate Framework

- Optimized BLAS and LAPACK routines for CPU-based numerical computing
- vDSP for signal processing, BNNS for neural network subroutines

---

## 7. Google Stack

### JAX

> *"JAX is a Python library for accelerator-oriented array computation and program transformation designed for high-performance numerical computing and large-scale ML."*
> — [Google Cloud TPU — JAX AI Stack](https://docs.cloud.google.com/tpu/docs/jax-ai-stack)

**Features:** JIT compilation, automatic differentiation, vectorization, parallelization

### XLA (Accelerated Linear Algebra)

> *"XLA (Accelerated Linear Algebra) is a domain-specific, hardware-agnostic compiler. Uses whole-program analysis to fuse operators and optimize memory layouts."*
> — [Google Developers Blog](https://developers.googleblog.com/building-production-ai-on-google-cloud-tpus-with-jax/)

### TPU Runtime

- Systolic array architecture optimized for tensor operations
- Major adopters: Anthropic, xAI, Midjourney, Meta

> *"JAX has become a key framework for developing state-of-the-art foundation models across the AI landscape, and not just at Google."*
> — [Google Developers Blog](https://developers.googleblog.com/building-production-ai-on-google-cloud-tpus-with-jax/)

---

## 8. Key Sources & Quotes

| Vendor | Component | URL | Key Quote |
|--------|-----------|-----|-----------|
| NVIDIA | cuDNN | https://developer.nvidia.com/cudnn | *"GPU-accelerated library of primitives for deep neural networks"* |
| NVIDIA | TensorRT | https://developer.nvidia.com/tensorrt-getting-started | *"Ecosystem of APIs for high-performance deep learning inference"* |
| NVIDIA | Triton | https://developer.nvidia.com/blog/optimizing-and-serving-models-with-nvidia-tensorrt-and-nvidia-triton/ | *"Simplify deployment of models at scale in production"* |
| NVIDIA | DeepStream | https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Overview.html | *"Streaming analytic toolkit to build AI-powered applications"* |
| NVIDIA | DOCA | https://developer.nvidia.com/blog/programming-the-entire-data-center-infrastructure-with-the-nvidia-doca-sdk/ | *"DOCA is to DPUs what CUDA is to GPUs"* |
| AMD | ROCm | https://www.amd.com/en/products/software/rocm/ai.html | *"Open software stack offering optimizations for AI workloads"* |
| AMD | MIGraphX | https://rocm.docs.amd.com/projects/AMDMIGraphX/en/latest/ | *"Graph compiler and inference engine for AMD hardware"* |
| AMD | MIVisionX | https://rocm.docs.amd.com/projects/MIVisionX/en/latest/ | *"Comprehensive machine intelligence and computer vision toolkit"* |
| Intel | oneAPI | https://oneapi.io/c-for-heterogeneous-programming-oneapi-dpc-and-onetbb/ | *"Cross-architecture language: Data Parallel C++"* |
| Intel | OpenVINO | https://www.intel.com/content/www/us/en/developer/articles/technical/machine-learning-hw-accelerators-web-platform.html | *"Comprehensive toolkit for deep learning on Intel platforms"* |
| Qualcomm | AI Hub | https://www.qualcomm.com/developer/blog/2025/05/deploy-ai-models-on-snapdragon-x-elite-with-qualcomm-ai-hub | *"Developer-centric platform for on-device AI"* |
| Qualcomm | QNN | https://docs.qualcomm.com/nav/home/QNN_general_overview.html | *"All constructs required to construct, optimize, and execute"* |
| Apple | CoreML | https://developer.apple.com/documentation/coreml | *"Unified representation for all models"* |
| Apple | MLX | https://github.com/ml-explore/mlx | *"Array framework for machine learning on Apple silicon"* |
| Apple | MPS | https://developer.apple.com/documentation/metalperformanceshaders | *"High-performance filters for images and compute"* |
| Google | JAX | https://docs.cloud.google.com/tpu/docs/jax-ai-stack | *"Python library for accelerator-oriented array computation"* |
| Google | XLA | https://developers.googleblog.com/building-production-ai-on-google-cloud-tpus-with-jax/ | *"Domain-specific, hardware-agnostic compiler"* |

---

## 9. Comparison Table

| Vendor | Stack | Open Source | HW Utilization | Ease of Use (1-5) | Model Support | Community | Best For |
|--------|-------|-------------|----------------|--------------------|--------------|-----------|---------|
| **NVIDIA** | CUDA/cuDNN/TensorRT/Triton | Partial | Excellent | 4 | Excellent | Very Large | Production AI, cloud, edge |
| **AMD** | ROCm/MIGraphX | Yes | Good | 3 | Good | Growing | Cost-sensitive, open source |
| **Intel** | oneAPI/OpenVINO | Yes | Good | 3 | Good | Medium | Edge deployment, CPU/GPU/NPU |
| **Qualcomm** | AI Hub/QNN | Partial | Good (mobile) | 3 | Good | Small | Mobile/edge AI |
| **Apple** | CoreML/MLX/MPS | Partial (MLX) | Excellent | 4 | Good | Growing | Apple ecosystem, on-device |
| **Google** | JAX/XLA/TPU | Yes | Excellent | 3 | Good | Growing | Cloud TPU, large-scale training |

### Key Differentiators

> *"The CUDA gap demonstrates 60-78 point performance advantage beyond raw hardware specs."*
> — [AI Multiple — CUDA vs ROCm](https://aimultiple.com/cuda-vs-rocm)

> *"AMD ROCm has become a 'real CUDA alternative' in 2026, with PyTorch 2.6 treating it as first-class."*
> — [Fordel Studios](https://fordelstudios.com/research/rocm-vs-cuda-2026-after-testing-both)

> *"Google TPU delivers 4x better price-performance for inference workloads vs NVIDIA H100."*
> — [AI News Hub](https://www.ainewshub.org/post/ai-inference-costs-tpu-vs-gpu-2025)

> *"Apple MLX offers 3x faster inference than PyTorch MPS for LLMs on Apple Silicon."*
> — [Medium](https://medium.com/@koypish/mps-or-mlx-for-domestic-ai-the-answer-will-surprise-you-df4b111de8a0)

---

## 10. Blog Posts & Tutorials

### NVIDIA
- TensorRT Getting Started: https://developer.nvidia.com/tensorrt-getting-started
- Optimizing and Serving with TensorRT + Triton: https://developer.nvidia.com/blog/optimizing-and-serving-models-with-nvidia-tensorrt-and-nvidia-triton/
- DeepStream Guide: https://docs.nvidia.com/metropolis/deepstream/dev-guide/text/DS_Overview.html

### AMD
- ROCm AI Overview: https://www.amd.com/en/products/software/rocm/ai.html
- ROCm vs CUDA Comparison: https://fordelstudios.com/research/rocm-vs-cuda-2026-after-testing-both
- ROCm Local AI Guide: https://www.kunalganglani.com/blog/amd-rocm-vs-cuda-local-ai-open-source-guide

### Intel
- OpenVINO Deployment Guide: https://huggingface.co/blog/deploy-with-openvino
- oneAPI Overview: https://oneapi.io/c-for-heterogeneous-programming-oneapi-dpc-and-onetbb/

### Apple
- MLX GitHub: https://github.com/ml-explore/mlx
- MLX vs PyTorch Comparison: https://metalcloud.space/blog/mlx-vs-pytorch-comparison/

### Google
- JAX AI Stack: https://docs.cloud.google.com/tpu/docs/jax-ai-stack
- Building Production AI on Cloud TPUs: https://developers.googleblog.com/building-production-ai-on-google-cloud-tpus-with-jax/

### Comparisons
- CUDA vs ROCm Analysis: https://aimultiple.com/cuda-vs-rocm
- TPU vs GPU Decision Framework: https://introl.com/blog/google-tpu-vs-nvidia-gpu-infrastructure-decision-framework-2025

---

*Sources compiled from: NVIDIA Developer Blog, AMD ROCm Docs, Intel Developer Articles, Qualcomm Developer Portal, Apple Developer Documentation, Google Cloud TPU Docs, AI Multiple, Fordel Studios*
