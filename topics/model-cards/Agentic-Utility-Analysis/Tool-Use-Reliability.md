# Tool-Use Reliability (TUR)

> "Tool-use reliability refers to the consistent and correct selection, invocation, and parameter grounding of external tools by LLMs and AI agents, directly determining whether agentic systems can be trusted in production." — Emergent Mind / Xu et al., 2024

## Contents

1. [Key Papers](#key-papers)
2. [Open-Source Libraries & Tools](#open-source-libraries--tools)
3. [GitHub Repositories](#github-repositories)
4. [Blog Posts & Articles](#blog-posts--articles)
5. [Key Findings](#key-findings)

---

## Key Papers

### ToolLLM: Facilitating Large Language Models to Master 16000+ Real-world APIs (2023)

> *"Despite the advancements of open-source large language models (LLMs), e.g., LLaMA, they remain significantly limited in tool-use capabilities, i.e., using external tools (APIs) to fulfill human instructions."* — Yujia Qin et al.
> — OpenReview / arXiv
> — [arXiv:2307.16789](https://arxiv.org/abs/2307.16789) | [OpenReview](https://openreview.net/forum?id=dHng2O0Jjr) | [GitHub](https://github.com/OpenBMB/ToolBench)

**Key Contributions:**
- Introduces **ToolBench**, a massive instruction-tuning dataset with **16,464 real-world RESTful APIs** spanning 49 categories, **126,486 instruction instances**, and **469,585 actual API calls**.
- Proposes **DFSDT** (Depth First Search-based Decision Tree) to overcome error propagation and limited exploration limitations of CoT/ReACT.
- Develops **ToolEval**, an automatic evaluator using **Pass Rate** (87.1% human agreement) and **Win Rate** (80.3% human agreement).

---

### API-Bank: A Comprehensive Benchmark for Tool-Augmented LLMs (2023)

> *"How effective are current LLMs in utilizing tools? How can we enhance LLMs' ability to utilize tools? What obstacles need to be overcome to leverage tools?"* — Minghao Li et al.
> — EMNLP 2023
> — [arXiv:2304.08244](https://arxiv.org/abs/2304.08244) | [ACL Anthology](https://aclanthology.org/2023.emnlp-main.187/)

**Key Contributions:**
- Builds a **runnable evaluation system** of 73 API tools with 314 annotated dialogues and 753 API calls.
- Constructs a **training corpus** of 1,888 tool-use dialogues from 2,138 APIs across 1,000 domains.
- Demonstrates that **GPT-4 excels in planning**, while **GPT-3.5** shows improved tool utilization over GPT-3. **Lynx** surpasses Alpaca by >26 pts.

---

### AgentBench: Evaluating LLMs as Agents (2023)

> *"While top commercial LLMs present a strong ability of acting as agents in complex environments, there is a significant disparity in performance between them and many OSS competitors that are no larger than 70B."* — Xiao Liu et al.
> — arXiv
> — [arXiv:2308.03688](https://arxiv.org/abs/2308.03688)

**Key Contributions:**
- Introduces a **multi-dimensional benchmark** across 8 interactive environments (OS, DB, KG, card game, puzzle, household, web shopping, web browsing).
- Evaluates **29 LLMs** and finds top commercial models lead, while **OSS models ≤70B lag considerably**.
- Identifies primary obstacles: **poor long-term reasoning, decision-making, and instruction following**.

---

### AgentBoard: An Analytical Evaluation Board of Multi-Turn LLM Agents (2024)

> *"Evaluating large language models (LLMs) as general-purpose agents is essential for understanding their capabilities and facilitating their integration into practical applications."* — Chang Ma et al.
> — arXiv
> — [arXiv:2401.13178](https://arxiv.org/abs/2401.13178) | [GitHub](https://github.com/hkust-nlp/AgentBoard)

**Key Contributions:**
- Provides **9 diverse tasks and 1,013 environments** covering embodied AI, game, web, and tool agents.
- Introduces a **fine-grained progress rate metric** beyond binary success/failure.
- Offers an **open-source evaluation framework** with interactive visualization.

---

### Reducing Tool Hallucination via Reliability Alignment (2024)

> *"Tool hallucinations are more severe than traditional text-based hallucinations because tools serve as the interface between LLMs and the physical world."* — Hongshen Xu et al.
> — arXiv
> — [arXiv:2412.04141](https://arxiv.org/abs/2412.04141)

**Key Contributions:**
- Categorizes tool hallucination into **four subtypes**: Tool Type, Tool Timing, Tool Format, and Tool Content.
- Proposes a **reliability alignment framework** with indecisive actions (`ChangeTools`, `TalkToUser`) and preference ranking.
- Introduces **reliability metrics** combining task success, hallucination penalty, and tool-call efficiency (Benefit-Cost Utility/Ratio).

---

### StableToolBench: Towards Stable Large-Scale Benchmarking on Tool Learning of LLMs (2024)

> *"This instability in API status led to considerable variability in model performance assessments and had a significant impact on the stability of the benchmark."* — Zhicheng Guo et al.
> — arXiv
> — [arXiv:2403.07714](https://arxiv.org/abs/2403.07714) | [GitHub](https://github.com/THUNLP-MT/StableToolBench)

**Key Contributions:**
- Replaces volatile real APIs with a **virtual API server** (caching + LLM simulation).
- Shows ** reproduced ToolBench results decay by up to 40%** over time due to API availability issues.
- Filters to **765 solvable tasks** via consensus of GPT-4, Gemini, and Claude, and proposes **SoPR/SoWR** metrics.

---

### Gorilla: Large Language Model Connected with Massive APIs (2023)

> *"We release Gorilla, a finetuned LLaMA-based model that surpasses the performance of GPT-4 on writing API calls."* — Shishir Patil et al.
> — NeurIPS 2023 Workshop
> — [arXiv:2305.15334](https://arxiv.org/abs/2305.15334) | [GitHub](https://github.com/ShishirPatil/gorilla)

**Key Contributions:**
- Fine-tunes LLaMA for **1,600+ API calls** while mitigating hallucination via a **document retriever**.
- Introduces **APIBench** dataset and demonstrates test-time adaptation to changing API documentation.

---

### ToolSandbox: A Stateful, Conversational, Interactive Evaluation Benchmark for LLM Tool Use Capabilities (2024)

> *"We show that open source and proprietary models have a significant performance gap, and complex tasks like State Dependency, Canonicalization and Insufficient Information... are challenging even the most capable SOTA LLMs."* — Apple ML Research
> — arXiv
> — [arXiv:2408.04682](https://arxiv.org/abs/2408.04682) | [GitHub](https://github.com/apple/ToolSandbox)

**Key Contributions:**
- Native Python testing environment with **stateful execution**, **implicit state dependencies**, and **built-in user simulator**.
- Evaluates against **Milestones** (desired events) and **Minefields** (unwanted events).
- Demonstrates significant gaps between open-source and proprietary models on state-dependent tasks.

---

### The Berkeley Function Calling Leaderboard (BFCL): From Tool Use to Agentic Evaluation of LLMs (2025)

> *"Function calling, also called tool use, refers to an LLM's ability to invoke external functions, APIs, or user-defined tools in response to user queries—an essential capability for agentic LLM applications."* — Shishir G. Patil et al.
> — ICML 2025
> — [PMLR](https://proceedings.mlr.press/v267/patil25a.html) | [GitHub](https://github.com/ShishirPatil/gorilla/tree/main/berkeley-function-call-leaderboard)

**Key Contributions:**
- Comprehensive executable benchmark for **single-turn, multi-turn, and agentic** function calling.
- Evolved through V1 (AST evaluation), V2 (enterprise/live APIs), V3 (stateful multi-turn), and **V4 (holistic agentic evaluation)**.
- Evaluates **accuracy, cost, and latency** across native FC and prompt-based models.

---

### MCP-Bench: Benchmarking Tool-Using LLM Agents with Complex Real-World Tasks via MCP Servers (2025)

> *"Unlike prior API-based benchmarks, each MCP server provides a set of complementary tools designed to work together, enabling the construction of authentic, multi-step tasks with rich input–output coupling."* — ICLR 2026
> — [arXiv:2508.20453](https://arxiv.org/abs/2508.20453)

**Key Contributions:**
- Evaluates LLMs on **28 live MCP servers** spanning 250 tools across finance, travel, scientific computing, and academic search.
- Tests **retrieval from fuzzy instructions**, **multi-hop planning**, and **cross-domain orchestration**.

---

## Open-Source Libraries & Tools

### Berkeley Function Calling Leaderboard (BFCL)

> [gorilla.cs.berkeley.edu/leaderboard.html](https://gorilla.cs.berkeley.edu/leaderboard.html) | GitHub: `ShishirPatil/gorilla`

The de facto standard benchmark for evaluating function-calling/tool-use in LLMs. Supports AST-based evaluation, executable real-world APIs, multi-turn stateful interactions, and agentic workflows. Includes cost and latency tracking.

---

### Phoenix (Arize AI)

> [phoenix.arize.com](https://phoenix.arize.com) | `pip install arize-phoenix`

Open-source tracing and evaluation library with a built-in **function calling evaluator** that checks for selection, parameter extraction, and formatting errors across major LLM frameworks.

---

### Gorilla / OpenFunctions V2

> [gorilla.cs.berkeley.edu](https://gorilla.cs.berkeley.edu) | HuggingFace: `gorilla-llm`

Drop-in open-source function-calling models with OpenAI-compatible endpoints, supporting parallel execution, multiple languages (Python/Java/JS/REST), and function relevance detection.

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [OpenBMB/ToolBench](https://github.com/OpenBMB/ToolBench) | ⭐ 3.3k+ | Large-scale instruction-tuning dataset and ToolEval evaluator for real-world API tool use (16k+ APIs). |
| [ShishirPatil/gorilla](https://github.com/ShishirPatil/gorilla) | ⭐ 13k+ | Gorilla LLM, BFCL, OpenFunctions, GoEx execution engine, and RAFT fine-tuning. |
| [hkust-nlp/AgentBoard](https://github.com/hkust-nlp/AgentBoard) | ⭐ 200+ | Analytical evaluation board for multi-turn LLM agents with progress tracking and visualizations. |
| [THUNLP-MT/StableToolBench](https://github.com/THUNLP-MT/StableToolBench) | N/A | Stable evolution of ToolBench with virtual API server and caching for reproducible evaluation. |
| [apple/ToolSandbox](https://github.com/apple/ToolSandbox) | N/A | Apple's stateful conversational benchmark with milestones, minefields, and user simulation. |

---

## Blog Posts & Articles

1. **Steel.dev** — *"ToolBench benchmark | Tool use agent evaluation"* — [link](https://leaderboard.steel.dev/registry/benchmarks/toolbench) — Overview of ToolBench metrics (Pass Rate, Win Rate) and ToolLLaMA results.

2. **Medium / Manish Negi** — *"ToolLLM — Empowering LLMs for Advanced Tool Use"* — [link](https://medium.com/@manishnegi101/toolllm-empowering-llms-for-advanced-tool-use-58ac65fbb5e9) — Describes DFSDT, neural API retrieval, and how ToolLLaMA matches ChatGPT-level performance.

3. **Arize AI / Phoenix** — *"LLM Function Calling: Evaluating Tool Calls In LLM Pipelines"* — [link](https://phoenix.arize.com/llm-function-calling-evaluating-tool-calls-in-llm-pipelines/) — Practical guide to evaluating selection, parameter, and formatting errors.

4. **Emergent Mind** — *"Tool-Use Hallucinations in LLM Agents"* — [link](https://www.emergentmind.com/topics/tool-use-hallucinations) — Taxonomy of tool hallucinations and evaluation patterns.

5. **Medium / Kaushal Sinh** — *"11 eval patterns that expose tool hallucinations in agents"* — [link](https://medium.com/@kaushalsinh73/11-eval-patterns-that-expose-tool-hallucinations-in-agents-39f045f5a1b7) — Patterns for detecting agents that describe non-existent tool outputs.

---

## Key Findings

> "Tool hallucinations are more severe than traditional text-based hallucinations because tools serve as the interface between LLMs and the physical world. Consequences include physical harm or system failures from incorrect tool use, misleading outputs that are harder to detect, and wasted operational resources from redundant or faulty tool calls." — Xu et al., *Reducing Tool Hallucination via Reliability Alignment*

1. **Reliability is multi-faceted.** Effective TUR requires correct **tool selection**, valid **parameter formatting**, accurate **content grounding**, and appropriate **timing/sequencing** of calls.

2. **Commercial models lead, but gaps remain.** GPT-4 and Claude demonstrate the strongest agentic tool-use capabilities, yet even SOTA models struggle with **state dependency**, **canonicalization**, and **insufficient information** scenarios (ToolSandbox findings).

3. **Open-source models ≤70B lag significantly.** AgentBench found a "significant disparity" between top commercial LLMs and open-source competitors under 70B parameters across code, game, and web tasks.

4. **Benchmark stability is a critical issue.** StableToolBench showed that **real API availability decay** can cause reported pass rates to drop by **up to 40%** over months, necessitating virtual API servers and caching.

5. **Hallucination taxonomies enable targeted fixes.** Categorizing errors into Tool Type, Tool Timing, Tool Format, and Tool Content hallucinations allows fine-grained alignment (Relign framework) and improves both success rate and efficiency.

6. **AST-based and executable evaluation are essential.** BFCL introduced executable verification for function calling, moving beyond string-matching to ensure parameters are semantically and syntactically correct.

---

### Metrics & Scoring Methodologies

| Metric | Definition | Used In |
|--------|-----------|---------|
| **Pass Rate** | % of tasks where all tool calls succeed and fulfill the instruction. | ToolBench, ToolLLM |
| **Win Rate** | Head-to-head comparison win rate of one model vs baseline. | ToolEval |
| **Benefit-Cost Utility (BCU)** | Task success minus hallucination penalty, normalized by tool-call cost. | Reliability Alignment |
| **Benefit-Cost Ratio (BCR)** | Ratio of successful outcomes to total tool calls made. | Reliability Alignment |
| **SoPR (Successful Procedure Rate)** | % of procedures (multi-step tasks) completed successfully. | StableToolBench |
| **SoWR (Successful Workflow Rate)** | % of workflows (higher-level tasks) completed successfully. | StableToolBench |
| **Milestone Hit Rate** | % of desired milestone events achieved during evaluation. | ToolSandbox |
| **Minefield Avoidance Rate** | % of unwanted minefield events successfully avoided. | ToolSandbox |
| **AST Match** | Whether predicted function call matches ground truth via abstract syntax tree. | BFCL V1 |
| **Executable Accuracy** | Whether predicted call executes successfully against real API. | BFCL V2+ |
| **Cost ($)** | Average API call cost per evaluation instance. | BFCL V4 |
| **Latency (s)** | Average response time for function calling. | BFCL V4 |
| **Selection Accuracy** | Did the model pick the correct tool/function? | Phoenix, general |
| **Parameter Extraction F1** | F1 score for extracted parameter values vs ground truth. | Phoenix, general |
| **Formatting Error Rate** | % of calls with invalid JSON schema or parameter structure. | Phoenix, general |
