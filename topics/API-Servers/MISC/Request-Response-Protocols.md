# Request/Response Protocols & Endpoint Design

## 📌 Overview

LLM API servers implement a variety of request/response protocols — from REST (dominant for external APIs) to MCP (emerging standard for LLM tool integration) to gRPC (internal microservices) to JSON-RPC 2.0 (used by MCP as its transport). Key design concerns include error response standardization (RFC 9457), pagination strategies, idempotency, versioning, and the growing adoption of MCP servers as LLM-facing API endpoints. Research shows 88.6% of MCP servers are REST-backed, but expose only 19% of available REST operations.

---

## Definitions & Standards

### REST (Representational State Transfer)

> "When you're designing a REST API, you should not use verbs in the endpoint paths."

— [freeCodeCamp — REST API Best Practices](https://www.freecodecamp.org/news/rest-api-best-practices-rest-endpoint-design-examples/)

> "Well designed APIs make it easy for consumer developers to find, explore, access, and use them."

— [Google Cloud — RESTful Web API Design Best Practices](https://cloud.google.com/blog/products/api-management/restful-web-api-design-best-practices)

> "REST APIs use Uniform Resource Identifiers (URIs) to address resources. REST API designers should create URIs that convey a REST API's resource model to the potential clients of the API. When resources are named well, an API is intuitive and easy to use."

— [restfulapi.net](https://restfulapi.net/resource-naming/)

### JSON-RPC 2.0 Specification

> "JSON-RPC is a stateless, light-weight remote procedure call (RPC) protocol... It uses JSON as data format. It is designed to be simple!"

— [JSON-RPC 2.0 Specification](https://www.jsonrpc.org/specification), JSON-RPC Working Group, updated 2013-01-04

### Model Context Protocol (MCP)

> "Model Context Protocol (MCP) is an open protocol that enables seamless integration between LLM applications and external data sources and tools."

> "MCP provides a standardized way for applications to: Share contextual information with language models, Expose tools and capabilities to AI systems, Build composable integrations and workflows."

> "Communication: Uses JSON-RPC 2.0 messages over stateful connections."

— [MCP Specification](https://modelcontextprotocol.io/specification/2025-06-18)

### gRPC (Google Remote Procedure Call)

> "gRPC is based on the Remote Procedure Call (RPC) model, in which the addressable entities are procedures, and the data is hidden behind the procedures. HTTP works the inverse way. In HTTP, the addressable entities are 'data entities' (called 'resources' in the HTTP specifications), and the behaviors are hidden behind the data."

— [Google Cloud Blog](https://cloud.google.com/blog/products/api-management/understanding-grpc-openapi-and-rest-and-when-to-use-them), Martin Nally, Apigee

### RFC 9457 — Problem Details for HTTP APIs

> "This document defines a 'problem detail' to carry machine-readable details of errors in HTTP response content to avoid the need to define new error response formats for HTTP APIs."

— [IETF RFC 9457](https://datatracker.ietf.org/doc/html/rfc9457)

> "RFC 9457 aims to perfect the art of delivering bad news, ensuring API errors are not just communicated but are done so in a way that negates assumptions, aids in quick diagnostics, and facilitates smoother interactions between machines."

— [Swagger Blog](https://swagger.io/blog/problem-details-rfc9457-doing-api-errors-well/)

---

## Key Sources

### Empirical Study: MCP Servers & REST

> "The Model Context Protocol (MCP) is emerging as a standard interface through which LLM agents invoke external tools, and a growing ecosystem of MCP servers now mediates access to vendor services."

> "88.6% of MCP servers are fully or partially REST-backed"

> "92% implement tools as bare API wrappers"

> "MCP servers expose a **median of 19%** of available REST operations"

> "Read-only and listing operations are preferentially exposed (lift 1.45). Destructive, deprecated, and complex operations are systematically omitted."

— [arXiv:2507.16044v4 — "From REST to MCP: An Empirical Study"](https://arxiv.org/html/2507.16044v4)

**AutoMCP tool:** Automatically generates MCP servers from OpenAPI specs with **94.2%** success rate after repair.

### A2A Protocol (Agent2Agent)

> "The A2A Protocol is an open-source agent communication standard initiated by Google and supported by over 50 top technology partners."

— [A2A Protocol Specification](https://a2agent.net/protocol-spec)

> "The Agent2Agent (A2A) protocol is a communication protocol for AI agents, initially introduced by Google in April 2025. This open protocol is designed for multi-agent systems, allowing interoperability between AI agents from varied providers."

— [IBM Think](https://www.ibm.com/think/topics/agent2agent-protocol)

Message types: `REQUEST`, `RESPONSE`, `EVENT`, `HEARTBEAT`

### Idempotency

> "A REST API is called idempotent when making multiple identical requests to a REST API has the same effect as making a single request."

> "The idempotent REST APIs help in enabling predictable behavior, even in the face of network failures and retries."

— [restfulapi.net](https://restfulapi.net/idempotent-rest-apis/)

### Pagination: Cursor vs Offset

> "Offset pagination scans and discards rows... When you write OFFSET 99980, the database does not magically jump to row 99,981. Every row before the offset is read and discarded."

— [Design Gurus — API Pagination Guide](https://designgurus.substack.com/p/api-pagination-guide-cursor-vs-offset)

---

## Repositories & Tools

| Repository | Description | URL |
|---|---|---|
| **psenger/Best-Practices-For-Rest-API** | Pragmatic REST API best practices — security, resilience, GraphQL, WebSockets, SSE | [GitHub](https://github.com/psenger/Best-Practices-For-Rest-API) |
| **packtpublishing/Hands-On-RESTful-API-Design-Patterns** | Code repository for Packt book on REST API design patterns | [GitHub](https://github.com/packtpublishing/hands-on-restful-api-design-patterns-and-best-practices) |
| **Aavache/fastapi-designs** | FastAPI design patterns and building blocks | [GitHub](https://github.com/Aavache/fastapi-designs) |
| **Kong** | Open-source API gateway built on NGINX | [konghq.com](https://konghq.com) |
| **Solo.io Gloo Gateway** | AI-native API gateway for MCP servers | [solo.io](https://solo.io) |

---

## Debates & Controversies

### REST vs gRPC

gRPC uses POST for all methods and hides HTTP details. Resolution: REST for external-facing APIs, gRPC for internal service-to-service communication.

### HATEOAS Adoption

> "Only a small minority of APIs are designed this way, even though the term 'REST' is widely (and often incorrectly) used."

— [Google Cloud Blog](https://cloud.google.com/blog/products/api-management/understanding-grpc-openapi-and-rest-and-when-to-use-them)

### API Versioning Strategies

| Strategy | Pros | Cons |
|---|---|---|
| **URL Path** (`/api/v1/users`) | Explicit, easy to route | URL bloat |
| **Header** (`Stripe-Version: 2024-12-18`) | Clean URLs | Less visible |
| **Date-based** (Stripe approach) | Pin to specific version | Complex migration |

---

## Summary Table

### Protocol Comparison

| Protocol | Transport | Format | Best For | Key Limitation |
|---|---|---|---|---|
| **REST** | HTTP | JSON/XML | Public APIs, browser clients | Over-fetching, multiple roundtrips |
| **gRPC** | HTTP/2 | Protocol Buffers | Internal microservices | Not browser-native, complex setup |
| **JSON-RPC** | Any | JSON | Simple RPC, tool calling | No batch operations natively |
| **MCP** | stdio/HTTP+SSE | JSON-RPC 2.0 | LLM tool integration | New, evolving standard |
| **A2A** | HTTP/WebSocket | JSON | Multi-agent orchestration | Early stage (April 2025) |

### HTTP Method Idempotency

| Method | Idempotent? | Notes |
|---|---|---|
| GET, PUT, DELETE, HEAD, OPTIONS, TRACE | ✅ Yes | Read-only or overwrites |
| POST, PATCH | ❌ No | Creates new resources |

### Error Response Structure (RFC 9457)

```json
{
  "type": "https://example.net/validation-error",
  "title": "Form validation failed",
  "status": 400,
  "detail": "One or more fields have validation errors.",
  "instance": "/log/registration/12345"
}
```

---

## Related Topics

- [OpenAI-Compatible-API-Layers](./OpenAI-Compatible-API-Layers.md) — Implementation of these protocols in practice
- [Streaming-Real-Time-Delivery](./Streaming-Real-Time-Delivery.md) — SSE and WebSocket streaming over HTTP
- [MCP-Protocol](../MCP-Protocol/README.md) — MCP as the emerging LLM tool protocol
