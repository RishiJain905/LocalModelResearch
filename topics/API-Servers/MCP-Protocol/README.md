# MCP-Protocol

> *The Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools.* — [modelcontextprotocol.io](https://modelcontextprotocol.io/specification/2025-06-18)

## Overview

This folder covers the **Model Context Protocol (MCP)** — an open standard for connecting LLMs to external tools, data sources, and services. MCP defines how AI models discover, connect to, and invoke capabilities from external servers through standardized transport mechanisms and a JSON-RPC message layer.

**Governed by:** The Linux Foundation (AAIF, since December 2025)

**Ecosystem Stats (as of early 2026):**
- 97+ million monthly SDK downloads
- 10,000+ published MCP servers
- 300+ MCP clients
- 84.1k stars on official `modelcontextprotocol/servers` repo

---

## Subtopics

| Document | Description |
|----------|-------------|
| [FastMCP](./FastMCP.md) | The dominant Python framework for building MCP servers — 1M PyPI downloads/day, powers ~70% of MCP servers |
| [Standard-Connectors](./Standard-Connectors.md) | Official transport mechanisms (stdio, Streamable HTTP), SDKs in 10 languages, company-maintained servers |
| [Tool-Discovery](./Tool-Discovery.md) | How tools are discovered — `tools/list` protocol, MCP Registry, `.well-known` web discovery |

---

## MCP Architecture Summary

### Primitives

| Primitive | Control | Description |
|-----------|---------|-------------|
| **Tools** | Model-controlled | Executable functions the LLM can call |
| **Resources** | Application-controlled | Structured data/context for the model |
| **Prompts** | User-controlled | Pre-defined templates invoked by user choice |

### Transport Mechanisms

| Transport | Type | Best For |
|-----------|------|----------|
| **stdio** | Local pipes | CLI tools, local development |
| **Streamable HTTP** | HTTP POST/GET | Remote access, multi-client, cloud |
| SSE | (deprecated) | Legacy — replaced by Streamable HTTP |

### Discovery Levels

| Level | Mechanism | Status |
|-------|-----------|--------|
| Protocol | `tools/list` JSON-RPC | Production |
| Registry | MCP Registry REST API | Production |
| Web | `.well-known/mcp` (SEP-1960) | Proposed |
| Config | `mcp.json` client config | Production |

---

## Key Organizations

| Organization | Role |
|--------------|------|
| Anthropic | Original MCP creator |
| Linux Foundation (AAIF) | Governance (since Dec 2025) |
| PrefectHQ | FastMCP framework |
| Microsoft | 10 official servers, C# SDK, VS Code integration |
| AWS Labs | 25+ official servers |
| GitHub | Official MCP server |
| Smithery | Third-party registry |
| AugmentCode | MCP Registry development |

---

## Related Topics

- [FastMCP](./FastMCP.md) — Build MCP servers with Python
- [Standard-Connectors](./Standard-Connectors.md) — Transports, SDKs, company servers
- [Tool-Discovery](./Tool-Discovery.md) — Discovery mechanisms and registries
