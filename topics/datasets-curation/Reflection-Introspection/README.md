# Reflection-Introspection

> Techniques that train models to **reflect on their own outputs** and **correct their own mistakes** — data curation approaches that create self-correction training signals.

## 📂 Contents

| Document | Description |
|----------|-------------|
| [Reflection-Tuning.md](./Reflection-Tuning.md) | Data recycling via oracle LLM introspection — V1 (NeurIPS 2023) and V2/Selective (ACL 2024) |
| [Self-Correction-Datasets.md](./Self-Correction-Datasets.md) | Key datasets: ReTrace (200k), SCoRe, Self-Refine, STaR, STaSC + benchmarks |

## 🔑 Key Themes

### Reflection-Tuning
- **V1 (NeurIPS 2023):** Oracle LLM reflects on instruction-response pairs → enhanced training data
- **V2 (ACL 2024 Findings):** Student model selects which recycled samples to use (IFD/r-IFD metrics)
- **83.48%** AlpacaEval win rate with selective recycling (only 926 samples)

### Self-Correction Datasets
- **ReTrace:** 200k Q→I→C→R structured traces (GPT-4o teacher)
- **SCoRe:** Multi-turn RL, reduces correct-to-incorrect transitions from 15.8% → 1.4%
- **Self-Refine:** ~20% improvement via iterative self-feedback
- **STaSC:** Self-generated data for small language models

## 📊 Taxonomy Matrix

| Technique | Venue | Method | Key Result |
|-----------|-------|--------|------------|
| Reflection-Tuning V1 | NeurIPS 2023 WS | Oracle LLM reflection | +50.5% win rate vs Alpaca |
| Selective Reflection-Tuning | ACL 2024 Findings | Student-selected recycling | 83.48% AlpacaEval (926 samples) |
| ReTrace | ICLR 2026 | Structured Q→I→C→R | 200k examples |
| SCoRe | ICLR 2025 | Multi-turn RL | 15.6% Δ on MATH |
| Self-Refine | 2023 | Iterative self-feedback | ~20% improvement |
| STaSC | ICLR 2025 | Self-generated data (SLMs) | Iterative fine-tuning |

## 🔗 Related Folder

← Back to [datasets-curation](../README.md)
