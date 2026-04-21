# text-generation-webui (oobabooga)

> **GitHub:** [oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui) (now `textgen`)  
> **Docs:** [github.com/oobabooga/text-generation-webui/wiki](https://github.com/oobabooga/text-generation-webui/wiki)  
> **Quantization comparison:** [oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/](https://oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/)

---

## Overview

> *"The original local LLM interface. Text, vision, tool-calling, training. UI + API, 100% offline and private."* — [GitHub](https://github.com/oobabooga/text-generation-webui)

**text-generation-webui** (aka oobabooga) is a **Gradio-based web UI** designed to be "the AUTOMATIC1111/stable-diffusion-webui of text generation." It is a **research and experimentation tool** — not a production server — offering maximum flexibility with multiple inference backends in a single UI.

---

## Supported Backends & Formats

| Backend | Format | Notes |
|---------|--------|-------|
| **Transformers** | FP16 / full precision | Standard HuggingFace models |
| **ExLlamaV2** | GPTQ, EXL2 | Fast GPU inference |
| **AutoGPTQ** | GPTQ 4-bit | Popular quantization |
| **AutoAWQ** | AWQ 4-bit | Activation-aware weight quantization |
| **llama.cpp** | GGUF | Most portable, CPU + GPU |
| **ExLlamaV3** | Latest | Multimodal/vision support |

> *"EXL2 offers best perplexity/VRAM tradeoff on LLaMA-2-13B."* — [oobabooga blog](https://oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/)

---

## Extensions Framework

> *"Extensions are defined by script.py files in extensions/ or user_data/extensions/ directories."* — [Wiki](https://github.com/oobabooga/text-generation-webui/wiki/07-%E2%80%90-Extensions)

**Built-in Extensions:**

| Extension | Function |
|-----------|----------|
| `superboogav2` | RAG with PDF/DOCX/PPTX |
| `send_pictures` | Image upload with BLIP captioning |
| `coqui_tts` / `silero_tts` | Text-to-speech |
| `whisper_stt` | Speech-to-text input |
| `sd_api_pictures` | Stable Diffusion integration |
| `google_translate` | Auto-translation |
| `perplexity_colors` | Token probability visualization |
| `ngrok` | Remote access |

**External extensions:** [oobabooga/text-generation-webui-extensions](https://github.com/oobabooga/text-generation-webui-extensions)

---

## Unique Features

| Feature | Description |
|---------|-------------|
| **Tool Calling (v4.1+)** | Built-in JSON output parsing; custom tools as .py files; MCP server support |
| **Native LoRA Training** | Fine-tune LoRA adapters directly in the UI |
| **Multimodal/Vision** | Image inputs via llama.cpp and ExLlamaV3 |
| **Image Generation** | Load image models, generate via API |
| **Character System** | Create/manage chat characters with bias controls |
| **Multiple Interface Modes** | Chat, Default (two-column), Notebook |

---

## API Server Mode

> *"Drop-in replacement for OpenAI API — Chat, Completions, Embeddings endpoints."* — [Docs](https://github.com/oobabooga/text-generation-webui/blob/main/docs/12%20-%20OpenAI%20API.md)

```bash
python server.py --extensions openai --api-key <key>
```

- Port 5000 by default
- SSE streaming supported
- Tool/function calling over API
- Image inputs (multimodal)
- Authentication via --api-key

---

## Comparison

### vs vLLM and Ollama

| Feature | text-gen-webui | Ollama | vLLM |
|---------|---------------|--------|------|
| **Focus** | Research, flexibility | Simplicity | Production throughput |
| **GUI** | Native Gradio | 3rd party | API only |
| **Formats** | All (EXL2, AWQ, GPTQ, GGUF) | GGUF only | PyTorch/Safetensors |
| **Training** | Native LoRA | No | No |
| **Multi-user** | No | No | Yes 35x throughput |
| **Best for** | Experimentation, fine-tuning | Local dev, simplicity | Production serving |

---

## Quantization Performance (LLaMA-2-13B, RTX 3090)

| Format | Perplexity | VRAM | Speed |
|--------|-----------|------|-------|
| **EXL2** | Best | Moderate | Fastest |
| **AWQ** | Competitive | Higher | Fast |
| **GPTQ (ExLlamaV2)** | Good | Moderate | Fast |
| **llama.cpp Q4_K_M** | Good | Lower | Moderate |

> *"EXL2 dominates llama.cpp Q4_K_M in both perplexity and size."* — [oobabooga blog](https://oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/)

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui) | Main repo — Gradio UI + multiple backends |
| [oobabooga/text-generation-webui-extensions](https://github.com/oobabooga/text-generation-webui-extensions) | Community extensions |

---

## Documentation

| Resource | URL |
|----------|-----|
| **GitHub Wiki** | [github.com/oobabooga/text-generation-webui/wiki](https://github.com/oobabooga/text-generation-webui/wiki) |
| **Tool Calling Tutorial** | [Wiki: Tool Calling](https://github.com/oobabooga/text-generation-webui/wiki/Tool-Calling-Tutorial) |
| **Extensions** | [Wiki: Extensions](https://github.com/oobabooga/text-generation-webui/wiki/07-%E2%80%90-Extensions) |
| **OpenAI API** | [docs/12 - OpenAI API.md](https://github.com/oobabooga/text-generation-webui/blob/main/docs/12%20-%20OpenAI%20API.md) |
| **Quantization Comparison** | [oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/](https://oobabooga.github.io/blog/posts/gptq-awq-exl2-llamacpp/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | Research, experimentation, fine-tuning workflows, maximum format flexibility |
| **Key advantage** | All backends in one UI, native LoRA training, extensions ecosystem |
| **Backend** | llama.cpp, ExLlamaV2/V3, AutoGPTQ, AutoAWQ, Transformers |
| **GUI** | Native Gradio |
| **Production?** | No — research tool |

---

*Last updated: 2026-04-21*
