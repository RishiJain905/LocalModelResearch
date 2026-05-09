# Continuous Listening: Multimodal Perception Pattern for LLM Interactions

> **Research Path:** `topics/web-uis/multimodal-perception/continuous-listening/`

---

## Table of Contents

1. [Overview](#overview)
2. [Academic Papers](#1-academic-papers-arxiv)
3. [Blog Posts & Articles](#2-blog-posts--articles)
4. [Open Source Repositories](#3-open-source-repos-github)
5. [Technical Reports & Documentation](#4-technical-reports--documentation)
6. [Talks & Videos](#5-talks--videos)
7. [Key Concepts Summary](#key-concepts-summary)
8. [Cross-Cutting Themes](#cross-cutting-themes)

---

## Overview

**Continuous Listening** is a multimodal perception pattern where an LLM-backed system maintains persistent audio processing, capturing and responding to speech in real-time without explicit push-to-talk activation. This paradigm enables:

- Always-on audio input with voice activity detection (VAD)
- Real-time speech-to-text (STT) streaming
- Full-duplex communication (simultaneous listen/speak)
- Interruption handling and turn-taking management
- Ambient context awareness alongside explicit commands

**Core tension:** Ambient/passive listening enables frictionless interaction but raises significant privacy implications. Push-to-talk offers explicit control but breaks conversational flow.

---

## 1. Academic Papers (arXiv)

### Paper 1: Interactive Multimodal Perception Using Large Language Models

**URL:** https://arxiv.org/abs/2303.08268

**Citation:** Zhao et al., "Chat with the Environment: Interactive Multimodal Perception Using Large Language Models," IROS 2023

**Key Concepts:**
> "LLMs can provide high-level planning and reasoning skills and control interactive robot behavior in a multimodal environment, while multimodal modules with the context of the environmental state help ground the LLMs."

**Valuable because:**
- First comprehensive framework for LLM-as-backbone controlling multimodal (vision, sound, haptics, proprioception) robot interaction
- Introduces "Matcha" agent that chats with environment using epistemic actions
- Bridges symbolic LLM reasoning with perceptual grounding

---

### Paper 2: MM-When2Speak — Multimodal LLM Timing Prediction

**URL:** https://arxiv.org/html/2505.14654v1

**Key Concepts:**
> "Modern LLMs 'know what to say but not when to speak', missing within-utterance openings."

- Predicts 9 response types including: affirmation, gratitude, farewell, greeting, question, surprise, pondering, full_response, and silence
- Architecture: Video (InternViT-300M) + Audio (Mel + Transformer) + Text (transcribed) → Self-Attention Fusion → Qwen2-7B LLM
- Achieves 4× better timing vs ChatGPT-4o text-only, 1.5× better vs VITA-1.5

**Valuable because:**
- Solves the "when to speak" problem LLMs lack
- Demonstrates multimodal fusion (video + audio + text) improves timing prediction
- Self-attention fusion yields average 3.62% F1 improvement across modalities

---

### Paper 3: Interruption Handling for Conversational Robots

**URL:** https://arxiv.org/abs/2501.01568v1

**Key Concepts:**
> "Interruptions can be used to signal understanding (cooperative agreement), assist the speaker (cooperative assistance), seek clarification when needed (cooperative clarification), or express disagreement and shift the topic (disruptive)."

- System detects user-initiated interruptions and manages them in real-time based on interrupter intent
- Four interruption types: cooperative agreement, cooperative assistance, cooperative clarification, disruptive
- Integrated into LLM-powered social robot; validated with 21 participants in decision-making and discussion tasks

**Valuable because:**
- First systematic interruption taxonomy for human-robot conversation
- Data-driven design based on interaction patterns from human-human interaction corpora
- Practical implementation validated in real user studies

---

### Paper 4: Exploring Multimodal Perception in Large Language Models

**URL:** https://arxiv.org/abs/2503.06980

**Citation:** IEEE Access 2025

**Key Concepts:**
- Investigates whether multimodal LLMs achieve human-like sensory grounding
- Examines perceptual strength ratings across modalities
- Published in IEEE Access with peer review

**Valuable because:**
- Academic rigor on MLLM perceptual capabilities assessment
- Systematic evaluation framework for multimodal grounding
- Published venue indicates validated methodology

---

### Paper 5: Building Enterprise Realtime Voice Agents from Scratch

**URL:** https://arxiv.org/html/2603.05413v1

**Key Concepts:**
- Technical tutorial covering full realtime voice agent architecture
- Three levels: (1) Text LLM with speech I/O, (2) Speech-to-Speech models, (3) Cascaded pipeline ASR→LLM→TTS
- Streaming LLM with vLLM, function calling, backend flexibility

**Valuable because:**
- Comprehensive technical guide from architecture to implementation
- Covers streaming pipeline components and orchestration
- Enterprise-grade considerations for production systems

---

## 2. Blog Posts & Articles

### Article 1: Using a Speech Language Model That Can Listen While Speaking

**URL:** https://getstream.io/blog/realtime-speech-language-models/

**Key Concepts:**
> "Traditional speech language models like Siri or Alexa use turn-taking as the primary interaction style — they cannot be interrupted in real time."

Introduces **Listening-While-Speaking Language Models (LSLM)** with Full-Duplex Modeling (FDM):

| Capability | Traditional SLM | LSLM |
|-----------|-----------------|------|
| Turn-based only | ✓ | ✗ |
| Real-time interruptions | ✗ | ✓ |
| Distinguish human voices from noise | ✗ | ✓ |
| Multi-modal integration | Limited | ✓ |

**Valuable because:**
- Clear taxonomy of half-duplex vs full-duplex speech systems
- Real-world examples: GPT-4o, Moshi Chat as LSLM implementations
- Use cases in healthcare, education, language learning, customer service

---

### Article 2: Advances in Audio Real-time Communication for Natural and Interactive Conversational AI

**URL:** https://atscaleconference.com/advances-in-audio-real-time-communication-for-natural-and-interactive-conversational-ai/

**Authors:** Karim Helwani, Hoang Do, Sriram Srinivasan (Meta)

**Key Concepts:**
Meta's multilayered audio AI stack:

1. **Voice Clarity Detector** — Near/far speaker detection using Gaussian Mixture Model (GMM) with energy, spectral centroid, clarity features
2. **Primary Speaker Segmentation** — TDNN backbone with causal multi-scale convolutions; no enrollment needed
3. **Echo Control** — Client-side AEC + server-side echo control catching stragglers
4. **Endpointer** — Uses semantic information from user "turn end"

> "Unlike human-to-human communication (where there is little to no tolerance for artifacts), in human-to-machine communication, a small degree of distortions can be accepted to ensure the user's voice is always prioritized."

**Valuable because:**
- Production-grade architecture from Meta's voice team
- Specific technical solutions for double-talk, speaker identification, echo cancellation
- Transport optimization (burst buffered audio faster than real-time)

---

### Article 3: Ambient Listening vs Voice Dictation

**URL:** https://sunoh.ai/blog/ambient-listening-technology-vs-voice-dictation/

**Key Concepts:**
| Feature | Voice Dictation | Ambient Listening |
|---------|-----------------|-------------------|
| Real-time transcription | ✅ | ✅ |
| Understands medical context | ❌ | ✅ |
| Multilingual support | ❌ | ✅ |
| Passive ambient listening | ❌ | ✅ |
| AI-driven personalization | ❌ | ✅ |

**Valuable because:**
- Clear contrast between passive ambient vs active dictation paradigms
- Healthcare-focused example showing contextual understanding importance
- Demonstrates ambient listening enables face-to-face care

---

### Article 4: Complete Guide to Real-Time Transcription

**URL:** https://www.assemblyai.com/blog/real-time-speech-to-text-best-for-voice-agents

**Key Concepts:**
Real-time STT processes audio in 20-100ms chunks, delivering partial and final results:

**Critical metrics:**
- **WER** (Word Error Rate) — accuracy target <10%
- **TTFT** (Time to First Token) — target under 300ms
- **RTF** (Real-Time Factor) — target below 0.5
- **TTCT** (Time to Complete Turn) — target under 1 second

> "If the transcription is wrong, the LLM responds to the wrong thing. An agent that mishears 'I need to cancel my order' as 'I need to scan my order' sends the conversation off a cliff."

**Valuable because:**
- Comprehensive metrics framework for evaluating real-time STT
- Distinguishes real-time vs batch processing trade-offs
- Voice-agent-specific requirements (entity accuracy, turn detection)

---

### Article 5: Ambient Agents: Always On, Always Listening, Always Working

**URL:** https://medium.com/data-science-collective/ambient-agents-always-on-always-listening-always-working-2bba910fc3e8

**Key Concepts:**
> "Ambient agents run in the background, are stateful, and long-running."

- Shift from request-response to proactive, context-aware AI
- Agentic Mesh ecosystem services for ambient agent support
- Stateful background operation vs stateless request handling

**Valuable because:**
- Distinguishes ambient agents from traditional reactive assistants
- architectural pattern for always-on AI systems
- Ecosystem perspective on infrastructure requirements

---

### Article 6: Architecting Low-Latency, Real-Time AI Voice Agents

**URL:** https://dev.to/lifeisverygood/architecting-low-latency-real-time-ai-voice-agents-challenges-solutions-hdn

**Key Concepts:**
Paradigm shift from sequential batch processing to concurrent streaming architecture:

```python
# Streaming STT pattern
async def stream_audio_to_stt(audio_chunk_generator):
    async for chunk in audio_chunk_generator:
        partial_result = await stt.process(chunk)
        yield partial_result
```

**Valuable because:**
- Code-level implementation patterns for streaming architecture
- Concurrency patterns for voice agent pipelines
- Serverless considerations for scalability

---

## 3. Open Source Repos (GitHub)

### Repo 1: LiveKit Agents

**URL:** https://github.com/livekit/agents

**Description:** Framework for building realtime voice AI agents

**Key Features:**
- Conversational, multi-modal voice agents that see, hear, and understand
- Python and Node.js SDKs
- Built-in examples: voice agents, multi-user push-to-talk, background audio, tool creation, outbound callers
- Adaptive Interruption Handling with trained audio-based interruption detection model
- Integrations with Daily, Twilio, telephony

**Stats:** 3,237 commits, Apache License

**Valuable because:**
- Production-ready framework with enterprise features
- Adaptive Interruption Handling built-in (multilingual, generalizes to unseen languages)
- Rich plugin ecosystem and examples

---

### Repo 2: Silero VAD

**URL:** https://github.com/snakers4/silero-vad

**Description:** Pre-trained enterprise-grade Voice Activity Detector

**Key Features:**
- **Fast:** <1ms per audio chunk (30+ ms) on single CPU thread
- **Lightweight:** JIT model ~2 megabytes
- **Multi-language:** Trained on 6000+ languages
- **MIT License** — no telemetry, keys, registration

**Ports:** C++, Rust, Go, Java, C#, JavaScript (Web/ONNX Runtime Web)

**Valuable because:**
- High accuracy with minimal computational overhead
- Truly open-source with permissive license
- Broad platform and language support

---

### Repo 3: TEN VAD

**URL:** https://github.com/ten-framework/ten-vad

**Description:** Low-latency, high-performance streaming VAD

**Key Features:**
- **Best precision-recall curves** vs WebRTC VAD and Silero VAD
- **306KB library size** vs Silero's 2.16MB
- **Faster transitions:** Silero has several hundred ms delay; TEN VAD enables rapid speech-to-non-speech detection
- **5 platforms:** Linux, Windows, macOS, Android, iOS
- **5+ languages:** Python, C, Java, Go, JavaScript (WASM)

**Benchmark (AMD Ryzen 9 5900X):**
| Metric | TEN VAD | Silero VAD |
|--------|---------|------------|
| RTF | 0.0150 | - |
| Lib Size | 306KB | 2.16MB |

**Valuable because:**
- Superior precision-recall performance
- Dramatically smaller binary size
- Fast enough for live applications with rapid transition detection

---

### Repo 4: Whisper Streaming

**URL:** https://github.com/ufal/whisper_streaming

**Description:** Real-time Whisper transcription using LocalAgreement policy

**Key Features:**
- Real-time transcription of long speech
- **3.3 second latency** on unsegmented long-form transcription
- Voice activity detection (VAD) and voice activity controller (VAC) options
- Supports faster-whisper, whisper-timestamped, OpenAI API, Whisper MLX backends

**Note:** In 2025, being replaced by SimulStreaming (faster, higher quality)

**Valuable because:**
- Production-proven real-time Whisper implementation
- LocalAgreement policy for quality + latency balance
- Multiple backend options for different deployment scenarios

---

### Repo 5: Picovoice Cheetah

**URL:** https://github.com/Picovoice/cheetah

**Description:** On-device streaming speech-to-text

**Key Features:**
- **On-device processing** — no network latency
- **590ms word emission latency** (vs Amazon Transcribe 920ms)
- Runs in web browsers, mobile, embedded, on-prem, or cloud
- Open-source benchmark shows Cheetah beats cloud providers on latency

**Valuable because:**
- True on-device STT for privacy-sensitive applications
- Web browser support enables JavaScript deployments
- Competitive accuracy with dramatically lower latency

---

## 4. Technical Reports & Documentation

### Report 1: Adaptive Interruption Handling — LiveKit

**URL:** https://livekit.io/blog/adaptive-interruption-handling

**Key Concepts:**
> "Traditional Voice Activity Detection (VAD) alone is insufficient — it triggers on backchannels ('mm-hmm', 'yeah'), user noises (sighs, coughs), and background sounds (typing, music), making agents feel jittery and robotic."

**Solution:** Audio-based interruption detection model combining:
- Audio encoder + Convolutional Neural Network (CNN)
- Trained on hundreds of hours of human-to-human conversations
- Multilingual and generalizes to unseen languages

**Availability:**
- Python Agents v1.5.0+, TypeScript Agents v1.2.0+
- LiveKit Cloud: included at no extra cost
- 40,000 free inference requests/month for local development

**Valuable because:**
- Solves the core "always interruptive" problem in voice agents
- Complements End-of-Turn detection (handles opposite direction)
- Production-tested with clear deployment path

---

### Report 2: Real-Time Speech-to-Speech APIs Comparison

**URL:** https://inworld.ai/resources/best-speech-to-speech-apis

**Key Comparisons:**

| Provider | API Type | Transport | Built-in VAD | Use Case |
|----------|----------|-----------|--------------|----------|
| **Inworld AI** | Realtime API | WebSocket, WebRTC | Yes | Voice agents, consumer AI |
| **OpenAI** | Realtime API | WebSocket | Built-in | Voice agents |
| **Deepgram** | Streaming STT + TTS | WebSocket | Optional | Low-latency apps |
| **AssemblyAI** | Realtime API | WebSocket | Yes | Transcription, voice agents |

**Key insight:** "The real-time or integrated speech-to-speech voice pipeline approach relies on a single multimodal AI model."

**Valuable because:**
- Current landscape of speech-to-speech API options
- Trade-off analysis between cascaded vs end-to-end approaches
- Pricing and use case guidance

---

### Report 3: PicoVoice — Local LLM-Powered Voice Assistant for Web Browsers

**URL:** https://picovoice.ai/blog/local-llm-powered-voice-assistant-for-web-browsers/

**Key Concepts:**
- **On-device voice AI** — voice processing and LLM inference performed locally
- **No third-party API** — user's data never leaves the device
- Works in Safari, Firefox, Chrome
- Uses Picovoice's Cheetah (STT) + Orca (TTS) + picoLLM

**Valuable because:**
- Complete on-device voice assistant architecture
- Web browser deployment without cloud dependency
- Privacy-preserving approach with full voice pipeline

---

### Report 4: I ditched cloud voice assistants for a local LLM

**URL:** https://www.howtogeek.com/how-a-local-llm-fixed-my-biggest-smart-home-privacy-problem/

**Key Concepts:**
> "Most smart speakers rely on the cloud, where every whispered command to a voice assistant is sent to remote servers for analysis and training, turning your house into a data collection point."

**Local LLM benefits:**
- Every byte of personal data stays on local network
- No latency from cloud round-trips
- No subscription or cloud dependency

**Valuable because:**
- Practical privacy implications documented
- Real-world migration path from cloud to local
- Smart home use case with concrete examples

---

## 5. Talks & Videos

### Video 1: Benchmarking LLMs for Voice Agent Use Cases

**URL:** https://www.youtube.com/watch?v=AjTDFvhTQdQ

**Description:** Discussion on creating benchmarks that accurately measure LLM performance for real-world voice agents

**Valuable because:**
- Industry perspective on voice agent evaluation
- Benchmark methodology for LLM + voice integration
- Real-world performance considerations

---

### Video 2: Talking to Your Local LLM in Real Time with .NET

**URL:** https://www.youtube.com/watch?v=zf3lfkeHCRA

**Description:** Building real-time AI conversations in .NET using Speech-to-Text (STT), Text-to-Speech (TTS), Voice Activity Detection (VAD)

**Valuable because:**
- .NET implementation patterns for real-time voice
- End-to-end pipeline demonstration
- Local LLM integration approach

---

### Video 3: Building Real-Time Voice AI with Python WebSockets

**URL:** https://www.youtube.com/watch?v=zghKpKGnw_A

**Description:** Python WebSockets + Deepgram for real-time Speech-to-Text

**Valuable because:**
- Python implementation patterns
- WebSocket streaming architecture
- Deepgram integration tutorial

---

## Key Concepts Summary

### Continuous Listening Modes

| Mode | Description | Pros | Cons |
|------|-------------|------|------|
| **Ambient/Always-On** | Microphone constantly processing | Frictionless, natural interaction | Privacy concerns, battery drain, false triggers |
| **Push-to-Talk** | User activates recording | Explicit control, privacy | Breaks conversational flow |
| **VAD-Triggered** | Recording starts on voice detection | Balances responsiveness and privacy | Requires accurate VAD, initial latency |

### Real-Time STT Pipeline

```
Microphone → Audio Chunk (20-100ms) → VAD → STT Streaming → LLM → TTS Streaming → Audio Output
```

**Latency targets:**
- Audio chunking: 20-100ms
- Time to First Token (TTFT): <300ms
- Time to Complete Turn (TTCT): <1 second
- Word Error Rate (WER): <10%

### VAD Solutions Comparison

| VAD | Latency | Size | License | Platforms |
|-----|---------|------|---------|-----------|
| **Silero VAD** | <1ms/chunk | ~2MB | MIT | All |
| **TEN VAD** | 0.008-0.015 RTF | 306KB | Open | Linux, Windows, macOS, Android, iOS, Web |
| **WebRTC VAD** | Low | Tiny | BSD | All |
| **FireRedVAD** | Variable | Variable | Open | Multi-language |

### Interruption Handling Approaches

1. **VAD-based:** Simple voice detection, high false-positive rate
2. **Adaptive ML models:** Audio encoder + CNN distinguishing true interruptions from backchannels
3. **LLM-based turn detection:** Prompt model to output single-token turn markers

---

## Cross-Cutting Themes

### Privacy Implications

**Always-on listening raises significant concerns:**
- Continuous audio capture creates constant surveillance risk
- Local processing (Picovoice, local LLMs) mitigates cloud dependency
- On-device VAD can reduce audio sent to cloud
- User consent mechanisms critical

**Mitigations:**
- Hardware-level mute switches
- Clear visual indicators when recording
- Local-first processing where possible
- Data minimization (VAD gates audio capture)

### Latency Considerations

**Pipeline stages and their latency contributions:**

| Component | Typical Latency | Optimization |
|-----------|-----------------|--------------|
| Audio capture | 20-100ms chunks | Smaller chunks |
| VAD | <10ms | Edge deployment |
| STT | 100-500ms | Streaming models, local inference |
| LLM | 200-2000ms | Streaming output, caching |
| TTS | 100-500ms | Streaming synthesis |
| **Total** | **500-3000ms** | Pipeline parallelism |

### Multimodal Integration Patterns

**Architecture patterns:**

1. **Cascaded:** ASR → LLM → TTS (modular, swap components)
2. **End-to-end:** Speech-to-Speech models (GPT-4o, Moshi)
3. **Hybrid:** Cascaded with tight synchronization

**Web UI implementations:**
- LiveKit Agents framework for real-time multi-modal
- Pipecat for conversational AI pipelines
- WebSocket streaming for custom implementations
- WebRTC for browser-based audio transport

### Key Research Gaps

1. **Interruption intent classification** — distinguishing why user interrupted
2. **Multilingual robustness** — most systems English-centric
3. **Privacy-preserving always-on** — local VAD + selective cloud processing
4. **Turn-taking timing** — MM-When2Speak addresses but limited to dyadic conversation
5. **Background noise adaptation** — real-world conditions degrade performance

---

## Source Index

| # | Title | URL | Type |
|---|-------|-----|------|
| 1 | Chat with the Environment (Matcha) | https://arxiv.org/abs/2303.08268 | Paper |
| 2 | MM-When2Speak | https://arxiv.org/html/2505.14654v1 | Paper |
| 3 | Interruption Handling for Conversational Robots | https://arxiv.org/abs/2501.01568v1 | Paper |
| 4 | Exploring Multimodal Perception in LLMs | https://arxiv.org/abs/2503.06980 | Paper |
| 5 | Building Enterprise Realtime Voice Agents | https://arxiv.org/html/2603.05413v1 | Paper |
| 6 | Realtime Speech Language Models | https://getstream.io/blog/realtime-speech-language-models/ | Blog |
| 7 | Meta Audio Real-time Communication | https://atscaleconference.com/advances-in-audio-real-time-communication/ | Blog |
| 8 | Ambient Listening vs Voice Dictation | https://sunoh.ai/blog/ambient-listening-technology-vs-voice-dictation/ | Blog |
| 9 | Real-Time Speech-to-Text Guide | https://www.assemblyai.com/blog/real-time-speech-to-text-best-for-voice-agents | Blog |
| 10 | Ambient Agents | https://medium.com/data-science-collective/ambient-agents-always-on-always-listening-always-working-2bba910fc3e8 | Blog |
| 11 | Architecting Voice Agents | https://dev.to/lifeisverygood/architecting-low-latency-real-time-ai-voice-agents-challenges-solutions-hdn | Blog |
| 12 | LiveKit Agents | https://github.com/livekit/agents | Repo |
| 13 | Silero VAD | https://github.com/snakers4/silero-vad | Repo |
| 14 | TEN VAD | https://github.com/ten-framework/ten-vad | Repo |
| 15 | Whisper Streaming | https://github.com/ufal/whisper_streaming | Repo |
| 16 | Picovoice Cheetah | https://github.com/Picovoice/cheetah | Repo |
| 17 | Adaptive Interruption Handling | https://livekit.io/blog/adaptive-interruption-handling | Doc |
| 18 | Speech-to-Speech APIs | https://inworld.ai/resources/best-speech-to-speech-apis | Doc |
| 19 | Local LLM Voice Assistant | https://picovoice.ai/blog/local-llm-powered-voice-assistant-for-web-browsers/ | Doc |
| 20 | Local vs Cloud Privacy | https://www.howtogeek.com/how-a-local-llm-fixed-my-biggest-smart-home-privacy-problem/ | Article |
