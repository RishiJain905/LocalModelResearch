# Dataset-Mix Optimization

## 📌 Overview

**Dataset-mix optimization** determines the optimal proportions and composition of training data across domains, sources, and quality levels. Scaling laws enable predicting model performance on unseen mixtures before training.

> *"Model performance follows quantitative patterns regarding mixture proportions. Enables predicting performance on unseen mixtures before actual training."*
> — [Data Mixing Laws ICLR 2025](https://arxiv.org/abs/2403.16952)

---

## Data Mixing Laws (ICLR 2025)

**Formula:**
```
L_i(r) = c_i + k_i * exp(Σ t_ij * r_j)
```

> *"Achieved equivalent performance at 73% of original training steps on RedPajama."*
> — [Data Mixing Laws Paper](https://arxiv.org/abs/2403.16952)

---

## DoReMi — Domain Reweighting with Minimax Optimization (NeurIPS 2023)

> *"Uses small proxy model (280M) to find domain weights for 30x larger model (8B)."*
> — [DoReMi Stanford](https://crfm.stanford.edu/2023/09/14/doremi.html)

| Metric | Result |
|--------|--------|
| Few-shot accuracy improvement | **+6.5% points** |
| Baseline accuracy speedup | **2.6× faster** |

> *"Dynamically updates domain weights to minimize worst-case excess loss."*
> — [DoReMi Paper](https://openreview.net/forum?id=lXuByUeHhd)

---

## Scaling Laws for Optimal Data Mixtures (Apple ML 2025)

> *"Additive scaling law: optimal domain weights are independent of model size N and tokens D."*
> — [Apple Scaling Laws arXiv:2507.09404](https://arxiv.org/abs/2507.09404)

> *"Parameters can be estimated from small-scale runs and extrapolated. Validated across LLM, multimodal, and vision models."*
> — [Apple Paper](https://arxiv.org/abs/2507.09404)

---

## RegMix Method

> *"Treats data mixture selection as regression task. 98.45% correlation predicting large-scale performance from small models."*
> — [RegMix HF Blog](https://huggingface.co/blog/SivilTaram/regmix)

> *"Only 2% extra training FLOPs required."*
> — [RegMix](https://huggingface.co/blog/SivilTaram/regmix)

> *"Found web corpora show strongest positive correlation with downstream performance."*
> — [RegMix](https://huggingface.co/blog/SivilTaram/regmix)

---

## Quality Weighting Methods

| Method | Approach |
|--------|----------|
| **Classifier-based** | GPT-3 approach — train classifier on quality labels |
| **Perplexity-based** | Small LM scores documents by perplexity |
| **Source-based** | Weight by academic publishers, newspapers, web |

> *"Quality filtering within a domain can substitute for domain diversification."*
> — [Michael Brenndoerfer](https://mbrenndoerfer.com/writing/data-mixing-domain-proportions-quality-weighting-optimal)

---

## Curriculum Learning

> *"Ordering training data by difficulty can reduce steps by 18-45%."*
> — [Stanford CRFM](https://crfm.stanford.edu/2023/09/14/doremi.html)

**Best practice:** Start with high-quality structured text → gradually introduce varied/noisy sources.

> *"Self-adaptive curriculum uses PLM-predicted difficulty scores."*
> — [Stanford CRFM](https://crfm.stanford.edu/2023/09/14/doremi.html)

---

## Practical Guidelines

| Finding | Guideline |
|---------|-----------|
| **Repeating high-quality data** | Beyond 2-4× produces diminishing returns (memorization) |
| **Cross-domain spillover** | Code improves math, reasoning, instruction following |
| **Early training steps** | Disproportionately shape representations |
| **Minimum floor** | 1-5% for all domains improves robustness |

---

## DeMix — Model Merging for Data Mixture Scaling

> *"Model merging for data mixture scaling — enables transferring optimal mixtures across domains."*
> — [DeMix arXiv:2602.00747](https://arxiv.org/html/2602.00747v1)

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [Data Mixing Laws ICLR 2025](https://arxiv.org/abs/2403.16952) |
| **Paper** | [DoReMi NeurIPS 2023](https://openreview.net/forum?id=lXuByUeHhd) |
| **Paper** | [Apple Scaling Laws arXiv:2507.09404](https://arxiv.org/abs/2507.09404) |
| **Blog** | [RegMix — Hugging Face](https://huggingface.co/blog/SivilTaram/regmix) |
| **Blog** | [Stanford CRFM DoReMi](https://crfm.stanford.edu/2023/09/14/doremi.html) |
| **Paper** | [DeMix arXiv:2602.00747](https://arxiv.org/html/2602.00747v1) |
| **Blog** | [Michael Brenndoerfer — Data Mixing](https://mbrenndoerfer.com/writing/data-mixing-domain-proportions-quality-weighting-optimal) |

---

## Related Topics

- [Model-Collapse.md](./Model-Collapse.md) — Training on synthetic data leads to distribution drift
