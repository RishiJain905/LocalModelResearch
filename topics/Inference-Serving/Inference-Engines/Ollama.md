# Ollama

> **GitHub:** [ollama/ollama](https://github.com/ollama/ollama)  
> **Website:** [ollama.com](https://ollama.com)  
> **Docs:** [docs.ollama.com](https://docs.ollama.com)

---

## Overview

> *"Ollama is an open-source platform for running large language models locally. It provides a CLI, REST API, and client libraries (Python, JavaScript) to build applications with open models."* — [Ollama GitHub](https://github.com/ollama/ollama)

Ollama is a **local model abstraction layer** — a Go-based server and CLI that wraps llama.cpp for inference, providing an easy-to-use interface for downloading, running, and managing LLMs locally. It abstracts hardware complexity and automatically detects available hardware (NVIDIA CUDA, AMD ROCm, Apple MLX, CPU).

---

## How It Works

| Layer | Technology |
|-------|-----------|
| **CLI/Server** | Go |
| **Inference Engine** | llama.cpp (C/C++) |
| **Model Format** | GGUF |
| **GPU Support** | NVIDIA (CUDA), AMD (ROCm), Apple Silicon (MLX), CPU |

---

## Supported Models

**100+ models** via [ollama.com/library](https://ollama.com/library):

| Category | Examples |
|----------|----------|
| **Text** | Llama 3/3.1/3.2, Mistral, Phi-3, Qwen2, Gemma3, WizardLM2, DeepSeek |
| **Vision** | LLaVA 1.6, Llama 3.2 Vision, BakLLaVA, moondream |
| **Code** | CodeLlama, WizardCoder, Devstral, SQLCoder |
| **Embeddings** | nomic-embed-text, embeddinggemma |

---

## API Endpoints

### REST API (port 11434)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/generate` | POST | Generate text |
| `/api/chat` | POST | Chat completions |
| `/api/embed` | POST | Generate embeddings |
| `/api/tags` | GET | List local models |
| `/api/ps` | GET | List running models |
| `/api/create` | POST | Create custom models |

### OpenAI-Compatible Endpoints

> *"Set base URL to `http://localhost:11434/v1` to use existing OpenAI clients."* — [Ollama API Docs](https://docs.ollama.com/api/openai-compatibility)

```
http://localhost:11434/v1/chat/completions
http://localhost:11434/v1/completions
http://localhost:11434/v1/embeddings
http://localhost:11434/v1/models
```

---

## Quantization Options

Ollama uses GGUF with these quantization levels:

| Format | Description |
|--------|-------------|
| **Q4_K_M** | Default — recommended 4-bit balance |
| Q5_K_M, Q5_K_S | Higher quality 5-bit |
| Q6_K | Near-lossless 6-bit |
| Q8_0 | Near lossless (largest) |
| IQ3_M, Q3_K_M | 3-bit options |

Models default to **Q4_K_M**. Custom quantizations specified via Modelfile.

---

## Modelfiles

> *"Customizable model configurations using FROM, PARAMETER, SYSTEM directives."* — [Ollama Docs](https://docs.ollama.com)

```dockerfile
FROM llama3.2
PARAMETER temperature 0.7
SYSTEM "You are a helpful assistant."
```

### CLI Commands

| Command | Description |
|---------|-------------|
| `ollama run <model>` | Interactive REPL |
| `ollama pull <model>` | Download model |
| `ollama ps` | List running models |
| `ollama create <name>` | Create from Modelfile |
| `ollama list` | List downloaded models |

---

## Comparison: Ollama vs LM Studio vs vLLM

| Feature | Ollama | LM Studio | vLLM |
|---------|--------|-----------|------|
| **Backend** | llama.cpp | llama.cpp + ExLlamaV2 | Python/vLLM |
| **GGUF Native** | ✅ | ✅ | ❌ (needs conversion) |
| **EXL2 Native** | ❌ | ✅ | ❌ |
| **OpenAI API** | ✅ | ✅ | ✅ |
| **Tool Calling** | Limited | Experimental | **Full** (Gold standard) |
| **GUI** | 3rd party only | ✅ Desktop app | ❌ API only |
| **Multi-GPU** | Limited | Limited | ✅ Production-grade |
| **Multi-user/concurrent** | Single-user focused | Single-user focused | **35× throughput** |
| **Open Source** | ✅ | ❌ | ✅ |
| **Best for** | Developers, simplicity | Beginners, low-spec/iGPU | Production throughput |

> *"Ollama uses llama.cpp backend; designed for simplicity; not optimized for multi-user concurrent scenarios."* — [Comparison Analysis](https://www.glukhov.org/llm-hosting/comparisons/hosting-llms-ollama-localai-jan-lmstudio-vllm-comparison/)

---

## Performance Notes

| Aspect | Details |
|--------|---------|
| **Single-user** | Excellent performance with minimal setup |
| **Multi-user/concurrent** | Not optimized — vLLM has 2-4× throughput advantage |
| **Apple Silicon** | MLX backend for efficient inference |
| **Request handling** | Queues requests (single model instance) |

---

## Integrations

- Claude Code, Codex, Copilot CLI
- OpenClaw (WhatsApp, Telegram, Slack, Discord)
- Tool calling with Llama 3.1, Mistral, Qwen2.5

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [ollama/ollama](https://github.com/ollama/ollama) | Main repo — Go server + CLI |
| [ollama/api](https://github.com/ollama/api) | REST API spec |

---

## Documentation

| Resource | URL |
|----------|-----|
| **Main Docs** | [docs.ollama.com](https://docs.ollama.com) |
| **API Reference** | [docs.ollama.com/api](https://docs.ollama.com/api) |
| **OpenAI Compatibility** | [docs.ollama.com/api/openai-compatibility](https://docs.ollama.com/api/openai-compatibility) |
| **CLI Reference** | [docs.ollama.com/cli](https://docs.ollama.com/cli) |
| **GPU Support** | [docs.ollama.com/gpu](https://docs.ollama.com/gpu) |
| **Model Library** | [ollama.com/library](https://ollama.com/library) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Developers, local prototyping, single-user, simplest CLI |
| **Key advantage** | Zero-config, auto hardware detection, OpenAI-compatible API |
| **Backend** | llama.cpp (GGUF) |
| **Open source** | ✅ Yes |
| **Multi-user?** | ❌ Not optimized — use vLLM for production |

---

*Last updated: 2026-04-21*
