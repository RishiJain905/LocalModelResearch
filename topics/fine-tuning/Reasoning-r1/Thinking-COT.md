# Thinking-COT (Chain-of-Thought Prompting)

## 📌 Overview

| Field | Value |
|-------|-------|
| **Full Name** | Chain-of-Thought Prompting |
| **Type** | Prompt engineering technique for reasoning |
| **Foundational Paper** | [arXiv:2201.11903](https://arxiv.org/abs/2201.11903) (Wei et al., Google Brain) |
| **Year Introduced** | January 2022 |
| **Key Finding** | Intermediate reasoning steps improve complex task performance |
| **Emergence Threshold** | ~100B parameters |

> **Note**: "Thinking-COT" is a colloquial/shorthand reference to Chain-of-Thought prompting. No distinct technique by exactly that name was found in the academic literature.

---

## Definitions

> "We explore how generating a chain of thought — a series of intermediate reasoning steps — significantly improves the ability of large language models to perform complex reasoning."

— [Wei et al., arXiv:2201.11903](https://arxiv.org/abs/2201.11903)

> "Chain of thought (CoT) is a prompt engineering technique that enhances the output of large language models (LLMs), particularly for complex tasks involving multistep reasoning."

— [IBM Think Glossary](https://www.ibm.com/think/topics/chain-of-thoughts)

> "LLMs are decent zero-shot reasoners by simply adding 'Let's think step by step' before each answer."

— [Kojima et al., arXiv:2205.11916](https://arxiv.org/abs/2205.11916)

### Key Characteristics

| Property | Description |
|----------|-------------|
| **Emergent ability** | CoT reasoning emerges at ~100B+ parameters |
| **Few-shot or zero-shot** | Can use exemplars or simple trigger phrases |
| **Applicable domains** | Arithmetic, commonsense, symbolic reasoning |
| **Mechanism** | Decomposes multi-step problems into intermediate steps |

---

## Key Sources

### Foundational Papers

| Paper | Authors | Year | URL | Key Quote |
|-------|---------|------|-----|-----------|
| Chain-of-Thought Prompting Elicits Reasoning in Large Language Models | Wei, Wang, Schuurmans, et al. (Google Brain) | 2022 | [arXiv:2201.11903](https://arxiv.org/abs/2201.11903) | "Generating a chain of thought — a series of intermediate reasoning steps — significantly improves the ability of LLMs to perform complex reasoning" |
| Large Language Models are Zero-Shot Reasoners | Kojima, Gu, Reid, Matsuo, Iwasawa | 2022 | [arXiv:2205.11916](https://arxiv.org/abs/2205.11916) | "Simply adding 'Let's think step by step' before each answer" |
| Self-Consistency Improves Chain of Thought Reasoning | Wang, Wei, Schuurmans, Le, et al. (Google Brain) | 2022 | [arXiv:2203.11171](https://arxiv.org/abs/2203.11171) | "A new decoding strategy to replace naive greedy decoding in CoT prompting" |
| Automatic Chain of Thought Prompting (Auto-CoT) | Zhang, Gao, Liu, et al. | 2022 | [arXiv:2210.03493](https://arxiv.org/abs/2210.03493) | Automated demonstration selection for CoT |
| ReAct: Synergizing Reasoning and Acting | Yao, Zhao, Yu, et al. (Princeton/Google) | 2022 | [arXiv:2210.03629](https://arxiv.org/abs/2210.03629) | "LLMs' abilities for reasoning and acting have primarily been studied as separate topics" |
| Tree of Thoughts: Deliberate Problem Solving with LLMs | Yao, Li, Zhao, et al. | 2023 | [arXiv:2305.10601](https://arxiv.org/abs/2305.10601) | Extends CoT into tree search framework |
| Least-to-Most Prompting Enables Complex Reasoning | Zhou, Zaheer, et al. | 2023 | [arXiv:2205.10625](https://arxiv.org/abs/2205.10625) | Decomposes complex problems into simpler subproblems |

### Chain-of-Thought Reasoning Without Prompting

> "Chain-of-thought reasoning can be elicited via decoding changes, not just prompting."

— [NeurIPS 2024](https://proceedings.neurips.cc/paper/2024/hash/7a8e7fd295aa04eac4b470ae27f8785c-Abstract-Conference.html)

---

## Blog Posts & Technical Explanations

| Source | Title | URL | Key Takeaway |
|--------|-------|-----|--------------|
| Google Research Blog | "Language Models Perform Reasoning via Chain of Thought" | [Link](https://research.google/blog/language-models-perform-reasoning-via-chain-of-thought/) | CoT is emergent at ~100B params; enables solving problems not solvable with standard prompting |
| Prompt Engineering Guide | "Chain-of-Thought (CoT) Prompting" | [Link](https://www.promptingguide.ai/techniques/cot) | Comprehensive guide — Few-shot CoT, Zero-shot CoT, Auto-CoT |
| IBM Think | "What is chain of thought (CoT) prompting?" | [Link](https://www.ibm.com/think/topics/chain-of-thoughts) | CoT simulates human-like reasoning |
| Han Lee | "Reasoning Series, Part 1: Understanding OpenAI o1" | [Link](https://leehanchung.github.io/blogs/2024/10/08/reasoning-understanding-o1/) | OpenAI o1 uses hidden reasoning tokens similar to Zero-Shot CoT |
| LessWrong | "Hidden Reasoning in LLMs: A Taxonomy" | [Link](https://www.lesswrong.com/posts/ZrgFfeWuckpwK5Lyi/hidden-reasoning-in-llms-a-taxonomy) | Taxonomy for non-verbal reasoning pathways |

---

## Repositories & Tools

| Repository | URL | Description |
|------------|-----|-------------|
| amazon-science/auto-cot | [GitHub](https://github.com/amazon-science/auto-cot) | Official Auto-CoT implementation |
| facebookresearch/coconut | [GitHub](https://github.com/facebookresearch/coconut) | Latent continuous thought training (Meta FAIR) — 2.5k+ stars |
| casper-hansen/OpenCoconut | [GitHub](https://github.com/casper-hansen/OpenCoconut) | Community latent reasoning implementation |
| GoogleCloudPlatform/specialized-training-content | [GitHub](https://github.com/GoogleCloudPlatform/specialized-training-content/blob/main/courses/generative_ai/app_dev_llm/cot_react.ipynb) | CoT + ReAct notebook |
| zchuz/cot-reasoning-survey | [GitHub](https://github.com/zchuz/cot-reasoning-survey) | ACL 2024 Survey of CoT Reasoning |

---

## Benchmark Results (CoT Impact)

| Benchmark | Baseline (No CoT) | With CoT | Improvement |
|-----------|-------------------|----------|-------------|
| GSM8K (GPT-3 175B) | ~55% | **74%** (PaLM-540B + self-consistency) | +17.9% |
| MultiArith | ~35% | **93%** (PaLM-540B) | +58% |
| SVAMP | ~60% | **70%** (PaLM-540B) | +11.0% |
| AQuA | ~30% | **42%** (PaLM-540B) | +12.2% |
| StrategyQA | ~70% | **76.4%** | +6.4% |

---

## Key Milestones

| Date | Event | Significance |
|------|-------|--------------|
| Jan 2022 | Wei et al. CoT paper (arXiv:2201.11903) | Founded the field of CoT prompting |
| May 2022 | Google Research Blog on CoT | Popularized the technique |
| 2022 | Zero-Shot CoT (Kojima et al.) | "Let's think step by step" trigger |
| 2023 | Self-Consistency (Wang et al., ICLR) | Major CoT improvement via majority voting |
| 2023 | Tree of Thoughts (Yao et al., NeurIPS) | Extended CoT to search |
| Nov 2024 | OpenAI o1 release | Hidden reasoning tokens; System 2 thinking |
| Jan 2025 | DeepSeek-R1 release | Open-source explicit CoT reasoning |

---

## Transparency Debate: Hidden vs Explicit CoT

| Model | CoT Transparency | Issue |
|-------|-----------------|-------|
| OpenAI o1/o3 | **Hidden** | Internal scratchpad not visible; explicit CoT prompts *harm* performance |
| DeepSeek-R1 | **Explicit** | Full `<reasoning_content>` tags visible; enables distillation concerns |
| Claude 3.7+ | **Visible but unfaithful** | Thinking blocks published but internal computation often diverges |

> "Reasoning models struggle to control their Chains of Thought."

— [OpenAI Research, 2025](https://cdn.openai.com/pdf/a21c39c1-fa07-41db-9078-973a12620117/cot_controllability.pdf)

### OpenAI vs DeepSeek — Distillation Controversy (2025)

> "We have observed accounts associated with DeepSeek employees developing methods to circumvent OpenAI's access restrictions and access models through obfuscated third-party routers."

— [OpenAI memo, via Rest of World](https://restofworld.org/2026/openai-deepseek-distillation-dispute-us-china/)

> "One possible reason for the accusation is to prevent DeepSeek and China companies from acquiring more chips to distill the U.S. model, so that the U.S. models can keep their leading position."

— Researcher comment in same article

---

## Debates & Limitations

### Scale Dependency

> "Chain-of-Thought prompting does better than standard prompting only at the scale of ~100B parameters; models of smaller scale produced fluent but illogical reasoning."

— [Reddit discussion on arXiv:2201.11903](https://www.reddit.com/r/mlscaling/comments/sh5tam/chain_of_thought_prompting_elicits_reasoning_in/)

### CoT Degradation Cases

> "Chain-of-thought prompting unexpectedly degrades LLM performances in certain..."

— ["On the Limitations of Chain-of-Thought in In-Context Learning," arXiv:2504.05081](https://arxiv.org/abs/2504.05081)

---

## Related Topics

- [GRPO](./GRPO.md) — RL algorithm that elicits CoT-like behavior through reward signals
- [RLVR](./RLVR.md) — The post-training paradigm that uses CoT as a training signal
- [GIFT](./GIFT.md) — Reward matching framework extending GRPO
- [Internal-Chain-of-Thought](../../model-cards/Reasoning/Internal-Chain-of-Thought.md) — Internal vs Explicit CoT taxonomy

---

## References

- [Wei et al. — Chain-of-Thought (arXiv:2201.11903)](https://arxiv.org/abs/2201.11903)
- [Kojima et al. — Zero-Shot CoT (arXiv:2205.11916)](https://arxiv.org/abs/2205.11916)
- [Wang et al. — Self-Consistency (arXiv:2203.11171)](https://arxiv.org/abs/2203.11171)
- [Zhang et al. — Auto-CoT (arXiv:2210.03493)](https://arxiv.org/abs/2210.03493)
- [Yao et al. — ReAct (arXiv:2210.03629)](https://arxiv.org/abs/2210.03629)
- [Yao et al. — Tree of Thoughts (arXiv:2305.10601)](https://arxiv.org/abs/2305.10601)
