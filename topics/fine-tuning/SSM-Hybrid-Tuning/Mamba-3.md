# Mamba-3

## 📌 Overview

**Mamba-3** is the third generation of State Space Model (SSM) architecture from [State Spaces Inc./Mamba](https://github.com/state-spaces/mamba), released March 2026, with inference efficiency as its primary design goal. It introduces three core architectural innovations: exponential-trapezoidal discretization, complex-valued state updates, and MIMO (Multi-Input, Multi-Output) parallelism.

> *"Mamba-3 is a state space model redesign with inference efficiency as primary goal."*
> — [Mamba Blog](https://www.state-spaces.com/mamba)

---

## Core Innovations

### 1. Exponential-Trapezoidal Discretization

Replaces heuristic discretization methods from Mamba-1/2 with a more expressive recurrence:

> *"Exponential-trapezoidal discretization — more expressive recurrence replacing heuristic methods from Mamba-1/2."*
> — [Mamba Release Notes](https://github.com/state-spaces/mamba/releases)

### 2. Complex-Valued State Updates

> *"Complex-valued state updates enable rotational dynamics for better state-tracking (e.g., 100% on parity tasks vs. <3% for Mamba-2)."*
> — [Mamba Technical Blog](https://sr pele.github.io/mamba-SSD/)

### 3. MIMO (Multi-Input, Multi-Output)

> *"Adds parallel work inside each update without increasing decode latency."*
> — [Mamba Release Notes](https://github.com/state-spaces/mamba/releases)

---

## Performance

### Benchmark Results

| Model | Parity (synthetic) | Prefill+Decode Latency | Notes |
|-------|---------------------|------------------------|-------|
| **Mamba-3-SISO** | **100%** | Fastest across all seq lengths | vs <3% for Mamba-2 |
| Mamba-2 | <3% | Baseline | Struggles with parity |
| Transformer (vLLM-optimized) | — | Baseline | Mamba-3 outperforms at 1.5B scale |

> *"Mamba-3-SISO achieves fastest prefill+decode latency across all sequence lengths, outperforming even vLLM-optimized Transformers at 1.5B scale."*
> — [Mamba Blog](https://www.state-spaces.com/mamba)

---

## Mamba Architecture Lineage

| Version | Key Innovation | State Size | Speed |
|---------|---------------|------------|-------|
| **Mamba-1** | Original SSM, selective state passing | d_state=16 | Baseline |
| **Mamba-2** | State Space Duality (SSD), faster training via matmuls | Larger state | 2-3× faster training |
| **Mamba-3** | Exponential-trapezoidal discretization, complex-valued states, MIMO | Optimized | Fastest inference |

> *"Mamba-2 connects SSMs to structured matrices via State Space Duality (SSD). Preserves efficient FLOP counts as SSMs while dramatically speeding up training using matmuls."*
> — [Tri Dao Blog](https://tridao.me/blog/2024/mamba2-part3-algorithm/)

---

## PEFT for Mamba/Hybrid Models

### What Works

> *"MambaPEFT: Comprehensive benchmarking — PEFT works *better* for Mamba than Transformers; proposes HybridPEFT combining 8 methods."*
> — [ICLR 2025](https://openreview.net/forum?id=LtzmeSMBTW)

| Method | Effectiveness | Notes |
|--------|---------------|-------|
| **LoRA on linear projections** | ✅ Effective | q_proj, k_proj, v_proj, o_proj |
| **LoRA on SSM modules** | ⚠️ Less effective | Not recommended |
| **Prefix/Prompt Tuning** | ❌ Fails | "Maximum expressiveness = tuning initial state h₀ only" |

> *"LoRA on linear projections outperforms LoRA on SSM modules."*
> — [arXiv:2410.09016](https://arxiv.org/abs/2410.09016)

### PEFT Methods for SSMs

| Method | Source | Description |
|--------|--------|-------------|
| **HybridPEFT** | ICLR 2025 | Combines 8 PEFT methods; PEFT works better for Mamba than Transformers |
| **State-offset Tuning** | ACL 2025 | Adds learnable state-offset at every timestep |
| **SDLoRA** | NeurIPS 2024 | LoRA with Selective Dimension tuning for SSM modules |
| **ProDiaL** | CVPR 2025 | Projector-targeted Diagonal-centric Linear Transformation |

### State-offset Tuning (ACL 2025)

> *"State-offset Tuning: Adds learnable state-offset h' directly to hidden state at every timestep: ŷ_t = y_t + C_th'."*
> — [arXiv:2503.03499](https://arxiv.org/abs/2503.03499)

> *"Consistent impact at every timestep vs prefix-tuning which diminishes over time."*
> — [arXiv:2503.03499](https://arxiv.org/abs/2503.03499)

---

## Mamba-3 in Hybrid Context

Mamba-3 can be combined with attention layers in hybrid architectures (similar to Jamba's approach):

> *"We found that in a hybrid architecture, the Mamba-1-Attention combination works better than Mamba-2-Attention."*
> — [Jamba-1.5 Paper](https://arxiv.org/abs/2408.12570)

> *"Mamba-2's larger state size advantages are diminished when interleaved with full attention layers."*
> — [Jamba-1.5 Paper](https://arxiv.org/abs/2408.12570)

---

## GitHub & Resources

| Resource | URL |
|----------|-----|
| **Mamba Official Repo** | [state-spaces/mamba](https://github.com/state-spaces/mamba) |
| **Mamba-3 Release** | [Mamba 3.0 Release Notes](https://github.com/state-spaces/mamba/releases) |
| **Mamba Blog** | [state-spaces.com/mamba](https://www.state-spaces.com/mamba) |
| **Tri Dao Blog (Mamba-2)** | [tridao.me/blog/2024/mamba2-part3-algorithm](https://tridao.me/blog/2024/mamba2-part3-algorithm/) |

---

## Sources

| Type | Citation |
|------|----------|
| **GitHub** | [state-spaces/mamba](https://github.com/state-spaces/mamba) |
| **Paper** | [arXiv:2403.19887 — Jamba](https://arxiv.org/abs/2403.19887) |
| **Paper** | [arXiv:2408.12570 — Jamba-1.5](https://arxiv.org/abs/2408.12570) |
| **Paper** | [arXiv:2410.09016 — PEFT for SSMs](https://arxiv.org/abs/2410.09016) |
| **Paper** | [arXiv:2503.03499 — State-offset Tuning](https://arxiv.org/abs/2503.03499) |
| **Blog** | [Tri Dao Mamba-2 Blog](https://tridao.me/blog/2024/mamba2-part3-algorithm/) |
| **ICLR 2025** | [MambaPEFT / HybridPEFT](https://openreview.net/forum?id=LtzmeSMBTW) |

---

## Related Topics

- [Jamba](./Jamba.md) — Hybrid Transformer-Mamba architecture
- [State-Optimization](./State-Optimization.md) — SSM state compression and optimization techniques
