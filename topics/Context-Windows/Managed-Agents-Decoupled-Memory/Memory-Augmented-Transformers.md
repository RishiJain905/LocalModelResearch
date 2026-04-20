# Memory-Augmented Transformers

> **Key Papers:** [MemGPT (arXiv:2310.08560)](https://arxiv.org/abs/2310.08560) · [Infini-attention (arXiv:2404.07143)](https://arxiv.org/abs/2404.07143) · [Titans (arXiv:2501.00663)](https://arxiv.org/abs/2501.00663) · [Memorizing Transformers (ICLR 2022)](https://arxiv.org/abs/2203.08913) · [RMT (NeurIPS 2022)](https://arxiv.org/abs/2207.06881)
> **GitHub:** ⭐2,538 [google-deepmind/dnc](https://github.com/google-deepmind/dnc) · ⭐778 [booydar/recurrent-memory-transformer](https://github.com/booydar/recurrent-memory-transformer) · ⭐299 [dingo-actual/infini-transformer](https://github.com/dingo-actual/infini-transformer) · ⭐3,699 [kimiyoung/transformer-xl](https://github.com/kimiyoung/transformer-xl)
> **Survey:** [Memory-Augmented Transformers: A Systematic Review (arXiv:2508.10824)](https://arxiv.org/abs/2508.10824)

---

## Overview

> *"Memory is fundamental to intelligence, enabling learning, reasoning, and adaptability across biological and artificial systems. While Transformer architectures excel at sequence modeling, they face critical limitations in long-range context retention, continual learning, and knowledge integration."* — Systematic Review, arXiv:2508.10824

Memory-Augmented Transformers incorporate **external memory modules** that transformers can read from and write to — distinct from internal KV cache or learned weights. These architectures enable continual learning, true infinite context, and computational universality.

> *"We show that transformer-based large language models are computationally universal when augmented with an external memory."* — Schuurmans, arXiv:2301.04589

---

## Key Architectural Patterns

| Pattern | Mechanism | Key Papers |
|---------|-----------|------------|
| **kNN-based Retrieval** | External memory with approximate nearest neighbors lookup | Memorizing Transformers |
| **Compressive Memory** | Compress KV states into fixed-size matrix | Infini-attention, Compressive Transformer |
| **Persistent Memory Vectors** | Learnable memory tokens augment attention | Augmenting Self-attention (Facebook) |
| **Segment-level Recurrence** | Hidden state reuse across segments | Transformer-XL, RMT, Compressive Transformer |
| **Token-based Memory** | Special memory tokens prepended to input | RMT |
| **Surprise-gated Updates** | Surprise signal triggers memory writing | Titans |
| **Differentiable Neural Computer** | Controller + external memory with read/write gates | DNC |

---

## Key Papers

### 1. MemGPT: Towards LLMs as Operating Systems (2023) ★

> **Paper:** [arXiv:2310.08560](https://arxiv.org/abs/2310.08560) · **Authors:** Packer, Wooders, Lin, Fang, Patil, Stoica, Gonzalez (UC Berkeley)

> *"We treat context windows as a constrained memory resource, and design a memory hierarchy for LLMs analogous to memory tiers used in traditional OSes."*

Virtual context management — pagination between main context (RAM) and external context (disk). Self-directed memory management via function calls.

### 2. Infini-attention: Leave No Context Behind (Google, 2024)

> **Paper:** [arXiv:2404.07143](https://arxiv.org/abs/2404.07143) · **Authors:** Google Research

> *"Combines local attention with compressive memory in a single Transformer block; achieves 114× compression ratio."*

**Key mechanism:** Compresses KV states from past segments into fixed-size memory matrix; retrieves via linear attention. **1B model scales to 1M sequence length; 8B achieves SoTA on 500K book summarization.**

### 3. Titans: Learning to Memorize at Test Time (2025)

> **Paper:** [arXiv:2501.00663](https://arxiv.org/abs/2501.00663) · **Authors:** Behrouz et al.

> *"We present a neural long-term memory module that learns to memorize historical context and helps attention to attend to the current context."*

**Three hyper-heads architecture:**
- **Core:** Short-term attention
- **Long-term Memory:** Neural LTM module (surprise-gated updates)
- **Persistent Memory:** Learnable date-independent parameters

**Key innovation:** O(n) linear scaling vs O(n²) quadratic attention. Learns to memorize at test time.

### 4. Memorizing Transformers (ICLR 2022 Spotlight)

> **Paper:** [arXiv:2203.08913](https://arxiv.org/abs/2203.08913) · **Authors:** Y. Wu et al.

> *"We extend language models with the ability to memorize the internal representations of past inputs."*

kNN-retrievable memory of (key, value) pairs scales to **262K tokens**; uses approximate nearest neighbors lookup into non-differentiable memory.

### 5. Recurrent Memory Transformer (NeurIPS 2022)

> **Paper:** [arXiv:2207.06881](https://arxiv.org/abs/2207.06881) · **Authors:** Bulatov, Kuratov, Burtsev
> **GitHub:** ⭐778 [booydar/recurrent-memory-transformer](https://github.com/booydar/recurrent-memory-transformer)

> *"Memory allows to store and process local and global information."*

Memory-augmented segment-level recurrent Transformer using special memory tokens. Scales to **1M+ tokens**.

**Later extension (BABILong):** arXiv:2402.10790 — **11M token needle finding**.

### 6. Transformer Memory as a Differentiable Search Index (NeurIPS 2022)

> **Paper:** [arXiv:2202.06991](https://arxiv.org/abs/2202.06991)

Transformer memory as associative memory for query-key-value retrieval.

### 7. Memformer: Memory-Augmented Transformer (2020)

> **Paper:** [arXiv:2010.06891](https://arxiv.org/abs/2010.06891) · **GitHub:** ⭐126 [lucidrains/memformer](https://github.com/lucidrains/memformer)

Uses external dynamic memory slots updated with attention. Includes **Memory-Replay BackPropagation (MRBP)** for efficient training.

> *"Memformer utilizes an external dynamic memory to encode and retrieve past information."*

### 8. Augmenting Self-attention with Persistent Memory (Facebook AI, 2019)

> **Paper:** [arXiv:1907.01470](https://arxiv.org/abs/1907.01470) · **Authors:** Sainbayar Sukhbaatar, Edouard Grave, Guillaume Lample, Armand Joulin

> *"We augment the self-attention layers with persistent memory vectors that play a similar role as the feed-forward layer."*

### 9. Compressive Transformer (DeepMind)

> **GitHub:** ⭐164 [lucidrains/compressive-transformer-pytorch](https://github.com/lucidrains/compressive-transformer-pytorch)

Extends Transformer-XL with compressed memory for long-range dependencies.

### 10. Differentiable Neural Computer (DeepMind, 2016)

> **GitHub:** ⭐2,538 [google-deepmind/dnc](https://github.com/google-deepmind/dnc)

Recurrent neural network with external memory and read/write operations. Foundational work bridging neural networks and external memory structures.

### 11. Transformer-XL (CMU/Apple)

> **GitHub:** ⭐3,699 [kimiyoung/transformer-xl](https://github.com/kimiyoung/transformer-xl)

Segment-level recurrence enabling longer dependency capture than vanilla Transformer. Foundation for most recurrent memory approaches.

### 12. Memory-Augmented Transformers: A Systematic Review (2025)

> **Paper:** [arXiv:2508.10824](https://arxiv.org/abs/2508.10824) · **Authors:** Omidi, Huang, Laborieux, et al.

Comprehensive taxonomy of memory types (parameter-encoded, state-based, explicit, hybrid), operations (read/write/forgetting), and mechanisms (attention fusion, gated control, associative retrieval).

### 13. MATTER: Memory-Augmented Transformer Using Heterogeneous Knowledge (2024)

> **Paper:** [arXiv:2406.04670](https://arxiv.org/abs/2406.04670)

Integrates unstructured and semi-structured sources into type-agnostic neural memories. **100× throughput improvement.**

### 14. Memory-Augmented Generative Adversarial Transformers (2024)

> **Paper:** [arXiv:2402.19218](https://arxiv.org/abs/2402.19218)

Extends standard Transformer with additional memory bank.

### 15. Recurrent Memory-Augmented Transformers with Chunked Attention (2025)

> **Paper:** [arXiv:2507.00453](https://arxiv.org/abs/2507.00453)

Hybrid attention fusing full self-attention, chunked attention, and memory attention.

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **MemGPT** | 2023 | arXiv | [arXiv:2310.08560](https://arxiv.org/abs/2310.08560) | OS-style memory hierarchy, virtual context paging |
| **Infini-attention** | 2024 | arXiv | [arXiv:2404.07143](https://arxiv.org/abs/2404.07143) | 114× compression, 1M seq len, compressive memory |
| **Titans** | 2025 | arXiv | [arXiv:2501.00663](https://arxiv.org/abs/2501.00663) | Surprise-gated LTM, O(n) linear scaling |
| **Memorizing Transformers** | 2022 | ICLR 2022 | [arXiv:2203.08913](https://arxiv.org/abs/2203.08913) | kNN KV retrieval, 262K tokens |
| **RMT** | 2022 | NeurIPS | [arXiv:2207.06881](https://arxiv.org/abs/2207.06881) | Segment-level recurrence with memory tokens, 1M+ tokens |
| **Memformer** | 2020 | arXiv | [arXiv:2010.06891](https://arxiv.org/abs/2010.06891) | External dynamic memory, MRBP training |
| **Persistent Memory** | 2019 | arXiv | [arXiv:1907.01470](https://arxiv.org/abs/1907.01470) | Learnable memory vectors augment attention |
| **Compressive Transformer** | 2019 | arXiv | [lucidrains/compressive-transformer-pytorch](https://github.com/lucidrains/compressive-transformer-pytorch) | Compressed memory for long-range dependencies |
| **DNC** | 2016 | Nature | [google-deepmind/dnc](https://github.com/google-deepmind/dnc) | Controller + external memory with read/write gates |
| **Transformer-XL** | 2019 | arXiv | [kimiyoung/transformer-xl](https://github.com/kimiyoung/transformer-xl) | Segment-level recurrence (foundation for memory approaches) |
| **Systematic Review** | 2025 | arXiv | [arXiv:2508.10824](https://arxiv.org/abs/2508.10824) | Comprehensive taxonomy of memory types and mechanisms |
| **MATTER** | 2024 | arXiv | [arXiv:2406.04670](https://arxiv.org/abs/2406.04670) | Heterogeneous knowledge, 100× throughput |
| **Memory Survey** | 2025 | GitHub ⭐1,840 | [Agent-Memory-Paper-List](https://github.com/Shichun-Liu/Agent-Memory-Paper-List) | Token/parametric/latent taxonomy |

---

## GitHub Repos

| Repo | ⭐ | Description |
|------|-----|-------------|
| [google-deepmind/dnc](https://github.com/google-deepmind/dnc) | 2,538 | TensorFlow DNC — controller + external memory |
| [kimiyoung/transformer-xl](https://github.com/kimiyoung/transformer-xl) | 3,699 | Official Transformer-XL — segment-level recurrence |
| [booydar/recurrent-memory-transformer](https://github.com/booydar/recurrent-memory-transformer) | 778 | RMT wrapper for HuggingFace models |
| [dingo-actual/infini-transformer](https://github.com/dingo-actual/infini-transformer) | 299 | PyTorch Infini-Transformer with compressive memory |
| [lucidrains/memorizing-transformers-pytorch](https://github.com/lucidrains/memorizing-transformers-pytorch) | 644 | Memorizing Transformers with kNN retrieval |
| [lucidrains/memformer](https://github.com/lucidrains/memformer) | 126 | Memformer in PyTorch |
| [lucidrains/compressive-transformer-pytorch](https://github.com/lucidrains/compressive-transformer-pytorch) | 164 | Compressive Transformers |
| [xiaowu0162/awesome-long-term-memory](https://github.com/xiaowu0162/awesome-long-term-memory) | — | Curated collection of LTM papers for LLMs |

---

## Blog Posts

| Title | Source | URL |
|-------|--------|-----|
| **How memory augmentation can improve LLMs** | IBM Research Blog | Larimar episodic memory approach |
| **Google's Titans: 4 Ways to Wire Long-Term Memory Into a Transformer** | AI pub substack | Titans architectural breakdown |
| **Titans vs Transformers: Memory-Augmented AI** | massimilianovurro.com | Comparison of memory architectures |
| **Leave No Context Behind — A Comment** | LessWrong | Analysis of Infini-attention paper |
| **Attention vs Memory: Why Transformers Killed the RNN** | Towards AI | Memory vs attention tradeoffs |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core concept** | Transformers with read/write external memory modules (beyond KV cache) |
| **Key mechanisms** | kNN retrieval, compressive memory, persistent vectors, segment recurrence, surprise gating |
| **Infinite context** | Infini-attention (114× compression, 1M seq), RMT (1M+ tokens), Titans |
| **Best survey** | Systematic Review arXiv:2508.10824 — comprehensive taxonomy |
| **Best foundation** | Transformer-XL (segment recurrence) → RMT, Compressive, Infini-attention |
| **Computational universality** | LLM + external memory = computationally universal (Schuurmans 2023) |

---

*Last updated: 2026-04-20*