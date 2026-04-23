# SDG (Self-Generated Data)

## 📌 Overview

**SDG** in synthetic data flywheels refers to systems where models alghorithmically produce artificial datasets to power self-reinforcing improvement cycles. The "flywheel" concept describes iterative loops where models generate data, which is refined and used to train improved models.

> *"Models iteratively improve by generating new, higher-quality training data based on their own operations and errors."*
> — [EmergentMind](https://www.emergentmind.com/topics/data-flywheel-paradigm)

---

## Data Flywheel Architecture (NVIDIA)

| Stage | Description |
|-------|-------------|
| **Data Processing** | Extract/refine raw enterprise data |
| **Model Customization** | DAPT, LoRA, SFT for domain knowledge |
| **Model Evaluation** | Verify outputs align to requirements |
| **AI Guardrails** | Privacy, security, safety requirements |
| **Custom Model Deployment** | RAG for runtime retrieval |
| **Enterprise Data Refinement** | Capture inference logs, human/AI feedback |

> *"Six-step self-reinforcing cycle for enterprise AI."*
> — [NVIDIA Data Flywheel](https://www.nvidia.com/en-us/glossary/data-flywheel/)

---

## Key Implementations

### SRDF — Self-Refining Data Flywheel

> *"Two-model collaborative system: Generator creates synthetic instructions, Navigator filters by executing and selecting high-fidelity pairs."*
> — [arXiv:2412.08467](https://arxiv.org/abs/2412.08467)

| Metric | Result |
|--------|--------|
| SPL improvement | 69.9% → **77.6%** |
| Human baseline | 76% |
| Scenario coverage | 214× increase |

### DexFlyWheel — Dexterous Manipulation (NeurIPS 2025 Spotlight)

> *"From single human demonstration per task: 500.0× increase in new trajectories, 214.7× increase in scenario configurations."*
> — [DexFlyWheel](https://dexflywheel.github.io/)

**Pipeline:** Seed Demonstrations → Imitation Learning → Residual RL → Trajectory Rollout → Data Augmentation → (Loop)

### RLSyn — Reinforcement Learning Approach to SDG

> *"Reframes SDG as RL problem using PPO with discriminator-derived rewards. Outperforms GANs and diffusion models on small datasets."*
> — [arXiv:2512.21395](https://arxiv.org/abs/2512.21395)

### Adaptive Data Flywheel with MAPE Control Loops

> *"Monitor → Analyze → Plan → Execute → Knowledge. 10× model size reduction (70B → 8B), 70% latency reduction, 3.7% accuracy improvement."*
> — [arXiv:2510.27051](https://arxiv.org/html/2510.27051v1)

### AITL — Agent-in-the-Loop (Airbnb, EMNLP 2025)

> *"Real-time annotation with four types: Response Preference, Rationale, Knowledge Relevance, Missing Knowledge. Reduced retraining cycles from months to weeks."*
> — [AITL ACL Anthology](https://aclanthology.org/2025.emnlp-industry.135.pdf)

---

## SDG Frameworks & Tools

### SDG Hub (Red Hat)

> *"Open-source Python framework for building synthetic data pipelines. Modular blocks: LLM, parsing, transform, filtering, agent blocks."*
> — [GitHub: Red-Hat-AI-Innovation-Team/sdg_hub](https://github.com/Red-Hat-AI-Innovation-Team/sdg_hub)

Available on PyPI: `pip install sdg-hub`

```yaml
# YAML-defined flows for declarative pipeline configuration
```

### SynthGuard

> *"Framework for modular and composable SDG workflow design. Enables scalable and adaptable privacy policy enforcement."*
> — [SynthGuard](https://synthguard.io/)

---

## Key Mechanisms

| Mechanism | Description |
|-----------|-------------|
| **Self-Generated Corpora** | Models produce new training data, filter for quality, fine-tune |
| **Synthetic Expansion** | Real data seeds → vast variations for edge cases |
| **Feedback Loops** | Models collect new data → fed back into training |
| **Continuous Fine-tuning** | Incremental updates as synthetic data flows in |

---

## Evaluation Metrics

| Category | Metric | Target |
|----------|--------|--------|
| **Privacy** | Membership Inference Attack | AUROC ≈ 0.5 |
| **Utility** | Synthetic-to-Real (S2R) | Higher = better |
| **Fidelity** | CWC, NMI, DWD | Lower = better |
| **Navigation** | SPL, nDTW | Higher = better |
| **Manipulation** | Success Rate | Higher = better |

---

## Key Findings

| Finding | Source |
|---------|--------|
| **Quality > Quantity**: 352× data scaling yielded worse results than small human-annotated data | SRDF |
| **Closed-loop evaluation**: Navigator filtering (nDTW≥0.9) outperforms CLIP-based methods | SRDF |
| **Saturation points**: Diminishing returns after certain iterations | Multiple papers |
| **Privacy-utility tradeoff**: RLSyn achieves better privacy than EHRDiff on small datasets | RLSyn |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2412.08467 — SRDF](https://arxiv.org/abs/2412.08467) |
| **Website** | [DexFlyWheel](https://dexflywheel.github.io/) |
| **Paper** | [arXiv:2512.21395 — RLSyn](https://arxiv.org/abs/2512.21395) |
| **Paper** | [arXiv:2510.27051 — MAPE Control](https://arxiv.org/html/2510.27051v1) |
| **Paper** | [Airbnb AITL](https://aclanthology.org/2025.emnlp-industry.135.pdf) |
| **GitHub** | [SDG Hub](https://github.com/Red-Hat-AI-Innovation-Team/sdg_hub) |
| **Glossary** | [NVIDIA Data Flywheel](https://www.nvidia.com/en-us/glossary/data-flywheel/) |
| **Article** | [EmergentMind — Data Flywheel Paradigm](https://www.emergentmind.com/topics/data-flywheel-paradigm) |

---

## Related Topics

- [Teacher-Student-Loops.md](./Teacher-Student-Loops.md) — Iterative improvement mechanisms
