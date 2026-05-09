# Checkpoint-Less Training

> *Techniques that eliminate or reduce dependence on traditional full-model checkpoint snapshots for fault tolerance in large-scale LLM training.*

## Table of Contents

1. [Overview](#overview)
2. [Recovery Mechanisms](#recovery-mechanisms)
3. [Key Papers](#key-papers)
4. [Open Source Implementations](#open-source-implementations)
5. [Comparison Matrix](#comparison-matrix)
6. [When to Use Checkpoint-Less](#when-to-use-checkpoint-less)
7. [References](#references)

---

## Overview

Traditional checkpoint-based fault tolerance suffers from:

> *"High I/O overhead: LLaMA 70B checkpoints require ~700GB storage and 20+ minutes at 500 Mb/s. Rollback waste: ~50% of work between checkpoints must be re-computed per failure."*

Checkpoint-less approaches eliminate or reduce this overhead through:

| Mechanism | Approach | Key Benefit |
|-----------|----------|-------------|
| **CheckFree** | Weighted averaging of neighboring layers | Zero storage overhead |
| **torchft/HSDP** | Data parallel replica synchronization | Per-step recovery |
| **DiLoCo** | Async island-based training | 500× communication reduction |
| **Bamboo** | Redundant computation | 2.4× cost reduction |
| **FlashRecovery** | Active failure detection + replication | <150s recovery at 4,800+ devices |

---

## Recovery Mechanisms

### Weighted Averaging (CheckFree)

> *"We propose CheckFree, an efficient recovery method where a failing stage (in a pipeline) is substituted by weighted averaging of the closest neighboring stages."*

**Formula:**
```
W_i^S = (ω_{i-1} * W_{i-1}^S + ω_{i+1} * W_{i+1}^S) / (ω_{i-1} + ω_{i+1})
```

Weights based on gradient norms (higher weight to less-converged stages).

**Extensions:**
- **CheckFree+** — Out-of-order pipeline execution enables recovery of first/last stages (which have only one neighbor)

---

### Data Parallel Replication (torchft)

> *"We actually don't need any checkpoints as long as we can guarantee that at least one group is healthy."*

From [PyTorch Blog: Fault Tolerant Llama Training](https://pytorch.org/blog/fault-tolerant-llama-training-with-2000-synthetic-failures-every-15-seconds-and-no-checkpoints-on-crusoe-l40s/):

- HSDP (Hierarchical Stochastic Data Parallel) uses replica groups for fault tolerance
- Per-step recovery without interrupting entire training jobs
- 60-second MTBF: 81.2% training efficiency
- 15-second MTBF (extreme): 1,015 failures, model STILL converges

---

### Async Islands (DiLoCo / Decoupled DiLoCo)

> *"By treating pre-training as a distributed systems problem—prioritizing availability and partition tolerance over strict parameter consistency—this method maintains zero global downtime and near-optimal goodput under massive simulated hardware failures."*

From [Google DeepMind: Decoupled DiLoCo](https://deepmind.google/blog/decoupled-diloco/):

- Outer optimizer (SGD with Nesterov) + Inner optimizers (local AdamW)
- Divides compute into independent "learners" that communicate via CPU-based synchronizer
- Self-healing: persists after losing entire learner units

---

## Key Papers

### CheckFree: "All is Not Lost: LLM Recovery without Checkpoints"

**Paper:** [arXiv:2506.15461](https://arxiv.org/abs/2506.15461)  
**Repository:** [gensyn-ai/CheckFree](https://github.com/gensyn-ai/CheckFree)

> *"CheckFree uses the weights of neighbouring stages to approximate the weights of the lost stage—when a fault occurs, the model is rolled back entirely to the previous checkpoint, thus losing potentially hours of training."*

**Results:**
| Scenario | Improvement vs Redundant Computation |
|----------|-------------------------------------|
| Wall-clock time | Up to 12% faster |
| Failure rates | 5-10% optimal |

**Key Properties:**
- ✅ Zero additional computation in non-failure cases
- ✅ Zero storage overhead for intermediate stages
- ✅ No consecutive failure handling (limitation)

---

### FlashRecovery: Scale-Independent Recovery

**Paper:** [arXiv:2509.03047](https://arxiv.org/abs/2509.03047)

> *"By incorporating a novel active and real-time failure detection mechanism, along with a scale-independent task restart mechanism and checkpoint-free recovery within one step, FlashRecovery can reduce the failure recovery time overhead of a training cluster with thousands of devices to less than 150 seconds."*

**Three Core Modules:**
1. **Active Failure Detection** — Controller + monitoring processes; seconds vs 30-minute hangs
2. **Scale-Independent Restart** — Only faulty nodes restart; O(n/p) TCP store establishment
3. **Checkpoint-Free Recovery** — Copy model state from normal DP group devices

**Results at 4,800+ Devices:**
- **~150 second recovery time** (nearly scale-independent)
- Tested on 10,000+ Huawei Ascend NPUs

---

### FT-HSDP: 100,000 GPUs

**Paper:** [arXiv:2602.00277](https://arxiv.org/abs/2602.00277)

> *"FT-HSDP uses data parallel replicas as units of fault tolerance. When failures occur, only a single data-parallel replica containing the failed GPU or server is taken offline and restarted, while the other replicas continue training."*

**Results:**
| Metric | Before (Synchronous) | FT-HSDP |
|--------|---------------------|---------|
| Stall time | 10 minutes | 3 minutes |
| Training efficiency | 44% | 80% |

---

### Bamboo: Preemptible Instances

**Paper:** [NSDI 2023](https://web.cs.ucla.edu/~harryxu/papers/bamboo-nsdi23.pdf)  
**Video:** [NSDI '23](https://www.classcentral.com/course/youtube-nsdi-23-bamboo-making-preemptible-instances-resilient-for-affordable-training-of-large-dnns-184327)

> *"Bamboo is the first distributed system that uses redundant computation to provide resilience and fast recovery for training DNNs over preemptible instances."*

**Results:**
- **3.7× training throughput** improvement over traditional checkpointing
- **2.4× cost reduction** vs on-demand instances

---

### DiLoCo & Decoupled DiLoCo

**DiLoCo Paper:** [arXiv:2311.08105](https://arxiv.org/abs/2311.08105)

> *"In this work, we propose a distributed optimization algorithm, Distributed Low-Communication (DiLoCo), that enables training of language models across poorly connected devices with 500× less communication."*

**Decoupled DiLoCo:** [Google DeepMind Blog](https://deepmind.google/blog/decoupled-diloco/)

**Gemma 4 Results (12B params, 4 U.S. regions):**
| Metric | Data-Parallel | Decoupled DiLoCo |
|--------|---------------|------------------|
| Goodput | 27% | **88%** |
| Bandwidth | 198 Gbps | **0.84 Gbps** |

---

### Oobleck: Pipeline Templates

**Paper:** [arXiv:2309.08125](https://arxiv.org/abs/2309.08125)  
**Video:** [SOSP '23](https://www.youtube.com/watch?v=-Q9MSZ4351A)

> *"Oobleck enables resilient distributed training of large DNN models with guaranteed fault tolerance. It takes a planning-execution co-design approach."*

- Pipeline parallelism with templates that can be rebuilt on failure
- First system with **guaranteed fault tolerance** for large DNN training

---

### Just-In-Time Checkpointing (EuroSys '24)

**Paper:** [Microsoft Research](https://www.microsoft.com/en-us/research/publication/just-in-time-checkpointing-low-cost-error-recovery-from-deep-learning-training-failures/)

> *"We present a novel approach of just-in-time checkpointing when failures happen, which enables recovery from failures with just a single minibatch iteration of work replayed by all GPUs."*

- **Near-zero steady state overhead** (checkpoint only on failure)
- Recovers with single minibatch replay
- Eliminates guesswork of choosing checkpoint frequency

---

## Open Source Implementations

### torchft (Meta/PyTorch)

**Repository:** [meta-pytorch/torchft](https://github.com/meta-pytorch/torchft)  
**Documentation:** [pytorch.org/torchft/](https://pytorch.org/torchft/)

> *"Meta's fault tolerance library for PyTorch enabling per-step recovery without interrupting entire training jobs."*

**Supported Algorithms:**
- HSDP (Hierarchical Stochastic Data Parallel)
- LocalSGD / DiLoCo
- Streaming DiLoCo

**Key Components:**
- Lighthouse server for cross-replica coordination
- Reconfigurable ProcessGroups for membership changes at step granularity

**Production Validation:**
- 300 NVIDIA L40S GPUs, 30 hosts × 10 GPUs
- Llama 3 1B parameters
- 60-second MTBF: 81.2% training efficiency
- 15-second MTBF: 1,015 failures, model STILL converges

---

### CheckFree

**Repository:** [gensyn-ai/CheckFree](https://github.com/gensyn-ai/CheckFree) (MIT License)

**Key Resources:**
- `simulate_training/` — Simulated fault training experiments
- `trainer.py` — Training logic with configurable recovery strategies
- Failure probability configs for various scenarios (124M, 500M, 1.5B models)

---

### OpenDiLoCo

**Repository:** [PrimeIntellect-ai/OpenDiloco](https://github.com/PrimeIntellect-ai/OpenDiloco)

> *"Open-source implementation and scaling of DeepMind's DiLoCo method."*

- `DiLoCoOptimizer` as near drop-in replacement
- 500× communication reduction demonstrated
- Scaled to 3× original DiLoCo parameter size

---

### DETrain (Deterministic Replay)

**Repository:** [XZ-X/DETrain-public](https://github.com/XZ-X/DETrain-public)

> *"DETrain can deterministically execute these programs and replay from checkpoints with reasonable overhead."*

- Bit-identical results across runs
- Fast-forward replay on resume
- Supports TensorFlow and PyTorch

---

### Awesome-LLM-Training-System (InternLM)

**Repository:** [InternLM/Awesome-LLM-Training-System](https://github.com/InternLM/Awesome-LLM-Training-System)  
**Fault Tolerance Doc:** [Fault_Tolerance.md](https://github.com/InternLM/Awesome-LLM-Training-System/blob/main/Fault_Tolerance.md)

Comprehensive survey covering:
- 20+ fault tolerance papers with venues
- Anomaly Detection, Checkpointing-Based Recovery, Checkpoint-Free Recovery categories

---

## Comparison Matrix

| Approach | Checkpoint Overhead | Recovery Time | Communication | Computation | Best For |
|----------|-------------------|---------------|---------------|-------------|----------|
| **Traditional Checkpointing** | High (periodic) | Minutes | O(model size) | None | Fixed infrastructure, infrequent failures |
| **Bamboo (Redundant)** | None | Seconds | None | 2× | Preemptible/spot instances |
| **CheckFree** | None | Seconds | None | None | Geo-distributed, low failure rates |
| **FlashRecovery** | None | <150s fixed | None | None | Massive clusters (4,800+ devices) |
| **torchft/HSDP** | None | Seconds (async) | Gradients | None | Production scale, replica groups |
| **Decoupled DiLoCo** | None | Near-zero | 0.84 Gbps | None | Global distribution |
| **Oobleck** | Templates | Rebuild | Pipeline rebuild | Pipeline rebuild | Large models, pipeline parallelism |
| **Just-In-Time Checkpoint** | None (until failure) | Single minibatch | Minimal | None | Unpredictable failure patterns |

---

## When to Use Checkpoint-Less

### Use Checkpoint-Less When:

| Scenario | Recommended Approach |
|----------|---------------------|
| **Geo-distributed training** (across datacenters) | CheckFree, Decoupled DiLoCo |
| **Preemptible/spot instances** | Bamboo, torchft |
| **Low-to-medium failure rates** (5-10%) | CheckFree, CheckFree+ |
| **Massive scale** (10,000+ devices) | FlashRecovery, FT-HSDP, Decoupled DiLoCo |
| **Heterogeneous hardware** | Decoupled DiLoCo, OpenDiLoCo |
| **Asynchronous training acceptable** | Decoupled DiLoCo |
| **Cost-sensitive training** | Bamboo (2.4× cost reduction) |

### Use Traditional Checkpointing When:

| Scenario | Rationale |
|----------|-----------|
| **Strict consistency required** | Checkpoint provides exact recovery point |
| **Infrequent failures** | Overhead of checkpoint-less infrastructure not justified |
| **Small models** | Checkpoint storage/overhead negligible |
| **Regulatory requirements** | Audit trail from checkpoints |
| **Debugging/deterministic replay** | DETrain-style deterministic checkpointing |
| **Consecutive failure risk** | CheckFree cannot handle consecutive stage failures |

---

## Key Takeaways

1. **Paradigm Shift**: Checkpoint-less training is moving from theoretical to production-ready (torchft, CheckFree, Decoupled DiLoCo all have implementations)

2. **Multiple Mechanisms**:
   - Weighted averaging (CheckFree)
   - Data parallel replication (torchft, FT-HSDP)
   - Asynchronous islands (DiLoCo, Decoupled DiLoCo)
   - Redundant computation (Bamboo)

3. **Performance Gains**: Up to 12% wall-clock improvement (CheckFree), 88% goodput vs 27% (Decoupled DiLoCo), 81.2% training efficiency with 1,000+ failures (torchft/Crusoe)

4. **Trade-offs**: No single solution—choice depends on failure patterns, scale, hardware heterogeneity, and consistency requirements

5. **Emerging Pattern**: Async training with replica groups becoming dominant paradigm for fault tolerance at scale

---

## References

### Papers
1. Geng et al. ["All is Not Lost: LLM Recovery without Checkpoints"](https://arxiv.org/abs/2506.15461) — arXiv:2506.15461 (2025)
2. FlashRecovery Authors. ["FlashRecovery: Fast and Low-Cost Recovery from Failures for Large-Scale Training of LLMs"](https://arxiv.org/abs/2509.03047) — arXiv:2509.03047 (2025)
3. Thorpe et al. ["Bamboo: Making Preemptible Instances Resilient for Affordable Training of Large DNNs"](https://web.cs.ucla.edu/~harryxu/papers/bamboo-nsdi23.pdf) — NSDI 2023
4. Douillard et al. ["DiLoCo: Distributed Low-Communication Training of Language Models"](https://arxiv.org/abs/2311.08105) — arXiv:2311.08105 (2023)
5. Google DeepMind. ["Decoupled DiLoCo: Resilient, Distributed AI Training at Scale"](https://deepmind.google/blog/decoupled-diloco/) (2026)
6. Gupta et al. ["Just-In-Time Checkpointing"](https://www.microsoft.com/en-us/research/publication/just-in-time-checkpointing-low-cost-error-recovery-from-deep-learning-training-failures/) — EuroSys 2024
7. Jang et al. ["Oobleck: Resilient Distributed Training of Large Models Using Pipeline Templates"](https://arxiv.org/abs/2309.08125) — SOSP 2023
8. ["Training LLMs with Fault Tolerant HSDP on 100,000 GPUs"](https://arxiv.org/abs/2602.00277) — arXiv:2602.00277 (2026)
9. ["Role-Based Fault Tolerance System for LLM RL Post-Training"](https://arxiv.org/abs/2512.22492) — arXiv:2512.22492 (2025)
10. Liu et al. ["Checkpointing and Deterministic Training for Deep Learning"](https://dl.acm.org/doi/10.1145/3522664.3528605) — DOI:10.1145/3522664.3528605 (2022)

### Blogs & Articles
11. Gensyn. ["CheckFree: fault tolerant training without checkpoints"](https://blog.gensyn.ai/checkfree-fault-tolerant-training-without-checkpoints/)
12. Marie, Benjamin. ["LLM Recovery without Checkpoints"](https://medium.com/@bnjmn_marie/llm-recovery-without-checkpoints-97a177c13f61)
13. Wright et al. ["Fault Tolerant Llama: training with 2000 synthetic failures every 15 seconds and no checkpoints"](https://pytorch.org/blog/fault-tolerant-llama-training-with-2000-synthetic-failures-every-15-seconds-and-no-checkpoints-on-crusoe-l40s/) — PyTorch Blog
14. Li, D. ["OpenDiLoCo and Distributed Training"](https://drli.blog/posts/opendiloco-distributed-training/)

### Repositories
15. [gensyn-ai/CheckFree](https://github.com/gensyn-ai/CheckFree)
16. [meta-pytorch/torchft](https://github.com/meta-pytorch/torchft)
17. [pytorch/torchtitan](https://github.com/pytorch/torchtitan)
18. [PrimeIntellect-ai/OpenDiloco](https://github.com/PrimeIntellect-ai/OpenDiloco)
19. [XZ-X/DETrain-public](https://github.com/XZ-X/DETrain-public)
20. [InternLM/Awesome-LLM-Training-System](https://github.com/InternLM/Awesome-LLM-Training-System)

### Talks & Videos
21. ["Oobleck: Resilient Distributed Training of Large Models Using Pipeline Templates"](https://www.youtube.com/watch?v=-Q9MSZ4351A) — SOSP 2023
22. ["Bamboo: Making Preemptible Instances Resilient"](https://www.classcentral.com/course/youtube-nsdi-23-bamboo-making-preemptible-instances-resilient-for-affordable-training-of-large-dnns-184327) — NSDI 2023

---

*Part of [[checkpointing-recovery]] — Training Infrastructure*
