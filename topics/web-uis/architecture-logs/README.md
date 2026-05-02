# Architecture Logs

> *Logging, observability, and audit patterns for LLM web interfaces — conversation history, user interaction logging, analytics, debugging tools, and compliance/audit trails.*

## Table of Contents

1. [Overview](#overview)
2. [Academic Foundations](#academic-foundations)
3. [Key Patterns & Architectures](#key-patterns--architectures)
4. [Logging Platforms & Tools](#logging-platforms--tools)
5. [Storage Decisions](#storage-decisions)
6. [Compliance & GDPR](#compliance--gdpr)
7. [References](#references)

---

## Overview

LLM web interfaces require specialized logging and observability beyond traditional application logging. Key challenges include:

- **Non-deterministic outputs** — need to capture full context for reproducibility
- **Token/cost tracking** — per-user, per-model, per-request accounting
- **Quality assessment** — user feedback, evaluation metrics, trace analysis
- **Audit trails** — compliance requirements for data processing events
- **Multi-turn conversations** — stateful interaction chains

> *"LLM observability differs from traditional observability: non-deterministic outputs, subjective quality measures, API cost focus."*
> — [ClickHouse Engineering](https://clickhouse.com/resources/engineering/llm-observability)

---

## Academic Foundations

### Audit Trails for Accountability in LLMs

> **Source:** [arXiv:2601.20727](https://arxiv.org/html/2601.20727v1)

**Three-Layer Architecture:**
```
Capture Layer → Store Layer → Use Layer
```

**Key Concepts:**
- Audit trail defined as "a chronological, tamper-evident, context-rich ledger of lifecycle events and decisions"
- Links technical provenance (models, data, training, deployments) with governance records (approvals, waivers, attestations)
- Hash chain mechanism for tamper-evidence: `prev_hash → curr_hash` using SHA-256
- Covers full lifecycle: pretraining, base model selection, adaptation/evaluation, deployment, operations

**Why Valuable:**
- Provides formal framework for compliance-focused logging
- Full lifecycle coverage from training to production

---

### AgentTrace: Structured Logging Framework for Agent Observability

> **Source:** [arXiv:2602.10133v1](https://arxiv.org/html/2602.10133v1)

**Three-Surface Taxonomy:**
| Surface | Focus | Captures |
|---------|-------|----------|
| **Cognitive** | LLM interaction introspection | Prompts, completions, CoT, confidence |
| **Operational** | Method-level execution tracing | Function calls, timing, returns |
| **Contextual** | External system I/O | HTTP, SQL, cache, vector stores, files |

**Schema:** `L(S:E:C) → R` where S=surface, E=event, C=context, R=record

**Properties:** Consistency, Causality, Fidelity, Interoperability

**Dual-path storage:** JSONL files for offline inspection + OpenTelemetry spans for real-time tracing

---

### Vector DBs and LLM Applications

> **Source:** [arXiv:2402.01763v2](https://arxiv.org/html/2402.01763v2)

> *"Vector DBs excel at semantic similarity search but lack structured query capabilities. RAG architectures benefit from vector storage for retrieval but require complementary relational DBs for transactional state."*

**Key Insight:** Vector DBs are NOT recommended for raw chat logs — only if semantic search is needed.

---

## Key Patterns & Architectures

### Three-Layer Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Capture Layer                         │
│  (Application code, SDK instrumentation, middleware)          │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         Store Layer                           │
│  (Append-only, tamper-evident, schema-based records)           │
└─────────────────────────────┬───────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                          Use Layer                            │
│  (Query, verify, audit, debug, evaluate)                      │
└─────────────────────────────────────────────────────────────┘
```

### Middleware/Pipeline Pattern

For adding logging to existing LLM web UIs:

```
OpenWebUI → Pipeline Server → LLM Endpoint → OTel Collector → Logging Backend → Dashboard
```

> *"OpenWebUI has basic conversation history but lacks visibility into token usage, latency, GPU load, cost per query."*
> — [OpenLIT Blog](https://openlit.io/blogs/openlit-openwebui)

**OpenLIT integration provides:**
- Distributed traces for every LLM call
- Token/cost metrics
- GPU performance monitoring

---

## Logging Platforms & Tools

### Langfuse

| Field | Value |
|-------|-------|
| **Repository** | [langfuse/langfuse](https://github.com/langfuse/langfuse) |
| **Stars** | 24.5k |
| **Storage** | ClickHouse |

**Features:**
- Open source LLM engineering platform
- Observability, prompt management, evaluations, datasets
- SDKs for Python, JS/TS
- Integrations: LangChain, LlamaIndex, Vercal AI, OpenAI

**Ecosystem:** Top projects using Langfuse: langflow (116k stars), open-webui (109k stars), lobe-chat (65k stars)

---

### OpenLLMetry (Traceloop)

| Field | Value |
|-------|-------|
| **Repository** | [traceloop/openllmetry](https://github.com/traceloop/openllmetry) |
| **Stars** | 7k |

**Features:**
- OpenTelemetry-based observability for GenAI/LLM
- 23+ destinations: Datadog, Grafana, New Relic, Honeycomb, Sentry, etc.
- 16+ LLM providers, 7 vector DBs, 10 frameworks
- Semantic conventions now part of OpenTelemetry standard

> *"OpenTelemetry GenAI semantic conventions (`gen_ai.*` attributes) becoming industry standard."*

---

### Helicone

| Field | Value |
|-------|-------|
| **Repository** | [helicone/helicone](https://github.com/helicone/helicone) |
| **License** | Apache v2.0 |

**Features:**
- Proxy-based request logging (single line of code to integrate)
- Handles billions of logs with real-time insights
- Self-hostable via Docker or Helm
- Architecture: worker, web, bifrost (gateway), clickhouse, supabase

**Why Valuable:** Simplest path to LLM observability — gateway pattern enables logging without code changes.

---

### Comparison Matrix

| Tool | Stars | Storage | Best For | Integration |
|------|-------|---------|----------|------------|
| **Langfuse** | 24.5k | ClickHouse | Full LLM engineering platform | SDK |
| **OpenLLMetry** | 7k | Any OTel backend | Enterprise observability stacks | SDK |
| **Helicone** | — | ClickHouse | Proxy-based (no code changes) | Proxy |
| **Open WebUI** | 109k | SQLite/PG | Self-hosted LLM UI | Built-in |
| **LangSmith** | — | Proprietary | Evaluation & debugging | SDK |

---

## Storage Decisions

### Storage Architecture Patterns

> *"PostgreSQL is fine for structured metadata and basic history. ClickHouse is emerging as standard for high-volume trace/observability data."*
> — [ClickHouse Engineering](https://clickhouse.com/resources/engineering/llm-observability)

### When to Use Which Storage

| Use Case | Recommended Storage |
|----------|---------------------|
| **Raw chat logs (searchable)** | PostgreSQL, MongoDB |
| **Semantic search on conversations** | Vector DB (Qdrant, Milvus, PGVector) |
| **High-volume tracing** | ClickHouse, TimescaleDB |
| **Audit trail (tamper-evident)** | Append-only store with hash chaining |
| **Session state** | Redis |
| **Full-text message search** | Meilisearch, Elasticsearch |

### LangChain Migration Case Study

> *"LangChain migrated from Postgres to ClickHouse to handle 'large percentage of production data' for tracing, datasets, evaluations, A/B testing."*

**ClickHouse advantages:**
- High-throughput ingestion
- Real-time analytics
- Efficient trace lookups
- Cost-effective scaling

---

## Compliance & GDPR

### GDPR Considerations for LLM Web UIs

> *"Every LLM API call routing user data is a GDPR data processing event. Requires Data Processing Agreement (DPA) with LLM provider under Article 28."*
> — [DEV Community](https://dev.to/tiamatenity/the-gdpr-fine-you-dont-know-youre-accumulating-why-every-llm-api-call-is-a-compliance-event-heb)

### Key Requirements:

1. **Data Processing Agreement (DPA)** — Required with each LLM provider
2. **Cross-border transfer rules** — EU user data to US-based LLM APIs
3. **Retention and deletion** — User data cannot be kept indefinitely
4. **Audit trail** — Must demonstrate compliance on demand

### Audit Trail Requirements

From [arXiv:2601.20727](https://arxiv.org/html/2601.20727v1):

> *"An audit trail is a chronological, tamper-evident, context-rich ledger of lifecycle events and decisions."*

**Tamper-evidence mechanism:**
```
prev_hash → curr_hash (SHA-256 hash chain)
```

**Must capture:**
- User identity and consent
- Input data processed
- Model/version used
- Output delivered
- Timestamp and duration

---

## OpenTelemetry Standards

### GenAI Semantic Conventions

> **Source:** [OpenTelemetry - GenAI Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)

**Standardized attributes:**
- `gen_ai.system` — LLM provider (openai, anthropic, etc.)
- `gen_ai.operation.name` — Operation type (chat, completion, embedding)
- `gen_ai.prompt.*` — Prompt token counts
- `gen_ai.completion.*` — Completion token counts
- `gen_ai.response.model` — Model used

**Provider-specific conventions:**
- OpenAI
- Anthropic
- AWS Bedrock
- Azure AI Inference

**Agent and framework conventions:**
- Model Context Protocol (MCP)
- GenAI agent span conventions

---

## References

### Academic Papers
1. [Audit Trails for Accountability in LLMs](https://arxiv.org/html/2601.20727v1) — arXiv:2601.20727
2. [AgentTrace: Structured Logging Framework](https://arxiv.org/html/2602.10133v1) — arXiv:2602.10133
3. [When LLMs Meet Vector Databases: A Survey](https://arxiv.org/html/2402.01763v2) — arXiv:2402.01763v2
4. [Sovereign Context Protocol](https://arxiv.org/html/2603.27094v1) — Attribution layer for content access

### Platform Documentation
5. [OpenTelemetry GenAI Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/) — Industry standard
6. [Langfuse Documentation](https://langfuse.com/) — LLM engineering platform
7. [OpenLLMetry (Traceloop)](https://github.com/traceloop/openllmetry) — OTel-native observability

### Engineering Blog Posts
8. [ClickHouse: LLM Observability](https://clickhouse.com/resources/engineering/llm-observability) — Storage architecture
9. [OpenLIT: Monitoring OpenWebUI](https://openlit.io/blogs/openlit-openwebui) — Middleware integration
10. [freeCodeCamp: LLM Observability in FastAPI](https://www.freecodecamp.org/news/build-end-to-end-llm-observability-in-fastapi-with-opentelemetry/) — Tutorial
11. [TrueFoundry: AI Gateway Logging](https://www.truefoundry.com/blog/logging-architecture-ai-gateway) — Enterprise patterns

### Compliance
12. [DEV Community: GDPR and LLM API Calls](https://dev.to/tiamatenity/the-gdpr-fine-you-dont-know-youre-accumulating-why-every-llm-api-call-is-a-compliance-event-heb) — Compliance awareness

### Talks & Videos
13. [Why You Should Care About Observability in LLM Workflows](https://www.youtube.com/watch?v=TcQ3Z1_Q-nY) — Practitioner perspective
14. [LLM observability in production: tracing and online evals](https://www.youtube.com/watch?v=r-CYvjJ-W3k) — Production evals
15. [The Only Way to Debug AI Agents](https://www.youtube.com/watch?v=2mD39iU8xZg) — Agent debugging
16. [ClickHouse: Intro to Observability](https://clickhouse.com/videos/intro-to-observability) — Storage tradeoffs

---

*Part of [[web-uis]] — Interface Comparison*
