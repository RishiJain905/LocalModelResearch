# Web UIs

> *Interface comparison for LLM interactions — self-hosted web UIs, agentic UX patterns, and logging/observability.*

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
- **Agentic UX** — autonomy dials, explainable rationale, human-in-the-loop patterns
- **Observability** — conversation history, token tracking, debugging

---

## Topics

### [[agentic-ux-patterns]] — Agentic UX Design Patterns

> *Design patterns for trustworthy, controllable, accountable AI agents.*

| Sub-topic | Focus |
|-----------|-------|
| [[autonomy-dials]] | User-controlled autonomy levels (Andrej Karpathy, June 2025) |
| [[explainable-rationale]] | Visible reasoning / Stream of Thought |
| [[hitl]] | Human-in-the-Loop patterns (approval gates, checkpoint/interrupt) |

### [[open-webui]] — Ollama-Native Web UI

> *An open, extensible, and usable interface for AI interaction — self-hosted, privacy-focused, with native Ollama integration.*

- **Architecture:** Python (FastAPI) + SvelteKit
- **Key Strength:** Best Ollama integration, extensive plugin ecosystem
- **Stars:** 135k+ GitHub
- **Enterprise:** RBAC, LDAP, SCIM 2.0 support

### [[librechat]] — Multi-Provider Gateway

> *A self-hosted AI chat platform that unifies multiple AI providers in a single, privacy-focused interface.*

- **Architecture:** Node.js/TypeScript + React + MongoDB
- **Key Strength:** Multi-provider abstraction (OpenAI, Anthropic, Azure, AWS, etc.)
- **Stars:** 36k+ GitHub
- **Enterprise:** OAuth2, SAML, LDAP authentication

### [[architecture-logs]] — Logging & Observability

> *Logging, observability, and audit patterns for LLM web interfaces.*

- **Key Patterns:** Three-layer architecture (Capture→Store→Use), middleware/pipeline pattern
- **Tools:** Langfuse, OpenLLMetry, Helicone, OpenTelemetry
- **Compliance:** GDPR requirements, audit trails, tamper-evidence

---

## Quick Reference

### Agentic UX Lifecycle (Smashing Magazine)

| Phase | Patterns |
|-------|----------|
| **Pre-Action** | Autonomy Dial, Intent Preview |
| **In-Action** | Explainable Rationale, Confidence Signal |
| **Post-Action** | Action Audit & Undo, Escalation Pathway |

### Decision Matrix

| Need | Best Choice |
|------|-----------|
| User controls how much agent can do independently | [[autonomy-dials]] |
| User wants to understand agent reasoning | [[explainable-rationale]] |
| High-stakes actions require human approval | [[hitl]] |
| Simplest Ollama setup | [[open-webui]] |
| Multi-provider (OpenAI + Anthropic + Azure) | [[librechat]] |

### HITL Modes

| Mode | Human Role | When Required |
|------|-----------|---------------|
| **HITL** | Approval before action | Irreversible, regulated, financial/medical |
| **HOTL** | Monitor and override post-hoc | Correctable, moderate stakes |
| **HOOTL** | None | Low-stakes, retryable |

---

## See Also

- [Open WebUI Official Docs](https://docs.openwebui.com/)
- [LibreChat Official Docs](https://www.librechat.ai/docs)
- [Smashing Magazine: Designing For Agentic AI](https://www.smashingmagazine.com/2026/02/designing-agentic-ai-practical-ux-patterns/)
- [LangGraph Checkpointing](https://langchain.com/langgraph/)
- [OpenTelemetry GenAI Conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)

---

*Last updated: 2026-05-02*
