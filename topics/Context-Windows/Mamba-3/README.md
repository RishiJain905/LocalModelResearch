# Mamba-3: Improved Sequence Modeling using State Space Principles

> **Paper:** [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) · **ICLR 2026 Oral** · March 2026  
> **Authors:** Aakash Lahoti, Kevin Y. Li, Berlin Chen, Caitlin Wang, Aviv Bick, J. Zico Kolter, Tri Dao, Albert Gu (CMU · Princeton · Together AI · Cartesia AI)  
> **Code:** [github.com/state-spaces/mamba](https://github.com/state-spaces/mamba) · **License:** Apache 2.0

---

## 📋 Overview

Mamba-3 is the third iteration of the Mamba selective state space model lineage. It introduces **three core innovations** targeting inference efficiency — the central design question being *"what would an SSM designed with inference in mind look like?"*

| Innovation | Mechanism | Effect |
|---|---|---|
| **Exponential-Trapezoidal Discretization** | Second-order accurate (O(Δ³) local error); three-term recurrence | More expressive dynamics; eliminates the short causal conv layer |
| **Complex-Valued SSM** | Complex state transitions ≡ data-dependent RoPE on B and C ("RoPE trick") | Enables rotational dynamics for state-tracking (parity: 0.9% → 100%) |
| **MIMO (Multi-Input, Multi-Output)** | Matrix-matrix (`B_t X_t^⊤`) instead of outer-product (`B_t x_t^⊤`); rank-R parameterization | >1pp accuracy boost; 2× state efficiency; minimal decode latency increase |

> *"We combine: (1) a more expressive recurrence derived from SSM discretization, (2) a complex-valued state update rule that enables richer state tracking, and (3) a multi-input, multi-output (MIMO) formulation for better model performance without increasing decode latency."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) (Abstract)

---

## 📚 Wiki Sections

### [🔀 MIMO (Multi-Input, Multi-Output)](./MIMO.md)
- Matrix-matrix recurrence (`B_t X_t^⊤`) replacing vector outer-product
- Hardware utilization problem solved: memory-bound decode → compute-efficient
- >1pp accuracy improvement, 2× state efficiency
- Benchmark latency tables, code instantiation

### [🔢 Complex-Valued States](./Complex-Valued-States.md)
- Complex eigenvalues enable rotational dynamics (parity, modular arithmetic)
- "RoPE trick" — rotations embedded into B and C projections
- Parity task: Mamba-2's 0.9% → Mamba-3's **100%**
- Mathematical formalism: Complex-to-Real SSM equivalence

### [📊 Results & Benchmarks](./Results.md)
- Language modeling (FineWeb-Edu, LAMB), state tracking, DNA, audio
- Inference latency tables across sequence lengths
- Hybrid architecture results (retrieval, NIAH, reasoning)
- Third-party reproductions, ICLR reviews, production deployments

---

## 🌐 Official Resources

| Resource | Link |
|---|---|
| **Primary Paper** | [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) |
| **Paper PDF** | [OpenReview PDF](https://openreview.net/pdf?id=HwCvaJOiCj) |
| **OpenReview** | [openreview.net/forum?id=HwCvaJOiCj](https://openreview.net/forum?id=HwCvaJOiCj) |
| **ICLR 2026 Poster** | [iclr.cc/virtual/2026/poster/10010352](https://iclr.cc/virtual/2026/poster/10010352) |
| **GitHub Repo** | [github.com/state-spaces/mamba](https://github.com/state-spaces/mamba) |
| **Tri Dao Blog (Part 1)** | [tridao.me/blog/2026/mamba3-part1/](https://tridao.me/blog/2026/mamba3-part1/) |
| **Goomba Lab Blog (Part 1)** | [goombalab.github.io/blog/2026/mamba3-part1/](https://goombalab.github.io/blog/2026/mamba3-part1/) |
| **Goomba Lab Blog (Part 2)** | [goombalab.github.io/blog/2026/mamba3-part2/](https://goombalab.github.io/blog/2026/mamba3-part2/) |
| **Together AI Blog** | [together.ai/blog/mamba-3](https://www.together.ai/blog/mamba-3) |
| **YouTube Talk** | [youtube.com/watch?v=eKDHDzoM43E](https://www.youtube.com/watch?v=eKDHDzoM43E) |
| **Albert Gu Interview** | [youtube.com/watch?v=1zjMalKLHiA](https://www.youtube.com/watch?v=1zjMalKLHiA) |

---

## 🔗 Connected Topics

| Topic | Relevance |
|---|---|
| [FlashAttention-4](../FlashAttention-4/) | Hardware-aware attention kernel — inference optimization context |
| [KV-Cache-Management](../KV-Cache-Management/) | SSM O(1) state vs Transformer KV-cache growth problem |
| [Managed-Agents-Decoupled-Memory](../Managed-Agents-Decoupled-Memory/) | Agentic workflows drive inference demand — Mamba-3's target use case |

---

## 📝 Citation

```bibtex
@article{lahoti2026mamba3,
  title     = {Mamba-3: Improved Sequence Modeling using State Space Principles},
  author    = {Aakash Lahoti and Kevin Y. Li and Berlin Chen and Caitlin Wang 
               and Aviv Bick and J. Zico Kolter and Tri Dao and Albert Gu},
  journal   = {arXiv:2603.15569},
  year      = {2026},
  note      = {ICLR 2026 Oral, March 2026}
}
```
