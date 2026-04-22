# Tool Discovery

> *"Tools in MCP are designed to be **model-controlled**, meaning that the language model can discover and invoke tools automatically based on its contextual understanding and the user's prompts."* — [MCP Specification](https://modelcontextprotocol.io/specification/draft/server/tools)

## Contents

1. [Definitions & Standards](#definitions--standards)
2. [Discovery Mechanisms](#discovery-mechanisms)
3. [MCP Registry](#mcp-registry)
4. [GitHub Repositories](#github-repositories)
5. [Blog Posts & Articles](#blog-posts--articles)
6. [Debates & Trade-offs](#debates--trade-offs)
7. [Key Findings](#key-findings)

---

## Definitions & Standards

### Core Definition

> *"To discover available tools, clients send a `tools/list` request."*
> — [MCP Specification — Tools](https://modelcontextprotocol.io/specification/draft/server/tools)

### Official Tool Discovery Protocol

> *"Servers that support tools **MUST** declare the `tools` capability."*
> — [MCP Specification](https://modelcontextprotocol.io/specification/draft/server/tools)

> *"`listChanged` indicates whether the server will emit notifications when the list of available tools changes."*
> — [MCP Specification](https://modelcontextprotocol.io/specification/draft/server/tools)

> *"When the list of available tools changes, servers that declared the `listChanged` capability **SHOULD** send a notification: `notifications/tools/list_changed`"*
> — [MCP Specification](https://modelcontextprotocol.io/specification/draft/server/tools)

### JSON-RPC Protocol Messages

```json
// Request (tools/list)
{ "jsonrpc": "2.0", "id": 1, "method": "tools/list", "params": { "cursor": "optional-cursor-value" } }

// Response
{ "jsonrpc": "2.0", "id": 1, "result": { "tools": [{ "name": "get_weather", "description": "...", "inputSchema": {...} }], "nextCursor": "..." } }
```

---

## Discovery Mechanisms

### 4-Level Discovery Architecture

| Level | Mechanism | Standard | Status |
|-------|-----------|----------|--------|
| **Protocol-level** | `tools/list` JSON-RPC | Official MCP Spec | Production |
| **Registry-level** | MCP Registry REST API | modelcontextprotocol/registry | Production |
| **Web-level** | `.well-known/mcp` (SEP-1960) | RFC 8615 | Proposed |
| **Configuration-level** | `mcp.json` client config | gofastmcp.com | Production |

### MCP JSON Configuration Standard

```json
{
  "mcpServers": {
    "server-name": {
      "command": "executable",
      "args": ["arg1", "arg2"],
      "env": { "VAR": "value" }
    }
  }
}
```

> *"Client configuration standard for MCP servers."*
> — [gofastmcp.com](https://gofastmcp.com/integrations/mcp-json-configuration)

---

## MCP Registry

### Official MCP Registry

> *"A community driven registry service for Model Context Protocol (MCP) servers—an **'app store' API** that lists MCP servers and supports publishing with namespace ownership verification."*
> — [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io/docs)

> *"The official MCP Registry provides a REST API for programmatic publishing and consumption, while GitHub's registry integrates directly into Copilot and VS Code with one-click installation."*
> — [registry.modelcontextprotocol.io](https://registry.modelcontextprotocol.io/docs)

### Namespace Verification Methods

| Namespace Type | Verification Method |
|---------------|---------------------|
| `io.github.username` | GitHub OAuth |
| `com.example` | DNS TXT records |

### MCP Registry Ecosystem

> *"The MCP Registry ecosystem defines relationships between: Package Registries, Server Developers, Downstream Aggregators, Other MCP Registries, and MCP Host Applications."*
> — [modelcontextprotocol.io/registry/about](https://modelcontextprotocol.io/registry/about)

### Self-Hosted Registry

```bash
docker run -p 8080:8080 ghcr.io/modelcontextprotocol/registry:latest
```

> [github.com/modelcontextprotocol/registry](https://github.com/modelcontextprotocol/registry)

---

## Web-Level Discovery (.well-known)

### SEP-1960: .well-known/mcp Discovery Endpoint

> *"This proposal defines a standardized `/.well-known/mcp` endpoint for MCP server metadata discovery, capability advertisement, and security policy declaration."*
> — [GitHub Issue #1960](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1960)

**Required Fields:**
- `mcp_version`
- `endpoints` array

### SEP-1649: Server Cards

> *Server Card metadata at `/.well-known/mcp/server-card.json`* — [Smithery](https://smithery.ai/docs/build/publish)

### .well-known/mcp Discussion

> *"This RFC defines how MCP Clients discover and connect to MCP Servers, inspired by the Web of Things (WoT) discovery standard."*
> *"The `.well-known` URI scheme (RFC 8615) is the established pattern for service metadata discovery."*
> — [GitHub Discussion #1147](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1147)

### Meta Tags vs .well-known Debate

| Approach | Pros | Cons |
|----------|------|------|
| `.well-known/mcp.json` | Most discussed, extensible | 99.9% of sites return 404 |
| Meta/Link tags | No 404s, only parsed if present | Requires parsing HTML |

> *"Meta tags avoid this entirely - they're only parsed if present, no 404s, no extra requests."*
> — [nickk](https://github.com/modelcontextprotocol/modelcontextprotocol/discussions/1147)

---

## GitHub Repositories

| Repository | Stars | Description |
|-----------|-------|-------------|
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | ⭐ 84.1k | Community MCP server registry |
| [modelcontextprotocol/registry](https://github.com/modelcontextprotocol/registry) | — | Self-hosted MCP registry (Docker) |
| [punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers) | ⭐ 85.3k | Curated list of MCP servers |
| [prefecthq/fastmcp](https://github.com/prefecthq/fastmcp) | ⭐ 23.5k | Framework with automatic tool manifest generation |
| [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) | ⭐ 22.7k | Official Python SDK (FastMCP v1) |
| [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) | ⭐ 12.2k | Official TypeScript SDK |

### MCP Inspector

> [github.com/modelcontextprotocol/inspector](https://github.com/modelcontextprotocol/inspector) | ⭐ 9.5k

Interactive tool for testing and debugging MCP servers — useful for inspecting what tools a server exposes.

---

## Blog Posts & Articles

### MCP Server Discovery: Implement .well-known/mcp.json (2026 Guide)

> *"Discovery answers where to connect and what is available, while initialization handles how to communicate."*
> *"Server discovery via `.well-known/mcp.json` endpoints solves this by allowing AI assistants (Claude, ChatGPT, Cursor) to automatically detect, inspect, and connect to servers without manual setup."*
> — [ekamoira.com](https://www.ekamoira.com/blog/mcp-server-discovery-implement-well-known-mcp-json-2026-guide)

> *"The Model Context Protocol (MCP) ecosystem has grown to **97 million monthly SDK downloads** and **10,000+ active public MCP servers**."*
> — [ekamoira.com](https://www.ekamoira.com/blog/mcp-server-discovery-implement-well-known-mcp-json-2026-guide)

### How the Model Context Protocol (MCP) Works

> *"Think of it as a JSON-RPC interface for AI: the model acts as the client, and the enterprise's tools and data systems act as MCP servers."*
> — [Lucidworks](https://lucidworks.com/blog/how-the-model-context-protocol-works-a-technical-deep-dive)

### MCP vs A2A vs ACP vs ANP: Complete AI Agent Protocol Guide

> *"MCP connects AI to tools, A2A enables agent-to-agent collaboration, and ANP provides decentralized discovery."*
> — [jitendrazaa.com](https://www.jitendrazaa.com/blog/ai/mcp-vs-a2a-vs-acp-vs-anp-complete-ai-agent-protocol-guide/)

> *"MCP Adoption Stats: 97+ million monthly SDK downloads, 10,000+ published MCP servers, 300+ MCP clients"*
> — [jitendrazaa.com](https://www.jitendrazaa.com/blog/ai/mcp-vs-a2a-vs-acp-vs-anp-complete-ai-agent-protocol-guide/)

### Dynamic Tool Registry with Anthropic's MCP

> *"Tools are registered server-side using MCP SDK (e.g., `FastMCP` in Python), and clients discover via `list_tools()`, returning names, descriptions, and schemas."*
> — [LinkedIn / Walid Negm](https://www.linkedin.com/pulse/dynamic-tool-registry-anthropics-mcp-foundation-multi-step-walid-negm-54lye)

### Building Your First MCP Server

> *"Uses `@modelcontextprotocol/sdk/server` for the Server class and `ListToolsRequestSchema`/`CallToolRequestSchema` from `@modelcontextprotocol/sdk/types.js` for handler registration."*
> — [dhanushka.dev](https://dhanushka.dev/blog/building-your-first-mcp-server-a-practical-guide)

### A2A vs MCP Protocol Comparison

> *"MCP servers expose tools; A2A agents advertise capabilities and accept task assignments."*
> — [fast.io](https://fast.io/resources/a2a-vs-mcp-protocol-comparison/)

---

## Debates & Trade-offs

### MCP vs OpenAPI/CLI Alternatives

> *"The whole protocol feels off to me... why do we need MCP for tool definitions?"*
> — Chris, [Materialized View](https://materializedview.io/p/mcp-server-could-have-been-json-file) — September 11, 2025

> *"There is no law that LLMs need a new protocol to interact with software... Existing standards (OpenAPI, gRPC, CLIs) handle most use cases."*
> — Chris, [Materialized View](https://materializedview.io/p/mcp-server-could-have-been-json-file)

**Benchmark Data:**
> *"MCP vs CLI truly is a wash: Both `terminalcp` MCP and CLI versions achieved 100% success rates. The MCP version was 23% faster (51m vs 66m) and 2.5% cheaper ($19.45 vs $19.95)."*
> — [Materialized View](https://materializedview.io/p/mcp-server-could-have-been-json-file)

### MCP Server Registry Decentralization Debate

> *"Centralizing its index would just create a single point of failure... seems unwise to put the future of MCP in the hands of just one administrator."*
> — [fjm2u](https://github.com/orgs/modelcontextprotocol/discussions/159)

> *"A grand central registry vs. a registry protocol/spec... standardize what a 'feed' looks like so that apps can build marketplaces/MCP pickers on top of feeds, and provide an 'official' feed."*
> — [kevklam](https://github.com/orgs/modelcontextprotocol/discussions/159)

### Namespace Verification Bug

> *"The validator enforces that `io.github.username` namespaces must match a derived `username.github.io` domain URL."*
> — [AugmentCode](https://www.augmentcode.com/mcp/mcp-registry)

**Affected deployments:** Cloudflare Workers, AWS Lambda, Railway

**Workaround:** Use custom domain namespace (`com.example`) with DNS TXT verification.

### Enterprise vs Consumer Discovery Use Cases

> *"Enterprise/Developer: Registry of local MCP servers for software development"*
> *"Consumer/AI: Arbitrary servers exposed via web (e.g., every WordPress site offering MCP for commenting)"*
> — [GitHub Discussion #159](https://github.com/orgs/modelcontextprotocol/discussions/159)

---

## Key Findings

- **4-level discovery stack**: Protocol (`tools/list`) → Registry (REST API) → Web (`.well-known`) → Config (`mcp.json`)
- **97+ million monthly SDK downloads**, **10,000+ published MCP servers** as of early 2026
- **`tools/list` is the core runtime discovery** — servers MUST declare `tools` capability and SHOULD emit `list_changed` notifications
- **`.well-known/mcp.json` (SEP-1960)** standardizes web-level discovery via RFC 8615
- **Meta tags vs .well-known debate** — no-404 meta tag approach proposed as alternative to 404-prone `.well-known` endpoints
- **MCP Registry** supports namespace ownership via GitHub OAuth or DNS TXT records
- **FastMCP automatically generates tool manifests** — making discovery effortless for Python servers
- **Competing protocols**: A2A (agent-to-agent), ANP (decentralized), ACP (merged into A2A)
- **Namespace validator bug** affects Cloudflare Workers, AWS Lambda, Railway deployments
- **Linux Foundation (AAIF)** governs MCP since December 2025

---

## Related Topics

- [FastMCP](./FastMCP.md) — Python framework with automatic tool discovery
- [Standard-Connectors](./Standard-Connectors.md) — Transport mechanisms and official SDKs
