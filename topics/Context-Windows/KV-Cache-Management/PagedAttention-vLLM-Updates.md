# PagedAttention & vLLM 2025–2026 Updates

> **Original Paper:** [arXiv:2309.06180](https://arxiv.org/abs/2309.06180) (SOSP 2023) · **GitHub:** ⭐77,363 [vllm-project/vllm](https://github.com/vllm-project/vllm)
> **Blog:** [vLLM V1 Alpha (Red Hat, Jan 2025)](https://developers.redhat.com/articles/2025/01/28/vllm-v1-a-major-upgrade-vllms-core-architecture) · [MRV2 (vLLM Blog, Mar 2026)](https://vllm.ai/blog/tags/engineering) · [PagedAttention Explainer (Red Hat, Jul 2025)](https://developers.redhat.com/articles/2025/07/24/how-pagedattention-resolves-memory-waste-llm-systems)
> **Docs:** [Automatic Prefix Caching](https://docs.vllm.ai/en/stable/design/prefix_caching/) · [Disaggregated Prefill](https://docs.vllm.ai/en/latest/features/disagg_prefill/)

---

## Overview

> *"PagedAttention eliminates external fragmentation, minimizes internal fragmentation, and enables memory sharing for beam search and parallel sampling."* — Kwon et al., SOSP 2023

vLLM has undergone a massive architectural rewrite since mid-2025. The V1 engine (alpha Jan 2025 → default by v0.12) and Model Runner V2 (v0.18+, Mar 2026) represent ground-up re-implementations that improve throughput up to 1.7×. This doc covers the **2025–2026 evolution** of PagedAttention and vLLM's KV cache management.

---

## V1 Architecture Rewrite

> *"V1 features isolated EngineCore loop, simplified scheduler, zero-overhead prefix caching, clean TP architecture, persistent Batch input preparation, piecewise CUDA graphs, enhanced multimodal, FlashAttention 3."* — [vLLM V1 Alpha Blog](https://developers.redhat.com/articles/2025/01/28/vllm-v1-a-major-upgrade-vllms-core-architecture)

| Component | V0 (Legacy) | V1 (Current) |
|-----------|-------------|--------------|
| **Scheduler** | Complex, multi-step | Simplified, single-step |
| **KV Cache Manager** | PagedAttention v0 | Zero-overhead prefix caching |
| **Input Preparation** | Per-step overhead | Persistent Batch (input diffing) |
| **CUDA Graphs** | Full-graph only | Piecewise CUDA graphs |
| **Chunked Prefill** | Configurable, off by default | **Always enabled by design** |
| **Prefix Caching** | Opt-in, overhead at 0% hit rate | **Enabled by default**, <1% overhead at 0% hit rate |
| **Throughput** | Baseline | **Up to 1.7× higher** |

**Key V1 performance:** [24% throughput improvement](https://developers.redhat.com/articles/2025/04/28/performance-boosts-vllm-081-switching-v1-engine) for generation-heavy workloads.

---

## Model Runner V2 (MRV2) — March 2026

> *"A modular and faster core for vLLM — ground-up re-implementation."* — [vLLM Blog, Mar 2026](https://vllm.ai/blog/tags/engineering)

| Metric | Improvement |
|--------|-------------|
| **Throughput (Qwen3-0.6B on GB200)** | **56.2%** improvement |
| **TPOT (GLM-4.7-FP8)** | **6.3%** improvement |
| **Code reduction** | V1 `gpu_model_runner.py` >6,700 lines → MRV2 largest file <1,300 lines |

**Key changes:**
- GPU-native bookkeeping
- Async-first design
- Triton-native sampler
- Modular `ModelState` abstraction
- Requires v0.17+, `VLLM_USE_V2_MODEL_RUNNER=1`

---

## Version Progression (v0.8 → v0.19.1)

| Version | Date | Key Features |
|---------|------|--------------|
| **v0.8.0** | Mar 2025 | V1 engine improvements, structured output backends |
| **v0.8.2** | Mar 2025 | Critical V1 memory bug fix, FP8 KV cache support |
| **v0.9.0** | — | NVIDIA Blackwell support, PD disaggregation (NIXL), full CUDA graph in V1, hybrid memory allocator, EAGLE3 spec decoding |
| **v0.12.0** | — | PyTorch 2.9.0, xformers deprecated, DeepSeek v3.2/SparseMLA, FlashAttention ViT default |
| **v0.15.0** | — | MRV2 VLM support, Whisper torch.compile, FlashInfer v0.6.1 |
| **v0.18.0** | Late Mar 2026 | gRPC serving, GPU NGram spec decode, smarter KV cache offloading, FlexKV |
| **v0.19.0** | Apr 2026 | Gemma 4 Day-0 support, async scheduling default, MRV2 improvements |
| **v0.19.1** | — | Transformers v5, Gemma4 quantized MoE |

---

## Chunked Prefill & Prefix Caching

> *"Caches the KV cache of existing queries, so that a new query can directly reuse the KV cache if it shares the same prefix."* — [vLLM APC Docs](https://docs.vllm.ai/en/stable/design/prefix_caching/)

**V1 changes:**
- Chunked prefill is **always enabled by design** (cannot be disabled)
- Prefix caching is **enabled by default** with <1% throughput overhead at 0% hit rate
- V1 scheduler prioritizes decode, inserts prefills into remaining token budget dynamically
- Uses **block-level hashing** (tokens + prefix hash) for KV cache reuse

---

## Disaggregated Prefill & KV Cache Transfer

> [RFC #10818](https://github.com/vllm-project/vllm/issues/10818) · [NixlConnector Roadmap #33702](https://github.com/vllm-project/vllm/issues/33702)

| Connector | Status | Description |
|-----------|--------|-------------|
| **ExampleConnector** | Stable | Reference implementation |
| **LMCacheConnectorV1** | Available | Via NIXL, S3-backed KV stores |
| **NixlConnector** | In progress | Fully async PD disaggregation |
| **MooncakeConnector** | Available | KV cache register + PD disaggregation |

---

## Distributed KV Cache Routing (llm-d)

> [llm-d Blog: KV-Cache Wins You Can See](https://llm-d.ai/blog/kvcache-wins-you-can-see) (Sep 2025)

llm-d provides **precise prefix-cache aware scheduling** across distributed pods:
- KVEvents stream from vLLM pods → KV-Block Index → cache affinity score + load-aware scoring
- **57× faster response times** and **2× throughput** on identical hardware
- Anthropic pricing reference: cached tokens $0.30/M vs uncached $3.00/M (10× cost difference)

---

## Competing Frameworks (2026)

From [LeetLLM: vLLM vs SGLang vs TensorRT-LLM vs Ollama](https://leetllm.com/blog/llm-inference-engine-comparison-2026):

| Framework | H100 Llama 3.3 70B Throughput | Cold Start | Best For |
|-----------|-------------------------------|------------|----------|
| **vLLM** | 2,400 tok/s @ 100 conc | ~62s | Safest broad-default |
| **SGLang** | 2,460 tok/s @ 100 conc | ~58s | Prefix-heavy workloads |
| **TensorRT-LLM** | 2,780 tok/s @ 100 conc | ~28 min (compiled) | Peak NVIDIA throughput |
| **Ollama** | — | Fast | Local dev |

- SGLang uses **RadixAttention** (token-level radix tree) vs vLLM's **block-level hashing**
- SGLang ~10% faster in multi-turn conversations with shared context
- vAttention (arXiv:2405.04437): alternative to PagedAttention using CUDA virtual memory APIs for contiguous KV cache layout

---

## Speculative Decoding in vLLM

| Method | Integration | Speedup |
|--------|------------|---------|
| **EAGLE-3** | v0.9+ | Up to 5.6× |
| **P-EAGLE** | v0.18+ | Parallel (non-autoregressive) drafting |
| **GPU NGram** | v0.18+ | N-gram based speculative decoding |

> [P-EAGLE (AWS Blog, Mar 2026)](https://aws.amazon.com/blogs/machine-learning/p-eagle-faster-llm-inference-with-parallel-speculative-decoding-in-vllm/): Removes autoregressive bottleneck in draft model.

> [EAGLE-3 Spec Decoding on vLLM (Red Hat, Apr 2026)](https://developers.redhat.com/articles/2026/04/16/performance-improvements-speculative-decoding-vllm-gpt-oss): 20.5% throughput improvement, 15.9% request latency reduction on gpt-oss-120B MoE.

---

## LMCache & KV Cache Offloading

| System | Backend | Key Feature |
|--------|---------|-------------|
| **LMCache** (⭐8,022) | CPU/disk/S3 | Cross-process sharing, CacheGen compression, being proposed to PyTorch Foundation |
| **Ceph + LMCache** | Ceph RGW | Content-addressable KV storage with Ceph backing |
| **NetApp ONTAP S3** | ONTAP S3 | LMCacheConnectorV1 for shared S3-backed KV stores |
| **llm-d** | Any FS | Native KV cache offloading to any filesystem |

> [LMCache Blog — CacheGen](https://blog.lmcache.ai/en/2025/07/31/cachegen-store-your-kv-cache-on-disk-or-s3-load-blazingly-fast/): *"CacheGen compresses your KV cache up to 3× smaller than quantization... cuts TTFT by up to 3×"*

---

## vLLM Roadmap

| Quarter | Key Priorities |
|---------|----------------|
| **Q1 2025** ([#11862](https://github.com/vllm-project/vllm/issues/11862)) | P0: hybrid memory allocator; P1: structured decoding, multimodal, LoRA |
| **Q2 2025** ([#15735](https://github.com/vllm-project/vllm/issues/15735)) | Path to v1.0.0, cluster scale serving, production hardening |
| **Q2 2026** ([#39749](https://github.com/vllm-project/vllm/issues/39749)) | SIGs: Core, Large Scale Serving, Model Performance, Quantization, Spec Decoding, Torch Compile, RL, MultiModality, CI |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **V1 engine** | Up to 1.7× throughput, chunked prefill always-on, prefix caching default |
| **MRV2** | 56% throughput improvement (small models), 80% code reduction |
| **Prefix caching** | <1% overhead at 0% hit, "several times" improvement at high hit rates |
| **PD disaggregation** | Connectors for NIXL, Mooncake, LMCache |
| **Spec decoding** | EAGLE-3 (5.6×), P-EAGLE (parallel), GPU NGram |
| **Competition** | SGLang RadixAttention ~10% faster for prefix-heavy; TensorRT-LLM peak throughput |

---

*Last updated: 2026-04-20*