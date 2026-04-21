# Benchmarks: HLE (Humanity's Last Exam) and MathArena Scores

> *"Benchmarks are important tools for tracking the rapid advancements in large language model (LLM) capabilities. However, benchmarks are not keeping pace in difficulty: LLMs now achieve over 90% accuracy on popular benchmarks like MMLU, limiting informed measurement of state-of-the-art LLM capabilities."*
> — Phan et al., *Humanity's Last Exam*

This page covers two frontier reasoning benchmarks: **HLE**, the most difficult closed-ended academic benchmark, and **MathArena**, the first dynamic benchmark evaluating on uncontaminated, newly released math competitions.

## Contents

1. [Humanity's Last Exam (HLE)](#humanitys-last-exam-hle)
2. [MathArena](#matharena)
3. [Key Papers](#key-papers)
4. [Leaderboards & Empirical Results](#leaderboards--empirical-results)
5. [Open-Source Repositories & Tools](#open-source-repositories--tools)
6. [Blog Posts & Articles](#blog-posts--articles)
7. [Key Findings](#key-findings)

---

## Humanity's Last Exam (HLE)

> *"High accuracy on HLE would demonstrate AI has achieved expert-level performance on closed-ended cutting-edge scientific knowledge, but it would not alone suggest autonomous research capabilities or 'artificial general intelligence.'"*
> — Phan et al.

| Feature | Detail |
|:--------|:-------|
| **Total Questions** | 2,500 (public) + held-out private set |
| **Subjects** | 100+ across math, humanities, natural sciences, engineering, law, philosophy, art history |
| **Format** | ~76% exact-match; ~24% multiple-choice (5+ options) |
| **Modality** | ~14% multi-modal (text + image/diagram) |
| **Contributors** | ~1,000 experts from 500+ institutions across 50 countries |
| **Prize Pool** | $500,000 USD for question submissions |
| **Evaluation** | Automated judging via `o3-mini-2025-01-31` with structured JSON decoding |
| **Dataset** | [lastexam.ai](https://lastexam.ai) |

### HLE Initial Scores (January 2025)

> *"Current frontier models perform poorly on HLE. State-of-the-art LLMs demonstrate low accuracy and calibration on HLE, highlighting a significant gap between current LLM capabilities and the expert human frontier on closed-ended academic questions."*
> — Phan et al.

| Model | Accuracy | RMS Calibration Error |
|:---|:---:|:---:|
| GPT-4o | 2.7% | 89% |
| Grok 2 | 3.0% | 87% |
| Claude 3.5 Sonnet | 4.1% | 84% |
| Gemini 1.5 Pro | 4.6% | 88% |
| Gemini 2.0 Flash Thinking | ~6.0% | — |
| OpenAI o1 | ~8.0% | — |
| DeepSeek-R1 | ~7.1–9.4% | — |
| o3-mini (High) | **13.4%** | — |
| OpenAI Deep Research | 26.6% | — |

### HLE Leaderboard (April 2026)

Source: [llm-stats.com](https://llm-stats.com/benchmarks/humanity's-last-exam)

| Rank | Model | Score | Organization | Context | Cost |
|:---:|:---|:---:|:---:|:---:|:---:|
| 1 | **Claude Mythos Preview** ⭐ | **0.647** | Anthropic | — | $25/$125 |
| 2 | Muse Spark ⭐ | 0.584 | Meta | — | — |
| 3 | Claude Opus 4.6 | 0.531 | Anthropic | 1.0M | $5/$25 |
| 4 | GLM-5.1 ⭐ | 0.523 | Zhipu AI | 200K | $1.40/$4.40 |
| 5 | Gemini 3.1 Pro | 0.514 | Google | 1.0M | $2.50/$15.00 |
| 6 | Kimi K2-Thinking-0905 | 0.510 | Moonshot AI | — | — |
| 7 | Grok-4 Heavy | 0.507 | xAI | — | — |
| 8 | Kimi K2.5 | 0.502 | Moonshot AI | 262K | $0.60/$2.50 |
| 9 | Claude Sonnet 4.6 | 0.490 | Anthropic | 200K | $3/$15 |
| 10 | Qwen3.5-27B | 0.485 | Alibaba/Qwen | 262K | $0.30/$2.40 |
| 16 | DeepSeek-V3.2 | 0.408 | DeepSeek | 164K | $0.26/$0.38 |
| 18 | GPT-5.4 | 0.398 | OpenAI | 1.0M | $2.50/$15.00 |
| 47 | OpenAI o3 | 0.147 | OpenAI | 200K | $2/$8 |

---

## MathArena

> *"Recurring math competitions provide a stream of high-quality, challenging problems that can be used for real-time evaluation of LLMs. By evaluating models as soon as new problems are released, we effectively eliminate the risk of contamination."*
> — Balunović et al., *MathArena*

| Feature | Detail |
|:--------|:-------|
| **Competitions** | AIME, HMMT, CMIMC, BRUMO, Project Euler, USAMO, IMO, Putnam, IMC |
| **Evaluation** | 4 samples per problem; pass@1 average; cost tracking |
| **Proof Grading** | Two independent expert human graders per solution |
| **Contamination** | Eliminated by evaluating on newly released problems |
| **License** | MIT (code), CC-BY-NC-SA 4.0 (data) |
| **URLs** | [matharena.ai](https://matharena.ai) \| [GitHub](https://github.com/eth-sri/matharena) |

### MathArena Top Final-Answer Results

| Model | AIME | HMMT | BRUMO | CMIMC | **Avg** | **Cost** |
|:---|:---:|:---:|:---:|:---:|:---:|:---:|
| **GPT-5 (High)** | 95.0% | 88.3% | 91.7% | 90.0% | **91.3%** | $4.83 |
| **Grok 4 Fast (Reasoning)** | 90.8% | 91.7% | 94.2% | 85.6% | **90.6%** | $0.18 |
| **Grok 4** | 90.8% | 92.5% | 95.0% | 83.1% | **90.4%** | $7.56 |

### IMO 2025 Proof Results

| Model | Score (/42) | Percentage |
|:---|:---:|:---:|
| Gemini 2.5 Pro | 13/42 | **31%** |
| Grok-4 (re-evaluated) | ~9/42 | 21.43% |
| o3, o4-mini, DeepSeek-R1 | Significantly below Gemini | <20% |

> *"The best-performing model is Gemini 2.5 Pro, achieving a score of 31% (13 points), which is well below the 19/42 score necessary for a bronze medal."*
> — MathArena IMO Blog

### MathArena Apex (Ultra-Hard 12-Problem Benchmark)

> *"MathArena Apex is a curated set of 12 problems drawn from 2025 public competitions. By design, the best model scores only 5.2%."*
> — MathArena Apex Blog, August 2025

| Metric | Value |
|--------|-------|
| Highest accuracy | 5.2% (Qwen3-A22B-2507-Think) |
| Problems 9–12 | Completely unsolved even with heavy compute |

---

## Key Papers

### Humanity's Last Exam (arXiv 2025 / Nature 2026)

| | |
|:---|:---|
| **Title** | Humanity's Last Exam |
| **Authors** | Long Phan, Alice Gatti, Ziwen Han, Nathaniel Li, et al. (~1,000 contributors) |
| **Venue** | arXiv:2501.14249 [cs.LG]; published in *Nature* (January 2026) |
| **Institutions** | Center for AI Safety, Scale AI |

> *"We observe systematic high calibration errors (greater than 80%) paired with low accuracy (less than 10%), which indicates strong evidence for confabulation/hallucination in all measured models."*

---

### MathArena: Evaluating LLMs on Uncontaminated Math Competitions (NeurIPS 2025)

| | |
|:---|:---|
| **Title** | MathArena: Evaluating LLMs on Uncontaminated Math Competitions |
| **Authors** | Mislav Balunović, Jasper Dekoninck, Ivo Petrov, Nikola Jovanović, Martin Vechev |
| **Venue** | NeurIPS 2025 Datasets and Benchmarks Track |
| **Institution** | ETH Zurich, INSAIT |

> *"We find strong signs of contamination in AIME 2024. Nonetheless, evaluations on harder competitions, such as CMIMC 2025, demonstrate impressive reasoning capabilities in top-performing models."*

> *"On USAMO 2025, even top models score below 25%, far behind their performance on final-answer problems."*

---

## Leaderboards & Empirical Results

### HLE vs. Other Benchmarks Comparison (Early 2026)

| Benchmark | Top Model Score | Human Expert | Saturation Status |
|:---|:---:|:---:|:---:|
| **HLE** | ~65% (Claude Mythos) | ~90% | **Not saturated** |
| **MMLU** | >90% | ~90% | **Saturated** |
| **GPQA Diamond** | ~95% | ~80% | Nearly saturated |
| **AIME 2025** | ~95% (GPT-5) | ~40% (top 1%) | Nearly saturated |
| **USAMO/IMO** | <25% | ~50–90% | **Not saturated** |
| **MathArena Apex** | 5.2% | ~N/A (elite tier) | **Not saturated** |

### AIME 2025 Standings

| Model | AIME 2025 Accuracy |
|:---|:---:|
| GPT-5 (full) | **~95%** |
| GPT-OSS-120B | **92.6%** |
| Grok-4 | **~91%** |
| DeepSeek-R1 | 74.0% |
| Claude 3.7 Sonnet | 52.7% |

---

## Open-Source Repositories & Tools

| Tool/Repo | URL | Description | License |
|:---|:---|:---|:---|
| **MathArena Code** | [eth-sri/matharena](https://github.com/eth-sri/matharena) | Evaluation pipeline for math competitions | MIT |
| **MathArena Datasets** | [huggingface.co/MathArena](https://huggingface.co/MathArena) | Full model outputs and logs | CC-BY-NC-SA 4.0 |
| **HLE Dataset** | [lastexam.ai](https://lastexam.ai) | 2,500 public questions + private held-out set | — |
| **Scale HLE Leaderboard** | [labs.scale.com/leaderboard/humanitys_last_exam](https://labs.scale.com/leaderboard/humanitys_last_exam) | Official leaderboard with calibration metrics | — |
| **HLE Text-Only Leaderboard** | [labs.scale.com/leaderboard/humanitys_last_exam_text_only](https://labs.scale.com/leaderboard/humanitys_last_exam_text_only) | Text-only subset (86% of dataset) | — |

---

## Blog Posts & Articles

| Article | Source | Key Quote |
|:---|:---|:---|
| **IMO 2025 Evaluation Blog** | [matharena.ai/imo](https://matharena.ai/imo/) | *"Gemini 2.5 Pro: 31% — well below bronze medal threshold (19/42)."* |
| **MathArena Apex Blog** | [matharena.ai/apex](https://matharena.ai/apex/) | *"By design, the best model scores only 5.2%."* |
| **HLE Paper Review** | [sulbhajain.medium.com](https://sulbhajain.medium.com) | Overview of HLE design and significance |

---

## Key Findings

1. **HLE Remains Unsaturated.** Even Claude Mythos Preview scores ~65%, far below human expert ~90%. It is the de facto frontier closed-ended benchmark.

2. **Math Competition Saturation Is Real.** AIME 2025 and HMMT are nearly solved by top models (90–95%), necessitating harder benchmarks like MathArena Apex and proof-based evaluations.

3. **Proof-Writing Is the New Frontier.** While final-answer math problems are being saturated, proof-based olympiad problems (USAMO, IMO) remain extremely difficult — top models score <25% and fail to reach bronze medal thresholds.

4. **Calibration Is Terrible.** All frontier models exhibit extreme overconfidence on HLE, with RMS calibration errors >70–80% while scoring <10% — a clear signal of hallucination/confabulation.

5. **Contamination Is Critical.** MathArena provides strong evidence of contamination in AIME 2024. Real-time evaluation on newly released problems is essential.

6. **Cost-Accuracy Tradeoffs Vary Widely.** On MathArena, Grok 4 Fast achieves 90.6% average accuracy at only **$0.18/problem**, while GPT-5 (High) costs $4.83/problem.

---

## Related Topics

- [Compute-to-Reasoning Ratio](./Compute-to-Reasoning-Ratio.md) — Overhead and efficiency of Chain-of-Thought reasoning
- [Internal Chain-of-Thought](./Internal-Chain-of-Thought.md) — Hidden reasoning processes inside LLMs
