# GRPO (Group Relative Policy Optimization)

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Group Relative Policy Optimization |
| **Type** | Reinforcement Learning Algorithm |
| **Introduced** | February 2024 (DeepSeekMath paper) |
| **Paper** | [arXiv:2402.03300](https://arxiv.org/abs/2402.03300) |
| **Authors** | Zhihong Shao et al., DeepSeek-AI |
| **Key Advantage** | Eliminates critic/value network — ~50% memory reduction vs PPO |
| **Used In** | DeepSeek-R1, DeepSeek-R1-Zero, DAPO |

---

## Definitions

> "We introduce Group Relative Policy Optimization (GRPO), a variant of Proximal Policy Optimization (PPO), that enhances mathematical reasoning abilities while concurrently optimizing the memory usage of PPO."

— [DeepSeekMath (arXiv:2402.03300)](https://arxiv.org/abs/2402.03300)

> "Group Relative Policy Optimization (GRPO) foregoes the critic model and estimates the baseline from group scores."

— [DeepSeek-R1 (arXiv:2501.12948)](https://arxiv.org/abs/2501.12948)

### How GRPO Works

GRPO generates multiple outputs (a "group") for each prompt, then uses the relative ranking within that group to estimate advantage — eliminating the need for a separate critic/value network:

- **Group size**: Typically 8 samples per prompt
- **Advantage estimation**: Relative ranking within group vs. absolute value estimation
- **Memory savings**: No critic network required (~50% less memory than PPO)
- **Reward normalization**: Group-relative reward normalization stabilizes training

---

## Key Sources

### Primary Papers

| Paper | Year | URL | Key Quote |
|-------|------|-----|-----------|
| DeepSeekMath: Pushing the Limits of Mathematical Reasoning in Open Language Models | 2024 | [arXiv:2402.03300](https://arxiv.org/abs/2402.03300) | "GRPO... enhances mathematical reasoning while optimizing memory usage of PPO" |
| DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning | 2025 | [arXiv:2501.12948](https://arxiv.org/abs/2501.12948) | "GRPO foregoes the critic model and estimates the baseline from group scores" |
| DAPO: An Open-Source LLM Reinforcement Learning System at Scale | 2025 | [arXiv:2503.14476](https://arxiv.org/abs/2503.14476) | "Initial experiments with naive GRPO on Qwen2.5-32B achieved only 30 points on AIME 2024 — far below DeepSeek's 47 points" |

### GRPO Improvements & Variants

| Paper | Venue | URL | Contribution |
|-------|-------|-----|--------------|
| DAPO | arXiv 2025 | [arXiv:2503.14476](https://arxiv.org/abs/2503.14476) | Addressed entropy collapse, reward noise, and training instability in vanilla GRPO |
| CPPO | NeurIPS 2025 | — | 7.98x speedup on GSM8K |
| Hybrid GRPO | arXiv:2502.01652 | [arXiv:2502.01652](https://arxiv.org/abs/2502.01652) | Combined GRPO with other techniques |
| GRPO with Verifiable Rewards: Dynamics, Loss Equivalence, and Success Amplification | 2025 | [arXiv:2503.06639](https://arxiv.org/pdf/2503.06639) | IBM Research — theoretical analysis of GRPO convergence |

> "Theorem 3 (Amplification): For any prompt where 0 < p_ref < 1, any fixed point satisfies p* > p_ref. Thus GRPO reliably amplifies the reference policy success probability."

— [Youssef Mroueh, IBM Research (arXiv:2503.06639)](https://arxiv.org/pdf/2503.06639)

---

## Repositories & Tools

| Repository | URL | Description |
|------------|-----|-------------|
| KShivendu/grpo | [GitHub](https://github.com/KShivendu/grpo) | PyTorch implementation from scratch |
| policy-gradient/GRPO-Zero | [GitHub](https://github.com/policy-gradient/GRPO-Zero) | Minimal dependencies implementation |
| ALucek/GRPO-Training | [GitHub](https://github.com/ALucek/GRPO-Training) | Training overview and examples |
| verl-project/verl | [GitHub](https://github.com/verl-project/verl) | Production RL framework — supports GRPO |
| openrlhf/openrlhf | [GitHub](https://github.com/openrlhf/openrlhf) | Scalable RLHF/GRPO framework |

---

## Benchmark Results

| Model | AIME 2024 | Notes |
|-------|-----------|-------|
| DeepSeek-R1-Zero | 47% | Pure GRPO RL, no SFT |
| DeepSeek-R1 | 79.8% | GRPO + SFT + cold-start |
| Qwen2.5-32B + DAPO | 50% | 50% fewer training steps vs vanilla GRPO |
| Qwen2.5-32B (vanilla GRPO) | 30% | Baseline — far below DeepSeek |

### Why DAPO Outperforms Vanilla GRPO

The DAPO paper identified three critical failure modes in vanilla GRPO:

1. **Entropy collapse** — policy becomes deterministic too early, reducing exploration
2. **Reward noise** — binary rewards are noisy signals for advantage estimation
3. **Training instability** — group-relative normalization alone is insufficient

DAPO introduces clipping, dynamic sampling, and token-level policy gradients to address these.

---

## Debates & Controversies

### GRPO vs. PPO

> "GRPO solves key adoption barriers in OpenAI PPO: it eliminates the Value Model entirely, uses sequence-level advantage estimation only, has fewer tunable parameters, and simpler implementation."

— [Fireworks AI](https://fireworks.ai/blog/reinforcement-learning-with-verifiable-reward)

### GRPO Limitations

| Issue | Description |
|-------|-------------|
| Group-size dependency | Performance scales with group size; smaller groups = noisier estimates |
| Entropy collapse | Over-training leads to deterministic outputs |
| Reward hacking | Model finds shortcut patterns that fool the verifier |

---

## Related Topics

- [RLVR](./RLVR.md) — The broader post-training paradigm GRPO belongs to
- [GIFT](./GIFT.md) — Group-Relative Implicit Fine-Tuning; extends GRPO with implicit reward matching
- [Thinking-COT](./Thinking-COT.md) — Chain-of-Thought prompting; foundational reasoning technique
- [DeepSeek-R1-Zero](../model-cards/Reasoning/Internal-Chain-of-Thought.md) — First pure RL reasoning model using GRPO

---

## References

- [DeepSeekMath (arXiv:2402.03300)](https://arxiv.org/abs/2402.03300)
- [DeepSeek-R1 (arXiv:2501.12948)](https://arxiv.org/abs/2501.12948)
- [DAPO (arXiv:2503.14476)](https://arxiv.org/abs/2503.14476)
- [GRPO with Verifiable Rewards (arXiv:2503.06639)](https://arxiv.org/pdf/2503.06639)
