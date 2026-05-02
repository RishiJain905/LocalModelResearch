# Open WebUI

> *An open, extensible, and usable interface for AI interaction — self-hosted, privacy-focused, with native Ollama integration.*

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Key Features](#key-features)
4. [Ollama Integration](#ollama-integration)
5. [Extensibility System](#extensibility-system)
6. [Comparison with Alternatives](#comparison-with-alternatives)
7. [Deployment](#deployment)
8. [When to Use](#when-to-use)
9. [References](#references)

---

## Overview

> *"Open WebUI is an extensible, feature-rich, and user-friendly self-hosted AI platform designed to operate entirely offline."*
> — [Official Documentation](https://docs.openwebui.com/)

Open WebUI (formerly Ollama WebUI) is a privacy-focused, self-hosted AI chat platform that supports multiple LLM providers with an emphasis on local deployment and data control.

**Key Metrics:**
| Metric | Value |
|--------|-------|
| GitHub Stars | **135k+** |
| Forks | **19k+** |
| Package Downloads | **13M+** |
| Contributors | **320+** |
| Community Functions | **541** |
| Community Tools | **276** |

---

## Architecture

### Technology Stack

| Layer | Technology |
|-------|------------|
| **Backend** | Python (FastAPI) |
| **Frontend** | SvelteKit |
| **Database** | SQLite (default), PostgreSQL (production) |
| **Vector DB** | ChromaDB, PGVector, Qdrant, Milvus, Elasticsearch, OpenSearch, Pinecone, S3Vector, Oracle 23ai |
| **Session Management** | Redis (multi-instance/WebSocket coordination) |
| **Container** | Docker, Kubernetes |

### Design Philosophy

> *"Open WebUI is architected from the ground up to support enterprise-scale deployments where reliability isn't optional."*
> — [Enterprise Architecture Docs](https://docs.openwebui.com/enterprise/architecture/)

**Key Principles:**
- **Stateless design** — horizontal scaling without rebuild
- **Container-first** — deploy anywhere (Docker, Kubernetes, bare metal)
- **Modular extensibility** — plugin system for custom logic
- **Multi-instance support** — Redis-backed session management

### High Availability Configuration

| Component | Capability |
|-----------|------------|
| **Load Balancing** | Multiple container instances behind a load balancer |
| **External Databases** | PostgreSQL (required for multi-instance) |
| **External Vector DB** | Client-server vector database (PGVector, Milvus, Qdrant) |
| **Redis** | Session management, WebSocket coordination, config sync |
| **Persistent Storage** | S3/GCS/Azure Blob Storage backends |
| **Observability** | OpenTelemetry integration |

---

## Key Features

### User Experience
- **Responsive design** — Desktop, Laptop, and Mobile support
- **PWA support** — Native app-like experience with offline access
- **Full Markdown & LaTeX support**
- **Internationalization (i18n)** — Multilingual UI
- **ChatGPT-like interface** — Familiar UX for quick adoption

### Model Management
- **Multi-model support** — Ollama, OpenAI, Claude, Mistral, Gemini, local models
- **Model whitelisting** — Control which models users can access
- **Auto-download** — Pull models directly from UI
- **Custom models** — Custom system prompts, files, tools
- **Model comparison** — Run same prompt across multiple models in parallel

### Chat & Conversations
- **Editable responses** — Edit both user messages AND LLM responses (great for few-shot learning)
- **RLHF dataset building** — Upvotes/downvotes stored locally
- **Output formatting** — Markdown, LaTeX, code blocks, Mermaid diagrams
- **Shared chats** — Share conversations with others
- **Memory** — Persistent conversation context

### RAG (Retrieval Augmented Generation)
- **Document upload** — PDF, TXT, DOCX, and more
- **Document extraction** — Built-in parsing
- **Web search integration** — 15+ providers (SearXNG, Google PSE, Brave Search, Tavily, Perplexity)
- **YouTube transcription** — RAG over video content
- **Multi-vector database support** — ChromaDB, Qdrant, Milvus, etc.

### Advanced Features

| Feature | Description |
|---------|-------------|
| **Voice/Video Calls** | Hands-free with STT (Whisper, Deepgram, Azure) and TTS (ElevenLabs, Azure) |
| **Image Generation** | DALL-E, ComfyUI, AUTOMATIC1111 |
| **Code Execution** | Pyodide (Python in browser) |
| **Model Builder** | Create Ollama models via Web UI |
| **Webhooks** | External system integration |
| **Message Queue** | Async message processing |

---

## Ollama Integration

Open WebUI was originally built as **Ollama WebUI** and maintains deep native integration with Ollama:

```bash
# Connect to local Ollama
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 \
  --name open-webui \
  ghcr.io/open-webui/open-webui:main

# Bundled Ollama (single container)
docker run -d -p 3000:8080 --gpus all \
  -v ollama:/root/.ollama \
  -v open-webui:/app/backend/data \
  --name open-webui \
  ghcr.io/open-webui/open-webui:ollama
```

**Key Ollama-specific features:**
- Native model management and downloading
- Model comparison (parallel inference)
- Ollama-specific parameter tuning
- Support for reasoning models

---

## Extensibility System

Open WebUI offers a powerful **two-pronged plugin architecture**:

### Three Extensibility Layers

| Layer | Runs Where | Best For | Trade-off |
|-------|-----------|----------|-----------|
| **Tools & Functions** | Inside Open WebUI process | Real-time data, filters, UI actions, new providers | Shares resources with main server |
| **OpenAPI / MCP** | Any HTTP endpoint | Connecting existing services, third-party APIs | Requires running external server |
| **Pipelines** | Separate Docker container | GPU workloads, heavy dependencies, sandboxed execution | Additional infrastructure to manage |

### Plugin Types

1. **Tools** (Model Level)
   - Execute Python scripts to extend LLM capabilities
   - Enable real-time data fetching, computations, external system interfacing

2. **Functions** (Interface Level)
   - **Filters**: Pre/post-processing conversational data (translation, PII scrubbing)
   - **Pipes**: Abstract Python functions as "models" — enables integration of non-standard models
   - **Actions**: Custom buttons in response toolbar for user-triggered tasks

3. **Pipelines**
   - Run on separate worker for heavy or sensitive processing
   - GPU access, large dependencies, sandboxed execution

### Community Plugins (Top Downloads)

| Plugin | Downloads | Type |
|--------|-----------|------|
| Enhanced Web Scrape | 16K | Tool |
| Visualize Data | 13K | Action |
| Mixture of Agents | 9.2K | Action |
| Add to Memories | 9.1K | Action |
| Stock Reporter | 7.6K | Tool |
| Run Code | 6.2K | Tool |
| Google Translate | 4.1K | Filter |
| SQL Server Access | 2.9K | Tool |

> *"Make Open WebUI do anything with Python, HTTP, or community plugins you install in one click."*
> — [Extensibility Docs](https://docs.openwebui.com/features/extensibility/)

---

## Comparison with Alternatives

### Open WebUI vs Alternatives

| Criterion | Open WebUI | AnythingLLM | Text Generation WebUI | Chatbot UI |
|-----------|------------|-------------|----------------------|------------|
| **License** | MIT | MIT | Apache 2.0 | AGPL |
| **Architecture** | FastAPI + SvelteKit | Node.js + React | Python + Gradio | Next.js + Supabase |
| **Primary Users** | Developers, Self-Hosters | Business Users, Teams | Researchers, Power Users | Developers |
| **Ollama Integration** | Native (best) | Good | Good | Requires setup |
| **RAG Support** | Advanced (9 vector DBs) | Good (5 vector DBs) | Basic | Basic |
| **Plugin Ecosystem** | Extensive | Limited | Moderate | Limited |
| **Multi-User** | Yes (basic) | Yes (workspaces/roles) | Limited | Via Supabase |
| **Enterprise Features** | Yes (RBAC, LDAP, SCIM) | Moderate | No | No |
| **Community** | 135k+ stars | 25k+ stars | 30k+ stars | 15k+ stars |

### Academic Validation

> *"Three design goals: **Open** (open-source, privacy, transparency), **Extensible** (plugin support, multi-model), **Usable** (one-command install, familiar UI)"*
> — [arXiv:2510.02546](https://arxiv.org/abs/2510.02546)

From user survey (20 respondents, 10 countries):
- 4.4/5 LLM familiarity
- 3.8/5 installation ease
- Top adoption reasons: multi-model support (17/20), open-source privacy (16/20), active community (13/20)

---

## Deployment

### Installation Methods

| Method | Use Case | Command |
|--------|----------|---------|
| **Docker (Recommended)** | General use | `docker run -d -p 3000:8080 -v open-webui:/app/backend/data --name open-webui ghcr.io/open-webui/open-webui:main` |
| **Docker + GPU** | ML workloads | Use `ghcr.io/open-webui/open-webui:cuda` image |
| **Bundled Ollama** | Quick start | Use `ghcr.io/open-webui/open-webui:ollama` image |
| **pip** | Python-native | `pip install open-webui && open-webui serve` |
| **uv** | Modern pip alternative | `uvx open-webui@latest serve` |
| **Kubernetes** | Production/Enterprise | See [Kubernetes docs](https://docs.openwebui.com/getting-started/quick-start/#kubernetes) |

### Production Deployment Checklist

1. **Use PostgreSQL** instead of SQLite for multi-instance
2. **Configure Redis** for session management and WebSocket coordination
3. **Use external vector database** (PGVector, Qdrant, Milvus) instead of local ChromaDB
4. **Enable authentication** (OAuth2/OIDC recommended)
5. **Configure LDAP/SCIM** for enterprise user management
6. **Set up OpenTelemetry** for observability
7. **Use load balancer** for multi-container resilience

---

## When to Use

### ✅ Best for:
- Developer teams with Ollama focus (native integration)
- Tech-savvy organizations needing high customization
- RAG-heavy workflows with complex pipelines
- Plugin-driven extensions (web scraping, code execution, custom APIs)
- Open source first strategy with community contribution
- Enterprise deployments requiring RBAC, LDAP, SCIM 2.0

### ❌ Consider alternatives when:
- Business users without IT background → **AnythingLLM** (desktop app, simpler UX)
- LoRA training/finetuning needed → **Text Generation WebUI**
- Heavy workspace management with roles → **AnythingLLM**
- Quick start without Docker knowledge → **AnythingLLM** (desktop version)

---

## References

### Official Resources
- [GitHub Repository](https://github.com/open-webui/open-webui) — 135k stars
- [Official Documentation](https://docs.openwebui.com/)
- [Architecture & HA Guide](https://docs.openwebui.com/enterprise/architecture/)
- [Extensibility Documentation](https://docs.openwebui.com/features/extensibility/)

### Academic Papers
1. ["Open WebUI: An Open, Extensible, and Usable Interface for AI Interaction"](https://arxiv.org/abs/2510.02546) — arXiv:2510.02546
2. ["Designing an open-source LLM interface and social platforms for evaluation"](https://openwebui.com/assets/files/whitepaper.pdf) — Whitepaper

### Comparison Articles
- [Open WebUI vs AnythingLLM: The detailed comparison](https://wz-it.com/en/blog/open-webui-vs-anythingllm-comparison/)
- [Open WebUI vs LibreChat vs LobeChat](https://blog.elest.io/the-best-open-source-chatgpt-interfaces-lobechat-vs-open-webui-vs-librechat/)

### Video Resources
- [Open WebUI + Ollama Is AMAZING For AI Self-Hosting](https://www.youtube.com/watch?v=mdMj2YGMcNo) — Tutorial
- [Getting Started with Ollama and Web UI](https://www.youtube.com/watch?v=BzFafshQkWw) — Beginner guide
- [Open WebUI Mastering Series](https://www.youtube.com/playlist?list=PLP50mZI6LSxOXDGGdbX1a3iOc7SvL9g9A) — 50+ video playlist

---

*Part of [[web-uis]] — Interface Comparison*
