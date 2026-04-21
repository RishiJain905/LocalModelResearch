# Data Contamination Reporting

> Detecting, measuring, and reporting when AI models have been trained on benchmark or test data — covering contamination detection methods, tools, benchmarks, contamination rates, and evaluation integrity practices.

## Contents

1. [Key Papers](#key-papers)
2. [Detection Methods & Taxonomy](#detection-methods--taxonomy)
3. [Open-Source Tools & Repositories](#open-source-tools--repositories)
4. [Contamination Rates & Benchmarks](#contamination-rates--benchmarks)
5. [Blog Posts & Articles](#blog-posts--articles)
6. [Key Findings](#key-findings)

---

## Key Papers

### Time Travel in LLMs: Tracing Data Contamination (ICLR 2024 Spotlight)

> *"Time Travel in LLMs: Tracing Data Contamination in Large Language Models"*
> — Shahriar Golchin, Mihai Surdeanu
> — ICLR 2024 (Spotlight)
> — [arXiv:2312.00151](https://arxiv.org/abs/2312.00151) | [GitHub](https://github.com/shahriargolchin/time-travel-in-llms)

**Method:** Black-box contamination detection via "guided instruction" prompting.
- 92–100% accuracy on GPT-4, ChatGPT, and Vicuna
- No model access required — works via API

> *"The key insight is that contaminated models cannot consistently 'unsee' instructions they've seen during training — they exhibit temporal confusion when probed with future-dated benchmarks."*

---

### Data Contamination Quiz: Estimating Contamination Amount (TACL/ACL 2025)

> *"Data Contamination Quiz: A Tool to Detect and Estimate Contamination in Large Language Models"*
> — Shahriar Golchin, Mihai Surdeanu
> — TACL/ACL 2025
> — [arXiv:2403.17911](https://arxiv.org/abs/2403.17911) | [GitHub](https://github.com/shahriargolchin/DCQ)

First method to **estimate the amount** of contamination in black-box LLMs — not just detect it.
- DCQ (Data Contamination Quiz) approach using multiple-choice probing
- GPT-4 found **56% actual contamination** vs. OpenAI's reported **25%** on XSum benchmark

> *"The Data Contamination Quiz can quantify contamination levels — revealing that self-reported contamination rates may significantly underestimate the true extent."*

---

### LLMSanitize: Comprehensive Contamination Survey (arXiv:2404.00699)

> *"How Much are LLMs Contaminated? A Comprehensive Survey and the LLMSanitize Library"*
> — Ravaut et al.
> — arXiv:2404.00699 (2024)
> — [GitHub](https://github.com/ntunlp/LLMSanitize)

**Taxonomy:** Data contamination vs. model contamination.

Contamination rate findings across benchmarks: **1% to 45%**.

> *"We systematically categorize contamination into data contamination (dataset in training) vs. model contamination (model seen evaluation data) — and find rates ranging from 1% to 45% across benchmarks."*

---

### Benchmark Data Contamination Survey (arXiv:2406.04244)

> *"Benchmark Data Contamination of Large Language Models: A Survey"*
> — Xu et al.
> — arXiv:2406.04244 (June 2024)

BDC taxonomy — **semantic→label severity spectrum**:
- Level 1: Semantic similarity (text overlap)
- Level 2: Label contamination (answers leaked)
- Level 3: Full memorization

Covers detection methods and mitigation strategies.

---

### Comprehensive Survey on Data Contamination (arXiv:2502.14425)

> *"A Survey on Data Contamination for Large Language Models"*
> — arXiv:2502.14425 (February 2025)

Comprehensive survey covering:
- Definitions and taxonomy
- Detection methods (white-box, gray-box, black-box)
- Contamination-free evaluation practices

---

### ConStat: Performance-Based Contamination Detection (NeurIPS 2024)

> *"Performance-Based Contamination Detection in LLMs"*
> — Dekoninck et al.
> — NeurIPS 2024
> — [GitHub](https://github.com/uid/ConStat)

**Novel definition:** Contamination = *inflated non-generalizing performance*

Detects contamination via performance comparison between:
- Reference models trained without contamination
- Target model performance on suspected contaminated benchmarks

> *"ConStat reframes contamination detection around performance inflation — if a model's performance only generalizes to data it has seen, that's contamination."*

---

### Clean Evaluation on Contaminated LLMs — CleanEval (NAACL 2024 Findings)

> *"Clean Evaluation on Contaminated Large Language Models"*
> — Zhu et al.
> — NAACL 2024 Findings
> — [arXiv:2311.14754](https://arxiv.org/abs/2311.14754)

Uses paraphrasing and back-translation to clean contaminated data:
- Paraphrase test set examples
- Compare model performance on original vs. paraphrased
- Significant performance drops indicate contamination

---

### LLM Decontaminator (Yang et al., 2023)

> *"Rethinking Benchmark and Contamination for Language Models with Rephrased Samples"*
> — Yang et al.
> — arXiv:2311.04850 (2023)
> — [GitHub](https://github.com/lm-sys/llm-decontaminator)

Key finding: **N-gram and embedding methods fail on paraphrased samples.**

| Dataset | Rephrased Contamination Rate |
|---------|------------------------------|
| HumanEval (The Stack) | 18.9% |
| StarCoder-Data | 15.9% |
| CodeAlpaca | 12.8% |
| RedPajama | 8.5% |

> *"A 13B Llama model rephrasing achieves GPT-4-level performance on HumanEval — revealing the test set was in the training data."*

---

### Min-K% Prob: Detecting Pretraining Data (arXiv:2310.16789)

> *"Min-K% Prob: Detecting Pretraining Data from Large Language Models"*
> — Shi et al.
> — arXiv:2310.16789 (2023)
> — [GitHub](https://github.com/swj0419/detect-pretrain-code)

Membership inference method based on token probability distributions:
- Identifies known training examples from model outputs
- **WikiMIA benchmark** for evaluating membership inference attacks

Applied to GPT-3: detected top 20 copyrighted books in training data.

---

### Evading Contamination Detection is Too Easy (ICLR 2025)

> *"Evading Data Contamination Detection for Language Models is (too) Easy"*
> — Dekoninck et al.
> — arXiv:2402.02823 | ICLR 2025

**EAL (Evasive Augmentation Learning):** Simple paraphrasing/translation evades standard detection methods.

> *"We demonstrate that contamination detection is fragile — even naive rephrasing of training data can evade n-gram and embedding-based detection methods."*

---

### Open-Source Contamination Report for LLMs (EMNLP 2024 Findings)

> *"An Open-Source Data Contamination Report for Large Language Models"*
> — Li et al.
> — EMNLP 2024 Findings

15 LLMs × 6 benchmarks analysis:
- Contamination range: **1% to 45%**
- Larger models gain more advantage on contaminated sets
- C-Eval shows +14% inflation, HellaSwag +7%, MMLU minimal

---

## Detection Methods & Taxonomy

### Detection Paradigms

| Paradigm | Access Required | Methods |
|----------|----------------|---------|
| **White-box** | Full model weights | Influence functions, Min-K% Prob, gradient analysis |
| **Gray-box** | Token probabilities | Sharded likelihood, embedding similarity |
| **Black-box** | API only | Guided prompting (Time Travel), DCQ, reference-based comparison |

---

### Open-Data Methods (No Model Access)

| Method | Type | Limitation |
|--------|------|-----------|
| **8-gram overlap** | String matching | Fails on paraphrasing |
| **13-gram overlap** (GPT-3) | String matching | Higher threshold |
| **Embedding similarity** | Semantic | Fails on rephrasing |
| **GPT-4 verification** | LLM-based | Computationally expensive |
| **Paraphrase/back-translate** | Augmentation | Clean-Eval approach |

---

### White/Gray-Box Methods

| Method | Approach | Reference |
|--------|----------|-----------|
| **Min-K% Prob** | Low-probability tokens signal membership | Shi et al. 2023 |
| **Sharded Likelihood** | Partition model outputs by likelihood | Various |
| **CDD (Contrastive Decoder)** | Memorization vs. likelihood comparison | Various |
| **Influence Functions** | Trace predictions to training samples | Koh & Liang 2020 |
| **ConStat** | Performance inflation vs. reference | Dekoninck et al. 2024 |

---

## Open-Source Tools & Repositories

### awesome-data-contamination

> [github.com/lyy1994/awesome-data-contamination](https://github.com/lyy1994/awesome-data-contamination)

Curated paper list — **122 papers** on data contamination for LLMs.

---

### LLMSanitize

> [github.com/ntunlp/LLMSanitize](https://github.com/ntunlp/LLMSanitize)

Open-source library implementing major contamination detection algorithms:
- String matching (n-gram)
- Embedding-based
- Guided prompting (Time Travel)
- Min-K% Prob
- And more

---

### Time Travel in LLMs

> [github.com/shahriargolchin/time-travel-in-llms](https://github.com/shahriargolchin/time-travel-in-llms)

ICLR 2024 paper implementation — guided instruction detection for black-box LLMs.

---

### DCQ (Data Contamination Quiz)

> [github.com/shahriargolchin/DCQ](https://github.com/shahriargolchin/DCQ)

TACL/ACL 2025 paper — estimates contamination **amount** in black-box LLMs.

---

### LLM Decontaminator

> [github.com/lm-sys/llm-decontaminator](https://github.com/lm-sys/llm-decontaminator)

Embedding retrieval + GPT-4 verification pipeline. Quantifies rephrased sample contamination.

---

### Contamination Detector

> [github.com/liyucheng09/contamination_detector](https://github.com/liyucheng09/contamination_detector)

Uses Bing search + CommonCrawl index:
- Categorizes test samples: Clean / Input-only contaminated / Input-and-label contaminated
- Open-data approach (no model needed)

---

### LM Contamination Index

> [github.com/hitz-zentroa/lm-contamination](https://github.com/hitz-zentroa/lm-contamination)

Manually curated database of contamination evidence across models and benchmarks.

---

### LiveBench

> [livebench.ai](https://livebench.ai)

Monthly-updated benchmark to avoid contamination:
> *"LiveBench is designed to be contamination-resistant — new problems are added monthly, before any model can be trained on them."*

---

## Contamination Rates & Benchmarks

### Contamination Rates by Study

| Study | Range | Key Benchmark |
|-------|-------|---------------|
| Li et al. EMNLP 2024 | 1–45% | Multiple |
| Yang et al. 2023 | 8.5–18.9% | HumanEval, CodeAlpaca |
| Golchin & Surdeanu DCQ | 56% (GPT-4, XSum) | XSum |
| ConStat NeurIPS 2024 | High | Mistral, Llama, Yi |

### Performance Inflation by Benchmark

| Benchmark | Inflation (approx.) | Notes |
|-----------|--------------------|-------|
| C-Eval | +14% | Significant |
| HellaSwag | +7% | Moderate |
| MMLU | Minimal | Slight |
| HumanEval | Varies | Code contamination |
| XSum | 56% (DCQ) | GPT-4 |

### Temporal Evidence of Contamination

> *"GPT-4 is significantly worse on Codeforces problems post-September 2021 — earlier success was partially memorization."* — Huang et al.

---

## Blog Posts & Articles

1. **The New Stack** — *"How to Detect and Clean up Data Contamination in LLMs"* — [thenewstack.io](https://www.thenewstack.io/how-to-detect-and-clean-up-data-contamination-in-llms/)
2. **Holisticai** — *"An Overview of Data Contamination: Causes, Risks, Signs, and Defenses"* — [holisticai.io](https://holisticai.io/blog)
3. **LMSYS Blog** — *"Catch me if you can! How to beat GPT-4 with a 13B model"* — [lmsys.org](https://lmsys.org/blog/) — Llama rephraser achieving GPT-4-level via rephrased test sets
4. **Medium** — *"Is AI Cheating on the Test: Data Contamination, Gaming and the Benchmark Crisis"* — [medium.com/@jarekwasowski](https://medium.com/@jarekwasowski)

---

## Key Findings

1. **Contamination is widespread but uneven** — Ranges from 1–45% depending on benchmark and model; larger models memorize more and gain greater advantage from contaminated data

2. **Black-box detection is feasible** — Time Travel (ICLR 2024) achieves 92–100% accuracy via guided prompting with no model access needed

3. **Simple evasion defeats simple detection** — N-gram and embedding methods fail completely against paraphrased training data; only LLM-as-judge approaches remain robust

4. **Self-reported rates are underestimates** — DCQ found GPT-4 had 56% actual contamination on XSum vs. OpenAI's reported 25%

5. **Dynamic benchmarks are the solution** — LiveBench and similar monthly-updated benchmarks prevent contamination by staying ahead of training windows

6. **Scale effect amplifies contamination** — Larger models gain more advantage from contaminated training data, making evaluation on static benchmarks increasingly unreliable

---

## Related Topics

- [Dataset Composition Analysis](./Dataset-Composition-Analysis.md) — Documentation and provenance
- [Copyright & Public Domain Disclosures](./Copyright-Public-Domain-Disclosures.md) — Rights and licensing
- [Benchmarks](../benchmarks/) — Evaluation frameworks
