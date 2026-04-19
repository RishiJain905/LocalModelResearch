# Logit Distillation — Soft Target / Output Distillation

> **Foundational:** [Hinton et al. 2015 — "Distilling the Knowledge in a Neural Network"](https://arxiv.org/abs/1503.02531) · [DistilBERT (EMC @ NeurIPS 2019)](https://arxiv.org/abs/1910.01108) · [TinyBERT (Findings EMNLP 2020)](https://arxiv.org/abs/1909.10351)  
> **LLM-Focused:** [MiniLLM ICLR 2024](https://arxiv.org/abs/2306.08543) · [GKD ICLR 2024](https://arxiv.org/abs/2306.13649) · [ULD TMLR 2025](https://arxiv.org/abs/2402.12030) · [Logit Standardization CVPR 2024](https://arxiv.org/abs/2403.01427) · [SDFT ACL 2024](https://arxiv.org/abs/2402.13669)  
> **GitHub:** [microsoft/LMOps/minillm](https://github.com/microsoft/LMOps/tree/main/minillm) · [huawei-noah/TinyBERT](https://github.com/huawei-noah/Pretrained-Language-Model/tree/master/TinyBERT) · [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm)  
> **Blogs:** [Hugging Face — Introducing DistilBERT](https://medium.com/huggingface/smaller-faster-cheaper-lighter-introducing-dilbert-a-distilled-version-of-bert-8cf3380435b5) · [Knowledge Distillation: A Comprehensive Guide](https://nicks-cheke44.medium.com/knowledge-distillation-a-comprehensive-guide-ca0b703ba68a)

---

## Overview

> *"Many insects have a larval form that is optimized for extracting energy and nutrients from the environment and a completely different adult form that is optimized for the very different requirements of traveling and reproduction... The analogy with insects suggests that we should be willing to train very cumbersome models if that makes it easier to extract structure from the data."* — Hinton et al., [arXiv:1503.02531](https://arxiv.org/abs/1503.02531)

Logit distillation (soft target / output distillation) transfers knowledge from a teacher model to a student model by matching the teacher's output probability distributions, using **softmax temperature** to soften distributions and capture "dark knowledge."

| Aspect | Details |
|--------|---------|
| **Method** | KL divergence between teacher/student softmax distributions |
| **Key Parameter** | Temperature (T) — higher T = softer distributions |
| **Loss** | $\mathcal{L} = \alpha \cdot CE(labels) + \beta \cdot KL_{soft}(teacher\|student) \cdot T^2$ |
| **Core Insight** | Soft targets transfer relative token importance, not just correct answers |

---

## Core Method

### The Distillation Loss

```
L = α * CE(labels, student) + β * KL(softmax(student_logits/T) || softmax(teacher_logits/T)) * T²
```

**At high T:** Gradient scaling by T² ensures consistency between hard and soft targets.

**The "Dark Knowledge":** Even when the teacher assigns low probability to incorrect classes, those relative rankings contain rich structural information about the task.

---

## Key Methods

### 1. Original: Distilling the Knowledge in a Neural Network (NIPS 2014)

> **Paper:** [arXiv:1503.02531](https://arxiv.org/abs/1503.02531) · **Authors:** Geoffrey Hinton, Oriol Vinyals, Jeff Dean (Google)

**Key Quote:**
> *"If we use logits instead of softmax outputs as the targets for the transfer case, we can sometimes learn a better model by using a much smaller model."*

**Insight:** At high T, matching logits is equivalent to minimizing squared difference. At intermediate T, distillation pays less attention to highly negative logits.

### 2. DistilBERT — Distilled Version of BERT (EMC @ NeurIPS 2019)

> **Paper:** [arXiv:1910.01108](https://arxiv.org/abs/1910.01108) · **Authors:** Victor Sanh et al. (Hugging Face)

> *"We leverage knowledge distillation during the pre-training phase and show that it is possible to reduce the size of a BERT model by 40%, while retaining 97% of its language understanding capabilities and being 60% faster."*

**Triple Loss:**
1. **Distillation Loss** — KL divergence of soft targets
2. **Masked Language Modeling Loss** — standard MLM cross-entropy
3. **Cosine Embedding Loss** — on hidden states

**Results:** 40% smaller (66M params), 60% faster, 97% of BERT's performance on GLUE.

### 3. TinyBERT — Transformer Distillation for NLU (Findings EMNLP 2020)

> **Paper:** [arXiv:1909.10351](https://arxiv.org/abs/1909.10351) · **GitHub:** [huawei-noah/TinyBERT](https://github.com/huawei-noah/Pretrained-Language-Model/tree/master/TinyBERT)  
> **Authors:** Jiao et al. (Huawei Noah)

> *"We firstly propose a novel Transformer distillation method that is specially designed for knowledge distillation (KD) of the Transformer-based models."*

**Two-Stage Framework:**
1. **General Distillation** — pre-training phase
2. **Task-specific Distillation** — fine-tuning phase

**Results:** 7.5× smaller, 9.4× faster than BERT-base; competitive on SQuAD 1.1 (EM 79.7 vs BERT 80.7).

### 4. MiniLLM — Reverse KL Distillation for LLMs (ICLR 2024)

> **Paper:** [arXiv:2306.08543](https://arxiv.org/abs/2306.08543) · **GitHub:** [microsoft/LMOps/minillm](https://github.com/microsoft/LMOps/tree/main/minillm)  
> **Authors:** Gu et al. (Microsoft)

**Problem:** Standard KD minimizes forward KL[p||q], forcing student to cover all teacher modes, causing **void regions** (overestimated low-probability tokens).

**Key Quote:**
> *"Minimizing reverse KLD causes qθ to seek the major modes of p, and assign low probabilities to p's void regions. In LLM text generation, this means the student model avoids learning too many long-tail variants in the teacher model's distribution and focuses on the correctness of the generated contents."*

**Three Stabilization Strategies:**
1. Single-Step Decomposition — reduces variance in policy gradient
2. Teacher-Mixed Sampling — prevents reward hacking (α=0.2 default)
3. Length Normalization — eliminates length bias

**Results (GPT-2 1.5B → 120M):**

| Method | DollyEval GPT4 | R-L |
|--------|----------------|-----|
| Teacher | 58.4 | 27.6 |
| KD | 40.3 | 22.8 |
| **MiniLLM** | **44.7** | **24.6** |

### 5. GKD — Generalized Knowledge Distillation (ICLR 2024)

> **Paper:** [arXiv:2306.13649](https://arxiv.org/abs/2306.13649) · **Authors:** Agarwal et al. (Google DeepMind)

**Problem:** Distribution mismatch between training sequences (teacher-generated) and inference (student-generated).

> *"GKD allows the use of alternative loss functions between the student and teacher. This is critical when the student model lacks the expressivity to perfectly mimic the teacher's distribution."*

**Solution:** On-policy learning — trains student on its own self-generated sequences, with teacher providing feedback.

### 6. ULD — Universal Logit Distillation for Cross-Tokenizer KD (TMLR 2025)

> **Paper:** [arXiv:2402.12030](https://arxiv.org/abs/2402.12030) · **GitHub:** [Nicolas-BZRD/llm-recipes](https://github.com/Nicolas-BZRD/llm-recipes)

**Problem:** KL divergence requires teacher and student to share the same tokenizer — limits cross-architecture distillation.

**Solution:** **Universal Logit Distillation (ULD)** using Wasserstein-1 distance instead of KL divergence. Works across different tokenizers.

> *"How can we build a general distillation loss that leverages logits capacity while staying within a black box framework that is easy to implement?"*

**Result:** Pythia 410m with ULD matches Pythia 1b distilled with teacher-generated text on QED and DIALOGSum.

### 7. Logit Standardization (CVPR 2024 Highlight)

> **Paper:** [arXiv:2403.01427](https://arxiv.org/abs/2403.01427) · **GitHub:** [sunshangquan/logit-standardization-KD](https://github.com/sunshangquan/logit-standardization-KD)

**Problem:** Shared temperature causes implicit mandatory logit match and inauthentic performance indication.

**Solution:** **Z-score normalization** (logit standardization) before softmax. Adaptive temperature allocation.

> *"The ratio between the temperatures of student and teacher equals the ratio between the standard deviations of their predicted logits for a well-distilled student."*

### 8. SDFT — Self-Distillation Fine-Tuning (ACL 2024)

> **Paper:** [arXiv:2402.13669](https://arxiv.org/abs/2402.13669) · **GitHub:** [sail-sg/sdft](https://github.com/sail-sg/sdft)

**Problem:** Fine-tuning LLMs causes **catastrophic forgetting** — loss of general capabilities and safety alignment.

**Solution:** Model generates responses aligned with its own distribution, then fine-tunes on them.

> *"Increasing the proportion of distilled dataset for fine-tuning leads to a decrease in catastrophic forgetting."*

### 9. Sparse Logit Sampling (ACL 2025 Oral)

> **Paper:** [arXiv:2503.16870](https://arxiv.org/abs/2503.16870) · **GitHub:** [akhilkedia/RandomSamplingKD](https://github.com/akhilkedia/RandomSamplingKD)

**Problem:** Full logit KD is expensive for LLMs (vocabulary up to 100K tokens).

**Solution:** Importance-sampling-based method providing unbiased gradient estimates. Reduces logits per byte from 100K to 12.

> *"Naive approaches for sparse knowledge distillation such as caching Top-K probabilities, while intuitive, provide biased estimates of the gradient."*

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **Distilling the Knowledge** | 2014 | NIPS Workshop | [arXiv](https://arxiv.org/abs/1503.02531) | Foundational soft target / temperature KD |
| **DistilBERT** | 2019 | EMC @ NeurIPS | [arXiv](https://arxiv.org/abs/1910.01108) | Pre-training distillation of BERT, triple loss |
| **TinyBERT** | 2020 | Findings EMNLP | [arXiv](https://arxiv.org/abs/1909.10351) | Two-stage Transformer distillation |
| **MiniLM** | 2020 | NeurIPS | [arXiv](https://arxiv.org/abs/2002.10957) | Last-layer self-attention + value-relation distillation |
| **MiniLLM** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2306.08543) | Reverse KL for mode-seeking LLM distillation |
| **GKD** | 2023 | ICLR 2024 | [arXiv](https://arxiv.org/abs/2306.13649) | On-policy learning, self-generated sequences |
| **ULD** | 2024 | TMLR 2025 | [arXiv](https://arxiv.org/abs/2402.12030) | Wasserstein-1 distance for cross-tokenizer KD |
| **Logit Standardization** | 2024 | CVPR 2024 | [arXiv](https://arxiv.org/abs/2403.01427) | Z-score normalization before softmax |
| **SDFT** | 2024 | ACL 2024 | [arXiv](https://arxiv.org/abs/2402.13669) | Self-distillation to mitigate catastrophic forgetting |
| **Sparse Logit Sampling** | 2025 | ACL 2025 | [arXiv](https://arxiv.org/abs/2503.16870) | Importance sampling for efficient large-vocabulary KD |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [microsoft/LMOps/minillm](https://github.com/microsoft/LMOps/tree/main/minillm) | Reverse KL distillation (ICLR 2024) |
| [huawei-noah/TinyBERT](https://github.com/huawei-noah/Pretrained-Language-Model/tree/master/TinyBERT) | Two-stage Transformer distillation |
| [microsoft/unilm/minilm](https://github.com/microsoft/unilm/tree/master/minilm) | Self-attention + value-relation distillation |
| [sunshangquan/logit-standardization-KD](https://github.com/sunshangquan/logit-standardization-KD) | CVPR 2024 Logit Standardization |
| [akhilkedia/RandomSamplingKD](https://github.com/akhilkedia/RandomSamplingKD) | ACL 2025 Sparse Logit Sampling |
| [sail-sg/sdft](https://github.com/sail-sg/sdft) | Self-Distillation Fine-Tuning |
| [Nicolas-BZRD/llm-recipes](https://github.com/Nicolas-BZRD/llm-recipes) | Universal Logit Distillation |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Transfers "dark knowledge" from large teacher models to smaller student models via soft probability distributions |
| **Key innovation** | Temperature scaling to soften distributions; reverse KL (MiniLLM) for mode-seeking; ULD for cross-tokenizer |
| **Best for** | Compression with teacher-student architecture; generation quality transfer |
| **Key insight** | Soft targets carry more structure than hard labels — ranking of incorrect answers matters |

---

*Last updated: 2026-04-19*
