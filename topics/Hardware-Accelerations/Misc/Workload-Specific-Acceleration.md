# Workload-Specific Acceleration

> *Domain-specific AI acceleration beyond general ML — covering optimized stacks for vision, speech, recommendation systems, LLMs, scientific computing, and medical imaging.*

---

## Table of Contents

1. [Vision & Image AI](#1-vision--image-ai)
2. [Speech & Audio AI](#2-speech--audio-ai)
3. [Recommendation Systems](#3-recommendation-systems)
4. [LLM & Transformer Acceleration](#4-llm--transformer-acceleration)
5. [RAG & Vector Databases](#5-rag--vector-databases)
6. [Scientific Computing](#6-scientific-computing)
7. [Medical Imaging](#7-medical-imaging)
8. [Training vs Inference](#8-training-vs-inference)
9. [Key Sources & Quotes](#9-key-sources--quotes)
10. [Summary Table](#10-summary-table)

---

## 1. Vision & Image AI

### NVIDIA DALI (Data Loading Library)

> *"NVIDIA DALI: A GPU-accelerated data augmentation and loading library for pixel-deep learning pipelines that significantly speeds up input preprocessing on CPUs and GPUs."*
> — [NVIDIA DALI](https://developer.nvidia.com/technical-blog/nvidia-dali-enabling-gpu-accelerated-image-data-pipelines)

- Replaces native PyTorch/DataLoader CPU-bound preprocessing
- GPU-accelerated resizing, crop, flip, color adjustments
- Reduces CPU bottleneck in vision training pipelines

### OpenCV DNN (Deep Neural Network Module)

> *"OpenCV's DNN module is a lightweight solution for running inference using models trained in major frameworks like Caffe, TensorFlow, PyTorch, ONNX, and OpenVINO."*
> — [OpenCV DNN](https://docs.opencv.org/4.x/d6/d0f/group__dnn.html)

- Cross-platform CPU/GPU inference
- Supports INT8 quantization for edge deployment
- Hardware acceleration via Vulkan, OpenCL, CUDA, OpenVINO

### cuDNN Convolution Primitives

- Optimized forward/backward convolution for CNNs
- Winograd convolution for FP32/FP16 inference
- Im2Col + GEMM for general convolution

### Vision Model Accelerators

| Tool | Focus | URL |
|------|-------|-----|
| **NVIDIA DALI** | GPU data augmentation | https://developer.nvidia.com/technical-blog/nvidia-dali-enabling-gpu-accelerated-image-data-pipelines |
| **OpenCV DNN** | Cross-framework inference | https://docs.opencv.org/4.x/d6/d0f/group__dnn.html |
| **TorchVision** | Native PyTorch transforms | https://pytorch.org/vision/stable/transforms.html |

---

## 2. Speech & Audio AI

### Whisper.cpp

> *"whisper.cpp is a high-performance inference engine for the Whisper model, enabling efficient transcription and translation on CPU and GPU with quantized weights."*
> — [whisper.cpp GitHub](https://github.com/ggerganov/whisper.cpp)

- Pure C/C++ implementation, no Python dependencies
- Support for INT8/INT4 quantization on CPU and GPU
- Cross-platform: Linux, macOS, Windows, mobile, browser
- Large vocabulary speech recognition

### NVIDIA Riva

> *"NVIDIA Riva is a GPU-accelerated SDK for building real-time speech AI applications with automatic speech recognition (ASR) and text-to-speech (TTS)."*
> — [NVIDIA Riva](https://developer.nvidia.com/riva)

- Deep learning-based ASR and TTS
- Optimized for low-latency streaming
- Pretrained models for multiple languages

### Coqui STT (DeepSpeech Fork)

> *"Coqui STT is an open-source deep-learning speech recognition engine trained on Mozilla's open audio dataset."*
> — [Coqui GitHub](https://github.com/coqui-ai/STT-PyTorch)

- Open-source alternative to proprietary ASR
- Fine-tunable on custom datasets
- ONNX export for cross-platform deployment

### Audio AI Acceleration Stack

| Tool | Type | Quantization | URL |
|------|------|-------------|-----|
| **Whisper.cpp** | ASR/STT | INT8/INT4 CPU+GPU | https://github.com/ggerganov/whisper.cpp |
| **NVIDIA Riva** | ASR/TTS | Full precision | https://developer.nvidia.com/riva |
| **Coqui STT** | ASR | FP16/INT8 | https://github.com/coqui-ai/STT-PyTorch |
| **Bark.cpp** | TTS | INT8 | https://github.com/gi人家ckmanogg/bark.cpp |

---

## 3. Recommendation Systems

### NVIDIA Merlin NVTabular

> *"NVTabular is a feature engineering and preprocessing library designed to manipulate large-scale datasets and prepare them for training recommender models."*
> — [NVIDIA Merlin](https://developer.nvidia.com/nvidia-merlin)

- GPU-accelerated feature engineering on massive datasets
- Integrates with TensorFlow, PyTorch, HugeCTR
- Handles datasets with billions of rows

### HugeCTR

> *"HugeCTR is a GPU-accelerated recommender framework designed to distribute training across multiple GPUs and nodes."*
> — [NVIDIA HugeCTR](https://github.com/NVIDIA/HugeCTR)

- Training recommender models at scale
- Embedding lookups optimized for memory bandwidth
- Multi-GPU and multi-node scaling

### Recommendation System Stack

| Tool | Focus | URL |
|------|-------|-----|
| **Merlin NVTabular** | Feature engineering | https://developer.nvidia.com/nvidia-merlin |
| **HugeCTR** | Distributed training | https://github.com/NVIDIA/HugeCTR |
| **TensorFlow Recommenders** | TF ecosystem | https://www.tensorflow.org/recommenders |
| **RecBole** | Research framework | https://recbole.io/ |

---

## 4. LLM & Transformer Acceleration

### vLLM

> *"vLLM is an open-source library for LLM inference and serving. It uses PagedAttention to manage attention key/value cache memory efficiently."*
> — [vLLM GitHub](https://github.com/vllm-project/vllm)

- **PagedAttention**: Virtual memory management for KV cache
- Continuous batching for high throughput
- FP8 weight quantization support
- Popular backends: OpenAI-compatible API

### llama.cpp

> *"llama.cpp is a pure C/C++ implementation of LLaMA inference with support for 4-bit integer quantization, Metal GPU acceleration on macOS, and CPU inference with AVX2/AVX512 optimizations."*
> — [llama.cpp GitHub](https://github.com/ggerganov/llama.cpp)

- INT4/INT5/INT8 quantization on CPU and GPU
- Metal GPU acceleration on Apple Silicon
- No Python dependency — standalone binaries
- GGUF model format for quantization

### TensorRT-LLM

> *"TensorRT-LLM provides developers with an easy-to-use API to define, optimize, and execute inference for large language models in production."*
> — [NVIDIA TensorRT-LLM](https://developer.nvidia.com/tensorrt)

- In-flight batching for LLMs
- FP8 quantization for H100
- Custom attention kernels (FlashAttention integration)
- Optimized for GPT, LLaMA, Mistral, Falcon

### ExecuTorch

> *"ExecuTorch is a runtime for deploying Llama models on-device — enabling AI features to run locally on mobile and edge devices."*
> — [Meta ExecuTorch](https://pytorch.org/executorch-overview)

- On-device LLM inference for iOS/Android
- Quantized INT4/INT8 model support
- Cross-platform: iOS, Android, Embedded Linux
- Exploits Neural Engine on Apple Silicon

### ONNX Runtime

> *"ONNX Runtime is a cross-platform inference engine that supports models from PyTorch, TensorFlow, and other frameworks via the ONNX format."*
> — [ONNX Runtime](https://onnxruntime.ai/docs/)

- 40+ Execution Providers (CUDA, TensorRT, OpenVINO, QNN, CoreML)
- Graph optimizations, operator fusion, quantization
- CPU/GPU/NPU cross-vendor deployment

---

## 5. RAG & Vector Databases

### FAISS (Facebook AI Similarity Search)

> *"FAISS is a library for efficient similarity search and clustering of dense vectors. It contains algorithms that search in sets of vectors of any size."*
> — [FAISS GitHub](https://github.com/facebookresearch/faiss)

- GPU-accelerated indexing (IVF, HNSW on CUDA)
- Billion-scale vector search
- Supports INT8 quantization for memory efficiency

### Milvus

> *"Milvus is an open-source vector database built to power embedding similarity search and AI applications."*
> — [Milvus](https://milvus.io/)

- Distributed vector database for production RAG
- GPU-accelerated indexing (RAFT-based)
- Connects to mainstream LLM orchestration tools

### Qdrant

> *"Qdrant is a vector similarity search engine with extended filtering support."*
> — [Qdrant](https://qdrant.tech/)

- High-performance vector search with Rust implementation
- GPU acceleration via optional CUDA support
- Standalone or Kubernetes deployment

### Vector DB Comparison

| Database | GPU Acceleration | Max Scale | URL |
|----------|----------------|-----------|-----|
| **FAISS** | Yes (CUDA) | 2B vectors | https://github.com/facebookresearch/faiss |
| **Milvus** | Yes (RAFT) | 10B+ vectors | https://milvus.io/ |
| **Qdrant** | Optional | 1B+ vectors | https://qdrant.tech/ |
| **Pinecone** | Managed | Unlimited | https://www.pinecone.io/ |
| **Weaviate** | Yes | 10B+ vectors | https://weaviate.io/ |

---

## 6. Scientific Computing

### Molecular Dynamics

| Tool | Focus | GPU Support | URL |
|------|-------|------------|-----|
| **AlphaFold** | Protein folding | TPU/GPU | https://alphafold.ebi.ac.uk/ |
| **RoseTTAFold** | Protein structure | CUDA | https://github.com/RosettaCommons/RoseTTAFold |
| **OpenMM** | MD simulations | CUDA/OpenCL | https://openmm.org/ |
| **GROMACS** | MD simulations | GPU | https://www.gromacs.org/ |

### Weather & Climate AI

> *"FourCastNet is a weather forecasting model that uses Fourier neural operators for high-resolution global weather predictions — running 45,000x faster than traditional numerical weather prediction."*
> — [FourCastNet](https://www.nvidia.com/en-us/ai-data-science/foundation-models/fourcastnet/)

### CFD (Computational Fluid Dynamics)

| Tool | Focus | URL |
|------|-------|-----|
| **NVIDIA SimNet** | Physics-informed neural networks | https://developer.nvidia.com/simnet |
| **DeepFlow** | Fluid simulation | https://doi.org/10.1016/j.jcp.2021.110580 |

---

## 7. Medical Imaging

### NVIDIA Clara

> *"NVIDIA Clara is a platform for AI-assisted annotation, federated learning for privacy-preserving training, and deployment of medical imaging models."*
> — [NVIDIA Clara](https://developer.nvidia.com/clara)

- AI-assisted annotation for CT, MRI, X-ray
- Federated learning for multi-hospital collaboration
- MONAI integration

### MONAI (Medical Open Network for AI)

> *"MONAI is a framework for deep learning in healthcare imaging, providing domain-optimized foundational capabilities for developing training workflows."*
> — [MONAI](https://monai.io/)

- Domain-specific: CT, MRI, pathology, ultrasound
- Supported by NHS, King's College London, Stanford
- GPU acceleration via PyTorch

### Intel OpenFL

> *"OpenFL is an open-source framework forFederated Learning, designed to enable secure, privacy-preserving collaborative training across institutions."*
> — [Intel OpenFL](https://github.com/intel/openfl)

- Privacy-preserving training for medical imaging
- Intel CPU/GPU optimization

---

## 8. Training vs Inference

| Aspect | Training Acceleration | Inference Acceleration |
|--------|---------------------|----------------------|
| **Primary Focus** | Gradient computation, large batch sizes | Low latency, high throughput |
| **Precision** | FP32/BF16 gradients | INT8/FP16 inference |
| **Key Libraries** | cuDNN, Transformer Engine, DeepSpeed | TensorRT, vLLM, ONNX Runtime |
| **Batching** | Large, static batches | Dynamic batching, continuous batching |
| **Memory** | Activations, gradients | KV cache, weights |
| **Tools** | Mixed-precision, gradient checkpointing | Quantization (INT4/INT8), pruning |
| **Key Challenge** | Scaling across GPUs (FSDP, DeepSpeed) | Minimizing latency (KV cache, kernel fusion) |

---

## 9. Key Sources & Quotes

| Domain | Tool | URL | Key Quote |
|--------|------|-----|-----------|
| Vision | NVIDIA DALI | https://developer.nvidia.com/technical-blog/nvidia-dali-enabling-gpu-accelerated-image-data-pipelines | *"GPU-accelerated data augmentation and loading library"* |
| Vision | OpenCV DNN | https://docs.opencv.org/4.x/d6/d0f/group__dnn.html | *"Lightweight inference using models from Caffe, TF, PyTorch, ONNX"* |
| Speech | Whisper.cpp | https://github.com/ggerganov/whisper.cpp | *"High-performance inference engine for Whisper"* |
| Speech | NVIDIA Riva | https://developer.nvidia.com/riva | *"GPU-accelerated SDK for real-time speech AI"* |
| Recommender | NVIDIA Merlin | https://developer.nvidia.com/nvidia-merlin | *"Feature engineering for recommender models"* |
| LLM | vLLM | https://github.com/vllm-project/vllm | *"PagedAttention for efficient KV cache management"* |
| LLM | llama.cpp | https://github.com/ggerganov/llama.cpp | *"Pure C/C++ LLaMA with INT4 quantization"* |
| LLM | TensorRT-LLM | https://developer.nvidia.com/tensorrt | *"Easy-to-use API for LLM inference in production"* |
| LLM | ExecuTorch | https://pytorch.org/executorch-overview | *"On-device LLM deployment"* |
| RAG | FAISS | https://github.com/facebookresearch/faiss | *"Efficient similarity search of dense vectors"* |
| RAG | Milvus | https://milvus.io/ | *"Vector database for embedding similarity search"* |
| Science | FourCastNet | https://www.nvidia.com/en-us/ai-data-science/foundation-models/fourcastnet/ | *"45,000x faster than NWP"* |
| Medical | MONAI | https://monai.io/ | *"Domain-optimized deep learning for healthcare imaging"* |
| Medical | NVIDIA Clara | https://developer.nvidia.com/clara | *"AI-assisted medical imaging"* |

---

## 10. Summary Table

| Domain | Key Tools | Primary Acceleration | URL |
|--------|----------|---------------------|-----|
| **Vision** | DALI, OpenCV DNN | CUDA, TensorRT | NVIDIA/OpenCV |
| **Speech** | Whisper.cpp, Riva | INT8 CPU/GPU, CUDA | ggerganov/NVIDIA |
| **Recommendations** | Merlin, HugeCTR | CUDA, multi-GPU | NVIDIA |
| **LLM Inference** | vLLM, llama.cpp, TRT-LLM | PagedAttention, INT4/INT8 | vllm/ggerganov/NVIDIA |
| **LLM On-Device** | ExecuTorch, MLX | Neural Engine, INT4 | Meta/Apple |
| **RAG** | FAISS, Milvus, Qdrant | CUDA, RAFT | Meta/Milvus/Qdrant |
| **Science** | AlphaFold, FourCastNet | TPU/GPU | DeepMind/NVIDIA |
| **Medical** | MONAI, Clara | CUDA, federated learning | MONAI/NVIDIA |

---

*Sources compiled from: NVIDIA Developer Blog, GitHub repositories (whisper.cpp, vLLM, llama.cpp, FAISS, MONAI), OpenCV Docs, Milvus.io, MONAI.io, PyTorch Blog, Meta ExecuTorch*
