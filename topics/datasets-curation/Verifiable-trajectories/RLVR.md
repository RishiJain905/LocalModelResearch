# RLVR

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Reinforcement Learning with Verifiable Rewards |
| **Also Known As** | RLVR, RL with Rule-Based Rewards |
| **Key Algorithm** | GRPO (Group Relative Policy Optimization) |
| **Key Debate** | Sampling efficiency vs. true reasoning capacity |
| **Awards** | NeurIPS 2025 Best Paper Runner-Up |

> "Reinforcement Learning with Verifiable Rewards is among the leading training strategies for injecting learning signals into LLMs, successfully employed by models such as DeepSeek R1 and TulU 3."

— [Label Studio](https://labelstud.io/blog/reinforcement-learning-from-verifiable-rewards/)

---

## What Is RLVR?

RLVR trains language models with **binary (0/1) rule-based rewards** that can be automatically verified — math answer matching, code execution results — rather than subjective human preference labels used in RLHF.

**Core mechanism:**
- At its core, Verifiable Rewards are simple functions that provide a clear-cut, binary ground truth signal — typically a 1 (correct) or 0 (incorrect) — to indicate whether a model output meets a predefined correctness criterion.

---

## Key Papers

### DeepSeek-R1 (Nature 2025)

| Field | Value |
|-------|-------|
| URL | [nature.com](https://www.nature.com/articles/s41586-025-09422-z) |
| DOI | 10.1038/s41586-025-09422-z |
| Published | 17 September 2025 |
| Authors | DeepSeek-AI Team |

> "The reasoning abilities of large language models (LLMs) can be incentivized through pure reinforcement learning (RL), eliminating the need for human-labeled reasoning trajectories."

**Key Results:**
| Benchmark | Score |
|-----------|-------|
| R1-Zero AIME 2024 | 71% (from 15.6% baseline) |
| R1 AIME 2024 | 79.8% |
| R1 MATH-500 | 97.3% |

### Limit of RLVR (Tsinghua/SJTU — NeurIPS 2025 Best Paper Runner-Up)

| Field | Value |
|-------|-------|
| URL | [limit-of-rlvr.github.io](https://limit-of-rlvr.github.io/) |
| arXiv | [2504.13837](https://arxiv.org/abs/2504.13837) |
| Code | [LeapLabTHU/limit-of-RLVR](https://github.com/LeapLabTHU/limit-of-RLVR) |
| Awards | NeurIPS 2025 Best Paper Runner-Up, ICML 2025 Workshop AI4Math Best Paper |

> "Crucially, all correct solutions from RL-trained models already exist in the base model distribution, proving RLVR enhances sampling efficiency, not reasoning capacity, while inadvertently shrinking the solution space."

> "If RL training truly expands reasoning, we would expect the RL model to solve more such problems than the base. However, we observe the opposite."

**Core Claims:**
- **0% problems solved exclusively by RLVR** — every problem RL model can solve, base model could also solve with enough samples
- RL trained models perform **WORSE** than base models at high pass@k
- RLVR enhances **sampling efficiency**, not reasoning capacity

### The Debate — Two-Stage Dynamic View (arXiv 2510.04028)

| Field | Value |
|-------|-------|
| URL | [arXiv:2510.04028](https://arxiv.org/html/2510.04028v1) |

> "Under the Pass at k metric, raising the probability of at least one correct action above 1/k corresponds to an expansion of the reasoning capability boundary."

> "Premature termination of training explains why some studies observe only shrinkage."

**Two-Stage Dynamic View:**
1. **Stage 1 (Exploitation):** Early training narrows capability boundary — over-reinforcing high-probability solutions
2. **Stage 2 (Exploration):** Prolonged training can expand boundary — redirecting to low-probability high-potential tokens

---

## How GRPO Works

1. **Group Sampling:** Generate multiple candidate responses for each prompt
2. **Rule-Based Scoring:** Apply rewards (math string match, code execution)
3. **GRPO Update:** Increase probability of above-average responses, decrease below-average

> "GRPO fine-tunes the model directly based on the output of the reward functions, without the need to collect preference data."

— [DataCamp](https://www.datacamp.com/blog/what-is-grpo-group-relative-policy-optimization)

---

## Verifiable Reward Types

| Type | Mechanism |
|------|-----------|
| **Mathematical Correctness** | Binary 1 (correct) or 0 (incorrect) based on string match |
| **Code Execution** | +1 all tests pass, -1 any test fails, -0.2 no valid code |
| **Format/Instruction** | String matching for keywords, structured output templates |

---

## PRM vs ORM

| Model | Scope | Signal |
|-------|-------|--------|
| **ORM** (Outcome Reward Model) | Final answer only | Binary correct/incorrect |
| **PRM** (Process Reward Model) | Individual reasoning steps | Dense step-level scores |

> "A process reward model (PRM) is a model that assigns a score to individual steps in a reasoning chain, rather than evaluating only the final outcome."

— [Michael Brenndoerfer](https://mbrenndoerfer.com/writing/process-reward-models-prm-outcome-math-training-limitations)

### PRM Reward Hacking — Stop Summation (NeurIPS 2025)

> "The canonical summation-form credit assignment in reinforcement learning (RL), i.e. cumulative gamma-decayed future rewards, causes the LLM to hack steps with high rewards."

— [OpenReview](https://openreview.net/forum?id=3Sxby0hH1q)

**Solution:** Min-form credit assignment (PURE approach) — defines value function as minimum future rewards rather than sum.

---

## Benchmark Results

| Model | AIME 2024 | MATH-500 | Notes |
|-------|-----------|----------|-------|
| Base model (R1-Zero start) | 15.6% | — | Baseline |
| R1-Zero (pure RL) | 71% | — | No SFT |
| R1 | 79.8% | 97.3% | With SFT |
| R1-Zero (ARC-AGI-1) | 14% | — | |
| R1 (ARC-AGI-1) | 15.8% | — | |

---

## The Core Debate

### Tsinghua/SJTU Position (Limit of RLVR)
- All correct solutions already exist in base model distribution
- RLVR only improves **sampling efficiency** — not reasoning capacity
- 0% of problems solved **exclusively** by RLVR

### Counterargument
- Production models (o1, o3) show genuine capability improvements at scale
- Two-stage theory: prolonged training can expand, not just narrow

### NeurIPS 2025 Recognition
- Limit of RLVR received **perfect scores (6,6,6,6)** — named Best Paper Runner-Up

---

## Related Topics

- [Code-Verification.md](./Code-Verification.md) — Execution-based code evaluation
- [Math-gold-standards.md](./Math-gold-standards.md) — Math datasets for verifiable rewards
- [Verifiable-trajectories](./README.md) — Parent folder overview

---

## References

- [DeepSeek-R1 (Nature 2025)](https://www.nature.com/articles/s41586-025-09422-z)
- [Limit of RLVR — Tsinghua](https://arxiv.org/abs/2504.13837)
- [Limit of RLVR — GitHub](https://github.com/LeapLabTHU/limit-of-RLVR)
- [GRPO Paper — arXiv:2503.06639](https://arxiv.org/abs/2503.06639)
- [Two-Stage Dynamic View — arXiv:2510.04028](https://arxiv.org/html/2510.04028v1)
- [Label Studio — RLVR Intro](https://labelstud.io/blog/reinforcement-learning-from-verifiable-rewards/)
- [DataCamp — GRPO Explained](https://www.datacamp.com/blog/what-is-grpo-group-relative-policy-optimization)
- [NeurIPS Award Announcement](https://x.com/jiqizhixin/status/1987710546674856051)
- [Stop Summation / PURE — NeurIPS 2025](https://openreview.net/forum?id=3Sxby0hH1q)
- [PRM Limitations — Michael Brenndoerfer](https://mbrenndoerfer.com/writing/process-reward-models-prm-outcome-math-training-limitations)
