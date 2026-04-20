# KV-Cache Reuse (RAGCache)

> **Key Papers:** [RAGCache (arXiv:2404.12457)](https://arxiv.org/abs/2404.12457) · [CacheBlend (arXiv:2405.16444)](https://arxiv.org/abs/2405.16444) · [CacheGen (arXiv:2310.07240)](https://arxiv.org/abs/2310.07240) · [DroidSpeak (arXiv:2411.02820)](https://arxiv.org/abs/2411.02820)
> **GitHub:** ⭐8,022 [LMCache/LMCache](https://github.com/LMCache/LMCache) · ⭐178 [YaoJiayi/CacheBlend](https://github.com/YaoJiayi/CacheBlend) · ⭐158 [uchi-jcl/CacheGen](https://github.com/uchi-jcl/CacheGen)
> **Blogs:** [LMCache Blog — CacheGen](https://blog.lmcache.ai/en/2025/07/31/cachegen-store-your-kv-cache-on-disk-or-s3-load-blazingly-fast/) · [vLLM Production Stack — Sharing KV Cache](https://docs.vllm.ai/projects/production-stack/en/latest/use_cases/sharing-kv-cache.html) · [BentoML — Prefix Caching](https://bentoml.com/llm/inference-optimization/prefix-caching/)

---

## Overview

> *"RAGCache reduces TTFT by caching the KV cache of frequently retrieved documents... leverages the retrieval pattern to organize and cache the intermediate states of retrieved knowledge in a knowledge tree across the GPU and host memory hierarchy."* — Jin et al., arXiv:2404.12457

KV-Cache reuse systems avoid redundant computation by caching, sharing, and reusing KV cache states across requests. In RAG settings where the same documents are retrieved across queries, and in multi-turn conversations with shared prefixes, KV cache reuse can reduce TTFT by **2–3×** and improve throughput by **2–5×**.

---

## Key Papers

### 1. RAGCache: Efficient Knowledge Caching for RAG (2024)

> **Paper:** [arXiv:2404.12457](https://arxiv.org/abs/2404.12457) · **Authors:** Chao Jin, Zili Zhang, Xuanlin Jiang, Fangyue Liu, Xin Liu, Xuanzhe Liu, Xin Jin (Peking University, ByteDance) · **Also at:** [ACM](https://dl.acm.org/doi/10.1145/3768628)
> **No official open-source repo** (RAGCacheSim simulator available on ScienceDirect)

**Three key innovations:**
1. **Knowledge tree** — organizes KV cache by document retrieval patterns
2. **PGDSF eviction policy** — Prefix-Gain-aware DSF (Dual-Size-Frequency) eviction
3. **Dynamic speculative pipelining** — overlaps retrieval and prefill to hide latency

### 2. CacheBlend: Fast LLM Serving for RAG with Cached Knowledge Fusion (2024)

> **Paper:** [arXiv:2405.16444](https://arxiv.org/abs/2405.16444) · **Authors:** Jiayi Yao et al. (UChicago) · **Venue:** EuroSys 2025 · **GitHub:** ⭐178 [YaoJiayi/CacheBlend](https://github.com/YaoJiayi/CacheBlend)

> *"CacheBlend reduces TTFT by 2.2–3.3× and increases inference throughput by 2.8–5× from full KV recompute without compromising generation quality."*

**Key innovation:** Fuses multiple pre-computed KV caches regardless of prefix match by selectively recomputing a small fraction of tokens. Works for **non-prefix** KV cache reuse — unlike prefix caching which requires identical prefixes.

### 3. CacheGen: KV Cache Compression and Streaming (2023)

> **Paper:** [arXiv:2310.07240](https://arxiv.org/abs/2310.07240) · **Authors:** Yuhan Liu, Hanchen Li, et al. (UChicago, Microsoft, Stanford) · **Venue:** ACM SIGCOMM 2024
> **GitHub:** ⭐158 [uchi-jcl/CacheGen](https://github.com/uchi-jcl/CacheGen) · **Now integrated into LMCache**

> *"CacheGen uses a custom tensor encoder, leveraging KV cache's distributional properties... adapts the compression level of different parts of a KV cache to cope with changes in available bandwidth."*

**Now in LMCache:** *"CacheGen compresses your KV cache up to 3× smaller than quantization... cuts TTFT by up to 3×"*

### 4. DroidSpeak: Cross-LLM KV Cache Sharing (2024)

> **Paper:** [arXiv:2411.02820](https://arxiv.org/abs/2411.02820) · **Authors:** Yuhan Liu, Yuyang Huang, Jiayi Yao, et al. (UChicago, Microsoft) · **Venue:** NSDI 2026

> *"DroidSpeak enables KV cache reuse across distributed nodes running inference of different LLMs... brings down computation latency by up to 3.1×, increases throughput by 4×."*

**Novelty:** First system enabling KV cache sharing **between different LLMs** (e.g., specialized agents), not just same-model prefix reuse.

### 5. CacheClip: Accelerating RAG with Effective KV Cache Reuse (2025)

> **Paper:** [arXiv:2510.10129](https://arxiv.org/abs/2510.10129) · **Authors:** Bin Yang, Qiuyu Leng, Jun Zeng, Zhenhua Wu

> *"Existing KV cache reuse methods face a fundamental trade-off: prefix caching requires identical prefixes that rarely occur in RAG scenarios... CacheClip integrates three techniques: auxiliary-model-guided token selection for selective KV cache recomputation."*

### 6. FusionRAG: From Prefix Cache to Fusion RAG Cache (2026)

> **Paper:** [arXiv:2601.12904](https://arxiv.org/abs/2601.12904)

> *"The challenge lies in how to reuse the precomputed KV cache of chunks while preserving generation quality. We propose FusionRAG, a novel inference framework that extends prefix-based KV cache reuse to RAG tasks."*

### 7. PCR: Prefetch-Enhanced Cache Reuse for RAG (2026)

> **Paper:** [arXiv:2603.23049](https://arxiv.org/abs/2603.23049) · **Authors:** Wenfeng Wang, Xiaofeng Hou, et al.

> *"PCR combines prefix-tree caching, layer-wise prefetching, and pipelined processing to maximize KV-cache reuse efficiency through intelligent prefetching."*

### 8. Cake: Compute Or Load KV Cache? Why Not Both? (2024)

> **Paper:** [arXiv:2410.03065](https://arxiv.org/abs/2410.03065) · **Authors:** UChicago/Jiang lab

> *"Cake is a novel KV cache loading system that optimally utilizes both computational and I/O resources in parallel... employs a bidirectional parallelized KV cache generation strategy."*

### 9. BatchLLM: Global Prefix Sharing (2024)

> **Paper:** [arXiv:2412.03594](https://arxiv.org/abs/2412.03594)

> *"BatchLLM achieves 85.2% token saving ratio... comparing to the 40.1% token saving ratio in vLLM + chunked-prefill + prefix-caching."*

### 10. HyperRAG: Reranker KV-Cache Reuse (2025)

> **Paper:** [arXiv:2504.02921](https://arxiv.org/abs/2504.02921)

> *"HyperRAG optimizes the trade-off between quality and efficiency in RAG pipelines by leveraging KV-cache reuse for efficient reranker inference."*

### 11. QCFuse: Query-Centric Cache Fusion (2026)

> **Paper:** [arXiv:2604.08585](https://arxiv.org/abs/2604.08585)

> *"QCFuse leverages semantic summary anchors to enhance query representations and selectively recomputes query-related tokens to improve accuracy."*

### 12. Security: Prompt Leakage via KV-Cache Sharing (2025)

> **Paper:** [NDSS 2025](https://www.ndss-symposium.org/ndss-paper/i-know-what-you-asked-prompt-leakage-via-kv-cache-sharing-in-multi-tenant-llm-serving/) · **Authors:** Guanlong Wu et al. (SUSTech, ByteDance)

> *"KV cache sharing may inadvertently create side channel information, which can be leveraged by the adversary to carefully craft requests... to determine if its requests match the other users', thereby recovering other users' prompts."*

---

## Production Systems

### LMCache (⭐8,022)

> *"The first open-source KV cache library designed to turbocharge LLM inference efficiency."*

| Feature | Description |
|---------|-------------|
| **CPU offloading** | KV cache to CPU memory |
| **Disk/S3 storage** | Persistent KV cache across instances |
| **Cross-process sharing** | Share KV cache between vLLM instances |
| **CacheGen integration** | 3× smaller than quantization |
| **Disaggregated P/D** | Transfer KV cache between prefill and decode nodes |
| **Multi-GPU P2P** | Direct GPU-to-GPU KV cache sharing |

Being proposed to **PyTorch Foundation** as a standard KV cache layer.

### vLLM Automatic Prefix Caching (APC)

> *"Caches the KV cache of existing queries, so that a new query can directly reuse the KV cache if it shares the same prefix."* — [vLLM Docs](https://docs.vllm.ai/en/latest/features/automatic_prefix_caching/)

- V1: prefix caching **enabled by default**, <1% throughput overhead
- Uses **block-level hashing** (tokens + prefix hash) with LRU eviction

### SGLang RadixAttention

> *"Retains the KV cache for both prompts and generation results in a radix tree... automatic KV cache reuse with RadixAttention."* — [SGLang Docs](https://lmsys.org/blog/2024-01-17-sglang/)

- Token-level radix tree (vs vLLM's block-level hashing)
- ~10% faster in multi-turn conversations with shared context
- Integrated into SGLang (⭐26,077)

---

## Systems Taxonomy

| System | Category | Key Mechanism | Speedup |
|--------|----------|---------------|---------|
| **RAGCache** | RAG-specific KV caching | Knowledge tree + PGDSF eviction + speculative pipelining | TTFT reduction |
| **CacheBlend** | Non-prefix KV fusion | Selective recomputation of small token fraction | 2.2–3.3× TTFT, 2.8–5× throughput |
| **CacheClip** | RAG KV cache reuse | Auxiliary-model-guided token selection | Selective recomputation |
| **CacheGen** | KV cache compression | Custom tensor encoder + adaptive compression | 3× compression, 3× TTFT improvement |
| **DroidSpeak** | Cross-LLM KV sharing | KV+E cache transfer between different models | 3.1× latency, 4× throughput |
| **FusionRAG** | RAG cache fusion | Chunk-level KV caching with quality-aware fusion | Prefix → RAG extension |
| **PCR** | Prefetch-enhanced RAG | Prefix-tree caching + layer-wise prefetching | Maximized reuse |
| **HyperRAG** | RAG quality-efficiency | KV-cache reuse for reranker inference | Quality + efficiency |
| **QCFuse** | Query-centric fusion | Semantic summary anchors + critical-layer attention | Selective recomputation |
| **Cake** | I/O-aware loading | Bidirectional parallelized KV generation | Compute + I/O overlap |
| **LMCache** | Production KV cache layer | Cross-process sharing, CPU/disk/S3 offloading | Production-grade |
| **vLLM APC** | Prefix caching | Block-level hashing with LRU eviction | <1% overhead |
| **SGLang RadixAttention** | Prefix caching | Radix tree for automatic KV reuse | ~10% in shared context |
| **BatchLLM** | Global prefix sharing | Explicit prefix ID + prefix-aware batching | 85.2% token saving |

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|------------------|
| **RAGCache** | 2024 | ACM | [arXiv:2404.12457](https://arxiv.org/abs/2404.12457) | Knowledge tree + PGDSF eviction for RAG |
| **CacheBlend** | 2024 | EuroSys 2025 | [arXiv:2405.16444](https://arxiv.org/abs/2405.16444) | Non-prefix KV cache fusion |
| **CacheGen** | 2023 | SIGCOMM 2024 | [arXiv:2310.07240](https://arxiv.org/abs/2310.07240) | KV cache compression + streaming |
| **DroidSpeak** | 2024 | NSDI 2026 | [arXiv:2411.02820](https://arxiv.org/abs/2411.02820) | Cross-LLM KV cache sharing |
| **CacheClip** | 2025 | arXiv | [arXiv:2510.10129](https://arxiv.org/abs/2510.10129) | Auxiliary-model-guided selective recomputation |
| **FusionRAG** | 2026 | arXiv | [arXiv:2601.12904](https://arxiv.org/abs/2601.12904) | Prefix → RAG cache extension |
| **PCR** | 2026 | arXiv | [arXiv:2603.23049](https://arxiv.org/abs/2603.23049) | Prefix-tree + prefetching + pipelining |
| **HyperRAG** | 2025 | arXiv | [arXiv:2504.02921](https://arxiv.org/abs/2504.02921) | Reranker KV-cache reuse |
| **QCFuse** | 2026 | arXiv | [arXiv:2604.08585](https://arxiv.org/abs/2604.08585) | Query-centric cache fusion |
| **Cake** | 2024 | arXiv | [arXiv:2410.03065](https://arxiv.org/abs/2410.03065) | Bidirectional parallelized KV loading |
| **BatchLLM** | 2024 | arXiv | [arXiv:2412.03594](https://arxiv.org/abs/2412.03594) | Global prefix sharing (85.2% token saving) |
| **Prompt Leakage (Security)** | 2025 | NDSS 2025 | [NDSS](https://www.ndss-symposium.org/ndss-paper/i-know-what-you-asked-prompt-leakage-via-kv-cache-sharing-in-multi-tenant-llm-serving/) | Side-channel attack via KV sharing |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **CacheGen: Store KV Cache on Disk/S3** | LMCache Blog | [Link](https://blog.lmcache.ai/en/2025/07/31/cachegen-store-your-kv-cache-on-disk-or-s3-load-blazingly-fast/) |
| **Sharing KV Cache Across Instances** | vLLM Production Stack | [Link](https://docs.vllm.ai/projects/production-stack/en/latest/use_cases/sharing-kv-cache.html) |
| **Prefix Caching Guide** | BentoML LLM Handbook | [Link](https://bentoml.com/llm/inference-optimization/prefix-caching/) |
| **Complete Guide to Inference Caching** | Machine Learning Mastery | [Link](https://machinelearningmastery.com/the-complete-guide-to-inference-caching-in-llms/) |
| **KV-Cache Wins You Can See** | llm-d Blog | [Link](https://llm-d.ai/blog/kvcache-wins-you-can-see) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core idea** | Cache and reuse KV cache states across requests to avoid redundant computation |
| **Prefix caching** | Same-prefix requests share KV cache (vLLM APC, SGLang RadixAttention) |
| **Non-prefix reuse** | CacheBlend fuses arbitrary KV chunks via selective recomputation |
| **RAG-specific** | RAGCache knowledge tree, CacheClip, FusionRAG — designed for document retrieval patterns |
| **Cross-LLM** | DroidSpeak shares KV cache between different models (3.1× latency reduction) |
| **Compression** | CacheGen compresses KV cache 3× smaller than quantization |
| **Production** | LMCache (⭐8,022) — being proposed to PyTorch Foundation |
| **Security risk** | KV cache sharing can leak prompts via side channels (NDSS 2025) |

---

*Last updated: 2026-04-20*