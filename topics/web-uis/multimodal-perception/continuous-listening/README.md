# Continuous Listening

> Always-on audio input pattern for LLM interactions — enabling real-time speech processing without explicit activation.

**Part of:** [[multimodal-perception]]

---

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
- [Academic Papers](#academic-papers)
- [Open Source Repositories](#open-source-repositories)
- [Technical Reports & Articles](#technical-reports--articles)
- [Latency & Performance](#latency--performance)
- [Privacy Considerations](#privacy-considerations)
- [Related Topics](#related-topics)

---

## Overview

**Continuous Listening** is a multimodal perception pattern where an LLM-backed system maintains persistent audio processing, capturing and responding to speech in real-time without explicit push-to-talk activation.

**Core tension:** Ambient/passive listening enables frictionless interaction but raises significant privacy implications. Push-to-talk offers explicit control but breaks conversational flow.

> *"Traditional speech language models like Siri or Alexa use turn-taking as the primary interaction style — they cannot be interrupted in real time."* — [Stream.io](https://getstream.io/blog/realtime-speech-language-models/)

---

## Core Concepts

### Listening Modes

| Mode | Description | Pros | Cons |
|------|-------------|------|------|
| **Ambient/Always-On** | Microphone constantly processing | Frictionless, natural interaction | Privacy concerns, battery drain, false triggers |
| **Push-to-Talk** | User activates recording | Explicit control, privacy | Breaks conversational flow |
| **VAD-Triggered** | Recording starts on voice detection | Balances responsiveness and privacy | Requires accurate VAD, initial latency |

### Full-Duplex vs Half-Duplex

| Capability | Traditional SLM | Listening-While-Speaking LSLM |
|-----------|-----------------|------------------------------|
| Turn-based only | ✓ | ✗ |
| Real-time interruptions | ✗ | ✓ |
| Distinguish human voices from noise | ✗ | ✓ |
| Multi-modal integration | Limited | ✓ |

> *"Modern LLMs 'know what to say but not when to speak', missing within-utterance openings."* — [MM-When2Speak](https://arxiv.org/html/2505.14654v1)

---

## Academic Papers

### Interactive Multimodal Perception Using LLMs (Matcha)

**URL:** https://arxiv.org/abs/2303.08268 | **Venue:** IROS 2023

> *"LLMs can provide high-level planning and reasoning skills and control interactive robot behavior in a multimodal environment, while multimodal modules with the context of the environmental state help ground the LLMs."*

- First framework for LLM-as-backbone controlling multimodal (vision, sound, haptics, proprioception) robot interaction
- Introduces "Matcha" agent that chats with environment using epistemic actions

---

### MM-When2Speak: Multimodal Timing Prediction

**URL:** https://arxiv.org/html/2505.14654v1

> *"Modern LLMs 'know what to say but not when to speak', missing within-utterance openings."*

- Architecture: Video + Audio + Text → Self-Attention Fusion → Qwen2-7B LLM
- Predicts 9 response types: affirmation, gratitude, farewell, greeting, question, surprise, pondering, full_response, silence
- Achieves 4× better timing vs ChatGPT-4o text-only

---

### Interruption Handling for Conversational Robots

**URL:** https://arxiv.org/abs/2501.01568v1

> *"Interruptions can be used to signal understanding (cooperative agreement), assist the speaker (cooperative assistance), seek clarification when needed (cooperative clarification), or express disagreement and shift the topic (disruptive)."*

- Four interruption types: cooperative agreement, cooperative assistance, cooperative clarification, disruptive
- Validated with 21 participants in decision-making and discussion tasks

---

### Building Enterprise Realtime Voice Agents

**URL:** https://arxiv.org/html/2603.05413v1

Three architecture levels:
1. Text LLM with speech I/O
2. Speech-to-Speech models
3. Cascaded pipeline ASR→LLM→TTS

---

## Open Source Repositories

### LiveKit Agents

**URL:** https://github.com/livekit/agents | Apache License

- Framework for building realtime voice AI agents
- Adaptive Interruption Handling with trained audio-based interruption detection model
- Python and Node.js SDKs

### Silero VAD

**URL:** https://github.com/snakers4/silero-vad | MIT License

| Metric | Value |
|--------|-------|
| Speed | <1ms per audio chunk |
| Size | ~2MB |
| Languages | 6000+ |

### TEN VAD

**URL:** https://github.com/ten-framework/ten-vad

| Metric | TEN VAD | Silero VAD |
|--------|---------|------------|
| RTF | 0.0150 | - |
| Lib Size | 306KB | 2.16MB |

### Picovoice Cheetah

**URL:** https://github.com/Picovoice/cheetah

- On-device streaming speech-to-text
- **590ms word emission latency** (vs Amazon Transcribe 920ms)
- Works in web browsers, mobile, embedded, on-prem, or cloud

---

## Technical Reports & Articles

### Adaptive Interruption Handling — LiveKit

**URL:** https://livekit.io/blog/adaptive-interruption-handling

> *"Traditional Voice Activity Detection (VAD) alone is insufficient — it triggers on backchannels ('mm-hmm', 'yeah'), user noises (sighs, coughs), and background sounds (typing, music), making agents feel jittery and robotic."*

**Solution:** Audio-based interruption detection model combining audio encoder + CNN

### Meta Audio Stack (@Scale)

**URL:** https://atscaleconference.com/advances-in-audio-real-time-communication/

Meta's multilayered audio AI stack:
1. **Voice Clarity Detector** — Near/far speaker detection using GMM
2. **Primary Speaker Segmentation** — TDNN backbone with causal multi-scale convolutions
3. **Echo Control** — Client-side AEC + server-side echo control
4. **Endpointer** — Uses semantic information from user "turn end"

> *"Unlike human-to-human communication (where there is little to no tolerance for artifacts), in human-to-machine communication, a small degree of distortions can be accepted to ensure the user's voice is always prioritized."*

### Real-Time Speech-to-Text: Critical Metrics

**URL:** https://www.assemblyai.com/blog/real-time-speech-to-text-best-for-voice-agents

| Metric | Target |
|--------|--------|
| **WER** (Word Error Rate) | <10% |
| **TTFT** (Time to First Token) | <300ms |
| **RTF** (Real-Time Factor) | <0.5 |
| **TTCT** (Time to Complete Turn) | <1 second |

> *"If the transcription is wrong, the LLM responds to the wrong thing."*

---

## Latency & Performance

### Real-Time STT Pipeline

```
Microphone → Audio Chunk (20-100ms) → VAD → STT Streaming → LLM → TTS Streaming → Audio Output
```

### VAD Solutions Comparison

| VAD | Latency | Size | License |
|-----|---------|------|---------|
| **Silero VAD** | <1ms/chunk | ~2MB | MIT |
| **TEN VAD** | 0.008-0.015 RTF | 306KB | Open |
| **WebRTC VAD** | Low | Tiny | BSD |

### Interruption Handling Approaches

1. **VAD-based** — Simple voice detection, high false-positive rate
2. **Adaptive ML models** — Audio encoder + CNN distinguishing true interruptions from backchannels
3. **LLM-based turn detection** — Prompt model to output single-token turn markers

---

## Privacy Considerations

> *"Most smart speakers rely on the cloud, where every whispered command to a voice assistant is sent to remote servers for analysis and training, turning your house into a data collection point."* — [HowToGeek](https://www.howtogeek.com/how-a-local-llm-fixed-my-biggest-smart-home-privacy-problem/)

**Mitigations:**
- Hardware-level mute switches
- Clear visual indicators when recording
- Local-first processing (Picovoice, local LLMs)
- Data minimization (VAD gates audio capture)

---

## Related Topics

- [[screen-perception]] — Visual input patterns for AI agents
- [[ambient-agents]] — Always-on agent architectures
- [[voice-activity-detection]] — VAD technologies
- [[speech-to-speech]] — End-to-end speech models
