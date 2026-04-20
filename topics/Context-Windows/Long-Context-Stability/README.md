# Long-Context Stability & "Rot"

> *"Models handle the 10,000th token in their context less reliably than the 100th."* — Context Rot, Chroma 2025

Research on how LLM performance degrades as context length grows — covering context rot, the middle-loss (U-shaped attention) phenomenon, and the benchmarks that expose these failures.

---

## Topics

| Topic | Description | Key Papers |
|-------|-------------|------------|
| [**Context Rot**](./Context-Rot.md) | Systematic performance degradation as input tokens increase — even with perfect retrieval | Chroma Report (2025), Intelligence Degradation (2026), Context Length Alone Hurts (EMNLP 2025) |
| [**The "Middle-Loss"**](./Middle-Loss.md) | U-shaped attention curve — models favor beginning/end, fail in the middle | Lost in the Middle (TACL 2023), Found in the Middle (ACL 2024), Exact Theory (2025) |
| [**RULER Benchmarking**](./RULER-Benchmarking.md) | NVIDIA's multi-task benchmark exposing the gap between claimed and effective context length | RULER (COLM 2024), OneRuler (26 languages), HELMET |

---

## Quick Reference

### The Problem Space

| Phenomenon | What Happens | First Studied |
|------------|-------------|---------------|
| **Context Rot** | Performance degrades monotonically with context length, even with perfect retrieval | Chroma, July 2025 |
| **Middle-Loss (Lost in the Middle)** | U-shaped curve — good at beginning/end, poor in middle | Liu et al., TACL 2023 |
| **Cliff Degradation** | ~43% of max context = critical threshold, sharp drop above it | arXiv:2601.15300, 2026 |
| **Effective vs. Claimed Context** | Models claiming 1M tokens may have effective context of only 4K–16K | RULER, COLM 2024 |

### Key Metrics & Benchmarks

| Benchmark | What It Tests | Paper |
|-----------|--------------|-------|
| **RULER** | 13 tasks: retrieval, multi-hop, aggregation, QA | [arXiv:2404.06654](https://arxiv.org/abs/2404.06654) |
| **NIAH** | Simple needle retrieval from haystack | [github.com/gkamradt](https://github.com/gkamradt/LLMTest_NeedleInAHaystack) |
| **NoLiMa** | Beyond literal matching — exposes NIAH overestimation | [arXiv:2502.05167](https://arxiv.org/abs/2502.05167) |
| **GSM-∞** | Reasoning degradation over infinite context | [arXiv:2502.05252](https://arxiv.org/abs/2502.05252) |
| **LongMemEval** | Long-term interactive memory | [arXiv:2410.10813](https://arxiv.org/abs/2410.10813) |
| **InfiniteBench** | Beyond 100K tokens | [arXiv:2402.13718](https://arxiv.org/abs/2402.13718) |
| **BABILong** | Reasoning up to 10M tokens | [arXiv:2406.07801](https://arxiv.org/abs/2406.07801) |

### Mitigation Strategies Quick Reference

| Strategy | Best For | Improvement | Source |
|----------|----------|-------------|--------|
| **Retrieve-Then-Solve** | Context rot | +31.2% | Du et al. |
| **Found-in-the-Middle** | Middle-loss | +15pp | [arXiv:2406.16008](https://arxiv.org/abs/2406.16008) |
| **Position-agnostic training** | Middle-loss | Significant | [arXiv:2311.09198](https://arxiv.org/abs/2311.09198) |
| **Document reordering** | RAG systems | Practical | LangChain LongContextReorder |
| **Context engineering** | Agent systems | Varies | Anthropic Engineering Blog |
| **LongReD distillation** | Short-text preservation | 99.4% vs 92.5% | [arXiv:2502.07365](https://arxiv.org/abs/2502.07365) |

---

*Last updated: 2026-04-20*