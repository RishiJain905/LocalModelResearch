# Dynamic Rendering

## 📌 Overview

> Dynamic Rendering in LLM web UIs covers the **how** of delivering LLM outputs to users in real time — streaming individual tokens or chunks as they are generated, progressively rendering markdown/code, and updating UI state incrementally rather than after a complete response.

---

## 📚 Key Concepts

### Streaming Architectures

**Server-Sent Events (SSE)** is the dominant protocol for LLM streaming:

> *"SSE allows the server to push updates to the client automatically over a single, long-lived HTTP connection, eliminating the need for constant polling and providing a more responsive, lightweight, and efficient real-time experience."*
> — [IBM Developer Blog](https://community.ibm.com/community/user/blogs/anjana-m-r/2025/10/03/server-sent-events-the-perfect-match-for-real-time)

**Why SSE over WebSockets for LLM streaming:**
- Works over standard HTTP — no special infrastructure
- Browser has native `EventSource` API with automatic reconnection
- Unidirectional (server → client) matches the LLM streaming model
- Easier to proxy through CDNs and load balancers

**Why WebSockets are NOT ideal for LLM streaming:**
- Bidirectional capability is wasted (client rarely sends during streaming)
- No native browser reconnection handling
- More complex infrastructure (sticky sessions, stateful connections)
- Serverless platforms (AWS Lambda, etc.) struggle with long-lived WebSocket connections

### Time to First Token (TTFT)

> *"With streaming, they start seeing content within the TTFT window — a 10–20x improvement in perceived responsiveness."*
> — [Dev.to: The Complete Guide to Streaming LLM Responses](https://dev.to/pockit_tools/the-complete-guide-to-streaming-llm-responses-in-web-applications-from-sse-to-real-time-ui-3534)

---

## 🛠️ Key Libraries & Implementations

### llm-ui
**URL:** [llm-ui.com](https://llm-ui.com/) | **Repo:** [github.com/richardgill/llm-ui](https://github.com/richardgill/llm-ui)

> *"The React library for LLMs — removes broken markdown syntax, renders **bold**, *italic*, ~~strikethrough~~ and links without showing any markdown syntax."*

**Key features:**
- Renders **bold**, *italic*, ~~strikethrough~~, `links` correctly
- Custom component syntax via a special delimiter format:
```
【{type:"‎buttons",buttons:[{text:"Star ⭐"}, {text:"Confetti 🎉"}]}】
```
- **Frame-rate matching** — smooths token streaming to native display refresh rate
- **Pause smoothing** — removes perceptually distracting pauses in LLM output
- Universal LLM support (operates on model's output string)

### React Streaming Patterns

**LangChain `useStream` hook:**
```typescript
import { useStream } from "@langchain/langgraph-sdk/react";

const { stream, handleSubmit } = useStream({
  agentId: "my-agent",
});
```

**Next.js Route Handler (SSE proxy to OpenAI):**
```typescript
res.setHeader('Content-Type', 'text/event-stream');
const stream = await openai.chat.completions.create({
  model: 'gpt-4o',
  stream: true,
  messages: [{ role: 'user', content: prompt }],
});

for await (const chunk of stream) {
  const content = chunk.choices[0]?.delta?.content || '';
  res.write(`data: ${JSON.stringify({ type: 'content', content })}\n\n`);
}
res.end();
```

**Vercel AI SDK (recommended approach):**
```typescript
import { streamText } from 'ai';

// In your route handler:
const result = await streamText({
  model: openai('gpt-4o'),
  system: 'You are a helpful assistant.',
  messages,
});

return result.toDataStreamResponse();
```

---

## 📰 Blog Posts & Guides

### "The Complete Guide to Streaming LLM Responses in Web Applications: From SSE to Real-Time UI"
**Source:** [Dev.to](https://dev.to/pockit_tools/the-complete-guide-to-streaming-llm-responses-in-web-applications-from-sse-to-real-time-ui-3534)

**Key quote:**
> *"Streaming LLM responses is one of the most common challenges developers face when building AI-powered applications in 2024–2025."*

**SSE production considerations (from Nginx config):**
```nginx
# Disable buffering for SSE
ProxyPreserveHost On
ProxyVia Off
Header always set Cache-Control "no-cache, no-store, must-revalidate"
Header always set Connection "keep-alive"
# Disable compression for text/event-stream
SetEnvIfNoCase Content-Type "^text/event-stream" no-gzip dont-vary
```

---

### "Server-Sent Events: The Perfect Match for Real-Time Chat"
**Source:** [IBM Developer Blog](https://community.ibm.com/community/user/blogs/anjana-m-r/2025/10/03/server-sent-events-the-perfect-match-for-real-time)

> *"SSE isn't trying to replace WebSockets — it's carving out its own niche as the go-to solution for server-to-client streaming."*

---

### "How We Used SSE to Stream LLM Responses at Scale"
**Source:** [Medium/@daniakabani](https://medium.com/@daniakabani/how-we-used-sse-to-stream-llm-responses-at-scale-fa0d30a6773f)

> *"It was about making sure those answers could be streamed to thousands of concurrent clients smoothly, reliably, and without blowing up costs."*

---

### "Rendering Realtime UIs with Streaming Structured LLM Completions"
**Source:** [Medium/@enginoid](https://medium.com/@enginoid/rendering-realtime-uis-with-streaming-structured-llm-completions-5d10479cefc0)

> *"Streaming tokens directly into a UI only works when the UI captures all the tokens into the same UI element. But if you want to partition the output into different UI regions (e.g., a code block vs. explanatory text), you need structured streaming — parsing the token stream into semantic units as they arrive."*

---

### "UX Design Without Designers? How LLMs Are Rewriting UI in Real Time"
**Source:** [Francesca Tabor](https://www.francescatabor.com/articles/2025/9/6/ux-design-without-designers-how-llms-are-rewriting-ui-in-real-time)

> *"Natural language interfaces backed by LLMs let users specify their intent in plain English, rather than interacting through code or complex UIs."*

---

## ⚠️ Key Challenges

| Challenge | Description | Solutions |
|-----------|-------------|-----------|
| **Code block rendering** | Tokens streaming into a code block mid-type | Use a separate buffer that holds content until a complete token boundary is reached |
| **Pause detection** | LLM "thinks" between tokens causing UI jitter | Implement a smoothing buffer that delays rendering by ~20ms |
| **Markdown parsing** | Markdown syntax appears before it forms valid HTML | Stream into a hidden buffer, parse on chunk boundaries |
| **Serverless incompatibility** | SSE requires long-lived connections most serverless platforms don't support | Use Edge Functions (Cloudflare Workers, Vercel Edge) or dedicated streaming services |
| **Proxy buffering** | Nginx, Cloudflare, and other proxies buffer responses | Configure `X-Accel-Buffering: no`, set no-gzip env vars |

---

## 🔗 Related Documents

← Back to [gen-ui-dynamics](./README.md)
