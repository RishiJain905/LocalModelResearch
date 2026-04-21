# Agentic Utility Analysis

> Benchmarks, metrics, and framework evaluations for measuring how well LLMs and AI agents perform in real-world software engineering and tool-use tasks.

## Contents

| Subtopic | Description |
|---------|-------------|
| [Multi-File Reasoning Scores](./Multi-File-Reasoning-Scores.md) | Benchmarks and metrics for evaluating code agents on multi-file, repository-level issue resolution |
| [Tool-Use Reliability (TUR)](./Tool-Use-Reliability.md) | Frameworks and benchmarks for measuring correct tool selection, invocation, and parameter grounding in LLM agents |
| [Agent SDK Compatibility](./Agent-SDK-Compatibility.md) | Interoperability protocols, cross-SDK portability, and standards for agent framework communication |

---

## Overview

Agentic Utility Analysis covers the evaluation landscape for LLM-powered agents performing real-world tasks: writing code across multiple files, reliably calling tools and APIs, and interoperating across frameworks. Unlike traditional NLP benchmarks, these metrics focus on **task completion** in interactive, execution-grounded environments.

Key themes across all three subtopics:
- **Execution-grounded evaluation** — patches must pass tests, not just look correct
- **Cross-file / cross-tool reasoning** — single-file benchmarks overstate real-world capability
- **Fragmentation** — every framework uses different tool schemas, agent protocols, and evaluation harnesses
- **Rapid evolution** — SWE-bench went from 2,294 tasks (2024) → Pro (2025) → Multilingual (2025) in under 18 months

---

## Key Findings

> "There is an evaluation crisis. I don't really know what metrics to look at right now… SWE-Bench Verified (real, practical, verified problems) I really like and is great but itself too narrow." — Andrej Karpathy, March 2024

> "Our findings show top-tier models like Opus 4.1 and GPT-5 achieving a 23% success rate on SWE-Bench Pro compared to over 70% on benchmarks like SWE-Bench Verified, highlighting a critical gap between current agent capabilities and the demands of real-world development." — SWE-Bench Pro paper, Scale AI

> "Tool hallucinations are more severe than traditional text-based hallucinations because tools serve as the interface between LLMs and the physical world." — Xu et al., *Reducing Tool Hallucination via Reliability Alignment*

> "The tool-augmented LLM ecosystem is fragmented across four dimensions: protocol fragmentation, manual implementation overhead, complex execution workflow, and OpenAI dominance." — *Unified Tool Integration for LLMs* (arXiv:2508.02979)

1. **Single-file benchmarks are saturated; multi-file is the frontier.** Top systems resolve ~90% of single-file SWE-bench Verified issues but only ~54% of multi-file issues. SWE-bench Pro (averaging ~4 files per fix) drops top models from 80%+ to ~23%.
2. **Tool-use reliability has four failure modes.** Hallucination taxonomy covers Tool Type, Tool Timing, Tool Format, and Tool Content errors — each requiring different alignment strategies.
3. **SDK ecosystem is converging on layers, not monoliths.** MCP for agent-tool, A2A for agent-agent, and gateway layers for governance. The Linux Foundation hosts A2A, AGNTCY, and Agentgateway.
4. **Contamination inflates simple benchmarks.** OpenAI audit found frontier models could reproduce verbatim gold patches on some Verified tasks — contamination-resistant benchmarks (Pro, Multilingual) are essential.

---

## Related Topics

- [Reasoning](../Reasoning/) — Chain-of-thought overhead, hidden reasoning, frontier reasoning benchmarks
- [Benchmarks](../../benchmarks/) — General evaluation frameworks and metrics
- [Quantization](../../quantization/) — Model size, quality/size tradeoffs for deployed agents
- [Safety](../../safety/) — Red-teaming, adversarial evaluation, hallucination detection
