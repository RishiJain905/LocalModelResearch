# AutoAdapt Framework

> **📌 Overview**
> AutoAdapt is an **end-to-end automated framework for LLM domain adaptation** developed by Microsoft Research, published April 2026. It automates the design and tuning of domain adaptation workflows — selecting among RAG, SFT, LoRA, DPO, and alignment steps — via an Adaptation Configuration Graph (ACG), Planning Agent, and AutoRefine loop. Achieves **25% average accuracy improvement** over SOTA AutoML baselines at ~$4 and ~30 min additional overhead.

---

## Definitions

### From the Paper (arXiv:2603.08181)

> *"To solve these challenges, we present AutoAdapt, a novel end-to-end automated framework for efficient and reliable LLM domain adaptation."*

**Source:** [AutoAdapt: An Automated Domain Adaptation Framework for LLMs](https://arxiv.org/abs/2603.08181) — CC BY 4.0

### From Microsoft Research Blog

> *"Given a task objective, available domain data, and practical requirements like accuracy, latency, hardware, and budget, AutoAdapt plans a valid adaptation pipeline, selecting among approaches like RAG and multiple fine-tuning methods, and tunes key hyperparameters using a budget-aware refinement loop."*

**Source:** [Microsoft Research Blog — AutoAdapt: Automated domain adaptation for large language models](https://www.microsoft.com/en-us/research/blog/autoadapt-automated-domain-adaptation-for-large-language-models/) (April 22, 2026)

### From GitHub README

> *"AutoAdapt leverages curated knowledge bases from literature and open-source resources to reduce expert intervention."*

**Source:** [microsoft/AutoAdapt](https://github.com/microsoft/AutoAdapt) — MIT License

---

## Key Sources

### Primary Paper

| Field | Value |
|-------|-------|
| **Title** | AutoAdapt: An Automated Domain Adaptation Framework for LLMs |
| **arXiv ID** | [2603.08181](https://arxiv.org/abs/2603.08181) |
| **DOI** | [10.48550/arXiv.2603.08181](https://doi.org/10.48550/arXiv.2603.08181) |
| **License** | CC BY 4.0 |
| **Authors** | Sidharth Sinha, Anson Bastos, Xuchao Zhang, Akshay Nambi, Rujia Wang, Chetan Bansal |

### Microsoft Research Blog

| Field | Value |
|-------|-------|
| **Title** | AutoAdapt: Automated domain adaptation for large language models |
| **Published** | April 22, 2026 |
| **URL** | [Microsoft Research Blog](https://www.microsoft.com/en-us/research/blog/autoadapt-automated-domain-adaptation-for-large-language-models/) |

> *"An operations team responding to an outage can't afford a model that drifts from domain requirements or a tuning process that takes weeks with no guarantee of a reproducible result."*

> *"The AutoAdapt workflow streamlines LLM domain adaptation from planning to refinement. This process transforms weeks of manual tuning into a disciplined, reproducible, and auditable pipeline, significantly reducing the time and cost associated with achieving domain-ready models."*

### StartupHub.ai Coverage

> *"The broader implication of AutoAdapt is its potential to elevate domain adaptation from an ad hoc art to a rigorous engineering discipline."*

**Source:** [StartupHub.ai — AutoAdapt: Microsoft's LLM Adaptation Fix](https://www.startuphub.ai/ai-news/ai-research/2026/autoadapt-microsoft-s-llm-adaptation-fix)

---

## Core Technical Components

### Adaptation Configuration Graph (ACG)

- Structured representation of the system's configuration space
- Enables efficient search while guaranteeing valid pipelines
- Maps the full scope of the adaptation process

### Planning Agent

- Builds on the ACG
- Proposes strategies, evaluates them against user requirements, and iterates
- Roots each decision in best practices and explicit constraints
- Produces an executable workflow with parameter ranges

### AutoRefine (Budget-Aware Refinement Loop)

- LLM-based surrogate that replaces costly black-box hyperparameter search
- Uses LLM-guided Gaussian Process sampling
- Strategically selects which experiments to run next
- Works under limited feedback and constrained budgets

### Multi-Agent System

| Agent | Purpose |
|-------|---------|
| `best_practice_agent.py` | Retrieves best practices from knowledge base |
| `data_agent.py` | Analyzes dataset characteristics |
| `knowledge_retriever_agent.py` | General knowledge retrieval |
| `research_agent.py` | Web search and research |
| `user_preference_agent.py` | Parses user preferences |
| `aggregator_agent.py` | Aggregates agent outputs |

### Supported Adaptation Strategies

- **RAG** — Retrieval-Augmented Generation
- **SFT** — Supervised Fine-Tuning
- **LoRA** — Low-Rank Adaptation
- **DPO** — Direct Preference Optimization
- **Alignment Steps**

---

## Repositories & Tools

### Microsoft Official Repository

| Field | Value |
|-------|-------|
| **URL** | [github.com/microsoft/AutoAdapt](https://github.com/microsoft/AutoAdapt) |
| **License** | MIT |
| **Status** | Research-only (not production-ready) |

**Folder Structure:**
```
AutoAdapt/
├── app/
│   ├── sample/           # CLI sample application
│   │   ├── conf/         # config.yaml, user_preference.md
│   │   ├── run.py        # Main CLI entry point
│   │   └── run_plan.py   # Plan-only execution
│   └── react/            # React UI (frontend/backend)
├── src/
│   ├── autoadapt/
│   │   ├── agents/       # Multi-agent system (6 agents)
│   │   ├── hpo/          # AutoRefine (HPO)
│   │   │   ├── llm_surrogate_sampler.py
│   │   │   └── gaussian_process.py
│   │   ├── template/     # Standalone templates (SFT, DPO, RAG)
│   │   └── config/        # Knowledge bases
│   └── deepcore/          # Data selection methods
```

**Two Usage Methods:**
1. **CLI Usage** — Configure via `app/sample/conf/config.yaml` and run `run.py`
2. **UI Usage** — React-based frontend via `app/react` with `setup.sh`

### Third-Party Replication Repository

| Field | Value |
|-------|-------|
| **URL** | [github.com/serval-uni-lu/AutoAdapt](https://github.com/serval-uni-lu/AutoAdapt) |
| **Purpose** | Full source code and scripts to replicate experiments from the AutoAdapt paper |

---

## Benchmarks & Performance

### Performance Results

> *"Across 10 tasks, AutoAdapt achieves a **25% average accuracy improvement** over state-of-the-art Automated Machine Learning baselines with minimal optimization overhead."*

**Metrics Evaluated:**
- Success Rate (SR)
- Normalized Performance Score (NPS)
- Cumulative Score (CS)

**Domains Tested:** Reasoning, Question Answering, Coding, Classification, Cloud-incident diagnosis

### Overhead

| Metric | Value |
|--------|-------|
| Additional Time | ~30 minutes |
| Additional Cost | ~$4 |

---

## Use Cases

| Domain | Quote |
|--------|-------|
| **Cloud Incident Diagnosis** | *"An operations team responding to an outage can't afford a model that drifts from domain requirements."* |
| **Clinical Note Drafting** | *"Critical for high-stakes domains: Healthcare documentation"* |
| **Legal Workflows** | *"In high-stakes settings like law, medicine, and cloud incident response, performance and reliability can quickly break down."* |
| **Regulatory Language Summarization** | *"Critical for high-stakes domains: Regulatory language summarization"* |
| **Support Incident Triage** | *"Critical for high-stakes domains: Incident response"* |

---

## Summary

| Aspect | Details |
|--------|---------|
| **Framework Name** | AutoAdapt |
| **Developed By** | Microsoft Research (M365 Research, AIOps) |
| **Paper** | [arXiv:2603.08181](https://arxiv.org/abs/2603.08181) |
| **Open Source** | [github.com/microsoft/AutoAdapt](https://github.com/microsoft/AutoAdapt) |
| **License** | MIT |
| **Primary Function** | Automated LLM domain adaptation |
| **Key Innovation** | ACG + Planning Agent + AutoRefine |
| **Adaptation Strategies** | RAG, SFT, LoRA, DPO, Alignment Steps |
| **Avg Accuracy Improvement** | 25% over SOTA AutoML baselines |
| **Time Overhead** | ~30 minutes |
| **Cost Overhead** | ~$4 |
| **Status** | Research-only (not production-ready) |
| **Publication Date** | April 2026 |

---

## Further Reading

- [New methods boost reasoning in small and large language models](https://www.microsoft.com/en-us/research/) — Microsoft Research (June 17, 2025)
- [BenchmarkQED: Automated benchmarking of RAG systems](https://www.microsoft.com/en-us/research/) — Microsoft Research (June 5, 2025)
- [LazyGraphRAG: Setting a new standard for quality and cost](https://www.microsoft.com/en-us/research/) — Microsoft Research (November 25, 2024)
- [GraphRAG auto-tuning provides rapid adaptation to new domains](https://www.microsoft.com/en-us/research/) — Microsoft Research (September 9, 2024)
- [Large-language models for automatic cloud incident management](https://www.microsoft.com/en-us/research/) — Microsoft Research

---

## Related Topics

- [Budget-Aware Refinement](./Budget-Aware-Refinement.md) — Budget-constrained optimization for models
- [LoRA / QLoRA](../Parameter-Efficient-Fine-Tuning/LoRA-QLoRA.md) — Parameter-efficient fine-tuning methods
- [RAG](../RAG/RAG-Systems.md) — Retrieval-Augmented Generation
- [Automated ML](../AutoML/AutoML-Systems.md) — Automated machine learning pipelines
