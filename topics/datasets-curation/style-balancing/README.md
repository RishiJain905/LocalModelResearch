# style-balancing

## 📌 Overview

Methods for balancing training data composition — optimizing dataset mixtures and mitigating model collapse from synthetic data.

> *"Model performance follows quantitative patterns regarding mixture proportions."*
> — [Data Mixing Laws ICLR 2025](https://arxiv.org/abs/2403.16952)

---

## Topics

| Topic | Description |
|-------|-------------|
| [Dataset-Mix-Optimization.md](./Dataset-Mix-Optimization.md) | DoReMi, RegMix, scaling laws, quality weighting, curriculum learning |
| [Model-Collapse.md](./Model-Collapse.md) | MAD/Model autophagy, Shumailov et al., mitigation strategies |

---

## Taxonomy Matrix

| Method | Key Result | Metric |
|--------|-----------|--------|
| **DoReMi** | +6.5% few-shot, 2.6× faster | NeurIPS 2023 |
| **RegMix** | 98.45% correlation, 2% extra FLOPs | Small→Large extrapolation |
| **Data Mixing Laws** | 73% steps = full performance | ICLR 2025 |
| **Model Collapse** | Manifests after ~5 cycles | MAD prevention |

---

## Related Topics

- [quality-filtering](../quality-filtering/) — Data quality filtering
- [Synthetic-flywheels](../Synthetic-flywheels/) — Synthetic data generation
