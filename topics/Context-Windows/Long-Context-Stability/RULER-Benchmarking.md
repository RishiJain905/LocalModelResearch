# RULER Benchmark

> **Paper:** [arXiv:2404.06654](https://arxiv.org/abs/2404.06654) (NVIDIA, COLM 2024 Spotlight)
> **GitHub:** ⭐1514 [NVIDIA/RULER](https://github.com/NVIDIA/RULER)
> **Leaderboard:** [llm-stats.com/benchmarks/ruler](https://llm-stats.com/benchmarks/ruler) · [HELM Long Context](https://crfm.stanford.edu/helm/long-context/latest/)
> **Extensions:** [OneRuler (Multilingual)](https://arxiv.org/abs/2503.01996) · [HELMET (Alternative)](https://arxiv.org/abs/2410.02694)

---

## Overview

> *"The needle-in-a-haystack (NIAH) test has been widely adopted to evaluate long-context language models. However, this simple retrieval-based test is indicative of only a superficial form of long-context understanding. RULER provides a more comprehensive evaluation with flexible configurations for sequence length and task complexity."* — Hsieh et al., 2024

RULER (What's the Real Context Size of Your Long-Context Language Models?) is a synthetic benchmark from NVIDIA that extends beyond simple NIAH retrieval to evaluate **4 categories of long-context capability** across configurable sequence lengths. It reveals that models claiming 128K+ context windows often have **effective context sizes far below their claims**.

---

## Task Categories

RULER includes **13 task configurations** across **4 categories**:

| Category | Task | Key Configurations |
|----------|------|--------------------|
| **Retrieval** | `niah` | `type_haystack`: repeat/essay/needle · `type_needle_k/v`: words/numbers/uuids · `num_needle_k`: int≥1 · `num_needle_v`: int≥1 · `num_needle_q`: int≥1 |
| **Multi-hop Tracing** | `variable_tracking` | `num_chains`: int≥1 (variable name-binding chains) · `num_hops`: int≥1 (times binding variable names) |
| **Aggregation** | `common_words_extraction` | `freq_cw`: int≥1 · `freq_ucw`: int≥1 · `num_cw`: int≥1 |
| **Aggregation** | `freq_words_extraction` | `alpha`: float>1.0 (distribution param; lower = harder; default 3) |
| **Question Answering** | `qa` | `dataset`: `squad` or `hotpotqa` |

**Evaluated context lengths:** 4K, 8K, 16K, 32K, 64K, 128K (and extended to 1M/2M on leaderboards)

---

## Key Finding: Claimed vs. Effective Context Length

> *"Despite achieving nearly perfect accuracy in the vanilla NIAH test, almost all models exhibit large performance drops as the context length increases. While these models all claim context sizes of 32K tokens or greater, only half of them can maintain satisfactory performance at the length of 32K."*

### Top Performers (Effective Length >128K)

| Model | Claimed | Effective | 4K | 8K | 16K | 32K | 64K | 128K | Avg |
|---|---|---|---|---|---|---|---|---|---|
| **Jamba-1.5-large** (94B/398B) | 256K | >128K | 96.7 | 96.6 | 96.4 | 96.0 | 95.4 | 95.1 | **96.0** |
| **Gemini-1.5-pro** | 1M | >128K | 96.7 | 95.8 | 96.0 | 95.9 | 95.9 | 94.4 | **95.8** |
| **Qwen2.5-14B-Instruct-1M** | 1M | >128K | 97.5 | 97.1 | 94.6 | 94.9 | 94.9 | 92.2 | **95.7** |
| **Qwen3-235B-A22B** | 128K | >128K | 97.7 | 97.2 | 96.4 | 95.1 | 93.3 | 90.6 | **95.0** |
| **Jamba-1.5-mini** (12B/52B) | 256K | >128K | 95.6 | 95.6 | 94.8 | 94.6 | 92.8 | 90.0 | **93.9** |

### Notable Gaps (Claimed >> Effective)

| Model | Claimed | Effective | Avg |
|---|---|---|---|
| **InternLM2.5** (7B) | 1M | **4K** | 80.9 |
| **LWM** (7B) | 1M | **<4K** | 72.8 |
| **GradientAI/Llama3** (70B) | 1M | **16K** | 86.5 |
| **GPT-4-1106-preview** | 128K | 64K | 91.6 |
| **Llama3.1** (70B) | 128K | 64K | 89.6 |
| **Mistral-Large-2411** (123B) | 128K | 64K | 86.0 |

---

## Paper

### RULER: What's the Real Context Size of Your Long-Context Language Models?

> **Paper:** [arXiv:2404.06654](https://arxiv.org/abs/2404.06654) · **Authors:** Cheng-Ping Hsieh, Simeng Sun, Samuel Kriman, Shantanu Acharya, Dima Rekesh, Fei Jia, Yang Zhang, Boris Ginsburg
> **Affiliation:** NVIDIA · **Venue:** COLM 2024 (Spotlight)

**Citation:**
```bibtex
@article{hsieh2024ruler,
  title={RULER: What's the Real Context Size of Your Long-Context Language Models?},
  author={Cheng-Ping Hsieh and Simeng Sun and Samuel Kriman and Shantanu Acharya and Dima Rekesh and Fei Jia and Yang Zhang and Boris Ginsburg},
  year={2024},
  journal={arXiv preprint arXiv:2404.06654},
}
```

**Key quote:**
> *"To provide a more comprehensive evaluation of long-context LMs, we create a new synthetic benchmark RULER with flexible configurations for customized sequence length and task complexity."*

---

## GitHub Repository

> ⭐**1514** · **101 commits** · Maintained by @hsiehjackson (Cheng-Ping Hsieh)

**Setup & Usage:**
```bash
# Docker
docker pull cphsieh/ruler:0.2.0

# Data download
cd scripts/data/synthetic/json/
python download_paulgraham_essay.py
bash download_qa_dataset.sh

# Supported frameworks: hf, vllm, trtllm, openai, gemini
bash run.sh YOUR_MODEL_NAME synthetic
```

**HuggingFace datasets built on RULER:**
- `tonychenxyz/ruler-full`
- `starbix/oneruler`
- `self-long/RULER-llama3-1M`

> *"This project is strictly for research purposes, and not an official product from NVIDIA."*

---

## Extension Benchmarks

### OneRuler (Multilingual)

> **Paper:** [arXiv:2503.01996](https://arxiv.org/abs/2503.01996) · **GitHub:** [mungg/OneRuler](https://github.com/mungg/OneRuler)

Extends RULER to **26 languages**, **7 synthetic tasks**, **4 context lengths** (8K–128K).

**Key finding:** English is only the **6th best language** (83.9% NIAH accuracy); Polish is #1 (88%). Adding a "none" option drops S-NIAH accuracy by **32% at 128K**.

### HELMET (Alternative/Successor)

> **Paper:** [arXiv:2410.02694](https://arxiv.org/abs/2410.02694) · **URL:** [princeton-nlp.github.io/HELMET](https://princeton-nlp.github.io/HELMET/)

> *"Synthetic tasks like NIAH do not reliably predict downstream performance."*

### Related Benchmarks

| Benchmark | Paper | Focus |
|-----------|-------|-------|
| **BABILong** | [arXiv:2406.07801](https://arxiv.org/abs/2406.07801) | Reasoning up to 10M tokens (NeurIPS 2024) |
| **LV-Eval** | [arXiv:2402.05136](https://arxiv.org/abs/2402.05136) | Multi-hop long-context evaluation |
| **Oolong** | [arXiv:2511.02817](https://arxiv.org/abs/2511.02817) | Long-context reasoning & aggregation |
| **LongProc** | [arXiv:2501.05414](https://arxiv.org/abs/2501.05414) | Long-form generation & procedure following |
| **InfiniteBench** | [arXiv:2402.13718](https://arxiv.org/abs/2402.13718) | Beyond 100K tokens |
| **LongMemEval** | [arXiv:2410.10813](https://arxiv.org/abs/2410.10813) | Long-term interactive memory (ICLR 2025) |

---

## Leaderboard Results

**[LLM Stats RULER Leaderboard](https://llm-stats.com/benchmarks/ruler):**
- Top 3: Nemotron 3 Super (0.917), Phi-3.5-MoE-instruct (0.871), Phi-3.5-mini-instruct (0.841)
- Sub-benchmarks: 4K, 8K, 16K, 32K, 64K, 128K, 512K, 1000K, 2048K
- Average score across all models: 0.877

**[HELM Long Context Leaderboard](https://crfm.stanford.edu/helm/long-context/latest/):**
- RULER SQuAD — open-ended single-hop QA
- RULER HotPotQA — open-ended multi-hop QA

---

## Stated Limitations

From the paper authors:

1. Only evaluates **13 task configurations** (selected because models perform well at ≤4K)
2. Did **not** include more complex tasks where models struggle at short context
3. Did **not** stress-test every model with harder configurations
4. **Cannot replace realistic tasks** — meant as a sanity-check with known upper bound performance
5. Welcomes community contributions of new tasks/categories

---

## Blog Posts & Discussions

| Title | Source | URL |
|-------|--------|-----|
| **Exposing True Context Capabilities** | Reddit r/LocalLLaMA | [Link](https://www.reddit.com/r/LocalLLaMA/comments/1c7f9b5/) |
| **How RULER Measures Context Length** | Analytics India Mag | [Link](https://analyticsindiamag.com/ai-news-updates/nvidia-introduces-ruler-to-measure-the-context-length-of-models/) |
| **RULER Benchmark Explained** | Medium (@techsachin) | [Link](https://medium.com/@techsachin/ruler-benchmark-to-evaluate-long-context-modeling-capabilities-of-language-models-7eb13a269e36/) |
| **HuggingFace Paper Page** | HuggingFace | [Link](https://huggingface.co/papers/2404.06654) (64 citing models, 3 datasets, 443 Spaces) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **What it tests** | 13 tasks across retrieval, multi-hop tracing, aggregation, and QA |
| **Key insight** | Models claiming 128K+ context often have effective context sizes far below claims |
| **Best performers** | Jamba-1.5-large, Gemini-1.5-pro, Qwen2.5-14B-1M (all >128K effective) |
| **Worst gaps** | InternLM2.5 (claims 1M, effective 4K), LWM (claims 1M, effective <4K) |
| **Extensions** | OneRuler (26 languages), HELMET (replaces synthetic with downstream tasks) |
| **Limitation** | Synthetic tasks only — doesn't fully predict real-world performance |

---

*Last updated: 2026-04-20*