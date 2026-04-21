# Automatic Prefix Caching (APC) & RadixAttention

> **Papers:** [SGLang arXiv:2312.07104](https://arxiv.org/abs/2312.07104) · [KVLink arXiv:2502.16002](https://arxiv.org/abs/2502.16002) · [Learned Prefix Caching NeurIPS 2025](https://neurips.cc/virtual/2025/poster/117662)  
> **GitHub:** [vLLM](https://github.com/vllm-project/vllm) · [SGLang](https://github.com/sgl-project/sglang)  
> **Blogs:** [LMSYS Blog](https://lmsys.org/blog/2024-01-17-sglang/) · [vLLM Docs](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/)

---

## Overview

> *"Since the KV cache computation depends only on the preceding tokens, identical prefixes produce identical caches — which can be shared across requests."* — SGLang Paper

**Automatic Prefix Caching (APC)** and **RadixAttention** are techniques for reusing KV cache across LLM inference requests that share common prefixes, avoiding redundant computation.

| Technique | Approach | Used By |
|-----------|----------|---------|
| **APC (Hash-based)** | Block-level hashing → O(1) global hash table lookup | vLLM |
| **RadixAttention (Token-level)** | Prefix tree (radix tree) → hierarchical token-level prefix matching | SGLang |

---

## How It Works

### vLLM Hash-Based APC

```
Each KV block hashed by: (parent_hash, block_tokens, extra_hashes)
→ Global hash table maps hash → physical block
→ Block pool + doubly-linked free queue for O(1) allocation/eviction
→ Only full blocks are cached; LRU eviction policy
```

### SGLang RadixAttention

```
KV cache stored in radix tree (token-level prefix tree)
→ Common prefixes stored once; divergent suffixes branch separately
→ LRU eviction recurses from leaf nodes
→ Compatible with paged attention + continuous batching
```

---

## Key Methods

### 1. RadixAttention — SGLang (2023)

> **Paper:** [SGLang: Efficient Execution of Structured Language Model Programs](https://arxiv.org/abs/2312.07104)  
> **GitHub:** [sgl-project/sglang](https://github.com/sgl-project/sglang)

**TL;DR:** Token-level radix tree storing KV cache hierarchically for efficient prefix matching and LRU eviction. Up to **6.4× throughput gain**.

| Aspect | Details |
|--------|---------|
| **Method** | Radix tree (prefix tree) for KV cache |
| **Eviction** | LRU from leaf nodes |
| **Compatibility** | Paged attention + continuous batching |
| **Result** | 6.4× throughput on Mixtral-8x7B |

> *"SGLang: RadixAttention enables 6.4× throughput improvement over vLLM on Mixtral-8x7B."* — [LMSYS Blog](https://lmsys.org/blog/2024-01-17-sglang/)

### 2. KVLink — Precomputed KV Cache (2025)

> **Paper:** [KVLink: Accelerating LLMs via Efficient KV Cache Reuse](https://arxiv.org/abs/2502.16002)

**TL;DR:** Precomputed KV cache concatenation for shared prefixes. Up to **96% TTFT reduction**.

| Aspect | Details |
|--------|---------|
| **Method** | Precomputed KV cache concatenation |
| **Result** | Up to 96% TTFT reduction |

### 3. Learned Prefix Caching (LPC) — NeurIPS 2025

> **Paper:** [NeurIPS 2025 Poster #117662](https://neurips.cc/virtual/2025/poster/117662)

**TL;DR:** ML-based cache eviction policy trained to predict which prefixes to keep.

| Aspect | Details |
|--------|---------|
| **Method** | ML-based cache eviction policy |
| **Result** | 74% TTFT reduction on cache hit |

### 4. KVFlow — Workflow-Aware KV Cache (NeurIPS 2025)

> **Paper:** [NeurIPS 2025 Poster #119883](https://neurips.cc/virtual/2025/poster/119883)

**TL;DR:** Workflow-aware KV cache management for multi-agent LLM applications.

---

## vLLM Integration

```python
from vllm import LLM, SamplingParams

# Enable APC via engine argument
llm = LLM(model="lmsys/longchat-13b-16k", enable_prefix_caching=True)
sampling_params = SamplingParams(temperature=0, max_tokens=100)

# First request processes the full prefix
response1 = llm.generate(LONG_PROMPT + "Question 1?", sampling_params)
# Second request reuses KV cache for LONG_PROMPT — skips recomputation
response2 = llm.generate(LONG_PROMPT + "Question 2?", sampling_params)
```

**Configuration options:**
- `--prefix-caching-hash-algo`: `sha256` (default), `sha256_cbor`, `xxhash`, `xxhash_cbor`
- `cache_salt`: Per-request salt for multi-tenant isolation

---

## Performance Gains & Benchmarks

| Metric | Improvement | Source |
|--------|-------------|--------|
| **Throughput (SGLang vs vLLM)** | Up to **5×** on Llama-7B; **6.4×** on Mixtral-8x7B | [LMSYS Blog](https://lmsys.org/blog/2024-01-17-sglang/) |
| **TTFT reduction (KVLink)** | Up to **96%** vs standard inference | [arXiv:2502.16002](https://arxiv.org/abs/2502.16002) |
| **TTFT reduction (LPC cache hit)** | **74%** reduction | [NeurIPS 2025](https://neurips.cc/virtual/2025/poster/117662) |
| **Cost savings (prompt caching)** | **41–80%** on API costs | [arXiv:2601.06007](https://arxiv.org/html/2601.06007v2) |
| **Per-token TTFT saving** | ~**0.15ms per cached token** | [Glean Blog](https://www.glean.com/blog/glean-kv-caches-llm-latency) |
| **Multi-turn conversations** | ~**10%** throughput gain over vLLM at same context | [Runpod Blog](https://www.runpod.io/blog/sglang-vs-vllm-kv-cache) |

---

## Best Use Cases

1. **Long document Q&A** — Same document + different questions → process doc once
2. **Multi-turn chat** — Shared conversation history across turns
3. **Few-shot learning** — Repeated examples across requests
4. **AI agents** — Shared tool definitions / system prompts
5. **RAG pipelines** — Identical retrieved context across queries

---

## Limitations

- APC only accelerates **prefilling phase** (not decoding phase)
- No benefit when answers are long or prefixes don't match
- GPU memory pressure requires careful LRU eviction
- Hash collisions (mitigated in vLLM v0.11+ with `sha256`)

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **SGLang: Efficient Execution of Structured LLM Programs** | 2023 | arXiv | [link](https://arxiv.org/abs/2312.07104) | RadixAttention; 6.4× throughput gain |
| **KVLink: Efficient KV Cache Reuse** | 2025 | arXiv | [link](https://arxiv.org/abs/2502.16002) | Up to 96% TTFT reduction |
| **Towards Efficient KV Cache Management for Prefix Prefilling** | 2025 | arXiv | [link](https://arxiv.org/abs/2505.21919) | Distributed caching for KVC prefilling |
| **Learned Prefix Caching (LPC)** | 2025 | NeurIPS | [link](https://neurips.cc/virtual/2025/poster/117662) | ML-based eviction; 74% TTFT reduction |
| **KVFlow: Workflow-Aware KV Cache** | 2025 | NeurIPS | [link](https://neurips.cc/virtual/2025/poster/119883) | Multi-agent workflow optimization |
| **LLM Query Scheduling with Prefix Reuse** | 2025 | NeurIPS | [link](https://neurips.cc/virtual/2025/poster/118899) | Formal k-LPM scheduling framework |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [vllm-project/vllm](https://github.com/vllm-project/vllm) | Hash-based APC (`enable_prefix_caching=True`); block-level hashing |
| [sgl-project/sglang](https://github.com/sgl-project/sglang) | RadixAttention with token-level radix tree; up to 5× throughput vs vLLM |
| [vllm/design/prefix_caching.md](https://github.com/vllm-project/vllm/blob/main/docs/design/prefix_caching.md) | Technical design doc for APC in vLLM |
| [vllm/benchmarks/benchmark_prefix_caching.py](https://github.com/vllm-project/vllm/blob/main/benchmarks/benchmark_prefix_caching.py) | Benchmark script for APC efficiency |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **Fast and Expressive LLM Inference with RadixAttention** | LMSYS Blog | [Link](https://lmsys.org/blog/2024-01-17-sglang/) |
| **Automatic Prefix Caching** | vLLM Docs | [Link](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/) |
| **Prefix Caching — SGLang vs vLLM** | Medium | [Link](https://medium.com/byte-sized-ai/prefix-caching-sglang-vs-vllm-token-level-radix-tree-vs-block-level-hashing-b99ece9977a1) |
| **SGLang vs vLLM Multi-Turn** | Runpod Blog | [Link](https://www.runpod.io/blog/sglang-vs-vllm-kv-cache) |
| **Prefix Caching in LLM Inference** | BentoML | [Link](https://bentoml.com/llm/inference-optimization/prefix-caching) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Problem solved** | Redundant KV cache computation for repeated prefixes |
| **Key innovation** | Hash-based APC (vLLM) vs token-level radix tree (SGLang) |
| **Best for** | Long document Q&A, multi-turn chat, few-shot, RAG, AI agents |

---

*Last updated: 2026-04-21*
