# Verifiable Reasoning

## 📌 Overview

**Verifiable reasoning** refers to reasoning tasks where answers can be mechanically checked — enabling automated quality filtering, step-level feedback, and process supervision. Process Reward Models (PRMs) provide fine-grained step-level evaluation, enabling reasoning verification that outperforms outcome-only methods.

> *"Process Reward Models (PRMs) offer a fine-grained evaluation of intermediate reasoning steps and guide the reasoning process."*
> — [OpenAI "Let's Verify Step by Step"](https://arxiv.org/abs/2305.20050)

---

## Process Reward Models (PRMs)

PRMs evaluate each intermediate step in multi-step reasoning, unlike Outcome Reward Models (ORMs) that only assess final results.

| Model Type | Evaluates | Use Case |
|-----------|-----------|----------|
| **PRM** | Each step | Fine-grained error detection, process supervision |
| **ORM** | Final answer only | Outcome-based training |

**Capabilities:**
- Fine-grained error detection in reasoning chains
- Test-time compute scaling via Best-of-N selection
- Guidance of policy models toward correct solutions

---

## Key Benchmarks

| Benchmark | Description | Size |
|-----------|-------------|------|
| **PRMBench** | Process-level evaluation across 9 dimensions (simplicity, soundness, sensitivity) | 6,216 problems, 83,456 step-level labels |
| **ProcessBench** | Step-level error identification in math reasoning | Multiple datasets |
| **VisualProcessBench** | Human-annotated multimodal reasoning | Multiple benchmarks |
| **AgentProcessBench** | Step-level evaluation for tool-using agents | 1,000 trajectories, 8,509 steps |
| **miniF2F** | Formal Olympiad-level math (Lean/Isabelle/Metamath) | 488 problems |
| **FormalMATH-Bench** | Large-scale Lean4 benchmark | 5,560 formally verified statements |
| **MathVista** | Multimodal mathematical reasoning | 6,141 examples |

---

## Key Training Datasets

| Dataset | Source | Description |
|---------|--------|-------------|
| **PRM800K** | OpenAI | 800,000 step-level human feedback labels on MATH problems |
| **FoVer-40K** | Formal verification (Z3, Isabelle) | Auto-annotated step-level error labels |
| **VisualPRM400K** | Multimodal reasoning | ~400K multimodal process supervision data |
| **OmegaPRM** | MCTS-based collection | 1.5M+ automated process supervision annotations |

---

## Notable Methods

### OpenAI — "Let's Verify Step by Step" (2023)

> *"Process supervision significantly outperforms outcome supervision on MATH dataset."*
> — [arXiv:2305.20050](https://arxiv.org/abs/2305.20050)

Introduced PRM800K dataset.

### VPRM — Verifiable Process Reward Models

> *"Uses deterministic, rule-based verifiers instead of neural judges; achieves up to 20% higher F1 than state-of-the-art."*
> — [arXiv:2601.17223](https://arxiv.org/abs/2601.17223)

### FoVer — Formally Verified Training Data

> *"Uses Z3/Isabelle for automatic step-level error annotation; demonstrates cross-task generalization."*
> — [FoVer Paper](https://arxiv.org/abs/2406.20044)

### ThinkPRM — Generative Long-CoT Verifier

> *"Uses only 1% of PRM800K labels but outperforms baselines."*
> — [ThinkPRM Paper](https://arxiv.org/abs/2504.16828)

### R-PRM — Reasoning-Driven PRM

> *"Outperforms Qwen2.5-Math-7B-PRM800K by +8.7 F1 with same data scale."*
> — [R-PRM Paper](https://arxiv.org/abs/2501.12521)

### PURE — Min-Form Credit Assignment

> *"Identifies reward hacking cause in summation-form credit assignment."*
> — [PURE Paper](https://arxiv.org/abs/2503.03499)

### VSRM — Verifiable Stepwise Reward Mechanism

Addresses "over-thinking" problem.

---

## PRM Methods Comparison

| Method | Key Innovation | Advantage |
|--------|---------------|-----------|
| **VPRM** | Rule-based verifiers | +20% F1 vs neural |
| **FoVer** | Formal verification (Z3/Isabelle) | Cross-task generalization |
| **ThinkPRM** | Long-CoT verification | 1% labels = full performance |
| **R-PRM** | Reasoning-driven approach | +8.7 F1 over PRM800K |
| **PURE** | Min-form credit assignment | Reduces reward hacking |
| **VisualPRM** | Multimodal PRM | Multimodal reasoning |
| **GM-PRM/DreamPRM** | Additional multimodal variants | Extended domains |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2305.20050 — "Let's Verify Step by Step" (OpenAI)](https://arxiv.org/abs/2305.20050) |
| **Paper** | [arXiv:2601.17223 — VPRM](https://arxiv.org/abs/2601.17223) |
| **Paper** | [arXiv:2406.20044 — FoVer](https://arxiv.org/abs/2406.20044) |
| **Paper** | [arXiv:2504.16828 — ThinkPRM](https://arxiv.org/abs/2504.16828) |
| **Paper** | [arXiv:2501.12521 — R-PRM](https://arxiv.org/abs/2501.12521) |
| **Paper** | [arXiv:2503.03499 — PURE](https://arxiv.org/abs/2503.03499) |
| **Benchmark** | [PRMBench](https://prmbench.github.io/) |
| **Benchmark** | [ProcessBench](https://github.com/microsoft/ProcessBench) |

---

## Related Topics

- [CoT-data.md](./CoT-data.md) — CoT data collection, generation, filtering
