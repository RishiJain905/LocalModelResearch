# Streaming & Real-Time Token Delivery

## 📌 Overview

Streaming and real-time token delivery refers to the techniques used to transmit LLM output tokens to clients as they are generated, rather than waiting for full completion. The dominant standards are **Server-Sent Events (SSE)** for unidirectional server-push and **WebSockets** for bidirectional communication. Key metrics include **Time to First Token (TTFT)** — the most user-perceptible latency — and **Time Per Output Token (TPOT)**. Modern inference engines (vLLM, SGLang, TensorRT-LLM) implement continuous batching and chunked prefill to optimize streaming performance. Research shows vLLM achieves up to 24× higher throughput than TGI at high concurrency, and Andes (a QoE-aware serving system) improves user-perceived streaming quality by up to 3.2× over vLLM.

---

## Definitions & Standards

### Server-Sent Events (SSE)

> "Server-Sent Events (SSE) is a server push technology enabling a client to receive automatic updates from a server via an HTTP connection. It describes how servers can initiate data transmission toward clients once an initial client connection has been established."

— [Wikipedia: Server-sent events](https://en.wikipedia.org/wiki/Server-sent_events)

**Standard:** The EventSource API is standardized as part of the HTML Living Standard by the WHATWG. W3C published Server-Sent Events as a Recommendation on February 3, 2015.
**Media Type:** `text/event-stream`

**Event format:**
```
data: {"token":"HTTP"}
data: {"token":" streaming"}
data: [DONE]
```

**Fields:** `data`, `event`, `id`, `retry`

### WebSocket

> "The WebSocket Protocol enables two-way communication between a client running untrusted code in a controlled environment to a remote host that has opted-in to communications from that code."

— [IETF RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455), December 2011

### Time to First Token (TTFT)

> "That delay between pressing send and seeing anything happen is what Time to First Token (TTFT) measures... It's one of the most visible metrics for production LLM apps because it directly shapes perceived responsiveness."

— [Redis Blog](https://redis.io/blog/ttft-meaning/)

**TTFT comprises three parts:** Network latency, queue wait time, and prefill phase (model computes attention across all input tokens).

**Human perception thresholds:**
- <100ms: feels instantaneous
- 100ms–1s: noticeable delay, doesn't interrupt flow
- >1s: users must actively wait
- >10s: users typically disengage

— [TianPan.co](https://tianpan.co/blog/2026-04-16-streaming-ttft-latency-perception)

### LLM Streaming Phases

**Prefill:** The model processes the entire input prompt to produce the first new token. Computationally heavy but fills the KV cache.

**Decode:** Each subsequent token is generated one-by-one, reading everything that came before. Thanks to the KV cache, this is much faster than prefill.

— [Hugging Face: Continuous Batching from First Principles](https://huggingface.co/blog/continuous_batching)

---

## Key Sources

### Andes — QoE-Aware LLM Streaming

> "We propose Andes, a QoE-aware LLM serving system that enhances user experience by ensuring that users receive the first token promptly and subsequent tokens at a pace aligned with their consumption speed."

> "Compared to vLLM, Andes improves average QoE by up to **3.2×** under high request rate, or handles up to **1.6×** higher request rate while preserving high QoE."

— [arXiv:2404.16283](https://arxiv.org/html/2404.16283v1)

### Eloquent — Robust Token Streaming

> "Eloquent adds unacked tokens, defined as tokens whose packets have been sent but have not been ACK'ed by the receiver, into the packet of the newly generated token."

> "Eloquent reduces stall ratio by **71.0%** compared to TCP retransmission and by **31.6%** compared to packet duplication under unstable network conditions."

— [arXiv:2401.12961](https://arxiv.org/html/2401.12961v2)

### Comparative: vLLM vs TGI

> "vLLM achieves up to **24× higher throughput** than TGI under high-concurrency workloads through its PagedAttention mechanism, while TGI demonstrates lower tail latencies for interactive single-user scenarios."

— [arXiv:2511.17593v1](https://arxiv.org/html/2511.17593v1)

### TTFT vs Throughput Paradox

> "Your model generates a 500-word response in 8 seconds. A competing model generates the same response in 12 seconds. Intuitively, yours should feel faster. But if your first token arrives at 2.5 seconds and theirs arrives at 400 milliseconds, your users will describe your product as slow — regardless of total generation time."

— [TianPan.co](https://tianpan.co/blog/2026-04-16-streaming-ttft-latency-perception)

---

## Repositories & Tools

### LLM Inference Engines

| Tool | URL | Streaming Notes |
|------|-----|----------------|
| **vLLM** | [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) | PagedAttention, continuous batching, chunked prefill |
| **SGLang** | [github.com/sgl-project/sglang](https://github.com/sgl-project/sglang) | RadixAttention for KV cache prefix sharing; streaming optimized |
| **TGI** | [github.com/huggingface/text-generation-inference](https://github.com/huggingface/text-generation-inference) | Now in maintenance mode |
| **NVIDIA TensorRT-LLM** | [developer.nvidia.com/tensorrt-llm](https://developer.nvidia.com/tensorrt-llm) | KV cache reuse; priority-based eviction |
| **LMDeploy** | OpenMMLab | C++-native, high throughput on H100 |

### Benchmarking Tools

| Tool | Description |
|------|-------------|
| **LLM Locust** | Open-source benchmarking built on Locust framework for streaming token-level LLM evaluation |
| **AIPerf** | Comprehensive benchmarking for generative AI model performance |

---

## Debates & Controversies

### SSE vs WebSocket for LLM Streaming

**Use SSE when:** Unidirectional server-to-client streaming is sufficient; cross-browser compatibility and simplicity matter; existing HTTP infrastructure should work without modification.

> "SSE provides only one-way communication — events can only be sent from the server to the client. In theory (and practice), all the things that can be done with SSE can also be done with WebSockets."

— [SoftwareMill](https://softwaremill.com/sse-vs-websockets-comparing-real-time-communication-protocols/)

**SSE limitation:** ~6 connections per domain under HTTP/1.1 — removed under HTTP/2 and HTTP/3.

**Use WebSockets when:** Bidirectional communication is required (interactive agents with tool use); very low latency (<100ms) is critical; high-frequency client-to-server messaging is needed.

### TTFT vs Throughput: Which Metric Matters More?

> "Track TTFT, TPS, and end-to-end latency separately; alert on p95; design for p99."

— [TianPan.co](https://tianpan.co/blog/2026-04-16-streaming-ttft-latency-perception)

For interactive chat/copilots, TTFT is paramount. For batch pipelines, throughput dominates.

### Resumable Streaming as UX Requirement

> "Refresh the page, lose signal, switch tabs — the AI conversation just keeps going. That's what reliable, resumable token streaming makes possible."

— [Ably Realtime](https://ably.com/blog/token-streaming-for-ai-ux)

Requires significant infrastructure investment (server-side buffering, session tracking, ordered exactly-once delivery).

---

## Summary Table

### Protocol Comparison

| Feature | SSE | WebSocket | HTTP Chunked |
|---|---|---|---|
| **Direction** | Server → Client only | Bidirectional | Server → Client |
| **Protocol** | HTTP/1.1+ | TCP-based (RFC 6455) | HTTP/1.1 |
| **Browser Support** | Native EventSource API | All modern browsers | Universal |
| **Auto-reconnection** | Yes (Last-Event-ID) | Manual | No |
| **Complexity** | Low | Medium | Low |
| **Connection limit** | ~6 per domain (HTTP/1.1) | Much higher | Per connection |

### LLM Serving Benchmark Summary (2025–2026)

| Engine | Throughput (H100, Llama 3.1 8B) | Best For |
|---|---|---|
| **vLLM** | ~12,500 tok/s | High-concurrency batch workloads |
| **SGLang** | ~16,200 tok/s | Long-context workloads; structured outputs |
| **TensorRT-LLM** | Best single-request latency | Production-grade latency optimization |
| **TGI v3** | Lower but memory-efficient | Long prompts (200K+ tokens): 13× faster than vLLM |

### Key Metrics

| Metric | Definition | Target for Interactive Chat |
|---|---|---|
| **TTFT** | Time to first token | < 1 second (p95) |
| **TPS** | Tokens per second | Varies by model |
| **TPOT** | Time per output token | Application-dependent |
| **p95/p99 TTFT** | Percentile TTFT | Monitor separately; p99 often 10× p50 |

---

## Related Topics

- [OpenAI-Compatible-API-Layers](./OpenAI-Compatible-API-Layers.md) — Streaming endpoints in OpenAI-compatible APIs
- [Parallelism-Distributed-Serving](./Parallelism-Distributed-Serving.md) — Continuous batching and KV cache management underlying streaming
- [Request-Response-Protocols](./Request-Response-Protocols.md) — HTTP transport protocols for streaming
