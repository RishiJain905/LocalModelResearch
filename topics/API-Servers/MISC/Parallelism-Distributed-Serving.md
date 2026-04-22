# Parallelism & Distributed Serving

## 📌 Overview

Distributed serving refers to the techniques used to run LLM inference across multiple GPUs and nodes — from tensor parallelism (sharding model weights across GPUs within a node) to pipeline parallelism (splitting layers across nodes) to disaggregated prefill/decode serving (separating the compute-intensive prefill phase from the memory-bandwidth-bound decode phase). Key innovations include **PagedAttention** (vLLM, SOSP '23) which reduces KV cache waste from 60–80% to near-zero, **DistServe** (OSDI '24) which achieves up to 7.4× goodput improvement via disaggregation, and **NVIDIA Dynamo** (GTC 2025) which claims up to 30× throughput improvement on Blackwell for DeepSeek-R1 models.

---

## Definitions & Standards

### PagedAttention

> "Inspired by the OS-based virtual memory systems, vLLM proposed PagedAttention to mitigate fragmentation by dynamically allocating memory for the KV cache."

> "vLLM achieves near-zero KV cache waste and flexible sharing."

— [vLLM Paper — arXiv:2309.06180](https://arxiv.org/pdf/2309.06180) (SOSP '23)

> "PagedAttention treats the KV cache like virtual memory in an operating system. Instead of one giant contiguous block per request, it breaks the..."

— [Plain English — PagedAttention vs Continuous Batching](https://python.plainenglish.io/pagedattention-vs-continuous-batching-vs-vllm-vs-sglang-a-practical-breakdown-4c19cc9e21c0)

### Continuous Batching (Iteration-Level Scheduling)

> "Requests that have finished earlier than other requests in a batch cannot return immediately to the client, while newly queued requests must wait to begin until the current batch completely finishes. The iteration batching we invented solves both of these problems by dynamically changing the requests that make up the batch while it is in progress."

— [FriendliAI](https://friendli.ai/blog/llm-iteration-batching) (originator via Orca paper, OSDI 2022)

### Disaggregated Serving

> "Disaggregated serving addresses this by splitting the inference pipeline into distinct stages such as prefill, decode, and routing, each running as an independent service that can be resourced and scaled on its own terms."

> "Prefill is compute-intensive and benefits from high floating point operations (FLOPS), while decode is memory-bandwidth-bound and benefits from large, fast memory."

— [NVIDIA Developer Blog, March 2026](https://developer.nvidia.com/blog/deploying-disaggregated-llm-inference-workloads-on-kubernetes/)

### Goodput

> "Goodput (DistServe): The maximum request rate at which a target percentage (e.g., P90) of requests meet both TTFT and TPOT/ITL SLOs simultaneously."

— [Optiver Tech Blog](https://optiversetech.com/blog/prefill-decode-disaggregation/)

### Tensor Parallelism

> "The tensor parallel size is the number of GPUs you want to use in each node, and the pipeline parallel size is the number of nodes you want to use."

— [vLLM Documentation](https://docs.vllm.ai/en/v0.8.2/serving/distributed_serving.html)

---

## Key Sources

### vLLM Paper (SOSP '23)

> "Our profiling results … show that only 20.4% - 38.2% of the KV cache memory is used to store the actual token states in the existing systems."

> "vLLM achieves 2–4× throughput improvement over state-of-the-art systems (FasterTransformer, Orca) at the same latency."

— [arXiv:2309.06180](https://arxiv.org/pdf/2309.06180)

### DistServe (OSDI '24)

> "First, colocation leads to strong prefill-decoding interference. A prefill step is much longer than a decoding step. When batched together, decoding steps are delayed by prefill steps (inflating TPOT), while prefill steps are slowed by decoding jobs (inflating TTFT)."

> "DistServe assigns prefill and decoding computation to different GPUs, hence eliminating prefill-decoding interferences."

> "Result: Up to **7.4× goodput improvement**, or same request rate served with **12.6× tighter SLO bounds**."

— [OSDI '24 — DistServe](https://www.usenix.org/system/files/osdi24-zhong-yinmin.pdf)

### Splitwise (ISCA 2024)

> "each phase of generative LLM inference has distinct latency, throughput, memory, and power characteristics"

> "Result: **1.4× higher throughput at 20% lower cost**, or **2.35× more throughput** under same power/cost budgets."

— [ISCA 2024 — Splitwise](https://www.computer.org/csdl/magazine/mi/2025/04/11024200/27kcMUnFh2o)

### NVIDIA Dynamo (GTC 2025)

> "The framework boosts the number of requests served by up to **30x**, when running the open-source DeepSeek-R1 models on NVIDIA Blackwell."

> "Compatible with PyTorch, SGLang, NVIDIA TensorRT-LLM, and vLLM"

— [NVIDIA Developer Blog](https://developer.nvidia.com/blog/nvidia-dynamo-a-new-paradigm-for-production-ai-inference/), [GitHub](https://github.com/ai-dynamo/dynamo)

### Cache-Aware Prefill-Decode Disaggregation (Together AI)

> "up to **40% higher sustainable throughput** and significantly lower time-to-first-token (TTFT) for long-context inference"

— [Together AI Blog](https://www.together.ai/blog/cache-aware-disaggregated-inference)

---

## Repositories & Tools

| Tool | URL | Key Features |
|------|-----|-------------|
| **vLLM** | [github.com/vllm-project/vllm](https://github.com/vllm-project/vllm) | PagedAttention, continuous batching, tensor/pipeline parallelism, chunked prefill, Ray backend |
| **NVIDIA Dynamo** | [github.com/ai-dynamo/dynamo](https://github.com/ai-dynamo/dynamo) | KV-aware Smart Router, KV Block Manager, NIXL transfer, Grove K8s operator |
| **llm-d** | [github.com/llm-d/llm-d](https://github.com/llm-d/llm-d) | CNCF Sandbox; disaggregated serving, tiered prefix cache, KServe/LWS integration |
| **SGLang** | [github.com/sgl-project/sglang](https://github.com/sgl-project/sglang) | RadixAttention for KV cache prefix sharing; 85–95% cache hit rate for few-shot vs vLLM's 15–25% |
| **LightLLM** | [github.com/ModelTC/lightllm](https://github.com/ModelTC/lightllm) | Python-based, lightweight, easy scalability |
| **TGI** | [github.com/huggingface/text-generation-inference](https://github.com/huggingface/text-generation-inference) | Now in maintenance mode; vLLM/SGLang recommended |
| **NIXL** | [docs.lmcache.ai](https://docs.lmcache.ai/disaggregated_prefill/nixl/index.html) | NVIDIA Inference Xfer Library for KV cache transfer; 30–50% faster than NCCL for typical message sizes |

---

## Debates & Controversies

### Continuous Batching Patents

> "If you encounter this type of batching in any other framework, it's probably iteration batching, regardless of its branding."

— [FriendliAI](https://friendli.ai/blog/llm-iteration-batching)

FriendliAI holds patents on iteration batching in the US and Korea. All major frameworks implement this technique — FriendliAI does not currently pursue enforcement.

### Disaggregation Overhead vs. Benefit

> "The honest summary: within a well-connected node, KV cache transfer is cheap enough to ignore. Across nodes with limited bandwidth, or at very high request rates, it is a real constraint that must be measured, not assumed away."

— [Optiver Tech Blog](https://optiversetech.com/blog/prefill-decode-disaggregation/)

**KV cache transfer sizing at 10 req/s (OPT-66B, 512 tokens):** ~90 Gbps required

### vLLM vs SGLang vs TensorRT-LLM

| Engine | Throughput (50 req, H100) | Cold Start | Best For |
|--------|---------------------|------------|----------|
| vLLM | 1,850 tok/s | ~62 sec | General use, model flexibility |
| TensorRT-LLM | 2,100 tok/s | ~28 min (compiled) | Max throughput, fixed model |
| SGLang | 1,920 tok/s | ~58 sec | Shared-prefix workloads |

> "TensorRT-LLM's 28-minute build is a deliberate tradeoff. It runs once per model version; reloading a compiled engine takes ~90 seconds."

— [Spheron Blog](https://www.spheron.network/blog/vllm-vs-tensorrt-llm-vs-sglang-benchmarks/)

### Chunked Prefill + Prefix Caching Limitation

> "In vLLM V1, when chunked prefill is enabled, only the first chunk of a long prompt benefits from prefix caching. After the initial chunk, subsequent chunks are treated as running requests."

— [vLLM Forums](https://discuss.vllm.ai/t/should-vllm-consider-prefix-caching-when-chunked-prefill-is-enabled/903)

---

## Summary Table

### Parallelism Strategies

| Strategy | Use Case | vLLM Config |
|----------|----------|-------------|
| Single GPU | Model fits in one GPU | Default |
| Tensor Parallel (TP) | Model fits in one node, not one GPU | `--tensor-parallel-size N` |
| Pipeline Parallel (PP) | Model too large for one node | `--pipeline-parallel-size N` |
| TP + PP Combined | Very large models across nodes | `--tensor-parallel-size N --pipeline-parallel-size M` |
| Data Parallel (DP) | Scale throughput, model fits | `--num-gpus` (replicated) |

### Key Performance Numbers

| Finding | Source | Value |
|---|---|---|
| vLLM throughput vs FasterTransformer/Orca | SOSP '23 | 2–4× higher |
| vLLM KV cache memory waste (before) | SOSP '23 | 20.4–38.2% used |
| DistServe goodput improvement | OSDI '24 | Up to 7.4× |
| Splitwise throughput improvement | ISCA '24 | 1.4× at 20% lower cost |
| CPD throughput improvement (long-context) | Together AI | Up to 40% |
| NVIDIA Dynamo (DeepSeek-R1 on GB200) | GTC 2025 | Up to 30× more requests |
| TensorRT-LLM vs vLLM (H100) | Spheron 2026 | 13% higher throughput |
| SGLang RadixAttention cache hit (few-shot) | SugiV Blog | 85–95% vs vLLM's 15–25% |

### Framework Feature Comparison

| Feature | vLLM | SGLang | TensorRT-LLM | TGI |
|---------|------|--------|--------------|-----|
| Continuous Batching | ✅ | ✅ | ✅ | ✅ |
| PagedAttention | ✅ | ✅ | ✅ | ✅ |
| Tensor Parallelism | ✅ | ✅ | ✅ | ✅ |
| Pipeline Parallelism | ✅ | ✅ | ✅ | ❌ |
| Disaggregated Serving | ✅ | ✅ | ✅ | Partial |
| RadixAttention / Prefix Cache | Block-based | Tree-based (superior) | Limited | ❌ |
| Structured Generation | Basic | xGrammar/FSM | Limited | ❌ |
| Multi-Modal | Limited | Extensive | ❌ | Limited |

---

## Related Topics

- [OpenAI-Compatible-API-Layers](./OpenAI-Compatible-API-Layers.md) — How these engines expose OpenAI-compatible endpoints
- [Streaming-Real-Time-Delivery](./Streaming-Real-Time-Delivery.md) — How streaming works on top of distributed serving
- [Structured-Outputs](./Structured-Outputs.md) — Grammar-constrained decoding in inference engines
