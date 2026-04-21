# LM Studio

> **Website:** [lmstudio.ai](https://lmstudio.ai/)  
> **GitHub:** [lmstudio-ai](https://github.com/lmstudio-ai)  
> **Docs:** [lmstudio.ai/docs](https://lmstudio.ai/docs/developer)  
> **⚠️ Note:** LM Studio is **proprietary** (not open source).

---

## Overview

> *"LM Studio is a desktop application for running LLMs locally on your own hardware. It provides a GUI + CLI for discovering, downloading, and running local AI models with an OpenAI-compatible API server."* — [LM Studio](https://lmstudio.ai/)

LM Studio is a **user-friendly local LLM runner** with a desktop GUI and OpenAI-compatible API, supporting GGUF (via llama.cpp), EXL2 (via ExLlamaV2), and Apple MLX. Designed for beginners and low-spec hardware with in-app HuggingFace model search.

---

## Supported Backends

| Backend | Hardware | Format |
|---------|----------|--------|
| **llama.cpp** | CPU, NVIDIA, AMD | GGUF |
| **ExLlamaV2** | NVIDIA GPU | EXL2 |
| **Apple MLX** | Apple M-series | MLX |
| **Vulkan** | Intel iGPU, AMD iGPU | Cross-vendor |

---

## Quantization Support

| Format | Backend | Notes |
|--------|---------|-------|
| **GGUF** | llama.cpp | Widest hardware support |
| **EXL2** | ExLlamaV2 | Best quality/speed on NVIDIA |
| **AWQ** | Conversion | Via model conversion |
| **MLX** | Apple MLX | Apple Silicon optimized |

---

## OpenAI-Compatible API

> *"Set base URL to `http://localhost:1234/v1` to use existing OpenAI clients."* — [LM Studio Docs](https://lmstudio.ai/docs/developer/openai-compat)

```
http://localhost:1234/v1/chat/completions
http://localhost:1234/v1/responses
http://localhost:1234/v1/completions
http://localhost:1234/v1/embeddings
http://localhost:1234/v1/models
```

---

## SDKs

| Language | Package | Install |
|----------|---------|---------|
| **TypeScript** | `@lmstudio/sdk` | `npm install @lmstudio/sdk` |
| **Python** | `lmstudio` | `pip install lmstudio` |

---

## HuggingFace Integration

> *"In-app model search and download (⌘+Shift+M on Mac, Ctrl+Shift+M on Windows). 'Use this model' button on HuggingFace model pages."* — [LM Studio](https://huggingface.co/docs/hub/lmstudio)

```bash
# CLI download
lms get <model>
```

Community quantized models: [huggingface.co/lmstudio-community/models](https://huggingface.co/lmstudio-community/models)

---

## LM Link — Remote Instances

> *"Connect to remote LM Studio instances as if they were local (end-to-end encrypted)."* — [LM Studio](https://lmstudio.ai/)

---

## Headless Deploy — llmster

> *"Headless daemon for server/cloud deployments."* — [LM Studio](https://lmstudio.ai/blog/0.4.0#deploy-on-servers-deploy-in-ci-deploy-anywhere)

```bash
curl -fsSL https://lmstudio.ai/install.sh | bash
```

---

## Comparison: LM Studio vs Ollama vs vLLM

| Feature | LM Studio | Ollama | vLLM |
|---------|-----------|--------|------|
| **Backend** | llama.cpp + ExLlamaV2 | llama.cpp | Python/vLLM |
| **GGUF Native** | ✅ | ✅ | ❌ (needs conversion) |
| **EXL2 Native** | ✅ | ❌ | ❌ |
| **OpenAI API** | ✅ | ✅ | ✅ |
| **Tool Calling** | ⚠️ Experimental | ❌ Limited | ✅ Full |
| **GUI** | ✅ Desktop app | 3rd party only | ❌ API only |
| **Multi-GPU** | Limited | Limited | ✅ Production-grade |
| **Open Source** | ❌ | ✅ | ✅ |
| **Best for** | Beginners, iGPU/low-spec | Developers, simplicity | Production throughput |

---

## Performance Benchmarks

> *"Gemma 3 on M3 Ultra: LM Studio with MLX engine achieved 26-30% more tokens/sec vs Ollama."* — [Medium](https://medium.com/google-cloud/gemma-3-performance-tokens-per-second-in-lm-studio-vs-ollama-mac-studio-m3-ultra-7e1af75438e4)

| Scenario | LM Studio | Ollama | Notes |
|----------|-----------|--------|-------|
| **Single-user (M3 Ultra)** | 26-30% faster | Baseline | Gemma 3 benchmark |
| **iGPU (Ryzen AI 9)** | 27% faster | Competitors | Best iGPU performance |
| **Multi-user/concurrent** | Limited | Limited | vLLM dominates |

---

## Unique Features

| Feature | Description |
|---------|-------------|
| **Desktop GUI** | User-friendly interface for non-technical users |
| **In-app HF search** | Browse and download HuggingFace models directly |
| **EXL2 support** | Best quality/speed on NVIDIA consumer GPUs |
| **Vulkan** | Intel/AMD integrated GPU support |
| **MLX engine** | Apple Silicon optimized |
| **llmster** | Headless server/cloud deployment |
| **MCP client** | Model Context Protocol for agents |
| **LM Link** | E2E encrypted remote instance connection |

---

## GitHub Repos

| Repo | Stars | Description |
|------|-------|-------------|
| [lmstudio-ai/lms](https://github.com/lmstudio-ai/lms) | 4.7k | CLI tool |
| [lmstudio-ai/lmstudio-js](https://github.com/lmstudio-ai/lmstudio-js) | 1.6k | TypeScript SDK |
| [lmstudio-ai/lmstudio-python](https://github.com/lmstudio-ai/lmstudio-python) | 794 | Python SDK |
| [lmstudio-ai/mlx-engine](https://github.com/lmstudio-ai/mlx-engine) | 1k | Apple MLX support |

---

## Documentation

| Resource | URL |
|----------|-----|
| **Developer Docs** | [lmstudio.ai/docs/developer](https://lmstudio.ai/docs/developer) |
| **OpenAI API** | [lmstudio.ai/docs/developer/openai-compat](https://lmstudio.ai/docs/developer/openai-compat) |
| **CLI (lms)** | [lmstudio.ai/docs/cli](https://lmstudio.ai/docs/cli) |
| **Python SDK** | [lmstudio.ai/docs/python](https://lmstudio.ai/docs/python) |
| **TypeScript SDK** | [lmstudio.ai/docs/sdk](https://lmstudio.ai/docs/sdk) |
| **HuggingFace Integration** | [huggingface.co/docs/hub/lmstudio](https://huggingface.co/docs/hub/lmstudio) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Beginners, low-spec hardware, integrated GPUs, desktop GUI users |
| **Key advantage** | EXL2 support (best NVIDIA quality/speed), in-app HuggingFace search, desktop GUI |
| **Backend** | llama.cpp + ExLlamaV2 + MLX |
| **Open source** | ❌ Proprietary (free for home/work use) |
| **Multi-user?** | ❌ Not optimized — use vLLM for production |

---

*Last updated: 2026-04-21*
