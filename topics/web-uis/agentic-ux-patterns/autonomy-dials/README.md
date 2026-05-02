# Autonomy Dials

> *A design pattern allowing users to adjust the degree of autonomous decision-making granted to an AI agent — on a continuous spectrum from human-directed to fully autonomous operation.*

**Coined by Andrej Karpathy (June 2025) at Y Combinator's AI Startup School.**

## Table of Contents

1. [Overview](#overview)
2. [Academic Foundations](#academic-foundations)
3. [Implementation Patterns](#implementation-patterns)
4. [Checkpoint Placement](#checkpoint-placement)
5. [Graceful Degradation](#graceful-degradation)
6. [When to Use](#when-to-use)
7. [References](#references)

---

## Overview

An autonomy dial is a UI pattern that allows users to set the level of AI autonomy within a tool or product on a **continuous spectrum**:

```
[Full User Control] ←──────────────→ [Full AI Autonomy]
     Human leads      Fluid shifts      Model leads
```

**Key Properties:**
- **Continuous spectrum** — not binary on/off
- **User or context driven** — level can change during a workflow
- **Trust building** — start low-risk, earn confidence, ascend
- **Graceful fallback** — always allow retreat down the slider when model limits appear

> *"You don't jump blindly from full 'Human-in-the-Loop' safety to completely autonomous API execution. You engineer a dial—and you turn it up one notch at a time."*
> — [Rajat Pandit](https://rajatpandit.com/ai-engineering/building-an-autonomy-dial)

### Karpathy's Iron Man Analogy

> *"Think Iron Man suits, not Iron Man robots."*
> — [Andrej Karpathy, YC AI Startup School 2025](https://www.youtube.com/watch?v=LCEmiRjPEtQ)

The agentic slider progression:
```
Augmented → Orchestrated → Automated
Human boss  Passenger     Observer
```

---

## Academic Foundations

### Levels of Automation (Parasuraman, Sheridan & Wickens, 2000)

> *"Function allocation between human and automation can be represented in terms of stages and levels. The degree of automation should match the demands of the task and the capabilities of the human."*

**Source:** [Sage Journals](https://journals.sagepub.com/doi/10.1177/1555343417727438)

| Level | Description |
|-------|-------------|
| 1 | Computer offers no assistance |
| 2 | Computer offers alternatives |
| ... | ... |
| 9 | Full automation |
| 10 | Full automation, ignores human |

**Four information processing functions:**
1. Information acquisition
2. Information analysis
3. Decision and action selection
4. Action implementation

Each function can be automated at different levels. Higher autonomy improves routine performance but degrades failure handling and situational awareness.

---

### Autonomy Depth Framework (AI Agents in Financial Markets)

> *"Agent design parameters such as autonomy depth, model heterogeneity, execution coupling, infrastructure concentration, and supervisory observability affect market-level outcomes."*

**Source:** [arXiv:2603.13942](https://arxiv.org/html/2603.13942v1)

**Four-layer architecture:**
```
Data Perception → Reasoning Engines → Strategy Generation → Execution with Control
```

Higher autonomy depth requires greater supervisory observability.

---

## Implementation Patterns

### Three-Phase Deployment (Rajat Pandit)

| Phase | Description | Agent Access |
|-------|-------------|--------------|
| **Observe (Shadow Ops)** | AI processes same data, logs intended actions, no write access | Read-only |
| **Escalate (Approval Gateway)** | Agent does heavy lifting but pauses before destructive actions for human approval | Two-Phase Commit |
| **Execute (Bounded Autonomy)** | Full autonomy within rigorously constrained blast radius | Bounded rules |

### Two-Phase Commit with LangGraph

```
1. DRAFT: Agent structures exact JSON payload
2. PAUSE: Orchestrator detects destructive tool → halts → state serialized
3. HANDOFF: Async alert fires to human via Slack/portal
4. RESUME: Human approves → webhook fires → agent resumes
```

```python
app = workflow.compile(
    checkpointer=memory,
    interrupt_before=["send_message"]  # Pause before destructive actions
)

# Human reviews, then:
app.update_state(config, {"approved": True})
```

### Risk-Based Gating Formula

```
risk_score = impact×0.30 + irreversibility×0.20 + blast_radius×0.20 + evidence_uncertainty×0.15 + policy_sensitivity×0.15
```

| Score | Action |
|-------|--------|
| 0.00–0.29 | Fully autonomous |
| 0.30–0.59 | Standard monitoring |
| 0.60–0.79 | Enhanced logging |
| 0.80–1.00 | Requires approval |

---

## Checkpoint Placement

### Key Principle

> *"Humans should approve transitions where the cost of being wrong exceeds the cost of waiting. Everything else should be automated, verified, and logged."*

### Workflow State Machine

```
planned → prepared → verified → awaiting_approval → executed → audited
```

### Placement Rules

| Rule | Description |
|------|-------------|
| **Gates at transitions** | State changes, not step outputs |
| **Graduated autonomy** | Auto-clear safe actions |
| **Bounded proposals** | Compact, explainable artifacts |
| **Centralized control** | Orchestration layer owns gates |
| **Incremental rollout** | Start narrow, expand with data |

### Avoid Over-Blocking

> *"If a reviewer sees the same harmless request fifty times, the threshold is wrong."*

---

## Graceful Degradation

### Layered Resilience Framework

| Layer | Strategy | Cost |
|-------|----------|------|
| 1 | Error classification | Free |
| 2 | Retries with backoff | Cheap |
| 3 | Circuit breakers | Cheap |
| 4 | Bulkheads | Moderate |
| 5 | Fallback models/providers | Moderate |
| 6 | Queue-based buffering | Expensive |
| 7 | Human escalation | Expensive |

### Degradation Behavior

When autonomy constraints are violated:
1. Recognize constraint failure
2. Shift to lower autonomy level
3. Pause and escalate to human
4. Log for post-hoc review

---

## When to Use

| Scenario | Recommended Autonomy Level |
|----------|---------------------------|
| High-stakes (financial, medical) | Low — Approval gates required |
| Well-defined, repeatable tasks | Medium-High — Bounded autonomy |
| Exploratory/research tasks | High — Full autonomy with observability |
| New user onboarding | Low → Graduated increase |
| High-cost actions (API calls, emails) | Medium — Risk-based approval |
| Reversible actions | Higher — With logging |

### UX Design Principles (Victor Dibia)

> *"Multi-Agent Systems where LLMs drive control flow are suited to problem spaces where the exact solution is unknown—but they introduce challenges around reliability, user trust, and system comprehensibility."*

**The 4 Core UX Principles:**

| Principle | Goal |
|-----------|------|
| **Capability Discovery** | Help users understand what agents can do |
| **Observability & Provenance** | Users can observe/trace agent actions |
| **Interruptibility** | Allow users to pause, resume, or cancel |
| **Cost-Aware Delegation** | Communicate cost of agent actions; let users decide |

---

## References

### Academic
- [Parasuraman, Sheridan & Wickens — A Model for Types and Levels of Human Interaction with Automation](https://journals.sagepub.com/doi/10.1177/1555343417727438)
- [AI Agents in Financial Markets — Autonomy Depth Framework](https://arxiv.org/html/2603.13942v1)

### Blog Posts & Articles
- [Building an Autonomy Dial — Rajat Pandit](https://rajatpandit.com/ai-engineering/building-an-autonomy-dial)
- [Autonomy Sliders — Andrew Miller](https://andrewships.substack.com/p/autonomy-sliders)
- [4 UX Design Principles — Victor Dibia](https://newsletter.victordibia.com/p/4-ux-design-principles-for-multi)
- [The Autonomy Slider — Engineer at Heart](https://engineeratheart.medium.com/the-autonomy-slider-a-decision-framework-for-when-to-use-workflows-single-agents-or-multi-agent-7da35e415923)
- [Designing for Control in AI UX — UX Collective](https://uxdesign.cc/from-journey-maps-to-control-maps-17aac58b9dd9)

### Repositories
- [awesome-agentic-patterns](https://github.com/nibzard/awesome-agentic-patterns) — HITL Approval Framework
- [langgraph-interrupt-workflow-template](https://github.com/KirtiJha/langgraph-interrupt-workflow-template) — Production-ready HITL
- [langgraph-agentic-workflows](https://github.com/ozereray/langgraph-agentic-workflows) — Strategic interrupt patterns

### Talks
- [Andrej Karpathy — YC AI Startup School](https://www.youtube.com/watch?v=LCEmiRjPEtQ) — Introduced autonomy slider concept

---

*Part of [[agentic-ux-patterns]] — Web UIs*
