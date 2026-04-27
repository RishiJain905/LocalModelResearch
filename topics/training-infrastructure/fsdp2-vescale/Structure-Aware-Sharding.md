# Structure-Aware Sharding

> **📌 Overview**
> **Structure-Aware Sharding** is the combination of (1) **RaggedShard** — a flexible DTensor `Placement` for arbitrary/uneven sharding granularity — and (2) a **structure-aware planning algorithm** — an NP-hard optimization solved via polynomial-time dynamic programming. Together in the **veScale-FSDP** system (ByteDance Seed, arXiv:2602.22437) they enable 5–66% throughput gains and 16–30% memory reduction over existing FSDP systems, natively supporting block-wise quantization, matrix optimizers (Muon, Shampoo), and MoE models. See also: [Muon-Optimizers](./Muon-Optimizers.md) — the optimizer that motivated structure-aware sharding design.

---

## Definitions & Core Concepts

### Core Problem: Rigid Sharding Conflicts with Modern Training

> *"During parallelization, complex model and optimizer operators can still preserve single-device semantics through a new sharding format, RaggedShard, which supports arbitrary sharding granularities over contiguous storage and arbitrary distributions across devices for each DTensor."*

**Source:** [arXiv:2602.22437v3 — §1, §5](https://arxiv.org/html/2602.22437v3)

Existing FSDP systems (DeepSpeed ZeRO, PyTorch FSDP2, Megatron-FSDP) use fixed sharding formats (element-wise or uniform row-wise) that conflict with modern training techniques:

| Technique | Conflict with Naive Sharding |
|-----------|------------------------------|
| **Block-wise quantization** (8-bit Adam) | Requires intact tensor blocks per device |
| **Matrix optimizers** (Shampoo, Muon) | Operate on full 2D matrices, misalign with element-wise sharding |
| **MoE models** | Expert-parallelism requires asymmetric sharding |

> *"By replacing rigid row-wise sharding with a novel **RaggedShard** format and a structure-aware planning algorithm, it achieves a 5–66% throughput boost and 30% memory savings, supporting advanced optimizers like Muon and block-wise quantization out-of-the-box."*

**Source:** [Wispaper Blog](https://www.wispaper.ai/en/blog/ve-scale-fsdps-flexible-high-performance-fsdps-at-scale-20260228/eng)

---

### RaggedShard — Flexible DTensor Placement

> *"`RaggedShard` is a new placement for PyTorch DistributedTensor (`DTensor`) that extends the standard `Shard` placement to natively express **asymmetric sharding**. This enables flexibility required by modern LLM workflows."*

**Source:** [GitHub: volcengine/veScale — docs/texts/raggedshard.md](https://github.com/volcengine/veScale/blob/main/docs/texts/raggedshard.md)

```python
RaggedShard(dims, local_units)
```

Where:
- **`dims`**: Dimensions for sharding
- **`local_units`**: Local unit sizes for sharding granularity

> *"`RaggedShard` enables several important capabilities... defined by two parameters, `dims` and `local_units`, which allow it to express element-wise, symmetric/asymmetric row-wise, and block-wise sharding granularities."*

**Source:** [PyTorch Issue #169320](https://github.com/pytorch/pytorch/issues/169320)

> *"`RaggedShard` composes with standard DTensor placements for N-dimensional parallelism: `(RaggedShard, Shard(0))` — 2D parallelism example."*

---

### Structure-Aware Planning Algorithm

> *"Our goal is to minimize S subject to three constraints: Non-Sharded Block, Contiguous Tensor Memory, and Balanced Load."*

**Source:** [arXiv:2602.22437v2 — §5](https://arxiv.org/html/2602.22437v2)

**Three Constraints:**
1. **Non-Sharded Block** — Blocks placed contiguously, not split across shard boundaries
2. **Contiguous Tensor Memory** — Padding between tensors, not within tensors
3. **Balanced Load** — Equal buffer sizes across devices for collective symmetry

> *"This is formulated as an NP-hard problem... We devise an efficient polynomial-time heuristic based on dynamic programming (DP) with complexity O(|T|² · m · log(E) · log(|T|·m))."*

**Source:** [arXiv:2602.22437v3 — §5](https://arxiv.org/html/2602.22437v3)

---

### DBuffer — Distributed Buffer

> *"We design a distributed buffer (DBuffer) that provides a global buffer semantics over N-dimensional device topology, enabling zero-copy access via persistent address mapping between the global collective buffer and per-parameter local memory."*

**Source:** [arXiv:2602.22437v2](https://arxiv.org/html/2602.22437v2)

---

## Key Sources

### Primary Paper: veScale-FSDP

| Field | Value |
|-------|-------|
| **Title** | veScale-FSDP: Flexible and High-Performance FSDP at Scale |
| **URL** | [arXiv:2602.22437](https://arxiv.org/abs/2602.22437) |
| **Versions** | v1 (Feb 2026), v2, v3 (April 2026) |
| **Authors** | Zezhou Wang, Youjie Li, Zhiqi Lin, Jiacheng Yang, Cong Xie, et al. (ByteDance Seed) |
| **Code** | [github.com/volcengine/veScale](https://github.com/volcengine/veScale) |

> *"We introduce veScale-FSDP, a redesigned FSDP system that couples a flexible sharding format, RaggedShard, with a structure-aware planning algorithm to deliver both flexibility and performance at scale."*

> *"veScale-FSDP achieves 5~66% higher throughput and 16~30% lower memory usage than existing FSDP systems, scaling efficiently to tens of thousands of GPUs."*

### PyTorch DTensor Issue (RaggedShard Proposal)

> *"To make PyTorch DTensor even more powerful, we would like to gently propose adding a new DTensor placement called `RaggedShard`, which expresses uneven sharding across devices with configurable sharding granularity (e.g., non-shardable blocks). The existing Shard placement only supports even sharding and does not meet the needs of many modern applications, especially block-wise quantization and structure-aware training."*

**Source:** [github.com/pytorch/pytorch/issues/169320](https://github.com/pytorch/pytorch/issues/169320)

### Blog: Wispaper

> *"Existing systems like **PyTorch FSDP2** also suffer from 'interleaved copy' overhead, where tensors must be copied out of a contiguous communication buffer into fragmented per-parameter memory addresses, **wasting up to 14% of training time**."*

**Source:** [Wispaper — veScale-FSDP](https://www.wispaper.ai/en/blog/ve-scale-fsdps-flexible-high-performance-fsdps-at-scale-20260228/eng)

---

## Repositories & Tools

### veScale (Primary)
| Field | Value |
|-------|-------|
| **URL** | [volcengine/veScale](https://github.com/volcengine/veScale) |
| **License** | Apache License v2.0 |
| **Key Doc** | [docs/texts/raggedshard.md](https://github.com/volcengine/veScale/blob/main/docs/texts/raggedshard.md) |

### Related PyTorch Repositories
| Repository | Relevance |
|------------|-----------|
| [pytorch/torchtitan](https://github.com/pytorch/torchtitan) | FSDP2 reference implementation; per-parameter DTensor sharding |
| [pytorch/pytorch#169320](https://github.com/pytorch/pytorch/issues/169320) | RaggedShard DTensor proposal to PyTorch |
| [pytorch/examples](https://github.com/pytorch/examples/blob/main/distributed/FSDP2/checkpoint.py) | FSDP2 checkpoint example |
| [huggingface/accelerate](https://huggingface.co/docs/accelerate/concept_guides/fsdp1_vs_fsdp2) | FSDP2 via Accelerate |

---

## Benchmarks & Performance

### veScale-FSDP Performance

| Metric | Improvement vs Existing FSDP |
|--------|------------------------------|
| **Throughput** | 5~66% higher |
| **Memory Usage** | 16~30% lower peak |
| **Scalability** | Linear scaling to 10,000 GPUs |
| **MFU with Muon** | >47% on 256 GPUs |
| **Planning Runtime** | <0.3 seconds (one-time) |

### End-to-End Benchmarks

| Model | Throughput Improvement | Memory Reduction |
|-------|----------------------|------------------|
| **MoE models** | 11~66% faster | 16~30% lower |
| **LLaMA-3-70B** | 5% faster | 16~30% lower |

### Planning Algorithm Ablation

| Component Disabled | Normalized Throughput |
|--------------------|---------------------|
| Full System | 100.0% |
| DBuffer only | 92.8% |
| Planning Algorithm only | 65.4% |

---

## Use Cases

### Block-wise Quantization (8-bit Adam)

> *"8-bit Adam applies block-wise INT8 quantization to the gradient statistics, substantially reducing optimizer-state memory. To enable communication-free quantization, veScale-FSDP's RaggedShard DTensor maps parameters to a global buffer with customizable granularity matching the quantization block size (32×32 = 1024 elements per block)."*

**Source:** [arXiv:2602.22437v2 — §6.3](https://arxiv.org/html/2602.22437v2)

### Muon & Shampoo (Matrix Optimizers)

> *"Structure-aware optimizers (e.g., Shampoo and Muon) are critical training technique empowering state-of-the-art models."*

**Source:** [veScale docs/texts/raggedshard.md](https://github.com/volcengine/veScale/blob/main/docs/texts/raggedshard.md)

> *"Distributed Muon optimizer achieves 47.3% MFU on 256 GPUs."*

**Source:** [arXiv:2602.22437v2](https://arxiv.org/html/2602.22437v2)

### Communication-Efficient FSDP

> *"`RaggedShard` enables **native asymmetric sharding** with flattened parameters sharded across device subset and **zero-copy collective communication**."*

### Distributed Checkpointing

> *"`RaggedShard` natively supports PyTorch Distributed Checkpoint (DCP): DTensor chunk-list interface analogous to `Shard` placement. Eliminates extra resharding communication of legacy FSDPs."*

### MoE Models

> *"Megatron-FSDP suffers from 33% buffer padding inflation in MoE models. veScale-FSDP achieves 11~66% throughput improvement on MoE models."*

---

## Summary

| Aspect | Details |
|--------|---------|
| **Components** | RaggedShard (DTensor Placement) + Structure-Aware Planning Algorithm |
| **Problem Solved** | Rigid sharding conflicts with block-wise quantization and matrix optimizers |
| **Key Innovation** | Arbitrary sharding granularity + uneven distribution across devices |
| **Paper** | [arXiv:2602.22437](https://arxiv.org/abs/2602.22437) (ByteDance Seed, v3 April 2026) |
| **Code** | [volcengine/veScale](https://github.com/volcengine/veScale) |
| **Throughput Gain** | 5~66% vs existing FSDP systems |
| **Memory Savings** | 16~30% lower peak memory |
| **Scalability** | Linear to 10,000 GPUs |
| **PyTorch Integration** | Proposed as new DTensor `Placement` ([Issue #169320](https://github.com/pytorch/pytorch/issues/169320)) |
| **Planner Complexity** | O(\|T\|² · m · log(E) · log(\|T|·m)) — polynomial-time DP |

---

## Comparison: PyTorch FSDP2 vs veScale-FSDP

| Feature | PyTorch FSDP2 (`Shard(0)`) | veScale-FSDP (`RaggedShard`) |
|---------|---------------------------|------------------------------|
| **Granularity** | Fixed row-wise (even) | Customizable (elements, rows, blocks) |
| **Distribution** | Equal-sized shards per device | Arbitrary blocks per device |
| **Block Quantization** | Misaligns with block boundaries | Perfect alignment |
| **Matrix Optimizers** | Breaks on misaligned elements | Full support |
| **Copy Overhead** | Up to 14% of training time | Zero-copy via DBuffer |
| **Padding (MoE)** | 33% inflation | <3% overhead |
| **Redistribute** | Even-only | Uneven/asymmetric supported |

---

## Related Topics

- [Muon-Optimizers](./Muon-Optimizers.md) — Matrix-level optimizer enabled by RaggedShard
- [FSDP2](../FSDP2/FSDP2-Overview.md) — PyTorch's native FSDP2 (context for why veScale-FSDP was needed)
- [Distributed-Training](../Distributed-Training/Distributed-Training-Overview.md) — Broader distributed training context
