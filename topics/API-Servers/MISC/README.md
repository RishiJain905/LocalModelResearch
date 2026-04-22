# MISC (API-Servers)

## 📌 Overview

MISC covers cross-cutting infrastructure topics for LLM API servers: OpenAI-compatible API layers (the de facto portability standard), request/response protocol design (REST, MCP, gRPC, JSON-RPC), streaming and real-time token delivery (SSE, WebSocket, TTFT), parallelism and distributed serving (tensor/pipeline parallelism, disaggregated prefill-decode), and structured outputs via schema-constrained generation (FSM, XGrammar, Guidance).

---

## Subtopics

| Subtopic | Description | Key Metric |
|----------|-------------|-----------|
| [OpenAI-Compatible-API-Layers](./OpenAI-Compatible-API-Layers.md) | Portability shims replicating OpenAI's schema/auth — vLLM, SGLang, Ollama, LiteLLM | All major engines implement it |
| [Request-Response-Protocols](./Request-Response-Protocols.md) | REST, MCP (JSON-RPC 2.0), gRPC, A2A — endpoint design, RFC 9457 errors | 88.6% of MCP servers REST-backed |
| [Streaming-Real-Time-Delivery](./Streaming-Real-Time-Delivery.md) | SSE, WebSocket, TTFT optimization, Andes, Eloquent | vLLM 24× throughput vs TGI |
| [Parallelism-Distributed-Serving](./Parallelism-Distributed-Serving.md) | PagedAttention, DistServe (7.4× goodput), Dynamo (30× on Blackwell) | 2–4× vLLM vs FasterTransformer |
| [Structured-Outputs](./Structured-Outputs.md) | Constrained decoding, XGrammar, Guidance, JSONSchemaBench (10K schemas) | Guidance 6–9ms TPOT; best framework 2× schema coverage |

---

## Summary Matrix

| | OpenAI-API-Layers | Req/Res Protocols | Streaming | Parallelism | Structured Outputs |
|---|---|---|---|---|---|
| **Layer** | API surface | Protocol | Transport | Infrastructure | Generation control |
| **Key Standard** | OpenAI schema | MCP/JSON-RPC 2.0 | SSE/WebSocket | PagedAttention | FSM/PDA |
| **Key Tool** | vLLM, LiteLLM | MCP, A2A | Andes, Eloquent | DistServe, Dynamo | Guidance, XGrammar |
| **Key Metric** | Universal compatibility | 88.6% REST-backed | 24× vLLM vs TGI | 7.4× DistServe goodput | 2× schema coverage gap |

---

## Key Interconnections

```
OpenAI-API-Layers (portability)
    ↑ serves over
Streaming (SSE/WebSocket)
    ↑ powered by
Parallelism/Distributed-Serving (KV cache, batching)
    ↑ enables
Structured-Outputs (constrained decoding in inference engine)
    ↑
Request-Response-Protocols (MCP tool calls → structured output)
```

- **OpenAI-API-Layers** expose **Streaming** and **Structured-Outputs** endpoints via `/v1/chat/completions`
- **Streaming** performance depends on **Parallelism** (continuous batching, KV cache management)
- **Structured-Outputs** (XGrammar in vLLM) is part of the **Parallelism** serving stack
- **Request-Response-Protocols** (MCP tool calling) is the primary use case for **Structured-Outputs**

---

## Related Topics

- [Async-Frameworks](../Async-Frameworks/README.md) — FastAPI, Pydantic, ASGI — the async stack these MISC topics run on
- [MCP-Protocol](../MCP-Protocol/README.md) — MCP as the emerging LLM-tool integration protocol
- [API-Servers parent](../README.md) — Back to API-Servers overview
