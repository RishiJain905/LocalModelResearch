# LibreChat

> *A self-hosted, open-source AI chat platform that unifies multiple AI providers in a single, privacy-focused interface.*

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Multi-Provider Support](#multi-provider-support)
4. [Key Features](#key-features)
5. [Comparison with Alternatives](#comparison-with-alternatives)
6. [Deployment](#deployment)
7. [When to Use](#when-to-use)
8. [References](#references)

---

## Overview

> *"LibreChat is a self-hosted AI chat platform that unifies all major AI providers in a single, privacy-focused interface."*
> — [librechat.ai](https://www.librechat.ai/)

LibreChat is an enhanced ChatGPT clone with multi-provider support, AI agents, code interpreter, and enterprise-ready authentication.

**Key Metrics:**
| Metric | Value |
|--------|-------|
| GitHub Stars | **36.1k** |
| Forks | **7.4k** |
| Contributors | **375+** |
| Docker Pulls | **26.9M** |
| License | **MIT** |

**Core Capabilities:**
- Self-hosted deployment with full data control
- Multi-provider support (OpenAI, Anthropic, Azure, AWS Bedrock, Google Vertex, Mistral, Groq, and more)
- Feature parity with ChatGPT plus additional capabilities
- Enterprise authentication (OAuth2, SAML, LDAP, social logins)

---

## Architecture

LibreChat uses a **provider abstraction layer** that normalizes disparate AI APIs into a unified interface:

| Component | Technology |
|-----------|------------|
| **Backend** | Node.js/TypeScript (api/ server) |
| **Frontend** | React |
| **Database** | MongoDB |
| **Search** | Meilisearch |
| **Cache** | Redis (for horizontal scaling) |
| **Vector Store** | PGVector (for RAG) |
| **RAG Pipeline** | LangChain + FastAPI |

### Key Architectural Features

1. **Endpoint Handlers**: Custom handlers for each provider that translate LibreChat's internal message format to provider-specific schemas
2. **Resumable Streams**: Backend stores partial responses in Redis; if WebSocket drops mid-stream, client reconnects and requests continuation from last token index
3. **Code Interpreter**: Spawns isolated execution environments (NOT in Node.js process) for Python, Node.js, Go, C/C++, Java, PHP, Rust, Fortran
4. **Multi-Tab & Multi-Device Sync**: Conversations persist across devices via Redis-backed state

> *"Rather than building yet another single-provider clone, it created an abstraction layer that treats AI providers as swappable backends."*
> — [Starlog](https://starlog.is/articles/ai-agents/danny-avila-librechat)

---

## Multi-Provider Support

### First-Class Integrations

| Provider | Models Supported |
|----------|-----------------|
| **OpenAI** | GPT-4, GPT-4o, o1, o3, GPT-4 Turbo, GPT-3.5 Turbo |
| **Anthropic** | Claude Opus 4.7 (1M context), Claude 3.5, Claude 3 |
| **Azure OpenAI** | GPT-4, GPT-4o, DALL-E 3, all Azure deployments |
| **AWS Bedrock** | Claude on Bedrock, Llama, Mistral |
| **Google Vertex AI** | Gemini models |
| **Mistral AI** | Mistral, Mixtral |
| **Groq** | Fast inference models |
| **DeepSeek** | DeepSeek-R1 with reasoning UI |

### Custom Endpoints (OpenAI-Compatible)

Connect to any OpenAI-compatible API without a proxy:

- Ollama, Groq, Cohere, Mistral AI, Apple MLX, Koboldcpp
- Together.ai, OpenRouter, Helicone, Perplexity, ShuttleAI
- DeepSeek, Qwen, and more

> *"Use any OpenAI-compatible API with LibreChat, no proxy required"*
> — [GitHub README](https://github.com/danny-avila/LibreChat)

---

## Key Features

### 🤖 AI Agents
- No-code custom assistants with tools, file handling, code execution, and API actions
- Agent Marketplace for community-built agents
- MCP (Model Context Protocol) native support
- File Context for grounding agents on documents

### 💻 Code Interpreter
- Secure, sandboxed execution in Python, Node.js, Go, C/C++, Java, PHP, Rust, Fortran
- Seamless file handling (upload, process, download)
- Fully isolated execution environment

### 🪄 Code Artifacts
- Create React components, HTML pages, and Mermaid diagrams directly in chat
- Renders output in a side panel
- Supports shadcn/ui component instructions

### 🌐 Model Context Protocol (MCP)
- Connect AI models to any external tool or service
- Open standard for AI tool integration
- Built-in MCP servers for common tools

### 🔍 Search & Knowledge
- Meilisearch-powered message search
- RAG API for document-based Q&A
- Web search with content scrapers and rerankers
- OCR for extracting text from images

### 🎨 Image Generation
- GPT-Image-1 (OpenAI - recommended)
- DALL-E 3/2
- Stable Diffusion (local)
- Flux (MCP server support)

### 💾 Presets & Context Management
- Create, save, share custom presets
- Switch endpoints mid-chat
- Fork messages & conversations
- Conversation branching

### 🔐 Security & Access
- Multi-user authentication: OAuth2, SAML, LDAP, Email
- Role-based access control
- Built-in moderation tools
- Token spend tracking

---

## Comparison with Alternatives

### LibreChat vs Open WebUI

| Aspect | LibreChat | Open WebUI |
|--------|-----------|------------|
| **Architecture** | Custom endpoints | Pipeline architecture |
| **Auth** | OAuth2, Azure AD, SAML, LDAP | Basic RBAC |
| **RAG** | LangChain + PGVector | Local/remote sources |
| **Ollama** | Via custom endpoints | Native integration |
| **Strength** | Enterprise auth, multi-provider | Simplicity, container-native |
| **Deployment** | Docker, npm, Kubernetes, Railway | Docker, Podman, Kubernetes |

> *"LibreChat excels for teams who want the polish of ChatGPT's UI, multi-modal features, and robust authentication options."*
> — [Requesty Blog](https://www.requesty.ai/blog/openwebui-vs-librechat-which-self-hosted-chatgpt-ui-is-right-for-you)

### LibreChat vs LobeChat vs Open WebUI

| Feature | LobeChat | Open WebUI | LibreChat |
|---------|----------|------------|-----------|
| **UI Polish** | Excellent | Good | Good |
| **Ollama Support** | Yes | Native | Via custom endpoints |
| **OpenAI Support** | Yes | Yes | Native |
| **RAG** | Yes | Yes | Plugins |
| **Plugin System** | 100+ | Limited | ChatGPT-compatible |
| **Multi-user** | Yes | Yes | Yes |
| **Docker Deploy** | Easy | Easiest | Easy |

> *"The ChatGPT interface—once OpenAI's moat—is now table stakes. Any of these three projects delivers 90% of the ChatGPT experience."*
> — [Elestio Blog](https://blog.elest.io/the-best-open-source-chatgpt-interfaces-lobechat-vs-open-webui-vs-librechat/)

---

## Deployment

### Quick Start

```bash
git clone https://github.com/danny-avila/LibreChat
cd LibreChat
cp .env.example .env  # Configure your API keys
docker-compose up -d
```

### Full Stack (Docker Compose)
MongoDB, Meilisearch, RAG API, and Vector DB are included automatically.

### Remote Server Deployment
Ubuntu Docker deployment guide available at [docs.librechat.ai](https://www.librechat.ai/docs/remote/docker_linux) with Nginx, SSL, and reverse proxy configuration.

### Cloud Deployments
- **Railway**: 1-click deploy with pre-wired MongoDB, Meilisearch, PGVector
- **DigitalOcean**: VM-based deployment
- **Azure**: Terraform infrastructure-as-code
- **Kubernetes**: Helm charts available

### Configuration

**Environment Variables (.env)**
Primary configuration file for most deployments. Configure:
- MongoDB, Redis connection
- API keys for providers
- Authentication settings

**librechat.yaml**
Custom AI endpoints, model settings, interface options, MCP servers, and agents:

```yaml
endpoints:
  custom:
    - name: "My Ollama"
      baseURL: "http://localhost:11434/v1"
      apiKey: "not-needed"
      models: ["llama3"]
```

---

## Version History

### Recent Releases

| Version | Date | Key Changes |
|---------|------|-------------|
| **v0.8.5** | April 2026 | Claude Opus 4.7, security fixes, MongoDB 8.0.20 |
| **v0.8.4** | March 2026 | Security improvements, accessibility fixes |
| **v0.8.3** | February 2026 | Stability improvements |
| **v0.7.8** | Late 2025 | MCP enhancements, OpenAI Web Search models |
| **v0.7.0** | 2024 | Breaking changes, Assistants API, Agents |

### Roadmap

**2025:**
- Agentic Tooling for all providers via LibreChat Agents
- Admin Panel (separate CMS app for usage, chats, users, settings)
- OpenAI-compatible API for LibreChat Agents
- Sources UI for conversation file/image visibility
- Agent Marketplace UI

**2026:**
- Admin Panel for GUI-based configuration
- Agent Skills and Programmatic Tool Calling
- Interactive Q&A workflows

---

## When to Use

### ✅ Use LibreChat when:
- Building internal AI tooling requiring multi-provider flexibility
- Need complete control over data residency and privacy
- Require code execution and agent capabilities
- Have DevOps resources to manage self-hosted infrastructure
- Need to consolidate AI spending across providers into single interface
- Want enterprise authentication (OAuth2, SAML, LDAP)

### ❌ Consider alternatives when:
- You want simplest local Ollama setup → **Open WebUI**
- Design/UX polish is priority → **LobeChat**
- You need workflow builders (RAG pipelines, multi-step agents) → **Dify + chat interface**
- No infrastructure management capacity → **Managed services**

> *"Use LibreChat if you're building internal AI tooling for an organization that needs multi-provider flexibility, want complete control over data residency and privacy, or require advanced features like code execution and agent marketplaces."*
> — [Starlog](https://starlog.is/articles/ai-agents/danny-avila-librechat)

---

## References

### Official Resources
- [GitHub Repository](https://github.com/danny-avila/LibreChat) — 36.1k stars, MIT
- [Website](https://www.librechat.ai/)
- [Documentation](https://www.librechat.ai/docs)
- [Changelog](https://www.librechat.ai/changelog)

### Architecture Deep-Dive
- [Starlog: LibreChat: Building a Multi-Provider AI Gateway](https://starlog.is/articles/ai-agents/danny-avila-librechat) — Best architecture overview

### Comparison Articles
- [Portkey.ai: LibreChat vs Open WebUI](https://portkey.ai/blog/librechat-vs-openwebui) — Best vs Open WebUI
- [Elestio: LobeChat vs Open WebUI vs LibreChat](https://blog.elest.io/the-best-open-source-chatgpt-interfaces-lobechat-vs-open-webui-vs-librechat/) — Best 3-way comparison

### Deployment Guides
- [TomOehlrich.com: Deploying LibreChat](https://www.tomoehlrich.com/blog/deploying-libre-chat-on-hetzner-or-any-cloud-provider/) — Cloud deployment
- [hleroy.com: Multi-provider deployment with Traefik](https://www.hleroy.com/2024/11/deploying-librechat-as-a-single-webui-for-multiple-llms-providers/)

### Real-World Usage
- [Stanford AI Playground](https://www.firecrawl.dev/blog/customer-story-stanford) — Built on LibreChat for Stanford University (10,000+ domains)

### Video Resources
- [Official YouTube Channel](https://www.youtube.com/@LibreChat) — Tutorials and feature demos
- [LibreChat v0.7.6: Code Interpreter, AI Agents, MCP](https://www.youtube.com/watch?v=ilfwGQtJNlI)

---

*Part of [[web-uis]] — Interface Comparison*
