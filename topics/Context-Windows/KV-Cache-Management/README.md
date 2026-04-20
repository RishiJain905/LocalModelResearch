# KV-Cache Management & Architecture

> *"PagedAttention eliminates external fragmentation, minimizes internal fragmentation, and enables memory sharing for beam search and parallel sampling."* — Kwon et al., SOSP 2023

Research on KV cache management strategies for efficient LLM inference — covering PagedAttention/vLLM architecture evolution, KV cache reuse systems (RAGCache, CacheGen, etc.), and speculative decoding adapted for long-context scenarios.

---

## Topics

| Topic | Description | Key Papers |
|-------|-------------|------------|
| [**PagedAttention & vLLM Updates**](./PagedAttention-vLLM-Updates.md) | 2025–2026 vLLM V1 rewrite, MRV2, chunked prefill, prefix caching, disaggregated prefill, competing frameworks | arXiv:2309.06180, vLLM v0.8–v0.19 |
| [**KV-Cache Reuse (RAGCache)**](./KV-Cache-Reuse-RAGCache.md) | RAGCache, CacheGen, CacheBlend, DroidSpeak, LMCache, vLLM APC, SGLang RadixAttention | RAGCache (ACM), CacheGen (SIGCOMM 2024), CacheBlend (EuroSys 2025) |
| [**Speculative Decoding for Long Context**](./Speculative-Decoding-Long-Context.md) | EAGLE-3, Medusa, TriForce, LongSpec, MagicDec, P-EAGLE, 2025–2026 advances | EAGLE-3 (NeurIPS 2025), TriForce (COLM 2024), LongSpec (ACL 2025) |

---

## Quick Reference

### The KV Cache Problem

| Problem | Solution | Key System |
|---------|----------|------------|
| **Memory fragmentation** | PagedAttention (OS-style paging) | vLLM |
| **Prefix redundancy** | Block-level hash caching | vLLM APC |
| **RAG reuse** | Knowledge tree + selective recomputation | RAGCache, CacheBlend |
| **Long-context KV loading** | StreamingLLM-style sliding window | TriForce, LongSpec |
| **Cross-LLM sharing** | KV+E cache transfer | DroidSpeak (NSDI 2026) |

### Version History: vLLM

| Version | Key Change |
|---------|-----------|
| **v0.8–0.8.2** | V1 engine, chunked prefill, FP8 KV cache |
| **v0.9** | Blackwell support, PD disaggregation, EAGLE3 spec decoding |
| **v0.12** | V1 becomes default, chunked prefill always-on |
| **v0.15** | MRV2 VLM support |
| **v0.18** | MRV2 default, gRPC, FlexKV, GPU NGram spec decode |
| **v0.19** | Async scheduling default, MRV2 improvements |

### Key Repos Quick Reference

| Repo | ⭐ | Description |
|------|-----|------------|
| [vllm-project/vllm](https://github.com/vllm-project/vllm) | 77,363 | PagedAttention + vLLM serving engine |
| [LMCache/LMCache](https://github.com/LMCache/LMCache) | 8,022 | KV cache layer: CPU/disk/S3 offloading, CacheGen |
| [FasterDecoding/Medusa](https://github.com/FasterDecoding/Medusa) | 2,724 | Multi-head speculative decoding |
| [SafeAILab/EAGLE](https://github.com/SafeAILab/EAGLE) | 2,279 | EAGLE-1/2/3 (feature-level spec decoding) |
| [sgl-project/sglang](https://github.com/sgl-project/sglang) | 26,077 | RadixAttention + spec decoding |
| [sgl-project/SpecForge](https://github.com/sgl-project/SpecForge) | 789 | EAGLE-3 training framework |
| [Infini-AI-Lab/TriForce](https://github.com/Infini-AI-Lab/TriForce) | 279 | COLM 2024 — hierarchical long-context SD |
| [sail-sg/LongSpec](https://github.com/sail-sg/LongSpec) | 82 | ACL 2025 — lossless long-context SD |

---

*Last updated: 2026-04-20*