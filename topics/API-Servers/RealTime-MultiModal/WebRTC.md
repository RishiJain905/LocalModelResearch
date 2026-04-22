# WebRTC for Real-Time Multimodal AI

> *"WebRTC defines ECMAScript APIs enabling media and application data to be sent/received between browsers via peer-to-peer connections."* — [W3C WebRTC Specification](https://www.w3.org/TR/webrtc/)

## Contents

1. [Definitions & Standards](#definitions--standards)
2. [Key RFCs & Specifications](#key-rfcs--specifications)
3. [Academic Papers](#academic-papers)
4. [Open-Source Libraries & Tools](#open-source-libraries--tools)
5. [GitHub Repositories](#github-repositories)
6. [Blog Posts & Articles](#blog-posts--articles)
7. [Protocol Latency Comparison](#protocol-latency-comparison)
8. [Debates & Trade-offs](#debates--trade-offs)
9. [Key Findings](#key-findings)

---

## Definitions & Standards

### What is WebRTC?

> *"WebRTC is a platform giving browsers, mobile apps, and desktop apps real-time communication capabilities, typically used for video calling."*
> — [web.dev](https://web.dev/articles/webrtc-standard-announcement)

> *"The internet's core protocols weren't designed for realtime media. HTTP is optimized for request-response communication, which is effective for the web's client-server model, but not for continuous audio and video streams. WebRTC is a browser-native technology for transmitting audio and video in realtime."*
> — [LiveKit Docs](https://docs.livekit.io/intro/about/)

> *"WebRTC connections are always encrypted using DTLS (Datagram Transport Layer Security) and SRTP (Secure Real-time Transport Protocol)."*
> — [W3C WebRTC Specification](https://www.w3.org/TR/webrtc/)

### Core WebRTC API

| API | Purpose |
|-----|---------|
| `RTCPeerConnection` | Central connection API between local and remote peers |
| `getUserMedia` | Access camera and microphone |
| `DataChannel` | Send/receive arbitrary binary data |
| `MediaStreamTrack` | Audio/video tracks |

> *"RTCPeerConnection represents a connection between the local device and a remote peer."*
> — [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection)

### Mandatory Codecs

> *"WebRTC implementations are required to support both Google's free-to-use VP8 video codec and H.264 for processing video."*
> — [web.dev](https://web.dev/articles/webrtc-standard-announcement)

> *"The new video codec AV1 which saves up to 50% of bandwidth is becoming available in WebRTC and web browsers."*
> — [web.dev](https://web.dev/articles/webrtc-standard-announcement)

### W3C Standard Status

> *"WebRTC"* — W3C Recommendation (May 2025)
> Editors: Cullen Jennings (Cisco), Florent Castelli (Google), Henrik Boström (Google), Jan-Ivar Bruaroey (Mozilla)

— [W3C WebRTC Specification](https://www.w3.org/TR/webrtc/)

---

## Key RFCs & Specifications

| RFC | Title |
|-----|-------|
| RFC 7742 | WebRTC Video Processing and Codec Requirements |
| RFC 7874 | WebRTC Audio Codec and Processing Requirements |
| RFC 8831 | WebRTC Data Channels |
| RFC 8829 | JavaScript Session Establishment Protocol (JSEP) |
| RFC 8834 | Media Transport and Use of RTP in WebRTC |
| RFC 8827 | WebRTC Security Architecture |
| RFC 9429 | JavaScript Session Establishment Protocol (updated JSEP) |

> *"IETF RTCWeb Working Group — concluded with 17 published RFCs."*
> — [IETF Datatracker](https://datatracker.ietf.org/wg/rtcweb/)

---

## Academic Papers

### Speech ReaLLM – Real-time Streaming Speech Recognition with Multimodal LLMs

> *"We introduce Speech ReaLLM, a new ASR architecture that marries decoder-only ASR with the RNN-T to make multi-modal LLM architectures capable of real-time streaming."*
> — [arXiv:2406.09569](https://arxiv.org/abs/2406.09569)

### Streaming Remote Rendering Services: Comparison of QUIC-based and WebRTC Protocols

> *"By examining Media over QUIC (MoQ), Real-time Transport Protocol (RTP) over QUIC (RoQ), and Web Real-Time Communication (WebRTC), we assess their latency performance over Wi-Fi and 5G networks."*
> — [arXiv:2505.22132v1](https://arxiv.org/html/2505.22132v1)

### Vision-Language Models on the Edge for Real-Time Robotic Perception

> *"We design a WebRTC-based pipeline that streams multimodal data to an edge node and evaluate LLaMA-3.2-11B-Vision-Instruct deployed at the edge."*
> — [ResearchGate](https://www.researchgate.net/publication/399961786_Vision-Language_Models_on_the_Edge_for_Real-Time_Robotic_Perception)

---

## Open-Source Libraries & Tools

### LiveKit (Open Source WebRTC Infrastructure)

> [livekit.io](https://livekit.io) | [github.com/livekit](https://github.com/livekit)

> *"Scalable, distributed WebRTC SFU (Selective Forwarding Unit); Modern, full-featured client SDKs; Built for production, supports JWT authentication"*
> — [github.com/livekit/livekit](https://github.com/livekit/livekit)

**LiveKit Agents Framework:**
> *"Build real-time multimodal AI applications. The Agents framework enables you to build AI-driven server programs that can see, hear, and speak in realtime."*
> — [github.com/livekit/agents](https://github.com/livekit/agents)

> *"WebRTC doesn't natively work on the server which an AI agent needs to communicate with clients in real-time."*
> — [LiveKit Blog](https://livekit.com/blog/open-source-realtime-multimodal-ai)

### Pipecat AI

> [pipecat.ai](https://github.com/pipecat-ai) | `pip install pipecat-ai` | ⭐ 11.5k stars

> *"Pipecat AI is an open-source framework for building voice and multimodal conversational agents. Pipecat simplifies the complex voice-to-voice AI pipeline."*
> — [github.com/pipecat-ai/pipecat](https://github.com/pipecat-ai/pipecat)

### Server-Side WebRTC Implementations

| Library | Language | URL | Notes |
|---------|----------|-----|-------|
| **Pion WebRTC** | Go | [github.com/pion/webrtc](https://github.com/pion/webrtc) | Pure Go WebRTC API; used by LiveKit |
| **aiortc** | Python | [github.com/aiortc/aiortc](https://github.com/aiortc/aiortc) | ⭐ 5k; asyncio WebRTC/ORTC |
| **werift-webrtc** | Node.js | [github.com/shinyoshiaki/werift-webrtc](https://github.com/shinyoshiaki/werift-webrtc) | Server-side WebRTC for Node.js |

---

## GitHub Repositories

| Repository | Description |
|-----------|-------------|
| [livekit/livekit](https://github.com/livekit/livekit) | WebRTC SFU server in Go |
| [livekit/agents](https://github.com/livekit/agents) | AI agent framework for real-time multimodal AI |
| [pipecat-ai/pipecat](https://github.com/pipecat-ai/pipecat) | Voice/multimodal conversational AI framework |
| [pion/webrtc](https://github.com/pion/webrtc) | Pure Go WebRTC implementation |
| [aiortc/aiortc](https://github.com/aiortc/aiortc) | Python asyncio WebRTC |
| [livekit-examples/multimodal-agent-python](https://github.com/livekit-examples/multimodal-agent-python) | Python multimodal AI example |
| [pipecat-ai/gemini-webrtc-web-simple](https://github.com/pipecat-ai/gemini-webrtc-web-simple) | Gemini Multimodal Live API + WebRTC example |

---

## Blog Posts & Articles

### Cloudflare: Multimodal Real-Time AI Applications

> *"WebRTC solves this problem by having native support for audio and video tracks over UDP-based channels directly between users, eliminating the need for chunking."*
> *"Previously, interactions with audio and video AIs were largely single-player: only one person could be able to be interacting with the AI unless you were in the same physical room."*
> — [Cloudflare Blog](https://blog.cloudflare.com/bring-multimodal-real-time-interaction-to-your-ai-applications-with-cloudflare-calls/), December 20, 2024

### LiveKit: Open Source Stack for Real-Time Multimodal AI

> *"Most LLMs take in text as input and generate text as output, but some AI models are purpose-built to transform other modalities to text, or synthesize audio/video from text in real-time."*
> *"AI Agents are about to become very real."*
> *"WebSocket uses TCP; packet loss causes Head-of-Line (HOL) blocking, leading to stutters and delays in audio/video playback."*
> — [LiveKit Blog](https://livekit.com/blog/open-source-realtime-multimodal-ai)

### GetStream: WebRTC for AI Voice and Video Agents

> *"WebRTC has become the standard transport layer for AI agents requiring real-time voice and video communication."*
> *"WebRTC uses UDP, accepting packet loss for consistent low latency."*
> — [GetStream Blog](https://getstream.io/blog/webrtc-ai-voice-video/)

### Twilio: AI Voice Assistant with OpenAI Realtime API

> *"S2S models improve latency by avoiding traditional speech-to-text (SST) and text-to-speech (TTS) steps."*
> — [Twilio Blog](https://www.twilio.com/en-us/blog/voice-ai-assistant-openai-realtime-api-python)

### WebRTCHacks: WebRTC vs. MoQ by Use Case

> *"MoQ aims to pair WebRTC-like interactivity/latency with HLS/DASH-like distribution and caching."*
> *"Voice AI is a relatively new use case for WebRTC where you send your audio and video to a real time LLM and it respond back with media stream."*
> — [webrtchacks.com](https://webrtchacks.com/webrtc-vs-moq-by-use-case/)

---

## Protocol Latency Comparison

| Protocol | Latency | Use Case |
|----------|---------|----------|
| HLS/DASH | 6–60 seconds | One-way broadcasts, VOD |
| LL-HLS | 2–4 seconds | Low-latency streaming |
| **WebRTC** | **<500ms** | Real-time communication |
| MoQ | Sub-second | Interactive streaming |
| RTMP | ≤5 seconds | Ingest/contribution |

> *"WebRTC: Sub-500ms latency"* — [nanocosmos](https://www.nanocosmos.net/blog/webrtc-latency/)

---

## Debates & Trade-offs

### WebRTC vs. WebSockets for AI

| Aspect | WebSockets | WebRTC |
|--------|------------|--------|
| Protocol | Single persistent TCP | UDP-based channels |
| Best For | Text chat, gameplay state | Audio/video streams |
| Latency | 100–500ms chunking | Near-zero latency |
| HOL Blocking | Yes (TCP) | No (UDP) |

> *"WebRTC generally provides a smoother, lower-latency experience due to its UDP protocol, which handles lost data packets better than TCP-based Realtime APIs."*
> — [eesel AI](https://eesel.ai)

### WebRTC vs. MoQ (Media over QUIC)

> *"MoQ is a client-server protocol, so I don't see how MoQ will ever be ideal [for 1:1 video calls] unless MoQ vendors can make an argument that putting more infrastructure in between two users is better."*
> *"MoQ bridges real-time communication and large-scale content delivery, making it well-suited for interactive live streaming and enterprise use cases."*
> — [webrtcHacks](https://webrtchacks.com/webrtc-vs-moq-by-use-case/)

**When to use MoQ instead of WebRTC:**
- Large-scale live streaming (100,000+ viewers)
- One-to-many broadcasts
- When CDN caching is needed

### Scalability Challenges

> *"Peer-to-peer design not built for large-scale broadcasting."*
> *"Additional protocols needed when streaming to >50 concurrent connections."*
> — [nanocosmos](https://www.nanocosmos.net/blog/webrtc-latency/)

**Solution: SFU (Selective Forwarding Unit)** — Publisher sends once, server forwards to subscribers (used by LiveKit).

---

## Key Findings

- **WebRTC is a W3C/IETF standard** (2025) — 17 IETF RTCWeb RFCs govern it
- **Sub-500ms latency** via UDP — critical for voice AI applications
- **WebRTC SFU architecture** (e.g., LiveKit) solves the server-side limitation — *"WebRTC doesn't natively work on the server which an AI agent needs to communicate with clients in real-time"*
- **Key players**: LiveKit (SFU + Agents), Pipecat (Daily.co), Cloudflare Calls, OpenAI Realtime API (WebRTC added Dec 2024)
- **Server-side libraries**: Pion (Go, used by LiveKit), aiortc (Python/asyncio), werift (Node.js)
- **MoQ emerging** for large-scale streaming, not replacing WebRTC for 1:1 AI voice/video
- **WebRTC market**: $247.7B expansion (2025–2029) at 62% CAGR

---

## Related Topics

- [WebSockets](./WebSockets.md) — Bidirectional persistent connections for AI streaming
- [Stateful-Sessions](./Stateful-Sessions.md) — Session management for real-time AI
