# Emergent

## 📌 Overview

> **Emergent** is an AI-powered vibe-coding platform that uses **5 coordinated AI agents** (Architect, Designer, Developer, Integration, PM) to design, code, test, and deploy production-ready full-stack applications from natural language. Each session runs in a full Linux Kubernetes pod.

> *"Build production-ready apps through conversation. Chat with AI agents that design, code, and deploy your application from start to finish."*
> — [emergent.sh](https://emergent.sh)

---

## 📊 Key Metrics

| Metric | Value |
|--------|-------|
| **ARR** | $50M+ (at 7 months) |
| **Users** | 3M+ worldwide |
| **Funding** | $100M total raised |
| **Valuation** | $300M (post-Series B) |
| **Employees** | 75 (70 in Bengaluru, 5 in SF) |
| **YC Batch** | S24 (Summer 2024) |
| **SWE-bench** | Claims #1 (internal agent system) |

---

## 🛠️ Open Source

**GitHub:** [github.com/emergentai](https://github.com/emergentai)

> *"We're a Toronto-based 🇨🇦 AI lab dedicated to building practical AI solutions that help businesses thrive."*

Note: The main Emergent product is proprietary. Limited public repositories.

---

## 🤖 The 5-Agent Architecture

```
User Prompt → Architect → Designer → Developer → Integration → PM (QA)
```

| Agent | Role |
|-------|------|
| **Architect** | Technical lead — analyzes prompts, creates blueprint (data models, API contracts, component hierarchy). Asks clarifying questions when underspecified. |
| **Designer** | UI/UX layer — layout, typography, colors, spacing, responsive breakpoints. Context-aware (fintech ≠ gaming ≠ medical). |
| **Developer** | Writes actual code — React components, FastAPI routes, MongoDB queries, auth logic, middleware. |
| **Integration** | External connections — Stripe, Slack, Airtable, GitHub. Handles OAuth, API keys, SDK config. |
| **PM** | Coordinates workflow, validates outputs, runs quality checks before deployment. |

---

## ⚙️ Generated Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | React + Next.js (SSR, static generation, component architecture) |
| Backend | FastAPI (Python) — async, auto OpenAPI docs |
| Database | MongoDB — NoSQL document model |
| Auth | Built-in registration, login, sessions, permissions |
| Infrastructure | Google Container Registry + Cloud Hosting |

**Key distinction:** Code is portable to any hosting environment. No proprietary runtime lock-in.

---

## ☸️ Kubernetes Architecture

**Source:** [emergent.sh/blog — Real Environments for AI Agents](https://emergent.sh/blog)

**Core Thesis:**
> *"The output is production-grade because the agent operates in a production-grade environment."*

### Kubernetes vs Alternatives

| Dimension | VMs (GCE/EC2) | Edge/Sandbox | MicroVMs (Firecracker) | K8s Pods (Emergent) |
|-----------|---------------|--------------|----------------------|-----------------|
| **Startup time** | 30–120s | <1s | ~125ms / 3–5s | **<8s** (warm pool) |
| **Full Linux + root** | Yes | No | Yes | Yes |
| **Database support** | Native | Managed only | Native | Native |
| **Security isolation** | Hypervisor | Process-level | Hypervisor (KVM) | **eBPF + namespace + cgroups** |
| **Ecosystem maturity** | Mature | Young | Young | **Mature (CNCF)** |
| **Scheduling** | Manual/custom | N/A | Custom orchestration | **Built-in** |
| **Relative cost/env** | ~4–5x | 1x (baseline) | ~3–4x | **~1x** |

### Session Lifecycle (1 Kubernetes Pod = 1 Session)

1. **Init: Restore** — Pulls user's previous state from content-addressed backup
2. **Main: Environment** — Full Linux with root, Node, Python, database, code server
3. **Sidecar: Operations** — Heartbeats, health reporting, backup on termination

---

## 💰 Pricing

**Source:** [emergent.sh/pricing](https://emergent.sh)

| Plan | Price | Credits | Key Features |
|------|-------|---------|--------------|
| **Free** | $0/mo | 10/month | Core platform, Web & Mobile, Advanced models, LLM integration |
| **Standard** | $20/mo (annual) | 100/month | Private hosting, GitHub integration, Fork tasks, Extra credits |
| **Pro** | $200/mo (annual) | 750/month | 1M context window, Ultra thinking, Custom AI agents, Priority support |

**Security:** SOC 2 Type I certified

---

## 🏆 Competitive Comparison

**Source:** [emergent.sh/learn/v0-vs-lovable-vs-bolt](https://emergent.sh/learn/v0-vs-lovable-vs-bolt)

| Criteria | v0 | Lovable | Bolt | **Emergent** |
|----------|-----|---------|------|-------------|
| **Full-Stack Coverage** | Frontend only | High (React + Supabase) | High | **Full-stack (React + FastAPI + MongoDB)** |
| **Code Ownership** | Full React code export | Full export to GitHub | Partial (vendor coupling) | **Full export, portable code** |
| **Scalability** | Medium | Medium | High | **High (1M context on Pro)** |
| **AI Agents** | No | No | No | **Yes (5 coordinated)** |
| **Agentic Approach** | No | No | No | **Yes — "agentic vibe coding"** |
| **Mobile App Support** | No | No | No | **Yes** |

---

## 🏢 Company

| Aspect | Details |
|--------|---------|
| **Founded** | ~2024 (YC S24) |
| **Headquarters** | San Francisco, CA |
| **Employees** | 75 total (70 Bengaluru, 5 SF) |
| **Total Raised** | $100M+ |
| **Valuation** | $300M |

### Founders

| Role | Name | Background |
|------|------|------------|
| **CEO** | Mukund Jha | Co-Founder/CTO of Dunzo (India's first quick-commerce platform, backed by Google and Reliance) |
| **CTO** | Madhav Jha | PhD in Theoretical Computer Science (Penn State), von Neumann postdoctoral fellow at Sandia National Labs, founding member of Amazon SageMaker team |

> *"Twin brothers dedicated to democratizing product development."*

### Funding History

| Round | Amount | Investors | Date |
|-------|--------|-----------|------|
| Seed (YC S24) | — | Y Combinator | Summer 2024 |
| Series A | $23M | Lightspeed Venture Partners | September 2025 |
| Series B | $70M | SoftBank Vision Fund 2, Khosla Ventures, Prosus, Together, Elad Gil | January 2026 |

### Mission Statement

> *"When creation costs fall, consumption increases. We're building towards a world where millions of businesses can exist that couldn't before."*

---

## 🔗 Related Documents

← Back to [ai-native-orchestration](./README.md)
