# Mamba-3 Complex-Valued States

> **Source:** [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) · [Tri Dao Blog](https://tridao.me/blog/2026/mamba3-part1/) · [Goomba Lab Blog Part 2](https://goombalab.github.io/blog/2026/mamba3-part2/)  
> **Subtopic of:** [Mamba-3](./README.md)

---

## The Problem: Real-Valued Transitions Cannot Represent Rotational Dynamics

Mamba-2 restricted the state transition matrix **A** to **real-valued scalars** to align with matrix multiplication accelerators (the "SSD" duality). This came at a cost:

> *"Real-valued, non-negative eigenvalue transitions **cannot represent rotational dynamics**, degrading state-tracking capability (e.g., parity requires rotation matrices)."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

> *"Real eigenvalues **cannot represent oscillatory/rotational mechanics** needed for periodicity tasks (parity, modular arithmetic)."*  
> — [arxiv-iqsubstack](https://arxiv-export1.github.io/)/[Goomba Lab](https://goombalab.github.io/blog/2026/mamba3-part2/)

### The Parity Task Failure

| Model | Parity Accuracy |
|---|---|
| Mamba-2 | **≈0.9%** (near-random) |
| **Mamba-3** | **100%** |

The parity task (determining if the sum of 0/1 bits is even) is **unsolvable by Mamba in constant layers with real-valued non-negative eigenvalues**. Real eigenvalues force a naive "add all then mod 2" behavior which fails on long sequences.

---

## How Complex-Valued States Work

### Core Mechanism: The "RoPE Trick"

Instead of modifying kernels to rotate the hidden state, **embed rotations into B and C projections** before the SSM recurrence:

> *"Instead of multiplying by a complex diagonal A, the model applies rotation matrices R_t to projections **B** and **C**. This recovers the 'expressivity of complex dynamics' while keeping the speed of real-valued recurrence."*  
> — [MarkTechPost](https://www.marktechpost.com/2026/03/18/meet-mamba-3-a-new-state-space-model-frontier-with-2x-smaller-states-and-enhanced-mimo-decoding-hardware-efficiency/)

From [Goomba Lab Blog Part 2](https://goombalab.github.io/blog/2026/mamba3-part2/):

> *"Since `Ā_t = e^{Δ_t(A_t + iθ_t)} = e^{Δ_t A_t} e^{iΔ_t θ_t}`, the complex exponential separates into scaling × rotation."*

> *"The complex SSM output is equivalent to a vanilla scalar-transition SSM with **data-dependent rotary embeddings** applied to B,C."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

### Mathematical Formalism

**Complex-to-Real SSM Equivalence (from arXiv paper):**

A complex SSM with state `h(t) ∈ ℂ^{N/2}` and transition `Diag(A(t) + iθ(t))` is equivalent under exponential-Euler discretization to a real SSM with:
- State `h_t ∈ ℝ^N`
- Transition: `e^{Δ_t A_t} R_t` where `R_t` is block-diagonal of 2×2 rotation matrices `R(Δ_t θ_t[i])`
- Projections: `B_t = [B_t; B̂_t]`, `C_t = [C_t; -Ĉ_t]`

### Key Insight

> *"Rotations solve modulo-m tasks: partition `[0, 2π]` into *m* sections, rotate by `2π/m`. Complex eigenvalues correspond to rotations and oscillations — needed for periodicity, parity, modular arithmetic."*  
> — [Goomba Lab Blog Part 2](https://goombalab.github.io/blog/2026/mamba3-part2/)

### Computational Overhead

> *"Mamba-3 adds minimal forward-pass cost, showing that the exponential-trapezoidal update, **complex state tracking**, and MIMO parameterization remain lightweight."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

> *"The 'RoPE trick' applies **aggregated data-dependent rotations across time steps** — this insight allows Mamba-3 to retain the speed of real-valued recurrence while capturing the expressive power of complex dynamics."*  
> — [Together AI Blog](https://www.together.ai/blog/mamba-3)

> *"Computational overhead: ~3-4% of layer wall-clock time for cumulative angle calculation."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

---

## State-Tracking Results

### Formal Language Tasks (440M scale)

| Model | Parity ↑ | Arithmetic w/o Brackets ↑ | Arithmetic w/ Brackets ↑ |
|---|---|---|---|
| **Mamba-3** | **100.00** | **98.51** | **87.75** |
| Mamba-3 (w/ Std. RoPE) | 1.56 | 20.70 | 2.62 |
| Mamba-3 (w/o RoPE) | 2.27 | 1.49 | 0.72 |
| Mamba-2 | 0.90 | 47.81 | 0.88 |
| Gated DeltaNet [-1,1] | 100.00 | 99.25 | 93.50 |

> *"Mamba-2 scored **0.9%** on parity (random guessing) → Mamba-3 with complex states: **100%**."*  
> — Compiled from [arXiv:2603.15569](https://arxiv.org/abs/2603.15569)

### Why This Matters

The historical knock on SSMs:

> *"They compressed too aggressively and lost the fine-grained recall that attention excels at. Mamba-1 closed some of that gap. Mamba-2 closed more of it. Mamba-3 arguably finished the job."*  
> — [BuildMVPFast](https://www.buildmvpfast.com/blog/mamba-3-vs-transformer-ssm-inference-2026)

---

## Relationship to MIMO

The complex-valued mechanism is **orthogonal to MIMO** — they address different bottlenecks:

| Mechanism | Problem Solved | How |
|---|---|---|
| **Complex-Valued States** | Expressivity loss (can't do rotations) | Data-dependent RoPE embedded in B,C |
| **MIMO** | Hardware underutilization (memory-bound decode) | Matrix-multiply instead of outer-product |

MIMO with complex-valued B,C projections enables **richer state dynamics per rank**.

---

## Key Quotes

> *"We combine: (1) a more expressive recurrence derived from SSM discretization, (2) a **complex-valued state update rule that enables richer state tracking**, and (3) a multi-input, multi-output (MIMO) formulation for better model performance without increasing decode latency."*  
> — [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) (Abstract)

> *"The exponential-trapezoidal discretization, complex-valued recurrence, and MIMO are independent axes of improvement — all three can be combined or used individually."*  
> — [Tri Dao Blog](https://tridao.me/blog/2026/mamba3-part1/)

---

## 📚 References

- [arXiv:2603.15569](https://arxiv.org/abs/2603.15569) — primary paper
- [Tri Dao Blog Part 1](https://tridao.me/blog/2026/mamba3-part1/)
- [Goomba Lab Blog Part 2](https://goombalab.github.io/blog/2026/mamba3-part2/)
- [Together AI Blog](https://www.together.ai/blog/mamba-3)
- [MarkTechPost](https://www.marktechpost.com/2026/03/18/meet-mamba-3-a-new-state-space-model-frontier-with-2x-smaller-states-and-enhanced-mimo-decoding-hardware-efficiency/)
- [BuildMVPFast](https://www.buildmvpfast.com/blog/mamba-3-vs-transformer-ssm-inference-2026)
- [YouTube: Mamba-3 Complex Rotations Talk](https://www.youtube.com/watch?v=9W3iO8B4PZw)
