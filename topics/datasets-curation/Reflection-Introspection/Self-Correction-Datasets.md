# Self-Correction Datasets

## 📌 Overview

> Self-correction datasets teach models to **identify and fix errors in their own outputs**. These datasets typically contain structured traces: an initial attempt, a self-critique, and a refined answer — forming the backbone of self-correction training.

---

## 📚 Key Datasets

### 1. ReTrace (Reflexion Framework)

**Paper:** *"Reflexion: Language Models that Think Twice for Internalized Self-Correction"*  
**URL:** [OpenReview](https://openreview.net/pdf?id=FDG2G7JDWO) (ICLR 2026)

| Property | Value |
|----------|-------|
| Size | 200,000 structured self-correction examples |
| Structure | Q → Initial Thought (I) → Self-Critique (C) → Refined Answer (R) |
| Teacher Model | GPT-4o |
| Seed Prompts | Open-Orca dataset |
| Avg Token Counts | Q: 85.2, I: 155.6, C: 68.1, R: 239.4 |

**Task Coverage:** Creative writing, code generation, math reasoning, factual Q&A, instruction-following

---

### 2. SCoRe (Self-Correction via Reinforcement Learning)

**Paper:** *"Training Language Models to Self-Correct via Reinforcement Learning"*  
**URL:** [ICLR 2025](https://proceedings.iclr.cc/paper_files/paper/2025/file/871ac99fdc5282d0301934d23945ebaa-Paper-Conference.pdf)  
**Authors:** Google DeepMind (Aviral Kumar et al.)

> *"Multi-turn online RL using self-generated data; avoids distribution shift problem that plagues SFT methods."*

**Two-Stage Process:**
1. **Stage I:** Trains initialization that decouples attempts
2. **Stage II:** Multi-turn RL with reward shaping

**Results:**
- Improved Δ(t1,t2) by **15.6% (absolute)** on MATH benchmark
- Reduced correct-to-incorrect transitions from **15.8% → 1.4%**

---

### 3. S³C-MATH (Spontaneous Step-level Self-correction)

**Paper:** *"S³C-Math: Spontaneous Step-level Self-correction Makes Large Language Models Better Mathematical Reasoners"*  
**URL:** [arXiv:2409.01524](https://arxiv.org/abs/2409.01524) (AAAI 2025)

> *"Self-correction achieves 5.2% accuracy gains on MATH dataset."*

**Method:** Fine-tuning strategy with loss-mask to train models for spontaneous step-level self-correction in math reasoning.

---

### 4. Self-Refine

**Paper:** *"Self-Refine: Iterative Refinement with Self-Feedback"*  
**URL:** [arXiv:2303.17651](https://arxiv.org/abs/2303.17651) (Madaan et al., 2023)

**Method:** Initial output → same LLM provides feedback → uses feedback to refine iteratively

> *"~20% absolute improvement across evaluated tasks."*

**Tasks:** Dialog response generation, mathematical reasoning, code generation, and more

---

### 5. STaR (Self-Taught Reasoner)

**Paper:** *"STaR: Self-Taught Reasoner"*  
**URL:** [OpenReview](https://openreview.net/pdf?id=_3ELRdg2sgI)

| Split | Size |
|-------|------|
| Training | 9,741 |
| Validation | 1,221 |
| Test | 1,285 |

**Method:** Model attempts to solve problems, then rationalizes failures to improve — iterative process.

---

### 6. STaSC (Self-Taught Self-Correction for Small Language Models)

**Paper:** *"Self-Taught Self-Correction for Small Language Models"*  
**URL:** [arXiv:2503.08681](https://arxiv.org/abs/2503.08681) (ICLR 2025)

> *"Iterative fine-tuning using exclusively self-generated data."* — Targets **small language models (SLMs)**

**Code:** [github.com/VityaVitalich/STASC](https://github.com/VityaVitalich/STASC)

---

## 📊 Benchmarks for Self-Correction

### CorrectBench
**URL:** [https://correctbench.github.io/](https://correctbench.github.io/)  
**Venue:** NeurIPS 2025 Datasets and Benchmarks Track  
**Domains:** Commonsense reasoning, mathematical reasoning, code generation

> *"Self-correction achieves 5.2% accuracy gains on complex math; hybrid approaches are ~40% slower."*

---

### CriticBench
**Paper:** *"CriticBench: Benchmarking LLMs for Critique-Correct Reasoning"*  
**URL:** [arXiv:2402.14809](https://arxiv.org/abs/2402.14809) (ACL 2024 Findings)

**Domains:** GSM8K, MATH, AQuA, CSQA, HotpotQA, BIG-Bench Hard, MBPP, HumanEval

**Structure:** Assesses **GQC reasoning** — Generation, Critique, and Correction reasoning.

---

### Self-Correction Bench
**Venue:** NeurIPS 2025

> *"64.5% average 'blind spot rate' — models fail to correct errors in their own outputs."*

**Discovery:** Simply appending "Wait" reduces blind spots by **89.3%**.

---

### MINT Benchmark
**URL:** [https://github.com/xingyaoww/mint-bench](https://github.com/xingyaoww/mint-bench)  
**Venue:** ICLR 2024

**Focus:** Multi-turn interaction with tools and natural language feedback.

---

### CRITIC Framework
**Paper:** *"CRITIC: Large Language Models Can Self-Correct with Tool-Interactive Critiquing"*  
**URL:** [OpenReview](https://openreview.net/forum?id=Sx038qxjek)

**Method:** Uses external tools (search, code interpreter) to validate and correct outputs.

---

## 🔄 Comparison Table

| Dataset | Size | Target | Key Method |
|---------|------|--------|------------|
| **ReTrace** | 200k | General | Structured Q→I→C→R traces |
| **SCoRe** | — | Math | Multi-turn RL, reward shaping |
| **S³C-MATH** | — | Math | Loss-mask fine-tuning |
| **Self-Refine** | — | Multi-task | Iterative self-feedback |
| **STaR** | ~12k | Reasoning | Rationalize failures |
| **STaSC** | — | Small LMs | Self-generated data only |
