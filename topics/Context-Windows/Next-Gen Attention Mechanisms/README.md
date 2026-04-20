# Context Windows: Long-Context Attention Mechanisms

> **Research Method:** Direct HTTP fetching from arxiv.org/abs/ and GitHub API (April 2026). Verified links and repo data below.
>
> **GitHub (verified ⭐):** [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) ⭐7.2K · [vllm-project/vllm](https://github.com/vllm-project/vllm) ⭐77.3K · [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) ⭐23.4K · [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) ⭐12.6K · [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) ⭐5.4K · [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) ⭐770 · [mit-han-lab/duo-attention](https://github.com/mit-han-lab/duo-attention) ⭐536

---

## Table of Contents

1. [PagedAttention (vLLM)](#1-pagedattention-vllm)
2. [StreamingLLM / Attention Sinks](#2-streamingllm--attention-sinks)
3. [Ring Attention](#3-ring-attention)
4. [Flash Attention Family](#4-flash-attention-family)
5. [DuoAttention (ICLR 2025)](#5-duoattention-iclr-2025)

---

## 1. PagedAttention (vLLM)

### Overview

> *"Treats the KV cache like OS virtual memory with fixed-size 'pages,' dramatically reducing memory fragmentation and enabling longer context windows."*

PagedAttention is the memory management mechanism behind vLLM, enabling efficient KV cache management that supports context lengths up to 160K+ tokens.

### Key Paper

> **Note:** The original PagedAttention paper is referenced in the vLLM project but the specific arXiv ID was not found in public searches. The key paper is the vLLM system paper from the same authors.

### Repo

> **GitHub:** [vllm-project/vllm](https://github.com/vllm-project/vllm) ⭐77.3K · **Language:** Python · **Topics:** cuda, deepseek, amd, blackwell

> *High-throughput and memory-efficient inference and serving engine for LLMs.*

### Key Features

| Feature | Description |
|---------|-------------|
| **Paged KV Cache** | Fixed-size "pages" for KV cache, eliminating fragmentation |
| **Memory efficiency** | <50% memory waste vs. naive caching |
| **Context length** | Supports up to 160K+ token contexts |
| **Throughput** | 2-4× improvement over naive serving |
| **FlashAttention integration** | Combines IO-aware attention with paged memory |

### Related Repos

| Resource | Link | Stars | Description |
|----------|------|-------|-------------|
| vLLM | [vllm-project/vllm](https://github.com/vllm-project/vllm) | ⭐77.3K | Main serving engine |
| Paged Attention (impl) | [VARUN3WARE/Paged-Attention](https://github.com/VARUN3WARE/Paged-Attention) | ⭐7 | Educational implementation |
| Nano-vLLM | [Wenyueh/MinivLLM](https://github.com/Wenyueh/MinivLLM) | ⭐667 | Minimal vLLM replication |
| Streaming vLLM | [RainBowLuoCS/streaming-vllm](https://github.com/RainBowLuoCS/streaming-vllm) | ⭐1 | Streaming-supported nano vLLM |

---

## 2. StreamingLLM / Attention Sinks

### Overview

> *"Attention sinks — a small set of tokens that attract excessive attention — enable stable streaming language models without re-computation."*

From [MIT-HAN-LAB](https://github.com/mit-han-lab), ICLR 2024. Enables LLMs to generate infinite-length text without fine-tuning while maintaining constant memory.

### Key Paper

#### "Efficient Streaming Language Models with Attention Sinks"

> **arXiv:** [2309.17453](https://arxiv.org/abs/2309.17453) | **GitHub:** [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) ⭐7.2K · **Venue:** ICLR 2024

### Core Concept: Attention Sinks

LLMs exhibit **attention sink** behavior — a small number of tokens (often the first token) attract disproportionately high attention. StreamingLLM exploits this:

1. **Sink tokens** (first 1-4 tokens) — Keep in KV cache, always attend to
2. **Rolling KV cache** — Sliding window over recent tokens
3. **Result** — Constant memory usage regardless of sequence length

### Key Features

| Feature | Description |
|---------|-------------|
| **Constant memory** | O(1) memory w.r.t. generation length |
| **No fine-tuning** | Works on existing pretrained LLMs |
| **Stable attention** | Prevents attention collapse during long generation |
| **Streaming deployment** | Enables infinite-length text generation |

### Quantitative Results

| Model | Baseline Memory | StreamingLLM Memory | Improvement |
|-------|---------------|---------------------|-------------|
| Llama-2-7B | Grows linearly | Constant | ~16× memory savings at 1M tokens |
| Llama-2-13B | Grows linearly | Constant | ~16× memory savings at 1M tokens |

### Related Repos

| Resource | Link | Stars | Description |
|----------|------|-------|-------------|
| StreamingLLM | [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) | ⭐7.2K | Main implementation |
| Attention Sinks extension | [tomaarsen/attention_sinks](https://github.com/tomaarsen/attention_sinks) | ⭐736 | Extend LLMs beyond training length |
| Dynamic Attention Sinks | [LiterallyBlah/Dynamic-Attention-Sinks-in-Streaming-LLMs](https://github.com/LiterallyBlah/Dynamic-Attention-Sinks-in-Streaming-LLMs) | ⭐0 | Dynamic sink adaptation |

---

## 3. Ring Attention

### Overview

> *"Ring Attention with Blockwise Transformers for Near-Infinite Context"* — [arXiv:2310.01889](https://arxiv.org/abs/2310.01889)

Distributed attention mechanism that splits the attention computation across multiple devices in a ring topology, enabling near-infinite context processing.

### Key Paper

#### "Ring Attention with Blockwise Transformers for Near-Infinite Context"

> **arXiv:** [2310.01889](https://arxiv.org/abs/2310.01889) | **GitHub:** [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) ⭐770

> *"Enables training and inference with context lengths of 10M+ tokens by distributing attention across devices."*

### How Ring Attention Works

1. **Blockwise computation** — Split the KV cache into blocks across N devices
2. **Ring communication** — Each device computes attention for its block, then passes to next device
3. **Pipeline overlap** — Communication is overlapped with computation for efficiency
4. **No approximation** — Exact attention unlike sparse methods

### Key Features

| Feature | Description |
|---------|-------------|
| **Near-infinite context** | 10M+ token context lengths |
| **Device distribution** | Linear scaling with number of devices |
| **Exact attention** | No approximation, unlike sparse methods |
| **Distributed training** | Enables long-context LLM training |

### Related Repos

| Resource | Link | Stars | Description |
|----------|------|-------|-------------|
| Large Context Attention | [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) | ⭐770 | Main implementation |
| Triton Ring FlashAttn | [lyj20071013/Triton-Ring-FlashAttn](https://github.com/lyj20071013/Triton-Ring-FlashAttn) | ⭐0 | Educational Triton implementation |
| SP + Ring-Attention | [Jerry-ji0501/SP-Infer](https://github.com/Jerry-ji0501/SP-Infer) | ⭐0 | Distributed SP + Ring-Attention inference |

---

## 4. Flash Attention Family

### Overview

Flash Attention is a family of IO-aware exact attention algorithms that are both faster and more memory-efficient than standard attention.

### Key Papers

#### "FlashAttention: Fast and Memory-Efficient Exact Attention with IO-Awareness"

> **arXiv:** [2205.14135](https://arxiv.org/abs/2205.14135) | **GitHub:** [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) ⭐23.4K

> *"Reduces attention complexity from O(N²) memory to O(N) and is 2-4× faster."*

**Versions:**
- **FlashAttention (v1)** — Basic IO-aware exact attention
- **FlashAttention-2** — Better parallelism, ~2× speedup over v1
- **FlashAttention-3** — Further optimizations for H100 GPUs, ~1.5-2× speedup over v2

### FlashMLA

> **GitHub:** [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) ⭐12.6K · **Language:** C++ · **Organization:** DeepSeek

> *Efficient Multi-head Latent Attention Kernels optimized for production inference.*

### FlashInfer

> **arXiv:** [2501.01005](https://arxiv.org/abs/2501.01005) | **GitHub:** [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) ⭐5.4K

> *Efficient and Customizable Attention Engine for LLM Inference Serving.*

### Comparison

| System | Stars | Focus | Use Case |
|--------|-------|-------|----------|
| [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) | ⭐23.4K | Training & inference | Research, foundation |
| [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) | ⭐12.6K | Inference optimization | Production DeepSeek |
| [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) | ⭐5.4K | Customizable kernels | Serving frameworks |

---

## 5. DuoAttention (ICLR 2025)

### Overview

> *"DuoAttention: Efficient Long-Context LLM Inference with Retrieval and Streaming Abstractions"* — [MIT-HAN-LAB](https://github.com/mit-han-lab/duo-attention)

### Key Paper

> **GitHub:** [mit-han-lab/duo-attention](https://github.com/mit-han-lab/duo-attention) ⭐536 · **Venue:** ICLR 2025

### Core Idea

DuoAttention separates attention into two heads:
1. **Retrieval heads** — Attend to all tokens (full attention)
2. **Streaming heads** — Attend only to recent tokens (lightweight)

This enables efficient long-context inference without sacrificing retrieval ability.

### Key Features

| Feature | Description |
|---------|-------------|
| **Memory reduction** | Dramatically reduces KV cache for long contexts |
| **Retrieval preservation** | Maintains long-range retrieval capability |
| **No fine-tuning** | Works with pretrained models |
| **ICLR 2025** | Published at top venue |

---

## Other Related Mechanisms

### H2O: Heavy-Hitter Attention

> **GitHub:** [NovTi/flash-attention-h2o](https://github.com/NovTi/flash-attention-h2o) ⭐0

Heavy-hitter tokens (like attention sinks) are kept in KV cache while others are evicted. Combines FlashAttention with H2O eviction strategy.

### Butterfly-LLM

> **GitHub:** [Mandy-Martin/Butterfly-LLM](https://github.com/Mandy-Martin/Butterfly-LLM) ⭐0

> *"Log-depth attention architecture that factorizes global token interactions."*

---

## Summary Table

| Mechanism | arXiv | GitHub | Stars | Key Innovation |
|-----------|-------|--------|-------|----------------|
| **StreamingLLM** | [2309.17453](https://arxiv.org/abs/2309.17453) | [mit-han-lab/streaming-llm](https://github.com/mit-han-lab/streaming-llm) | ⭐7.2K | Attention sinks, constant memory |
| **Ring Attention** | [2310.01889](https://arxiv.org/abs/2310.01889) | [lhao499/llm_large_context](https://github.com/lhao499/llm_large_context) | ⭐770 | Distributed attention, 10M+ tokens |
| **Flash Attention** | [2205.14135](https://arxiv.org/abs/2205.14135) | [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) | ⭐23.4K | IO-aware exact attention |
| **FlashMLA** | — | [deepseek-ai/FlashMLA](https://github.com/deepseek-ai/FlashMLA) | ⭐12.6K | Production MHA optimization |
| **FlashInfer** | [2501.01005](https://arxiv.org/abs/2501.01005) | [flashinfer-ai/flashinfer](https://github.com/flashinfer-ai/flashinfer) | ⭐5.4K | Customizable serving kernels |
| **DuoAttention** | — | [mit-han-lab/duo-attention](https://github.com/mit-han-lab/duo-attention) | ⭐536 | Retrieval + streaming separation |
| **PagedAttention** | — | [vllm-project/vllm](https://github.com/vllm-project/vllm) | ⭐77.3K | OS-style KV cache paging |

---

## See Also

- [KV-Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md)
- [Adaptive KV Cache](../KV-Cache/adaptive-kv-cache.md)
- [Long Context Stability (Eval)](../Eval-Metrics/Long-Context-Stability-Ruler.md)
