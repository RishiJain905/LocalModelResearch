# Web UIs

> *Interface comparison for LLM interactions — self-hosted web UIs, their architectures, features, and logging/observability patterns.*

## Table of Contents

1. [Overview](#overview)
2. [Topics](#topics)
3. [Quick Reference](#quick-reference)

---

## Overview

Web UIs for LLM interactions provide user-facing interfaces for chatting with language models. Key considerations include:

- **Multi-provider support** — unified access to OpenAI, Anthropic, local models, etc.
- **Self-hosting** — data control, privacy, cost savings vs SaaS
- **Extensibility** — plugins, custom tools, RAG pipelines
- **Enterprise features** — authentication, RBAC, audit logging
- **Observability** — conversation history, token tracking, debugging

> *"The ChatGPT interface—once OpenAI's moat—is now table stakes. Any of these three projects delivers 90% of the ChatGPT experience."*
> — [Elestio Blog](https://blog.elest.io/the-best-open-source-chatgpt-interfaces-lobechat-vs-open-webui-vs-librechat/)

---

## Topics

### [[open-webui]] — Ollama-Native Web UI

> *An open, extensible, and usable interface for AI interaction — self-hosted, privacy-focused, with native Ollama integration.*

- **Architecture:** Python (FastAPI) + SvelteKit
- **Key Strength:** Best Ollama integration, extensive plugin ecosystem
- **Stars:** 135k+ GitHub
- **Enterprise:** RBAC, LDAP, SCIM 2.0 support
- **Best for:** Developers, self-hosters, Ollama users

### [[librechat]] — Multi-Provider Gateway

> *A self-hosted AI chat platform that unifies multiple AI providers in a single, privacy-focused interface.*

- **Architecture:** Node.js/TypeScript + React + MongoDB
- **Key Strength:** Multi-provider abstraction (OpenAI, Anthropic, Azure, AWS, etc.)
- **Stars:** 36k+ GitHub
- **Enterprise:** OAuth2, SAML, LDAP authentication
- **Best for:** Organizations needing multi-provider flexibility

### [[architecture-logs]] — Logging & Observability

> *Logging, observability, and audit patterns for LLM web interfaces — conversation history, user interaction logging, analytics, debugging tools, and compliance/audit trails.*

- **Key Patterns:** Three-layer architecture (Capture→Store→Use), middleware/pipeline pattern
- **Tools:** Langfuse, OpenLLMetry, Helicone, OpenTelemetry
- **Storage:** ClickHouse for high-volume, PostgreSQL for structured, Vector DBs for semantic search
- **Compliance:** GDPR requirements, audit trails, tamper-evidence

---

## Quick Reference

### Decision Matrix

| Need | Best Choice |
|------|-----------|
| Simplest Ollama setup | [[open-webui]] |
| Multi-provider (OpenAI + Anthropic + Azure) | [[librechat]] |
| Native Ollama integration | [[open-webui]] |
| Enterprise auth (OAuth2, SAML, LDAP) | [[librechat]] |
| Extensive plugin ecosystem | [[open-webui]] |
| Code interpreter (sandboxed) | [[librechat]] |
| RAG-heavy workflows | [[open-webui]] |
| LLM observability/logging | [[architecture-logs]] |

### Comparison Table

| Feature | Open WebUI | LibreChat |
|---------|-----------|-----------|
| **License** | MIT | MIT |
| **Architecture** | FastAPI + SvelteKit | Node.js + React |
| **Ollama Integration** | Native (best) | Via custom endpoints |
| **Multi-Provider** | Good | Excellent (15+ providers) |
| **RAG Support** | Advanced (9 vector DBs) | Via LangChain |
| **Plugin Ecosystem** | Extensive (800+ plugins) | Moderate |
| **Enterprise Auth** | RBAC, LDAP, SCIM | OAuth2, SAML, LDAP |
| **Code Interpreter** | Pyodide (browser) | Sandboxed (multi-language) |
| **GitHub Stars** | 135k+ | 36k+ |

### Observability Stack

```
┌─────────────────────────────────────────────────────┐
│              LLM Web UI Application                  │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│         Instrumentation (SDK / Middleware)            │
│   Langfuse | OpenLLMetry | Helicone | OpenLIT       │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│              OpenTelemetry Collector                  │
└─────────────────────────┬───────────────────────────┘
                          ▼
┌─────────────────────────────────────────────────────┐
│                   Storage Backend                     │
│  ClickHouse | PostgreSQL | Vector DB | Elasticsearch │
└─────────────────────────────────────────────────────┘
```

### Standards

- **OpenTelemetry GenAI Conventions** — `gen_ai.*` attributes for standardized LLM observability
- **Model Context Protocol (MCP)** — Open standard for AI tool integration

---

## See Also

- [Open WebUI Official Docs](https://docs.openwebui.com/)
- [LibreChat Official Docs](https://www.librechat.ai/docs)
- [OpenTelemetry GenAI Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)
- [Langfuse](https://langfuse.com/) — LLM engineering platform
- [OpenLLMetry](https://github.com/traceloop/openllmetry) — OTel-native observability

---

*Last updated: 2026-05-02*
