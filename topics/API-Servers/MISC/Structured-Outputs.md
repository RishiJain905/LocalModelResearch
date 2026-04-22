# Structured Outputs & Schema-Constrained Generation

## 📌 Overview

Structured outputs ensure LLM responses conform to machine-readable schemas (JSON, XML, regex patterns) rather than free-form text. The field has evolved through four generations: prompt engineering (5–20% failure rates), function/tool calling (95–99% reliability), native schema-enforced APIs (100% guarantee via constrained decoding), and inference-engine-level constrained decoding for self-hosted models. Key techniques include **Finite State Machine (FSM)**-based decoding (Outlines), **Pushdown Automaton** / Context-Free Grammar approaches (XGrammar, used by vLLM as default), and **Guidance/llguidance** (used by Microsoft/OpenAI for lowest per-token latency). JSONSchemaBench (10K real-world schemas) finds the best framework supports roughly twice as many schemas as the worst.

---

## Definitions & Standards

> "Constrained decoding intervenes in the decoding process of LMs by masking out invalid tokens based on given constraints and prefix tokens. This intervention guides the LM to sample only from valid tokens, ensuring that the final output perfectly conforms to a predefined structure."

— [arXiv:2501.10868 — JSONSchemaBench](https://arxiv.org/html/2501.10868v1)

### Three Levels of Output Control

| Level | Method | Reliability | Drawbacks |
|-------|--------|-------------|-----------|
| **1** | Prompt Engineering | 80–95% | Silent failures, no type guarantees |
| **2** | Function / Tool Use | 95–99% | Schema is a hint, not a hard constraint |
| **3** | **Native Structured Output** | **100%** (guaranteed) | Constrained decoding / FSM masking |

### How Constrained Decoding Works

A **Finite State Machine (FSM)** tracks the model's position in the JSON schema and masks out invalid next tokens:

```
State Machine for {"name": string, "age": integer}:
START → expect "{"
→ expect "\"name\""
→ expect ":"
→ expect string value
→ expect "," or "}"
→ if "," → expect "\"age\""
→ expect ":"
→ expect integer value
→ expect "}"
→ DONE
```

The model chooses the most likely *valid* token, preserving quality while guaranteeing structure.

---

## Key Sources

### JSONSchemaBench — Comprehensive Benchmark

> "We pair the benchmark with the existing official JSON Schema Test Suite and evaluate six state-of-the-art constrained decoding frameworks."

> "Constrained decoding can **speed up generation by 50%** compared to unconstrained decoding depending on the framework."

> "The best-performing constrained decoding framework supports roughly **twice as many schemas** as the worst-performing one."

> "Constrained decoding consistently **improves downstream task performance by up to ~4%**, even for tasks with minimal structure like GSM8K."

— [arXiv:2501.10868](https://arxiv.org/html/2501.10868v1)

**Schema Coverage by Framework (from JSONSchemaBench paper):**

| Dataset | Guidance | XGrammar | Outlines | Llamacpp |
|---------|----------|----------|----------|----------|
| GlaiveAI | 96.7% | 89.2% | 72.4% | 82.1% |
| Github-Easy | 99.1% | 94.3% | 68.9% | 88.7% |
| Kubernetes | 87.2% | 78.5% | 41.2% | 65.3% |
| JSONSchemaStore | 91.8% | 85.6% | 52.3% | 74.9% |

### Production Guidance

> "LLMs are text generators. Your application needs data structures. The gap between these two things is where bugs live."

> "JSON mode (`type: "json_object"`) guarantees valid JSON, NOT schema compliance."

— [DEV Community — "LLM Structured Output in 2026"](https://dev.to/pockit_tools/llm-structured-output-in-2026-stop-parsing-json-with-regex-and-do-it-right-34pk)

### Framework Performance (per-token latency)

| Framework | TPOT (ms/token) | Grammar Compilation | Notes |
|-----------|-----------------|---------------------|-------|
| **Guidance (llguidance)** | 6–9 ms | Near-zero | Lowest latency; Microsoft-backed |
| **XGrammar** | ~10–15 ms | Near-zero | Default in vLLM; CMU MLC team |
| **Outlines** | 15–46 ms | 3.5–12.8s | Highest compilation overhead |
| **Llamacpp** | ~30 ms | 0.05s | Moderate overhead |

*Source: JSONSchemaBench (arXiv:2501.10868), TianPan.co*

### Grammar-Constrained Decoding (ICML 2025)

> "Grammar-constrained decoding (GCD) can guarantee that LLM outputs match specified context-free grammar rules by masking out tokens that will provably lead to invalid outputs."

> "We developed a new algorithm that prepares grammar rules much more quickly while keeping generation just as fast and accurate."

— [ICML 2025 Poster](https://icml.cc/virtual/2025/poster/45613)

---

## Repositories & Tools

### Provider Implementations

| Provider | Method | URL |
|----------|--------|-----|
| **OpenAI** | Native structured outputs with `response_format` | [docs](https://developers.openai.com/) |
| **Google Gemini** | `response_schema` in generation config | [docs](https://ai.google.dev/gemini-api/docs/openai) |
| **Anthropic Claude** | Tool use pattern | [docs](https://docs.anthropic.com/) |

### Constrained Decoding Libraries

| Library | URL | Notes |
|---------|-----|-------|
| **Guidance / llguidance** | [github.com/microsoft/guidance](https://github.com/microsoft/guidance) | Lowest per-token latency; broadest JSON Schema support |
| **XGrammar** | [github.com/microsoft/xgrammar](https://github.com/microsoft/xgrammar) | Default in vLLM; Pushdown Automaton approach |
| **Outlines** | [github.com/outlines-dev/outlines](https://github.com/outlines-dev/outlines) | FSM-based; high compilation overhead |
| **Instructor** | [python.useinstructor.com](https://python.useinstructor.com/) | Multi-provider (15+); Pydantic integration; retry logic |
| **Marvin** | [github.com/PrefectHQ/marvin](https://github.com/PrefectHQ/marvin) | `cast()` / `extract()` / `classify()` for OpenAI |
| **vLLM (built-in)** | [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) | xGrammar and Outlines backends |

### JSONSchemaBench

> "We release JSONSchemaBench, a benchmark of **10K real-world JSON schemas** across function signatures, service APIs, and system configurations."

**URL:** [arXiv:2501.10868](https://arxiv.org/html/2501.10868v1)
**Code:** Evaluation framework for comparing constrained decoding frameworks

### Awesome LLM Constrained Decoding (Paper List)

**URL:** [github.com/Saibo-creator/Awesome-LLM-Constrained-Decoding](https://github.com/Saibo-creator/Awesome-LLM-Constrained-Decoding)

Recent papers tracked:
- ICML 2025: Efficient and Asymptotically Unbiased Constrained Decoding
- ACL 2025: Grammar-Constrained Decoding Makes LLMs Better Logical Parsers
- PLDI 2025: Type-Constrained Code Generation with Language Models
- ICLR 2026: Constrained Decoding of Diffusion LLMs with Context-Free Grammars

---

## Debates & Controversies

### Constrained Decoding Speed vs. Quality

> "Constrained decoding can speed up generation by 50% compared to unconstrained decoding" — JSONSchemaBench finds guidance acceleration can make TPOT *lower* than unconstrained generation.

Counterpoint: Outlines' high grammar compilation (8–60 seconds for complex schemas) makes it impractical for dynamic schema scenarios.

### JSON Schema Coverage Gaps

Even native schema-enforced APIs exclude `minLength`, `maxLength`, `minItems`, `maxItems`, and complex regex. Models may still invent keys unless `additionalProperties: false` is set on all objects.

### Format Restrictions vs. Model Performance

> "In the LLaMA 3 8B setting, the parsing error rate for the Last Letter task in JSON format is only 0.148%, yet there exists a substantial **38.15% performance gap** as seen in Table 1."

— [arXiv:2408.02442 — "Let Me Speak Freely?"](https://arxiv.org/html/2408.02442v1)

Strict format constraints can significantly impair task performance, especially for smaller models.

---

## Summary Table

### Provider Comparison

| Feature | OpenAI | Anthropic | Gemini |
|---------|--------|-----------|--------|
| **Method** | Native SO | Tool Use | Native SO |
| **Constrained decode** | Yes | Partial | Yes |
| **100% schema valid** | Yes | 99%+ | Yes |
| **Streaming** | Yes | Yes | Yes |
| **Pydantic native** | Yes (`.parse`) | Manual | Manual |
| **Recursive schemas** | Limited | Yes | Limited |

**Recommendation:** Use OpenAI or Gemini for guaranteed schema compliance. Use Anthropic with Pydantic/Zod validation as a safety net.

### Constrained Decoding Framework Comparison

| Framework | TPOT | Grammar Compilation | JSON Schema Coverage | Best For |
|-----------|------|---------------------|----------------------|----------|
| **Guidance** | 6–9 ms | Near-zero | Highest (~90%+) | Maximum coverage, low latency |
| **XGrammar** | ~10–15 ms | Near-zero | High (~85%+) | vLLM default; balanced |
| **Outlines** | 15–46 ms | 3.5–12.8s | Moderate (~70%) | Simpler schemas; caching helps |
| **Llamacpp** | ~30 ms | 0.05s | Moderate (~80%) | llama.cpp integration |

---

## Related Topics

- [OpenAI-Compatible-API-Layers](./OpenAI-Compatible-API-Layers.md) — How structured outputs are exposed via API endpoints
- [Parallelism-Distributed-Serving](./Parallelism-Distributed-Serving.md) — Inference engine internals that power structured generation
- [Request-Response-Protocols](./Request-Response-Protocols.md) — JSON-RPC and schema-based API patterns
