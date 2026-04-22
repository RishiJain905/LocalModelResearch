# OpenAI-Compatible API Layers

## 📌 Overview

OpenAI-compatible API layers are portability shims that replicate OpenAI's request/response schema and authentication model, allowing existing OpenAI SDKs and ecosystem tools to work with self-hosted or alternative inference backends. This has become a de facto standard in the AI infrastructure space — vLLM, SGLang, TGI, LocalAI, Ollama, and oobabooga all implement these endpoints natively. The ecosystem ranges from zero-config drop-ins to enterprise multi-provider gateways with load balancing and spend tracking.

---

## Definitions & Standards

> "An OpenAI-compatible API is any API that replicates the interface, request/response schema, and authentication model of OpenAI's original API."

— [BentoML LLM Inference Handbook](https://bentoml.com/llm/llm-inference-basics/openai-compatible-api)

> "OpenAI's widespread adoption created ecosystem lock-in. Most developer tools, frameworks, and SDKs are built specifically around the OpenAI schema."

— [BentoML](https://bentoml.com/llm/llm-inference-basics/openai-compatible-api)

> "OpenAI's Chat Completion API has become the industry's gold standard for chat applications. Whether you are working with AI through direct CURL requests and agentic frameworks (LangChain, LlamaIndex) to inference engines (vLLM, SGLang) and proprietary endpoints (OpenAI, Anthropic, AWS Bedrock) — the OpenAI API specification is supported as a first-class citizen."

— Oscar Savolainen (PhD), [Nscale Blog](https://www.nscale.com/blog/has-openai-api-compatibility-become-the-gold-standard)

> "Ollama now has built-in compatibility with the OpenAI Chat Completions API, making it possible to use more tooling and applications with Ollama locally."

— [Ollama Blog](https://ollama.com/blog/openai-compatibility), Feb 8 2024

### Core Endpoints

| Endpoint | Description |
|----------|-------------|
| `POST /v1/chat/completions` | Primary chat completions |
| `POST /v1/completions` | Legacy text completions |
| `GET /v1/models` | List available models |
| `GET /v1/models/{model}` | Retrieve specific model |
| `POST /v1/embeddings` | Text embeddings |
| `POST /v1/audio/transcriptions` | Audio transcription |

---

## Key Sources

### OpenAI Compatibility in Major Inference Engines

> "LocalAI is an API written in Go that serves as an OpenAI shim, enabling software already developed with OpenAI SDKs to seamlessly integrate with LocalAI."

— [LocalAI Architecture](https://localai.io/reference/architecture/), mudler (LocalAI creator)

> "Text Generation Inference (TGI) now supports the Messages API, which is fully compatible with the OpenAI Chat Completion API."

— [Hugging Face TGI — Messages API](https://huggingface.co/docs/text-generation-inference/messages_api)

> "Gemini models are accessible using the OpenAI libraries (Python, TypeScript/JavaScript) and REST API by updating three lines of code."

— [Google Gemini API Docs](https://ai.google.dev/gemini-api/docs/openai)

```python
client = OpenAI(
    api_key="GEMINI_API_KEY",
    base_url="https://generativelanguage.googleapis.com/v1beta/openai/"
)
```

### Hacker News Community Perspectives

> "Right now telling me something is 'OpenAI API compatible' really isn't enough information. Which bits of that API? Which particular date-in-time was it created to match?"

— [simonw](https://news.ycombinator.com/item?id=39307809), Hacker News

> "OpenAI compatible just seems to mean 'you can format your prompt like the `messages` array'."

— [te_chris](https://news.ycombinator.com/item?id=39307809), Hacker News

> "It is weird being beholden to someone else's API which can dictate what features we should (or shouldn't) be adding to our own project."

— [Patrick_Devine](https://news.ycombinator.com/item?id=39307809) (Ollama)

> "There is a difference between a standard and a monopoly, though."

— [dimask](https://news.ycombinator.com/item?id=39307809), Hacker News

### Official OpenAI OpenAPI Specification

**Repository:** [github.com/openai/openai-openapi](https://github.com/openai/openai-openapi)
**Stars:** ~2,400 | **Forks:** ~507

---

## Repositories & Tools

### Inference Servers with Built-in OpenAI-Compatible Endpoints

| Tool | URL | Key Features |
|------|-----|-------------|
| **vLLM** | [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) | PagedAttention; chat/completions, embeddings, audio, realtime API |
| **SGLang** | [github.com/sgl-project/sglang](https://github.com/sgl-project/sglang) | OpenAI-compatible server + native SGLang endpoints |
| **TGI** | [github.com/huggingface/text-generation-inference](https://github.com/huggingface/text-generation-inference) | Messages API from v1.4.0; now in maintenance mode |
| **LocalAI** | [localai.io](https://localai.io) | OpenAI shim in Go using ggml/llama.cpp backends |
| **Ollama** | [ollama.com](https://ollama.com) | Local models at `localhost:11434/v1` |
| **oobabooga/textgen** | [github.com/oobabooga/textgen](https://github.com/oobabooga/textgen) | Drop-in replacement for OpenAI/Anthropic APIs |

### Proxy / Gateway Layers

| Tool | URL | Description |
|------|-----|-------------|
| **LM-Proxy** | [github.com/Nayjest/lm-proxy](https://github.com/Nayjest/lm-proxy) | Multi-provider proxy (Google, Anthropic, OpenAI, PyTorch); MIT License |
| **LiteLLM** | [docs.litellm.ai/docs/simple_proxy](https://docs.litellm.ai/docs/simple_proxy) | Unified interface to 100+ LLMs; load balancing, failover, spend tracking |
| **openai-proxy** | [github.com/fangwentong/openai-proxy](https://github.com/fangwentong/openai-proxy) | Transparent proxy for enterprise monitoring |
| **Lightning AI Proxy** | [lightning.ai](https://lightning.ai/lightning-ai/studios/openai-fault-tolerant-proxy-server) | Fault-tolerant proxy with fallback to Cohere |

### Client Libraries

| Library | Language | Notes |
|---------|----------|-------|
| **openai-oxide** | Rust, Node.js, Python | Feature-complete; streaming, WebSockets, WASM |
| **openai-client-base** | Rust | Auto-generated from OpenAPI spec |
| **Official OpenAI SDKs** | Python, TypeScript, .NET, Java, Go | All support custom base URLs |

---

## Debates & Controversies

### Lack of Formal Specification

No official conformance test suite exists. "OpenAI API compatible" is vague — unclear which features are supported at what version.

### API Instability

> "the sudden deprecation of 'functions' for 'tools' with only minor apparent benefit" — swyx (Hacker News)

OpenAI acknowledged this and committed to continued `functions` support.

### Innovator's Dilemma

Downstream projects (Ollama, etc.) feel constrained by OpenAI's API design decisions. Adding novel features requires either forgoing compatibility or lobbying OpenAI to adopt them.

### Vendor Lock-in Concerns

TechCrunch (Nov 2023): "OpenAI mess exposes the dangers of vendor lock-in for startups." Enterprises increasingly use AI gateways (TrueFoundry, AICC) to maintain abstraction.

---

## Summary Table

### OpenAI-Compatible Endpoint Support by Tool

| Tool | `/v1/chat/completions` | `/v1/completions` | `/v1/embeddings` | Streaming | Tool Calling |
|------|----------------------|-------------------|-----------------|----------|-------------|
| **vLLM** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **SGLang** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **TGI** | ✅ (Messages API) | ✅ | ✅ | ✅ | ✅ |
| **LocalAI** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Ollama** | ✅ | ✅ | ✅ | ✅ | Partial |
| **oobabooga/textgen** | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## Related Topics

- [Request/response protocols](./Request-Response-Protocols.md) — REST, MCP, gRPC, JSON-RPC standards that underpin API layer design
- [Streaming](./Streaming-Real-Time-Delivery.md) — Real-time token delivery over SSE/WebSocket in OpenAI-compatible APIs
- [Parallelism](./Parallelism-Distributed-Serving.md) — Distributed serving infrastructure behind API endpoints
- [Structured Outputs](./Structured-Outputs.md) — Schema-constrained generation via API
