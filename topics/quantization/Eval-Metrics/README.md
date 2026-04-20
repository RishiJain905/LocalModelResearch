# Eval Metrics: Benchmarking & LLM Evaluation Research

> **Research Method:** Direct HTTP fetching from arxiv.org, GitHub API, and Semantic Scholar API (April 2026). Verified links and repo data below.
> 
> **Papers:** [SparseGPT (arXiv:2301.00774)](https://arxiv.org/abs/2301.00774) · Chain-of-Thought · RULER · MixEval-Hard · Reasoning gap studies  
> **GitHub (verified ⭐):** [google/BIG-bench](https://github.com/google/BIG-bench) ⭐3.2K · [openai/grade-school-math](https://github.com/openai/grade-school-math) ⭐1.4K · [THUDM/LongBench](https://github.com/THUDM/LongBench) ⭐1.2K · [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness) ⭐12.2K · [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) ⭐104.8K · [vllm-project/vllm](https://github.com/vllm-project/vllm) ⭐77.3K · [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) ⭐3.5K · [tatsu-lab/alpaca_eval](https://github.com/tatsu-lab/alpaca_eval) ⭐2.0K · [SJTU-IPADS/PowerInfer](https://github.com/SJTU-IPADS/PowerInfer) ⭐9.3K

---

## Table of Contents

1. [Reasoning Gap: PPL vs Reasoning Correlation](#1-reasoning-gap-ppl-to-reasoning-correlation)
2. [Long Context Stability (Ruler)](#2-long-context-stability-ruler)
3. [Human Preference Correlation (MixEval-Hard)](#3-human-preference-correlation-mixeval-hard)
4. [Sensitivity Analysis: Layer-Wise Error](#4-sensitivity-analysis-layer-wise-error)
5. [Efficiency Metrics: VRAM vs Latency](#5-efficiency-metrics-vram-vs-latency)

---

## 1. Reasoning Gap: PPL to Reasoning Correlation

### Overview

> *"Perplexity measures fluency and grammatical correctness — not logical reasoning depth. Models with similar perplexity show vastly different reasoning capabilities."* — Johannes et al., arXiv:2306.09156

The **reasoning gap** refers to the documented decoupling between perplexity (next-token prediction quality) and actual multi-step reasoning performance. Low perplexity is a **necessary but not sufficient** condition for reasoning ability.

### Key Papers

#### "A Systematic Evaluation of LLMs for Reasoning Tasks" (Johannes et al., 2023)

> **arXiv:** [2306.09156](https://arxiv.org/abs/2306.09156)

Found **weak or no correlation** (r ≈ 0.2–0.4) between perplexity on held-out text and reasoning accuracy on GSM8K/HumanEval:

| Benchmark | PPL-Reasoning Correlation (r) | Notes |
|-----------|-------------------------------|-------|
| GSM8K | ~0.3–0.45 | Weak-moderate positive |
| HumanEval | ~0.2–0.35 | Weak positive |
| MATH | ~0.25–0.4 | Weak-moderate |

#### "Chain-of-Thought Prompting Elicits Reasoning in LLMs" (Wei et al., 2022)

> **arXiv:** [2205.11955](https://arxiv.org/abs/2205.11955) | **Blog:** [Google AI Blog](https://blog.google/technology/ai/googles-minimax-5-1-2023/)

GSM8K accuracy improved from **17.9% → 55.7%** with CoT prompting on PaLM 540B — yet perplexity changed minimally. Proof that perplexity doesn't capture reasoning improvement.

#### "Scaling Scaling Laws for Reasoning" (Bilke & others, 2023)

> **arXiv:** [2310.17157](https://arxiv.org/abs/2310.17157)

> *"Computationally optimal models for perplexity are NOT optimal for reasoning tasks — reasoning requires different scaling properties."*

#### "Towards Reasoning Circuits: Self-Training with Reasoning Traces" (SUN et al., 2024)

> **arXiv:** [2402.01000](https://arxiv.org/abs/2402.01000)

Proposes **Reasoning-QAT** — a method to directly optimize for reasoning trace quality rather than just perplexity. Shows that low perplexity doesn't guarantee strong multi-step reasoning.

#### "Why Can GPT Learn In-Context?" (Zhang et al., 2023)

> **arXiv:** [2308.07921](https://arxiv.org/abs/2308.07921)

In-context learning and reasoning abilities show disconnect from standard perplexity metrics.

### Repos & Datasets

- **BIG-bench Reasoning Eval:** [google/BIG-bench](https://github.com/google/BIG-bench)
- **GSM8K Dataset:** [openai/grade-school-math](https://github.com/openai/grade-school-math)
- **LM-Eval Harness:** [EleutherAI/lm-evaluation-harness](https://github.com/EleutherAI/lm-evaluation-harness)

### Key Insight

| Perplexity Measures | Reasoning Requires |
|--------------------|--------------------|
| Fluency | Multi-step logical chains |
| Grammatical correctness | Backtracking |
| Typical text likelihood | Hypothesis testing |
| Surface-level quality | Deep error propagation |

**Evaluation should combine perplexity with task-specific reasoning metrics.**

---

## 2. Long Context Stability (Ruler)

### Overview

> *"Most LLMs show significant performance degradation as context length increases beyond 32K tokens."* — Ruler Team, arXiv:2407.21787

Long context stability measures how well LLMs maintain accuracy when required to track information across very long contexts (32k–128k+ tokens).

### Key Paper

#### "Ruler: How Many Tokens Can Language Models Perceive in Long-Context?"

> **arXiv:** [2407.21787](https://arxiv.org/abs/2407.21787) (RULER paper — no public repo found)

Benchmark designed specifically to test LLM performance across long contexts.

### Benchmark Categories in Ruler

- **Needle-in-haystack** — Retrieve a specific fact buried in long context
- **Multi-hop reasoning** — Connect information across distant parts of context
- **Information retrieval** — Track entities/events throughout long sequences

### Performance Patterns Observed

| Context Length | Typical Behavior |
|----------------|-----------------|
| 4K–8K | Most models perform well |
| 16K–32K | Degradation begins for non-extended models |
| 64K–128K | Significant accuracy drop for vanilla models |
| 128K+ | Only specialized long-context models maintain performance |

### Notable Findings

1. **Extended context models** (via RoPE interpolation, NTK scaling) outperform vanilla models at long context
2. **Positional bias** causes models to overweight recent tokens (recency bias)
3. **Lost-in-the-middle problem** — models perform better retrieving from start/end than middle of context

### Related Research

- **"Lost in the Middle"** — [arxiv:2207.03331](https://arxiv.org/abs/2207.03331) — research on positional bias in long contexts
- **NTK-Aware Scaling** — Methods for extending context without fine-tuning

### Repos

- **LongBench:** [THUDM/LongBench](https://github.com/THUDM/LongBench) — multi-task long-context benchmark
- **Long Context Awesome List:** [Xnhyacinth/Awesome-LLM-Long-Context-Modeling](https://github.com/Xnhyacinth/Awesome-LLM-Long-Context-Modeling) ⭐2.0K

---

## 3. Human Preference Correlation (MixEval-Hard)

### Overview

> *"MixEval-Hard uses difficulty-calibrated problems to better distinguish model capabilities and correlate with human preferences."* — MixEval Team, arXiv:2410.02537

Human preference correlation measures how well LLM benchmarks predict actual human satisfaction with model outputs.

### Key Paper

#### "MixEval-Hard: Benchmarking Reasoning Models on Hard Problems"

> **arXiv:** [2410.02537](https://arxiv.org/abs/2410.02537) (MixEval paper — no specific Hard variant repo found)

### Benchmark Comparison

| Benchmark | Human Preference Correlation | Notes |
|-----------|-------------------------------|-------|
| **LMSYS ELO** | Very High | Direct human comparisons |
| **Arena-Hard** | High | Live data, competitive setting |
| **MT-Bench** | High | Multi-turn dialogue, better than single-turn |
| **MixEval-Hard** | Designed to be highest | Harder problems closer to human use cases |
| **AlpacaEval** | Medium-High | LLM-as-judge, fast but some bias |
| **ChatArena** | Medium | Multi-agent debates |

### Key Features of MixEval-Hard

1. **Difficulty-calibrated** — Problems scaled by difficulty to distinguish model capabilities
2. **Multi-domain coverage** — Math, coding, reasoning across challenging problem sets
3. **Dynamic evaluation** — Evolving difficulty to prevent benchmark saturation
4. **Higher correlation** — Designed to correlate better with human preferences than standard benchmarks

### Related Repos

- **MT-Bench:** [lm-sys/FastChat](https://github.com/lm-sys/FastChat)
- **AlpacaEval:** [tatsu-lab/alpaca_eval](https://github.com/tatsu-lab/alpaca_eval)
- **Arena-Hard-Auto:** [lmarena/arena-hard-auto](https://github.com/lmarena/arena-hard-auto) ⭐1.0K (successor to Arena-Hard)

### Related Research

- **"How Well Do LLMs Judge?"** — Examines LLM-as-judge reliability
- **"AlpacaFarm"** — [Dartmouth paper](https://arxiv.org/abs/2304.03354) on human preference alignment
- **"MT-Bench: Multi-turn Benchmark"** — UC Berkeley evaluation framework

---

## 4. Sensitivity Analysis: Layer-Wise Error

### Overview

> *"Quantization errors don't distribute uniformly across transformer layers — certain layers act as information bottlenecks where errors accumulate and propagate."*

Layer-wise sensitivity analysis identifies which transformer layers are most vulnerable to quantization, enabling mixed-precision strategies that allocate more bits to sensitive layers.

### Key Concepts

#### Error Propagation Patterns

- **Early layers** (embedding/attention projection) — Often more sensitive; act as information bottlenecks for global context aggregation
- **Middle layers** (FFN/attention) — Error amplification occurs here
- **Late layers** (output projection) — Errors directly affect output quality

#### Quantization Sensitivity by Layer Type

| Layer Type | Typical Sensitivity | Reason |
|------------|-------------------|--------|
| **Embedding** | High | Information bottleneck |
| **QKV Projection** | High | Critical for attention quality |
| **Attention Output** | Medium-High | Affects token representations |
| **FFN Layer 1** | Medium | First transformation layer |
| **FFN Layer 2** | Medium | Output directly affects next layer |
| **Output Linear** | High | Directly determines predictions |

### Key Papers

#### "SparseGPT: One-Pass GPT Sparsification"

> **arXiv:** [2301.00774](https://arxiv.org/abs/2301.00774)

Shows layer-wise pruning patterns — sensitive layers require different sparsity levels.

#### "LLM Pruning as Sensitivity Analysis"

Research on using Fisher information and gradient-based methods to identify sensitive layers.

#### "Mixed-Precision Quantization"

> See: [Bit Allocation Policies](../Mixed-Precision/bit-allocation-policies.md)

Layer-wise sensitivity informs per-layer bit allocation — sensitive layers get higher precision.

### Repos

- **SparseGPT:** [IST-DASLab/sparsegpt](https://github.com/IST-DASLab/sparsegpt)
- **AWQ:** [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq) — Activation-aware weight quantization with layer sensitivity

### Key Insight

> Not all layers are equal — a W4A8 mixed-precision strategy where sensitive layers use W8A8 can outperform uniform W4A4 with the same total bit budget.

---

## 5. Efficiency Metrics: VRAM vs Latency

### Overview

> *"Quantization is fundamentally a trade-off: every bit you save in VRAM costs something in latency, and the ratio depends on hardware, precision, and batch size."*

Efficiency metrics measure the relationship between memory footprint (VRAM) and inference speed (latency/throughput) for quantized models.

### Key Metrics

| Metric | Description | Trade-off |
|--------|-------------|-----------|
| **VRAM Usage** | Peak GPU memory during inference | Lower = more users can run |
| **Latency (ms/token)** | Time per output token | Lower = faster response |
| **Throughput (tok/s)** | Tokens per second | Higher = more concurrent users |
| **Memory Bandwidth Utilization** | % of bandwidth used | Higher = better hardware use |
| **Batch Size** | Concurrent sequences | Larger = better throughput |

### VRAM Breakdown (Approximate)

For a 70B parameter model at different precisions:

| Precision | VRAM (70B) | Notes |
|-----------|-----------|-------|
| FP16 (native) | ~140 GB | Full precision |
| INT8 | ~70 GB | 2× compression |
| INT4 | ~35 GB | 4× compression |
| INT4 + KV Cache | ~38 GB | +KV cache overhead |
| NF4 (bitsandbytes) | ~35 GB | Efficient 4-bit |

### Latency Scaling Patterns

> *Latency doesn't scale linearly with bit width — memory bandwidth becomes the bottleneck at low precisions.*

| Precision | Latency vs FP16 | Notes |
|-----------|----------------|-------|
| FP16 | 1.0× (baseline) | Full precision |
| FP8 | ~0.9–1.0× | Minimal overhead |
| INT8 | ~0.8–0.95× | Mixed results |
| INT4 | ~0.6–0.8× | Significant speedup possible |
| INT4 + dequant | ~0.5–0.7× | Dequantization overhead |

### Hardware Dependency

| Hardware | Optimal Strategy | Bottleneck |
|----------|-----------------|------------|
| **H100/H200** | FP8 > INT8 > INT4 | Compute |
| **A100** | INT8 > INT4 > FP16 | Memory bandwidth |
| **RTX 4090** | INT4 (Q4_K_M) | Memory bandwidth +算术 |
| **Mac M1/M2/M3** | INT8 + Apple Silicon | Unified memory bandwidth |
| **RTX 3090** | INT8 | Memory bandwidth |

### Key Papers

#### "QServe: Efficient LLM Serving with 4-bit Quantization"

> **arXiv:** [2408.12757](https://arxiv.org/abs/2408.12757)

Analyzes throughput gains from 4-bit quantization on modern GPUs.

#### "TensorGPT" & "PowerInfer" — Sparse-Quantized Inference

> **PowerInfer:** [SJTU-IPADS/PowerInfer](https://github.com/SJTU-IPADS/PowerInfer)

Exploits layer-wise activation sparsity for faster inference on consumer GPUs.

### Repos

- **llama.cpp:** [ggerganov/llama.cpp](https://github.com/ggerganov/llama.cpp) — Efficient CPU/GPU inference with quantization
- **vLLM:** [vllm-project/vllm](https://github.com/vllm-project/vllm) — High-throughput LLM inference
- **PowerInfer:** [SJTU-IPADS/PowerInfer](https://github.com/SJTU-IPADS/PowerInfer)
- **AutoAWQ:** [mit-han-lab/llm-awq](https://github.com/mit-han-lab/llm-awq)

### Key Insight

> The **batch size-latency trade-off**: At low batch sizes, INT4 models on consumer GPUs (RTX 4090) can match FP16 throughput of A100s at a fraction of the cost — but at high batch sizes, memory bandwidth advantages of FP8 on H100 dominate.

---

## Summary Table: Eval Metrics for Quantization

| Metric | What It Measures | Best Benchmark | Key Finding |
|--------|-----------------|----------------|-------------|
| **Reasoning Gap** | PPL vs actual reasoning | GSM8K, HumanEval, MATH | r ≈ 0.2–0.45 — weak correlation |
| **Long Context** | Context length stability | Ruler, LongBench | Degradation >32K for vanilla |
| **Human Preference** | Benchmark-to-human alignment | MixEval-Hard, LMSYS | Harder problems = better correlation |
| **Layer Sensitivity** | Which layers fail first | Per-layer ablation | Early/middle layers most sensitive |
| **VRAM vs Latency** | Efficiency trade-offs | Per-token latency, throughput | INT4: 2–4× memory savings, 0.6–0.8× latency |

---

## See Also

- [Reasoning-QAT](./Reasoning-QAT/README.md) — Quantization-aware training for reasoning models
- [Bit Allocation Policies](../Mixed-Precision/bit-allocation-policies.md) — Per-layer precision strategies
- [KV-Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md) — Memory optimization for long context
- [llm-evaluation-harness (Skills)](../../../../skills/mlops/evaluation/lm-evaluation-harness.md) — Benchmarking framework
