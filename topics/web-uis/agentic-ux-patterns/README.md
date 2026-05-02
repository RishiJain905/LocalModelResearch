# Agentic UX Patterns

> *Design patterns for building AI agents that are trustworthy, controllable, and accountable — spanning autonomy control, explainability, and human oversight.*

## Table of Contents

1. [Overview](#overview)
2. [Topics](#topics)
3. [Quick Reference](#quick-reference)

---

## Overview

Agentic UX patterns address the unique challenges of AI systems that plan, act, and delegate autonomously. Unlike static AI outputs, agentic AI involves:

- **Multi-step reasoning** with tool use and intermediate decisions
- **Dynamic autonomy** — the system can take actions without explicit user approval
- **Trust calibration** — users need to understand when and how much to trust the agent
- **Accountability gaps** — who is responsible when an agent makes a mistake?

> *"Autonomy is an output of a technical system, but trustworthiness is an output of a design process."*
> — [Smashing Magazine](https://www.smashingmagazine.com/2026/02/designing-agentic-ai-practical-ux-patterns/)

---

## Topics

### [[autonomy-dials]] — User-Controlled Autonomy Levels

> *A design pattern allowing users to adjust the degree of autonomous decision-making granted to an AI agent — on a continuous spectrum from human-directed to fully autonomous.*

- **Coined by:** Andrej Karpathy (June 2025)
- **Key Concept:** Graduated autonomy phases (Shadow Ops → Approval Gateway → Bounded Autonomy)
- **Patterns:** Two-Phase Commit, Risk-Based Gating, HumanLayer decorators
- **When:** High-stakes tasks, new users, escalating trust over time

### [[explainable-rationale]] — Visible Reasoning

> *An agentic UX pattern where an AI system surfaces its reasoning process — goals, constraints, intermediate decisions, tool calls, and evidence — in human-readable form.*

- **Aliases:** Stream of Thought, Visible Thinking, Chain-of-Thought Display
- **Key Warning:** CoT is NOT faithful explainability — visible reasoning ≠ actual reasoning
- **Engineering:** Langfuse, OpenLLMetry, llm-visuals for observability infrastructure
- **When:** Complex tasks, developer tools, trust-building in professional contexts

### [[hitl]] — Human-in-the-Loop

> *Design patterns where humans maintain oversight and control over autonomous AI agents — approval workflows, interrupt/pause patterns, checkpoint-based review, and escalation procedures.*

- **Modes:** HITL (in execution), HOTL (on execution), HOOTL (out of loop)
- **Patterns:** Checkpoint/Interrupt, Approval Gate, Return of Control, Intent Preview
- **Implementation:** LangGraph `interrupt_before`, AWS Bedrock user confirmation, MCP elicitation
- **When:** High-stakes, irreversible, regulated actions

---

## Quick Reference

### Agentic UX Lifecycle (Smashing Magazine)

| Phase | Patterns |
|-------|----------|
| **Pre-Action** | [[autonomy-dials]], Intent Preview |
| **In-Action** | [[explainable-rationale]], Confidence Signal |
| **Post-Action** | Action Audit & Undo, Escalation Pathway |

### Pattern Selection Guide

| Need | Pattern |
|------|---------|
| User controls how much agent can do independently | [[autonomy-dials]] |
| User wants to understand agent reasoning | [[explainable-rationale]] |
| High-stakes actions require human approval | [[hitl]] |
| User wants to preview plan before execution | Intent Preview ([[hitl]]) |
| Gradual trust building over time | [[autonomy-dials]] + [[explainable-rationale]] |

### Key Academic Insights

- **CoT faithfulness:** Visible chain-of-thought is post-hoc rationalization, not true interpretability (Oxford/AI2/Mila/DeepMind, 2025)
- **State tracking:** State consistency violations lead to 2.7× higher failure rate in agent runs (arXiv:2602.06841)
- **Cost transparency:** Unconstrained agents cost $5–8/task; token counts should be visible (arXiv:2506.04301)

### HITL Modes

| Mode | Human Role | When Required |
|------|-----------|---------------|
| **HITL** | Approval before action | Irreversible, regulated, financial/medical |
| **HOTL** | Monitor and override post-hoc | Correctable, moderate stakes |
| **HOOTL** | None | Low-stakes, retryable |

---

## See Also

- [[web-uis]] — Parent topic
- [Smashing Magazine: Designing For Agentic AI](https://www.smashingmagazine.com/2026/02/designing-agentic-ai-practical-ux-patterns/)
- [LangGraph Checkpointing](https://langchain.com/langgraph/) — HITL implementation
- [AWS Bedrock HITL](https://aws.amazon.com/blogs/machine-learning/implement-human-in-the-loop-confirmation-with-amazon-bedrock-agents/) — Enterprise patterns

---

*Last updated: 2026-05-02*
