# Bolt.new

## 📌 Overview

> **Bolt.new** is StackBlitz's AI-powered vibe-coding platform that lets users prompt, run, edit, and deploy full-stack web applications directly in the browser — no local setup required. It runs Node.js entirely in the browser via **WebContainers** (WebAssembly-based OS).

> *"From idea to live product, Bolt adapts to the way you work turning every vision into something real & fast. Go from insight to prototype in hours and test ideas with your team before the day is over."*
> — [bolt.new](https://bolt.new)

---

## 📊 Key Metrics

| Metric | Value | Source |
|--------|-------|--------|
| **ARR** | $40M | Lenny's Newsletter |
| **Registered Users** | 3 million | Lenny's Newsletter |
| **Monthly Active Users** | 1 million | Lenny's Newsletter |
| **Time to $4M ARR** | 30 days | Evil Martians |
| **Time to $40M ARR** | ~5 months | Evil Martians |
| **Team Size** | 15 engineers | PostHog Newsletter |
| **Error loops reduction (v2)** | 98% fewer | Bolt Blog |
| **Project capacity (v2)** | 1000x bigger | Bolt Blog |

---

## 🛠️ Open Source

### Main Repository
**URL:** [github.com/stackblitz/bolt.new](https://github.com/stackblitz/bolt.new)
- **Stars:** 16.3k | **Forks:** 14.6k | **License:** MIT

### Open-Source Version (bolt.diy)
**URL:** [github.com/stackblitz-labs/bolt.diy](https://github.com/stackblitz-labs/bolt.diy)

> *"bolt.diy is the official open source version of Bolt.new, which allows you to choose the LLM that you use for each prompt!"*

**Supported LLM Providers:**
- **Cloud:** OpenAI, Anthropic, OpenRouter, Gemini, HuggingFace, DeepSeek, Groq, Cohere, Together, Perplexity, Moonshot (Kimi), Hyperbolic, GitHub Models, Amazon Bedrock, Mistral, xAI
- **Local:** Ollama, LMStudio, Local endpoints

**License Note:** bolt.diy is MIT, but WebContainers API requires licensing for **production/commercial usage**. Prototypes/POCs are free.

---

## ⚙️ Core Technology: WebContainers

### What Are WebContainers?
**URL:** [webcontainers.io](https://webcontainers.io/guides/introduction)

> *"A browser-based runtime for executing Node.js applications and operating system commands, entirely inside the browser tab."*

**Key quote from Albert Pai (Co-founder & CTO):**
> *"People assume we're running a giant server farm. In reality, the **server is your browser.**"*

### WebContainers vs Cloud VMs

| Aspect | Cloud VMs | WebContainers |
|--------|-----------|---------------|
| Latency | Network round-trips | Near-zero latency |
| Cost | Per-minute billing | Client-side compute |
| Security | Remote compute risks | Abuser uses own CPU |
| Experience | Feels like remote server | Feels like localhost |

### Technical Challenges Solved

**1. Virtual File System**
> *"Node's `fs` API expects POSIX-compliant disk operations. Browsers don't expose low-level file system APIs."*
> **Solution:** Rust-based file system compiled to WebAssembly (WASM) running in SharedArrayBuffer for near-native speed with thread-safe file locks.

**2. Process & Threading Model**
> *"Node apps spawn multiple processes, but browsers only have one UI thread + Web Workers."*
> **Solution:** Web Workers act as processes, each with a tiny Rust "kernel" managing tasks. Shared memory space via SharedArrayBuffer with Atomics for IPC.

**3. Networking Without Localhost**
> *"Dev servers bind to `127.0.0.1:PORT` but browsers can't listen on ports."*
> **Solution:** Virtual localhost via Service Worker intercepting special URLs (e.g., `/__bolt/3000/...`) and routing to correct Web Worker.

**4. npm Install Performance**
> *"Instant installs: Popular packages pre-compressed on Bolt's CDN, cached in browser → `npm install` often completes in **< 500ms** or skipped entirely."*

---

## 🚀 Bolt v2 Features

**Source:** [bolt.new/blog/bolt-v2](https://bolt.new/blog/bolt-v2)

| Feature | Description |
|---------|-------------|
| Error loops | **98% fewer** with autonomous debugging |
| Project size | **1000x bigger** projects |
| AI agents | Multi-agent power, switch between leading models |
| Built-in databases | Unlimited, automatically created |
| Free domain | `.bolt.host` included |
| Auth | One-prompt authentication |
| Payments | Stripe integration |
| SEO | Pre-rendering boost |
| Enterprise | Security by default |

---

## 💰 Pricing

**Source:** [bolt.new/pricing](https://bolt.new/pricing)

| Plan | Price | Key Features |
|------|-------|--------------|
| **Free** | $0 | 1M tokens/month, 300K daily limit, Bolt branding |
| **Pro** | $25/mo | 10M tokens, no daily limit, no branding, custom domain, SEO boost |
| **Teams** | $30/mo/member | Everything Pro + shared workspace, admin controls, design systems |
| **Enterprise** | Custom | SSO, audit logs, dedicated support, custom SLAs |

---

## 🔬 Architecture Deep Dive

**Source:** [PostHog Newsletter](https://newsletter.posthog.com/p/from-0-to-40m-arr-inside-the-tech) | [Evil Martians](https://evilmartians.com/chronicles/bolt-new-from-stackblitz-how-they-surfed-the-ai-wave-with-no-wipeouts)

### How Bolt Works
1. User enters a prompt (e.g. *"Build a Tinder for Dogs web app"*)
2. Prompt sent to LLM to generate app code
3. Browser boots up **WebContainer** (in-browser virtual machine)
4. WebContainer runs setup commands (`npm install`, creates files, etc.)
5. Code executes and displays preview in the browser

### The Claude Breakthrough
**Quote from Eric Simons (Founder & CEO):**
> *"There's an order of magnitude difference in the LLM's required infrastructure to make it functional versus zero shot. Claude 3.5 Sonnet is the enabling technology that made this product possible, period."*

---

## ⚖️ Comparison with Alternatives

### Bolt.new vs Lovable vs v0.dev

**Source:** [Pixeljets](https://pixeljets.com/blog/lovable-dev-vs-bolt-new/), [Hostinger](https://www.hostinger.com/my/tutorials/lovable-vs-bolt-new)

| Feature | Bolt.new | Lovable | v0.dev |
|---------|----------|---------|--------|
| **Code Execution** | WebContainers (browser) | Fly.io VMs | Cloud |
| **IDE Experience** | Full IDE + Terminal | No built-in editor | Limited |
| **Code Editing** | Built-in | Via GitHub only | Limited |
| **Database** | Supabase | Supabase | Vercel |
| **Deployment** | Bolt Cloud/Netlify | Vercel/Netlify | Vercel |
| **Framework Support** | Most JS frameworks | React focused | React/Vercel |
| **Target User** | Developers/Pro | Non-technical | Designers |

**Author's Verdict (Pixeljets):**
> *"Prefers Bolt.new because: more mature UX, real editor and terminal, better pricing model, cutting-edge WebAssembly sandboxing."*

---

## 🏢 Company: StackBlitz

| Aspect | Details |
|--------|---------|
| **Founded** | 2018 |
| **Founders** | Eric Simons (CEO), Albert Pai (CTO) |
| **Headquarters** | San Francisco, CA |
| **Seed Round** | $7.9M — Greylock, Google Ventures, Tom Preston-Werner |
| **Series A** | $135M — 2025 |

---

## 🔗 Related Documents

← Back to [ai-native-orchestration](./README.md)
