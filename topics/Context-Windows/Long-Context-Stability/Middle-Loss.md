# The "Middle-Loss" Problem (Lost in the Middle)

> **Papers:** [Lost in the Middle (TACL 2023)](https://arxiv.org/abs/2307.03172) · [Found in the Middle (ACL 2024)](https://arxiv.org/abs/2406.16008) · [Never Lost in the Middle (ACL 2024)](https://arxiv.org/abs/2311.09198) · [Exact Theory of Position Bias (2025)](https://arxiv.org/abs/2603.10123)
> **GitHub:** ⭐376 [nelson-liu/lost-in-the-middle](https://github.com/nelson-liu/lost-in-the-middle) · ⭐2260 [gkamradt/LLMTest_NeedleInAHaystack](https://github.com/gkamradt/LLMTest_NeedleInAHaystack) · ⭐2 [hejunqing/never-lost-in-the-middle](https://github.com/hejunqing/never-lost-in-the-middle)
> **Blogs:** [David William Silva — Context Crisis](https://davidwsilva.substack.com/p/lost-in-the-middle-the-context-crisis) · [Nick Femia — U-Shape](https://nickfemia.substack.com/p/context-window-u-shape) · [dev.to — Why LLMs Ignore the Middle](https://dev.to/thousand_miles_ai/the-lost-in-the-middle-problem-why-llms-ignore-the-middle-of-your-context-window-3al2)

---

## Overview

> *"Changing the location of relevant information within the language model's input context results in a U-shaped performance curve — models are better at using relevant information that occurs at the very beginning (primacy bias) or end of its input context (recency bias), and performance degrades significantly when models must access and use information located in the middle of its input context."* — Liu et al., 2023

The "Middle-Loss" problem (also called "Lost in the Middle") describes a **U-shaped performance curve** where LLMs retrieve information well from the beginning and end of long contexts, but fail for information in the middle. This is one of the most well-studied aspects of long-context degradation.

---

## Key Papers

### 1. Lost in the Middle: How Language Models Use Long Contexts (2023) ★ Seminal

> **Paper:** [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) · **Authors:** Nelson F. Liu, Kevin Lin, John Hewitt, Ashwin Paranjape, Michele Bevilacqua, Fabio Petroni, Percy Liang
> **Venue:** TACL 2023 (Stanford/Meta AI) · **GitHub:** ⭐376 [nelson-liu/lost-in-the-middle](https://github.com/nelson-liu/lost-in-the-middle)

The foundational paper that named and characterized the phenomenon. Demonstrated the U-shaped performance curve across multiple models and tasks:

> *"Models are better at using relevant information that occurs at the very beginning (primacy bias) or end of its input context (recency bias), and performance degrades significantly when models must access and use information located in the middle."*

**Key findings:**
- U-shaped curve holds across GPT-3.5, Claude, MPT, and Falcon models
- Effect persists regardless of model size
- Multi-document QA and key-value retrieval both affected
- Not just a retrieval problem — reasoning degrades too

### 2. Found in the Middle: Calibrating Positional Attention Bias (2024)

> **Paper:** [arXiv:2406.16008](https://arxiv.org/abs/2406.16008) · **Authors:** Chao-Hsiu Hsieh, Simeng Sun, Samuel Kriman, Shantanu Acharya, Dima Rekesh, Fei Jia, Yang Zhang, Boris Ginsburg
> **Venue:** ACL Findings 2024 (MIT / Google Cloud AI)

Establishes that the Lost-in-the-Middle phenomenon is caused by **intrinsic U-shaped attention bias** — tokens at the beginning and end receive disproportionately high attention regardless of relevance.

> *"We establish a connection between lost-in-the-middle to LLMs' intrinsic attention bias: LLMs exhibit a U-shaped attention bias where the tokens at the beginning and at the end of its input receive higher attention, regardless of their relevance."*

**Proposed solution:** "Found-in-the-Middle" calibration mechanism that disentangles positional attention bias — **up to 15 percentage points improvement** over existing methods.

### 3. Never Lost in the Middle: Position-Agnostic Decompositional Training (2024)

> **Paper:** [arXiv:2311.09198](https://arxiv.org/abs/2311.09198) · **Authors:** Junqing He, Kunhao Zheng, et al. · **Venue:** ACL 2024 Main Conference
> **GitHub:** [hejunqing/never-lost-in-the-middle](https://github.com/hejunqing/never-lost-in-the-middle)

Proposes **position-agnostic decompositional training** to even out attention scores across the input context, reducing positional bias.

### 4. Lost in the Middle, and In-Between: Multi-Hop QA (2024)

> **Paper:** [arXiv:2412.10079](https://arxiv.org/abs/2412.10079)

Extends the Lost-in-the-Middle problem to **multi-hop reasoning** where multiple reasoning "hops" over relevant information are required:

> *"We demonstrate the effects of the 'lost in the middle' problem in the multi-hop question answering setting — in which multiple reasoning 'hops' over relevant information are required."*

### 5. Lost in the Middle at Birth: An Exact Theory (2025)

> **Paper:** [arXiv:2603.10123](https://arxiv.org/abs/2603.10123) · **Author:** Borun D. Chowdhury

Provides a **formal mathematical theory** showing the U-shaped positional bias is an inherent property of transformer architecture, not just a training artifact:

> *"The 'Lost in the Middle' phenomenon — a U-shaped performance curve where LLMs retrieve well from the beginning and end..."*

### 6. A Residual-Aware Theory of Position Bias in Transformers (2025)

> **Paper:** [arXiv:2602.16837](https://arxiv.org/abs/2602.16837) · **Authors:** Hanna Herasimchyk, Robin Labryga, Thomas Prusina, Stefan Laue

Explains that **residual connections** prevent attention weight collapse to the first token and produce U-shaped positional bias at finite depth.

### 7. Distance Between Relevant Information Pieces Causes Bias (2025)

> **Paper:** [arXiv:2410.14641](https://arxiv.org/abs/2410.14641) · **Venue:** ACL 2025 Findings

Shows that the **distance between multiple relevant information pieces**, not just absolute position, causes bias — wider spacing between relevant facts hurts performance.

### 8. StreamingLLM: Attention Sinks (2023)

> **Paper:** [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) · **Authors:** Xiao et al.

Introduces **"attention sinks"** — explains why initial tokens receive disproportionately high attention, which is a root cause of primacy bias:

> *"The first few tokens receive disproportionately high attention, acting as 'attention sinks' that stabilize the attention distribution."*

### 9. PAPEFT: Position-Aware Parameter Efficient Fine-Tuning (2024)

> **Paper:** [arXiv:2404.01430](https://arxiv.org/abs/2404.01430)

Introduces data augmentation (permuting document ordering) and a position-aware fine-tuning module with trainable adapter to address positional bias.

### 10. Lost in the Middle in Long-Text Generation (2025)

> **Paper:** [arXiv:2503.06868](https://arxiv.org/abs/2503.06868)

Extends the phenomenon from **retrieval/reading to generation** tasks. The middle-loss problem isn't just about understanding — it affects what LLMs produce.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **Lost in the Middle** | 2023 | TACL | [arXiv:2307.03172](https://arxiv.org/abs/2307.03172) | Seminal paper — U-shaped performance curve |
| **Found in the Middle** | 2024 | ACL Findings | [arXiv:2406.16008](https://arxiv.org/abs/2406.16008) | Attention calibration — 15pp improvement |
| **Never Lost in the Middle** | 2024 | ACL | [arXiv:2311.09198](https://arxiv.org/abs/2311.09198) | Position-agnostic decompositional training |
| **Multi-Hop QA** | 2024 | arXiv | [arXiv:2412.10079](https://arxiv.org/abs/2412.10079) | Extends LITM to multi-hop reasoning |
| **Exact Theory** | 2025 | arXiv | [arXiv:2603.10123](https://arxiv.org/abs/2603.10123) | Mathematical proof — U-shape is inherent to transformers |
| **Residual-Aware Theory** | 2025 | arXiv | [arXiv:2602.16837](https://arxiv.org/abs/2602.16837) | Residual connections produce U-shaped bias |
| **Distance Causes Bias** | 2025 | ACL Findings | [arXiv:2410.14641](https://arxiv.org/abs/2410.14641) | Inter-fact distance, not just position, causes bias |
| **StreamingLLM** | 2023 | arXiv | [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) | Attention sinks — root cause of primacy bias |
| **PAPEFT** | 2024 | arXiv | [arXiv:2404.01430](https://arxiv.org/abs/2404.01430) | Position-aware fine-tuning with adapter |
| **LITM in Generation** | 2025 | arXiv | [arXiv:2503.06868](https://arxiv.org/abs/2503.06868) | Extends to generation, not just retrieval |

---

## GitHub Repos

| Repo | ⭐ | Description |
|------|-----|-------------|
| [nelson-liu/lost-in-the-middle](https://github.com/nelson-liu/lost-in-the-middle) | 376 | Official code and data for the seminal paper (NQ-open, key-value retrieval) |
| [gkamradt/LLMTest_NeedleInAHaystack](https://github.com/gkamradt/LLMTest_NeedleInAHaystack) | 2,260 | Original NIAH benchmark — standard test for positional retrieval |
| [hejunqing/never-lost-in-the-middle](https://github.com/hejunqing/never-lost-in-the-middle) | 2 | ACL 2024 — position-agnostic decompositional training |
| [thunlp/LongPiBench](https://github.com/thunlp/LongPiBench) | 0 | Positional bias benchmark at 32K–256K tokens |
| [NVIDIA/RULER](https://github.com/NVIDIA/RULER) | 1,514 | RULER benchmark — comprehensive long-context evaluation |
| [OpenBMB/InfiniteBench](https://github.com/OpenBMB/InfiniteBench) | 381 | Extends evaluation beyond 100K tokens |
| [Xnhyacinth/Awesome-LLM-Long-Context-Modeling](https://github.com/Xnhyacinth/Awesome-LLM-Long-Context-Modeling) | 1,968 | Curated list of papers/repos on long-context LLM research |

---

## Mitigation Strategies

| Strategy | Method | Improvement | Source |
|----------|--------|-------------|--------|
| **Found-in-the-Middle** | Calibrate U-shaped attention bias | +15pp | [arXiv:2406.16008](https://arxiv.org/abs/2406.16008) |
| **Position-agnostic training** | Decompositional training to even attention | Significant | [arXiv:2311.09198](https://arxiv.org/abs/2311.09198) |
| **Document reordering** | Place relevant docs at beginning and end | Practical | [LangChain LongContextReorder](https://reference.langchain.com/python/langchain-community/document_transformers/long_context_reorder/LongContextReorder) |
| **PAPEFT** | Position-aware PEFT with data augmentation | Moderate | [arXiv:2404.01430](https://arxiv.org/abs/2404.01430) |
| **StreamingLLM attention sinks** | Preserve initial tokens for stable streaming | N/A | [arXiv:2309.17453](https://arxiv.org/abs/2309.17453) |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **The Context Crisis of LLMs** | David William Silva (Substack) | [Link](https://davidwsilva.substack.com/p/lost-in-the-middle-the-context-crisis) |
| **Context Window U-Shape** | Nick Femia (Substack) | [Link](https://nickfemia.substack.com/p/context-window-u-shape) |
| **Why LLMs Ignore the Middle** | dev.to | [Link](https://dev.to/thousand_miles_ai/the-lost-in-the-middle-problem-why-llms-ignore-the-middle-of-your-context-window-3al2) |
| **Context Window Optimization** | AI Carma | [Link](https://aicarma.com/blog/context-window-optimization/) |
| **Solving LITM for RAG** | GetMaxim | [Link](https://www.getmaxim.ai/articles/solving-the-lost-in-the-middle-problem-advanced-rag-techniques-for-long-context-llms/) |
| **Found in the Middle** | Google Research | [Link](https://research.google/pubs/found-in-the-middle-calibrating-positional-attention-bias-improves-long-context-utilization/) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem** | LLMs exhibit U-shaped retrieval — good at beginning/end, poor in the middle |
| **Root cause** | Inherent U-shaped attention bias in transformer architecture |
| **Formal proof** | Mathematical theory confirms bias is architectural, not just training-related |
| **Best mitigation** | Found-in-the-Middle calibration (+15pp), position-agnostic training |
| **Practical fix** | Document reordering (LangChain LongContextReorder) |
| **Key insight** | Both absolute position AND inter-fact distance cause bias |

---

*Last updated: 2026-04-20*