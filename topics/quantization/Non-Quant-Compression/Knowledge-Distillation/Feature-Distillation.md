# Feature Distillation — Hidden State / Intermediate Layer Distillation

> **Key Papers:** [FitNet ICLR 2015](https://arxiv.org/abs/1412.6550) · [Attention Transfer ICLR 2017](https://arxiv.org/abs/1612.03928) · [MiniLM NeurIPS 2020](https://arxiv.org/abs/2002.10957) · [Hidden State Matching ICLR 2025](https://openreview.net/forum?id=IcVSKhVpKu) · [Flex-KD arXiv:2507.10155](https://arxiv.org/abs/2507.10155) · [TED ICML 2023](https://arxiv.org/abs/2210.01351)  
> **GitHub:** [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm) · [x-labs-xyz/feature-distillation](https://github.com/x-labs-xyz/feature-distillation) · [cliang1453/task-aware-distillation](https://github.com/cliang1453/task-aware-distillation) · [szagoruyko/attention-transfer](https://github.com/szagoruyko/attention-transfer)

---

## Overview

Feature distillation transfers knowledge from a teacher model to a student by matching **intermediate representations** (hidden states, attention maps), not just outputs. This captures deeper structural knowledge about how the teacher processes information, enabling more effective compression than logit-only distillation.

| Aspect | Details |
|--------|---------|
| **Method** | Match hidden states, attention maps, or intermediate representations |
| **Challenge** | Teacher/student often have different hidden dimensions |
| **Key Innovation** | CKA for dimension-agnostic matching; attention transfer; value-relation distillation |
| **vs. Logit KD** | Captures "how" not just "what" — richer representation knowledge |

---

## Key Methods

### 1. FitNet — Hints for Thin Deep Nets (ICLR 2015)

> **Paper:** [arXiv:1412.6550](https://arxiv.org/abs/1412.6550) · **GitHub:** (third-party implementations)

**Pioneering Work:** First to use intermediate layer hints for knowledge distillation.

**Key Quote:**
> *"We propose a new technique - FitNet - to train thin deep networks that are not only deep but also thin, to learn intermediate representations of a teacher network."*

**Method:** Guided layer training where a regressor (guided layer) maps student features to match teacher hints:

$$\mathcal{L}_{HT}(W_{Guided}, W_r) = \frac{1}{2} \|u_h(x; W_{Hint}) - r(v_g(x; W_{Guided}); W_r)\|^2$$

### 2. Attention Transfer (ICLR 2017)

> **Paper:** [arXiv:1612.03928](https://arxiv.org/abs/1612.03928) · **GitHub:** [szagoruyko/attention-transfer](https://github.com/szegokar/attention-transfer)

**Key Idea:** Transfer attention maps from teacher to student, not just output distributions.

> *"Paying More Attention to Attention improves the performance of convolutional neural networks."*

**Attention Map:** $F_{att}(Q, K) = \sum_i \frac{Q_i \cdot K_i}{\sqrt{d_k}}$

### 3. MiniLM — Deep Self-Attention Distillation (NeurIPS 2020)

> **Paper:** [arXiv:2002.10957](https://arxiv.org/abs/2002.10957) · **GitHub:** [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm)  
> **Authors:** Wang et al. (Microsoft Research)

**Key Innovation:** Distills the **last layer's self-attention module** of the teacher model, avoiding the need for layer-to-layer mapping between teacher and student.

**Quote:**
> *"Deep self-attention distillation is all you need."*

**Two Components:**
1. **Self-Attention Distribution Transfer** — KL-divergence between teacher and student attention maps
2. **Value-Relation Transfer** — Scaled dot-product between values ($V \cdot V^T$) captures fine-grained knowledge

**Benchmark Results:**

| Model | SQuAD 2.0 F1 | MNLI-m | Speedup |
|-------|-------------|---------|---------|
| BERT-BASE (Teacher) | 76.8 | 84.5 | 1.0× |
| TinyBERT (6×768) | 73.1 | 83.5 | — |
| **MiniLM (6×768)** | **76.4** | **84.0** | **2.0×** |
| MiniLM (6×384) | 72.4 | 82.2 | 5.3× |

### 4. Hidden State Matching with CKA (ICLR 2025)

> **Paper:** [OpenReview](https://openreview.net/forum?id=IcVSKhVpKu) · **GitHub:** [x-labs-xyz/feature-distillation](https://github.com/x-labs-xyz/feature-distillation)

**Problem:** Traditional cosine loss restricts student dimensionality to teacher's, limiting compression ratios.

**Solution:** Uses **Centered Kernel Alignment (CKA)** to match hidden states of different dimensionality.

> *"Hidden State Matching is shown to improve knowledge distillation of language models by encouraging similarity between a student and its teacher's hidden states."*

**Key Features:**
- Works with encoder-decoder (BART, mBART, T5) and encoder-only (BERT) architectures
- No pretrained student required — can synthesize new students from scratch
- Scales to students smaller than current methods

### 5. Flex-KD — Task-Based Flexible Feature Distillation (arXiv 2025)

> **Paper:** [arXiv:2507.10155](https://arxiv.org/abs/2507.10155)

**Problem:** Traditional feature distillation requires teacher and student to share same hidden size (d_T = d_S), or requires linear projector that degrades performance.

**Key Insight:**
> *"Only a subset of LLM components contribute significantly to a specific downstream task."*

**Method:**
1. **Task-Relevant Units Localization** — Uses gradient attribution to identify important neurons
2. **Cross-Correlation Loss** — More effective than MSE/cosine for feature space relationships

**Results (up to 2.1% improvement over Projector):**

| Task Type | Improvement |
|-----------|-------------|
| Classification (IMDB) | +1.08% over Projector |
| Instruction-Following | +2.1% over Projector |
| Summarization (ROUGE-L) | +3.75% over Projector |

### 6. TED — Task-aware Layerwise Distillation (ICML 2023)

> **Paper:** [arXiv:2210.01351](https://arxiv.org/abs/2210.01351) · **GitHub:** [cliang1453/task-aware-distillation](https://github.com/cliang1453/task-aware-distillation)

**Method:** Two-stage approach:
1. **Stage I:** Learn task-aware filters for teacher/student
2. **Stage II:** Distill student to match teacher's filtered outputs

**Loss:**
```
L = pred_loss + kl_alpha * kl_loss + mse_alpha * layer_averaged_mse_loss
```

**Results (DeBERTaV3-base → DeBERTaV3-xs):**

| Task | Teacher | Student (TED) |
|------|---------|---------------|
| MNLI | 90.6 | 88.8 |
| QQP | 89.9 | 89.5 |
| SST-2 | 96.4 | 94.4 |

### 7. Revisiting Intermediate Layer Distillation (EACL 2023 Findings)

> **Paper:** [arXiv:2302.01530](https://arxiv.org/abs/2302.01530)

**Finding:** Existing ILD methods are prone to overfitting to training datasets, although they transfer more information than output-only distillation.

---

## Key Technical Terms

| Term | Definition |
|------|------------|
| **CKA** | Centered Kernel Alignment — similarity measure for representations of different dimensions |
| **Value-Relation** | Scaled dot-product between value matrices ($V \cdot V^T$) |
| **Hint Layer** | Teacher intermediate layer used to guide student training |
| **Guided Layer** | Student layer trained to match teacher hints via regressor |
| **Task-Relevant Units** | Neurons identified via gradient attribution as important for a specific task |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **FitNet** | 2014 | ICLR 2015 | [arXiv](https://arxiv.org/abs/1412.6550) | Hint-based intermediate layer training |
| **Attention Transfer** | 2016 | ICLR 2017 | [arXiv](https://arxiv.org/abs/1612.03928) | Attention map transfer for CNNs |
| **MiniLM** | 2020 | NeurIPS 2020 | [arXiv](https://arxiv.org/abs/2002.10957) | Last-layer self-attention + value-relation distillation |
| **TED** | 2022 | ICML 2023 | [arXiv](https://arxiv.org/abs/2210.01351) | Task-aware layerwise filters for distillation |
| **Revisiting ILD** | 2023 | EACL 2023 | [arXiv](https://arxiv.org/abs/2302.01530) | Overfitting issue in intermediate layer distillation |
| **Hidden State Matching (CKA)** | 2025 | ICLR 2025 | [OpenReview](https://openreview.net/forum?id=IcVSKhVpKu) | Dimension-agnostic hidden state alignment |
| **Flex-KD** | 2025 | — | [arXiv](https://arxiv.org/abs/2507.10155) | Task-relevant neuron selection via gradients |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm) | MiniLM self-attention distillation |
| [x-labs-xyz/feature-distillation](https://github.com/x-labs-xyz/feature-distillation) | CKA-based hidden state matching |
| [cliang1453/task-aware-distillation](https://github.com/cliang1453/task-aware-distillation) | Task-aware layerwise distillation (TED) |
| [szagoruyko/attention-transfer](https://github.com/szegokar/attention-transfer) | Attention transfer for CNNs |
| [arcee-ai/DistillKit](https://github.com/arcee-ai/DistillKit) | LLM distillation toolkit |
| [Tebmer/Awesome-Knowledge-Distillation-of-LLMs](https://github.com/Tebmer/Awesome-Knowledge-Distillation-of-LLMs) | Survey of LLM KD techniques |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Transfers structural knowledge of "how" the teacher processes information, not just "what" it outputs |
| **Key innovation** | CKA for dimension-agnostic matching; value-relation transfer; task-relevant neuron selection |
| **Best for** | High compression ratios where hidden representations diverge significantly from teacher |
| **Challenge** | Different hidden dimensions require projectors or CKA; layer-to-layer mapping complexity |
| **vs. Logit KD** | More informative but more complex; often combined with logit KD in practice |

---

*Last updated: 2026-04-19*
