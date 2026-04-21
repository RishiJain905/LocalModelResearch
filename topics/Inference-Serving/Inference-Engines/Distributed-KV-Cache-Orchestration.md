# Distributed KV-Cache Orchestration

> **Papers:** [DistServe arXiv:2401.09670](https://arxiv.org/html/2401.09670v2) · [Mooncake arXiv:2407.00079](https://arxiv.org/pdf/2407.00079) · [LMCache Tech Report](https://lmcache.ai/tech_report.pdf)  
> **GitHub:** [llm-d/llm-d](https://github.com/llm-d/llm-d) · [LMCache/LMCache](https://github.com/LMCache/LMCache) · [kvcache-ai/Mooncake](https://github.com/kvcache-ai/Mooncake)  
> **Docs:** [llm-d.ai/docs/architecture](https://llm-d.ai/docs/architecture)

---

## Overview

> *"llm-d provides state-of-the-art orchestration above model servers to serve high-scale real world traffic efficiently and reliably."* — [llm-d.ai](https://llm-d.ai)

Distributed KV-Cache orchestration manages KV cache across multiple GPUs/nodes — enabling disaggregated prefill/decode serving, cross-request cache sharing, and tiered memory hierarchies.

---

## llm-d — Kubernetes-Native Distributed Inference

> **GitHub:** [llm-d/llm-d](https://github.com/llm-d/llm-d) (Apache 2.0, v0.5)  
> **Sub-projects:** [llm-d-kv-cache](https://github.com/llm-d/llm-d-kv-cache) · [llm-d-inference-scheduler](https://github.com/llm-d/llm-d-inference-scheduler)

### Architecture

llm-d adds four key components above model servers (vLLM/SGLang):

| Component | Role |
|-----------|------|
| **KV-Cache Indexer** | Global near-real-time view of KV-block locality across vLLM pod fleet |
| **Inference Scheduler** | P/D-awareness, KV-cache-awareness, SLA-awareness, load-awareness |
| **Disaggregated Serving Sidecar** | Orchestrates prefill/decode onto independent instances via NIXL for RDMA |
| **KV-Cache Offloading** | Pluggable hierarchy via vLLM's `KVConnector` abstraction |

### Key Capabilities

- KV-aware routing — routes requests to pods with relevant KV cache already present
- PD disaggregation — separates prefill and decode onto different GPU pools
- RDMA support — NIXL for InfiniBand/RoCE, TPU ICI, DCN cross-node KV transfer
- Offload targets — host memory, remote storage, LMCache, Mooncake, KVBM

---

## PD Disaggregation Systems

### DistServe (OSDI '24)

> **Paper:** [DistServe: Disaggregating Prefill and Decoding for Goodput-Optimized LLM Serving](https://arxiv.org/html/2401.09670v2)  
> **Repo:** [LLMServe/DistServe](https://github.com/LLMServe/DistServe)

> *"DistServe disaggregates prefill/decoding onto separate GPUs, eliminating P/D interference and enabling phase-specific resource optimization."*

| Metric | Improvement |
|--------|-------------|
| **SLO-compliant throughput** | Up to **4.48×** more requests |
| **Approach** | Phase-specific resource allocation |

### TetriInfer

> **Paper:** [TetriInfer](https://arxiv.org/html/2401.11181v1)

| Metric | Improvement |
|--------|-------------|
| **TTFT reduction** | **97%** |
| **JCT reduction** | **47%** |
| **Approach** | Chunked prefill + two-level scheduling |

---

## Production Systems

### Mooncake (Kimi/Moonshot AI)

> **Paper:** [Mooncake: A KVCache-centric Disaggregated Architecture for LLM Serving](https://arxiv.org/pdf/2407.00079)  
> **GitHub:** [kvcache-ai/Mooncake](https://github.com/kvcache-ai/Mooncake)

> *"Mooncake achieves up to 525% throughput increase over coupled baselines by using a distributed KVCache pool with CPU/DRAM/SSD and GPUDirect RDMA."*

| Metric | Value |
|--------|-------|
| **Throughput gain** | **525%** over coupled baselines |
| **KV Transfer** | GPUDirect RDMA for cross-machine |
| **Status** | Powers Kimi production traffic |

### LMCache — Enterprise KV Cache Layer

> **Tech Report:** [lmcache.ai/tech_report.pdf](https://lmcache.ai/tech_report.pdf)  
> **GitHub:** [LMCache/LMCache](https://github.com/LMCache/LMCache)

| Feature | Description |
|---------|-------------|
| **Context caching** | Prefix reuse across engines |
| **PD disaggregation** | Cross-engine KV transfer |
| **Throughput** | Up to **15×** improvement with vLLM |
| **Connectors** | Modular decoupling from inference engines |

### vLLM Disaggregated Prefill

> **Docs:** [docs.vllm.ai/en/latest/features/disagg_prefill/](https://docs.vllm.ai/en/latest/features/disagg_prefill/)

vLLM implements disaggregated prefill via two vLLM instances (prefill + decode) connected via **KV Connectors**:

| Connector | Purpose |
|-----------|---------|
| `LMCacheConnectorV1` | NIXL-based KV transmission |
| `NixlConnector` | Fully async send/recv |
| `P2pNcclConnector` | Peer-to-peer NCCL |
| `MooncakeConnector` | Production-oriented |
| `OffloadingConnector` | CPU memory offloading |

---

## Cloud Provider Solutions

### Amazon SageMaker HyperPod — Managed Tiered KV Cache

> **Blog:** [AWS Blog: Managed Tiered KV Cache](https://aws.amazon.com/blogs/machine-learning/managed-tiered-kv-cache-and-intelligent-routing-for-amazon-sagemaker-hyperpod/)

| Feature | Description |
|---------|-------------|
| **L1 cache** | Local CPU memory |
| **L2 cache** | Distributed cluster (tieredstorage/Redis) |
| **Routing** | `prefixaware`, `kv-aware`, `round-robin` |
| **Result** | **40% TTFT reduction**, **25% compute cost reduction** |

### Crusoe MemoryAlloy

> **Blog:** [crusoe.ai/blog/memoryalloy](https://www.crusoe.ai/resources/blog/crusoe-memoryalloy-reinventing-kv-caching-for-cluster-scale-inference)

| Metric | Improvement |
|--------|-------------|
| **TTFT** | **9.9× faster** |
| **Throughput** | **5× higher** |
| **KV storage** | Nearly **1 TB** GPU-resident via RDMA sharing |

---

## Research Systems

### FlexKV (Tencent + NVIDIA)

> **GitHub:** [taco-project/FlexKV](https://github.com/taco-project/FlexKV)

Scalable distributed runtime for KV cache offloading — memory-disaggregated KV store with index proxying and two-level CN memory optimization. Integrates with NVIDIA Dynamo.

### KVDirect

> **Paper:** [arXiv:2501.14743](https://arxiv.org/abs/2501.14743)

Optimizes KV cache transfer for distributed disaggregated LLM inference.

### FlowKV

> **Paper:** [arXiv:2404.03775v1](https://arxiv.org/html/2404.03775v1)

| Metric | Improvement |
|--------|-------------|
| **KV transfer latency** | **96% reduction** (0.944s → 0.053s) |
| **Approach** | Load-aware scheduling for balanced P/D allocation |

### DéjàVu (ICML '24)

> **Paper:** [Proceedings of ICML 2024](https://proceedings.mlr.press/v235/strati24a.html)

KV-cache streaming for fast, fault-tolerant LLM serving — addresses pipeline bubbles, GPU memory overprovisioning, and long recovery times via microbatch swapping and state replication.

---

## Systems Summary Table

| System | Type | Key Contribution |
|--------|------|-----------------|
| **llm-d** | Framework | Kubernetes-native distributed inference with KV-aware routing, PD disaggregation |
| **LMCache** | Library | Enterprise KV caching layer for vLLM/SGLang; 15× throughput gain |
| **DistServe** | Research | P/D disaggregation; 4.48× SLO-compliant throughput |
| **TetriInfer** | Research | 97% TTFT reduction via chunked prefill + disaggregation |
| **Mooncake** | Production | KVCache-centric architecture; 525% throughput; powers Kimi |
| **KVDirect** | Research | Optimized distributed KV transfer |
| **FlowKV** | Research | 96% latency reduction in KV transfer |
| **DéjàVu** | Research | Fault-tolerant streaming; pipeline bubble reduction |
| **vLLM Disagg** | Feature | Native PD disaggregation via KV Connectors |
| **SageMaker HyperPod** | Cloud | Managed tiered KV cache + intelligent routing |
| **Crusoe MemoryAlloy** | Cloud | Cluster-wide KV caching; 9.9× TTFT improvement |
| **FlexKV** | Research | Memory-disaggregated KV store from Tencent/NVIDIA |

---

## GitHub Repos

| Repo | Description |
|------|-------------|
| [llm-d/llm-d](https://github.com/llm-d/llm-d) | Kubernetes-native distributed inference orchestration |
| [llm-d/llm-d-kv-cache](https://github.com/llm-d/llm-d-kv-cache) | KV-cache indexer and offloading components |
| [llm-d/llm-d-inference-scheduler](https://github.com/llm-d/llm-d-inference-scheduler) | P/D/SLA-aware request scheduling |
| [LMCache/LMCache](https://github.com/LMCache/LMCache) | Enterprise KV caching layer; 15× throughput |
| [LLMServe/DistServe](https://github.com/LLMServe/DistServe) | PD disaggregation research system |
| [kvcache-ai/Mooncake](https://github.com/kvcache-ai/Mooncake) | Production KVCache-centric architecture; powers Kimi |
| [taco-project/FlexKV](https://github.com/taco-project/FlexKV) | Memory-disaggregated KV store |
| [jjiantong/Awesome-KV-Cache-Optimization](https://github.com/jjiantong/Awesome-KV-Cache-Optimization) | Comprehensive KV cache optimization survey |

---

*Last updated: 2026-04-21*
