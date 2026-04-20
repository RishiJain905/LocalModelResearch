# Human Preference Correlation: MixEval-Hard

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026).
>
> **GitHub:** [tatsu-lab/alpaca_eval](https://github.com/tatsu-lab/alpaca_eval) ⭐2.0K · [lm-sys/FastChat](https://github.com/lm-sys/FastChat) ⭐39.5K · [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) ⭐12.2K

---

## Overview

> *"MixEval-Hard uses difficulty-calibrated problems to better distinguish model capabilities and correlate with human preferences."* — [MixEval Team, arXiv:2410.02537](https://arxiv.org/abs/2410.02537)

Human preference correlation measures how well LLM benchmarks predict actual human satisfaction with model outputs.

---

## Key Paper

### "MixEval-Hard: Benchmarking Reasoning Models on Hard Problems"

> **arXiv:** [2410.02537](https://arxiv.org/abs/2410.02537) (MixEval paper — specific Hard variant repo not found)

---

## Benchmark Comparison

| Benchmark | Human Preference Correlation | Notes |
|-----------|-------------------------------|-------|
| **LMSYS ELO** | Very High | Direct human comparisons |
| **Arena-Hard** | High | Live data, competitive setting |
| **MT-Bench** | High | Multi-turn dialogue, better than single-turn |
| **MixEval-Hard** | Designed to be highest | Harder problems closer to human use cases |
| **AlpacaEval** | Medium-High | LLM-as-judge, fast but some bias |
| **ChatArena** | Medium | Multi-agent debates |

---

## Key Features of MixEval-Hard

| Feature | Description |
|---------|-------------|
| **Difficulty-calibrated** | Problems scaled by difficulty to distinguish model capabilities |
| **Multi-domain coverage** | Math, coding, reasoning across challenging problem sets |
| **Dynamic evaluation** | Evolving difficulty to prevent benchmark saturation |
| **Higher correlation** | Designed to correlate better with human preferences than standard benchmarks |

---

## Why Harder Problems = Better Correlation

Standard benchmarks (MMLU, SimpleQA) often test surface-level knowledge that doesn't reflect real-world usage. MixEval-Hard focuses on:

1. **Hard math** — Multi-step problem solving
2. **Complex coding** — Algorithm design, debugging
3. **Multi-hop reasoning** — Connecting disparate facts
4. **Ambiguous queries** — Realistic user questions

---

## Related Repos

| Resource | Link | Description |
|----------|------|-------------|
| AlpacaEval | [tatsu-lab/alpaca_eval](https://github.com/tatsu-lab/alpaca_eval) ⭐2.0K | LLM-as-judge evaluation |
| MT-Bench | [lm-sys/FastChat](https://github.com/lm-sys/FastChat) ⭐39.5K | Multi-turn benchmark |
| Arena-Hard-Auto | [lmarena/arena-hard-auto](https://github.com/lmarena/arena-hard-auto) ⭐1.0K | Live competition benchmark |
| LM-Eval Harness | [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) ⭐12.2K | Benchmarking framework |

---

## Related Research

### "How Well Do LLMs Judge?"

Examines LLM-as-judge reliability — whether one LLM can reliably judge another.

### "AlpacaFarm: A Simulation Framework for Learning from Human Feedback"

> **arXiv:** [2304.03354](https://arxiv.org/abs/2304.03354)

Dartmouth paper on human preference alignment — shows how simulated feedback compares to real human preferences.

### "MT-Bench: Multi-turn Benchmark"

UC Berkeley evaluation framework for multi-turn dialogue.

---

## LLM-as-Judge Limitations

| Issue | Impact |
|-------|--------|
| **Position bias** | LLMs prefer responses in certain positions |
| **Length bias** | Longer responses scored higher |
| **Self-preference** | Models may favor their own outputs |
| **Stylistic bias** | Format over substance |

---

## See Also

- [Eval Metrics README](./README.md)
- [LMSYS Arena](https://chat.lmsys.org/) — Live leaderboard
- [Reasoning Gap](./Reasoning-Gap-PPL-vs-Reasoning.md)
