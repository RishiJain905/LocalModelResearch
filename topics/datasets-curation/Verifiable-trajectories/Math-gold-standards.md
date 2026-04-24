# Math-gold-standards

## 📌 Overview

| Field | Value |
|-------|-------|
| **Also Known As** | Verified Math Benchmarks, Process Supervision Datasets |
| **Key Advantage** | Objective ground-truth answers for RL training |
| **Core Distinction** | Final-answer-only vs. step-by-step annotated datasets |
| **Latest Trend** | Process reward datasets (PRM800K, ProcessBench) |

---

## What Are Math Gold Standards?

Math gold standard datasets are **curated, verified math problem sets** with ground-truth answers used for training and evaluating reasoning models. Unlike subjective preference data, math answers are **objectively correct or verifiable**, enabling automatic RL reward signals.

---

## Major Datasets

### MATH — Hendrycks et al. (NeurIPS 2021)

| Field | Value |
|-------|-------|
| URL | [github.com/hendrycks/math](https://github.com/hendrycks/math) |
| arXiv | [2103.03874](https://arxiv.org/abs/2103.03874) |
| Problems | 12,500 competition math problems |
| Sources | AMC 10, AMC 12, AIME |
| Format | Step-by-step solutions + final boxed answers |

### GSM8K — Cobbe et al. (OpenAI, 2021)

| Field | Value |
|-------|-------|
| URL | [github.com/openai/grade-school-math](https://github.com/openai/grade-school-math) |
| arXiv | [2110.14168](https://arxiv.org/abs/2110.14168) |
| Problems | ~8,500 grade school math word problems |
| Split | 7,473 train / 1,319 test |

### GSM8K-KO (Korean variant)

- HAE-RAE Math 8K (HRM8K): 8,011 bilingual grade school math instances

### MMLU-Math

- Subset of MMLU by Hendrycks et al., multiple-choice format
- Released September 2020

### PRM800K — OpenAI (2023)

| Field | Value |
|-------|-------|
| URL | [github.com/openai/prm800k](https://github.com/openai/prm800k) |
| arXiv | [2305.20050](https://arxiv.org/abs/2305.20050) |
| Labels | 800,000 step-level correctness labels |
| Data | LLM-generated MATH solutions annotated by humans |

> "Let's Verify Step by Step" (2023) — shows process supervision significantly outperforms outcome-only supervision.

### ProcessBench — ACL 2025

| Field | Value |
|-------|-------|
| arXiv | [2412.06559](https://arxiv.org/abs/2412.06559) |
| Test Cases | 3,400 for step-level error identification |

### MathVista

| Field | Value |
|-------|-------|
| URL | [mathvista.github.io](https://mathvista.github.io/) |
| arXiv | [2310.02255](https://arxiv.org/abs/2310.02255) |
| Examples | 6,141 multimodal examples from 28 datasets |

### CAMEL Math

| Field | Value |
|-------|-------|
| URL | [huggingface.co/datasets/camel-ai/math](https://huggingface.co/datasets/camel-ai/math) |

- Step-by-step executable rationale

### DART-Math — NeurIPS 2024

| Field | Value |
|-------|-------|
| Key Feature | Difficulty-Aware Rejection Sampling |

- Addresses bias in vanilla rejection sampling

---

## Process vs. Outcome Supervision

### The Critical Distinction

| Supervision Type | Feedback | Used By |
|----------------|----------|---------|
| **Outcome** | Final answer correct/incorrect only | Standard RLVR, ORM |
| **Process** | Step-by-step correctness labels | PRM800K, ProcessBench |

### Why Process Supervision Wins

> "Let's Verify Step by Step" (OpenAI, 2023) — process supervision achieves **78.2%** on MATH (best-of-1860) vs. **72.4%** with outcome-only supervision.

---

## Ground-Truth Format Comparison

| Format | Supports ORM | Supports PRM | Example |
|--------|-------------|-------------|---------|
| **Final answer only** | ✅ | ❌ | GSM8K, MMLU-Math |
| **Step-by-step + final** | ✅ | ✅ | MATH, CAMEL Math |
| **Step-level annotations** | ✅ | ✅ | PRM800K, ProcessBench |

Step-by-step solutions enable **process reward models (PRM)** — dense step-level credit assignment rather than sparse outcome-only signals.

---

## Construction & Quality Methodology

### MATH Construction

> "Competition math problems from AMC 10, AMC 12, AIME — each with step-by-step solutions manually written and final answers boxed for automatic verification."

### Active Learning Efficiency (PRM800K)

> "OpenAI's strategy of targeting 'convincing wrong-answer solutions' yields **~2.6× data efficiency**."

### Data Contamination Risk

> "Models can score **44.5 percentage points higher** on contaminated subsets."

### Inter-Rater Agreement

- AgentProcessBench reports **89.1% expert agreement** (~10% disagreement on edge cases)

---

## Role in Verifiable Trajectory Training

Math gold standards feed into RLVR training loops:

1. **Dataset** → ground-truth problems with verifiable answers
2. **Model** → generates reasoning trajectories
3. **Reward** → binary match against final answer (ORM) or step correctness (PRM)
4. **Update** → GRPO/PPO policy update

### Why Math Is Ideal for RLVR

| Property | Math | Code | Natural Language |
|----------|------|------|------------------|
| Ground-truth | ✅ Binary verifiable | ✅ Test-based | ❌ Subjective |
| Step structure | ✅ Chain-of-thought | ✅ Line-by-line | ⚠️ Fuzzy |
| Automatic verification | ✅ String match | ✅ Execution | ❌ Requires human |

---

## Summary Data Points

| Dataset | Problems | Format | Step-level |
|---------|----------|--------|-----------|
| MATH | 12,500 | Solutions + boxed | ⚠️ Partial |
| GSM8K | 8,500 | Final answer | ❌ |
| PRM800K | 800K labels | Step annotations | ✅ |
| ProcessBench | 3,400 | Error identification | ✅ |
| MathVista | 6,141 | Multimodal | ⚠️ |
| DART-Math | — | Difficulty-aware | ⚠️ |

---

## Related Topics

- [RLVR.md](./RLVR.md) — Reinforcement Learning with Verifiable Rewards
- [Code-Verification.md](./Code-Verification.md) — Execution-based code evaluation
- [Verifiable-trajectories](./README.md) — Parent folder overview

---

## References

- [MATH — Hendrycks et al.](https://github.com/hendrycks/math)
- [GSM8K — OpenAI](https://github.com/openai/grade-school-math)
- [PRM800K — OpenAI](https://github.com/openai/prm800k)
- [ProcessBench — arXiv:2412.06559](https://arxiv.org/abs/2412.06559)
- [MathVista](https://mathvista.github.io/)
- [CAMEL Math](https://huggingface.co/datasets/camel-ai/math)
- ["Let's Verify Step by Step" — OpenAI 2023](https://arxiv.org/abs/2305.20050)
