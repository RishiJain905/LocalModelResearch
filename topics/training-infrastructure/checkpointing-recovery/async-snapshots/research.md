# Async-Snapshots: Checkpointing/Recovery for LLM Training Infrastructure

## Table of Contents

1. [Overview](#overview)
2. [Academic Papers](#1-academic-papers-arxiv)
3. [Blog Posts & Articles](#2-blog-posts--articles)
4. [Open Source Repositories](#3-open-source-repos-github)
5. [Technical Reports & Documentation](#4-technical-reports--documentation)
6. [Talks & Videos](#5-talks--videos)
7. [Comparison Matrix](#6-comparison-matrix)
8. [Key Insights & Recommendations](#7-key-insights--recommendations)

---

## Overview

Async-snapshots (asynchronous checkpointing) is a technique for persisting LLM training state without blocking GPU computation. Instead of halting training to serialize and save model/optimizer state to storage, async checkpointing overlaps data staging (GPU→CPU) and persistence (CPU→storage) with forward/backward passes.

**Core Value Proposition:**
- Eliminates GPU idle time during checkpoint writes
- Enables higher checkpoint frequency without performance penalty
- Critical for large-scale clusters (1000+ GPUs) where failure rates make frequent checkpoints essential

**Key Challenge:** Model and optimizer state shards (often 100s of GB per GPU) must traverse PCIe links, creating bandwidth contention with training traffic.

---

## 1. Academic Papers (arXiv)

### Paper 1: DataStates-LLM: Lazy Asynchronous Checkpointing for Large Language Models

**URL:** https://arxiv.org/abs/2406.10707  
**Venue:** HPDC '24 (The 33rd International Symposium on High-Performance Parallel and Distributed Computing)  
**Authors:** Avinash Maurya (RIT), Robert Underwood (Argonne), M. Mustafa Rafique (RIT), Franck Cappello (Argonne), Bogdan Nicolae (Argonne)

**Key Concepts:**
> *"The model parameters and optimizer state remain immutable during both the forward and backward passes."*

- Leverages immutability of tensors during forward/backward to copy in background
- Coalesced GPU→Host memory transfers using pre-allocated pinned memory
- Streaming multi-level flushing: GPU→Host and Host→Storage happen in parallel via separate physical links
- Up to **48× faster checkpointing** and **2.2× faster end-to-end training** vs DeepSpeed default

**Why Valuable:**
- First comprehensive academic treatment of lazy async checkpointing for LLMs
- Provides mathematical framework for understanding async snapshot overhead
- Open-source implementation at https://github.com/DataStates/datastates-llm

---

### Paper 2: FlashRecovery: Fast and Low-Cost Recovery from Failures for Large-Scale Training of LLMs

**URL:** https://arxiv.org/html/2509.03047v1  
**Authors:** arXiv preprint

**Key Concepts:**
- Achieves **~150 second recovery time** across 4,800 devices (nearly scale-independent)
- Checkpoint-free recovery within one step using data parallelism redundancy
- Active failure detection (seconds) vs passive timeout-based (up to 30 minutes)
- Optimized ranktable loading: O(n) → O(1) via shared file

**Why Valuable:**
- Pushes beyond checkpoint-centric recovery to nearly checkpoint-free approach
- Demonstrates scale-independent RTO (Recovery Time Objective)
- Active monitoring approach addresses detection latency in traditional systems

---

### Paper 3: ByteCheckpoint: A Unified Checkpointing System for Large Foundation Model Development

**URL:** https://arxiv.org/abs/2407.20143v1  
**Venue:** NSDI '25  
**Authors:** Borui Wan et al. (ByteDance Inc., University of Hong Kong)

**Key Concepts:**
- Parallelism-agnostic checkpoint representation enabling efficient load-time resharding
- Decoupled tensor metadata from specific parallelism configurations
- Generic saving/loading workflow supporting multiple frameworks (Megatron, FSDP, DDP, veScale)
- **6.05× faster** end-to-end saving, **54.20×** checkpoint stalls reduction

**Why Valuable:**
- Solves the "resharding problem" for checkpoints across training stages
- Unified API across different training frameworks
- Production-validated at ByteDance scale (12,288 GPUs)

---

### Paper 4: Lightweight and Fast Error Recovery for LLM Training in a Just-In-Time Context

**URL:** https://dl.acm.org/doi/10.1145/3735358.3735372  
**Venue:** ACM SIGMOD/PODS '25 or similar  

**Key Concepts:**
- Just-In-Time (JIT) checkpointing: creates checkpoints only after failures
- Eliminates periodic checkpoint saving overhead entirely
- Trade-off: recovery requires recomputation vs checkpoint restore

**Why Valuable:**
- Alternative paradigm: checkpoint-on-failure vs periodic checkpointing
- Complements understanding of async snapshot trade-offs

---

## 2. Blog Posts & Articles

### Article 1: Demystifying Distributed Checkpointing

**URL:** https://expertofobsolescence.substack.com/p/demystifying-distributed-checkpointing  
**Author:** Joy Gao  
**Date:** October 27, 2024

**Key Concepts:**
> *"Having a consistent global training state for checkpointing is critical to the correctness and effectiveness of the training process."*

- Three-step checkpoint process: D2H Copy → Serialize → Dump to storage
- 3D parallelism (Data/Pipeline/Tensor) creates synchronization challenges for checkpoints
- Full vs Incremental vs Fuzzy snapshot trade-offs explained
- In-memory checkpoint hierarchy from Gemini paper (SOSP '23)

**Why Valuable:**
- Accessible explanation from infrastructure engineer's perspective
- Connects traditional distributed systems concepts to ML checkpointing
- Excellent overview of why uncoordinated checkpoints don't work

---

### Article 2: Architecting Scalable Checkpoint Storage for Large-Scale ML Training on AWS

**URL:** https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/  
**Platform:** AWS Storage Blog

**Key Concepts:**
- Checkpoint size formula: `(P × Wp) + (P × Os)` where P=params, Wp=weight bytes, Os=optimizer state bytes
- Example: 100B-param model = 1 TB checkpoint per model replica
- Multi-level checkpointing: Local NVMe (5 min) → Instance storage (30 min) → S3/FSx (1-2 hours)
- Asymmetric read/write: writes 1 TB, reads 125 TB across 125 model replicas

**Why Valuable:**
- Storage architecture perspective often missing from compute-focused articles
- Quantifies I/O demands at scale
- Practical AWS implementation patterns with code examples

---

### Article 3: 6x Faster Async Checkpointing in PyTorch: Cached Plans & No GIL Contention

**URL:** https://pytorch.org/blog/6x-faster-async-checkpointing/  
**Authors:** Less Wright, Meet Vadakkanchery, Saurabh Mishra et al. (Meta/Crusoe)

**Key Concepts:**
> *"The Global Interpreter Lock (GIL) in Python prevents multiple native threads from executing Python bytecode at the same time."*

- GIL contention causes ~7 minute MFU suppression during async checkpoint background processing
- **Solution 1:** Process-based async checkpointing (eliminates GIL contention)
- **Solution 2:** Cached save plans (amortize planning cost across checkpoints)
- Results: ~436s → ~67s background processing time at 1856 GPU scale

**Why Valuable:**
- Identifies and solves real-world Python GIL bottleneck in DCP
- Clear before/after metrics for large-scale deployment
- Available in PyTorch nightly via TorchTitan

---

### Article 4: Distributed Checkpoint: Efficient Checkpointing in Large-Scale Jobs

**URL:** https://pytorch.org/blog/distributed-checkpoint-efficient-checkpointing-in-large-scale-jobs/

**Key Concepts:**
- Training badput formula: overhead + save time + load time
- DCP optimizations: plan caching, metadata caching, process-based async, pinned memory staging
- In-cluster local checkpointing: each node saves to local SSD vs shared storage
- Hierarchical checkpoint distribution: Storage → Leader nodes → Worker nodes via NCCL

**Why Valuable:**
- Comprehensive technical overview of PyTorch DCP internals
- Performance breakdown table showing impact of each optimization
- Practical guidance on local checkpointing patterns

---

### Article 5: Understanding How GIL Affects Checkpoint Performance in PyTorch Training

**URL:** https://www.shayon.dev/post/2026/38/understanding-how-gil-affects-checkpoint-performance-in-pytorch-training/

**Key Concepts:**
- GIL sharing between training thread and checkpoint thread causes ~50% step time increase
- Baseline: 108ms → 162ms during checkpoint steps
- Checkpoint thread serializes Python objects, competing for GIL with training thread

**Why Valuable:**
- Detailed measurement of GIL impact on checkpoint performance
- Explains why thread-based async has limitations in Python environments

---

## 3. Open Source Repositories (GitHub)

### Repo 1: DataStates/datastates-llm

**URL:** https://github.com/DataStates/datastates-llm  
**Language:** C++/Python (CUDA, liburing)  
**License:** Apache 2.0

**Description:**
High-performance I/O engine for GPU-accelerated workloads with focus on DeepSpeed/Megatron training. Provides lazy async checkpointing backed by CUDA and io_uring.

**Key Features:**
- Overlaps checkpoint I/O with forward/backward passes
- Pre-allocated pinned host memory for all shards
- Circular buffer manager with producer-consumer pattern
- Single JSON config entry for DeepSpeed integration

**Installation:**
```bash
./install.sh  # Builds C++ core + Python bindings
```

**Why Valuable:**
- Production-grade async checkpointing for DeepSpeed
- Reference implementation of DataStates-LLM paper
- 48× throughput improvement over DeepSpeed default

---

### Repo 2: pytorch/torchtitan

**URL:** https://github.com/pytorch/torchtitan  
**Language:** Python/PyTorch

**Description:**
Proof-of-concept for large-scale LLM training using native PyTorch. Implements DCP with async optimizations including cached plans and process-based checkpointing.

**Key Features:**
- Distributed checkpoint support with async_save
- Modular components for multi-dimensional parallelism
- Interoperable checkpoints loadable into torchtune

**Why Valuable:**
- Reference implementation for PyTorch DCP async optimizations
- Easy-to-understand codebase for learning DCP patterns
- Nightly access to newest async checkpoint features

---

### Repo 3: NVIDIA/Megatron-LM (async checkpoint issue #651)

**URL:** https://github.com/NVIDIA/Megatron-LM/issues/651  
**Framework:** NVIDIA Megatron-LM

**Description:**
Feature request for asynchronous checkpoint saving with minimized training pause time.

**Current Status:**
- Megatron-LM added async checkpoint support in v0.7.0
- Ongoing issues with distributed async checkpoint (v0.8)

**Why Valuable:**
- Shows NVIDIA's approach to async checkpointing in Megatron
- Understanding of current limitations in NVIDIA's ecosystem

---

### Repo 4: intelligent-machine-learning/dlrover (Flash Checkpoint)

**URL:** https://github.com/intelligent-machine-learning/dlrover/blob/master/docs/blogs/megatron_flash_checkpoint.md  
**Framework:** DLRover Flash Checkpoint

**Description:**
Fast checkpoint export for Megatron-LM achieving <1 second save time vs hundreds of seconds.

**Key Features:**
- Synchronous GPU→CPU export, asynchronous CPU→disk persistence
- Parallel save/load for distributed optimizer (no cross-rank communication)
- Supports DDP, FSDP, DeepSpeed, Megatron-LM

**Key Results:**
| Metric | Megatron-LM | Flash Checkpoint |
|--------|-------------|------------------|
| Save (18GB) | 151s | **0.5s** |
| Save (24GB distributed) | 242s | **0.5s** |

**Why Valuable:**
- Practical solution for existing Megatron-LM deployments
- Dramatic reduction in checkpoint pause time

---

### Repo 5: Lightning-AI/pytorch-lightning (AsyncCheckpointIO)

**URL:** https://github.com/Lightning-AI/pytorch-lightning/issues/11561  
**Issue:** #11561

**Description:**
Async checkpointing for PyTorch Lightning using ThreadPoolExecutor for non-blocking saves.

**Current Status:**
- `AsyncCheckpointIO` exists as experimental feature
- Migration to PyTorch DCP in progress (issue #21557)

**Why Valuable:**
- Historical context for async checkpointing in Lightning
- Shows evolution toward DCP-based solutions

---

### Repo 6: llm-d-incubation/llm-d-async

**URL:** https://github.com/llm-d-incubation/llm-d-async

**Description:**
Composable async processor for LLM training infrastructure requests.

**Why Valuable:**
- Alternative architecture for async request management
- General pattern for async operations beyond just checkpointing

---

## 4. Technical Reports & Documentation

### Doc 1: PyTorch DCP Async Checkpointing Recipe

**URL:** https://docs.pytorch.org/tutorials/recipes/distributed_async_checkpoint_recipe.html  
**Version:** PyTorch v2.4.0+

**Key Concepts:**
```python
from torch.distributed.checkpoint import async_save

# Basic async save
future = async_save(state_dict, storage_writer=writer, checkpoint_id=path)
```

- Memory considerations: `checkpoint_size_per_rank × number_of_ranks`
- Pinned memory optimization speeds up GPU→CPU copy but sustains memory pressure
- New in PyTorch 2.9: `DefaultStager` for fully async staging (background D2H copy)

**Why Valuable:**
- Official PyTorch documentation for async checkpoint API
- Best practices for memory management

---

### Doc 2: DataStates-LLM Checkpointing Engine (DeepSpeed Tutorial)

**URL:** https://www.deepspeed.ai/tutorials/datastates-async-checkpointing/

**Key Concepts:**
```json
{
  "datastates_ckpt": {
    "host_cache_size": 16
  }
}
```

- Single JSON entry enables DataStates-LLM in DeepSpeed
- Up to 48× faster checkpointing, 2.2× faster end-to-end training
- Currently only supports CUDA runtime, tested with ZeRO stage-1 only

**Limitations:**
- No universal or elastic checkpointing support yet
- Not fully safetensor compatible due to DeepSpeed restart requirements

**Why Valuable:**
- Official DeepSpeed integration guide for DataStates-LLM
- Quick start for DeepSpeed users

---

### Doc 3: AWS Storage Architecture for ML Checkpointing

**URL:** https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/

**Key Concepts:**
- Hierarchical distribution: S3 → Leader nodes (32 per MR) → Workers via EFA/NCCL
- Reduces storage throughput demand by DP degree factor
- Multi-level checkpointing with tiered frequency

**Why Valuable:**
- Production patterns for AWS-based LLM training infrastructure
- Storage-centric view of checkpoint optimization

---

## 5. Talks & Videos

### Talk 1: DataStates-LLM: Lazy Asynchronous Checkpointing

**URL:** https://www.youtube.com/watch?v=VpzbLK6R8MI  
**Event:** Guest presentation, Argonne National Laboratory  
**Presenter:** Avinash Maurya

**Key Topics:**
- Paper presentation of DataStates-LLM research
- Live Q&A on async checkpointing design decisions
- Performance benchmarks across multiple model scales

**Why Valuable:**
- Direct explanation from paper authors
- Visual presentation of lazy async concepts

---

### Talk 2: AI Explained: Checkpointing in LLMs and the Trade-Offs

**URL:** https://www.youtube.com/watch?v=3Rwgb8s_49g

**Key Topics:**
- Introduction to checkpointing for IT professionals
- Trade-offs between checkpoint frequency and performance
- Recovery patterns for complex training environments

**Why Valuable:**
- Accessible introduction for infrastructure engineers new to ML
- High-level overview of trade-offs without deep technical details

---

## 6. Comparison Matrix

| Technique | Framework | Async? | Checkpoint Overhead | Recovery Time | Scale | License |
|-----------|-----------|--------|---------------------|---------------|-------|---------|
| **DataStates-LLM** | DeepSpeed/Megatron | ✓ | 4-48× faster | Standard | 512 GPUs | Apache 2.0 |
| **PyTorch DCP** | PyTorch native | ✓ | 6× faster (with optimizations) | Standard | 1856+ GPUs | BSD |
| **Flash Checkpoint** | Megatron/DDP/FSDP | ✓ (sync D2H, async disk) | <1s save vs minutes | Standard | Multi-node | Apache 2.0 |
| **FlashRecovery** | Various | ✗ (checkpoint-free) | Eliminated | ~150s (scale-independent) | 10,000+ NPUs | - |
| **ByteCheckpoint** | Megatron/FSDP/DDP/veScale | ✓ | 6× faster saving | Faster loading (resharding) | 12,288 GPUs | - |
| **Lightning AsyncCheckpointIO** | PyTorch Lightning | ✓ | Reduced | Standard | Limited | Apache 2.0 |

---

## 7. Key Insights & Recommendations

### How Async Snapshots Work

1. **Lazy Staging:** GPU→CPU copy starts during forward/backward pass when tensors are immutable
2. **Non-Blocking:** Training resumes immediately after D2H enqueue; copies overlap with computation
3. **Parallel Persistence:** CPU→storage writes happen in background threads/processes
4. **Two-Phase Execution:** Staging (D2H) and Persistence (H2Storage) use separate physical links (PCIe and storage bus)

### Performance Trade-offs

| Optimization | Benefit | Cost/Trade-off |
|--------------|---------|----------------|
| Pinned memory staging | 2× faster D2H copy | Sustained memory pressure |
| Process-based async | Eliminates GIL contention | Higher CPU overhead |
| Cached save plans | Amortizes planning cost | First checkpoint still slow |
| Local checkpointing | 3× faster saves/loads | Nodes must hold replicated state |
| Lazy async (DataStates) | 48× throughput | Host memory can bottleneck |

### When to Use

**Use async snapshots when:**
- Cluster scale > 64 GPUs (failure frequency makes frequent checkpoints necessary)
- Checkpoint write time > 30 seconds (worth the async complexity)
- Model size > 7B parameters (checkpoint sizes become significant)
- Training stability issues require frequent recovery points

**Consider alternatives when:**
- Cluster < 16 GPUs (synchronous checkpointing may be acceptable)
- Checkpoint latency not critical (batch training, less frequent failures)
- Checkpoint-free recovery possible (FlashRecovery pattern if supported)

### Existing Tools/Libraries

| Tool | Best For | Integration Complexity |
|------|----------|------------------------|
| **PyTorch DCP** (async_save) | Native PyTorch, FSDP | Low (2.4+ API) |
| **DataStates-LLM** | DeepSpeed/Megatron users | Low (1 JSON config) |
| **Flash Checkpoint** | Megatron-LM users | Medium (code replacement) |
| **ByteCheckpoint** | Multi-framework, resharding | Medium |
| **Lightning AsyncCheckpointIO** | PyTorch Lightning users | Low (plugin) |

---

## References

1. Maurya et al. "DataStates-LLM: Lazy Asynchronous Checkpointing for Large Language Models." *HPDC '24*. https://arxiv.org/abs/2406.10707
2. FlashRecovery Authors. "FlashRecovery: Fast and Low-Cost Recovery from Failures for Large-Scale Training of LLMs." *arXiv:2509.03047*.
3. Wan et al. "ByteCheckpoint: A Unified Checkpointing System for Large Foundation Model Development." *NSDI '25*. https://arxiv.org/abs/2407.20143
4. PyTorch Team. "Asynchronous Saving with Distributed Checkpoint (DCP)." *PyTorch Documentation*. https://docs.pytorch.org/tutorials/recipes/distributed_async_checkpoint_recipe.html
5. DeepSpeed Team. "DataStates-LLM Checkpointing Engine." *DeepSpeed Tutorial*. https://www.deepspeed.ai/tutorials/datastates-async-checkpointing/
6. Gao, Joy. "Demystifying Distributed Checkpointing." *Expert of Obsolescence*. October 2024.
7. AWS Storage Blog. "Architecting Scalable Checkpoint Storage for Large-Scale ML Training on AWS." 2024.
8. Wright et al. "6x Faster Async Checkpointing in PyTorch." *PyTorch Blog*. 2024.
9. DataStates Team. "DataStates-LLM: Asynchronous I/O Engine." *GitHub*. https://github.com/DataStates/datastates-llm
10. PyTorch Team. "TorchTitan." *GitHub*. https://github.com/pytorch/torchtitan
11. DLRover Team. "Flash Checkpoint for Megatron-LM." *GitHub*. https://github.com/intelligent-machine-learning/dlrover
12. Maurya, Avinash. "DataStates-LLM Presentation." *YouTube*. https://www.youtube.com/watch?v=VpzbLK6R8MI
13. Red Hat Blog. "Just-In-Time Checkpointing: Low Cost Error Recovery from Deep Learning Training Failures." 2024.
14. DLRover Team. "Megatron Flash Checkpoint." *GitHub Docs*. https://github.com/intelligent-machine-learning/dlrover/blob/master/docs/blogs/megatron_flash_checkpoint.md
