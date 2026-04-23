# CoT-data

## 📌 Overview

**CoT-data** (Chain-of-Thought data) refers to datasets containing intermediate reasoning steps that guide models through multi-step problem solving. These datasets are critical for training reasoning models, and require careful collection, generation, and filtering strategies.

> *"It is better to have less high-quality synthetic data than more lower-quality data."*
> — [CoT-Self-Instruct arXiv:2507.23751](https://arxiv.org/abs/2507.23751)

---

## Key Methods

### CoT-Self-Instruct (arXiv:2507.23751)

Two-stage pipeline: (1) Synthetic instruction creation via CoT, (2) Synthetic instruction curation with filtering.

**Filtering methods:**

| Task Type | Method | Description |
|-----------|--------|-------------|
| **Verifiable reasoning** | Answer-Consistency | Compares majority-voted LLM answer against CoT-generated target |
| **Non-verifiable** | RIP (Rejecting Instruction Preferences) | Uses reward model scores |

> *"CoT-Self-Instruct with Answer-Consistency filtered data achieved **58.7%** on MATH500 vs 47.5% for OpenMathReasoning (10k)."*
> — [CoT-Self-Instruct Paper](https://arxiv.org/abs/2507.23751)

---

### SelF-Reasoner (LREC-COLING 2024)

Filters misleading CoT reasoning with selective verification.

**Three components:**

| Component | Role |
|-----------|------|
| **Reasoner** | Generates chain |
| **Answerer** | Predicts directly or extracts |
| **CoT Filter** | Validates reasoning |

> *"~30% of CoTs filtered out as invalid; small models struggle with self-generated CoT."*
> — [SelF-Reasoner Paper](https://arxiv.org/abs/2404.11406)

> *"SelF-Reasoner (Large) comparable to human accuracy on ScienceQA (87.24% vs 88.4%)."*
> — [SelF-Reasoner Paper](https://arxiv.org/abs/2404.11406)

> *"Small-scale language models are unlikely to achieve better reasoning performance with self-generated CoT alone."*
> — [SelF-Reasoner Paper](https://arxiv.org/abs/2404.11406)

---

### SWiRL — Step-Wise Reinforcement Learning (arXiv:2504.04736)

Multi-step reasoning via synthetic data + RL.

**Key finding:**

> *"Process filtering outperforms outcome-only filtering. Process filter: retain trajectories where each step is 'reasonable' given prior context → **Best performance**."*
> — [SWiRL Paper](https://arxiv.org/abs/2504.04736)

> *"Outcome filter: keep only correct final answers → Often hurts performance."*
> — [SWiRL Paper](https://arxiv.org/abs/2504.04736)

> *"SWiRL, unlike SFT, can learn even from trajectories that end in incorrect final answers."*
> — [SWiRL Paper](https://arxiv.org/abs/2504.04736)

**Results:**

| Benchmark | Improvement |
|-----------|-------------|
| GSM8K | **+21.5%** |
| HotPotQA | **+12.3%** |

---

### Simula (arXiv:2603.29791)

Reasoning-driven synthetic data generation framework.

**Three-stage pipeline:**

| Stage | Description |
|-------|-------------|
| **Conceptual Mapping** | Build taxonomies |
| **Agentic Refinement** | Generate diverse/complex examples |
| **Dual-Critic Filtering** | Assesses correctness AND incorrectness (mitigates sycophancy bias) |

> *"Dual-critic filtering independently assesses correctness AND incorrectness to mitigate sycophancy bias."*
> — [Simula Paper](https://arxiv.org/abs/2603.29791)

---

### DC-CoT (ICLR 2026)

**First data-centric benchmark for CoT distillation.**

Investigates data manipulation in CoT distillation from method, model, and data perspectives — augmentation, selection, and mixing impact on student LLMs.

---

### Process Reward Models (PRMs) for Filtering

| Model | Description |
|-------|-------------|
| **ThinkPRM** | Verifies every step via long-CoT reasoning |
| **SPARK** | Stepwise Process-Aware Rewards using aggregated verifications |
| **Uncertainty-aware** | Uses CoT-based entropy to capture verifier uncertainty |

---

### NVIDIA Llama Nemotron Post-Training Dataset

**32M sample reasoning dataset.**

| Distribution | Size |
|-------------|------|
| Math | 22M |
| Coding | 10M |
| Science | 708k |
| Instruction Following | 56k |
| Chat | 39k |

> *"Sample structure includes `reasoning: "on/off"` flag for controllable reasoning modes."*
> — [NVIDIA Llama Nemotron](https://huggingface.co/datasets/NVIDIA/PKU-Alignment-data)

---

### Flow of Reasoning (FoR) (ICML 2025)

Uses GFlowNets on DAG-structured reasoning graphs to sample diverse solutions.

> *"Achieves high quality + diversity with minimal training examples (e.g., 15 examples)."*
> — [FoR Paper](https://arxiv.org/abs/2505.14809)

---

### OpenThoughts / OpenThinker (Bespoke Labs)

| Model | Base | Dataset |
|-------|-----|---------|
| **OpenThinker-7B** | Qwen2.5-7B-Instruct | 114k examples |
| **OpenThoughts3** | QwQ | 1.2M examples (scaled pipeline) |

---

## Key Patterns

| Aspect | Finding |
|--------|---------|
| **Collection** | Teacher models (GPT-4, Gemini, Claude) generate reasoning trajectories; smaller models learn from these |
| **Generation** | Multi-stage: seed tasks → CoT reasoning → synthetic examples; taxonomic sampling for diversity |
| **Filtering** | Process-level filtering (step validity) outperforms outcome-only; dual-critic better than single |
| **Quality > Quantity** | Filtered high-quality data outperforms large low-quality datasets consistently |
| **Scalability** | Agentic approaches (Simula) enable seedless, scalable synthetic data generation |

---

## Sources

| Type | Citation |
|------|----------|
| **Paper** | [arXiv:2507.23751 — CoT-Self-Instruct](https://arxiv.org/abs/2507.23751) |
| **Paper** | [arXiv:2404.11406 — SelF-Reasoner](https://arxiv.org/abs/2404.11406) |
| **Paper** | [arXiv:2504.04736 — SWiRL](https://arxiv.org/abs/2504.04736) |
| **Paper** | [arXiv:2603.29791 — Simula](https://arxiv.org/abs/2603.29791) |
| **Paper** | [arXiv:2603.29791 — DC-CoT](https://arxiv.org/abs/2507.02467) |
| **Paper** | [arXiv:2505.14809 — FoR (ICML 2025)](https://arxiv.org/abs/2505.14809) |
| **Dataset** | [NVIDIA Llama Nemotron](https://huggingface.co/datasets/NVIDIA/PKU-Alignment-data) |

---

## Related Topics

- [Verifiable-Reasoning.md](./Verifiable-Reasoning.md) — Process reward models, verifiable benchmarks
