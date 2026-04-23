# RLVR (Reinforcement Learning with Verifiable Rewards)

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Reinforcement Learning with Verifiable Rewards |
| **Type** | Post-training paradigm for LLMs |
| **Key Signal** | Deterministic binary reward (correct/incorrect) |
| **Dominant Algorithm** | GRPO (Group Relative Policy Optimization) |
| **Primary Reference** | [DeepSeek-R1 (arXiv:2501.12948)](https://arxiv.org/abs/2501.12948) |
| **Controversy** | Does RLVR expand reasoning capacity or just improve sampling efficiency? |

---

## Definitions

> "Verifiable Rewards are simple functions that provide a clear-cut, binary ground truth signal — typically a 1 (correct) or 0 (incorrect) — to indicate whether a model output meets a predefined correctness criterion."

— [Label Studio Blog](https://labelstud.io/blog/reinforcement-learning-from-verifiable-rewards/)

> "Reinforcement Learning with Verifiable Rewards (RLVR) is standard reinforcement learning with deterministic reward functions (the technique is not new, but applying it to LLM post-training at scale is)."

— [Promptfoo](https://www.promptfoo.dev/blog/rlvr-explained/)

> "DeepSeek-R1-Zero, a model trained via large-scale reinforcement learning (RL) without supervised fine-tuning (SFT) as a preliminary step, demonstrates remarkable reasoning capabilities."

— [DeepSeek-R1 (arXiv:2501.12948)](https://arxiv.org/abs/2501.12948)

### Core Mechanism

RLVR uses **deterministic, programmatic reward functions** instead of:
- Learned reward models (RLHF)
- Human preference data (DPO, KTO)
- Value/critic networks (standard PPO)

The reward is typically:
- `1` — final answer matches ground truth
- `0` — final answer does not match

---

## Key Sources

### Primary Reference: DeepSeek-R1

| Field | Value |
|-------|-------|
| Paper | [DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning](https://arxiv.org/abs/2501.12948) |
| Authors | DeepSeek-AI, January 2025 |
| Published | Nature (2025) |
| Benchmark | AIME 2024: **79.8%** (o1-1217: 79.2%) |

> "R1-Zero autonomously develops extended test-time computation, reflection, and alternative problem-solving. Researchers observed an aha moment where the model learned to reevaluate initial approaches."

— [DeepSeek-R1](https://arxiv.org/abs/2501.12948)

> "On AIME 2024, DeepSeek-R1 achieved 79.8 percent pass at 1, slightly edging out OpenAI-o1-1217 (79.2 percent). On MATH-500, it hit 97.3 percent."

— [DeepSeek-R1](https://arxiv.org/abs/2501.12948)

### Critical Studies

#### Tsinghua/SJTU (Skeptical Position) — NeurIPS 2025 Best Paper

| Field | Value |
|-------|-------|
| Paper | [Does Reinforcement Learning Really Incentivize Reasoning Capacity in LLMs Beyond the Base Model?](https://arxiv.org/abs/2504.13837) |
| Authors | Yang Yue, Zhiqi Chen, Rui Lu, et al. (Tsinghua, SJTU) |
| URL | [limit-of-rlvr.github.io](https://limit-of-rlvr.github.io/) |
| Finding | RLVR does NOT expand reasoning — only improves sampling efficiency |

> "Our key finding is that all reasoning paths in the RLVR model are already present in the base model."

— [Tsinghua/SJTU (arXiv:2504.13837)](https://arxiv.org/abs/2504.13837)

> "As RLVR training progresses, the average performance (pass at 1) improves, but the coverage of solvable problems (pass at 256) decreases, indicating a reduction in LLM reasoning boundary."

— [Tsinghua/SJTU (arXiv:2504.13837)](https://arxiv.org/abs/2504.13837)

> "Crucially, all correct solutions from RL-trained models already exist in the base model distribution, proving RLVR enhances sampling efficiency, not reasoning capacity, while inadvertently shrinking the solution space."

— [Tsinghua/SJTU (arXiv:2504.13837)](https://arxiv.org/abs/2504.13837)

#### Microsoft Research (Supportive Position)

| Field | Value |
|-------|-------|
| Paper | [Reinforcement Learning with Verifiable Rewards Implicitly Incentivizes Correct Reasoning in Base LLMs](https://arxiv.org/html/2506.14245v2) |
| Authors | Xumeng Wen, Zihan Liu, et al. (Microsoft Research Asia, Peking, CUHK, UCLA) |
| Finding | RLVR DOES extend reasoning — CoT-Pass at K reveals true boundary |

> "RLVR can fundamentally extend the reasoning capability boundary for both mathematical and coding tasks — not merely improve sampling efficiency."

— [Microsoft Research (arXiv:2506.14245v2)](https://arxiv.org/html/2506.14245v2)

> "We show theoretically and empirically that reinforcement learning with verifiable rewards implicitly incentivizes correct reasoning in base LLMs."

— [Microsoft Research (arXiv:2506.14245v2)](https://arxiv.org/html/2506.14245v2)

### Additional Sources

| Paper | Authors | URL | Notes |
|-------|---------|-----|-------|
| Expanding RL with Verifiable Rewards Across Diverse Domains | Tencent AI Lab | [arXiv:2503.23829](https://arxiv.org/pdf/2503.23829) | Only 60.3% of real-world math problems have single-term numerical answers |
| GRPO with Verifiable Rewards: Dynamics, Loss Equivalence, and Success Amplification | Youssef Mroueh, IBM Research | [arXiv:2503.06639](https://arxiv.org/pdf/2503.06639) | Theorem: GRPO always amplifies reference policy success probability |

---

## Repositories & Tools

| Repository | URL | Description |
|------------|-----|-------------|
| opendilab/awesome-RLVR | [GitHub](https://github.com/opendilab/awesome-RLVR) | Curated list (108 stars) — papers, codebases, resources |
| Intelligent-Internet/ii_verl | [GitHub](https://github.com/Intelligent-Internet/ii_verl) | veRL — supports PPO, GRPO, DAPO, REINFORCE++, RLOO |
| openrlhf/openrlhf | [GitHub](https://github.com/openrlhf/openrlhf) | Scalable RL framework |
| RAGEN-AI/RAGEN | [GitHub](https://github.com/RAGEN-AI/RAGEN) | Reasoning agents in stochastic environments |
| deepseek-ai/DeepSeek-R1 | [HuggingFace](https://huggingface.co/deepseek-ai/DeepSeek-R1) | Model weights — R1, R1-Zero, distilled variants |

---

## The Core Debate: Sampling Efficiency vs. Reasoning Capacity

### Tsinghua/SJTU Position (Skeptical)

| Evidence | Value |
|----------|-------|
| Both base and RL solve | 63.3% |
| Only base solves (pass@256) | 13.3% |
| Only RL solves | **0.0%** |
| Neither solves | 23.3% |

**Conclusion**: RLVR never solves any problem the base model cannot already solve with enough samples. RLVR narrows the solution space.

### Microsoft Research Counter

- Standard **Pass@K** metric is flawed for math — base LLMs can "guess" correct answers via incorrect CoTs
- Proposes **CoT-Pass@K** metric to reveal true extended reasoning boundary
- Shows consistent gap between base and RLVR models under this metric

### Random Rewards Controversy

> "Qwen2.5-Math-7B improved 21.4 percent on MATH-500 with random rewards, nearly matching the 29.1 percent gain from ground truth rewards."

This suggests RL updates can guide internal pathway refinement even without correct verifier signals.

---

## Benchmark Results: DeepSeek-R1

| Benchmark | DeepSeek-R1 | OpenAI o1-1217 |
|-----------|-------------|----------------|
| AIME 2024 | **79.8%** | 79.2% |
| MATH-500 | **97.3%** | 96.4% |
| MMLU | 90.8% | **91.8%** |
| GPQA Diamond | 71.5% | **75.7%** |
| Codeforces | 2,029 Elo | **2,061** |
| LiveCodeBench | **65.9%** | 63.4% |
| SWE-Bench Verified | **49.2%** | 48.9% |

### Distilled Model Performance

| Model | AIME 2024 | MATH-500 |
|-------|-----------|----------|
| R1-Distill-Qwen-1.5B | 28.9% | 83.9% |
| R1-Distill-Qwen-7B | 55.5% | 92.8% |
| R1-Distill-Qwen-14B | 69.7% | 93.9% |
| R1-Distill-Qwen-32B | 72.6% | 94.3% |
| R1-Distill-Llama-70B | 70.0% | 94.5% |

---

## Related Topics

- [GRPO](./GRPO.md) — The dominant RL algorithm used with RLVR
- [GIFT](./GIFT.md) — Group-Relative Implicit Fine-Tuning; extends GRPO with implicit reward matching
- [Thinking-COT](./Thinking-COT.md) — Chain-of-Thought prompting; foundational reasoning technique
- [Internal-Chain-of-Thought](../../model-cards/Reasoning/Internal-Chain-of-Thought.md) — DeepSeek-R1's explicit CoT implementation

---

## References

- [DeepSeek-R1 (arXiv:2501.12948)](https://arxiv.org/abs/2501.12948)
- [Tsinghua/SJTU — Does RLVR Really Incentivize Reasoning? (arXiv:2504.13837)](https://arxiv.org/abs/2504.13837)
- [Microsoft Research — RLVR Implicitly Incentivizes Correct Reasoning (arXiv:2506.14245v2)](https://arxiv.org/html/2506.14245v2)
- [Tencent AI Lab — Expanding RLVR Across Domains (arXiv:2503.23829)](https://arxiv.org/pdf/2503.23829)
- [IBM Research — GRPO Dynamics (arXiv:2503.06639)](https://arxiv.org/pdf/2503.06639)
