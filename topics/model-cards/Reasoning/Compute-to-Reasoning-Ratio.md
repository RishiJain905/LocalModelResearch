# Compute-to-Reasoning Ratio

> *"Chain-of-thought prompting is an emergent property of model scale."*
> — Wei et al., NeurIPS 2022

The **Compute-to-Reasoning Ratio** refers to the computational overhead incurred by Chain-of-Thought (CoT) reasoning relative to direct-answer generation. Because CoT forces LLMs to generate intermediate reasoning steps before the final answer, inference cost, latency, and memory usage scale linearly with the length of the reasoning trace.

## Contents

1. [Key Papers](#key-papers)
2. [Open-Source Libraries & Tools](#open-source-libraries--tools)
3. [GitHub Repositories](#github-repositories)
4. [Blog Posts & Articles](#blog-posts--articles)
5. [Key Findings](#key-findings)

---

## Key Papers

### Chain-of-Thought Prompting Elicits Reasoning in Large Language Models (NeurIPS 2022)

> *"Simply increasing the scale of the model does not substantially improve performance on these benchmarks [with standard prompting]... chain of thought prompting unlocks substantially improved performance on complex reasoning tasks."*
> — Jason Wei, Xuezhi Wang, et al.
> — [arXiv:2201.11903](https://arxiv.org/abs/2201.11903)

**Key Contributions:**
- CoT is emergent at ~100B parameters
- Solves math, commonsense, and symbolic tasks standard prompting cannot

---

### Scaling LLM Test-Time Compute Optimally (ICLR 2025)

> *"Using this compute-optimal strategy, we can improve the efficiency of test-time compute scaling for math reasoning problems by more than 4× compared to a best-of-N baseline."*
> — Snell, Lee, Xu, Kumar
> — [OpenReview](https://openreview.net/forum?id=4FWAwZtd2n)

**Key Contributions:**
- Compute-optimal test-time scaling improves efficiency **>4× vs. best-of-N**
- Optimal compute allocation depends on question difficulty

---

### Token-Budget-Aware LLM Reasoning — TALE (arXiv 2024)

> *"LLM reasoning can be compressed by prompting with explicit token budgets; 'token elasticity' means too-small budgets cause rebound usage."*
> — Han et al.
> — [arXiv:2412.18547](https://arxiv.org/abs/2412.18547)

**Key Contributions:**
- "Token elasticity": too-small budgets cause models to generate *more* tokens
- Zero-shot EP + post-training compression

---

### OckBench: Measuring the Efficiency of LLM Reasoning (arXiv 2025)

> *"Open-source models consume up to 5.1× more tokens than commercial models for comparable accuracy; accuracy-only benchmarks mask efficiency differences."*
> — [arXiv:2511.05722](https://arxiv.org/abs/2511.05722) | [ockbench.github.io](https://ockbench.github.io/)

**Key Contributions:**
- Model-agnostic benchmark measuring accuracy + decoding token count jointly
- Defines **Reasoning Efficiency = Tokens per accuracy point**

---

### s1: Simple Test-Time Scaling (EMNLP 2025)

> *"Budget forcing (appending 'Wait') prolongs reasoning; under-$50 training recipe matches o1-preview on math with test-time adaptivity."*
> — Muennighoff et al.
> — [arXiv:2501.19393](https://arxiv.org/abs/2501.19393)

**Key Contributions:**
- "Budget forcing" controls reasoning depth at inference time
- Matches o1-preview at < $50 training cost

---

### Early Stopping Chain-of-Thought — ES-CoT (ICLR 2025)

> *"ES-CoT reduces the number of inference tokens by about 41% on average while maintaining accuracy comparable to standard CoT."*
> — [OpenReview](https://openreview.net/pdf?id=vuMabnSok0)

**Key Contributions:**
- Detects answer-convergence at inference time
- Reduces CoT inference tokens by **~41%**

---

### Not All Tokens Are What You Need In Thinking (arXiv 2025)

> *"Conditional Token Selection (CTS) prunes up to 75.8% of reasoning tokens with only 5% accuracy drop by conditioning on question+answer."*
> — Yuan et al.
> — [arXiv:2505.17827](https://arxiv.org/abs/2505.17827)

**Key Contributions:**
- CTS prunes up to **75.8%** of reasoning tokens
- Only **5%** accuracy drop

---

### Extra-CoT: Extreme-Ratio Chain-of-Thought Compression (arXiv 2026)

> Three-stage framework compresses CoT to **20% tokens** while preserving reasoning fidelity and delivering real wall-clock speedups.
> — Tang et al.
> — [arXiv:2602.08324](https://arxiv.org/abs/2602.08324) | [GitHub](https://github.com/Mwie1024/Extra-CoT)

---

### A Principled Analysis of Chain-of-Thought Reasoning (arXiv 2025)

> Identifies a **7B parameter threshold**: below 7B, direct answers are often Pareto-optimal; above 7B, CoT is essential.
> — Boizard
> — [PDF](https://arxiv.org/pdf/2509.22193)

---

### To CoT or not to CoT? (ICLR 2025)

> *"On MMLU, directly generating the answer without CoT leads to almost identical accuracy as CoT unless the question contains symbolic operations."*
> — Sprague et al.
> — [ICLR Poster](https://iclr.cc/virtual/2025/poster/27873)

**Key Contributions:**
- Meta-analysis of 100+ papers
- CoT benefits mainly math/logic; on multiple-choice, direct ≈ CoT

---

### Additional Papers

| Paper | Authors | Year | Key Finding |
|-------|---------|------|-------------|
| A Principled Analysis of CoT | Boizard | 2025 | 7B threshold for CoT benefit; math tasks only show gain at scale |
| To CoT or not to CoT? | Sprague et al. | 2025 | Direct answering matches CoT on multiple-choice; symbolic tasks benefit |

---

## Open-Source Libraries & Tools

| Tool | Description | Link |
|------|-------------|------|
| **TALE** | Token-Budget-Aware LLM Reasoning framework | [GeniusHTX/TALE](https://github.com/GeniusHTX/TALE) |
| **Extra-CoT** | Extreme-ratio CoT compression (20% budgets) | [Mwie1024/Extra-CoT](https://github.com/Mwie1024/Extra-CoT) |
| **OckBench** | Model-agnostic benchmark for reasoning efficiency | [ockbench.github.io](https://ockbench.github.io/) |
| **Awesome-Efficient-Reasoning** | Curated paper list | [hemingkx/Awesome-Efficient-Reasoning](https://github.com/hemingkx/Awesome-Efficient-Reasoning) |
| **Awesome-Efficient-CoT-Reasoning-Summary** | Efficient CoT methods with code links | [zwxandy/Awesome-Efficient-CoT-Reasoning-Summary](https://github.com/zwxandy/Awesome-Efficient-CoT-Reasoning-Summary) |
| **LM Evaluation Harness** | Standard benchmarking framework | [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) |
| **vLLM** | High-throughput serving; supports reasoning workflows | [vllm-project/vllm](https://github.com/vllm-project/vllm) |

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [GeniusHTX/TALE](https://github.com/GeniusHTX/TALE) | — | Token-budget-aware reasoning |
| [Mwie1024/Extra-CoT](https://github.com/Mwie1024/Extra-CoT) | — | Extreme-ratio CoT compression |
| [simplescaling/s1](https://github.com/simplescaling/s1) | — | Simple test-time scaling |
| [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) | ⭐ 16k+ | Standard LLM benchmarking |
| [vllm-project/vllm](https://github.com/vllm-project/vllm) | ⭐ 43k+ | High-throughput LLM serving |

---

## Blog Posts & Articles

| Article | Source | Key Quote |
|---------|--------|-----------|
| **Measuring Thinking Efficiency in Reasoning Models** | Nous Research | *"Open-weight models use 1.5–4× more tokens than closed-weight models, ballooning to up to 10× for simple knowledge questions."* |
| **Unlocking LLM Performance: A Principled Analysis of CoT** | Diabolocom Research | *"Reasoning is not a universal enhancement but a targeted tool whose value scales with task complexity."* |
| **Language Models Perform Reasoning via Chain of Thought** | Google Research Blog (2022) | Original CoT announcement |
| **The State of LLM Reasoning and Inference Scaling** | Sebastian Raschka (2025) | Comprehensive survey |
| **Chain-of-Thought with a Token Budget** | Benjamin Marie (2025) | TALE analysis |
| **Inside s1: An o1-Style Reasoning Model Under $50** | TWIML Podcast | Budget forcing deep dive |

---

## Key Findings

### Token/S Efficiency vs. Accuracy Trade-offs

- **CoT overhead is massive.** Reasoning models often generate thousands of tokens for problems solvable in dozens. OckBench reports open-weight models average **20,000+** reasoning tokens on math while commercial models use **2,000–5,000**.
- **Per-token intelligence** is the emergent frontier metric. GPT-5 scores 32.2 tokens/accuracy pt on math; Qwen3-4B-Thinking scores **649.3** — a **20× span**.
- **Token elasticity:** TALE finds too-small budgets cause models to "give up" and generate *more* tokens — cost bottoms in a plateau.
- **Selective CoT:** On multiple-choice/simple tasks, direct answering ≈ CoT accuracy. CoT should be applied selectively.

### Compute Ratios: CoT vs. Direct Answering

| Metric | CoT | Direct | Ratio |
|--------|-----|--------|-------|
| Tokens (math, open-source) | 5,000–27,000 | 100–500 | **10–50×** |
| Open-source vs. commercial | ~20,000 avg | ~2,000–5,000 | **5.1×** |
| Simple knowledge Qs | ~400–2,000 | ~10–50 | **1.5–10×** |
| CTS compression (α=0.5) | baseline | −75.8% tokens | **4× reduction** |
| ES-CoT early stopping | baseline | −41% tokens | **1.7× reduction** |

### Cost and Scaling

- **Closed-source lead:** OpenAI/Gemini models are heavily optimized; open-weight models output full traces. Magistral uses up to **10×** more tokens than the most efficient closed models.
- **Test-time compute trade-off:** Snell et al. show adaptive test-time compute outperforms a **14× larger model** on easy/medium questions in FLOPs-matched evals.
- **7B threshold:** Below ~7B params, direct fine-tuning is often Pareto-optimal; above it, CoT is essential.
- **Scaling to longer CoT:** Frontier models can require **over 10 hours** to solve six math problems; reasoning output grows at **~5× per year**.

### Open-Source Implications

- **gpto-oss (20B, 120B):** Open-weight releases establish new per-token efficiency references.
- **Sky-T1-7B:** Benchmarked as most efficient open-source model in OckBench.
- **Tooling gap:** CoT-aware benchmarking tools remain fragmented vs. accuracy-only frameworks.

---

> *"On MMLU, directly generating the answer without CoT leads to almost identical accuracy as CoT unless the question or model's response contains an equals sign, indicating symbolic operations and reasoning."*
> — Sprague et al., ICLR 2025
