# Standard Connectors

> *"In the same way that a USB provides a standardized way to connect various devices to a computer, the MCP is designed to standardize how AI models connect to and interact with external systems."* — [Sourcing Speak](https://www.sourcingspeak.com/model-context-protocol-mcp-connector-basics/)

## Contents

1. [Definitions & Standards](#definitions--standards)
2. [Transport Mechanisms](#transport-mechanisms)
3. [Official SDKs & Servers](#official-sdks--servers)
4. [GitHub Repositories](#github-repositories)
5. [Company-Maintained Connectors](#company-maintained-connectors)
6. [Blog Posts & Articles](#blog-posts--articles)
7. [Debates & Trade-offs](#debates--trade-offs)
8. [Key Findings](#key-findings)

---

## Definitions & Standards

### What is MCP?

> *"The Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools."*
> — [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-06-18)

### MCP Primitives (Server Capabilities)

> *"Servers provide the fundamental building blocks for adding context to language models via MCP. These primitives enable rich interactions between clients, servers, and language models:
> - **Prompts**: Pre-defined templates or instructions that guide language model interactions
> - **Resources**: Structured data or content that provides additional context to the model
> - **Tools**: Executable functions that allow models to perform actions or retrieve information"*
> — [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-06-18/server)

### Control Hierarchy for Primitives

| Primitive | Control | Interaction | Examples |
|-----------|---------|-------------|----------|
| Prompts | User-controlled | Interactive templates invoked by user choice | Slash commands, menu options |
| Resources | Application-controlled | Contextual data attached and managed by the client | File contents, git history |
| Tools | Model-controlled | Functions exposed to the LLM to take actions | API POST requests, file writing |

> — [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-06-18/server)

### MCP Connectors vs Standardization

> *"By standardizing these, MCP makes it clear how an AI can discover 'what can I do?' when connected to a new server (list tools, list resources, list prompts) and then how to execute actions or gather context."*
> — [GPT Trainer](https://gpt-trainer.com/blog/anthropic+model+context+protocol+mcp)

> *"MCP solves the **transport layer problem**, not the **reasoning problem**."*
> — [Unblocked](https://getunblocked.com/blog/why-mcp-servers-arent-enough)

---

## Transport Mechanisms

### Official Transport Specification (Latest: 2025-03-26)

> *"MCP currently defines two standard transport mechanisms for client-server communication:
> 1. **stdio**, communication over standard in and standard out
> 2. **Streamable HTTP**, HTTP-based communication (replaces HTTP+SSE from 2024-11-05)"*
> — [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports)

### Transport Comparison

| Aspect | stdio | Streamable HTTP | SSE (deprecated) |
|--------|-------|-----------------|-------------------|
| Communication | Local pipes | HTTP POST/GET | HTTP + SSE stream |
| Latency | Lowest | Medium | High |
| Remote Access | No | Yes | Yes |
| Multi-Client | No | Yes | Yes |
| Stateful | No | Yes (optional session) | Yes |
| Best Use Case | CLI, local tools | Cloud, web apps | Legacy only |

> *"Clients **SHOULD** support stdio whenever possible."*
> — [MCP Specification](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports)

### Custom Transport Extension

> *"Clients and servers **MAY** implement additional custom transport mechanisms to suit their specific needs. The protocol is transport-agnostic and can be implemented over any communication channel that supports bidirectional message exchange."*
> — [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports)

### Transport Recommendation

> *"Choose stdio for local CLI tools, StreamableHTTP for web/remote access. SSE is deprecated."*
> — [mcpcat.io](https://mcpcat.io/guides/comparing-stdio-sse-streamablehttp/)

### Security Requirements for HTTP Transport

> *"When implementing HTTP with SSE transport:
> 1. Servers **MUST** validate the `Origin` header on all incoming connections to prevent DNS rebinding attacks
> 2. When running locally, servers **SHOULD** bind only to localhost (127.0.0.1) rather than all network interfaces (0.0.0.0)
> 3. Servers **SHOULD** implement proper authentication for all connections"*
> — [modelcontextprotocol.io/specification](https://modelcontextprotocol.io/specification/2025-03-26/basic/transports)

### DNS Rebinding Vulnerability (CVE-2025-66414 / CVE-2025-66416)

> *"Both the official MCP TypeScript SDK (< 1.24.0) and Python SDK (< 1.23.0) lack DNS rebinding protection for localhost-bound SSE and StreamableHTTP servers"*
> — [vulnerablemcp.info](https://vulnerablemcp.info/vuln/cve-2025-66414-66416-dns-rebinding-mcp-sdks.html)

---

## Official SDKs & Servers

### MCP Specification & Governance

> *"The Model Context Protocol is an open source project hosted by The Linux Foundation and is open to contributions from the entire community."*
> — [modelcontextprotocol.io](https://github.com/modelcontextprotocol)

### Official SDKs (10 Languages)

| Language | Repository | Stars (approx) |
|----------|------------|----------------|
| Python | [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) | ⭐ 22.7k |
| TypeScript | [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) | ⭐ 12.2k |
| C# | [modelcontextprotocol/csharp-sdk](https://github.com/modelcontextprotocol/csharp-sdk) | ⭐ 4.2k |
| Rust | [modelcontextprotocol/rust-sdk](https://github.com/modelcontextprotocol/rust-sdk) | — |
| Go | [modelcontextprotocol/go-sdk](https://github.com/modelcontextprotocol/go-sdk) | — |
| Java | [modelcontextprotocol/java-sdk](https://github.com/modelcontextprotocol/java-sdk) | — |
| Kotlin | [modelcontextprotocol/kotlin-sdk](https://github.com/modelcontextprotocol/kotlin-sdk) | — |
| PHP | [modelcontextprotocol/php-sdk](https://github.com/modelcontextprotocol/php-sdk) | — |
| Ruby | [modelcontextprotocol/ruby-sdk](https://github.com/modelcontextprotocol/ruby-sdk) | — |
| Swift | [modelcontextprotocol/swift-sdk](https://github.com/modelcontextprotocol/swift-sdk) | — |

### Official Reference Servers

| Server | Language | Description |
|--------|----------|-------------|
| [Everything](https://github.com/modelcontextprotocol/servers/tree/main/src/everything) | TypeScript | Reference/test server with prompts, resources, and tools |
| [Fetch](https://github.com/modelcontextprotocol/servers/tree/main/src/fetch) | TypeScript | Web content fetching and conversion for LLM usage |
| [Filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem) | TypeScript | Secure file operations with configurable access controls |
| [Git](https://github.com/modelcontextprotocol/servers/tree/main/src/git) | Python | Read, search, and manipulate Git repositories |
| [Memory](https://github.com/modelcontextprotocol/servers/tree/main/src/memory) | TypeScript | Knowledge graph-based persistent memory system |
| [Sequential Thinking](https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking) | Python | Dynamic and reflective problem-solving through thought sequences |
| [Time](https://github.com/modelcontextprotocol/servers/tree/main/src/time) | Python | Time and timezone conversion capabilities |

---

## GitHub Repositories

| Repository | Stars | Forks | Contributors | Description |
|-----------|-------|-------|---------------|-------------|
| [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers) | ⭐ 84.1k | 10.4k | 905 | Official community MCP server registry |
| [punkpeye/awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers) | ⭐ 85.3k | — | — | Curated list of MCP servers |
| [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) | ⭐ 22.7k | — | — | Official Python SDK (includes FastMCP v1) |
| [modelcontextprotocol/typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) | ⭐ 12.2k | — | — | Official TypeScript SDK |

> *"This repository is a collection of reference implementations for the Model Context Protocol (MCP), as well as references to community-built servers."*
> — [modelcontextprotocol/servers](https://github.com/modelcontextprotocol/servers)

---

## Company-Maintained Connectors

### Microsoft (10 Official Servers)

> *"10 Microsoft MCP servers to accelerate your development workflow"* — Jon Galloway, Principal Tech Program Manager, July 16, 2025

| Server | Description |
|--------|-------------|
| Microsoft Learn Docs | Documentation search |
| Azure | Azure resource management |
| GitHub | GitHub API integration |
| Azure DevOps | CI/CD pipeline management |
| MarkItDown | Markdown document conversion |
| SQL Server | Database operations |
| Playwright | Browser automation |
| Dev Box | Cloud development workstations |
| Azure AI Foundry | ML model management |
| Microsoft 365 Agents Toolkit | M365 integration |

— [Microsoft Developer Blog](https://developer.microsoft.com/blog/10-microsoft-mcp-servers-to-accelerate-your-development-workflow)

### AWS Labs Official MCP Servers

> *"Official MCP Servers for AWS"* — AWS Labs

25+ AWS-focused servers including: AWS API, Bedrock, CloudWatch, IAM, Lambda, S3, Secrets Manager, etc.

— [github.com/awslabs/mcp](https://github.com/awslabs/mcp)

### GitHub Official MCP Server

Capabilities: Actions, Pull Requests, Issues, Security, Notifications, Repository Management

— [github.com/github/github-mcp-server](https://github.com/github/github-mcp-server)

---

## Blog Posts & Articles

### MCP Transport Protocols: STDIO vs Streamable HTTP

> *"MCP (Model Context Protocol) replaced its HTTP+SSE transport mechanism with Streamable HTTP starting from protocol version 2026-03-26."*
> — [Bright Data](https://brightdata.com/blog/ai/sse-vs-streamable-http)

### Why MCP Switched from SSE to Streamable HTTP

> *"Key limitation of SSE: 'no support for resumable streams,' 'requires servers to maintain long-lived, highly available connections,' 'only allows server messages via SSE'"*
> — [Bright Data](https://brightdata.com/blog/ai/sse-vs-streamable-http)

### Model Context Protocol (MCP) and Connectors: A Primer

> *"In the same way that a USB provides a standardized way to connect various devices to a computer, the MCP is designed to standardize how AI models connect to and interact with external systems."*
> — [Sourcing Speak](https://www.sourcingspeak.com/model-context-protocol-mcp-connector-basics/)

### MCP Transport Mechanisms: STDIO vs Streamable HTTP

> — [AWS Builder](https://builder.aws.com/content/35A0IphCeLvYzly9Sw40G1dVNzc/mcp-transport-mechanisms-stdio-vs-streamable-http) — Shashi Kiran (AWS), November 7, 2025

### MCP Introduction

> Code examples for Python SDK with `stdio_client` and `StdioServerParameters`.
> — [philschmid.de](https://www.philschmid.de/mcp-introduction)

---

## Debates & Trade-offs

### Pros and Cons of Adopting MCP Today

**Source:** [getknit.dev](https://www.getknit.dev/blog/the-pros-and-cons-of-adopting-mcp-today)

**Pros identified:**
1. Standardization & Reusability: *"Build Once, Connect Many"*
2. Growing open-source ecosystem
3. Dynamic discovery and modularity
4. Real-time, two-way communication
5. Scalability through microservice design
6. Improved AI performance and output quality
7. Enhanced security & governance
8. Faster development cycles
9. Future-proofing through industry alignment

**Cons identified:**
1. Immature standards and evolving tooling
2. Implementation complexity
3. Deployment, monitoring, and scaling overhead
4. Tool invocation reliability issues
5. Security and consent handling gaps

### Why MCP Servers Aren't Enough for Enterprise AI

> *"Three failure modes:*
> 1. **Conflict Blindness**: Multiple sources return contradictory information with equal authority
> 2. **Permission Bleed**: MCP servers authenticate as service accounts, not per-query requesters
> 3. **Shallow Retrieval**: Most MCP servers return surface-level results"*
> — [Unblocked](https://getunblocked.com/blog/why-mcp-servers-arent-enough)

### SSE Deprecation Debate

> *"SSE is effectively legacy now and is basically unworkable for truly distributed MCP anyway. The new world (when the SDKs catch up) will be Streamable HTTP."*
> — [Reddit r/mcp](https://www.reddit.com/r/mcp/comments/1kdyse2/sse_vs_streamable_http_which_will_be_the_standard/)

### Security & Legal Risks

> *"The key security risk associated with MCP connectors is that they generally widen the scope of data accessible to the AI model."*
> *"MCP connections can enable not only read-access to systems and data, but also write-access"*
> — [Sourcing Speak](https://www.sourcingspeak.com/mcp-connectors-legal-operational-risks/)

> *"Diffuse Liability Chains": An MCP-driven workflow might involve an AI model, MCP connector, enterprise data source, and user.*
> — [Sourcing Speak](https://www.sourcingspeak.com/mcp-connectors-legal-operational-risks/)

---

## Key Findings

- **Two official transports**: stdio (local/CLI) and Streamable HTTP (remote/multi-client) — SSE deprecated
- **10 official SDKs** across Python, TypeScript, C#, Rust, Go, Java, Kotlin, PHP, Ruby, Swift
- **84.1k stars** on the official `modelcontextprotocol/servers` repo — massive community adoption
- **Microsoft published 10 official MCP servers** covering Azure, GitHub, SQL Server, Playwright, and more
- **AWS Labs maintains 25+ MCP servers** for Bedrock, Lambda, S3, CloudWatch, IAM, etc.
- **SSE is deprecated** — Streamable HTTP is the replacement (as of 2025-03-26 spec)
- **DNS rebinding vulnerabilities** (CVE-2025-66414/66416) affect older SDK versions — update required
- **MCP solves transport layer only** — not reasoning, conflict resolution, or permission delegation
- **Linux Foundation (via AAIF)** now governs MCP as an open standard

---

## Related Topics

- [FastMCP](./FastMCP.md) — Python framework for building MCP servers
- [Tool-Discovery](./Tool-Discovery.md) — How tools are discovered in the MCP ecosystem
