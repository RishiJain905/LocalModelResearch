# gen-ui-dynamics

> Techniques for **how** LLM outputs are delivered and rendered in web UIs — streaming tokens, progressive rendering, and runtime component generation.

---

## 📂 Contents

| Document | Description |
|----------|-------------|
| [Dynamic-Rendering.md](./Dynamic-Rendering/) | Token streaming, SSE, real-time UI updates from LLM outputs, React patterns |
| [Just-in-time-Components.md](./Just-in-time-Components/) | On-demand UI generation, LLM-to-component, schema-driven dynamic UIs |

---

## 🔑 Key Themes

### Dynamic Rendering
- **SSE** is the de facto standard for LLM streaming (works over plain HTTP, auto-reconnects)
- **TTFT** (Time to First Token) — streaming cuts perceived latency 10–20x vs waiting for full response
- Token-by-token rendering requires smoothing (frame-rate matching) to avoid jitter
- React streaming patterns: `useStream`, streaming React Server Components, WebSocket fallbacks
- Key challenge: code block rendering, markdown parsing, and pause-smoothing in live streams

### Just-in-Time Components
- LLMs generate **actual UI components** (React, HTML/CSS/JS) at runtime — not just text
- **Generative UI** outperforms markdown significantly (82.8% human preference in Google research)
- Three patterns: Static (AG-UI), Declarative (A2UI/Open-JSON-UI), Open-ended (MCP Apps)
- Security: sandboxed execution (iframe), declarative formats (A2UI sends data not code), policy profiles
- Token efficiency: OpenUI's language is 52–67% more efficient than JSON for UI description

---

## 📊 Taxonomy Matrix

| Technique | Venue | Method | Key Result |
|-----------|-------|--------|------------|
| Generative UI (Google) | arXiv:2604.09577 | LLM → HTML/CSS/JS | 82.8% prefer over markdown |
| A2UI | Google (Apache 2.0) | Declarative JSON spec | Cross-platform, secure by design |
| AG-UI Protocol | CopilotKit | Bidirectional agent-UI | 30k+ GitHub stars |
| Renderify | Community | Browser-runtime JSX transpile | Zero build step, sandboxed |
| OpenUI | Thesys | Streaming DSL | 52–67% token reduction vs JSON |

---

## 🔗 Related Topics

- [fine-tuning](../fine-tuning/) — Parent directory
- [training-infrastructure](../training-infrastructure/) — Infrastructure concerns (latency, streaming at scale)
