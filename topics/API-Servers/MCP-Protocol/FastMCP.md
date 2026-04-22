# FastMCP

> *"The fast, Pythonic way to build MCP (Model Context Protocol) servers, clients, and applications."* — [gofastmcp.com](https://gofastmcp.com/getting-started/welcome)

## Contents

1. [Definitions & Overview](#definitions--overview)
2. [Key Papers](#key-papers)
3. [Open-Source Libraries & Tools](#open-source-libraries--tools)
4. [GitHub Repositories](#github-repositories)
5. [Blog Posts & Articles](#blog-posts--articles)
6. [Performance Benchmarks](#performance-benchmarks)
7. [Debates & Trade-offs](#debates--trade-offs)
8. [Key Findings](#key-findings)

---

## Definitions & Overview

**FastMCP** is the standard framework for building MCP applications — defined as:

> *"FastMCP is the standard framework for building MCP applications. The Model Context Protocol (MCP) connects LLMs to tools and data. FastMCP gives you everything you need to go from prototype to production — build servers that expose capabilities, connect clients to any MCP service, and give your tools interactive UIs."*
> — [gofastmcp.com](https://gofastmcp.com/getting-started/welcome)

**Key Metric:**
> *"Today, the actively maintained standalone project is downloaded a million times a day, and some version of FastMCP powers 70% of MCP servers across all languages."*
> — [gofastmcp.com](https://gofastmcp.com/getting-started/welcome)

**FastMCP v1.0 was incorporated into the official MCP Python SDK in 2024:**
> *"FastMCP 1.0 has been integrated into the official MCP Python SDK."*
> — [en.kelen.cc/posts/fastmcp](https://en.kelen.cc/posts/fastmcp)

### Version History

| Version | Key Feature |
|---------|-------------|
| **v1.0** | Incorporated into official MCP Python SDK |
| **v2.0** | Client infrastructure, server composition, OpenAPI/FastAPI generation |
| **v3.0** | Provider/transform architecture, CodeMode, MultiAuth |
| **v3.2.4** | Current stable release |

### Provider Architecture (v3+)

> *"FileSystemProvider, OpenAPIProvider, ProxyProvider, SkillsProvider"*
> — [gofastmcp.com/updates](https://gofastmcp.com/updates)

### Transforms System (v3+)

> *"Middleware for components (namespace, rename, filter by version, control visibility)"*
> — [gofastmcp.com/updates](https://gofastmcp.com/updates)

> *"Surface API unchanged — `@mcp.tool()` still works as before. Underlying architecture rebuilt with provider/transform system."*
> — [gofastmcp.com/updates](https://gofastmcp.com/updates)

---

## Key Papers

### FastMCP: A Simplified Framework for Model Context Protocol (2025)

> *"FastMCP addresses this complexity by abstracting protocol-level details, enabling developers to focus on core functionality rather than implementation specifics. This post demonstrates the utility of FastMCP through a practical medical imaging application, presenting a chest X-ray classification and segmentation server implementation."*
> — Emily Xie, MICCAI 2025 Challenge MEC Submission

— [arXiv/OpenReview](https://openreview.net/forum?id=TDfLx1w1Cb) | [DOI](OpenReview)

---

## Open-Source Libraries & Tools

### FastMCP (main)

> [gofastmcp.com](https://gofastmcp.com) | `uv pip install fastmcp`

The primary FastMCP framework, maintained by PrefectHQ. Provides declarative tool registration via decorators, automatic schema generation, validation, and documentation.

### Prefect Horizon

> [prefect.io/horizon](https://www.prefect.io/horizon)

Free hosting platform for FastMCP servers.

### FastMCP CLI

> [gofastmcp.com/servers/cli](https://gofastmcp.com/servers/cli)

Commands: `fastmcp run`, `fastmcp dev`, `fastmcp install`

> *"We recommend installing FastMCP with uv."*
> — [PyPI](https://pypi.org/project/fastmcp/)

---

## GitHub Repositories

| Repository | Stars | License | Description |
|-----------|-------|---------|-------------|
| [prefecthq/fastmcp](https://github.com/prefecthq/fastmcp) | ⭐ 23.5k | Apache 2.0 | Main FastMCP framework by PrefectHQ |
| [fastmcp-me/fastmcp-python](https://github.com/fastmcp-me/fastmcp-python) | — | — | Community-driven Python implementation |
| [modelcontextprotocol/python-sdk](https://github.com/modelcontextprotocol/python-sdk) | ⭐ 22.7k | — | Official MCP Python SDK (contains FastMCP v1) |

From the PrefectHQ README:

> *"Building an effective MCP application is harder than it looks. FastMCP handles all of it. Declare a tool with a Python function, and the schema, validation, and documentation are generated automatically."*
> — [prefecthq/fastmcp](https://github.com/prefecthq/fastmcp)

---

## Blog Posts & Articles

### FastMCP — A Better Framework for Building MCP Than the Official SDK

> *"FastMCP 2 has 'nicer' APIs (from UX perspective). It has documentation. The guy who developed it will launch a SaaS cloud to host MCP servers."*
> — [en.kelen.cc/posts/fastmcp](https://en.kelen.cc/posts/fastmcp)

### Comparing MCP Server Frameworks: Which One Should You Choose?

> Compares FastMCP against other frameworks from a development efficiency and simplicity standpoint.
> — [Medium / Divyansh Bhatia](https://medium.com/@divyanshbhatiajm19/comparing-mcp-server-frameworks-which-one-should-you-choose-cbadab4ddc80) — April 10, 2025

### In Depth: Gram vs FastMCP

> *"FastMCP is an open-source Python framework for building MCP servers, and FastMCP Cloud is the managed hosting platform for deploying FastMCP-based applications."*
> *"Tools defined with FastMCP are tightly coupled to the MCP protocol. The framework is designed as a lower-level abstraction that closely mirrors the MCP specification."*
> — [Speakeasy](https://www.speakeasy.com/blog/gram-vs-fastmcp-cloud) — November 11, 2025

### How to Build Your First MCP Server using FastMCP

> Tutorial walkthrough for creating a FastMCP server from scratch.
> — [freeCodeCamp](https://www.freecodecamp.org/news/how-to-build-your-first-mcp-server-using-fastmcp/)

### Building an MCP Server with FastMCP: A Hands-On Demo

> — [Medium / ale.garavaglia](https://medium.com/@ale.garavaglia/building-an-mcp-server-with-fastmcp-a-hands-on-demo-98208c5370df)

### FastMCP with Adam Azzam and Jeremiah Lowin

> Podcast interview with the creator of FastMCP.
> — [Software Engineering Daily](https://softwareengineeringdaily.com/2026/04/07/fastmcp-with-adam-azzam-and-jeremiah-lowin/) — April 7, 2026

---

## Performance Benchmarks

### MCP Server Performance Benchmark v2

> *"Four MCP servers were built Java (Spring Boot + Spring AI), Go (official SDK), Node.js, and Python (FastMCP) each containerized with identical..."*
> — [tmdevlab.com](https://www.tmdevlab.com/mcp-server-performance-benchmark-v2.html)

**Python (FastMCP) Performance Tier 4:**
| Server | RPS | Avg Latency | P95 Latency | RAM |
|--------|-----|-------------|-------------|-----|
| **bun** | 876 | 48.46 ms | 98.50 ms | 541 MB |
| **nodejs** | 423 | 123.50 ms | 200.07 ms | 389 MB |
| **python** | 259 | 251.62 ms | 342.41 ms | 259 MB |

> *"Python (FastMCP): 259 RPS, 251.62 ms avg latency, 342.41 ms P95 latency"*
> *"optimized Python (259 RPS with 4 workers and uvloop)"*
> — [tmdevlab.com](https://www.tmdevlab.com/mcp-server-performance-benchmark-v2.html)

**Overall Benchmark Rankings:**

| Rank | Server | RPS | Avg Latency | RAM |
|------|--------|-----|-------------|-----|
| 1 | Rust | 4,845 | 5.09 ms | 10.9 MB |
| 2 | Quarkus | 4,739 | 4.04 ms | 194.5 MB |
| 3 | Go | 3,616 | 6.87 ms | 23.9 MB |
| 4 | Java MVC | 3,540 | 6.13 ms | 368.1 MB |
| ... | ... | ... | ... | ... |
| 15 | **Python (FastMCP)** | **259** | **251.62 ms** | **259 MB** |

---

## Debates & Trade-offs

### FastMCP vs Official MCP Python SDK

Comparison from [en.kelen.cc/posts/fastmcp](https://en.kelen.cc/posts/fastmcp):

| Feature | Official SDK | FastMCP |
|---------|:------------:|:-------:|
| API Simplicity | Moderate | ⭐⭐⭐⭐⭐ |
| Development Efficiency | Medium | ⭐⭐⭐⭐⭐ |
| Extensibility | Basic | Rich |
| Client Usability | Moderate | Extremely Simple |
| Documentation & Community | Moderate | Comprehensive |
| Transport Methods | Limited | SSE, Stdio, Memory, etc. |
| FastAPI Integration | No | ✅ Supported |

### Performance Trade-off

> *"Pattern: 27-81% memory reduction at cost of 20-36% throughput regression."*
> — [Medium / kanishks772](https://medium.com/@kanishks772/python-is-93-slower-the-mcp-benchmark-that-shocked-developers-7e1c5be6604e)

> *"Python is 93% slower" in raw throughput, but achieves lower memory usage with optimization."*
> — [Medium](https://medium.com/@kanishks772/python-is-93-slower-the-mcp-benchmark-that-shocked-developers-7e1c5be6604e)

### FastMCP vs Gram

From [Speakeasy](https://www.speakeasy.com/blog/gram-vs-fastmcp-cloud):

> *"FastMCP is open-source and fully self-hostable. Gram provides a managed platform with proprietary extensions."*

---

## Key Findings

- **FastMCP powers ~70% of all MCP servers** across languages despite being Python-only, via its influence on the official SDK
- **1 million PyPI downloads/day** — massively adopted by the MCP ecosystem
- **v1.0 merged into official MCP Python SDK** — FastMCP is now the de facto standard approach in the official SDK
- **v3 introduced Provider/Transform architecture** — decouples tool sources (FileSystem, OpenAPI, Proxy, Skills) from MCP protocol
- **Python is the slowest language** for MCP servers (259 RPS vs Rust's 4,845 RPS), but uses less memory than Java/Node in some configs
- **Creator Jeremiah Lowin** (PrefectHQ) maintains tight control; FastMCP Cloud is being launched as a SaaS hosting layer
- **FastMCP vs Gram debate**: Gram is a managed alternative; FastMCP is open-source with deeper protocol alignment

---

## Related Topics

- [Standard-Connectors](./Standard-Connectors.md) — Official transport mechanisms (stdio, Streamable HTTP)
- [Tool-Discovery](./Tool-Discovery.md) — How tools are discovered in the MCP ecosystem
