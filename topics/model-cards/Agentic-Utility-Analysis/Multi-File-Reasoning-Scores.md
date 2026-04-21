# Multi-File Reasoning Scores

> "Resolving issues in SWE-bench frequently requires understanding and coordinating changes across multiple functions, classes, and even files simultaneously, calling for models to interact with execution environments, process extremely long contexts and perform complex reasoning that goes far beyond traditional code generation." — Jimenez et al., ICLR 2024

## Contents

1. [Key Papers](#key-papers)
2. [Open-Source Libraries & Tools](#open-source-libraries--tools)
3. [GitHub Repositories](#github-repositories)
4. [Blog Posts & Articles](#blog-posts--articles)
5. [Key Findings](#key-findings)

---

## Key Papers

### SWE-bench: Can Language Models Resolve Real-World GitHub Issues? (2024)

> *"SWE-bench evaluates language models' capabilities in resolving real-world software engineering issues by requiring them to understand and modify code across multiple files."*
> — Carlos E. Jimenez, John Yang, Alexander Wettig, Shunyu Yao, Kexin Pei, Ofir Press, Karthik R. Narasimhan
> — ICLR 2024
> — [arXiv:2310.06770](https://arxiv.org/abs/2310.06770) | [GitHub](https://github.com/SWE-bench/SWE-bench)

**Key Contributions:**
- Introduced the first large-scale benchmark for issue resolution requiring multi-file understanding, built from 2,294 real GitHub issues across 12 popular Python repositories.
- Showed that existing models (Claude 2, GPT-3.5, GPT-4) resolve only the simplest issues; even GPT-4 with oracle file retrieval scores modestly.
- Demonstrated that real-world fixes require cross-file dependency tracking, long-context comprehension, and execution feedback.

---

### SWE-Bench Pro: Can AI Agents Solve Long-Horizon Software Engineering Tasks? (2025)

> *"The benchmark contains complex, long-horizon tasks requiring edits across multiple files and repositories."*
> — Scale AI SEAL Lab
> — arXiv preprint
> — [arXiv:2509.16941](https://arxiv.org/abs/2509.16941) | [Leaderboard](https://labs.scale.com/leaderboard/swe_bench_pro_public)

**Key Contributions:**
- 1,865 long-horizon tasks from 41 actively maintained repositories across Python, Go, TypeScript, and JavaScript.
- Average task complexity: ~107 lines of code across ~4 files (vs. 2–3 files in Verified).
- Top models drop from 70–80% on Verified to ~23% on Pro, revealing the gap between single-file benchmarks and real-world multi-file engineering.

---

### Multi-SWE-bench: A Multilingual Benchmark for Issue Resolving (2025)

> *"With its wide coverage of languages and issue types, Multi-SWE-bench introduces realistic challenges that push the boundaries of LLM-based software agents."*
> — ByteDance Seed / OpenReview
> — [arXiv:2504.02605](https://arxiv.org/abs/2504.02605) | [Hugging Face](https://huggingface.co/datasets/ByteDance-Seed/Multi-SWE-bench) | [GitHub](https://github.com/multi-swe-bench)

**Key Contributions:**
- 1,632 human-verified instances across 7 languages: Java, TypeScript, JavaScript, Go, Rust, C, and C++.
- 77.1% of tasks rated medium or hard; significant performance degradation on cross-file issues.
- Evaluated 9 frontier models across agent frameworks (Agentless, SWE-agent, OpenHands) and found Python > Java > Go/Rust > C/C++ > TS/JS performance hierarchy.
- Launched Multi-SWE-RL community with 4,723 containerized instances for reinforcement learning.

---

### MultiFileTest: A Multi-File-Level LLM Unit Test Generation Benchmark (2025)

> *"To address such a limitation, we propose MultiFileTest, a multi-file-level benchmark for unit test generation covering Python, Java, and JavaScript."*
> — ACL 2026 Findings
> — [arXiv:2502.06556](https://arxiv.org/abs/2502.06556)

**Key Contributions:**
- Focuses specifically on unit test generation requiring reasoning across multiple files (not just isolated function generation).
- Covers Python, Java, and JavaScript projects with realistic cross-file dependencies.
- Analyzes impact of error-fixing mechanisms on multi-file test generation quality.

---

### SWE-QA: Can Language Models Answer Repository-level Code Questions? (2025)

> *"Existing benchmarks target isolated code snippets, functions, or APIs. These benchmarks fail to capture the complexity of real-world repositories, including architecture, cross-file dependencies, lifecycle flows, and design rationales."*
> — arXiv preprint
> — [arXiv:2509.14635](https://arxiv.org/abs/2509.14635)

**Key Contributions:**
- 576 human-verified Q&A pairs across 12 Python repositories (>3M LOC).
- Targets cross-file reasoning, multi-hop dependency analysis, and intention understanding.
- Introduces SWE-QA-Agent, a ReAct-based agent with semantic RAG (`voyage-code-3`) for repository-level retrieval.

---

### SWE-bench Multimodal: Do AI Systems Generalize to Visual Software Domains? (2024)

> *"Our analysis finds that top-performing SWE-bench systems struggle with SWE-bench Multimodal, revealing limitations in visual problem-solving and cross-language reasoning."*
> — Princeton NLP
> — [arXiv:2410.03859](https://arxiv.org/abs/2410.03859)

**Key Contributions:**
- Extends SWE-bench to JavaScript visual/UI issues requiring multimodal reasoning.
- Shows that agents strong on text-only code struggle when UI elements and visual layout must be understood.

---

### CodePlan: Repository-level Coding using LLMs and Planning (2023)

> *"We frame repository-level coding as a planning problem and present a task-agnostic framework, called CodePlan to solve it."*
> — Microsoft Research / TOSEM
> — [arXiv:2309.12499](https://arxiv.org/abs/2309.12499) | [ACM DL](https://dl.acm.org/doi/10.1145/3643757)

**Key Contributions:**
- Treats repository-level editing as a planning problem with incremental dependency analysis and change may-impact analysis.
- Evaluated on package migration (C#) and temporal code edits (Python), showing the necessity of multi-step planning across files.

---

### RepoQA: Evaluating Long-Context Code Understanding (2024)

> *"RepoQA: Evaluating Long Context Code Understanding"*
> — EvalPlus team, ICML 2024 Workshop on Long Context FMs
> — [OpenReview](https://openreview.net/pdf?id=hK9YSrFuGf) | [GitHub](https://github.com/evalplus/repoqa)

**Key Contributions:**
- Uses a "Searching Needle Function" (SNF) task to test whether models can retrieve a function from a large repository given only its description.
- Measures long-context understanding as a prerequisite for multi-file reasoning.

---

## Open-Source Libraries & Tools

### SWE-bench Evaluation Harness

> [SWE-bench website](https://www.swebench.com/) | [swebench.com](https://www.swebench.com/)

The canonical evaluation framework for issue-resolving agents. Supports:
- Original SWE-bench (2,294 Python tasks)
- SWE-bench Verified (500 human-filtered tasks)
- SWE-bench Lite (300 curated tasks)
- SWE-bench Multilingual (300 tasks, 9 languages)
- SWE-bench Multimodal (visual JS tasks)

Primary metric: **%Resolved** (patch passes all tests without breaking existing ones).

---

### mini-SWE-agent

> [GitHub — SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent)

A 100-line minimal agent scaffold used by SWE-bench leaderboards for standardized comparisons. Top systems score >74% on Verified using this scaffold.

---

### EvalPlus / RepoQA

> [evalplus.github.io](https://evalplus.github.io/) | [repoqa.html](https://evalplus.github.io/repoqa.html)

Rigorous evaluation suite for code correctness (HumanEval+, MBPP+) and long-context code understanding (RepoQA). RepoQA specifically tests retrieval of "needle" functions across large repositories.

---

### OpenHands (formerly OpenDevin)

> [openhands.com](https://www.all-hands.dev/) | [docs](https://docs.all-hands.dev/)

An open-source agent framework supporting multiple LLMs and tool integrations. Used as a baseline in Multi-SWE-bench under the label `OpenHands + CodeAct v2.1`.

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [SWE-bench/SWE-bench](https://github.com/SWE-bench/SWE-bench) | ⭐ 3.4k+ | Canonical benchmark harness for evaluating LLMs on real-world GitHub issues. |
| [SWE-agent/mini-swe-agent](https://github.com/SWE-agent/mini-swe-agent) | ⭐ 3.9k | Minimal 100-line agent that scores >74% on SWE-bench Verified. |
| [multi-swe-bench/multi-swe-bench](https://github.com/multi-swe-bench/multi-swe-bench) | ⭐ N/A | Multilingual benchmark data production pipeline and Docker images. |
| [multi-swe-bench/experiments](https://github.com/multi-swe-bench/experiments) | ⭐ N/A | Results submission and reasoning traces for Multi-SWE-bench. |
| [OpenHands/OpenHands](https://github.com/OpenHands/OpenHands) | ⭐ 71.6k | OpenHands agent framework (formerly OpenDevin). |
| [princeton-nlp/SWE-agent](https://github.com/princeton-nlp/SWE-agent) | ⭐ 4.3k+ | Original SWE-agent system with custom agent-computer interfaces. |
| [evalplus/evalplus](https://github.com/evalplus/evalplus) | ⭐ 2k+ | Rigorous evaluation of LLM-synthesized code (NeurIPS 2023 / COLM 2024). |
| [evalplus/repoqa](https://github.com/evalplus/repoqa) | ⭐ N/A | Long-context code understanding benchmark (ICML 2024 workshop). |
| [CognitionAI/devin-swebench-results](https://github.com/CognitionAI/devin-swebench-results) | ⭐ N/A | Devin's evaluation harness and code edits on SWE-bench. |
| [multi-swe-bench/MopenHands](https://github.com/multi-swe-bench/MopenHands) | ⭐ N/A | OpenHands adaptation for Multi-SWE-bench multilingual evaluation. |

---

## Blog Posts & Articles

1. **Jatin Ganhotra** — *"The Multi-File Frontier: Why SWE-Bench Verified Doesn't Reflect Real-World Programming Challenges"* — [link](https://jatinganhotra.dev/blog/swe-agents/2025/03/30/swe-bench-verified-single-file-saturation.html) — Demonstrates that 85.8% of SWE-Bench Verified is single-file edits, while ~50% of real-world issues require multi-file changes; no system exceeds 30% resolution on multi-file tasks.

2. **Jatin Ganhotra** — *"Do SWE-Agents Solve Multi-File Issues Like Humans?"* — [link](https://jatinganhotra.dev/blog/swe-agents/2025/01/05/swe-bench-mutliple-files.html) — Deep-dive into which files agents actually modify versus human patches on multi-file SWE-bench instances.

3. **CodeAnt AI** — *"Understand the SWE-Bench Leaderboard 2026 in Depth"* — [link](https://www.codeant.ai/blogs/swe-bench-scores) — Analysis of the Verified vs. Pro gap (Claude Opus 4.5 drops 35 points from 80.9% to 45.9%).

4. **Cognition Labs** — *"SWE-bench technical report"* — [link](https://cognition.ai/blog/swe-bench-technical-report) — Devin's original evaluation methodology on SWE-bench; found that ~70% file localization accuracy translated to only 13.86% task resolution.

5. **Morph LLM** — *"SWE-Bench Pro Leaderboard (2026): Why 46% Beats 81%"* — [link](https://www.morphllm.com/swe-bench-pro) — Argues Pro is a better measure because Verified suffers from training-data contamination and underrepresents multi-file complexity.

6. **EvalPlus / ICML 2024** — *"RepoQA: Evaluating Long Context Code Understanding"* — [link](https://evalplus.github.io/repoqa.html) — Introduces needle-function retrieval as a prerequisite test for multi-file reasoning.

---

## Key Findings

> "The current SWE-Bench-Verified benchmark understates the complexity of real-world software engineering by presenting a distribution of challenges that skews heavily toward single-file edits—only 14.2% of issues requiring multi-file changes compared to 50.27% in real-world scenarios." — Jatin Ganhotra, 2025

> "My reaction is that there is an evaluation crisis. I don't really know what metrics to look at right now… SWE-Bench Verified (real, practical, verified problems) I really like and is great but itself too narrow." — Andrej Karpathy, March 2024

> "Our findings, which show top-tier models like Opus 4.1 and GPT-5 achieving a 23% success rate on SWE-Bench Pro compared to over 70% on benchmarks like SWE-Bench Verified, highlight a critical gap between current agent capabilities and the demands of real-world development." — SWE-Bench Pro paper, Scale AI

1. **Single-File Saturation, Multi-File Frontier:** Top systems collectively resolve ~90% of single-file SWE-Bench Verified issues, but only ~54% of multi-file issues. No individual system exceeds 30% on multi-file tasks.
2. **Real-World Distribution Mismatch:** Enterprise codebases have ~50% multi-file issues; SWE-Bench Verified has only 14.2%, while SWE-Bench Pro averages ~4 files per fix.
3. **Performance Collapse on Multi-File Tasks:** Claude Opus 4.5 scores 80.9% on Verified but 45.9% on Pro; GPT-5 drops from ~55% to 23.3%.
4. **Multilingual Multi-File Is Even Harder:** Multi-SWE-bench shows performance generally follows Python > Java > Go/Rust > C/C++ > TypeScript/JavaScript, with cross-file issues causing significant drops across all languages.
5. **Contamination Inflates Verified Scores:** OpenAI's audit found frontier models could reproduce verbatim gold patches on some Verified tasks, motivating the move to contamination-resistant Pro benchmarks.
6. **Localization ≠ Resolution:** Devin correctly identified files to change in ~70% of cases but only resolved 13.86% of tasks, showing that multi-file reasoning requires more than just file retrieval.
7. **Repository-Level QA Is Under-Benchmarked:** SWE-QA found that existing benchmarks focus on isolated snippets; cross-file dependency and intention-understanding questions remain a major gap.

---

### Metrics & Scoring Methodologies

| Metric | Definition | Used In |
|--------|-----------|---------|
| **%Resolved / Pass@1** | Percentage of issues where the agent's patch passes all tests without regressions. | SWE-bench family, Multi-SWE-bench |
| **Single-file %Resolved** | %Resolved restricted to issues requiring changes in only one file. | Leaderboard breakdowns |
| **Multi-file %Resolved** | %Resolved restricted to issues requiring changes across >1 file. | Ganhotra analysis, Pro |
| **Success Location** | File-level fault localization accuracy (did the agent touch the right files?). | Multi-SWE-bench |
| **Average Cost ($)** | Estimated API cost per issue attempt. | Multi-SWE-bench, Pro |
| **Line Coverage (LC)** | % of lines covered by generated unit tests. | MultiFileTest |
| **Branch Coverage (BC)** | % of branches covered by generated unit tests. | MultiFileTest |
| **Needle Retrieval Score** | Accuracy of retrieving a target function from a large codebase given its description. | RepoQA |
| **Human Evaluation (Correctness, Completeness, Relevance, Clarity)** | Expert-annotated Likert-scale scores for repository-level QA answers. | SWE-QA |
