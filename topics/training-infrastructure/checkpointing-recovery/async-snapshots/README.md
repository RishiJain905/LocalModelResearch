# Async-Snapshots

> *Techniques for persisting LLM training state without blocking GPU computation — overlapping data staging (GPU→CPU) and persistence (CPU→storage) with forward/backward passes.*

## Table of Contents

1. [Overview](#overview)
2. [How Async Snapshots Work](#how-async-snapshots-work)
3. [Key Papers](#key-papers)
4. [Implementations & Frameworks](#implementations--frameworks)
5. [Performance Comparisons](#performance-comparisons)
6. [When to Use](#when-to-use)
7. [References](#references)

---

## Overview

Async-snapshots (asynchronous checkpointing) eliminates GPU idle time during checkpoint writes by overlapping data staging and persistence with training computation.

> *"Having a consistent global training state for checkpointing is critical to the correctness and effectiveness of the training process."*
> — [Joy Gao, "Demystifying Distributed Checkpointing"](https://expertofobsolescence.substack.com/p/demystifying-distributed-checkpointing)

**Core Value Proposition:**
- Eliminates GPU idle time during checkpoint writes
- Enables higher checkpoint frequency without performance penalty
- Critical for large-scale clusters (1000+ GPUs) where failure rates make frequent checkpoints essential

**Key Challenge:** Model and optimizer state shards (often 100s of GB per GPU) must traverse PCIe links, creating bandwidth contention with training traffic.

---

## How Async Snapshots Work

### Four-Phase Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Training Iteration                            │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐       │
│  │ Forward │───▶│  Loss   │───▶│ Backward│───▶│  Step   │       │
│  └─────────┘    └─────────┘    └─────────┘    └─────────┘       │
│       │                                          ▲              │
│       │                                          │              │
│       ▼           ┌─────────────────────┐        │              │
│  ┌─────────┐      │  GPU→CPU (Pinned)    │────────┘              │
│  │ Immut.  │─────▶│  Async Copy          │  (during forward/     │
│  │ Tensors │      │                     │   backward when        │
│  └─────────┘      └─────────────────────┘   tensors immutable)   │
│                                                 │                  │
│                                                 ▼                  │
│                   ┌─────────────────────┐  ┌─────────┐            │
│                   │  CPU→Storage (Disk)  │◀─│ Persist │            │
│                   │  Async Write         │  │ Thread  │            │
│                   └─────────────────────┘  └─────────┘            │
└─────────────────────────────────────────────────────────────────┘
```

### Key Mechanisms

| Mechanism | Description | Benefit |
|-----------|-------------|---------|
| **Lazy Staging** | GPU→CPU copy starts during forward/backward when tensors are immutable | Overlaps with computation |
| **Non-Blocking** | Training resumes immediately after D2H enqueue | Zero GPU idle time |
| **Parallel Persistence** | CPU→storage writes happen in background threads/processes | Full parallelism |
| **Two-Phase Execution** | Staging (D2H) and Persistence (H2S) use separate physical links | No bandwidth contention |

---

## Key Papers

### DataStates-LLM (HPDC '24)

> *"The model parameters and optimizer state remain immutable during both the forward and backward passes."*

**Paper:** [arXiv:2406.10707](https://arxiv.org/abs/2406.10707)  
**Authors:** Maurya, Underwood, Rafique, Cappello, Nicolae (Argonne/RIT)  
**Framework:** DeepSpeed/Megatron

**Key Contributions:**
- Leverages tensor immutability during forward/backward for background copying
- Coalesced GPU→Host memory transfers using pre-allocated pinned memory
- Streaming multi-level flushing: GPU→Host and Host→Storage in parallel
- **48× faster checkpointing**, **2.2× faster end-to-end training** vs DeepSpeed default

**Results:**
| Metric | DeepSpeed Default | DataStates-LLM |
|--------|------------------|----------------|
| Checkpoint Throughput | 1× | 48× faster |
| End-to-End Training | 1× | 2.2× faster |

**Repository:** [DataStates/datastates-llm](https://github.com/DataStates/datastates-llm)

---

### PyTorch DCP: 6× Faster Async Checkpointing

> *"The Global Interpreter Lock (GIL) in Python prevents multiple native threads from executing Python bytecode at the same time."*

**Blog:** [PyTorch Blog](https://pytorch.org/blog/6x-faster-async-checkpointing/)  
**Authors:** Wright, Vadakkanchery, Mishra et al. (Meta/Crusoe)

**The GIL Problem:**
- GIL contention causes ~7 minute MFU suppression during async checkpoint background processing
- Baseline: 108ms → 162ms step time increase during checkpoint steps

**Solutions Implemented:**
1. **Process-based async checkpointing** — Eliminates GIL contention entirely
2. **Cached save plans** — Amortizes planning cost across checkpoints

**Results at 1856 GPU Scale:**
| Metric | Before | After |
|--------|--------|-------|
| Background Processing | ~436s | ~67s |
| GIL Suppression | ~7 min | Eliminated |

---

### ByteCheckpoint (NSDI '25)

**Paper:** [arXiv:2407.20143](https://arxiv.org/abs/2407.20143)  
**Authors:** Borui Wan et al. (ByteDance/University of Hong Kong)

> *"Parallelism-agnostic checkpoint representation enabling efficient load-time resharding."*

**Key Features:**
- Decoupled tensor metadata from specific parallelism configurations
- Generic saving/loading workflow supporting Megatron, FSDP, DDP, veScale
- **6.05× faster** end-to-end saving, **54.20×** checkpoint stalls reduction
- Production-validated at ByteDance scale (12,288 GPUs)

---

### Flash Checkpoint (DLRover)

**Repository:** [intelligent-machine-learning/dlrover](https://github.com/intelligent-machine-learning/dlrover/blob/master/docs/blogs/megatron_flash_checkpoint.md)

**Megatron-LM Results:**
| Metric | Megatron-LM Default | Flash Checkpoint |
|--------|---------------------|------------------|
| Save (18GB) | 151s | **0.5s** |
| Save (24GB distributed) | 242s | **0.5s** |

---

## Implementations & Frameworks

### PyTorch DCP (Native)

**Documentation:** [Distributed Checkpoint Recipe](https://docs.pytorch.org/tutorials/recipes/distributed_async_checkpoint_recipe.html)

```python
from torch.distributed.checkpoint import async_save

# Basic async save
future = async_save(state_dict, storage_writer=writer, checkpoint_id=path)
```

**Optimizations:**
- `DefaultStager` for fully async staging (background D2H copy)
- Plan caching to amortize planning cost
- Process-based execution to eliminate GIL contention
- Pinned memory staging for faster D2H copy

**Repository:** [pytorch/torchtitan](https://github.com/pytorch/torchtitan)

---

### DeepSpeed Integration

**Tutorial:** [DataStates-Async-Checkpointing](https://www.deepspeed.ai/tutorials/datastates-async-checkpointing/)

```json
{
  "datastates_ckpt": {
    "host_cache_size": 16
  }
}
```

Single JSON entry enables DataStates-LLM in DeepSpeed configurations.

**Limitations:**
- Currently only supports CUDA runtime
- Tested with ZeRO stage-1 only
- No universal or elastic checkpointing support yet

---

### Comparison Table

| Technique | Framework | Async? | Checkpoint Overhead | Scale | License |
|-----------|-----------|--------|---------------------|-------|---------|
| **DataStates-LLM** | DeepSpeed/Megatron | ✓ | 4-48× faster | 512 GPUs | Apache 2.0 |
| **PyTorch DCP** | PyTorch native | ✓ | 6× faster (optimized) | 1856+ GPUs | BSD |
| **Flash Checkpoint** | Megatron/DDP/FSDP | ✓ (sync D2H, async disk) | <1s save | Multi-node | Apache 2.0 |
| **ByteCheckpoint** | Megatron/FSDP/DDP/veScale | ✓ | 6× faster saving | 12,288 GPUs | — |
| **Lightning AsyncCheckpointIO** | PyTorch Lightning | ✓ | Reduced | Limited | Apache 2.0 |

---

## Performance Comparisons

### Optimization Trade-offs

| Optimization | Benefit | Cost/Trade-off |
|--------------|---------|----------------|
| Pinned memory staging | 2× faster D2H copy | Sustained memory pressure |
| Process-based async | Eliminates GIL contention | Higher CPU overhead |
| Cached save plans | Amortizes planning cost | First checkpoint still slow |
| Local checkpointing | 3× faster saves/loads | Nodes must hold replicated state |
| Lazy async (DataStates) | 48× throughput | Host memory can bottleneck |

### Training Badput Formula

> *Training badput = overhead + save time + load time*

From [PyTorch DCP Blog](https://pytorch.org/blog/distributed-checkpoint-efficient-checkpointing-in-large-scale-jobs/):
- **Overhead:** Time spent coordinating checkpoint
- **Save time:** Time to write checkpoint to storage
- **Load time:** Time to restore from checkpoint

---

## When to Use

### Use Async Snapshots When:

| Condition | Rationale |
|-----------|-----------|
| Cluster scale > 64 GPUs | Failure frequency makes frequent checkpoints necessary |
| Checkpoint write time > 30 seconds | Worth the async complexity |
| Model size > 7B parameters | Checkpoint sizes become significant |
| Training stability issues | Require frequent recovery points |

### Consider Alternatives When:

| Condition | Alternative |
|-----------|-------------|
| Cluster < 16 GPUs | Synchronous checkpointing may be acceptable |
| Checkpoint latency not critical | Batch training, less frequent failures |
| Checkpoint-free possible | FlashRecovery pattern if supported |

---

## References

1. Maurya et al. ["DataStates-LLM: Lazy Asynchronous Checkpointing for Large Language Models"](https://arxiv.org/abs/2406.10707) — HPDC '24
2. FlashRecovery Authors. ["FlashRecovery: Fast and Low-Cost Recovery from Failures for Large-Scale Training of LLMs"](https://arxiv.org/abs/2509.03047)
3. Wan et al. ["ByteCheckpoint: A Unified Checkpointing System for Large Foundation Model Development"](https://arxiv.org/abs/2407.20143) — NSDI '25
4. Wright et al. ["6x Faster Async Checkpointing in PyTorch"](https://pytorch.org/blog/6x-faster-async-checkpointing/) — PyTorch Blog 2024
5. ["Distributed Checkpoint: Efficient Checkpointing in Large-Scale Jobs"](https://pytorch.org/blog/distributed-checkpoint-efficient-checkpointing-in-large-scale-jobs/) — PyTorch Blog
6. Gao, Joy. ["Demystifying Distributed Checkpointing"](https://expertofobsolescence.substack.com/p/demystifying-distributed-checkpointing) — October 2024
7. AWS Storage Blog. ["Architecting Scalable Checkpoint Storage for Large-Scale ML Training on AWS"](https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/)
8. DeepSpeed Team. ["DataStates-LLM Checkpointing Engine"](https://www.deepspeed.ai/tutorials/datastates-async-checkpointing/)
9. DataStates Team. [DataStates-llm](https://github.com/DataStates/datastates-llm)
10. PyTorch Team. [TorchTitan](https://github.com/pytorch/torchtitan)
11. DLRover Team. ["Flash Checkpoint for Megatron-LM"](https://github.com/intelligent-machine-learning/dlrover/blob/master/docs/blogs/megatron_flash_checkpoint.md)
12. Maurya, Avinash. [DataStates-LLM Presentation](https://www.youtube.com/watch?v=VpzbLK6R8MI) — YouTube

---

*Part of [[checkpointing-recovery]] — Training Infrastructure*
