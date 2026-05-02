# Checkpointing & Recovery

> *Fault tolerance techniques for large-scale LLM training — from traditional periodic checkpoints to emerging checkpoint-less approaches.*

## Table of Contents

1. [Overview](#overview)
2. [Topics](#topics)
3. [Quick Reference](#quick-reference)

---

## Overview

Fault tolerance is critical for LLM training at scale. As model sizes grow (7B–1T+ parameters) and clusters expand (64–100,000+ GPUs), failures become inevitable rather than exceptional.

**The Core Trade-off:**
- More frequent checkpoints → faster recovery, higher I/O overhead
- Less frequent checkpoints → lower overhead, longer recovery time
- No checkpoints → zero overhead, requires alternative recovery mechanisms

> *"Training badput = overhead + save time + load time"*
> — [PyTorch Distributed Checkpoint Blog](https://pytorch.org/blog/distributed-checkpoint-efficient-checkpointing-in-large-scale-jobs/)

---

## Topics

### [[async-snapshots]] — Asynchronous Checkpointing

> *Persisting LLM training state without blocking GPU computation.*

- **Key Papers:** DataStates-LLM (HPDC '24), ByteCheckpoint (NSDI '25)
- **Implementations:** PyTorch DCP, DeepSpeed DataStates, Flash Checkpoint
- **Key Benefit:** 48× faster checkpointing, 2.2× faster end-to-end training
- **When:** Clusters >64 GPUs, checkpoint writes >30s, models >7B

### [[checkpoint-less-training]] — Recovery Without Checkpoints

> *Eliminating or reducing dependence on traditional full-model snapshots.*

- **Approaches:** Weighted averaging (CheckFree), replication (torchft), async islands (DiLoCo), redundant computation (Bamboo)
- **Key Papers:** CheckFree, FlashRecovery, FT-HSDP, Decoupled DiLoCo, Bamboo (NSDI '23), Oobleck (SOSP '23)
- **Key Benefit:** Zero checkpoint storage overhead, 88% goodput vs 27% for synchronous
- **When:** Geo-distributed training, spot/preemptible instances, massive scale

---

## Quick Reference

### Spectrum of Approaches

```
Traditional Checkpointing          Async Snapshots              Checkpoint-Less
         │                              │                            │
         ▼                              ▼                            ▼
  ┌─────────────┐              ┌─────────────────┐         ┌────────────────┐
  │ Periodic     │              │ Non-blocking    │         │ Replication    │
  │ Full Save   │      →       │ GPU→CPU→Disk    │    →    │ Weighted Avg   │
  │ (blocks)    │              │ (overlaps)      │         │ Async Islands  │
  └─────────────┘              └─────────────────┘         └────────────────┘
  
  Recovery Time: Minutes         Recovery Time: Seconds      Recovery Time: ~150s fixed
  Overhead: 5-10%               Overhead: <1%               Overhead: Near-zero
  Scale: Any                    Scale: 1856+ GPUs           Scale: 10,000+ devices
```

### Decision Matrix

| Scenario | Recommended Approach |
|----------|---------------------|
| Fixed infrastructure, infrequent failures | Traditional checkpoints |
| Cluster >64 GPUs, frequent failures | Async snapshots (PyTorch DCP) |
| Geo-distributed, limited bandwidth | CheckFree, Decoupled DiLoCo |
| Preemptible/spot instances | Bamboo, torchft |
| Massive scale (10,000+ devices) | FlashRecovery, FT-HSDP |
| Cost-sensitive training | Bamboo (2.4× cost reduction) |
| Heterogeneous hardware | Decoupled DiLoCo |

### Key Metrics

| Approach | Training Efficiency | Recovery Time | Checkpoint Overhead |
|----------|-------------------|---------------|-------------------|
| Traditional | Varies | Minutes | 5-10% |
| DataStates-LLM | 2.2× faster | Seconds | <1% |
| CheckFree | +12% wall-clock | Seconds | 0% |
| Decoupled DiLoCo | 88% vs 27% | Near-zero | 0% |
| torchft (Crusoe) | 81.2% | Seconds | 0% |

---

## See Also

- [[training-infrastructure]] — Parent topic
- [PyTorch Distributed Checkpoint Documentation](https://docs.pytorch.org/tutorials/recipes/distributed_async_checkpoint_recipe.html)
- [torchft Library](https://pytorch.org/torchft/)
- [Awesome-LLM-Training-System (InternLM)](https://github.com/InternLM/Awesome-LLM-Training-System)

---

*Last updated: 2026-05-02*
