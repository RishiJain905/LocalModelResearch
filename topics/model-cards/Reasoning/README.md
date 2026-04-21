# Reasoning

> *"Chain-of-thought prompting is an emergent property of model scale."*> — Wei et al., NeurIPS 2022

This section documents the **reasoning taxonomy** for AI models — covering how models allocate compute to reasoning, what happens inside hidden reasoning traces, and how we benchmark reasoning capability at the frontier.

## 📚 Contents

| Subtopic | Description |
|----------|-------------|
| [Compute-to-Reasoning Ratio](./Compute-to-Reasoning-Ratio.md) | Token overhead, efficiency trade-offs, and compression methods for Chain-of-Thought reasoning |
| [Internal Chain-of-Thought (iCoT)](./Internal-Chain-of-Thought.md) | Hidden reasoning processes, latent continuous thought, and model transparency (OpenAI o1, DeepSeek-R1, Claude 3.7+) |
| [Benchmarks: HLE & MathArena](./Benchmarks-HLE-MathArena.md) | Frontier reasoning benchmarks: Humanity's Last Exam and MathArena scores, leaderboards, and saturation analysis |

## Overview

Reasoning in LLMs has evolved from explicit "let's think step by step" prompting to complex internal scratchpads, latent hidden states, and dedicated reasoning tokens. This creates a tension between:

- **Capability** — longer reasoning traces improve accuracy on complex tasks
- **Efficiency** — CoT can consume 10–50× more tokens than direct answering
- **Transparency** — some models hide reasoning (o1), some reveal it (DeepSeek-R1), some reveal potentially unfaithful traces (Claude 3.7+)
- **Evaluation** — benchmarks like HLE and MathArena push the frontier, with proof-writing remaining unsolved

### Key Findings

- **CoT overhead is 10–50×** over direct answering; compression methods (TALE, ES-CoT, Extra-CoT) can reduce this by 41–76%
- **Language is not the optimal reasoning medium** — latent continuous thought (Coconut) moves reasoning into hidden states
- **HLE remains unsaturated** at ~65% top score vs. ~90% human expert; proof-based math (USAMO/IMO) is the new frontier with models scoring <25%
- **Calibration is terrible** — frontier models show >70–80% calibration error while scoring <10% on HLE, indicating severe overconfidence

## Related Topics

- [Quantization](../../quantization/) — Model efficiency and size trade-offs
- [Benchmarks](../../benchmarks/) — General evaluation frameworks
- [Model Cards](../) — Parent directory for model documentation standards
