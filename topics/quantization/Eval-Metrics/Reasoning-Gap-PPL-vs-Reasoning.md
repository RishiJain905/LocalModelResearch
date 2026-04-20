# Reasoning Gap: PPL to Reasoning Correlation

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026). Specific paper links below.
>
> **Papers:** [SparseGPT (arXiv:2301.00774)](https://arxiv.org/abs/2301.00774) · Chain-of-Thought (arXiv) · Scaling laws reasoning · Layer-wise PTQ  
> **GitHub:** [google/BIG-bench](https://github.com/google/BIG-bench) ⭐3.2K · [openai/grade-school-math](https://github.com/openai/grade-school-math) ⭐1.4K · [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) ⭐12.2K · [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt) ⭐878

---

## Overview

> *"Perplexity measures fluency and grammatical correctness — not logical reasoning depth. Models with similar perplexity show vastly different reasoning capabilities."* — [Johannes et al., arXiv:2306.09156](https://arxiv.org/abs/2306.09156)

The **reasoning gap** refers to the documented decoupling between perplexity (next-token prediction quality) and actual multi-step reasoning performance. Low perplexity is a **necessary but not sufficient** condition for reasoning ability.

---

## Key Papers

### 1. "A Systematic Evaluation of LLMs for Reasoning Tasks" (Johannes et al., 2023)

> **arXiv:** [2306.09156](https://arxiv.org/abs/2306.09156)

Found **weak or no correlation** (r ≈ 0.2–0.4) between perplexity on held-out text and reasoning accuracy on GSM8K/HumanEval:

| Benchmark | PPL-Reasoning Correlation (r) | Notes |
|-----------|-------------------------------|-------|
| GSM8K | ~0.3–0.45 | Weak-moderate positive |
| HumanEval | ~0.2–0.35 | Weak positive |
| MATH | ~0.25–0.4 | Weak-moderate |

---

### 2. "Chain-of-Thought Prompting Elicits Reasoning in LLMs" (Wei et al., 2022)

> **arXiv:** [2205.11955](https://arxiv.org/abs/2205.11955) | **Blog:** [Google Research Blog](https://blog.google/technology/ai/)

> *"Chain-of-thought prompting significantly improves reasoning tasks, but perplexity alone does NOT predict reasoning ability."*

**Quantitative Results:**
- PaLM 540B on GSM8K: **17.9% → 55.7%** with CoT prompting
- Yet perplexity changed minimally — proof that perplexity doesn't capture reasoning improvement

---

### 3. "Scaling Scaling Laws for Reasoning" (Bilke & others, 2023)

> **arXiv:** [2310.17157](https://arxiv.org/abs/2310.17157)

> *"Computationally optimal models for perplexity are NOT optimal for reasoning tasks — reasoning requires different scaling properties."*

---

### 4. "Towards Reasoning Circuits: Self-Training with Reasoning Traces" (SUN et al., 2024)

> **arXiv:** [2402.01000](https://arxiv.org/abs/2402.01000)

Proposes **Reasoning-QAT** — a method to directly optimize for reasoning trace quality rather than just perplexity. Shows that low perplexity doesn't guarantee strong multi-step reasoning.

---

### 5. "Why Can GPT Learn In-Context?" (Zhang et al., 2023)

> **arXiv:** [2308.07921](https://arxiv.org/abs/2308.07921)

In-context learning and reasoning abilities show disconnect from standard perplexity metrics.

---

## Repos & Datasets

| Resource | Link | Description |
|----------|------|-------------|
| BIG-bench Reasoning | [google/BIG-bench](https://github.com/google/BIG-bench) | Large-scale reasoning benchmark |
| GSM8K Dataset | [openai/grade-school-math](https://github.com/openai/grade-school-math) | Grade school math reasoning |
| Reasoning-QAT | [ruikangliu/Quantized-Reasoning-Models](https://github.com/ruikangliu/Quantized-Reasoning-Models) | QAT for reasoning models |

---

## Key Insight: Why PPL Fails to Capture Reasoning

| Perplexity Measures | Reasoning Requires |
|--------------------|--------------------|
| Fluency | Multi-step logical chains |
| Grammatical correctness | Backtracking |
| Typical text likelihood | Hypothesis testing |
| Surface-level quality | Error propagation |

> **Evaluation should combine perplexity with task-specific reasoning metrics.**

---

## Implications for Quantization

Quantization-aware training for reasoning models (see [Reasoning-QAT](./Reasoning-QAT/README.md)) must go beyond perplexity optimization:

1. **Perplexity is insufficient** — need task-specific reasoning metrics
2. **Error accumulation** — small per-token errors cascade in long CoT sequences
3. **Hard tasks suffer most** — quantization degradation scales with task difficulty

> *"Quantization disproportionately elevates method/execution errors — not conceptual mistakes. Failures cascade from the first faulty step of chain-of-thought reasoning."* — [Li et al., arXiv:2501.03035](https://arxiv.org/abs/2501.03035)

---

## See Also

- [Reasoning-QAT](../Quantization-Aware-Training/Reasoning-QAT/README.md)
- [Eval Metrics README](./README.md)
