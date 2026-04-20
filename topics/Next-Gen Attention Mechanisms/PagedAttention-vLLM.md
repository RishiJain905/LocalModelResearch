# PagedAttention (vLLM)

> **Research Method:** Direct HTTP fetching from GitHub API (April 2026).
>
> **GitHub:** [vllm-project/vllm](https://github.com/vllm-project/vllm) ⭐77.3K · [VARUN3WARE/Paged-Attention](https://github.com/VARUN3WARE/Paged-Attention) ⭐7

---

## Overview

> *"Treats the KV cache like OS virtual memory with fixed-size 'pages,' dramatically reducing memory fragmentation and enabling longer context windows."*

PagedAttention is the memory management mechanism behind [vLLM](https://github.com/vllm-project/vllm), enabling efficient KV cache management that supports context lengths up to 160K+ tokens.

---

## Key Paper

> The original PagedAttention paper is published as the **vLLM system paper** by the same authors (Kwon et al., 2023). The key concept is treating KV cache like operating system virtual memory.

---

## How It Works

### Traditional KV Cache

| Problem | Impact |
|---------|--------|
| Contiguous memory allocation | Memory fragmentation as cache grows |
| Fixed max sequence length | Can't exceed pre-allocated memory |
| No sharing between requests | Wasted memory on duplicate prefixes |

### Paged KV Cache

```
Traditional:    [Request 1: KV KV KV KV KV......] <- memory wasted
               [Request 2:       KV KV KV KV......] <- can't reuse

Paged:         [Page 1: KV KV KV KV] [Page 2: KV KV KV KV]
               [Page 3: KV KV KV KV] [Page 4: KV KV KV KV]
               Sharing via copy-on-write for common prefixes
```

### Key Innovation

1. **Fixed-size pages** (typically 16 tokens/page) — Like OS pages
2. **Non-contiguous storage** — Pages scattered in physical memory
3. **Copy-on-write** — Shared KV for common prefixes across requests
4. **No memory wasted** — Exact allocation matching usage

---

## Key Features

| Feature | Description | Impact |
|---------|-------------|--------|
| **Paged KV Cache** | Fixed-size "pages" for KV cache | Eliminates fragmentation |
| **Memory efficiency** | <50% memory waste vs. naive caching | 2× more requests served |
| **Context length** | Supports up to 160K+ token contexts | Long document understanding |
| **Throughput** | 2-4× improvement over naive serving | Lower latency |
| **FlashAttention integration** | Combines IO-aware attention | Best of both worlds |
| **Prefix sharing** | Copy-on-write for common prefixes | Efficient multi-turn |

---

## Quantitative Results

### Memory Efficiency

| Setup | Naive KV Cache | vLLM Paged | Savings |
|-------|---------------|------------|---------|
| 16 concurrent requests | ~100% peak memory | ~50% peak memory | 50% reduction |
| 1000 concurrent requests | OOM | ~80% peak memory | Infinite |

### Throughput

| Metric | Naive | vLLM | Improvement |
|--------|-------|------|-------------|
| Throughput (req/s) | 1.0× | 2-4× | 2-4× faster |
| Time-to-first-token | Baseline | Same | — |
| Memory per request | Variable | Fixed page size | Predictable |

---

## Related Repos

| Resource | Link | Stars | Description |
|----------|------|-------|-------------|
| **vLLM** | [vllm-project/vllm](https://github.com/vllm-project/vllm) | ⭐77.3K | Main serving engine with PagedAttention |
| Paged Attention (impl) | [VARUN3WARE/Paged-Attention](https://github.com/VARUN3WARE/Paged-Attention) | ⭐7 | Educational implementation |
| Nano-vLLM | [Wenyueh/MinivLLM](https://github.com/Wenyueh/MinivLLM) | ⭐667 | Minimal vLLM replication |
| Streaming vLLM | [RainBowLuoCS/streaming-vllm](https://github.com/RainBowLuoCS/streaming-vllm) | ⭐1 | Streaming-supported nano vLLM |
| Micro-vLLM | [mukhal/micro-vllm](https://github.com/mukhal/micro-vllm) | ⭐2 | Single-file KV cache paging |

---

## See Also

- [README](./README.md) — Context Windows overview
- [StreamingLLM](./StreamingLLM-Attention-Sinks.md) — Attention sinks for infinite generation
- [Flash Attention](./Flash-Attention-Family.md) — IO-aware exact attention
- [KV-Cache Eviction](../Non-Quant-Compression/Token-Context-Compression/KV-Cache-Eviction.md)
