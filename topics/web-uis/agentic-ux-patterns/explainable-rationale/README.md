# Explainable Rationale

> *An agentic UX pattern where an AI system surfaces its reasoning process — goals, constraints, intermediate decisions, tool calls, and evidence — in human-readable form before, during, or after task execution.*

**Aliases:** Visible Thinking, Stream of Thought, Chain-of-Thought Display, Reasoning Transparency

## Table of Contents

1. [Overview](#overview)
2. [Academic Research](#academic-research)
3. [Design Patterns](#design-patterns)
4. [Engineering Infrastructure](#engineering-infrastructure)
5. [Transparency vs. Cognitive Load](#transparency-vs-cognitive-load)
6. [References](#references)

---

## Overview

The pattern directly answers the user's *"Why did it do that?"* and *"How did it reach that conclusion?"* questions.

> *"The AI's Stream of Thought reveals the visible trace of how it navigated from input to answer. This includes: the plan it formed, tools called, code executed, checks and decisions made along the way."*
> — [ShapeofAI.com](https://www.shapeofai/patterns/stream-of-thought)

**Three canonical expressions:**
- **Human-readable plans** — preview what the AI will do before acting
- **Execution logs** — record tool calls, code, and results as they happen
- **Compact summaries** — capture logical reasoning, insights, and decisions post-hoc

**Relation to CoT:** Chain-of-Thought (CoT) prompting elicits step-by-step reasoning *from the model*; Explainable Rationale surfaces that reasoning *to the user*. CoT is a model behavior; Explainable Rationale is a UX design pattern.

---

## Academic Research

### User Perceptions of Visible Thinking

> *"Both the presence and framing of visible thinking significantly shape user perceptions — emotionally-supportive thinking increases warmth and empathy, while expertise-supportive thinking increases trust and competence."*

**Source:** [arXiv:2601.16720](https://arxiv.org/html/2601.16720v1)

**Study (N=234):**
- Emotionally-supportive thinking utterances → highest Emotional Responsiveness scores
- Expertise-supportive thinking → highest Understanding & Trust scores
- Thinking content type should match the conversational context

### Interactive Reasoning (Hippo)

> *"Hippo allows users to quickly identify and interrupt erroneous generations, efficiently steer the model towards customized responses, and better understand both model reasoning and model outputs."*

**Source:** [arXiv:2506.23678](https://arxiv.org/html/2506.23678v1)

- Tree structure for CoT visualization (branching when model self-corrects)
- User study (N=16): significant improvements in Control, Sense-making, Layout, Awareness, and Confidence
- Key finding: users rarely edited mid-generation; most edits happened after completion

### Chain-of-Thought Is NOT Explainability

> *"Chains-of-thought can appear persuasive even when they do not faithfully reflect a model's actual decision process."*

> *"LLMs deploy multiple parallel pathways of answer generation for step-by-step reasoning. The sequential CoT is at best a selective and often lossy projection of distributed computation."*

**Source:** [Oxford/AI2/Mila/DeepMind](https://aigi.ox.ac.uk/wp-content/uploads/2025/07/Cot_Is_Not_Explainability.pdf)

**Critical Warning:** Visible CoT ≠ genuine explainability. The pattern must manage this risk via disclaimers, evidence linking, and confidence signals.

### Cost of Dynamic Reasoning

> *"An unconstrained agent can cost $5 to $8 dollars per task to solve a software engineering issue."*

**Source:** [arXiv:2506.04301](https://arxiv.org/html/2506.04301v2)

Agentic reasoning is expensive — token counts, cost estimates, and latency should be visible alongside reasoning.

---

## Design Patterns

### Agentic UX Lifecycle (Smashing Magazine)

| Phase | Patterns |
|-------|----------|
| **Pre-Action** | Intent Preview, Autonomy Dial |
| **In-Action** | **Explainable Rationale**, Confidence Signal |
| **Post-Action** | Action Audit & Undo, Escalation Pathway |

### Design Principles (ShapeofAI.com)

> *"Tailor visibility to the context. Users running complicated, computer-heavy tasks will look for full traces and logic, while users in simple conversations may require little-to-no visibility."*

- Show the plan **before** acting — editable sequence of steps
- Separate Plan, Execution, and Evidence into three synchronized views
- Tailor visibility to context (mode vs. intensity)
- Make steps into **states** with progress cues

### Model Comparison (Transparency vs. Cognitive Load)

| Model | Transparency | Structure | Cognitive Load |
|-------|-------------|-----------|----------------|
| DeepSeek R1 | Most transparent | Low | High |
| Gemini 2.0 | High | High | Medium |
| Grok 3 | Medium | Low | High |
| Claude 3.7 | Minimal (on-demand) | High | Low |
| ChatGPT o3-mini | Minimal (on-demand) | Medium | Low |

---

## Engineering Infrastructure

### Langfuse — Agent Observability

> *"Agents use multiple steps to solve complex tasks, and inaccurate intermediary results can cause failures of the entire system."*

**Source:** [Langfuse Blog](https://langfuse.com/blog/2024-07-ai-agent-observability-with-langfuse)

| Capability | How It Supports Rationale Visibility |
|------------|-------------------------------------|
| **Tracing** | Captures every LLM call, tool execution, retrieval step |
| **Prompt Management** | Version and compare CoT prompt formulations |
| **Datasets** | Test edge cases; benchmark reasoning quality |
| **Evaluation** | LLM-as-judge metrics assess reasoning faithfulness |
| **Scores** | Attach quality scores to reasoning traces |

### llm-visuals — Reverse Proxy Observatory

Zero-code-change observability. Architecture: `LLM Client → LLM Visuals Proxy (localhost:4000) → LLM API`

**What you can see:**
- Full Request Inspector — every token, hidden system prompt, every tool call
- Real-Time Metrics — KPIs, latency, duration, cost tracking
- Context Chain Visualization — full conversation chain for any request

### DeepSeek R1 API — Streaming Reasoning

> *"DeepSeek-R1 reasoning tokens arrive first in the stream. Only after reasoning completes does `content` begin populating."*

**Source:** [DeepSeek API Docs](https://api-docs.deepseek.com/guides/reasoning_model)

Response contains: `reasoning_content` (intermediate) and `content` (final answer). This is the technical foundation for real-time reasoning display.

---

## Transparency vs. Cognitive Load

### The Fundamental Tension

More reasoning visibility builds trust, but too much overwhelms users.

> *"AI transparency isn't about revealing everything — it's about showing the right reasoning at the right time to build trust without overwhelming users."*
> — [DigestibleUX](https://www.digestibleux.com/p/how-ai-models-show-their-reasoning)

### Progressive Disclosure

The pattern must adapt to user expertise and task stakes:

| Approach | Who Uses It | Cognitive Load |
|----------|-------------|----------------|
| Collapsed, on-demand (ChatGPT, Claude) | Consumer products | Low |
| Structured, collapsible (Gemini, Hippo) | Research tools | Medium |
| Full stream, no collapse (DeepSeek R1) | Developer tools | High |

### Three Cross-Cutting Themes

1. **CoT is not faithful explainability** — visible reasoning = post-hoc rationalization, not true interpretability
2. **Reasoning visibility must span the full lifecycle** — Pre-action (Intent Preview) → In-action (Explainable Rationale) → Post-action (Undo/Audit)
3. **Cost/latency must be visible** — agent reasoning costs $5–8/task; token counts and pricing should surface alongside reasoning

---

## References

### Academic Papers
1. [Watching AI Think: User Perceptions of Visible Thinking](https://arxiv.org/html/2601.16720v1) — arXiv:2601.16720
2. [Interactive Reasoning: Visualizing CoT (Hippo)](https://arxiv.org/html/2506.23678v1) — arXiv:2506.23678
3. [From Features to Actions: Explainability in AI Systems](https://arxiv.org/html/2602.06841v1) — arXiv:2602.06841
4. [Chain-of-Thought Is Not Explainability](https://aigi.ox.ac.uk/wp-content/uploads/2025/07/Cot_Is_Not_Explainability.pdf) — Oxford/AI2/Mila/DeepMind
5. [The Cost of Dynamic Reasoning](https://arxiv.org/html/2506.04301v2) — arXiv:2506.04301

### Blog Posts & Articles
6. [Designing For Agentic AI — Smashing Magazine](https://www.smashingmagazine.com/2026/02/designing-agentic-ai-practical-ux-patterns/)
7. [Stream of Thought Pattern — ShapeofAI.com](https://www.shapeofai/patterns/stream-of-thought)
8. [How AI Models Show Their Reasoning — DigestibleUX](https://www.digestibleux.com/p/how-ai-models-show-their-reasoning)
9. [Explainable AI in Chat Interfaces — Nielsen Norman Group](https://www.nngroup.com/articles/explainable-ai/)
10. [Build Reasoning UIs with DeepSeek R1 — SitePoint](https://www.sitepoint.com/build-reasoning-uis-with-deepseek-r1-visualize-chainofthought-2026/)
11. [AI Agent Observability with Langfuse](https://langfuse.com/blog/2024-07-ai-agent-observability-with-langfuse)

### Open Source Repos
12. [how-does-AI-think](https://github.com/lutzfinger/how-does-AI-think) — Glass-box agent teaching tool
13. [claude-code-agents-ui](https://github.com/Ngxba/claude-code-agents-ui) — Real-time Claude agent dashboard
14. [llm-visuals](https://github.com/hoodini/llm-visuals) — Reverse proxy LLM observatory

### Technical Documentation
15. [OpenAI Reasoning Models API](https://developers.openai.com/api/docs/guides/reasoning)
16. [DeepSeek API Reasoning Model Guide](https://api-docs.deepseek.com/guides/reasoning_model)
17. [Langfuse Documentation](https://langfuse.com/)

---

*Part of [[agentic-ux-patterns]] — Web UIs*
