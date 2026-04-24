# Benchmark-Leak Detection

## 📌 Overview

**Benchmark-leak detection** identifies when LLM training data contains evaluation benchmark content, leading to distorted/inflated performance metrics. Methods range from n-gram matching to black-box detection when training data is inaccessible.

> *"Exposure of a large language model to benchmark data during the training process leads to distorted evaluation results."*
> — [Benchmark Data Contamination Survey arXiv:2406.04244](https://arxiv.org/html/2406.04244v1)

---

## Severity Levels

| Level | Description | Detection Difficulty |
|-------|-------------|---------------------|
| **Semantic** | Same topic/source content | Hard |
| **Information** | Metadata, label distributions | Moderate |
| **Data** | Benchmark content without labels | Moderate |
| **Label** | Complete exposure with answers | Easy |

> *"Severity increases from semantic → label level, but **detection complexity inversely correlates** with severity."*
> — [Benchmark Contamination Survey](https://arxiv.org/html/2406.04244v1)

---

## Matching-Based Methods

### N-gram Overlap

| Standard | Threshold |
|----------|-----------|
| GPT-3 (2020) | 13-gram overlap |
| GPT-4 | 50-character overlap |

> *"13-gram overlap = contamination signal."*
> — [GPT-3 Paper](https://arxiv.org/abs/2005.14165)

### Min-K% Prob

> *"Uses low-probability outlier words to detect training data; 7.4% improvement over baselines."*
> — [Min-K% ICLR 2024](https://arxiv.org/abs/2403.19833)

### Min-K%++ (ICLR 2025)

> *"Normalizes token probability with vocabulary statistics; 6.2-10.5% improvement."*
> — [Min-K%++ arXiv:2404.02936](https://arxiv.org/html/2404.02936v1)

---

## Comparison-Based Methods

| Method | Description |
|--------|-------------|
| **Perplexity-Based** | Contaminated models assign lower perplexity to test examples |
| **Sequential Analysis** | Assess sequence alignment with evaluation dataset |
| **Chronological Analysis** | Gauge performance on dated datasets |

---

## Black-Box Detection Methods

No training data access required.

### TS-Guessing Protocol (arXiv:2311.09783)

> *"Masks incorrect answers in MCQs → model fills gaps. ChatGPT could precisely predict missing incorrect choices in MMLU test set with **57% EM rate**."*
> — [TS-Guessing arXiv:2311.09783](https://arxiv.org/html/2311.09783v2)

### Time Travel in LLMs (ICLR 2024)

> *"Uses 'guided instruction' prompts with dataset names. Achieves **92-100% detection accuracy**."*
> — [Time Travel ICLR 2024](https://openreview.net/forum?id=2Rwq6c3tvr)

> *"Coax LLMs into outputting their memorized data."*
> — [Time Travel Paper](https://openreview.net/forum?id=2Rwq6c3tvr)

### CoDeC — Contamination Detection via Context (arXiv:2510.27055)

> *"Models respond differently: ICL *improves* unseen data but *reduces* confidence on contaminated data."*
> — [CoDeC arXiv:2510.27055](https://arxiv.org/html/2510.27055v1)

> *"Achieves **99.9% AUC** vs ~76-90% for baselines."*
> — [CoDeC Paper](https://arxiv.org/html/2510.27055v1)

> *"Key property: dataset-agnostic and parameter-free."*
> — [CoDeC Paper](https://arxiv.org/html/2510.27055v1)

### Proving Test Set Contamination (ICLR 2024)

> *"Exact false positive guarantees without access to pretraining data or weights."*
> — [Black-Box Contamination ICLR 2024](https://openreview.net/forum?id=KS8mIvetg2)

Leverages exchangeability: canonically ordered benchmarks should be equally likely vs shuffled.

---

## Watermarking-Based Detection (Proactive)

> *"Embed cryptographic watermarks into benchmarks *before* release. Subtle rephrasing via language model."*
> — [Watermarking Detection ICLR 2026](https://openreview.net/forum?id=WFGxFzFDmQ)

> *"Detects watermark signal in suspect models. Achieves **p-value < 10^-5** for 5% performance gain on 5000 MMLU questions."*
> — [Watermarking Paper](https://openreview.net/forum?id=WFGxFzFDmQ)

---

## Data Contamination Quiz (DCQ)

> *"Multiple-choice questions with 3 perturbed versions. LLM identifies original instance among perturbed ones. Reveals higher contamination levels than other methods."*
> — [DCQ Paper](https://aclanthology.org/2024.findings-emnlp.118.pdf)

---

## Additional Methods

| Method | Description | Reference |
|--------|-------------|-----------|
| **Kernel Divergence Score (KDS)** | Divergence between kernel similarity matrices before/after fine-tuning | ICML 2025 |
| **Reference-Free MIA** | Uses only model's own outputs | SimMIA 2026 |
| **ReCaLL** | Relative conditional log-likelihoods; SOTA on WikiMIA | EMNLP 2024 |

---

## Summary Table

| Method | Black-Box? | Key Metric |
|--------|-----------|-----------|
| **CoDeC** | ✅ | **99.9% AUC** |
| **Time Travel** | ✅ | **92-100% accuracy** |
| **Min-K%++** | ❌ | 6.2-10.5% improvement |
| **TS-Guessing** | ✅ | 57% EM on MMLU |
| **Watermarking** | ✅ | p < 10^-5 |
| **Black-Box Proof** | ✅ | False positive guarantees |

---

## Sources

| Type | Citation |
|------|----------|
| **Survey** | [Benchmark Data Contamination Survey arXiv:2406.04244](https://arxiv.org/html/2406.04244v1) |
| **Paper** | [Min-K% ICLR 2024](https://arxiv.org/abs/2403.19833) |
| **Paper** | [Min-K%++ arXiv:2404.02936](https://arxiv.org/html/2404.02936v1) |
| **Paper** | [TS-Guessing arXiv:2311.09783](https://arxiv.org/html/2311.09783v2) |
| **Paper** | [CoDeC arXiv:2510.27055](https://arxiv.org/html/2510.27055v1) |
| **Paper** | [Time Travel ICLR 2024](https://openreview.net/forum?id=2Rwq6c3tvr) |
| **Paper** | [Black-Box Contamination ICLR 2024](https://openreview.net/forum?id=KS8mIvetg2) |
| **Paper** | [Watermarking Detection ICLR 2026](https://openreview.net/forum?id=WFGxFzFDmQ) |
| **Awesome List** | [github.com/lyy1994/awesome-data-contamination](https://github.com/lyy1994/awesome-data-contamination) |
