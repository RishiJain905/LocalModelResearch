# Explainable Rationale — Agentic UX Pattern Research

> **Pattern Name:** Explainable Rationale
> **Aliases:** Visible Thinking, Stream of Thought, Chain-of-Thought Display, Reasoning Transparency
> **Context:** Agentic UX — LLM-powered AI agents that plan, act, and delegate autonomously
> **Problem:** Users experience AI agents as opaque black boxes; trust erodes when reasoning is hidden

---

## Table of Contents

1. [Definition](#definition)
2. [Academic Papers](#1-academic-papers-arxiv)
3. [Blog Posts & Articles](#2-blog-posts--articles)
4. [Open Source Repositories](#3-open-source-repos-github)
5. [Technical Reports & Documentation](#4-technical-reports--documentation)
6. [Talks & Videos](#5-talks--videos)
7. [Cross-Cutting Themes](#cross-cutting-themes)

---

## Definition

**Explainable Rationale** is an agentic UX pattern where an AI system surfaces its reasoning process — goals, constraints, intermediate decisions, tool calls, and evidence — in human-readable form before, during, or after task execution. The pattern directly answers the user's *"Why did it do that?"* and *"How did it reach that conclusion?"* questions.

> *"The AI's Stream of Thought reveals the visible trace of how it navigated from input to answer. This includes: the plan it formed, tools called, code executed, checks and decisions made along the way."*
> — ShapeofAI.com, "Stream of Thought" Pattern

Three canonical expressions:
- **Human-readable plans** — preview what the AI will do before acting
- **Execution logs** — record tool calls, code, and results as they happen
- **Compact summaries** — capture logical reasoning, insights, and decisions post-hoc

**Relation to CoT:** Chain-of-Thought (CoT) prompting elicits step-by-step reasoning *from the model*; Explainable Rationale surfaces that reasoning *to the user*. CoT is a model behavior; Explainable Rationale is a UX design pattern that makes CoT (or any reasoning) visible.

---

## 1. Academic Papers (arXiv)

### Paper 1: "Watching AI Think: User Perceptions of Visible Thinking in Chatbots"

- **URL:** https://arxiv.org/html/2601.16720v1
- **Authors:** arXiv preprint
- **Year:** 2025

**Key Quotes/Concepts:**

> *"Both the presence and framing of visible thinking significantly shape user perceptions — emotionally-supportive thinking increases warmth and empathy, while expertise-supportive thinking increases trust and competence."*

> *"No visible thinking leads to perceptions of impersonal, unengaging interaction."*

- Study of 234 participants across emotionally-supportive vs. expertise-supportive thinking displays
- Emotionally-supportive thinking utterances led to highest Emotional Responsiveness scores
- Expertise-supportive thinking led to highest Understanding & Trust scores
- Feeling-related contexts rated higher than habit-related contexts for all empathy measures

**Why It's Valuable:**
- Provides **empirical evidence** that visible thinking UX directly impacts user trust and engagement
- Distinguishes between **emotional vs. expertise framing** of thinking displays — different goals need different approaches
- Gives concrete design guidance: thinking content type should match the conversational context

---

### Paper 2: "Interactive Reasoning: Visualizing and Controlling Chain-of-Thought Reasoning in Large Language Models"

- **URL:** https://arxiv.org/html/2506.23678v1
- **Authors:** arXiv preprint
- **Year:** 2025

**Key Quotes/Concepts:**

> *"Hippo allows users to quickly identify and interrupt erroneous generations, efficiently steer the model towards customized responses, and better understand both model reasoning and model outputs."*

> *"Participants valued transparency of viewing reasoning chains and ability to repurpose model response."*

> *"User agency improves perceived output quality — participants felt their 'voice was heard'; outputs felt personalized."*

- Introduces **Hippo** — an interactive system visualizing CoT as a hierarchy of topics (tree structure)
- Enables direct manipulation of reasoning nodes: add, edit, delete, collapse subtrees
- User study (N=16) showed significant improvements in **Control, Sense-making, Layout, Awareness, and Confidence**
- Key finding: users rarely edited mid-generation; most edits happened after completion
- Model reasoning may be more valuable than final output — users gained understanding through reasoning alone

**Why It's Valuable:**
- First **controlled user study** of interactive reasoning UIs with quantitative metrics
- Proposes **tree data structure** for representing corrections (branching when model self-corrects)
- Shows that **user control over reasoning** (not just output) drives trust and quality perception

---

### Paper 3: "From Features to Actions: Explainability in Traditional and Agentic AI Systems"

- **URL:** https://arxiv.org/html/2602.06841v1
- **Authors:** arXiv preprint
- **Year:** 2025

**Key Quotes/Concepts:**

> *"Traditional explainability methods designed for static predictions are inadequate for agentic AI systems where behavior emerges over multi-step trajectories."*

> *"State tracking inconsistency is 2.7× more prevalent in failed vs. successful agent runs."*

> *"The Minimal Explanation Packet (MEP) packages: (1) Explanation Artifact, (2) Linked Evidence, (3) Verification Signals."*

- Compares static (SHAP/LIME) vs. agentic explainability systematically
- Introduces **Minimal Explanation Packet (MEP)** framework for agentic contexts
- Empirically shows: traditional attribution methods achieve Spearman ρ=0.86 stability in static settings but don't transfer to multi-step agent trajectories
- **State tracking consistency** is the single strongest predictor of agent success (2.7× failure rate when violated)
- RR=0.51: state consistency violations reduce success rate to 51% of baseline

**Why It's Valuable:**
- Establishes that **trajectory-level explanation** (not feature attribution) is needed for agents
- Provides quantitative evidence that **state tracking transparency** is critical for trust
- MEP framework gives a concrete, implementable structure for agentic explanation packets

---

### Paper 4: "Chain-of-Thought Is Not Explainability"

- **URL:** https://aigi.ox.ac.uk/wp-content/uploads/2025/07/Cot_Is_Not_Explainability.pdf
- **Authors:** Fazl Barez et al. (Oxford, AI2, Mila, Google DeepMind, others)
- **Year:** 2025

**Key Quotes/Concepts:**

> *"Chains-of-thought can appear persuasive even when they do not faithfully reflect a model's actual decision process."*

> *"LLMs deploy multiple parallel pathways of answer generation for step-by-step reasoning. The sequential CoT is at best a selective and often lossy projection of distributed computation."*

> *"The portion of CoT interpretability claim does not show a declining trend over time."* (Across 1,000 arXiv papers)

- Systematic analysis showing CoT is **unfaithful by design** — it rationalizes rather than explains
- Failure modes: **bias-driven rationalization** (answer chosen by subtle prompt cues, CoT doesn't mention), **silent error correction** (model corrects internally without reflecting in CoT)
- On hard math problems, models insert "non-sensical simplifications" yet output correct answers
- Research methods to improve faithfulness: causal validation (black-box, grey-box, white-box), cognitive science-inspired error monitoring

**Why It's Valuable:**
- **Critical counterbalance** — warns that visible CoT ≠ genuine explainability
- Gives designers and engineers a **faithfulness evaluation framework**
- Essential reading before deploying CoT-as-explanation in production UX

---

### Paper 5: "The Cost of Dynamic Reasoning: Demystifying AI Agents and Test-Time Scaling"

- **URL:** https://arxiv.org/html/2506.04301v2
- **Authors:** arXiv preprint
- **Year:** 2025

**Key Quotes/Concepts:**

> *"Unlike static reasoning models that process a user request with a single LLM inference step, AI agents perform multiple reasoning steps iteratively, introducing new challenges for efficient serving."*

> *"An unconstrained agent can cost $5 to $8 dollars per task to solve a software engineering issue."*

- Quantifies the **cost/latency cost** of iterative agent reasoning
- Agents perform dozens of inference calls per task vs. single-shot LLM
- Prefix caching reduces KV cache size but agents still face memory/energy constraints
- 5–8× cost difference between constrained and unconstrained agent task completion

**Why It's Valuable:**
- Makes the **cost dimension explicit** — reasoning visibility must account for token/cost transparency
- Provides concrete cost benchmarks for agents doing multi-step reasoning
- Supports designing **cost/latency disclosure** as part of the Explainable Rationale pattern

---

## 2. Blog Posts & Articles

### Article 1: "Design Patterns For AI Interfaces" — Smashing Magazine

- **URL:** https://www.smashingmagazine.com/2025/07/design-patterns-ai-interfaces/
- **Author:** Editorial team
- **Date:** 2025

**Key Concepts:**
- Documents the emerging vocabulary of AI-specific design patterns
- Covers: conversational AI patterns, streaming output, progressive disclosure
- Notes that agents shift users from "chatting" to **orchestrating** AI work

**Why It's Valuable:**
- Establishes **terminology and pattern taxonomy** for the broader field
- Contextualizes Explainable Rationale within the pattern landscape

---

### Article 2: "Designing For Agentic AI: Practical UX Patterns For Control, Consent, And Accountability"

- **URL:** https://www.smashingmagazine.com/2026/02/designing-agentic-ai-practical-ux-patterns/
- **Author:** Victor Yocco, PhD (UX Researcher at ServiceNow)
- **Date:** February 2026

**Key Quotes:**

> *"Autonomy is an output of a technical system, but trustworthiness is an output of a design process."*

> *"The single most powerful mechanism for building user confidence is the ability to easily reverse an agent's action."*

> *"Explainable Rationale: proactively answers the user's 'Why did it do that?' with a concise justification."*

**Pattern Framework (6 patterns across 3 phases):**

| Phase | Patterns |
|-------|----------|
| **Pre-Action** | Intent Preview, Autonomy Dial |
| **In-Action** | **Explainable Rationale**, Confidence Signal |
| **Post-Action** | Action Audit & Undo, Escalation Pathway |

- **Explainable Rationale** best practices: grounded in precedent, simple and direct, use "Because you said X, I did Y" structure
- Confidence signals should surface the agent's own certainty about its plan
- Metrics: track "Why?" support ticket volume per 1,000 users

**Why It's Valuable:**
- Defines **Explainable Rationale as a named, standalone UX pattern** with clear design guidelines
- Gives concrete **implementation anatomy** (when to use, risk of omission, success metrics)
- Connects to the broader 6-pattern framework across the agent lifecycle

---

### Article 3: "AI UX Patterns | Stream of Thought" — ShapeofAI.com

- **URL:** https://www.shapeofai/patterns/stream-of-thought
- **Author:** Emily Campbell
- **License:** CC-BY-NC-SA

**Key Quotes:**

> *"When this information is visible, the system becomes more legible and easier to trust."*

> *"Tailor visibility to the context. Users running complicated, computer-heavy tasks will look for full traces and logic, while users in simple conversations may require little-to-no visibility."*

> *"Treat every step as a clear state: Queued, Running, Waiting for approval, Error, Retried, Completed."*

**Design Principles:**
- Show the plan **before** acting — editable sequence of steps
- Separate Plan, Execution, and Evidence into three synchronized views
- Tailor visibility to context (mode vs. intensity)
- Make steps into **states** with progress cues

**Examples by Product:** Accio, ChatGPT, Grok, Lovable, Perplexity, v0

**Why It's Valuable:**
- Gives the most **comprehensive UX design guidance** for this pattern
- Provides a **product comparison table** showing real-world implementations
- Emphasizes **progressive disclosure** — visibility should match task complexity

---

### Article 4: "How AI Models Show Their Reasoning Process in Real-Time" — Digestible UX

- **URL:** https://www.digestibleux.com/p/how-ai-models-show-their-reasoning
- **Date:** March 2025

**Key Quotes:**

> *"AI transparency isn't about revealing everything — it's about showing the right reasoning at the right time to build trust without overwhelming users."*

> *"More transparency ≠ Better UX. AI should explain itself, but too much detail can be overwhelming."*

**Model Comparison Table:**

| Model | Transparency | Structure | Cognitive Load |
|-------|-------------|-----------|----------------|
| DeepSeek R1 | Most transparent | Low | High |
| Gemini 2.0 | High | High | Medium |
| Grok 3 | Medium | Low | High |
| Claude 3.7 | Minimal (on-demand) | High | Low |
| ChatGPT o3-mini | Minimal (on-demand) | Medium | Low |

**Why It's Valuable:**
- **Comparative UX analysis** across 5 major reasoning model interfaces
- Demonstrates the **transparency vs. cognitive load trade-off** empirically
- Actionable insight: **structured, collapsible reasoning** (Claude style) may outperform maximally transparent unstructured displays (DeepSeek style)

---

### Article 5: "Explainable AI in Chat Interfaces" — Nielsen Norman Group

- **URL:** https://www.nngroup.com/articles/explainable-ai/
- **Author:** Megan Chan
- **Date:** December 2025

**Key Quotes:**

> *"Explanations currently offered by LLMs are often inaccurate, hidden, or confusing."*

> *"The more users trust AI, the more likely they are to use AI tools without question."*

> *"Despite users' stated intention to verify AI citations, research shows people rarely click citation links."*

**Three Types of Explanation Text:**
1. **Source Citations** — often hallucinated; link to relevant passage, not full document
2. **Step-by-Step Reasoning** — frequently unfaithful; risks promoting overtrust
3. **Disclaimers** — effective only if prominent, plain-language, action-oriented

**Why It's Valuable:**
- **Human factors research grounding** — explains why explanations succeed or fail from a cognitive perspective
- Challenges the assumption that more explanation = better UX
- Warns against **anthropomorphic language** (e.g., "I thought about your problem") which inflates trust inappropriately

---

### Article 6: "Build Reasoning UIs with DeepSeek R1: Visualize Chain-of-Thought" — SitePoint

- **URL:** https://www.sitepoint.com/build-reasoning-uis-with-deepseek-r1-visualize-chainofthought-2026/
- **Date:** 2026

**Key Quotes:**

> *"DeepSeek R1 changed that calculus by exposing reasoning tokens as a first-class output, separate from the final answer."*

> *"The end result is an interactive interface where users submit a prompt, watch R1's reasoning steps appear as an animated vertical flow, see self-corrections rendered visually."*

**Technical Implementation:**
- DeepSeek API returns two fields: `reasoning_content` (intermediate) and `content` (final)
- Reasoning tokens arrive **first** in the stream
- Data model uses **tree structure** (not flat list) to represent corrections
- `parentId` enables branching when model self-corrects
- Step transitions align with numbered markers or logical pivots

**Why It's Valuable:**
- **Complete code implementation** for building a DeepSeek R1 reasoning UI
- Introduces tree data model for representing **model self-correction as branching**
- Shows how `reasoning_content` field enables real-time chain-of-thought visualization

---

### Article 7: "AI Agent Observability, Tracing & Evaluation with Langfuse" — Langfuse Blog

- **URL:** https://langfuse.com/blog/2024-07-ai-agent-observability-with-langfuse
- **Date:** 2024

**Key Quotes:**

> *"Agents use multiple steps to solve complex tasks, and inaccurate intermediary results can cause failures of the entire system."*

> *"The tradeoff between accuracy and costs is crucial — higher accuracy often leads to increased operational expenses."*

> *"Often, the agent decides autonomously how many LLM calls or paid external API calls it needs to make, potentially leading to high costs for single-task executions."*

**Core Concepts:**
- **5 Agent Components:** Language Model, Planning Module, Action Module, Memory Module, Profile Module
- **Observability scope:** real-time monitoring of multiple LLM calls, control flows, decision-making, outputs
- **OpenTelemetry (OTEL)** as the industry standard for agent telemetry
- Langfuse supports tracing for LangGraph, OpenAI Agents SDK, Pydantic AI, CrewAI, and more

**Why It's Valuable:**
- Frames **observability as the engineering counterpart** to the Explainable Rationale UX pattern
- Connects reasoning visibility to **cost/latency transparency** (agent autonomy → unpredictable costs)
- Lists concrete frameworks with Langfuse integration guides

---

## 3. Open Source Repos (GitHub)

### Repo 1: `lutzfinger/how-does-AI-think`

- **URL:** https://github.com/lutzfinger/how-does-AI-think
- **Stars:** Not specified
- **License:** Not specified
- **Stack:** Python, Streamlit

**What It Does:**
A **glass-box teaching tool** — a Streamlit demo that exposes agent internals step-by-step. Users click through each workflow stage (Planning, Execution, Memory, Tools, Quality Checks, Policy/Guarardrails, Graph Layer, Logs/Trace).

**Key Quotes:**

> *"How does an AI agent actually work when we make its internal system visible? A useful agent depends on workflow structure, memory boundaries, tool design, approvals, validations, fallbacks, and observability."*

**11 Enterprise-Readiness Patterns:**
- Structured, atomic workflows
- Assistive before autonomous (pause often, gate risky actions)
- Least-privileged tool access
- Evaluations as first-class
- **Observable by default** — plan state, tool usage, memory, checks all inspectable
- Explicit failure handling
- Stateful memory separation
- **Grounded outputs with provenance**
- Human approval for high-risk
- Validation vs. policy separation
- Cost-efficient execution

**Why It's Valuable:**
- Maps the **full surface area** of what "explainable" means in an agent context
- The "observable by default" and "grounded outputs with provenance" principles directly inform Explainable Rationale UX

---

### Repo 2: `Ngxba/claude-code-agents-ui`

- **URL:** https://github.com/Ngxba/claude-code-agents-ui
- **License:** MIT
- **Stack:** Nuxt 3, Bun, Tailwind CSS, TypeScript, Anthropic SDK

**What It Does:**
Visual dashboard for managing Claude Code agents. Provides relationship graphs, real-time metrics, visual workflow builders, and terminal emulation. Streams agent thinking, tool use, and text deltas via SSE.

**Key Features:**
- **Real-time Context Monitoring** — tokens, costs, file changes, tool calls live
- **SSE-powered Agent Studio** — streams thinking and tool use as they happen
- **Visual Relationship Mapping** — auto-detects links between agents, commands, skills
- Works directly with `~/.claude` directory — no separate database

**Why It's Valuable:**
- Production-grade example of **real-time reasoning display** in a UI
- Demonstrates streaming of both thinking content AND tool calls (multi-channel transparency)
- Shows integration with cost tracking (tokens, pricing visible in real-time)

---

### Repo 3: `hoodini/llm-visuals`

- **URL:** https://github.com/hoodini/llm-visuals
- **License:** Not specified
- **Stack:** Node.js, npm

**What It Does:**
A **reverse proxy + dashboard** that sits between LLM clients and LLM APIs. Intercepts all traffic, displays full request/response payloads, provides real-time KPIs, latency, duration, cost tracking, and context chain visualization.

**Key Quotes:**

> *"Zero code changes. Zero certificates. Just one environment variable per provider."*

**Architecture:** `LLM Client → LLM Visuals Proxy (localhost:4000) → LLM API`

**Supported Tools:** Claude Code, Cursor, Aider, Continue, GitHub Copilot (BYOK), any OpenAI-compatible SDK (LiteLLM, LangChain)

**What You Can See:**
- Full Request Inspector — every token, hidden system prompt, every tool call
- **Real-Time Metrics** — KPIs, latency, duration, cost tracking
- **Context Chain Visualization** — full conversation chain for any request

**Why It's Valuable:**
- Demonstrates that **observability infrastructure** (reverse proxy tracing) is a complement to UX-level explanation
- Shows how to expose token counts and cost per call — enabling the cost transparency dimension of Explainable Rationale
- Zero-integration approach means it can wrap any existing agentic application

---

### Repo 4: `langfuse/skills`

- **URL:** https://github.com/langfuse/skills
- **License:** Not specified
- **Stack:** Claude Code, Cursor skills

**What It Does:**
**Agent Skills** that teach AI coding assistants (Claude Code, Cursor, etc.) how to work with Langfuse. Provides skills for querying/ managing traces, prompts, datasets, and scores via the Langfuse API.

**Skills Available:**
- `langfuse` skill — query traces, manage prompts, look up documentation
- Installs as a Cursor plugin or via `skills` CLI
- Enables agents to **read their own traces** and self-debug

**Why It's Valuable:**
- Shows how to make Langfuse tracing accessible **within the agent's own workflow** (agent reads its reasoning traces)
- Practical demonstration of **trace-as-explanation** — the agent's own execution log serves as its rationale

---

## 4. Technical Reports & Documentation

### Doc 1: OpenAI "Reasoning Models" API Documentation

- **URL:** https://developers.openai.com/api/docs/guides/reasoning
- **Provider:** OpenAI

**Key Technical Details:**

> *"Reasoning models like o3 and o4-mini use internal reasoning tokens before producing a response."*

> *"Reasoning models work better with the Responses API."*

- Models: o3, o4-mini (and variants)
- Reasoning happens **internally** (not user-visible by default)
- `reasoning_effort` parameter controls thinking depth
- `max_output_tokens` manages total cost by limiting combined reasoning + output tokens
- Some reasoning models support **reasoning summaries** (abbreviated reasoning visible via API)

**For UX Implementors:**
- DeepSeek-R1 exposes `reasoning_content` field directly in API response
- OpenAI reasoning content is **hidden by default**; requires specific model configuration to surface
- Cost control is built into the API design (`max_output_tokens`)

**Why It's Valuable:**
- **API-level reference** for what reasoning models expose vs. hide
- Confirms that reasoning token exposure varies by provider — a key constraint for UX designers

---

### Doc 2: DeepSeek API "Reasoning Model" Guide

- **URL:** https://api-docs.deepseek.com/guides/reasoning_model
- **Provider:** DeepSeek

**Key Technical Details:**

> *"DeepSeek-R1 reasoning tokens arrive first in the stream. Only after reasoning completes does `content` begin populating."*

- Model identifier: `deepseek-reasoner`
- Response contains: `reasoning_content` (intermediate) and `content` (final answer)
- Multi-round conversation: remove `reasoning_content` from API response before making the next request
- Supported features: JSON Output, Chat Completion, Function Calling

**Why It's Valuable:**
- DeepSeek is currently the **only major model that exposes reasoning tokens as a first-class API field**
- This is the **technical foundation** for all DeepSeek R1 reasoning UIs (like the SitePoint tutorial)
- Confirms streaming order: reasoning tokens → final content (enables real-time display)

---

### Doc 3: Langfuse Documentation (General)

- **URL:** https://langfuse.com/
- **Provider:** Langfuse (open source, MIT license)
- **Founded:** 2023; acquired by ClickHouse January 2026

**Key Capabilities for Explainable Rationale:**

| Capability | How It Supports Rationale Visibility |
|------------|-------------------------------------|
| **Tracing** | Captures every LLM call, tool execution, retrieval step |
| **Prompt Management** | Version and compare CoT prompt formulations |
| **Datasets** | Test edge cases; benchmark reasoning quality |
| **Evaluation** | LLM-as-judge metrics assess reasoning faithfulness |
| **Scores** | Attach quality scores to reasoning traces |
| **Self-hosting** | Data sovereignty — critical for enterprise trust |

**Langfuse vs. LangSmith (for context):**

| Dimension | Langfuse | LangSmith |
|-----------|----------|-----------|
| License | Open source (MIT) | Proprietary SaaS |
| Self-hosting | First-class citizen | Enterprise only |
| Framework | Agnostic | LangChain/LangGraph native |
| Pricing | Units-based (cheaper at scale) | Per-seat + trace volume |

**Why It's Valuable:**
- The **primary open-source tool** for implementing agent reasoning observability
- Framework-agnostic — works with LangGraph, OpenAI Agents SDK, CrewAI, Pydantic AI, etc.
- Tracing + scoring + datasets = the **engineering infrastructure** for Explainable Rationale

---

## 5. Talks & Videos

### Talk 1: "Stanford Seminar — Human-Centered Explainable AI"

- **URL:** https://www.youtube.com/watch?v=ZxPV_KVq-tI
- **Event:** Stanford HCI Seminar
- **Speaker:** Q Vera Liao (Microsoft Research)
- **Date:** February 2023

**Topic:** Human-Centered Explainable AI (XAI) — how to design explanations that help users make better decisions, not just feel informed.

**Why It's Valuable:**
- Foundational **HCI perspective** on explainability from a leading researcher
- Addresses the gap between "explanation available" and "explanation useful"

---

### Talk 2: "(Un)Reliability of Chain-of-Thought Reasoning" — Chirag Agarwal

- **URL:** https://www.youtube.com/watch?v=nqZ6EiPltSo
- **Event:** University of Virginia / public talk
- **Speaker:** Chirag Agarwal

**Topic:** The (Un)Reliability of Chain-of-Thought Reasoning — empirical evidence that CoT explanations are frequently unfaithful.

**Why It's Valuable:**
- Visual presentation of the CoT faithfulness research (Turpin et al., etc.)
- Complements the Oxford paper with **oral presentation** of the same findings

---

### Talk 3: "How AI Agents Reason and Act"

- **URL:** https://www.youtube.com/watch?v=1clv5-5ib38
- **Topic:** AI Agents & AI Reasoning Models

**Key Quote:**

> *"Memory and context set the stage — but cognition is where AI agents stop being reactive and start acting with intent."*

**Why It's Valuable:**
- Accessible overview of agent reasoning architecture
- Frames reasoning transparency as part of the broader **agent cognition** narrative

---

### Talk 4: "AI Agent UX Patterns: Lessons From 1000 Startups" — Jonas Braadbaart

- **URL:** https://www.youtube.com/watch?v=3kiTAGa7VFI
- **Topic:** Survey of UX patterns in agentic AI applications from startup landscape

**Why It's Valuable:**
- Empirical observation of what **actual products** are doing — pattern catalog from real deployments
- Likely covers reasoning visibility as a recurring theme

---

## Cross-Cutting Themes

### Theme 1: The Transparency–Cognitive Load Trade-off

Every source grapples with the same fundamental tension: more reasoning visibility builds trust, but too much overwhelms users.

| Approach | Who Uses It | Cognitive Load |
|----------|-------------|----------------|
| Collapsed, on-demand (ChatGPT, Claude) | Consumer products | Low |
| Structured, collapsible (Gemini, Hippo) | Research tools | Medium |
| Full stream, no collapse (DeepSeek R1) | Developer tools | High |

**Design implication:** Progressive disclosure is not optional — it's essential. The pattern must adapt to user expertise and task stakes.

---

### Theme 2: CoT Is Not Faithful Explainability

The Oxford paper's core finding reverberates across all sources: visible chain-of-thought is **post-hoc rationalization**, not true interpretability. This has major implications:

- Users may trust reasoning that doesn't reflect actual computation
- The UX pattern must manage this risk (disclaimers, evidence linking, confidence signals)
- Verification signals (as in MEP framework) may be more valuable than reasoning display

---

### Theme 3: Reasoning Visibility Must Span the Full Agent Lifecycle

The Smashing Magazine framework shows that "explainable" spans:

- **Pre-action** (Intent Preview) — show the plan before doing
- **In-action** (Explainable Rationale + Confidence Signal) — show reasoning while doing
- **Post-action** (Action Audit & Undo) — show what happened and allow reversal

No single view suffices; users need all three synchronized perspectives.

---

### Theme 4: Cost and Latency Are Part of Rationale

Agentic reasoning is expensive. Sources consistently tie transparency to cost:

- DeepSeek R1 API exposes reasoning tokens as separate billable content
- Langfuse tracks cost per trace, per tool call
- Research confirms $5–8/task for unconstrained agent execution
- **Design implication:** Token counts, cost estimates, and latency should be visible alongside reasoning

---

### Theme 5: Observable Agent Internals (Engineering ↔ UX)

The engineering tools (Langfuse, llm-visuals, LangSmith) and the UX pattern are two halves of the same solution:

- **Engineering layer:** Tracing, instrumentation, cost tracking (Langfuse, llm-visuals)
- **UX layer:** Human-readable plan, execution log, compact summary (Explainable Rationale pattern)
- **Both halves are necessary** for genuine transparency

---

## Summary Comparison Table

| Source | Type | Core Focus | Key Insight |
|--------|------|------------|-------------|
| Watching AI Think (arXiv) | Academic | User perception of visible thinking | Framing matters: emotional vs. expertise thinking |
| Interactive Reasoning (arXiv) | Academic | Interactive CoT visualization | Tree structure for model self-correction |
| From Features to Actions (arXiv) | Academic | Static vs. agentic explainability | Trajectory-level explanation needed; state tracking is key |
| Chain-of-Thought Is Not Explainability | Academic | CoT faithfulness analysis | CoT = post-hoc rationalization, not interpretability |
| Cost of Dynamic Reasoning (arXiv) | Academic | Agent cost/latency scaling | $5–8/task; dozens of inference calls per task |
| Designing For Agentic AI (Smashing) | Article | 6-pattern UX framework | Named pattern with anatomy, metrics, risk of omission |
| Stream of Thought (ShapeofAI) | Article | UX design pattern catalog | 3 expressions + 5 design principles |
| How AI Models Show Reasoning (DigestibleUX) | Article | 5-model UX comparison | Transparency vs. cognitive load trade-off table |
| Explainable AI in Chat (NN/g) | Article | Human factors analysis | Citations often hallucinated; disclaimers ineffective if hidden |
| Build Reasoning UIs with DeepSeek R1 (SitePoint) | Tutorial | R1 reasoning UI implementation | Tree data model; streaming reasoning + final answer |
| AI Agent Observability (Langfuse) | Blog | Agent observability engineering | Tracing as the engineering half of the UX pattern |
| how-does-AI-think (GitHub) | Repo | Glass-box agent teaching tool | 11 enterprise-readiness patterns; "observable by default" |
| claude-code-agents-ui (GitHub) | Repo | Claude agent visual dashboard | Real-time streaming of thinking + tool calls + costs |
| llm-visuals (GitHub) | Repo | LLM traffic observatory | Zero-change proxy; exposes token counts and context chains |
| langfuse/skills (GitHub) | Repo | Agent skills for Langfuse | Agents can read their own traces for self-debugging |
| OpenAI Reasoning API (Docs) | Doc | API design for reasoning models | Hidden by default; summary mode available on some models |
| DeepSeek API (Docs) | Doc | DeepSeek R1 API | `reasoning_content` field exposed as first-class output |
| Langfuse Documentation | Doc | Observability platform | Tracing + evaluation + datasets = rationale engineering |
| Stanford XAI Seminar (Video) | Talk | HCI perspective on XAI | Explanation must aid decision-making, not just feel informative |
| CoT Unreliability (Video) | Talk | CoT faithfulness critique | Empirical evidence of unfaithfulness in accessible format |

---

## Key Citations for the Wiki

> "Explainable Rationale: proactively answers the user's 'Why did it do that?' with a concise justification."
> — Victor Yocco, *Designing For Agentic AI: Practical UX Patterns For Control, Consent, And Accountability*, Smashing Magazine, February 2026
> URL: https://www.smashingmagazine.com/2026/02/designing-agentic-ai-practical-ux-patterns/

> "The AI's Stream of Thought reveals the visible trace of how it navigated from input to answer."
> — Emily Campbell, *Stream of Thought Pattern*, ShapeofAI.com
> URL: https://www.shapeofai/patterns/stream-of-thought

> "Chains-of-thought can appear persuasive even when they do not faithfully reflect a model's actual decision process."
> — Fazl Barez et al., *Chain-of-Thought Is Not Explainability*, Oxford / AI2 / Mila / Google DeepMind, 2025
> URL: https://aigi.ox.ac.uk/wp-content/uploads/2025/07/Cot_Is_Not_Explainability.pdf

> "Tailor visibility to the context. Users running complicated, computer-heavy tasks will look for full traces and logic, while users in simple conversations may require little-to-no visibility."
> — Emily Campbell, *Stream of Thought Pattern*, ShapeofAI.com
> URL: https://www.shapeofai/patterns/stream-of-thought

> "State tracking inconsistency is 2.7× more prevalent in failed vs. successful agent runs."
> — *From Features to Actions: Explainability in Traditional and Agentic AI Systems*, arXiv:2602.06841, 2025
> URL: https://arxiv.org/html/2602.06841v1

> "An unconstrained agent can cost $5 to $8 dollars per task to solve a software engineering issue."
> — *The Cost of Dynamic Reasoning: Demystifying AI Agents and Test-Time Scaling*, arXiv:2506.04301, 2025
> URL: https://arxiv.org/html/2506.04301v2

---

*Research compiled: May 2026. Sources span academic papers, practitioner articles, open source tooling, and technical documentation covering the Explainable Rationale pattern from both UX and engineering perspectives.*
