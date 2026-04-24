# reasoning-Trajectories

## 📌 Overview

Datasets and methods for collecting, generating, and filtering reasoning trajectories. Covers CoT data collection pipelines and verifiable reasoning methods (PRMs, benchmarks, step-level verification).

> *"Process supervision significantly outperforms outcome supervision on MATH dataset."*
> — [OpenAI "Let's Verify Step by Step"](https://arxiv.org/abs/2305.20050)

---

## Topics

| Topic | Description |
|-------|-------------|
| [CoT-data.md](./CoT-data.md) | CoT data collection: CoT-Self-Instruct, SelF-Reasoner, SWiRL, Simula |
| [Verifiable-Reasoning.md](./Verifiable-Reasoning.md) | PRMs, ProcessBench, PRM800K, VPRM, FoVer |

---

## Taxonomy Matrix

| Method | Type | Key Result |
|-------|------|-----------|
| **CoT-Self-Instruct** | CoT generation | 58.7% MATH500 (vs 47.5% baseline) |
| **SelF-Reasoner** | CoT filtering | 87.24% ScienceQA (≈ human 88.4%) |
| **SWiRL** | Process RL | GSM8K +21.5%, HotPotQA +12.3% |
| **Simula** | Agentic generation | Dual-critic for correctness + incorrectness |
| **ThinkPRM** | Step verifier | 1% labels = full performance |
| **R-PRM** | Reasoning PRM | +8.7 F1 over Qwen-PRM800K |

---

## Key Insight

> *"It is better to have less high-quality synthetic data than more lower-quality data."*
> — [CoT-Self-Instruct](https://arxiv.org/abs/2507.23751)

---

## Related Topics

- [quality-filtering](../quality-filtering/) — General data quality filtering methods
- [Synthetic-flywheels](../Synthetic-flywheels/) — Synthetic data generation loops
