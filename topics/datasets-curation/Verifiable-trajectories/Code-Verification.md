# Code-Verification

## 📌 Overview

| Field | Value |
|-------|-------|
| **Also Known As** | Execution-based Evaluation, Test-time Verification |
| **Key Metric** | pass@k (at least 1 of k samples passes unit tests) |
| **Core Advantage** | Unambiguous binary signals for RL training |
| **Primary Orgs** | BigCode Project, OpenAI, Anthropic |

> "Execution feedback is essential for humans to judge code quality reliably."

— [BigCodeArena](https://huggingface.co/blog/bigcode/arena)

---

## What Is Code-Verification?

Code-verification evaluates code correctness through **execution against unit tests** — providing unambiguous, binary pass/fail signals. Unlike natural language reasoning, code execution is ground-truth verifiable, making it ideal for RL training with verifiable rewards.

### Core Approaches

1. **Execution-based evaluation** — Run generated code against unit tests (pass@k metrics)
2. **Process Reward Models (PRM)** — Line-level dense rewards during code generation
3. **Outcome Reward Models (ORM)** — Binary pass/fail rewards after full code execution
4. **Self-verification** — Model checking its own output before final submission

---

## Key Frameworks & Repositories

### BigCode Evaluation Harness

| Field | Value |
|-------|-------|
| URL | [github.com/bigcode-project/bigcode-evaluation-harness](https://github.com/bigcode-project/bigcode-evaluation-harness) |
| Org | BigCode Project (Hugging Face + ServiceNow Research) |
| Stars | 1k+ |

> "This is a framework for the evaluation of code generation models. This work is inspired from EleutherAI/lm-evaluation-harness for evaluating language models."

**Features:**
- Distributed generation via Hugging Face accelerate
- Multi-GPU and Multi-CPU support
- Evaluates solutions with code execution or BLEU-based metrics
- Containerized execution for safety
- Supports: HumanEval, MBPP, APPS, MultiPL-E

### BigCodeArena

| Field | Value |
|-------|-------|
| URL | [huggingface.co/blog/bigcode/arena](https://huggingface.co/blog/bigcode/arena) |
| arXiv | [2510.08697v1](https://arxiv.org/abs/2510.08697v1) |

> "First human-in-the-loop platform for evaluating code generation models through live code execution"
> "14,000+ conversations, 500+ unique users, 4,700+ high-quality preference votes"

### OctoPack / COMMITPACK

| Field | Value |
|-------|-------|
| URL | [OpenReview](https://openreview.net/pdf?id=CjrPqvvUXL) |
| Venue | NeurIPS 2023 Workshop |
| Dataset | 4TB, 350 programming languages |

> "We focus on more permissively licensed data and avoid using a closed-source model to generate synthetic data."

**Contributions:**
- **COMMITPACK:** 4TB dataset of permissively licensed Git commits across 350 languages
- **HUMANEVALPACK:** Benchmark across 3 scenarios and 6 languages
- **OCTOCODER:** Best-performing permissive Code LLM

### SantaCoder

| Field | Value |
|-------|-------|
| URL | [huggingface.co/bigcode/santacoder](https://huggingface.co/bigcode/santacoder) |

> "The SantaCoder models are a series of 1.1B parameter models trained on the Python, Java, and JavaScript subset of The Stack (v1.1)."

- 1.1B parameters, Multi Query Attention, 2048 context, Fill-in-the-Middle objective

### RLTF (Reinforcement Learning from Unit Test Feedback)

| Field | Value |
|-------|-------|
| URL | [OpenReview](https://openreview.net/forum?id=hjYmsV6nXZ) |
| Venue | TMLR 2024 |
| Org | Zyq-scut |

> "We proposed RLTF, i.e., Reinforcement Learning from Unit Test Feedback, a novel online RL framework with unit test feedback of multi-granularity for refining code LLMs."

**Key Results:** SOTA on APPS and MBPP benchmarks

### ReVeal — ICLR 2025

| Field | Value |
|-------|-------|
| URL | [OpenReview](https://openreview.net/forum?id=q56ZI1Co43) |

> "ReVeal structures long-horizon reasoning as iterative generation-verification loops, where the model generates code, constructs test cases, executes them via sandbox, and verifies its outputs."

### StepCoder — ACL 2024

| Field | Value |
|-------|-------|
| URL | [ACL Anthology](https://aclanthology.org/2024.acl-long.251.pdf) |

> "APPS+ benchmark: 59.7% pass@1 vs 55.1% for RLTF"

Uses CCCS (Curriculum Complexity-based Selection) and FGO (Fine-grained Optimization).

### SelfCodeAlign — NeurIPS 2024

| Field | Value |
|-------|-------|
| URL | [arXiv:2410.24198](https://arxiv.org/abs/2410.24198) |

> "SelfCodeAlign is the first fully transparent and permissive pipeline for self-aligning code LLMs without extensive human annotations or distillation."

---

## Benchmark Datasets

### HumanEval (OpenAI)

| Field | Value |
|-------|-------|
| Problems | 164 hand-crafted programming problems |
| Metric | pass@k |
| Task | Function-level code generation |

> "Uses pass@k metric for evaluation — evaluates probability that at least one of top k generated samples passes unit tests."

### APPS (Automated Programming Progress Standard)

| Field | Value |
|-------|-------|
| URL | [github.com/bigcode-project/bigcode-evaluation-harness](https://github.com/bigcode-project/bigcode-evaluation-harness) |
| Problems | 10,000 Python problems (5,000 train / 5,000 eval) |
| Difficulty | Introductory, Interview, Competition |

### BigCodeBench — ICLR 2025

| Field | Value |
|-------|-------|
| URL | [github.com/bigcode-project/bigcodebench](https://github.com/bigcode-project/bigcodebench) |
| Tasks | 1,140 software-engineering-oriented programming tasks |

### HUMANEVALPACK

| Languages | Python, JavaScript, Java, Go, C++, Rust |
|-----------|------------------------------------------|
| Tasks | HUMANEVALSYNTHESIZE, HUMANEVALFIX, HUMANEVALEXPLAIN |

### LiveCodeBench

| Field | Value |
|-------|-------|
| Tasks | 612 public coding tasks (May 2023 – Sep 2024) |
| Scope | Comprehensive evaluation over time |

---

## Code Execution Sandboxes

| Sandbox | Cold Start | Security | Notes |
|---------|-----------|----------|-------|
| **Docker** | 500ms–2s | Moderate | Namespace/cgroups, used in bigcode-evaluation-harness |
| **gVisor** | Fast | High | User-space kernel |
| **Firecracker** | ~100ms | Highest | MicroVM, used by E2B |
| **E2B** | — | High | Cloud-hosted, SDK for custom interpreters |

> "Open-source infrastructure/SDK for running AI-generated code in secure isolated cloud sandboxes"

— [E2B](https://www.e2b.dev/)

---

## PRM vs ORM for Code

### ORM (Outcome Reward Model)

| Characteristic | Value |
|---------------|-------|
| Signal | Binary pass/fail after complete execution |
| Granularity | Sparse — only feedback when entire program passes |
| Used in | RLTF, standard RL approaches |

### PRM (Process Reward Model)

| Characteristic | Value |
|---------------|-------|
| Signal | Dense, line-level feedback during generation |
| Granularity | Step-by-step intermediate rewards |
| Benefit | Better credit assignment in RL |

**PRM Performance on LiveCodeBench:**

| Model | DenseReward | ValueInit | LiveCodeBench | InHouseBench |
|-------|:-----------:|:---------:|:-------------:|:------------:|
| Ours-SFT | No | No | 23.5 | 20.0 |
| Ours-RL | No | No | 28.2 | 31.8 |
| Ours-RL | No | Yes | 28.2 | 31.4 |
| Ours-RL | Yes | No | 28.9 | 32.3 |
| **Ours-RL** | **Yes** | **Yes** | **29.8** | **35.8** |

---

## Integration with RL Training

### RLTF Flow

1. LLM generates code samples
2. Code executed against unit tests
3. Multi-granularity feedback signals extracted
4. Online RL training with fine-grained rewards

### CodePRM — ACL 2025 Findings

> "Each partial reasoning process is linked to its corresponding code and execution feedback... CodePRM uses execution feedback to enhance process reward model training."

---

## Summary Data Points

| Paper | Venue | Year | Key Contribution |
|-------|-------|------|------------------|
| OctoPack | NeurIPS Workshop | 2023 | COMMITPACK dataset, OCTOCODER |
| RLTF | TMLR | 2023 | Multi-granularity unit test feedback |
| Process PRM | arXiv | 2024 | Line-level dense rewards for code |
| StepCoder | ACL | 2024 | CCCS + FGO for APPS+ |
| SelfCodeAlign | NeurIPS | 2024 | Self-alignment pipeline |
| BigCodeBench | ICLR | 2025 | 1,140 software engineering tasks |
| BigCodeArena | arXiv | 2025 | Human preference via execution |
| CodePRM | ACL Findings | 2025 | Execution feedback-enhanced PRM |
| ReVeal | ICLR | 2025 | Iterative generation-verification |

---

## Related Topics

- [RLVR.md](./RLVR.md) — Reinforcement Learning with Verifiable Rewards
- [Math-gold-standards.md](./Math-gold-standards.md) — Math datasets for verifiable rewards
- [Verifiable-trajectories](./README.md) — Parent folder overview

---

## References

- [BigCode Evaluation Harness](https://github.com/bigcode-project/bigcode-evaluation-harness)
- [BigCode Project Organization](https://github.com/bigcode-project)
- [BigCodeArena](https://huggingface.co/blog/bigcode/arena)
- [OctoPack — NeurIPS 2023](https://openreview.net/pdf?id=CjrPqvvUXL)
- [RLTF — TMLR](https://openreview.net/forum?id=hjYmsV6nXZ)
- [Process PRM — arXiv:2410.17621](https://arxiv.org/html/2410.17621v1)
- [StepCoder — ACL 2024](https://aclanthology.org/2024.acl-long.251.pdf)
- [SelfCodeAlign — NeurIPS 2024](https://arxiv.org/abs/2410.24198)
- [ReVeal — ICLR 2025](https://openreview.net/forum?id=q56ZI1Co43)
- [BigCodeBench — ICLR 2025](https://github.com/bigcode-project/bigcodebench)
- [CodePRM — ACL 2025](https://aclanthology.org/2025.findings-acl.428.pdf)
- [E2B Sandbox](https://www.e2b.dev/)
