# Model Streamers for Large Model Loading on Kubernetes

> **Note**: This document covers techniques and tools for streaming large models into training jobs on Kubernetes — including HuggingFace model downloading, torch.load with streaming, model sharding formats, and integration with Kubernetes PVCs and dynamic provisioning.

## Table of Contents

- [Overview](#overview)
- [HuggingFace Model Downloading](#huggingface-model-downloading)
- [torch.load with Memory-Efficient Streaming](#torchload-with-memory-efficient-streaming)
- [Model Sharding Formats](#model-sharding-formats)
  - [SafeTensors](#safetensors)
  - [DeepSpeed ZeRO Sharding](#deepspeed-zero-sharding)
  - [EasyTensor](#easytensor)
- [Specialized Model Streaming Libraries](#specialized-model-streaming-libraries)
  - [Run:ai Model Streamer](#runai-model-streamer)
  - [CoreWeave Tensorizer](#coreweave-tensorizer)
- [Model Bootstrap / Pre-Download Patterns](#model-bootstrap--pre-download-patterns)
  - [Init Container Download](#init-container-download)
  - [Job-Based Pre-Population](#job-based-pre-population)
- [Streaming Weights into GPU Memory](#streaming-weights-into-gpu-memory)
- [Kubernetes PVC Integration](#kubernetes-pvc-integration)
- [Comparison Tables](#comparison-tables)
- [References](#references)

---

## Overview

Large language models range from 7B parameters (~14GB) to 70B+ parameters (140GB+). Loading these into GPU memory for training introduces unique challenges:

- **Cold start latency**: Downloading then loading can take 10+ minutes
- **Memory pressure**: Full model must fit in GPU VRAM
- **Storage efficiency**: Model weights duplicated across pods/nodes
- **Network bandwidth**: HuggingFace Hub rate limits, cluster egress costs

Model streamers address these by enabling incremental, streaming, or zero-copy loading — avoiding the need to hold the entire model in CPU memory before transferring to GPU.

---

## HuggingFace Model Downloading

### snapshot_download()

**Source:** [huggingface_hub GitHub - snapshot_download.py](https://github.com/huggingface/huggingface_hub/blob/main/src/huggingface_hub/_snapshot_download.py)

```python
from huggingface_hub import snapshot_download

# Download entire repo to local directory
model_dir = snapshot_download(
    repo_id="meta-llama/Llama-3-8B-Instruct",
    local_dir="/path/to/local/model",
    local_dir_use_symlinks=False  # Write actual files instead of symlinks
)
```

> "If `local_dir` is provided, the file structure from the repo will be replicated in this location."

### Key Download Functions

**Source:** [HuggingFace - Download files from the Hub](https://huggingface.co/docs/huggingface_hub/en/guides/download)

| Function | Use Case |
|----------|----------|
| `hf_hub_download()` | Single file download with caching |
| `snapshot_download()` | Entire repo download |
| `huggingface-cli download` | CLI for batch downloading |

### HuggingFace Hub + Kubernetes Integration

**Source:** [HuggingFace Blog - Fine Tuning Llama 3 using Kubernetes](https://huggingface.co/blog/omarkhleif/gaudi-k8s-llm-finetuning)

> "For this blog we will be using a Hugging Face dataset, however, a storage location can also be used for custom datasets as well as output from the optimum-habana example script using a vanilla K8s cluster with an NFS backed storage class."

**Source:** [HuggingFace Transformers - perf_train_cpu_many.md](https://github.com/huggingface/transformers/blob/main/docs/source/en/perf_train_cpu_many.md)

> "The PVC is typically used to hold the dataset, checkpoint files, and the model after it has finished training."

### Production Pattern: Mirror to Object Storage

**Source:** [DigitalOcean - vLLM Kubernetes Model Loading Strategies](https://www.digitalocean.com/community/conceptual-articles/vllm-kubernetes-model-loading-caching-strategies)

> "For **development/experimentation**: Pull directly from HuggingFace (convenient, fast).
> For **production**: Mirror models to storage you control. External service availability should not determine your ability to scale, recover, or replace failed pods."

---

## torch.load with Memory-Efficient Streaming

### mmap=True for Memory-Efficient Loading

**Source:** [PyTorch Tutorials - Tips for Loading nn.Module from Checkpoint](https://docs.pytorch.org/tutorials/recipes/recipes/module_load_state_dict_tips.html)

> "The `mmap` keyword argument to `torch.load` attempts to solve the above two. As its name implies, the `mmap` keyword argument to `torch.load` loads the information from disk when you slice the tensor (or you apply it in a relevant operation)."

**Source:** [PyTorch Forums - torch.load and mmap](https://discuss.pytorch.org/t/torch-load-and-mmap/209629)

```python
# Memory-mapped loading - tensors loaded on-demand
checkpoint = torch.load("model.ckpt", weights_only=True, mmap=True)
```

> "mmap loads the information from disk when you slice the tensor (or you apply it in a relevant operation)."

### Meta Device + Sequential Loading

**Source:** [Analytics Vidhya - Memory-Efficient Model Weight Loading in PyTorch](https://www.analyticsvidhya.com/blog/2024/10/memory-efficient-model-weight-loading-in-pytorch/)

> "Using the meta device, we can first initialize the model without any memory allocation, and then load the model weights directly into GPU memory, bypassing the CPU. By using the meta device and directly loading the model weights into GPU memory, we drastically reduce CPU memory consumption from 6.3 GB to just 1.3 GB."

```python
from torch import nn

# Initialize model on meta device (no memory allocated)
model = MyModel().to("meta")

# Load weights directly to GPU
state_dict = torch.load("weights.pt", map_location="cuda:0")
model.load_state_dict(state_dict)
```

### Key torch.load Parameters

| Parameter | Purpose |
|-----------|---------|
| `mmap=True` | Memory-map file, load tensors on-demand |
| `map_location` | Remap tensor storage to different device |
| `weights_only=True` | Restrict to tensor loading (safe) |
| `assign=True` | Assign weights without in-place modification |

---

## Model Sharding Formats

### SafeTensors

**Source:** [HuggingFace Safetensors - GitHub](https://github.com/huggingface/safetensors)

> "SafeTensors is a new simple format for storing tensors (currently numpy arrays). It provides a way to store and load tensors quickly and safely, without requiring arbitrary code execution during the loading process."

#### Zero-Copy Loading

**Source:** [Kaitchup - Safe, Fast, and Memory Efficient Loading of LLMs with Safetensors](https://kaitchup.substack.com/p/safe-fast-and-memory-efficient-loading)

> "With a pickled model, you would need 200 GB to load 100 GB. safetensors also supports lazy loading, i.e., you can read parts of the model without loading the entire model."

> "When using pickle, Python's 'eval' is applied on the loaded file (i.e., the model). In contrast, safetensors directly loads the model on the specified device without doing any copy."

#### Lazy Loading Issue

**Source:** [GitHub Issue - Stream load models](https://github.com/huggingface/safetensors/issues/373)

> "I'd like to load a 20GB model while having only 8 GB system memory. Currently, safetensors loads the entire model into system memory. Is it possible to load..."

#### Key-Value Streaming Request

**Source:** [GitHub Issue - Efficient key-wise streaming](https://github.com/huggingface/safetensors/issues/443)

> "Feature request: I'm interested in streaming the tensors in a model key by key without having to hold all keys at the same time in memory."

#### fastsafetensors (IBM Research)

**Source:** [IBM Research - Speeding up Model Loading with Fastsafetensors](https://research.ibm.com/publications/speeding-up-model-loading-with-fastsafetensors)

> "We found that this approach underutilized storage throughput and significantly slowed down loading large models with a widely-used model file formats, safetensors. In this work, we present fastsafetensors, a Python library designed to optimize the deserialization of tensors in safetensors files. Our approach first copies groups of on-disk parameters to device memory, where they are directly instantiated as tensor objects. Experimental results show performance improvements of 4.8x to 7.5x in loading models such as Llama (7, 13, and 70 billion parameters)."

### DeepSpeed ZeRO Sharding

**Source:** [DeepSpeed Documentation - ZeRO](https://deepspeed.readthedocs.io/en/latest/zero3.html)

> "The Zero Redundancy Optimizer (ZeRO) removes the memory redundancies across data-parallel processes by partitioning the three model states (optimizer states, gradients, and parameters) across data-parallel processes instead of replicating them."

#### Three Stages

| Stage | Partitioned State | Memory Benefit |
|-------|------------------|----------------|
| **Stage 1** | Optimizer states | ~4x reduction per GPU |
| **Stage 2** | Gradients + Optimizer states | ~8x reduction per GPU |
| **Stage 3** | Parameters + Gradients + Optimizer states | Linear scaling with DP degree |

#### ZeRO-3 Parameter Partitioning

**Source:** [DeepSpeed Tutorial - Zero Redundancy Optimizer](https://www.deepspeed.ai/tutorials/zero/)

> "ZeRO-3 partitions the full model state (i.e., weights, gradients, and optimizer states) to scale memory savings linearly with the degree of data parallelism."

#### ZeRO-Infinity: CPU/NVMe Offload

**Source:** [DeepSpeed Documentation - ZeRO](https://deepspeed.readthedocs.io/en/latest/zero3.html)

> "ZeRO-Infinity (paper), which can offload all model states to both CPU and NVMe memory for huge memory savings."

```json
{
  "zero_optimization": {
    "stage": 3,
    "offload_param": {"device": "nvme", "nvme_path": "/path/to/nvme"},
    "offload_optimizer": {"device": "cpu"},
    "stage3_max_live_parameters": 1e9,
    "stage3_max_reuse_distance": 1e9
  }
}
```

#### ZeRO + Kubernetes (Kubeflow)

**Source:** [Kubeflow - DeepSpeed Guide](https://www.kubeflow.org/docs/components/trainer/user-guides/deepspeed/)

> "This guide describes how to use TrainJob to train or fine-tune AI models with DeepSpeed. Kubeflow Trainer uses the MPI-based runtime and `mpirun` launcher to run DeepSpeed scripts on every training node."

### EasyTensor

**Source:** [Coinbase Blog - Accelerating Deep Learning Adoption at Coinbase](https://www.coinbase.com/blog/accelerating-deep-learning-adoption-at-coinbase)

> "EasyTensor is a Python framework that simplifies building, training, and deploying deep learning models on tabular and multimodal data."

> **Note**: EasyTensor appears to be an internal Coinbase framework. Public documentation is limited. Search continues for additional public EasyTensor resources.

---

## Specialized Model Streaming Libraries

### Run:ai Model Streamer

**Source:** [Google Cloud - Accelerate Model Loading on GKE with Run:ai Model Streamer](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/run-ai-model-streamer)

> "This document shows you how to accelerate loading the large AI model weights from Cloud Storage by using Run:ai Model Streamer with the vLLM inference server on Google Kubernetes Engine (GKE). The solution described in this document uses three core components—Run:ai Model Streamer, vLLM, and the `safetensors` file format—to accelerate the process of loading model weights from Cloud Storage onto GPU or TPU nodes."

#### Architecture

> "Run:ai Model Streamer integrates with vLLM on GKE to accelerate model loading by streaming model weights directly from Cloud Storage to accelerator memory, bypassing local disk."

#### Key Benefits

| Benefit | Description |
|---------|-------------|
| **Reduced Cold Start Times** | Loads model weights **up to 6x faster** than conventional methods |
| **Direct Cloud Storage Access** | Stream from S3, GCS, Azure Blob without local download |
| **Accelerator Utilization** | GPUs/TPUs dedicate more time to inference tasks |

#### vLLM Integration

**Source:** [vLLM Documentation - Loading models with Run:ai Model Streamer](https://docs.vllm.ai/en/stable/models/extensions/runai_model_streamer/)

```bash
vllm serve s3://bucket/model --load-format runai_streamer
```

**Source:** [vLLM Documentation](https://docs.vllm.ai/en/stable/models/extensions/runai_model_streamer/)

> "Run:ai Model Streamer is a library that reads tensors in concurrency while streaming them to GPU memory. vLLM supports loading weights in Safetensors format using this streamer."

#### Cloud Storage Support

| Provider | Protocol | Example |
|----------|----------|---------|
| **Amazon S3** | `s3://` | `s3://bucket/model` |
| **Google Cloud Storage** | `gs://` | `gs://bucket/model` |
| **Azure Blob** | `az://` | `az://container/path` |

#### Tunable Parameters

**Source:** [vLLM Documentation](https://docs.vllm.ai/en/stable/models/extensions/runai_model_streamer/)

```bash
vllm serve /path/to/model \
    --load-format runai_streamer \
    --model-loader-extra-config '{"concurrency":16, "memory_limit":5368709120}'
```

| Parameter | Purpose |
|-----------|---------|
| `distributed` | Enable distributed streaming from object storage |
| `concurrency` | Number of OS threads reading tensors |
| `memory_limit` | CPU memory buffer size for reading tensors |

#### Sharded Model Loading

**Source:** [vLLM Documentation](https://docs.vllm.ai/en/stable/models/extensions/runai_model_streamer/)

> "The sharded loader supports all the same tunable parameters as the regular Run:ai Model Streamer, including `concurrency` and `memory_limit`."

```bash
vllm serve /path/to/sharded/model \
    --load-format runai_streamer_sharded \
    --model-loader-extra-config '{"pattern":"model-rank-{rank}-part-{part}.safetensors"}'
```

### CoreWeave Tensorizer

**Source:** [CoreWeave Blog - Decrease PyTorch Model Load Times with CoreWeave's Tensorizer](https://www.coreweave.com/blog/coreweaves-tensorizer-decrease-pytorch-model-load-times)

> "CoreWeave Tensorizer solves this by enabling extremely fast and efficient PyTorch model loading through a 'zero-copy' approach."

#### Key Features

> "**Zero-copy model loading**: Instead of loading the entire model into RAM before transferring to GPU, Tensorizer pulls data chunk by chunk"

> "**Negligible RAM usage**: Uses only a buffer of the largest tensor size plus metadata to fetch tensor locations"

#### Performance

| Method | Median Speed |
|--------|-------------|
| `torch.load()` | ~1.9 GiB/s |
| `tensorizer file` | ~2.25 GiB/s |
| `tensorizer file, plaid_mode` | ~4.6 GiB/s |

**Source:** [GitHub - coreweave/tensorizer](https://github.com/coreweave/tensorizer)

> "Extremely fast model loads from HTTP/HTTPS, Redis, and S3 endpoints. GPT-J (20GB) loads at wire-speed (~5GB/s) on a 40GbE network."

#### Kubernetes Integration

**Source:** [CoreWeave - How to Deploy AI Models on CoreWeave](https://neuraplus-ai.github.io/blog/how-to-deploy-ai-models-on-coreweave.html)

> "CoreWeave runs on Kubernetes, which makes AI model deployment structured and reproducible."

#### vLLM Integration

**Source:** [vLLM Documentation - Loading models with CoreWeave's Tensorizer](https://docs.vllm.ai/en/stable/models/extensions/tensorizer/)

```bash
vllm serve s3://my-bucket/model \
    --load-format tensorizer \
    --model-loader-extra-config '{"deserialization_kwargs": {"num_readers": 2}}'
```

#### Serialization

**Source:** [vLLM Documentation](https://docs.vllm.ai/en/stable/models/extensions/tensorizer/)

```bash
python examples/others/tensorize_vllm_model.py \
    --model facebook/opt-125m \
    serialize \
    --serialized-directory s3://my-bucket \
    --suffix v1
```

---

## Model Bootstrap / Pre-Download Patterns

### Init Container Download

**Source:** [DigitalOcean - vLLM Kubernetes Model Loading Strategies](https://www.digitalocean.com/community/conceptual-articles/vllm-kubernetes-model-loading-caching-strategies)

> "Init container runs before main vLLM container. Downloads model to shared volume (typically emptyDir). Main container starts with model already in place."

**Benefits:**
- Clean separation of concerns
- Specialized download tools
- Retry logic support
- Different credentials for private registries

**Source:** [Medium - Using Kubernetes Init Containers to Decouple ML Application Deployment from Models](https://medium.com/@christianberzi/using-kubernetes-init-containers-to-decouple-the-deployment-of-machine-learning-applications-from-1d557ad52b99)

> "Init Containers run to completion before the main container starts, and each must complete successfully."

#### Example Init Container Pattern

```yaml
spec:
  initContainers:
  - name: model-downloader
    image: huggingface/transformers-pytorch-deepspeed
    command: ["sh", "-c"]
    args:
      - "huggingface-cli download meta-llama/Llama-3-8B-Instruct --local-dir /models/llama"
    volumeMounts:
    - name: model-volume
      mountPath: /models
  containers:
  - name: training-container
    image: pytorch/pytorch:latest
    command: ["python", "train.py"]
    volumeMounts:
    - name: model-volume
      mountPath: /models
```

### Job-Based Pre-Population

**Source:** [DigitalOcean - vLLM Kubernetes Model Loading Strategies](https://www.digitalocean.com/community/conceptual-articles/vllm-kubernetes-model-loading-caching-strategies)

| Approach | Description |
|----------|-------------|
| **Per-Node Job** | Job/DaemonSet downloads models to node-local storage (hostPath on local SSD) |
| **Central Job** | Kubernetes Job pre-populates shared storage (PVC/NFS) before inference pods run |
| **Argo Workflows** | Pipeline-based model management with versioning |

**Source:** [ML in Production - Kubernetes Jobs for Machine Learning](https://mlinproduction.com/k8s-jobs/)

> "By using Kubernetes Jobs to perform model training and model inference we guarantee that are processes are fault tolerant and resilient."

### Pre-Tensorized Models on CoreWeave Cloud

**Source:** [GitHub - coreweave/tensorizer](https://github.com/coreweave/tensorizer)

> "CoreWeave Cloud provides several **free pre-Tensorized models** for immediate use."

| Model | Precision | S3 URI |
|-------|-----------|--------|
| EleutherAI/gpt-j-6B | fp16 | `s3://tensorized/EleutherAI/gpt-j-6B/fp16/model.tensors` |
| EleutherAI/gpt-neox-20b | fp16 | `s3://tensorized/EleutherAI/gpt-neox-20b/fp16/model.tensors` |

---

## Streaming Weights into GPU Memory

### Torch-TensorRT Weight Streaming

**Source:** [PyTorch TensorRT - Weight Streaming Example](https://docs.pytorch.org/TensorRT/tutorials/_rendered_examples/dynamo/weight_streaming_example.html)

> "Weight streaming in TensorRT is a powerful feature designed to overcome GPU memory limitations when working with large models. It enables running models larger than available GPU memory by streaming weight data from host (CPU) memory to GPU memory during inference."

```python
# Compile with weight streaming
model = trace.compile(backend="torch_tensorrt", options={"enable_weight_streaming": True})
```

> "Streaming larger amounts of memory will likely result in lower performance. But if streaming weights allows the user to run larger batch sizes and it can lead to higher throughput."

#### Context Manager Control

**Source:** [PyTorch TensorRT Documentation](https://docs.pytorch.org/TensorRT/tutorials/_rendered_examples/dynamo/weight_streaming_example.html)

```python
# Limit streaming budget
with ctx.weight_streaming_budget(budget_size):
    output = model(input)
```

### NVIDIA GPUDirect Storage (GDS)

**Source:** [NVIDIA GPUDirect Storage Overview](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

> "GPUDirect Storage creates a direct data path between storage and GPU memory and avoids extra copies through a bounce buffer in the CPU's memory."

> "GDS enables a direct data path for direct memory access (DMA) transfers between GPU memory and storage, which avoids a bounce buffer. This direct path increases system bandwidth and decreases the latency and utilization load on the CPU."

#### cuFile API

**Source:** [NVIDIA GPUDirect Storage Overview](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

> "cuFile API is the primary API for file I/O independent of memory type for GPU applications. Best-performing access to local or distributed file/block storage. Increased performance and lower CPU load vs standard Linux file IO."

---

## Kubernetes PVC Integration

### Storage Strategies Comparison

**Source:** [DigitalOcean - vLLM Kubernetes Model Loading Strategies](https://www.digitalocean.com/community/conceptual-articles/vllm-kubernetes-model-loading-caching-strategies)

| Strategy | Startup Latency | Storage Efficiency | Complexity |
|----------|----------------|-------------------|------------|
| **vLLM Native (local)** | Slow (minutes) | Low (duplicated) | Easiest |
| **vLLM Native (shared PVC)** | Medium | High (one copy) | Medium |
| **Init Container + PVC** | Medium | High | Medium |
| **Job-Based Pre-Population** | Fast for subsequent pods | High | Higher |

### Dynamic Provisioning with StorageClasses

**Source:** [LocalModelResearch - Dynamic-Provisioning.md](./Dynamic-Provisioning.md)

> "Dynamic provisioning allows you to create PVs and PVCs on-demand. You can use Helm or direct Kubernetes manifests to customize the Job template."

### ReadWriteMany (RWX) for Multi-Pod Access

**Source:** [Stack Overflow - Kubernetes Block, File, Object](https://stackoverflow.com/questions/65868661/how-kubernetes-block-file-and-object-storage-types-are-used-inside-containers)

> "NFS is Read-Write Many (RWX) which means it can be mounted read-write to many pods."

### Model Bootstrap with Persistent Volume

**Source:** [HPE AI Essentials - Manual Model Pre-loading (PVC)](https://support.hpe.com/hpesc/public/docDisplay?docId=a00aie112hen_us&page=MLIS/admin/manual-preloading.html&docLocale=en_US)

> "Manual pre-loading is an advanced method for pre-loading models in MLIS using PersistentVolumeClaims (PVCs) to store models."

### Anywhere Cache for GKE

**Source:** [Google Cloud - Accelerate Model Loading on GKE with Run:ai Model Streamer](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/run-ai-model-streamer)

> "Optional: Cloud Storage Rapid Cache caches bucket data in the same zone as GKE nodes, further accelerating access."

---

## Comparison Tables

### Model Streaming Libraries

| Library | Zero-Copy | Cloud Storage | Kubernetes | Sharding | License |
|---------|-----------|---------------|------------|----------|---------|
| **Run:ai Model Streamer** | Yes | S3/GCS/Azure | Native vLLM | Yes | Proprietary |
| **CoreWeave Tensorizer** | Yes | S3/HTTP/Redis | Via vLLM | No | Apache 2.0 |
| **SafeTensors (fastsafetensors)** | Partial | Via custom impl | Via vLLM | No | Apache 2.0 |
| **Torch-TensorRT** | Streaming | No | No | No | NVIDIA |
| **DeepSpeed ZeRO-3** | Yes (partitioned) | No | Via Kubeflow | Yes | Apache 2.0 |

### Kubernetes Model Loading Patterns

| Pattern | Best For | Limitations |
|---------|----------|-------------|
| **HuggingFace Hub direct** | Dev/test | Rate limits, external dependency |
| **Init Container download** | Separated concerns | Still downloads per-pod without shared storage |
| **Job pre-population** | Production multi-replica | Storage cleanup/versioning complexity |
| **Run:ai Model Streamer** | GKE + vLLM inference | GKE-specific, Safetensors only |
| **CoreWeave Tensorizer** | CoreWeave Cloud | Cloud-specific |

### torch.load Options

| Option | Memory Efficiency | Use Case |
|--------|-------------------|----------|
| `torch.load()` default | Low (2x model size) | Fast local SSDs |
| `torch.load(mmap=True)` | High (on-demand) | Memory-constrained |
| `meta device + load_state_dict(assign=True)` | Highest | GPU-only loading |
| `weights_only=True` | Safety benefit | Untrusted checkpoints |

---

## References

### Official Documentation

- [PyTorch torch.load](https://docs.pytorch.org/docs/stable/generated/torch.load.html)
- [HuggingFace huggingface_hub](https://huggingface.co/docs/huggingface_hub)
- [DeepSpeed ZeRO Documentation](https://deepspeed.readthedocs.io/en/latest/zero3.html)
- [vLLM Model Loader Extensions](https://docs.vllm.ai/en/stable/models/extensions/)
- [NVIDIA GPUDirect Storage](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

### GitHub Repositories

- [huggingface/safetensors](https://github.com/huggingface/safetensors)
- [coreweave/tensorizer](https://github.com/coreweave/tensorizer)
- [deepspeedai/DeepSpeed](https://github.com/deepspeedai/DeepSpeed)
- [run-ai/runai-model-streamer](https://github.com/run-ai/runai-model-streamer)

### Blog Posts & Technical Reports

- [Google Cloud - Accelerate Model Loading on GKE with Run:ai Model Streamer](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/run-ai-model-streamer)
- [CoreWeave - Decrease PyTorch Model Load Times with CoreWeave's Tensorizer](https://www.coreweave.com/blog/coreweaves-tensorizer-decrease-pytorch-model-load-times)
- [DigitalOcean - vLLM Kubernetes: Model Loading & Caching Strategies](https://www.digitalocean.com/community/conceptual-articles/vllm-kubernetes-model-loading-caching-strategies)
- [IBM Research - Speeding up Model Loading with Fastsafetensors](https://research.ibm.com/publications/speeding-up-model-loading-with-fastsafetensors)
- [Kubeflow - DeepSpeed Guide](https://www.kubeflow.org/docs/components/trainer/user-guides/deepspeed/)
