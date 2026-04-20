# Context Rot

> **Papers:** [Context Rot (Chroma)](https://research.trychroma.com/context-rot) · [Intelligence Degradation](https://arxiv.org/abs/2601.15300) · [Context Length Alone Hurts](https://arxiv.org/abs/2510.05381) · [Shadows in the Attention](https://arxiv.org/abs/2505.16894)
> **GitHub:** ⭐249 [chroma-core/context-rot](https://github.com/chroma-core/context-rot) · ⭐2260 [gkamradt/LLMTest_NeedleInAHaystack](https://github.com/gkamradt/LLMTest_NeedleInAHaystack)
> **Blogs:** [Understanding AI — "Context Rot"](https://www.understandingai.org/) · [Anthropic — Context Engineering](https://www.anthropic.com/engineering/context-engineering) · [Inkeep — Fighting Context Rot](https://inkeep.com/blog/fighting-context-rot)

---

## Overview

> *"Context rot is the phenomenon where the performance of large language models degrades as the number of tokens in context grows — models handle the 10,000th token less reliably than the 100th."* — Chroma Technical Report, July 2025

Context rot refers to the systematic degradation of LLM performance as input context length increases. The term was coined by a Hacker News commenter ("Workaccount2") around June 2025 and was systematically studied in the Chroma technical report (Hong, Troynikov, Huber, July 2025). Unlike simple retrieval failure, context rot persists even when retrieval accuracy is perfect — reasoning itself degrades.

---

## Key Papers

### 1. Context Rot: How Increasing Input Tokens Impacts LLM Performance (2025)

> **Paper:** [Chroma Technical Report](https://research.trychroma.com/context-rot) · **GitHub:** ⭐249 [chroma-core/context-rot](https://github.com/chroma-core/context-rot)

**Authors:** Kelly Hong, Anton Troynikov, Jeff Huber (Chroma)

The foundational paper coining and systematically studying "context rot." Extends NIAH testing with LongMemEval and repeated-words experiments to show that performance degrades monotonically with context length, even on simple tasks.

> *"Models handle the 10,000th token in their context less reliably than the 100th."*

### 2. Intelligence Degradation in Long-Context LLMs (2026)

> **Paper:** [arXiv:2601.15300](https://arxiv.org/abs/2601.15300) · **Published:** January 2026

Identifies a **"cliff-like" degradation threshold** at approximately 43% of a model's maximum context length. Below this threshold, performance is stable; above it, performance drops sharply.

| Metric | Description |
|--------|-------------|
| **Critical threshold (L_c)** | ~43% of max context length |
| **Behavior below L_c** | Stable, near-optimal performance |
| **Behavior above L_c** | Cliff-like degradation |

### 3. Context Length Alone Hurts LLM Performance Despite Perfect Retrieval (2025)

> **Paper:** [arXiv:2510.05381](https://arxiv.org/abs/2510.05381) · **Venue:** EMNLP 2025 Findings

Shows that performance drops **7–85%** from sheer context length, even with 100% retrieval accuracy. Context rot is not just a retrieval problem — reasoning itself degrades.

> *"Even when the model perfectly retrieves relevant information, its ability to reason over that information degrades with context length."*

### 4. Shadows in the Attention: Contextual Perturbation and Representation Drift (2025)

> **Paper:** [arXiv:2505.16894](https://arxiv.org/abs/2505.16894)

Studies hallucination dynamics through **representation drift metrics**: Cos-Drift and JS-Drift. Quantifies how intermediate layer representations shift as context grows.

### 5. NoLiMa: Long-Context Evaluation Beyond Literal Matching (2025)

> **Paper:** [arXiv:2502.05167](https://arxiv.org/abs/2502.05167) · **Venue:** ICML 2025 · **Authors:** Adobe Research · **GitHub:** ⭐190 [adobe-research/NoLiMa](https://github.com/adobe-research/NoLiMa)

Shows that NIAH benchmarks **overestimate** real capability because models exploit literal matches. When evaluated beyond literal string matching, long-context performance drops significantly.

### 6. GSM-Infinite: Benchmarking LLM Reasoning Over Infinite Context (2025)

> **Paper:** [arXiv:2502.05252](https://arxiv.org/abs/2502.05252) · **GitHub:** ⭐63 [Infini-AI-Lab/gsm_infinite](https://github.com/Infini-AI-Lab/gsm_infinite)

Benchmarks LLM **reasoning** (not just retrieval) over infinitely increasing context. Measures how mathematical reasoning degrades as distractor context grows.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **Context Rot** | 2025 | Chroma Tech Report | [Link](https://research.trychroma.com/context-rot) | Coins term, systematic study of context degradation |
| **Intelligence Degradation** | 2026 | arXiv | [arXiv:2601.15300](https://arxiv.org/abs/2601.15300) | Cliff-like threshold at ~43% context length |
| **Context Length Alone Hurts** | 2025 | EMNLP 2025 | [arXiv:2510.05381](https://arxiv.org/abs/2510.05381) | 7–85% performance drop even with perfect retrieval |
| **NoLiMa** | 2025 | ICML 2025 | [arXiv:2502.05167](https://arxiv.org/abs/2502.05167) | NIAH overestimates capability beyond literal matching |
| **GSM-Infinite** | 2025 | arXiv | [arXiv:2502.05252](https://arxiv.org/abs/2502.05252) | Reasoning degradation over infinite context |
| **Shadows in the Attention** | 2025 | arXiv | [arXiv:2505.16894](https://arxiv.org/abs/2505.16894) | Representation drift metrics (Cos-Drift, JS-Drift) |

---

## GitHub Repos

| Repo | ⭐ | Description |
|------|-----|-------------|
| [chroma-core/context-rot](https://github.com/chroma-core/context-rot) | 249 | Toolkit for replicating Context Rot study (NIAH extension, LongMemEval, repeated words) |
| [gkamradt/LLMTest_NeedleInAHaystack](https://github.com/gkamradt/LLMTest_NeedleInAHaystack) | 2,260 | Original NIAH benchmark — simple retrieval at varying context lengths |
| [adobe-research/NoLiMa](https://github.com/adobe-research/NoLiMa) | 190 | NoLiMa benchmark — evaluation beyond literal matching |
| [Infini-AI-Lab/gsm_infinite](https://github.com/Infini-AI-Lab/gsm_infinite) | 63 | GSM-∞ — reasoning over infinitely increasing context |

---

## Why Context Rot Happens

| Cause | Description | Source |
|-------|-------------|--------|
| **O(n²) attention scaling** | Every token attends to every other token — attention dispersion grows | Chroma Report |
| **Attention dispersion** | Weights become uniformly distributed at longer lengths | [arXiv:2505.16894](https://arxiv.org/abs/2505.16894) |
| **U-shaped positional bias** | Models favor beginning (primacy) and end (recency) over middle | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) |
| **RoPE extrapolation failures** | Position aliasing causes confusion at extended context lengths | [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) |
| **Training data bias** | Models primarily trained on short-to-medium contexts | [arXiv:2510.05381](https://arxiv.org/abs/2510.05381) |

---

## Mitigation Strategies

| Strategy | Description | Source |
|----------|-------------|--------|
| **Retrieve-Then-Solve** | Recite evidence, then solve from short context — up to 31.2% improvement | Du et al. |
| **Context engineering** | Curate minimal high-signal tokens, use tool-result clearing, memory tools | [Anthropic Engineering Blog](https://www.anthropic.com/engineering/context-engineering) |
| **Found-in-the-Middle calibration** | Calibrate positional attention bias during inference — up to 15pp improvement | [arXiv:2406.16008](https://arxiv.org/abs/2406.16008) |
| **LongReD restoration distillation** | Preserve short-text performance during long-context extension (99.4% vs 92.5%) | [arXiv:2502.07365](https://arxiv.org/abs/2502.07365) |
| **Context compaction** | Summarize and compress context periodically | Inkeep Blog |
| **Multi-agent architectures** | Lead agent coordinates, sub-agents work in clean contexts | Anthropic Engineering Blog |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Context Rot: The Emerging Challenge** | Understanding AI (Timothy B. Lee) | [Link](https://www.understandingai.org/) |
| **Effective Context Engineering for AI Agents** | Anthropic Engineering | [Link](https://www.anthropic.com/engineering/context-engineering) |
| **Fighting Context Rot: Essential Skill** | Inkeep | [inkeep.com/blog/fighting-context-rot](https://inkeep.com/blog/fighting-context-rot) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem** | LLM performance degrades monotonically as context grows — even with perfect retrieval |
| **Root cause** | Attention dispersion, positional bias, RoPE failures, short-context training |
| **Critical threshold** | ~43% of max context length (cliff-like degradation above this) |
| **Best benchmark** | NoLiMa (beyond literal matching), GSM-∞ (reasoning), RULER (multi-task) |
| **Best mitigation** | Retrieve-Then-Solve (31.2%), Found-in-the-Middle calibration (15pp) |
| **Key insight** | Context rot ≠ retrieval failure — reasoning itself degrades with length |

---

*Last updated: 2026-04-20*