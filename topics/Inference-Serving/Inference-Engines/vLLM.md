# vLLM

> **GitHub:** [vllm-project/vllm](https://github.com/vllm-project/vllm) — 32.6k stars  
> **Papers:** [PagedAttention arXiv:2309.06180](https://arxiv.org/abs/2309.06180) (SOSP 2023) · [SARATHI arXiv:2308.16369](https://arxiv.org/abs/2308.16369) · [Speculative Decoding arXiv:2302.01318](https://arxiv.org/pdf/2302.01318)  
> **Docs:** [docs.vllm.ai](https://docs.vllm.ai/) · [vLLM Blog](https://blog.vllm.ai/)

---

## Overview

> *"vLLM is an open-source library for LLM inference and serving. It provides state-of-the-art throughput and latency with PagedAttention, continuous batching, and other optimizations."* — [vLLM Blog](https://blog.vllm.ai/)

vLLM is the dominant high-throughput inference serving engine, using **PagedAttention** to manage KV cache in non-contiguous paged blocks — eliminating memory fragmentation and enabling 35× higher throughput than naive serving.

---

## Core Architecture

### PagedAttention

> *"Separating logical and physical KV blocks allows vLLM to dynamically grow the KV cache memory without reserving it for all positions in advance, which eliminates most memory waste in existing systems."* — [PagedAttention Paper](https://arxiv.org/abs/2309.06180)

- **Inspired by:** OS virtual memory paging
- **KV cache:** Stored as `[num_blocks, num_kv_heads, head_size/x, block_size, x]` for keys
- **Default block size:** 16 tokens per block
- **Memory savings:** 19-27% reduction in KV cache memory vs contiguous allocation

### Continuous Batching

- As soon as one sequence finishes generation, vLLM slots in a new request
- No idle GPU cycles waiting for batch boundaries
- **Also called:** Iteration-level scheduling

### Chunked Prefill (SARATHI)

> *"Chunked prefill splits prefill requests into equal-sized chunks to overlap compute-bound prefill with memory-bound decode."* — [SARATHI Paper](https://arxiv.org/abs/2308.16369)

- Breaks long prefill into chunks to prevent blocking decode
- Improves TBT (Time Between Tokens) stability for interactive apps

---

## vLLM v1 Architecture (Major Re-architecture)

> **Announced:** January 2025 — [Blog](https://blog.vllm.ai/2025/01/27/v1-alpha-release.html)

| Change | Description |
|--------|-------------|
| **Scheduler redesign** | Removes prefill/decode distinction; unified `{request_id: num_tokens}` dict |
| **Zero-overhead prefix caching** | Hash-based with LRU; <1% throughput degradation at 0% cache hit |
| **Persistent Batch** | Caches input tensors, applies only diffs at each step |
| **torch.compile** | Automatic optimization with custom op fusion |
| **FlashAttention 3** | Handles high dynamism of combined prefill+decode batches |
| **Clean TP architecture** | Caches request states on workers; transmits only incremental diffs |

```bash
pip install vllm --upgrade
export VLLM_USE_V1=1
# Zero API changes required
```

---

## Key Features

### Automatic Prefix Caching

- Hash-based: each KV cache block hashed by tokens in block + prefix tokens before block
- Default hash: `sha256` (v0.11+); alternatives: `xxhash`, `sha256_cbor`
- v1: Near-zero overhead, enabled by default

### FP8 Quantization

> *"FP8 can provide ~2× memory reduction and up to 1.6× throughput improvement."* — [vLLM FP8 Docs](https://docs.vllm.ai/en/stable/features/quantization/fp8/)

| Format | Hardware | Notes |
|--------|----------|-------|
| E4M3 | H100, MI300x | ±448 range |
| E5M2 | H100, MI300x | ±57,344 range |
| **Dynamic** | No calibration data needed | |

### Speculative Decoding

> *"Speculative decoding is theoretically lossless up to the precision limits of hardware numerics."* — [vLLM Docs](https://docs.vllm.ai/en/latest/features/speculative_decoding/)

| Method | Low QPS | High QPS | Notes |
|--------|---------|----------|-------|
| **EAGLE** | High gain | Medium-high | Strong general-purpose |
| **MTP** | High gain | Medium-high | Best with native MTP support |
| Draft model | High gain | Medium | Needs separate draft model |
| **PARD** | High gain | Medium-high | Low draft latency |
| MLP | Medium-high | Medium | Good for compatible MLP |
| N-gram | Low-medium | Medium | Lightweight, easy |
| Suffix decoding | Low-medium | Medium | No extra model |

### Disaggregated Prefill

> *"Does NOT improve throughput — improves TTFT/ITL control."* — [vLLM Docs](https://docs.vllm.ai/en/latest/features/disagg_prefill/)

- Runs 2 vLLM instances (prefill + decode) connected via KV Connectors
- Supported: LMCacheConnectorV1, NixlConnector, P2pNcclConnector, MooncakeConnector

### Distributed Inference

```bash
# 8 GPUs across 2 nodes
vllm serve /path/to/model \
  --tensor-parallel-size 8 \
  --pipeline-parallel-size 2 \
  --nnodes 2 --node-rank 0 \
  --master-addr <head_ip>
```

---

## Performance Benchmarks

### High Concurrency (GPT-OSS-120B, 2× H100)

> **Source:** [Clarifai Blog](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b)

| Concurrency | vLLM Throughput | SGLang Throughput | TRT-LLM Throughput |
|-------------|-----------------|-------------------|-------------------|
| 1 | 187 tok/s | 231 tok/s | **243 tok/s** |
| 10 | 863 tok/s | **988 tok/s** | 867 tok/s |
| 50 | 2,212 tok/s | **3,109 tok/s** | 2,163 tok/s |
| **100** | **4,742 tok/s** | 3,222 tok/s | 1,943 tok/s |

| Concurrency | vLLM TTFT | SGLang TTFT | TRT-LLM TTFT |
|-------------|-----------|-------------|--------------|
| 1 | **0.053s** | 0.125s | 0.177s |
| 10 | 1.91s | **1.16s** | 2.50s |
| 100 | **1.87s** | 8.99s | 5.47s |

**Key takeaway:** vLLM dominates at **extreme concurrency (100 requests)** — fastest TTFT, highest throughput (4,742 tok/s).

---

## Papers Table

| Paper | Year | Venue | URL | Key Contribution |
|-------|------|-------|-----|-----------------|
| **PagedAttention** | 2023 | SOSP | [arXiv](https://arxiv.org/abs/2309.06180) | Non-contiguous KV cache paging, 24× throughput gain |
| **SARATHI** | 2023 | arXiv | [arXiv](https://arxiv.org/abs/2308.16369) | Chunked prefill, stall-free batching |
| **Speculative Decoding** | 2023 | arXiv | [arXiv](https://arxiv.org/pdf/2302.01318) | Lossless decoding acceleration |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [vllm-project/vllm](https://github.com/vllm-project/vllm) | Main repo — 32.6k stars |
| [vllm-project/vllm/releases](https://github.com/vllm-project/vllm/releases) | v0.19.1 (latest), Gemma4, transformers v5 |

---

## Blog Posts & Technical Reports

| Title | Source | URL |
|-------|--------|-----|
| **vLLM v1 Alpha Release** | vLLM Blog | [Link](https://blog.vllm.ai/2025/01/27/v1-alpha-release.html) |
| **vLLM 2024 Wrapped / 2025 Vision** | vLLM Blog | [Link](https://vllm.ai/blog/vllm-2024-wrapped-2025-vision) |
| **Comparing SGLang, vLLM, TensorRT-LLM** | Clarifai Blog | [Link](https://www.clarifai.com/blog/comparing-sglang-vllm-and-tensorrt-llm-with-gpt-oss-120b) |
| **vLLM v1: A Major Upgrade** | Red Hat Developer | [Link](https://developers.redhat.com/articles/2025/01/28/vllm-v1-a-major-upgrade-vllms-core-architecture) |
| **Distributed Inference with vLLM** | Red Hat Developer | [Link](https://developers.redhat.com/articles/2025/02/06/distributed-inference-with-vllm) |
| **vLLM or llama.cpp: Choosing the Right Engine** | Red Hat Developer | [Link](https://developers.redhat.com/articles/2025/09/30/vllm-or-llamacpp-choosing-right-llm-inference-engine-your-use-case) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Best for** | High-concurrency serving, data center, API workloads |
| **Key innovation** | PagedAttention — OS-style KV cache paging |
| **Strength** | Scales best at 50-100+ concurrent requests |
| **Quantization** | FP8, INT8, AWQ, GPTQ |
| **Multi-GPU** | Tensor parallelism + pipeline parallelism + multi-node via Ray |

---

*Last updated: 2026-04-21*
