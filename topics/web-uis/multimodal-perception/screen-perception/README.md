# Screen Perception

> AI agents visually understanding and interacting with screen content through screenshots — perceiving GUIs the way humans do.

**Part of:** [[multimodal-perception]]

---

## Table of Contents

- [Overview](#overview)
- [Academic Papers](#academic-papers)
- [Open Source Repositories](#open-source-repositories)
- [Benchmarks & Evaluation](#benchmarks--evaluation)
- [Commercial Implementations](#commercial-implementations)
- [Key Concepts](#key-concepts)
- [Related Topics](#related-topics)

---

## Overview

**Screen perception** enables AI agents to visually understand and interact with graphical user interfaces through screenshot data — without DOM access, HTML parsing, or APIs.

> *"You no longer need to reverse-engineer a website's DOM structure, maintain fragile CSS selectors, or deal with shadow DOMs and iframes. Claude sees the screen the same way a human does."* — [Callsphere](https://callsphere.tech/blog/claude-computer-use-api-anthropic-vision-model-controls-desktop-browser)

**Why it matters:** Screen perception enables agents to work with *any* software interface without custom integrations — making AI automation truly universal.

---

## Academic Papers

### ScreenAI: Vision-Language Model for UI Understanding

**URL:** https://arxiv.org/abs/2402.04615 | **Venue:** IJCAI 2024 | **Authors:** Google Research

> *"ScreenAI is a vision-language model specialized in UI and infographics understanding. Our model improves upon the PaLI architecture with a flexible patching strategy that preserves native aspect ratio of input images."*

- Two-stage training: self-supervised pre-training + fine-tuning
- **5B parameters** achieving SOTA on WebSRC and MoTIF benchmarks
- Introduces Screen Annotation task for UI element identification

---

### ShowUI: Vision-Language-Action Model for GUI

**URL:** https://arxiv.org/abs/2411.17465 | **Venue:** CVPR 2025 | **Authors:** NUS, Microsoft

> *"ShowUI is a Vision-Language-Action model that enhances GUI assistants by using UI-guided token selection and interleaved vision-language-action streaming."*

- **2B parameters**, 75.1% zero-shot screenshot grounding accuracy
- UI-guided visual token selection (e.g., 1296→291 patches for sparse UIs)
- First open-source VLA model for GUI automation

---

### Ferret-UI: Mobile UI Understanding

**URL:** https://arxiv.org/abs/2404.05719 | **Venue:** ECCV 2024 | **Authors:** Apple

> *"Ferret-UI is a new MLLM tailored for enhanced comprehension and interaction with mobile UI screens."*

- Specialized for mobile UI (iPhone, Android)
- Elementary task data: icon recognition, find text, widget listing
- "Any-resolution" approach for varying aspect ratios

---

### Ferret-UI 2: Universal Cross-Platform UI

**URL:** https://arxiv.org/html/2410.18967v1 | **Venue:** ICLR 2026 (under review) | **Authors:** Apple

> *"Ferret-UI 2 introduces support for multiple platform types (iPhone, Android, iPad, Webpage, AppleTV), high-resolution perception through adaptive scaling, and advanced task training data generation powered by GPT-4o with set-of-mark visual prompting."*

- Universal UI understanding across desktop, mobile, tablet, TV
- Dynamic high-resolution image encoding
- Set-of-mark (SoM) visual prompting

---

### Auto-GUI: Chain-of-Action Agents

**URL:** https://arxiv.org/html/2309.11436v4 | **Authors:** SJTU, Meta AI

> *"We introduce Auto-GUI, a multimodal solution that directly interacts with the interface, bypassing the need for environment parsing or reliance on application-dependent APIs."*

- **74.27%** on AITW benchmark vs GPT-4V's 52.96%
- Chain-of-action mechanism for temporal reasoning
- BLIP-2 based architecture

---

### OS Agents Survey

**URL:** https://arxiv.org/abs/2508.04482 | **Venue:** ACL 2025 Oral

> *"OS Agents are (M)LLM-based Agents using computers, phones and browsers by operating within the environments and interfaces provided by operating systems."*

- Comprehensive survey of **100+ papers**
- Unified taxonomy: Foundation Models, Agent Frameworks, Evaluation Benchmarks, Safety & Privacy

---

## Open Source Repositories

### ShowUI

**URL:** https://github.com/showlab/ShowUI

- 2B parameter Vision-Language-Action model
- UI-guided token selection for efficiency
- HuggingFace: `showlab/ShowUI-2B`
- CVPR 2025, fully open-source

### ScreenAgent

**URL:** https://github.com/niuzaisheng/ScreenAgent

- IJCAI-24 implementation
- Planning-Execution-Reflection loop
- VNC-based universal action space
- Supports GPT-4V, LLaVA-1.5, CogAgent

### Auto-GUI

**URL:** https://github.com/cooelf/Auto-GUI

- Implementation of "You Only Look at Screens" paper
- Chain-of-action technique for temporal reasoning
- Strong performance on AITW benchmark

### OS-Agent-Survey

**URL:** https://github.com/OS-Agent-Survey

- ACL 2025 Oral survey repository
- 100+ papers categorized
- Unified taxonomy and evaluation benchmarks

### Awesome-LLM-Powered-Phone-GUI-Agents

**URL:** https://github.com/PhoneLLM/Awesome-LLM-Powered-Phone-GUI-Agents

- Comprehensive list of papers, datasets, frameworks
- Focus on mobile GUI automation
- Milestones tracking

---

## Benchmarks & Evaluation

### OSWorld

**URL:** https://os-world.github.io/

First-of-its-kind scalable, real computer environment (Ubuntu, Windows, macOS).

| Metric | Value |
|--------|-------|
| Tasks | 369 computer tasks |
| Platforms | Ubuntu, Windows, macOS |
| Human Baseline | 72% |

**Leaderboard (as of early 2026):**

| Agent | OSWorld Score |
|-------|---------------|
| UiPath Screen Agent (Claude Opus 4.5) | #1 |
| OpenAI CUA | 38.1% |
| Claude 3.5 Sonnet (Computer Use) | 14.9% → 22% |

---

### WebArena

**URL:** https://webarena.dev/ | **Venue:** NeurIPS 2024 Oral

Realistic web environment for autonomous agents.

**Variants:**
- **WebArena** — Self-hosted open-source websites
- **WebArena-Infinity** — Evolving environments
- **VisualWebArena** — Multimodal visual web tasks (ACL 2024)
- **WorkArena** — Knowledge worker tasks on ServiceNow

---

### AITW (Android in the Wild)

Large-scale Android UI interaction dataset.

| Metric | Value |
|--------|-------|
| Episodes | 715K+ |
| Screens | 4.9M+ |
| Instruction Types | 30+ |

---

## Commercial Implementations

### Anthropic Claude Computer Use

**URL:** https://docs.anthropic.com/en/docs/build-with-claude/computer-use

> *"Available today on the API, developers can direct Claude to use computers the way people do—by looking at a screen, moving a cursor, clicking buttons, and typing text."*

**Capabilities:**
- Screenshot capture and analysis
- Mouse: click, double-click, triple-click, right-click, middle-click
- Keyboard: type, key press with modifiers
- Scroll: up/down at coordinates

---

### OpenAI Operator / CUA

**URL:** https://openai.com/index/computer-using-agent/

> *"CUA is trained to interact with graphical user interfaces (GUIs)—the buttons, menus, and text fields people see on a screen—just as humans do."*

- **38.1%** on OSWorld (vs Claude's 14.9%)
- Perception-reasoning-action loop pattern
- Available through Operator research preview for Pro users

---

### UiPath Screen Agent

- #1 ranking on OSWorld-Verified benchmark
- Enterprise RPA platform integrating Claude Opus 4.5
- Bridges RPA reliability with AI screen perception

---

## Key Concepts

### Screenshot-Action Feedback Loop

```
┌─────────────────────────────────────────────────┐
│  1. Capture Screenshot                          │
│     (base64-encoded PNG)                        │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│  2. Send to VLM with Task                       │
│     (image + instruction)                       │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│  3. VLM Analyzes and Returns Action             │
│     (click x,y, type "text", scroll, etc.)      │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│  4. Execute Action on Real/Simulated Screen     │
│     (mouse, keyboard, scroll)                  │
└────────────────────┬────────────────────────────┘
                     ↓
┌─────────────────────────────────────────────────┐
│  5. Repeat until task complete                 │
└─────────────────────────────────────────────────┘
```

### Pixel-Level vs Semantic-Level Understanding

| Level | Description | Use Case |
|-------|-------------|----------|
| **Pixel-Level** | Raw pixel data processed by vision encoders | Initial screen capture, OCR |
| **Semantic-Level** | UI elements identified (buttons, text, icons) | Action planning, navigation |
| **Functional-Level** | Element purpose understood (submit, cancel, search) | Task completion, reasoning |

### Resolution Considerations

> **Critical rule:** Screenshot resolution **must match** the declared display dimensions in the API.

| Resolution | Declared | Result |
|------------|----------|--------|
| 1920×1080 | 1920×1080 | ✅ Correct |
| 960×540 | 1920×1080 | ❌ 2x off |

---

## Related Topics

- [[continuous-listening]] — Audio input patterns
- [[computer-use]] — Claude's computer use API
- [[os-agents]] — OS agent research
- [[web-agents]] — Web browsing agents
