# llm-d Cluster Setups

## 📌 Overview

**llm-d** is a Kubernetes-native distributed inference framework — a CNCF Sandbox project (donated by Red Hat and IBM in 2025) that provides production-grade orchestration for multi-node, multi-GPU LLM serving. It builds on vLLM as the inference engine, LeaderWorkerSet (LWS) for multi-host pod management, and KServe for enterprise control plane features. The project is backed by Red Hat, IBM, Google, NVIDIA, CoreWeave, AMD, Hugging Face, Intel, Lambda, and Mistral AI. Key capabilities include prefill-decode (PD) disaggregation, KV-cache-aware routing, prefix caching, and Mixture-of-Experts (MoE) support.

---

## Definitions & Standards

### llm-d (CNCF Sandbox)

> "llm-d is a high-performance distributed inference serving stack optimized for production deployments on Kubernetes. We help you achieve the fastest 'time to SOTA performance' for open-source large language models across most hardware accelerators and infrastructure providers."

— [llm-d.ai](https://llm-d.ai/)

> "llm-d is organized around three core areas: disaggregated inference, smarter scheduling with KV cache and prefix cache-aware routing, and Mixture of Experts (MoE)."

— [Red Hat Developer](https://developers.redhat.com/articles/2025/11/21/introduction-distributed-inference-llm-d), November 2025

### LeaderWorkerSet (LWS)

> "LeaderWorkerSet: An API for deploying a group of pods as a unit of replication. It aims to address common deployment patterns of AI/ML inference workloads, especially multi-host inference workloads where the LLM will be sharded and run across multiple devices on multiple nodes."

— [github.com/kubernetes-sigs/lws](https://github.com/kubernetes-sigs/lws)

### Prefill-Decode (PD) Disaggregation

> "**Prefill**: Processes the entire input prompt in parallel, computing attention states and storing key and value tensors in the KV Cache. Involves extensive matrix multiplication, making it compute-bound."

> "**Decode**: Auto-regressively generates output tokens one-by-one. Each new token requires loading massive model weights from HBM, making it memory-bound and serial in nature."

— [rudeigerc.dev](https://rudeigerc.dev/posts/kubernetes-based-llm-inference-architectures-an-overview/)

> "llm-d separates these phases across different GPU groups, assigning compute-optimized resources to prefill and latency-optimized resources to decode. This 'phase-aware architecture increases GPU utilization, reduces tail latency, and lowers cost per token by eliminating resource contention.'"

— [KServe Blog](https://kserve.github.io/website/blog/cloud-native-ai-inference-kserve-llm-d)

### NVIDIA Dynamo

> "NVIDIA Dynamo is a high-throughput, low-latency open-source inference serving framework for deploying generative AI and reasoning models in large-scale distributed environments."

— [NVIDIA Developer Blog](https://developer.nvidia.com/blog/introducing-nvidia-dynamo-a-low-latency-distributed-inference-framework-for-scaling-reasoning-ai-models/)

---

## Key Sources

### llm-d Official Site & Repository

**URL:** [llm-d.ai](https://llm-d.ai/) | [github.com/llm-d/llm-d](https://github.com/llm-d/llm-d)

> "llm-d bridges the gap between traditional distributed systems and the emerging AI inference stack, making large-scale model serving a first-class, cloud-native workload."

— [IBM Research Blog](https://research.ibm.com/blog/donating-llm-d-to-the-cloud-native-computing-foundation)

**Founding contributors:** Red Hat, IBM, Google Cloud, NVIDIA, CoreWeave
**Supporting contributors:** AMD, Cisco, Hugging Face, Intel, Lambda, Mistral AI

### CNCF Donation Announcement

> "When we launched llm-d in May 2025, we set out to solve the massive capabilities gap between AI experimentation and mission-critical production inference at scale."

— [Red Hat Blog](https://www.redhat.com/en/blog/why-were-contributing-llm-d-cncf-standardizing-future-ai)

> "The marriage of Kubernetes and AI has arrived in llm-d, a replicable Kubernetes blueprint to deploy inference stacks for any model, on any accelerator, in any cloud."

— [The New Stack](https://thenewstack.io/llm-d-cncfc-kubernetes-inference/)

### NVIDIA Dynamo (GTC 2025)

> "NVIDIA Dynamo is a high-throughput, low-latency open-source inference serving framework for deploying generative AI and reasoning models in large-scale distributed environments."

— [NVIDIA Technical Blog](https://developer.nvidia.com/blog/introducing-nvidia-dynamo-a-low-latency-distributed-inference-framework-for-scaling-reasoning-ai-models/)

**DynamoLLM paper (HPCA 2025):** 52% energy savings, 38% carbon reduction, 61% cost reduction on Azure GPUs.

### KServe Integration

> "If KServe is the control plane for models, llm-d is the distributed intelligence scheduling layer."

— [KServe Blog](https://kserve.github.io/website/blog/cloud-native-ai-inference-kserve-llm-d)

> "Built on the foundation of llm-d—a production-ready framework for scalable LLM serving—LLMInferenceService delivers enterprise-grade capabilities for deploying and managing Large Language Model inference workloads on Kubernetes."

— [KServe LLMInferenceService Overview](https://kserve.github.io/website/docs/model-serving/generative-inference/llmisvc/llmisvc-overview)

### Academic — Dynamic Partitioning (UC Berkeley)

> "We propose a dynamic partitioning strategy for distributed LLM inference which dynamically switches between different partitioning strategies at inference time, optimizing for both GPU characteristics and input length."

— [UC Berkeley EECS-2024-108](https://www2.eecs.berkeley.edu/Pubs/TechRpts/2024/EECS-2024-108.html)

**Result:** 40% reduction in TTFT, 18% latency reduction on Llama 2 7B (L4/A100).

### Parallelism Strategies

| Strategy | Mechanism | Trade-off |
|----------|-----------|-----------|
| **Data Parallelism** | Replicates model weights across GPUs; divides input batch | Faster but duplicates entire model |
| **Tensor Parallelism** | Slices individual layers into parallel blocks | Enables larger models but communication overhead |
| **Pipeline Parallelism** | Divides layers across sequential devices | Better GPU utilization but can increase latency |
| **Expert Parallelism** | Splits experts across devices (MoE only) | Reduces memory but introduces all-to-all communication |
| **Hybrid Parallelism** | Combines TP + DP + PP | Balances trade-offs but increases complexity |

— [BentoML — Data/Tensor/Pipeline/Expert/Hybrid Parallelism](https://bentoml.com/llm/inference-optimization/data-tensor-pipeline-expert-hybrid-parallelism)

---

## Repositories & Tools

### Primary Frameworks

| Project | Repository | Description | Backer |
|---------|------------|-------------|--------|
| **llm-d** | [github.com/llm-d/llm-d](https://github.com/llm-d/llm-d) | Kubernetes-native distributed inference | Red Hat, IBM, CNCF Sandbox |
| **LeaderWorkerSet (LWS)** | [github.com/kubernetes-sigs/lws](https://github.com/kubernetes-sigs/lws) | K8s API for multi-host pod replication | Kubernetes SIGs |
| **NVIDIA Dynamo** | [github.com/ai-dynamo/dynamo](https://github.com/ai-dynamo/dynamo) | Distributed inference for reasoning models | NVIDIA |
| **AIBrix** | [github.com/vllm-project/aibrix](https://github.com/vllm-project/aibrix) | vLLM serving on Kubernetes | ByteDance |
| **OME** | [github.com/sgl-project/ome](https://github.com/sgl-project/ome) | SGLang-based serving stack | SGLang/Oracle |

### Supporting Tools

| Tool | Purpose |
|------|---------|
| **NIXL** | GPU memory transfer optimization for disaggregated serving |
| **KEDA** | Event-driven autoscaling for Kubernetes |
| **Karpenter** | Node-level autoscaling for Kubernetes |
| **Ray / KubeRay** | Distributed execution for vLLM multi-node |

---

## Debates & Controversies

### PD Disaggregation: Not a Silver Bullet

> "PD disaggregation is the future of large-scale inference, but not a silver bullet: Separating compute-heavy prefill from memory-heavy decode improves resource utilization but adds KV Cache transmission overhead and orchestration complexity."

— [rudeigerc.dev](https://rudeigerc.dev/posts/kubernetes-based-llm-inference-architectures-an-overview/)

> "Not All Prefills Are Equal: PPD Disaggregation for Multi-turn LLM Serving" — not all prefill workloads benefit equally from disaggregation.

— [arXiv:2603.13358](https://arxiv.org/abs/2603.13358)

### MoE Communication Overhead

> "more than half of MoE inference time is communication, not compute. meta, perplexity, deepseek, and cornell built 4 completely different systems and converged on the same bottleneck."

— [LinkedIn / Emi Anderegg](https://www.linkedin.com/posts/emi-andere_more-than-half-of-moe-inference-time-is-communication-activity-7435496885777416192-z9Ql)

### vLLM Multi-Node Kubernetes Limitations

> "I've tried scaling up vLLM using replicas/horizontal pod autoscaling. However the performance seems to be worse than a single node."

— [vLLM GitHub Issue #17645](https://github.com/vllm-project/vllm/issues/17645)

### Parallelism Strategy Trade-offs

> "High tensor parallelism introduces significant communication overhead between GPUs, especially during inference. A high TP degree does not always translate to better performance."

> "Lowering tensor parallelism means fewer GPUs share the model, leaving less room for KV cache. This can degrade optimizations like prefix caching."

— [BentoML](https://bentoml.com/llm/inference-optimization/data-tensor-pipeline-expert-hybrid-parallelism)

---

## Summary Table

### PD Disaggregation Performance

| Framework | Model | Hardware | Result | Source |
|---|---|---|---|---|
| **Dynamo (disaggregated)** | DeepSeek-R1 671B | GB200 NVL72 | 30× throughput boost | NVIDIA Blog |
| **Dynamo (disaggregated)** | Llama 70B | Hopper | >2× throughput improvement | NVIDIA Blog |
| **DynamoLLM** | Various | Azure GPUs | 52% energy savings, 61% cost reduction | HPCA 2025 |
| **Berkeley Dynamic Partitioning** | Llama 2 7B | L4/A100 | 40% TTFT reduction, 18% latency reduction | Berkeley Tech Report |

### Architecture Comparison

| Feature | llm-d | NVIDIA Dynamo | AIBrix | OME |
|---|---|---|---|---|
| **Inference Engine** | vLLM | vLLM, TRT-LLM | vLLM | SGLang |
| **PD Disaggregation** | ✅ | ✅ | ✅ | ✅ |
| **KV Cache Aware Routing** | ✅ | ✅ | ✅ | ✅ |
| **Kubernetes Native** | ✅ | Partial | ✅ | Partial |
| **CNCF Project** | ✅ (Sandbox) | ❌ | ❌ | ❌ |

---

## Related Topics

- [2000Gbps-NICs](./2000Gbps-NICs.md) — The network fabric these clusters run on (ConnectX-7, Thor Ultra 800G, RoCE vs InfiniBand)
- [GPU-Architectures](../README.md) — Hardware context (H100, Blackwell, NVLink) these clusters use
