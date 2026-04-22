# API-Servers

## 📌 Overview

API-Servers covers protocols, frameworks, and tooling for building and serving APIs — from low-level interface specs (ASGI) to data validation layers (Pydantic) to full-featured frameworks (FastAPI), integration protocols (MCP), reasoning metadata, and cross-cutting infrastructure topics (OpenAI-compatible layers, streaming, distributed serving, structured outputs).

---

## Subfolders

| Folder | Description |
|--------|-------------|
| [Async-Frameworks](./Async-Frameworks/) | FastAPI, Pydantic V3, ASGI-Native — async web stack |
| [MCP-Protocol](./MCP-Protocol/) | Model Context Protocol — AI tool and tool-discovery standard |
| [Reasoning-Metadata](./Reasoning-Metadata/) | Reasoning-effort params, iCoT tracking — LLM reasoning observability |
| [MISC](./MISC/) | OpenAI-compatible layers, streaming, parallelism, structured outputs, protocols |

---

## Summary

- **Async-Frameworks** — the modern Python async web stack: FastAPI (97.2k stars), Pydantic V3 (Rust-core validation), ASGI v3.0 (async server spec)
- **MCP-Protocol** — Model Context Protocol for AI agent tool integration
- **Reasoning-Metadata** — reasoning effort params and implicit CoT tracking
- **MISC** — cross-cutting: OpenAI-compatible API layers, streaming/real-time delivery, parallelism/distributed serving, structured outputs/schema-constrained generation
