# Autonomy Dials: Agentic UX Pattern for LLM Interactions

> A design pattern allowing users to adjust the degree of autonomous decision-making granted to an AI agent, typically along a spectrum ranging from human-directed to fully autonomous operation.

**Coined by Andrej Karpathy (June 2025)** at Y Combinator's AI Startup School.

---

## Table of Contents

1. [Overview](#overview)
2. [Academic Foundations](#academic-foundations)
3. [Blog Posts & Articles](#blog-posts--articles)
4. [Open Source Repositories](#open-source-repositories)
5. [Technical Implementation Patterns](#technical-implementation-patterns)
6. [Human-in-the-Loop Checkpoint Patterns](#human-in-the-loop-checkpoint-patterns)
7. [Graceful Degradation & Fallback](#graceful-degradation--fallback)
8. [Talks & Videos](#talks--videos)
9. [Related Concepts](#related-concepts)

---

## Overview

**What is an Autonomy Dial?**

An autonomy dial (also called "autonomy slider" or "agentic slider") is a UI pattern that allows users to set the level of AI autonomy within a tool or product. It exists on a **continuous spectrum**, not a binary toggle:

```
[Full User Control] ←──────────────→ [Full AI Autonomy]
     Human leads      Fluid shifts      Model leads
```

**Key Properties:**
- **Continuous spectrum** — not binary on/off
- **User or context driven** — level can change during a workflow
- **Trust building** — start low-risk, earn confidence, ascend
- **Graceful fallback** — always allow retreat down the slider when model limits appear

**Why It Matters:**
- Addresses the **trust-risk tension** inherent in AI products
- Backed by **40+ years of human factors research** (Sheridan & Verplanck, 1978)
- Most products don't yet feature this pattern — opportunity exists

---

## Academic Foundations

### Levels of Automation Research

**Source:** [Parasuraman, Sheridan & Wickens (2000) — "A Model for Types and Levels of Human Interaction with Automation"](https://journals.sagepub.com/doi/10.1177/1555343417727438)

> *"Function allocation between human and automation can be represented in terms of stages and levels. The degree of automation should match the demands of the task and the capabilities of the human."*

**Key Concepts:**
- Automation exists on a spectrum across **four information processing functions**:
  1. Information acquisition
  2. Information analysis
  3. Decision and action selection
  4. Action implementation
- Each function can be automated at different **10 levels** (from fully manual to fully automated)

**Why Valuable:**
- Provides theoretical grounding for autonomy dial design
- Shows that automation level should be **task-dependent**, not global
- Higher autonomy improves routine performance but degrades failure handling and situational awareness

---

### Agentic Finance: Autonomy Depth Framework

**Source:** [arXiv: AI Agents in Financial Markets (2603.13942)](https://arxiv.org/html/2603.13942v1)

> *"Agent design parameters such as autonomy depth, model heterogeneity, execution coupling, infrastructure concentration, and supervisory observability affect market-level outcomes."*

**Key Concepts:**
- Introduces **"autonomy depth"** as a key agent design parameter
- Four-layer architecture: data perception → reasoning engines → strategy generation → execution with control
- Higher autonomy depth requires greater supervisory observability

**Why Valuable:**
- Formalizes autonomy as a design parameter
- Links autonomy settings to systemic risk in multi-agent environments
- Provides vocabulary for discussing graduated autonomy in production systems

---

### Fundamentals of Building Autonomous LLM Agents

**Source:** [arXiv:2510.09244](https://arxiv.org/abs/2510.09244)

**Key Concepts:**
- Comprehensive survey of LLM agent architecture
- Covers planning, reasoning, tool use, and memory systems
- Does not focus on UX/UI but provides the underlying technical foundations

**Why Valuable:**
- Technical foundation for understanding what "autonomy" means at the agent level
- Helps contextualize where human oversight can be inserted

---

## Blog Posts & Articles

### 1. Building an Autonomy Dial: Safely Shipped Agentic Architecture

**Source:** [Rajat Pandit — rajatpandit.com](https://rajatpandit.com/ai-engineering/building-an-autonomy-dial)

> *"You don't jump blindly from full 'Human-in-the-Loop' safety to completely autonomous API execution. You engineer a dial—and you turn it up one notch at a time."*

**Key Concepts — The Three Phases:**

| Phase | Description | Agent Access |
|-------|-------------|--------------|
| **Observe (Shadow Ops)** | AI processes same data, logs intended actions, no write access | Read-only |
| **Escalate (Approval Gateway)** | Agent does heavy lifting but pauses before destructive actions for human approval | Two-Phase Commit |
| **Execute (Bounded Autonomy)** | Full autonomy within rigorously constrained blast radius | Bounded rules |

**Graceful Degradation Example:**
> *"If agent encounters scenario violating bounding box rules (e.g., $400 refund request), it gracefully degrades—recognizes constraint failure, shifts back to Phase 2, pauses, and escalates edge-case to human."*

**Why Valuable:**
- Provides a **phased deployment methodology** for gradually increasing autonomy
- Shows how to implement **bounded autonomy** with constraint rules
- Includes code patterns for the Two-Phase Commit pattern with LangGraph

---

### 2. The Autonomy Slider: A Decision Framework for When to Use Workflows, Single Agents, or Multi-Agent

**Source:** [Engineer at Heart — Medium](https://engineeratheart.medium.com/the-autonomy-slider-a-decision-framework-for-when-to-use-workflows-single-agents-or-multi-agent-7da35e415923)

> *"The industry is over-indexing on multi-agent. Here's a concrete framework — with code — for choosing the right level of autonomy."*

**Key Framework:**

| Aspect | Workflows | Agents |
|--------|-----------|--------|
| Execution | Fixed, predefined steps | Dynamic, LLM-decided |
| Reliability | High (predictable) | Variable |
| Cost/Latency | Consistent | Variable |
| Flexibility | Rigid | Adaptable |

**Why Valuable:**
- Helps developers **choose** between workflow vs. agent approaches
- Includes code examples for implementing the spectrum
- Practical decision framework for production systems

---

### 3. Autonomy Sliders — An Increasingly Important Design Pattern in AI Products

**Source:** [Andrew Miller — Substack](https://andrewships.substack.com/p/autonomy-sliders)

> *"Autonomy Slider (n.) — a design pattern that allows individuals to adjust the degree of autonomous decision-making and action-taking granted to an AI system."*

**Historical Roots:**

| Year | Researchers | Contribution |
|------|-------------|--------------|
| 1978 | Sheridan & Verplanck | Proposed **10 levels of automation** |
| 1999 | Endsley & Kaber | Expanded levels by assigning task ownership |
| 2000 | Parasuraman, Sheridan, Wickens | Four functions of information processing framework |

**Product Examples:**
- **Cursor IDE:** Tab completion → AI edit → Agent mode (multiple file edits)
- **Perplexity:** Search → Research → Deep Research

**Why Valuable:**
- Formal definition and historical context
- Maps the pattern to existing products
- Shows the academic lineage of the concept

---

### 4. 4 UX Design Principles for Autonomous Multi-Agent AI Systems

**Source:** [Victor Dibia — Newsletter](https://newsletter.victordibia.com/p/4-ux-design-principles-for-multi)

> *"Multi-Agent Systems where LLMs drive control flow are suited to problem spaces where the exact solution is unknown—but they introduce challenges around reliability, user trust, and system comprehensibility."*

**The 4 Core UX Principles:**

| Principle | Goal |
|-----------|------|
| **Capability Discovery** | Help users understand what agents can do |
| **Observability & Provenance** | Users can observe/trace agent actions |
| **Interruptibility** | Allow users to pause, resume, or cancel |
| **Cost-Aware Delegation** | Communicate cost of agent actions; let users decide |

**5-Stage Development Process:**
1. Goal Definition
2. Baseline (non-agentic)
3. Tools (spend most time here)
4. Eval Testbed
5. Agent

> ⚠️ ~**70% of effort** should be on steps 3 and 4.

**Why Valuable:**
- UX-focused framework for multi-agent systems
- Practical guidance on building evaluation harnesses
- Links to **BlenderLM** example implementation

---

### 5. Designing for Control in AI UX

**Source:** [UX Collective — Rob Chappell](https://uxdesign.cc/from-journey-maps-to-control-maps-17aac58b9dd9)

> *"After watching Karpathy's talk, I kept thinking about the 'autonomy slider' — the idea that users choose how much control to hand over to an AI."*

**The Control-Aware UX Framework:**

| Principle | Question It Answers |
|-----------|---------------------|
| **Intent Scaffolding** | How do I suggest actions? |
| **Clarity and Preview** | What is the AI doing? Why? |
| **Steerability** | Can I change the path? |
| **Reversibility** | Can I undo what just happened? |
| **Transparency & Alignment** | Will this system respect my oversight? |

**Two Fundamental Interaction Modes:**
1. **Human-as-Driver** — detailed explicit commands, AI executes
2. **Model-as-Driver** — high-level goals, model plans and iterates

**Why Valuable:**
- UX design perspective on autonomy control
- Introduces **OESDs (Operator Event Sequence Diagrams)** from safety-critical fields
- Practical example from **Adobe Firefly** (slider for visual intensity)

---

### 6. AI Workflows vs Agents: The Autonomy Slider

**Source:** [Decoding AI Magazine](https://www.decodingai.com/p/ai-workflows-vs-agents-the-autonomy)

**Key Insight:**
> *"The decision between workflows and agents is not binary—it exists on a spectrum."*

**Real-World Example — Deep Research Hybrid (Perplexity):**
```python
while gaps_exist:
    sub_queries = LLM.decompose_query(user_query, all_findings)
    for query in sub_queries:
        result = Agent(query=query, tools=[web_search, analyze])
        findings.append(result.synthesize())
```

**Why Valuable:**
- Part of a comprehensive 9-article series on AI Agent Engineering
- Code examples for hybrid architectures
- Clear comparison table of workflows vs agents

---

### 7. The Autonomy Dial: Why Every AI Agent Builder Landed on the Same Design Trick

**Source:** [Tao An — Medium](https://tao-hpu.medium.com/the-autonomy-dial-why-every-ai-agent-builder-landed-on-the-same-design-trick-e795cc9ae713)

**Six Patterns for Production Agents:**
1. Mode switching (Claude Code, Cursor, Copilot Studio)
2. Guardrails and constrained autonomy
3.graduated trust via observability
4. Human-in-the-loop checkpoints
5. Bounded execution contexts
6. Fallback escalation chains

**Why Valuable:**
- Documents pattern convergence across major AI products
- Practical patterns for production systems

---

## Open Source Repositories

### 1. awesome-agentic-patterns — Human-in-the-Loop Approval Framework

**Source:** [nibzard/awesome-agentic-patterns](https://github.com/nibzard/awesome-agentic-patterns/blob/main/patterns/human-in-loop-approval-framework.md)

**Core Principle:**
> *"Systematically insert human approval gates for designated high-risk functions while maintaining agent autonomy for safe operations."*

**Key Components:**
- Risk classification system
- Multi-channel approval interface
- Approval workflow automation
- Comprehensive audit trail

**Why Valuable:**
- Pattern catalog for production agent architectures
- References actual implementation sources (Claude Code, HumanLayer)
- Covers decorator patterns and interrupt patterns

---

### 2. LangGraph Human-in-the-Loop Examples

**Source:** [KirtiJha/langgraph-interrupt-workflow-template](https://github.com/KirtiJha/langgraph-interrupt-workflow-template)

**Key Pattern:**
```python
app = workflow.compile(
    checkpointer=memory,
    interrupt_before=["send_message"]  # Pause before destructive actions
)
```

**Why Valuable:**
- Production-ready LangGraph template
- Modern web interface for approval workflows
- Demonstrates pause/resume with state persistence

---

### 3. LangGraph Agentic Workflows with Strategic Interrupts

**Source:** [ozereray/langgraph-agentic-workflows](https://github.com/ozereray/langgraph-agentic-workflows)

**Key Feature:**
> *"Strategic interrupt_before points that allow humans to approve or modify agent actions (e.g., confirming a search query or a financial transaction)."*

**Why Valuable:**
- Illustrates placement of interrupt points
- Shows how to handle different approval scenarios
- Modern Python implementation

---

## Technical Implementation Patterns

### Pattern 1: Two-Phase Commit with LangGraph

**Source:** [Rajat Pandit](https://rajatpandit.com/ai-engineering/building-an-autonomy-dial)

```
1. DRAFT: Agent structures exact JSON payload
2. PAUSE: Orchestrator detects destructive tool → halts → state serialized
3. HANDOFF: Async alert fires to human via Slack/portal
4. RESUME: Human approves → webhook fires → agent resumes
```

**Code Pattern:**
```python
app = workflow.compile(
    checkpointer=memory,
    interrupt_before=["send_message"]  # Pause before destructive actions
)

# Human reviews, then:
app.update_state(config, {"approved": True})
```

---

### Pattern 2: Risk-Based Gating

**Source:** [GitHub Gist — Designing HITL Gates](https://gist.github.com/prasad-kumkar/36c847ffc99681a657ee4a1d7f1a5d46)

**Risk Scoring Formula:**
```
risk_score = impact×0.30 + irreversibility×0.20 + blast_radius×0.20 + evidence_uncertainty×0.15 + policy_sensitivity×0.15
```

**Threshold Levels:**

| Score | Action |
|-------|--------|
| 0.00–0.29 | Fully autonomous |
| 0.30–0.59 | Standard monitoring |
| 0.60–0.79 | Enhanced logging |
| 0.80–1.00 | Requires approval |

**Key Rules:**
- Gate proposals, not raw reasoning
- Gates at transitions, not every step
- Agents must not recommend AND authorize their own actions

---

### Pattern 3: HumanLayer Decorator Pattern

**Source:** [awesome-agentic-patterns](https://github.com/nibzard/awesome-agentic-patterns)

```python
@human_layer.require_approval(for_tools=["send_email", "delete_files"])
def agent_task():
    # High-risk operations require human approval
    pass
```

---

### Pattern 4: Cost-Aware Delegation

**Source:** [Victor Dibia](https://newsletter.victordibia.com/p/4-ux-design-principles-for-multi)

> *"Implement 'risk/cost classifiers' that return risk levels (low/medium/high). Allow users to configure behavior—auto-allow low-risk, require approval for medium/high-risk operations."*

**UX Implementation:**
- Stream all updates (structured plans, step counts, duration, cost)
- Show tokens used, actions taken
- Let users configure default behaviors per risk level

---

## Human-in-the-Loop Checkpoint Patterns

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

## Graceful Degradation & Fallback

### Layered Resilience Framework

**Source:** [Zylos Research — Graceful Degradation Patterns](https://zylos.ai/zh/research/2026-02-20-graceful-degradation-ai-agent-systems)

| Layer | Strategy | Cost |
|-------|----------|------|
| 1 | Error classification | Free |
| 2 | Retries with backoff | Cheap |
| 3 | Circuit breakers | Cheap |
| 4 | Bulkheads | Moderate |
| 5 | Fallback models/providers | Moderate |
| 6 | Queue-based buffering | Expensive |
| 7 | Human escalation | Expensive |

### Key Insight

> *"Agent failures are often subtle: a slow model, a rate-limited API, or a hallucinated tool call can silently degrade output quality long before a hard crash occurs."*

### Degradation Behavior

When autonomy constraints are violated:
1. Recognize constraint failure
2. Shift to lower autonomy level
3. Pause and escalate to human
4. Log for post-hoc review

---

## Talks & Videos

### 1. Andrej Karpathy: Software Is Changing (Again)

**Source:** [YouTube — Y Combinator AI Startup School](https://www.youtube.com/watch?v=LCEmiRjPEtQ)

**Key Quotes:**
> *"Prompts are programs, written in English, executed by neural nets."*

> *"Keep the AI on a leash. Let it assist, not run wild."*

> *"Think Iron Man suits, not Iron Man robots."*

**The Agentic Slider Progression:**
```
Augmented → Orchestrated → Automated
Human boss  Passenger     Observer
```

**Why Valuable:**
- Introduced the "autonomy slider" concept to the startup community
- Iron Man analogy for human-AI collaboration
- Emphasizes partial autonomy over full autonomy

---

### 2. UX Design Principles for Multi-Agent Systems

**Source:** [Victor Dibia — 2025 AI.Engineer Conference](https://newsletter.victordibia.com/p/4-ux-design-principles-for-multi)

**Keynote Focus:**
- Capability discovery
- Observability and provenance
- Interruptibility
- Cost-aware delegation

**Why Valuable:**
- Multi-agent UX from a framework maintainer (AutoGen)
- Links theory to production experience

---

## Related Concepts

### Levels of Automation (Sheridan & Verplanck, 1978)

A foundational 10-level scale from fully manual to fully automated:

| Level | Description |
|-------|-------------|
| 1 | Computer offers no assistance |
| 2 | Computer offers alternatives |
| 3 | ... | 
| 9 | Full automation |
| 10 | Full automation, ignores human |

**Relevance:** Provides academic grounding for graduated autonomy in AI agents.

---

### Human-on-the-Loop vs Human-in-the-Loop

| Mode | Description |
|------|-------------|
| **Human-in-the-Loop** | Human intervenes during execution |
| **Human-on-the-Loop** | Human monitors and can override post-hoc |
| **Human-out-of-the-Loop** | Full autonomy, no human involvement |

**Relevance:** Autonomy dials allow shifting between these modes dynamically.

---

### Operator Event Sequence Diagrams (OESDs)

Used in aerospace, autonomous vehicles, and robotics to visualize control shifts between humans and machines.

**Relevance:** Can be adapted to map autonomy level changes in agentic UX.

---

## Summary: When to Use Different Autonomy Levels

| Scenario | Recommended Autonomy Level |
|----------|---------------------------|
| High-stakes (financial, medical) | Low — Approval gates required |
| Well-defined, repeatable tasks | Medium-High — Bounded autonomy |
| Exploratory/research tasks | High — Full autonomy with observability |
| New user onboarding | Low → Graduated increase |
| High-cost actions (API calls, emails) | Medium — Risk-based approval |
| Reversible actions | Higher — With logging |

---

## Key Takeaways

1. **Autonomy dials are a spectrum, not a toggle** — Graduated levels outperform binary on/off
2. **Trust is engineered, not given** — Start with observation/shadow mode, earn higher autonomy
3. **Place gates at control points, not everywhere** — Avoid approval fatigue
4. **Graceful degradation is essential** — When constraints are violated, shift down and escalate
5. **UX matters as much as architecture** — Observability, interruptibility, cost-awareness
6. **40+ years of research underpins this** — Leverage established frameworks (Parasuraman)
7. **Keep the human in the loop strategically** — Not every step, just meaningful transitions

---

## References

- [Building an Autonomy Dial — Rajat Pandit](https://rajatpandit.com/ai-engineering/building-an-autonomy-dial)
- [Autonomy Sliders — Andrew Miller](https://andrewships.substack.com/p/autonomy-sliders)
- [4 UX Design Principles — Victor Dibia](https://newsletter.victordibia.com/p/4-ux-design-principles-for-multi)
- [The Autonomy Slider — Engineer at Heart](https://engineeratheart.medium.com/the-autonomy-slider-a-decision-framework-for-when-to-use-workflows-single-agents-or-multi-agent-7da35e415923)
- [Designing for Control in AI UX — UX Collective](https://uxdesign.cc/from-journey-maps-to-control-maps-17aac58b9dd9)
- [Human-in-the-Loop Approval Framework — awesome-agentic-patterns](https://github.com/nibzard/awesome-agentic-patterns)
- [Designing HITL Gates — GitHub Gist](https://gist.github.com/prasad-kumkar/36c847ffc99681a657ee4a1d7f1a5d46)
- [AI Agents in Financial Markets — arXiv](https://arxiv.org/html/2603.13942v1)
- [Levels of Automation (Parasuraman et al.)](https://journals.sagepub.com/doi/10.1177/1555343417727438)
- [Graceful Degradation Patterns — Zylos Research](https://zylos.ai/zh/research/2026-02-20-graceful-degradation-ai-agent-systems)
- [Andrej Karpathy — YC AI Startup School](https://www.youtube.com/watch?v=LCEmiRjPEtQ)
