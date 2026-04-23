# Synthetic-flywheels

## 📌 Overview

**Synthetic data flywheels** are self-reinforcing loops where models generate their own training data, which is filtered and used to train improved models, creating continuous iterative improvement. These systems combine Teacher-Student loops with SDG (Self-Generated Data) frameworks.

> *"Models iteratively improve by generating new, higher-quality training data based on their own operations and errors."*
> — [EmergentMind](https://www.emergentmind.com/topics/data-flywheel-paradigm)

---

## Topics

| Topic | Description |
|-------|-------------|
| [Teacher-Student-Loops.md](./Teacher-Student-Loops.md) | Iterative self-improvement: LLM2LLM, SynPO, Arena Learning, DexFlyWheel |
| [SDG.md](./SDG.md) | Self-Generated Data frameworks: SRDF, DexFlyWheel, RLSyn, SDG Hub |

---

## Taxonomy Matrix

| Method | Domain | Iteration | Key Result |
|--------|--------|-----------|------------|
| **Self-Instruct** | Instruction tuning | Bootstrap | +33% instruction-following |
| **LLM2LLM** | Math/Legal | Targeted augmentation | +24.2% GSM8K |
| **SynPO** | General (ICLR 2025) | Self-preference optimization | +34.0% AlpacaEval |
| **Arena Learning** | Chat (Microsoft) | Model battles | +460 Elo (WizardLM-β) |
| **DexFlyWheel** | Robotics (NeurIPS 2025) | 3 cycles | 16.5% → 81.9% success |
| **SRDF** | Vision-Language Nav | 3 iterations | Exceeds human 76% |

---

## Model Collapse Risk

> *"MAD typically manifests after ~5 training cycles. Repeated training on synthetic data causes model distribution to drift from reality."*
> — [Rice DSP](https://dsp.rice.edu/ai-loops/)

**Mitigation:** Mix real + synthetic data, watermarking, active filtering, Constitutional AI.

---

## Related Topics

- [datasets-curation](../) — Parent folder
