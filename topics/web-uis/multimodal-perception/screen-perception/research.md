# Screen Perception: Multimodal Perception Pattern for LLM Interactions

> Screen perception refers to the capability of AI agents to visually understand and interact with screen content through screenshots, enabling them to perceive GUIs the way humans do—as raw visual data rather than through DOM/HTML parsing.

## Table of Contents

- [Overview](#overview)
- [Academic Papers](#1-academic-papers-arxiv)
- [Blog Posts & Articles](#2-blog-posts--articles)
- [Open Source Repositories](#3-open-source-repos-github)
- [Benchmarks & Evaluation](#4-benchmarks--evaluation)
- [Commercial Implementations](#5-commercial-implementations)
- [Key Concepts](#key-concepts)
- [Summary](#summary)

---

## Overview

Screen perception is a multimodal pattern where LLM-based agents receive screenshot data as visual input, enabling them to understand and interact with graphical user interfaces without relying on DOM access or APIs. This approach mirrors human-computer interaction—observing the screen and executing mouse/keyboard actions.

**Why it matters:** Screen perception enables agents to work with *any* software interface without custom integrations, making AI automation truly universal.

---

## 1. Academic Papers (arXiv)

### 1.1 ScreenAI: A Vision-Language Model for UI and Infographics Understanding

**URL:** https://arxiv.org/abs/2402.04615  
**Venue:** IJCAI 2024  
**Authors:** Google Research

**Key Concepts:**
> "ScreenAI is a vision-language model specialized in UI and infographics understanding. Our model improves upon the PaLI architecture with a flexible patching strategy that preserves native aspect ratio of input images."

- Uses a **two-stage training pipeline**: self-supervised pre-training followed by fine-tuning with human-labeled data
- Introduces **Screen Annotation task** requiring models to identify UI element type, location, and description
- Achieves state-of-the-art on WebSRC and MoTIF benchmarks with only **5B parameters**

**Why it's valuable:**
- First dedicated VLM for UI understanding that handles both mobile/desktop UIs and infographics
- Provides scalable data generation pipeline using PaLM 2 to auto-generate QA and navigation training data
- Demonstrates that relatively small models (5B) can achieve strong UI understanding with proper training

---

### 1.2 ShowUI: One Vision-Language-Action Model for GUI Visual Agent

**URL:** https://arxiv.org/abs/2411.17465  
**Venue:** CVPR 2025  
**Authors:** Show Lab @ NUS, Microsoft

**Key Concepts:**
> "ShowUI is a Vision-Language-Action model that enhances GUI assistants by using UI-guided token selection and interleaved vision-language-action streaming, achieving high accuracy and efficiency in zero-shot screenshot grounding."

- **UI-Guided Visual Token Selection**: Formulates screenshots as UI-connected graphs to reduce computational costs (e.g., 1296→291 patches for sparse UIs)
- **Interleaved Vision-Language-Action Streaming**: Flexibly manages visual-action history for navigation tasks
- Lightweight **2B model** achieving **75.1% zero-shot screenshot grounding accuracy**

**Why it's valuable:**
- First open-source VLA (Vision-Language-Action) model specifically designed for GUI automation
- UI-guided token selection solves the computational bottleneck of processing high-resolution screenshots
- Outperforms much larger models by focusing on GUI-specific visual features

---

### 1.3 Ferret-UI: Grounded Mobile UI Understanding with Multimodal LLMs

**URL:** https://arxiv.org/abs/2404.05719  
**Venue:** ECCV 2024  
**Authors:** Apple

**Key Concepts:**
> "Ferret-UI is a new MLLM tailored for enhanced comprehension and interaction with mobile UI screens. Through careful design of training recipes and task formulations, Ferret-UI can understand both the overall screen and localized UI elements."

- MLLM specifically designed for **mobile UI understanding** (iPhone, Android)
- Introduces **elementary task data** including icon recognition, find text, widget listing
- Uses "any-resolution" approach adapting to varying aspect ratios across mobile devices

**Why it's valuable:**
- Apple's research contribution showing mobile-specific UI perception requires specialized models
- Demonstrates that mobile UIs have unique challenges (smaller elements, touch-focused) requiring dedicated approaches
- Released comprehensive training datasets for mobile UI tasks

---

### 1.4 Ferret-UI 2: Mastering Universal User Interface Understanding Across Platforms

**URL:** https://arxiv.org/html/2410.18967v1  
**Venue:** ICLR 2026 (under review)  
**Authors:** Apple

**Key Concepts:**
> "Ferret-UI 2 introduces three key innovations: support for multiple platform types (iPhone, Android, iPad, Webpage, AppleTV), high-resolution perception through adaptive scaling, and advanced task training data generation powered by GPT-4o with set-of-mark visual prompting."

- Extends Ferret-UI to **universal UI understanding** across all major platforms
- Uses **set-of-mark (SoM) visual prompting** for enhanced grounding
- **Dynamic high-resolution image encoding** adapts to varying resolutions without fixed constraints

**Why it's valuable:**
- First unified model handling desktop, mobile, tablet, and TV UIs simultaneously
- Adaptive scaling solves the resolution variation problem across platforms
- Cross-platform transfer capabilities demonstrated

---

### 1.5 You Only Look at Screens: Multimodal Chain-of-Action Agents (Auto-GUI)

**URL:** https://arxiv.org/html/2309.11436v4  
**Authors:** SJTU, Meta AI

**Key Concepts:**
> "We introduce Auto-GUI, a multimodal solution that directly interacts with the interface, bypassing the need for environment parsing or reliance on application-dependent APIs."

- **Chain-of-Action technique**: Uses previous action histories and future action plans to decide actions
- Operates purely through **vision-based input**—no DOM/HTML access required
- Achieves **74.27%** on AITW benchmark vs. GPT-4V's 52.96%

**Why it's valuable:**
- First work to demonstrate that pure vision-based GUI agents can outperform LLM-based approaches with tool use
- Introduced the chain-of-action mechanism for temporal reasoning in GUI tasks
- Open-source implementation enables reproducible research

---

### 1.6 ScreenLLM: Stateful Screen Schema for Efficient Action Understanding and Prediction

**URL:** https://arxiv.org/html/2503.20978v1  
**Authors:** 

**Key Concepts:**
> "ScreenLLM proposes stateful screen schema—a compact, informative textual representation of GUI interactions that captures key user actions and intentions over time."

- **Stateful Screen Schema**: Compact representation of GUI state changes over time
- **Key frame extraction** using second-order pixel changes to identify significant UI states
- Supports both open-source (LLaVA) and proprietary (GPT-4o) models

**Why it's valuable:**
- Addresses the scalability challenge of processing long screen recording sessions
- Shows that augmenting screenshots with structured state information improves action prediction by up to 47%
- Enables real-time GUI understanding by focusing on informative frames

---

### 1.7 OS Agents: A Survey on MLLM-based Agents for Computer, Phone and Browser Use

**URL:** https://arxiv.org/abs/2508.04482  
**Venue:** ACL 2025 Oral  
**Authors:** OS-Agent-Survey Team

**Key Concepts:**
> "OS Agents are (M)LLM-based Agents using computers, phones and browsers by operating within the environments and interfaces (e.g., GUI and CLI) provided by operating systems to automate tasks."

- Comprehensive survey of 100+ papers on OS agents
- Categorizes work into: **Foundation Models, Agent Frameworks, Evaluation Benchmarks, Safety & Privacy**
- Provides unified taxonomy of perception, planning, memory, and action components

**Why it's valuable:**
- Most complete overview of the field—essential starting point for researchers
- Identifies key architectural patterns across successful OS agents
- Highlights gaps and future research directions

---

## 2. Blog Posts & Articles

### 2.1 Anthropic's Announcement: Introducing Computer Use

**URL:** https://www.anthropic.com/news/3-5-models-and-computer-use

**Key Quotes:**
> "Available today on the API, developers can direct Claude to use computers the way people do—by looking at a screen, moving a cursor, clicking buttons, and typing text."

> "On OSWorld, which evaluates AI models' ability to use computers like people do, Claude 3.5 Sonnet scored 14.9% in the screenshot-only category—notably better than the next-best AI system's score of 7.8%."

**Why it's valuable:**
- First major commercial release of screen perception capability in a production LLM
- Establishes OSWorld as the benchmark for computer use evaluation
- Detailed safety considerations for deploying screen-perceiving agents

---

### 2.2 OpenAI's Computer-Using Agent (CUA)

**URL:** https://openai.com/index/computer-using-agent/

**Key Quotes:**
> "CUA is trained to interact with graphical user interfaces (GUIs)—the buttons, menus, and text fields people see on a screen—just as humans do."

> "CUA sets new state-of-the-art benchmark results across computer and web use tasks using a single general action space."

**Why it's valuable:**
- Achieved **38.1%** on OSWorld vs. Claude's 14.9% (later improved to 22% with more steps)
- Introduces the perception-reasoning-action loop as a standard pattern
- Available through Operator research preview for Pro users

---

### 2.3 ScreenAI: A Visual Language Model for UI Understanding

**URL:** https://research.google/blog/screenai-a-visual-language-model-for-ui-and-visually-situated-language-understanding/

**Key Concepts:**
> "ScreenAI leverages multimodal vision-language models to interpret and autonomously control GUI interactions across desktop, mobile, and tablet devices."

- Detailed technical explanation of the data generation pipeline
- Explains how UI annotations are used to generate synthetic training data for LLMs

**Why it's valuable:**
- Google's official blog explaining ScreenAI architecture and training methodology
- Insight into how large-scale synthetic UI data is generated
- Shows the path from research model to practical UI understanding

---

### 2.4 Claude Computer Use API: How Anthropic's Vision Model Controls Desktop and Browser

**URL:** https://callsphere.tech/blog/claude-computer-use-api-anthropic-vision-model-controls-desktop-browser

**Key Concepts:**
> "You no longer need to reverse-engineer a website's DOM structure, maintain fragile CSS selectors, or deal with shadow DOMs and iframes. Claude sees the screen the same way a human does."

- Detailed implementation guide for the screenshot-action feedback loop
- Explains coordinate system requirements and resolution matching

**Why it's valuable:**
- Practical code examples for implementing computer use
- Highlights common pitfalls (resolution mismatches, coordinate systems)
- Shows how to build production systems with Claude Computer Use

---

### 2.5 Understanding Multimodal LLMs

**URL:** https://magazine.sebastianraschka.com/p/understanding-multimodal-llms

**Key Concepts:**
> "Fuyu passes the input patches directly into a linear projection to learn its own image patch embeddings rather than relying on an additional pretrained image encoder."

- Technical deep-dive into multimodal LLM architectures
- Compares unified embedding decoder vs. cross-attention approaches

**Why it's valuable:**
- Excellent background for understanding how VLMs process visual information
- Helps understand trade-offs in different multimodal architectures for screen perception

---

### 2.6 Screenshot Understanding Agents

**URL:** https://www.arunbaby.com/ai-agents/0022-screenshot-understanding-agents/

**Key Concepts:**
> "Multimodal LLMs collapse the entire vision-detection-OCR pipeline into a single transformer that can read, reason about, and act on screenshots."

- Explains why screenshots are the "Final Boss" of vision tasks
- Discusses resolution bottlenecks and dynamic high-res tiling solutions

**Why it's valuable:**
- Accessible introduction to the challenges of screen perception
- Compares different VLM architectures for screenshot understanding

---

## 3. Open Source Repos (GitHub)

### 3.1 ShowUI

**URL:** https://github.com/showlab/ShowUI

**Description:** Open-source, end-to-end Vision-Language-Action model for GUI agents. Published at CVPR 2025.

**Key Features:**
- 2B parameter lightweight model
- UI-guided token selection for efficiency
- Supports Web, Mobile, Desktop platforms
- Available on HuggingFace: `showlab/ShowUI-2B`

**Why valuable:**
- First fully open-source VLA model for GUI automation
- Out-of-the-box computer use capability
- Comprehensive training code and datasets released

---

### 3.2 ScreenAgent

**URL:** https://github.com/niuzaisheng/ScreenAgent

**Description:** IJCAI-24 paper implementation. VLM agent that interacts with real computer screens via VNC.

**Key Features:**
- Planning-Execution-Reflection loop for multi-step tasks
- VNC-based universal action space (works across OS and apps)
- Supports GPT-4V, LLaVA-1.5, CogAgent, ScreenAgent models

**Why valuable:**
- Complete framework for building screen-perceiving agents
- VNC abstraction makes it OS-agnostic
- Includes training code and pre-trained models

---

### 3.3 Auto-GUI

**URL:** https://github.com/cooelf/Auto-GUI

**Description:** Implementation of "You Only Look at Screens" paper. Multimodal Chain-of-Action agents.

**Key Features:**
- Chain-of-action technique for temporal reasoning
- BLIP-2 based architecture
- Strong performance on AITW benchmark

**Why valuable:**
- Open research platform for vision-based GUI agents
- Ablation studies show importance of action history

---

### 3.4 OS-Agent-Survey

**URL:** https://github.com/OS-Agent-Survey

**Description:** ACL 2025 Oral survey paper repository with comprehensive paper list.

**Key Features:**
- 100+ papers categorized
- Unified taxonomy of OS agents
- Evaluation benchmarks comparison table

**Why valuable:**
- Most complete resource for OS agent research
- Regularly updated with new papers

---

### 3.5 Awesome-LLM-Powered-Phone-GUI-Agents

**URL:** https://github.com/PhoneLLM/Awesome-LLM-Powered-Phone-GUI-Agents

**Description:** Comprehensive list of papers, datasets, and frameworks for phone GUI automation.

**Key Features:**
- Taxonomy of LLM-powered phone GUI agents
- Milestones in development tracking
- Links to datasets and benchmarks

**Why valuable:**
- Focus specifically on mobile GUI automation
- Good complement to the broader OS-Agent-Survey

---

### 3.6 arbigent

**URL:** https://github.com/takahirom/arbigent

**Description:** AI Agent for testing Android, iOS, and web apps using screenshots.

**Key Features:**
- UI Tree optimization for AI comprehension
- Accessibility-independent screenshot annotations
- Multi-platform support

**Why valuable:**
- Practical tool for automated UI testing
- Shows commercial applications of screen perception

---

### 3.7 ClickAgent

**URL:** https://github.com/Samsung/ClickAgent

**Description:** Samsung's framework for autonomous UI agents using MLLM reasoning.

**Key Features:**
- MLLM handles reasoning and action planning
- Click detection as a specialized capability
- Refinement loop for error correction

**Why valuable:**
- Enterprise perspective on screen perception
- Shows integration with existing mobile frameworks

---

## 4. Benchmarks & Evaluation

### 4.1 OSWorld

**URL:** https://os-world.github.io/

**Description:** First-of-its-kind scalable, real computer environment for multimodal agents. Supports Ubuntu, Windows, and macOS.

**Key Stats:**
- 369 computer tasks involving real web and desktop apps
- Tasks span file I/O, multi-application workflows
- Gold standard for computer use evaluation

**Leaderboard (as of early 2026):**
| Agent | OSWorld Score |
|-------|---------------|
| UiPath Screen Agent (Claude Opus 4.5) | #1 |
| Claude 3.5 Sonnet (Computer Use) | 14.9% → 22% |
| OpenAI CUA | 38.1% |

**Why valuable:**
- Most realistic evaluation of computer use agents
- Tests full range of OS interactions

---

### 4.2 WebArena

**URL:** https://webarena.dev/

**Description:** Realistic web environment for autonomous agents. NeurIPS 2024 Oral.

**Variants:**
- **WebArena**: Self-hosted open-source websites
- **WebArena-Infinity**: Continuous evaluation in evolving environments
- **VisualWebArena**: Multimodal visual web tasks (ACL 2024)
- **WorkArena**: Knowledge worker tasks on ServiceNow

**Why valuable:**
- Tests web browsing agents in realistic, complex scenarios
- Multiple variants cover different aspects of web interaction

---

### 4.3 AITW (Android in the Wild)

**Description:** Large-scale dataset of Android UI interactions with 715K episodes.

**Key Stats:**
- Screens: 4.9M+
- Instruction types: 30+
- Covers Google Apps, Install, General, Single, WebShopping tasks

**Why valuable:**
- Standard benchmark for mobile GUI agents
- Used to train and evaluate most mobile UI models

---

## 5. Commercial Implementations

### 5.1 Anthropic Claude Computer Use

**API:** https://docs.anthropic.com/en/docs/build-with-claude/computer-use

**Capabilities:**
- Screenshot capture and analysis
- Mouse: click, double-click, triple-click, right-click, middle-click
- Keyboard: type, key press with modifiers
- Scroll: up/down at coordinates

**How it works:**
```
Screenshot → Claude analyzes → Returns tool call → Execute action → Repeat
```

**Why valuable:**
- First production-grade computer use API
- Integrated with Claude 3.5 Sonnet

---

### 5.2 OpenAI Operator / CUA

**URL:** https://operator.chatgpt.com/

**Capabilities:**
- Web browsing automation
- Multi-step task completion
- User confirmation for sensitive actions

**Safety Features:**
- Blocklist for sensitive sites
- CAPTCHAs trigger user confirmation
- Prompt injection detection

**Why valuable:**
- Consumer-facing agent powered by CUA
- API coming soon for developers

---

### 5.3 UiPath Screen Agent

**Description:** Enterprise RPA platform integrating Claude Opus 4.5 for desktop automation.

**Key Achievement:**
- #1 ranking on OSWorld-Verified benchmark
- Combines RPA reliability with AI screen perception

**Why valuable:**
- Shows enterprise-grade reliability requirements
- Bridges gap between experimental and production deployments

---

## Key Concepts

### Pixel-Level vs Semantic-Level Screen Understanding

| Level | Description | Use Case |
|-------|-------------|----------|
| **Pixel-Level** | Raw pixel data processed by vision encoders | Initial screen capture, OCR |
| **Semantic-Level** | UI elements identified (buttons, text, icons) | Action planning, navigation |
| **Functional-Level** | Element purpose understood (submit, cancel, search) | Task completion, reasoning |

Modern screen perception chains these together—VLMs process pixels → identify UI elements → reason about their purpose → plan actions.

---

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

---

### Resolution Considerations

**Critical rule:** Screenshot resolution **must match** the declared display dimensions in the API.

| Resolution | Declared | Coordinates Returned | Result |
|------------|----------|---------------------|--------|
| 1920×1080 | 1920×1080 | Correct | ✅ |
| 960×540 | 1920×1080 | 2x off | ❌ |

---

## Summary

Screen perception represents a fundamental shift in how AI agents interact with software—replacing DOM/API access with human-like visual understanding. Key findings:

1. **Academic Progress**: Models like ScreenAI (Google), Ferret-UI (Apple), and ShowUI (NUS/Microsoft) have established strong foundations for UI understanding with specialized VLMs.

2. **Benchmark Maturity**: OSWorld and WebArena provide rigorous evaluation frameworks, with current state-of-the-art achieving ~38% on OSWorld (vs. 72% human baseline)—significant room for improvement.

3. **Commercial Deployment**: Anthropic's Computer Use and OpenAI's CUA represent first-generation production systems, with enterprises like UiPath already achieving top benchmark scores.

4. **Open Source**: ShowUI, ScreenAgent, and Auto-GUI provide accessible platforms for research and development.

5. **Key Challenges**:
   - Efficiency: Processing high-resolution screenshots remains computationally expensive
   - Temporal reasoning: Tracking screen state changes across long interactions
   - Cross-platform transfer: UIs vary significantly across OS, apps, and devices

6. **Future Directions**:
   - Smaller, specialized models (ShowUI's 2B parameters suggest efficiency gains are possible)
   - Better temporal modeling (chain-of-action, stateful screen schema)
   - Unified cross-platform understanding (Ferret-UI 2's multi-platform approach)

---

## Citation

If you use this research summary, cite the original papers and resources.
