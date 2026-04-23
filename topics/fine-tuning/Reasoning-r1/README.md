# Reasoning-r1

## 📌 Overview

This section covers the key post-training techniques and algorithms that enabled the reasoning model breakthrough starting with DeepSeek-R1 (January 2025). The core insight: applying reinforcement learning with verifiable rewards (binary correct/incorrect signals) to base language models produces emergent reasoning capabilities — chain-of-thought behavior, self-verification, and extended test-time computation — without human annotation.

## Subtopics

| File | Topic | Type |
|------|-------|------|
| [GRPO.md](./GRPO.md) | Group Relative Policy Optimization | RL Algorithm |
| [RLVR.md](./RLVR.md) | Reinforcement Learning with Verifiable Rewards | Post-training Paradigm |
| [GIFT.md](./GIFT.md) | Group-Relative Implicit Fine-Tuning | RL Framework |
| [Thinking-COT.md](./Thinking-COT.md) | Chain-of-Thought Prompting | Prompt Engineering |

---

## Summary Matrix

| Technique | Full Name | Key Innovation | Primary Paper |
|-----------|-----------|---------------|---------------|
| **GRPO** | Group Relative Policy Optimization | Eliminates critic network; group-relative advantage estimation | DeepSeekMath (arXiv:2402.03300) |
| **RLVR** | Reinforcement Learning with Verifiable Rewards | Binary reward signals; no learned reward model needed | DeepSeek-R1 (arXiv:2501.12948) |
| **GIFT** | Group-Relative Implicit Fine-Tuning | Reward matching instead of reward maximization | arXiv:2510.23868 |
| **Thinking-COT** | Chain-of-Thought Prompting | Intermediate reasoning steps before final answer | Wei et al. (arXiv:2201.11903) |

---

## The Core Debate: Does RLVR Expand Reasoning?

This is the central controversy in the Reasoning-r1 lineage:

| Position | Claim | Key Evidence |
|----------|-------|--------------|
| **Skeptical** (Tsinghua/SJTU) | RLVR only improves sampling efficiency, does NOT expand reasoning boundary | 0% of problems solved exclusively by RL; all RL solutions exist in base model |
| **Supportive** (Microsoft Research) | RLVR DOES extend reasoning; Pass@K metric is flawed for math | CoT-Pass@K shows consistent gap between base and RL models |

---

## Benchmark Results

| Model | AIME 2024 | MATH-500 | Notes |
|-------|-----------|----------|-------|
| DeepSeek-R1-Zero | 47% | — | Pure GRPO RL, no SFT |
| DeepSeek-R1 | 79.8% | 97.3% | GRPO + SFT + cold-start |
| R1-Distill-Qwen-7B | 55.5% | 92.8% | Distilled reasoning |
| R1-Distill-Qwen-32B | 72.6% | 94.3% | Best distilled performance |

---

## Related Topics

- [Internal-Chain-of-Thought](../model-cards/Reasoning/Internal-Chain-of-Thought.md) — DeepSeek-R1's explicit CoT implementation; transparency debate
- [Reasoning-Effort-Params](../API-Servers/Reasoning-Metadata/) — Reasoning effort parameters for API serving
- [Reasoning-Gap-PPL-vs-Reasoning](../quantization/Eval-Metrics/Reasoning-Gap-PPL-vs-Reasoning.md) — Quantization effects on reasoning
- [Fine-Tuning README](../README.md) — Parent fine-tuning section
