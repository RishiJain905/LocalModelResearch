# Checkpoint-Less Training: A Survey of Recovery Techniques for LLM Training Infrastructure

> *Techniques that eliminate or reduce dependence on traditional full-model checkpoint snapshots for fault tolerance in large-scale LLM training.*

## Table of Contents

1. [Overview](#overview)
2. [Academic Papers](#academic-papers)
   - [Checkpoint-Less Recovery Methods](#checkpoint-less-recovery-methods)
   - [Deterministic Replay & Recovery](#deterministic-replay--recovery)
   - [Redundancy-Based Approaches](#redundancy-based-approaches)
3. [Blog Posts & Articles](#blog-posts--articles)
4. [Open Source Repositories](#open-source-repositories)
5. [Technical Reports & Documentation](#technical-reports--documentation)
6. [Talks & Videos](#talks--videos)
7. [Comparison Matrix](#comparison-matrix)
8. [When to Use Checkpoint-Less vs Traditional Checkpointing](#when-to-use-checkpoint-less-vs-traditional-checkpointing)

---

## Overview

Traditional checkpoint-based fault tolerance in LLM training suffers from:
- **High I/O overhead**: LLaMA 70B checkpoints require ~700GB storage and 20+ minutes at 500 Mb/s
- **Rollback waste**: ~50% of work between checkpoints must be re-computed per failure
- **Storage scaling**: Model sizes now demand hundreds of GB to TB of checkpoint storage
- **Trade-off paradox**: Higher checkpoint frequency reduces recovery time but increases steady-state overhead

Checkpoint-less approaches aim to eliminate or reduce this overhead through:
- **Weighted averaging of neighboring layers** (CheckFree)
- **Data parallel replica synchronization** (torchft, FT-HSDP)
- **Asynchronous training with self-healing** (DiLoCo, Decoupled DiLoCo)
- **Just-in-time failure detection** (FlashRecovery)
- **Deterministic replay** (DETrain)

---

## Academic Papers

### Checkpoint-Less Recovery Methods

---

#### 1. "All is Not Lost: LLM Recovery without Checkpoints"

| Field | Value |
|-------|-------|
| **arXiv** | [2506.15461](https://arxiv.org/abs/2506.15461) |
| **GitHub** | [gensyn-ai/CheckFree](https://github.com/gensyn-ai/CheckFree) |
| **Venue** | arXiv preprint (2025) |

**Key Concepts:**
> "We propose CheckFree, an efficient recovery method where a failing stage (in a pipeline) is substituted by weighted averaging of the closest neighboring stages."

> "CheckFree uses the weights of neighbouring stages to approximate the weights of the lost stage—when a fault occurs, the model is rolled back entirely to the previous checkpoint, thus losing potentially hours of training."

**Recovery Mechanism:**
```
W_i^S = (ω_{i-1} * W_{i-1}^S + ω_{i+1} * W_{i+1}^S) / (ω_{i-1} + ω_{i+1})
```
Weights based on gradient norms (higher weight to less-converged stages).

**Why Valuable:**
- ✅ Eliminates checkpoint storage overhead entirely for intermediate stages
- ✅ Up to 12% faster wall-clock time than redundant computation at 5-10% failure rates
- ✅ Zero additional computation or communication in non-failure cases

**CheckFree+ Extension:**
Uses out-of-order pipeline execution to enable recovery of first/last stages (which have only one neighbor). Every other batch swaps order of first two and last two stages, making edge stages partially learn their neighbor's behavior.

---

#### 2. "FlashRecovery: Fast and Low-Cost Recovery from Failures for Large-Scale Training of LLMs"

| Field | Value |
|-------|-------|
| **arXiv** | [2509.03047](https://arxiv.org/abs/2509.03047) |
| **Venue** | arXiv preprint (2025) |

**Key Concepts:**
> "By incorporating a novel active and real-time failure detection mechanism, along with a scale-independent task restart mechanism and checkpoint-free recovery within one step, FlashRecovery can reduce the failure recovery time overhead of a training cluster with thousands of devices to less than 150 seconds."

**Three Core Modules:**
1. **Active and Real-time Failure Detection** — Controller + monitoring processes + device plugins; failures detected within seconds vs. 30-minute hangs
2. **Scale-Independent Task Restart** — Only faulty nodes restart; parallelized TCP store establishment O(n/p)
3. **Checkpoint-Free Recovery within One Step** — Copy model state from normal DP group devices instead of loading checkpoints

**Why Valuable:**
- ✅ Achieves near-constant recovery time regardless of cluster scale (tested on 4,800 devices)
- ✅ Optimal RPO (Recovery Point Objective) limited to single-step loss
- ✅ Tested on 10,000+ Huawei Ascend NPUs with real failure workloads

---

#### 3. "Training LLMs with Fault Tolerant HSDP on 100,000 GPUs"

| Field | Value |
|-------|-------|
| **arXiv** | [2602.00277](https://arxiv.org/abs/2602.00277) |
| **Venue** | arXiv preprint (2026) |

**Key Concepts:**
> "FT-HSDP uses data parallel replicas as units of fault tolerance. When failures occur, only a single data-parallel replica containing the failed GPU or server is taken offline and restarted, while the other replicas continue training."

**Key Techniques:**
- Fault Tolerant All Reduce (FTAR) protocol for gradient exchange across data parallel replicas
- Non-blocking catch-up protocol allowing recovering replica to join with minimal stall

**Results:**
| Metric | Before (Synchronous) | FT-HSDP |
|--------|---------------------|---------|
| Stall time | 10 minutes | 3 minutes |
| Training efficiency | 44% | 80% |

**Why Valuable:**
- ✅ Scales to 100,000 GPUs with only ~3 minute stall time on failure
- ✅ Achieves 80% training efficiency at massive scale
- ✅ Reduces failure recovery overhead by 70%

---

#### 4. "Role-Based Fault Tolerance System for LLM RL Post-Training"

| Field | Value |
|-------|-------|
| **arXiv** | [2512.22492](https://arxiv.org/abs/2512.22492) |
| **Venue** | arXiv preprint (2025) |

**Key Concepts:**
> "Our key insight is role-based fault isolation so the failure in one machine does not affect the others. We treat trainer, rollout, and other components as different roles with independent checkpointing."

**Why Valuable:**
- ✅ First comprehensive robust system for GPU machine errors in RL post-training
- ✅ Achieves high Effective Training Time Ratio through role-based isolation
- ✅ Async RL training allows trainer checkpoint to only block briefly while rollouts continue

---

### Deterministic Replay & Recovery

---

#### 5. "Checkpointing and Deterministic Training for Deep Learning" (DETrain)

| Field | Value |
|-------|-------|
| **DOI** | [10.1145/3522664.3528605](https://dl.acm.org/doi/10.1145/3522664.3528605) |
| **GitHub** | [XZ-X/DETrain-public](https://github.com/XZ-X/DETrain-public) |

**Key Concepts:**
> "In this paper, we propose DETrain, a new solution to checkpointing and faithful execution/replay for long running DL training programs."

> "DETrain can deterministically execute these programs and replay from checkpoints with reasonable overhead."

**Mechanism:**
- Records epoch number for each training epoch
- Fast-forward replay: on resume, if epoch < saved_epoch, executes only data loading and skips loop body
- Achieves bit-identical results across runs

**Why Valuable:**
- ✅ Reproducible results across training runs (critical for debugging)
- ✅ Configurable checkpoint frequency
- ✅ Supports both TensorFlow and PyTorch models

---

#### 6. "Just-In-Time Checkpointing: Low Cost Error Recovery from Deep Learning Training Failures"

| Field | Value |
|-------|-------|
| **Venue** | EuroSys 2024 |
| **Microsoft Research** | [Link](https://www.microsoft.com/en-us/research/publication/just-in-time-checkpointing-low-cost-error-recovery-from-deep-learning-training-failures/) |

**Key Concepts:**
> "We present a novel approach of just-in-time checkpointing when failures happen, which enables recovery from failures with just a single minibatch iteration of work replayed by all GPUs."

> "This reduces the cost of error recovery from several minutes to a few seconds per GPU, with nearly zero steady state overhead."

**Why Valuable:**
- ✅ Near-zero steady state overhead (checkpoint only on failure)
- ✅ Recovers with single minibatch replay
- ✅ Eliminates guesswork of choosing checkpoint frequency

---

### Redundancy-Based Approaches

---

#### 7. "Bamboo: Making Preemptible Instances Resilient for Affordable Training of Large DNNs"

| Field | Value |
|-------|-------|
| **Venue** | NSDI 2023 |
| **PDF** | [web.cs.ucla.edu/~harryxu/papers/bamboo-nsdi23.pdf](https://web.cs.ucla.edu/~harryxu/papers/bamboo-nsdi23.pdf) |
| **Video** | [NSDI 2023](https://www.classcentral.com/course/youtube-nsdi-23-bamboo-making-preemptible-instances-resilient-for-affordable-training-of-large-dnns-184327) |

**Key Concepts:**
> "Bamboo is the first distributed system that uses redundant computation to provide resilience and fast recovery for training DNNs over preemptible instances."

> "Bamboo outperforms traditional checkpointing by 3.7× in training throughput, and reduces costs by 2.4× compared to a setting where on-demand instances are used."

**Mechanism:**
- Each node performs computations over its own layers AND some layers of its neighbor
- When preemption occurs, neighbor has required state to continue
- Redundancy is proactive (constant) rather than reactive (on-failure)

**Why Valuable:**
- ✅ 3.7× training throughput improvement over traditional checkpointing
- ✅ 2.4× cost reduction vs. on-demand instances
- ✅ First system to enable efficient training on preemptible/cloud spot instances

---

#### 8. "DiLoCo: Distributed Low-Communication Training of Language Models"

| Field | Value |
|-------|-------|
| **arXiv** | [2311.08105](https://arxiv.org/abs/2311.08105) |

**Key Concepts:**
> "In this work, we propose a distributed optimization algorithm, Distributed Low-Communication (DiLoCo), that enables training of language models across poorly connected devices with 500× less communication."

**Architecture:**
- Outer optimizer (SGD with Nesterov) + Inner optimizers (local AdamW)
- Enables training across datacenters with limited bandwidth
- Reduces communication requirements by 500×

**Why Valuable:**
- ✅ Foundation for globally distributed, fault-tolerant training
- ✅ 500× communication reduction enables cross-datacenter training
- ✅ Foundation for Decoupled DiLoCo and OpenDiLoCo

---

#### 9. "Decoupled DiLoCo: Resilient, Distributed AI Training at Scale"

| Field | Value |
|-------|-------|
| **Blog** | [Google DeepMind](https://deepmind.google/blog/decoupled-diloco/) |
| **arXiv** | [2604.21428v1](https://arxiv.org/abs/2604.21428v1) |

**Key Concepts:**
> "Decoupled DiLoCo replaces the tightly coupled Single Program Multiple Data (SPMD) paradigm with a fully asynchronous architecture. By dividing compute into independent 'learners' that communicate parameter fragments to a central, CPU-based synchronizer via a minimum-quorum and adaptive grace-window system, the framework isolates hardware failures."

**Results (Gemma 4, 12B parameter model across 4 U.S. regions):**
| Metric | Data-Parallel | Decoupled DiLoCo |
|--------|---------------|------------------|
| Goodput | 27% | **88%** |
| Required Bandwidth | 198 Gbps | **0.84 Gbps** |

**Why Valuable:**
- ✅ Self-healing: system persists after losing entire learner units
- ✅ Successfully trained across 4 regions with 2-5 Gbps wide-area networking
- ✅ 88% goodput vs 27% for synchronous approaches at massive scale

---

### Pipeline-Based Fault Tolerance

---

#### 10. "Oobleck: Resilient Distributed Training of Large Models Using Pipeline Templates"

| Field | Value |
|-------|-------|
| **arXiv** | [2309.08125](https://arxiv.org/abs/2309.08125) |
| **Video** | [SOSP '23](https://www.youtube.com/watch?v=-Q9MSZ4351A) |
| **Venue** | SOSP 2023 |

**Key Concepts:**
> "Oobleck enables resilient distributed training of large DNN models with guaranteed fault tolerance. It takes a planning-execution co-design approach."

**Mechanism:**
- Pipeline parallelism with templates that can be rebuilt on failure
- Planning-execution co-design for fault tolerance
- Guaranteed fault tolerance through pipeline reconstruction

**Why Valuable:**
- ✅ First system with guaranteed fault tolerance for large DNN training
- ✅ Video presentation available (SOSP '23)
- ✅ Template-based approach enables flexible recovery strategies

---

## Blog Posts & Articles

---

#### 1. "CheckFree: fault tolerant training without checkpoints"

| Field | Value |
|-------|-------|
| **URL** | [blog.gensyn.ai/checkfree-fault-tolerant-training-without-checkpoints/](https://blog.gensyn.ai/checkfree-fault-tolerant-training-without-checkpoints/) |
| **Organization** | Gensyn |

**Key Quotes:**
> "CheckFree and CheckFree+ provide a viable alternative in large scale geo-distributed training, as it incurs no additional computation or communication."

> "The conventional approaches to recover from failures is to either use checkpointing, where periodically a copy of the entire model is sent to an additional storage, or redundant computation."

**Why Valuable:**
- ✅ Clear explanation of when checkpoint-less makes sense (geo-distributed, spot instances)
- ✅ Practical comparison table: Checkpoint vs Redundant Comp vs CheckFree vs CheckFree+

---

#### 2. "LLM Recovery without Checkpoints"

| Field | Value |
|-------|-------|
| **URL** | [medium.com/@bnjmn_marie/llm-recovery-without-checkpoints-97a177c13f61](https://medium.com/@bnjmn_marie/llm-recovery-without-checkpoints-97a177c13f61) |
| **Author** | Benjamin Marie |

**Key Quotes:**
> "The authors propose CheckFree, a lightweight recovery method that doesn't require checkpointing or redundant execution."

> "CheckFree significantly reduces recovery overhead and improves training efficiency, cutting down training time by up to 12% compared to redundant computation."

**Why Valuable:**
- ✅ Accessible explanation for ML practitioners
- ✅ Context on paper significance and real-world applicability

---

#### 3. "Fault Tolerant Llama: training with 2000 synthetic failures every 15 seconds and no checkpoints"

| Field | Value |
|-------|-------|
| **URL** | [pytorch.org/blog/fault-tolerant-llama-training-with-2000-synthetic-failures-every-15-seconds-and-no-checkpoints-on-crusoe-l40s/](https://pytorch.org/blog/fault-tolerant-llama-training-with-2000-synthetic-failures-every-15-seconds-and-no-checkpoints-on-crusoe-l40s/) |
| **Authors** | Less Wright, Howard Huang, Chien-Chin Huang (PyTorch/Crusoe) |

**Key Quotes:**
> "We actually don't need any checkpoints as long as we can guarantee that at least one group is healthy."

> "We averaged 29.6 participating replica groups per step, so the total training efficiency of this was 81.2%."

**Technical Details:**
- 300 NVIDIA L40S GPUs, 30 hosts × 10 GPUs
- Llama 3 1B parameters
- torchft + torchtitan framework
- 60-second MTBF: 81.2% training efficiency
- 15-second MTBF (extreme): 1,015 failures, model STILL converges

**Why Valuable:**
- ✅ Real-world production validation with extreme failure rates
- ✅ Demonstrates torchft's async recovery capabilities
- ✅ Open source implementation via torchtitan

---

#### 4. "OpenDiLoCo and Distributed Training"

| Field | Value |
|-------|-------|
| **URL** | [drli.blog/posts/opendiloco-distributed-training/](https://drli.blog/posts/opendiloco-distributed-training/) |

**Key Quotes:**
> "OpenDiLoCo is an open-source framework designed to implement the Distributed Low-Communication (DiLoCo) training method."

> "This dual optimization approach allows DiLoCo to reduce communication requirements by up to 500 times."

**Why Valuable:**
- ✅ Clear explanation of DiLoCo's dual optimizer design
- ✅ Context on OpenDiLoCo open-source implementation

---

#### 5. "Decoupled DiLoCo: Resilient, Distributed AI Training at Scale"

| Field | Value |
|-------|-------|
| **URL** | [deepmind.google/blog/decoupled-diloco/](https://deepmind.google/blog/decoupled-diloco/) |

**Key Quotes:**
> "By treating pre-training as a distributed systems problem—prioritizing availability and partition tolerance over strict parameter consistency—this method maintains zero global downtime and near-optimal goodput under massive simulated hardware failures."

**Why Valuable:**
- ✅ Production implementation details from Google DeepMind
- ✅ CAP theorem analogy for training vs. traditional distributed systems

---

## Open Source Repositories

---

#### 1. gensyn-ai/CheckFree

| Field | Value |
|-------|-------|
| **URL** | [github.com/gensyn-ai/CheckFree](https://github.com/gensyn-ai/CheckFree) |
| **License** | MIT |

**Description:**
Official implementation of "All is not Lost: LLM Recovery without Checkpoints" — checkpoint-less recovery via weighted averaging of neighboring stages.

**Key Resources:**
- `simulate_training/` — Simulated fault training experiments
- `trainer.py` — Training logic with configurable recovery strategies
- Failure probability configs for various scenarios (124M, 500M, 1.5B models)

**Why Valuable:**
- ✅ Direct implementation of checkpoint-less weighted averaging
- ✅ Configurable failure rates for simulation
- ✅ MIT license for integration

---

#### 2. meta-pytorch/torchft

| Field | Value |
|-------|-------|
| **URL** | [github.com/meta-pytorch/torchft](https://github.com/meta-pytorch/torchft) |
| **Documentation** | [pytorch.org/torchft/](https://pytorch.org/torchft/) |

**Description:**
Meta's fault tolerance library for PyTorch enabling per-step recovery without interrupting entire training jobs.

**Supported Algorithms:**
- HSDP (Hierarchical Stochastic Data Parallel)
- LocalSGD / DiLoCo
- Streaming DiLoCo

**Key Components:**
- Lighthouse server for cross-replica coordination
- Reconfigurable ProcessGroups for membership changes at step granularity
- `train_ddp.py`, `train_diloco.py` examples

**Why Valuable:**
- ✅ Production-grade from Meta/PyTorch team
- ✅ Integrates with torchtitan for Llama training
- ✅ BSD 3-Clause license

---

#### 3. pytorch/torchtitan

| Field | Value |
|-------|-------|
| **URL** | [github.com/pytorch/torchtitan](https://github.com/pytorch/torchtitan) |

**Description:**
PyTorch native platform for large-scale LLM training with built-in fault tolerance via torchft integration.

**Features:**
- FSDP2, HSDP2, TP, PP support
- Fault tolerant HSDP training loop
- Context parallelism for 1M sequence length

**Why Valuable:**
- ✅ Production demonstration of torchft (Llama 3 70B trained with fault tolerance)
- ✅ Clean, modular architecture for understanding
- ✅ Active development by PyTorch team

---

#### 4. PrimeIntellect-ai/OpenDiloco

| Field | Value |
|-------|-------|
| **URL** | [github.com/PrimeIntellect-ai/OpenDiloco](https://github.com/PrimeIntellect-ai/OpenDiloco) |

**Description:**
Open-source implementation and scaling of DeepMind's DiLoCo method, enabling globally distributed low-communication training.

**Key Features:**
- `DiLoCoOptimizer` as near drop-in replacement for `hivemind.optim.optimizer`
- 500× communication reduction demonstrated
- Scaled to 3× original DiLoCo parameter size

**Why Valuable:**
- ✅ First open-source implementation of DiLoCo
- ✅ Enables globally distributed training on heterogeneous hardware
- ✅ Well-documented with experiment scripts

---

#### 5. XZ-X/DETrain-public

| Field | Value |
|-------|-------|
| **URL** | [github.com/XZ-X/DETrain-public](https://github.com/XZ-X/DETrain-public) |

**Description:**
Deterministic training and replayable checkpointing for DL programs. Based on Purdue University research.

**Features:**
- Modified TensorFlow for deterministic execution
- Dynamic analysis for automatic program structure discovery
- Configurable checkpoint frequency

**Why Valuable:**
- ✅ Enables bit-identical replay for debugging/certification
- ✅ Automatic loop detection and fast-forward
- ✅ Foundation for deterministic training research

---

#### 6. InternLM/Awesome-LLM-Training-System

| Field | Value |
|-------|-------|
| **URL** | [github.com/InternLM/Awesome-LLM-Training-System](https://github.com/InternLM/Awesome-LLM-Training-System) |
| **Fault Tolerance Doc** | [Fault_Tolerance.md](https://github.com/InternLM/Awesome-LLM-Training-System/blob/main/Fault_Tolerance.md) |

**Description:**
Comprehensive survey of LLM training system optimizations including fault tolerance mechanisms.

**Why Valuable:**
- ✅ Excellent literature survey with categorization
- ✅ Links to 20+ fault tolerance papers with venues
- ✅ Covers both checkpointing and checkpoint-less approaches

---

## Technical Reports & Documentation

---

#### 1. torchft Design Document

| Field | Value |
|-------|-------|
| **URL** | [Design Doc](https://docs.google.com/document/d/1OZsOsz34gRDSxYXiKkj4WqcD9x0lP9TcsfBeu_SsOY4/edit) |

**Key Concepts:**
- Per-step fault tolerance architecture
- Lighthouse coordination protocol
- Membership changes at training step granularity

---

#### 2. "Fault Tolerance in LLM Training Systems" (InternLM Survey)

| Field | Value |
|-------|-------|
| **URL** | [github.com/InternLM/Awesome-LLM-Training-System/blob/main/Fault_Tolerance.md](https://github.com/InternLM/Awesome-LLM-Training-System/blob/main/Fault_Tolerance.md) |

**Categories Covered:**
1. **Anomaly Detection** — Statistical monitoring, proactive validation
2. **Checkpointing-Based Recovery** — Persistent/in-memory checkpointing
3. **Checkpoint-Free Recovery** — Live migration, module redundancy

**Key Papers Cataloged:**
- MegaScale (NSDI 24)
- MegaScale (NSDI 24)
- Varuna (EuroSys 22)
- Just-in-time Checkpointing (EuroSys 24)
- Bamboo (NSDI 23)
- Oobleck (SOSP 23)
- GEMINI (SOSP 23)
- And 15+ more

---

## Talks & Videos

---

#### 1. "Oobleck: Resilient Distributed Training of Large Models Using Pipeline Templates"

| Field | Value |
|-------|-------|
| **URL** | [youtube.com/watch?v=-Q9MSZ4351A](https://www.youtube.com/watch?v=-Q9MSZ4351A) |
| **Venue** | SOSP 2023 |

**Why Valuable:**
- ✅ Complete talk on pipeline template-based fault tolerance
- ✅ Planning-execution co-design explanation
- ✅ SOSP archival quality

---

#### 2. "Bamboo: Making Preemptible Instances Resilient for Affordable Training of Large DNNs"

| Field | Value |
|-------|-------|
| **URL** | [NSDI 2023](https://www.classcentral.com/course/youtube-nsdi-23-bamboo-making-preemptible-instances-resilient-for-affordable-training-of-large-dnns-184327) |
| **Venue** | NSDI 2023 |

**Why Valuable:**
- ✅ NSDI formal presentation with slides
- ✅ Redundant computation approach (contrasts with CheckFree)
- ✅ Cost-performance analysis

---

#### 3. "Handling Multi-Terabyte LLM Checkpoints"

| Field | Value |
|-------|-------|
| **URL** | [youtube.com/watch?v=6MY-IgqiTpg](https://www.youtube.com/watch?v=6MY-IgqiTpg) |
| **Venue** | MLOps Community |

**Why Valuable:**
- ✅ Practical perspective on checkpoint challenges
- ✅ Storage optimization strategies
- ✅ Real-world production experience at Nebius AI

---

#### 4. "AI Explained: Checkpointing in LLMs and the Trade-Offs between Reliability and Performance"

| Field | Value |
|-------|-------|
| **URL** | [weka.io/resources/video/ai-demystified-episode-03-ai-explained-checkpointing-in-llms-and-the-trade-offs-between-reliability-and-performance/](https://www.weka.io/resources/video/ai-demystified-episode-03-ai-explained-checkpointing-in-llms-and-the-trade-offs-between-reliability-and-performance/) |

**Why Valuable:**
- ✅ Accessible introduction to checkpoint trade-offs
- ✅ Storage layer optimization context

---

## Comparison Matrix

| Approach | Checkpoint Overhead | Recovery Time | Communication | Computation Overhead | Best For |
|----------|-------------------|---------------|---------------|---------------------|----------|
| **Traditional Checkpointing** | High (periodic) | Minutes | O(model size) | None | Fixed infrastructure, infrequent failures |
| **Bamboo (Redundant)** | None | Seconds | None | 2× | Preemptible/spot instances, cost-sensitive |
| **CheckFree** | None | Seconds | None | None | Geo-distributed, low-to-medium failure rates |
| **FlashRecovery** | None | <150s fixed | None | None | Massive clusters (4,800+ devices) |
| **torchft/HSDP** | None | Seconds (async) | Gradients | None | Production scale, multiple replica groups |
| **Decoupled DiLoCo** | None | Near-zero | 0.84 Gbps (vs 198 Gbps) | None | Global distribution, heterogeneous hardware |
| **Oobleck** | Pipeline templates | Rebuild | Pipeline rebuild | Pipeline rebuild | Large models with pipeline parallelism |
| **Just-In-Time Checkpoint** | None (until failure) | Single minibatch | Minimal | None | Unpredictable failure patterns |

---

## When to Use Checkpoint-Less vs Traditional Checkpointing

### Use Checkpoint-Less When:

| Scenario | Recommended Approach |
|----------|---------------------|
| **Geo-distributed training** (across datacenters) | CheckFree, Decoupled DiLoCo |
| **Preemptible/spot instances** | Bamboo, torchft |
| **Low-to-medium failure rates** (5-10%) | CheckFree, CheckFree+ |
| **Massive scale** (10,000+ devices) | FlashRecovery, FT-HSDP, Decoupled DiLoCo |
| **Heterogeneous hardware** | Decoupled DiLoCo, OpenDiLoCo |
| **Asynchronous training acceptable** | Decoupled DiLoCo |
| **Cost-sensitive training** | Bamboo (reduces cost 2.4×) |

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

1. Geng et al. "All is Not Lost: LLM Recovery without Checkpoints" arXiv:2506.15461 (2025)
2. Zhang et al. "FlashRecovery: Fast and Low-Cost Recovery from Failures for Large-Scale Training of LLMs" arXiv:2509.03047 (2025)
3. Thorpe et al. "Bamboo: Making Preemptible Instances Resilient for Affordable Training of Large DNNs" NSDI 2023
4. Douillard et al. "DiLoCo: Distributed Low-Communication Training of Language Models" arXiv:2311.08105 (2023)
5. Google DeepMind. "Decoupled DiLoCo: Resilient, Distributed AI Training at Scale" (2026)
6. Gupta et al. "Just-In-Time Checkpointing" EuroSys 2024
7. Jang et al. "Oobleck: Resilient Distributed Training of Large Models Using Pipeline Templates" SOSP 2023
8. "Training LLMs with Fault Tolerant HSDP on 100,000 GPUs" arXiv:2602.00277 (2026)
9. "Role-Based Fault Tolerance System for LLM RL Post-Training" arXiv:2512.22492 (2025)
10. Liu et al. "Checkpointing and Deterministic Training for Deep Learning" DOI:10.1145/3522664.3528605 (2022)

---

*Last updated: 2026-05-02*
