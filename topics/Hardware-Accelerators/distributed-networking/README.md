# distributed-networking

## 📌 Overview

Distributed-networking covers the two critical infrastructure layers for multi-node AI workloads: **high-speed NICs and network fabrics** (InfiniBand, RoCEv2, 800G Ethernet, NVLink) and **Kubernetes-based distributed inference orchestration** (llm-d, LeaderWorkerSet, NVIDIA Dynamo). Together these enable GPU clusters ranging from single-node to 100K+ GPU scale.

---

## Subtopics

| Subtopic | Description | Key Metric |
|----------|-------------|-----------|
| [2000Gbps-NICs](./2000Gbps-NICs.md) | 400G→800G NIC evolution, UEC 1.0, RoCE vs InfiniBand, NVLink scale-up | 800G Thor Ultra = current frontier |
| [llm-d-Cluster-Setups](./llm-d-Cluster-Setups.md) | Kubernetes-native distributed inference, PD disaggregation, CNCF Sandbox | 30× throughput via Dynamo disaggregation |

---

## Summary Matrix

| | 2000Gbps NICs | llm-d Cluster Setups |
|---|---|---|
| **Layer** | Physical network fabric | Orchestration / control plane |
| **Key Hardware** | ConnectX-7 (400G), Thor Ultra (800G), NVLink 5 (1.8TB/s) | vLLM + Kubernetes + LWS |
| **Key Standard** | UEC 1.0 (June 2025), RoCEv2, InfiniBand NDR | CNCF Sandbox, KServe |
| **Key Metric** | 85-95% IB throughput via RoCE at <10K GPUs | 30× throughput via PD disaggregation |

---

## Key Interconnections

```
2000Gbps NICs (physical fabric: 400G→800G Ethernet, InfiniBand NDR)
    ↓ connects
llm-d Cluster Setups (Kubernetes orchestration: vLLM, LWS, KServe)
    ↓ powers
Distributed LLM Inference (multi-node, PD disaggregation)
    ↑
NVLink 5 (1.8TB/s scale-up within rack, complementary to network fabrics)
```

- **2000Gbps NICs** provides the physical fabric — RoCEv2 (Ethernet) or InfiniBand NDR connecting GPUs across racks
- **llm-d/LWS** provides the Kubernetes-native orchestration layer that schedules inference across those networked GPUs
- **NVLink 5** handles within-rack GPU interconnect at 1.8TB/s — complementary to, not competing with, the network fabrics

---

## Related Topics

- [GPU-Architectures](../README.md) — Hardware context: H100, Blackwell, GB200 NVL72, ConnectX-7
- [Parallelism-Distributed-Serving](../../API-Servers/MISC/Parallelism-Distributed-Serving.md) — How distributed serving works at the software level (PagedAttention, DistServe, NCCL)
