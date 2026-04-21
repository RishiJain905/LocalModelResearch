# LocalAI

> **GitHub:** [mudler/localai](https://github.com/mudler/localai) — 35k stars  
> **Docs:** [localai.io](https://localai.io/)  
> **Model Gallery:** [models.localai.io](https://models.localai.io/)  
> **Paper:** [arXiv:2511.07885](https://arxiv.org/abs/2511.07885) — Measuring Intelligence Efficiency of Local AI

---

## Overview

> *"LocalAI is an open-source AI engine written in Go that acts as a drop-in replacement for the OpenAI API. It enables running LLMs, vision, voice, image, and video models locally on any hardware."* — [LocalAI GitHub](https://github.com/mudler/localai)

**LocalAI** is a **Go-based API-first inference engine** that provides a unified OpenAI-compatible REST API across 36+ backends. Unlike Ollama (CLI-first) or text-generation-webui (GUI-first), LocalAI is **API-first** — designed for production integration with a single endpoint serving text, image, audio, and video.

---

## Supported Backends

| Backend | Formats | Hardware |
|---------|---------|----------|
| **llama.cpp** | GGUF, GGML | CPU, CUDA, ROCm, Intel SYCL, Vulkan, Metal |
| **vLLM** | PyTorch, Safetensors | CUDA 12, ROCm, Intel |
| **Transformers** | PyTorch, Safetensors | CPU, CUDA, ROCm, Intel, Metal |
| **MLX** | Apple Silicon | Metal |
| **whisper.cpp** | Audio STT | CPU, CUDA, ROCm, Metal |
| **diffusers** | Image/video generation | CPU, CUDA, ROCm, Metal |

**Supported formats:** GGUF, GGML, Safetensors, PyTorch, GPTQ, AWQ

---

## API: OpenAI-Compatible REST

> *"Drop-in replacement for OpenAI API — streaming, tools, embeddings, images, audio."* — [LocalAI Docs](https://localai.io/)

| Endpoint | Description |
|---------|-------------|
| `/v1/chat/completions` | Streaming supported |
| `/v1/embeddings` | Text embeddings |
| `/v1/images/generate` | Image generation |
| `/v1/audio/speech` | TTS |
| `/v1/audio/transcriptions` | STT (Whisper) |
| `/v1/audio/vad` | Voice Activity Detection |
| **Tools/Function Calling** | Full OpenAI tools support |
| **Model Context Protocol (MCP)** | For agents |

---

## Multimodal Capabilities

| Modality | Models |
|----------|--------|
| **Text** | GPT-compatible LLMs |
| **Image** | Stable Diffusion, Flux, Diffusers |
| **Audio** | STT (Whisper, Qwen3-ASR), TTS (Coqui, Kokoro, Piper, VibeVoice, fish-speech) |
| **Video** | Video generation |
| **Vision** | SmolVLM, Gemma vision |

---

## Special Features

| Feature | Description |
|---------|-------------|
| **P2P Distributed Inference** | Cluster multiple LocalAI instances |
| **Unified Multimodal API** | Single endpoint for text, image, audio, video |
| **Auto GPU Detection** | Downloads appropriate backend automatically |
| **CPU Support** | Runs on CPU (with acceptable performance) |
| **LocalAGI** | Autonomous agent platform built-in |
| **Realtime WebSocket API** | For voice conversations |
| **Model Gallery** | Auto-download from models.localai.io |
| **Per-user Auth** | API keys, usage tracking |

---

## Comparison: LocalAI vs Ollama vs vLLM

| Feature | LocalAI | Ollama | vLLM |
|---------|---------|--------|------|
| **Focus** | API compatibility, multimodal | CLI simplicity | High-throughput production |
| **Stars** | 35k | 160k | 45k |
| **GUI** | Web UI + REST | Desktop apps | API only |
| **Formats** | GGUF, PyTorch, GPTQ, AWQ, Safetensors | GGUF only | PyTorch, Safetensors (no native GGUF) |
| **Multimodal** | Full (text, image, audio, video, TTS) | Limited | Limited (vision only) |
| **Concurrency** | Multiple backends | Max 4 parallel | 2-4x throughput vs Ollama |
| **P2P/Distributed** | Yes | No | No |
| **Function Calling** | Full OpenAI tools | Basic | Full |
| **CPU-only** | Yes | Yes | No |

> *"Ollama excels for development simplicity; LocalAI is better for production with OpenAI SDK compatibility; vLLM wins for enterprise throughput."* — [Comparison analysis](https://www.secondtalent.com/resources/localai-vs-ollama-differences-use-cases-and-trade-offs/)

---

## vs text-generation-webui

| Aspect | LocalAI | text-generation-webui |
|--------|---------|----------------------|
| **Primary interface** | REST API (OpenAI-compatible) | Web UI (Gradio) |
| **Use case** | Production API serving, integration | Experimentation, parameter tuning |
| **Features** | Multimodal, distributed, agents | LoRA training, fine-tuning, TTS |
| **API** | First-class | Requires extension |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [mudler/localai](https://github.com/mudler/localai) | Main repo — 35k stars, Go |
| [localai.io/models](https://models.localai.io/) | Model gallery |
| [localai.io/backends](https://localai.io/backends/) | Backend gallery |

---

## Documentation

| Resource | URL |
|----------|-----|
| **Main Docs** | [localai.io](https://localai.io/) |
| **Quickstart** | [localai.io/basics/getting_started](https://localai.io/basics/getting_started/) |
| **Model Compatibility** | [localai.io/model-compatibility](https://localai.io/model-compatibility/) |
| **Features** | [localai.io/features](https://localai.io/features/) |
| **Comparison (Second Talent)** | [Link](https://www.secondtalent.com/resources/localai-vs-ollama-differences-use-cases-and-trade-offs/) |
| **Comparison (Glukhov)** | [Link](https://www.glukhov.org/llm-hosting/comparisons/hosting-llms-ollama-localai-jan-lmstudio-vllm-comparison/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Production API serving, multimodal (text+image+audio+video), P2P distributed inference |
| **Key advantage** | 36+ backends, OpenAI SDK drop-in, full multimodal, P2P clustering |
| **Backend** | llama.cpp, vLLM, Transformers, whisper.cpp, diffusers |
| **API** | OpenAI-compatible REST |
| **CPU support** | Yes |

---

*Last updated: 2026-04-21*
