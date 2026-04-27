# fsdp2-vescale

> **📌 Overview**
> Research on FSDP2 (Fully Sharded Data Parallel v2) and the veScale system — covering structure-aware sharding, RaggedShard DTensor placement, and Muon optimizers for distributed LLM training at scale.

---

## Subtopics

| File | Description |
|------|-------------|
| [Structure-Aware-Sharding.md](./Structure-Aware-Sharding.md) | RaggedShard + structure-aware planning algorithm (veScale-FSDP, ByteDance Seed). 5–66% throughput gain, 16–30% memory reduction, NP-hard planning solved via DP. |
| [Muon-Optimizers.md](./Muon-Optimizers.md) | Matrix-level optimizer using Newton-Schulz orthogonalization. ~52% FLOPs vs AdamW, 50% memory savings. Used in Kimi K2, Moonlight, GLM-5. |

---

## Overview Comparison

| Aspect | Structure-Aware Sharding | Muon Optimizers |
|--------|-------------------------|-----------------|
| **Core Focus** | FSDP sharding format + planning algorithm | Optimizer design |
| **Primary System** | veScale-FSDP (ByteDance Seed) | Multiple (MoonshotAI, NVIDIA, veScale) |
| **Key Innovation** | RaggedShard DTensor, NP-hard DP planner | Newton-Schulz orthogonalization |
| **Main Benefit** | 5–66% throughput, 16–30% memory reduction | ~52% FLOPs vs AdamW |
| **Problem Solves** | Rigid FSDP sharding vs block quantization/matrix optimizers | High-condition-number gradient updates in transformers |

---

## Key Papers

| Paper | Venue | URL |
|-------|-------|-----|
| veScale-FSDP: Flexible and High-Performance FSDP at Scale | arXiv 2026 | [arXiv:2602.22437](https://arxiv.org/abs/2602.22437) |
| Muon is Scalable for LLM Training | arXiv 2025 | [arXiv:2502.16982](https://arxiv.org/abs/2502.16982) |
| Kimi K2: Open Agentic Intelligence | arXiv 2025 | [arXiv:2507.20534](https://arxiv.org/abs/2507.20534) |
| Delving into Muon and Beyond | arXiv 2026 | [arXiv:2602.04669](https://arxiv.org/html/2602.04669v1) |
| AdaMuon: Adaptive Muon Optimizer | arXiv 2025 | [arXiv:2507.11005](https://arxiv.org/abs/2507.11005) |

---

## Repositories

| Repo | Description |
|------|-------------|
| [volcengine/veScale](https://github.com/volcengine/veScale) | ByteDance's FSDP redesign with RaggedShard |
| [KellerJordan/Muon](https://github.com/KellerJordan/Muon) | Original Muon implementation |
| [MoonshotAI/Moonlight](https://github.com/MoonshotAI/Moonlight) | Muon-trained 3B/16B MoE model |
| [NVIDIA-NeMo/Emerging-Optimizers](https://github.com/NVIDIA-NeMo/Emerging-Optimizers) | NVIDIA's Muon integration |
| [samsja/muon_fsdp_2](https://github.com/samsja/muon_fsdp_2) | Muon + FSDP2 integration |
| [nil0x9/flash-muon](https://github.com/nil0x9/flash-muon) | CUDA-optimized Newton-Schulz |

---

## Related Topics

- [Distributed-Training](../Distributed-Training/Distributed-Training-Overview.md) — Broader distributed training context
- [FSDP2](../FSDP2/FSDP2-Overview.md) — PyTorch FSDP2 native (pre-veScale)
- [Optimizer-Variants](../Optimizer-Variants/Shampoo.md) — Shampoo optimizer (Muon predecessor)

---

*Last updated: 2026-04-27*
