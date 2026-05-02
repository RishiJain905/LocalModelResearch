# RoCE v2 (RDMA over Converged Ethernet v2)

> **Scope:** RoCE v2 protocol, deployment for AI training clusters, network fabric configuration, performance benchmarks, and comparison vs InfiniBand.  
> **Last updated:** May 2026

## Table of Contents

- [Overview](#overview)
- [RoCE v1 vs v2](#roce-v1-vs-v2)
- [Technical Principles](#technical-principles)
- [Lossless Ethernet Requirements](#lossless-ethernet-requirements)
- [Network Topology for AI Clusters](#network-topology-for-ai-clusters)
- [Congestion Control: DCQCN](#congestion-control-dcqcn)
- [Meta's RoCE Deployment at Scale](#metas-roce-deployment-at-scale)
- [Performance Benchmarks](#performance-benchmarks)
- [RoCE v2 vs InfiniBand](#roce-v2-vs-infiniband)
- [Deployment Complexity & Pain Points](#deployment-complexity--pain-points)
- [Vendor Implementations](#vendor-implementations)
- [Linux Kernel & Tools](#linux-kernel--tools)
- [References](#references)

---

## Overview

**RoCE v2** (RDMA over Converged Ethernet version 2) is the dominant protocol for building **lossless Ethernet networks** that deliver ultra-low latency, massive throughput, and minimal CPU overhead for AI/HPC workloads.

> "In the fast-evolving world of AI training, high-performance computing (HPC), and cloud infrastructure, network performance is no longer just a supporting role — it's the bottleneck breaker."  
> **Source:** [fibermall.com — RoCEv2 Explained](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)

RoCE v2 encapsulates RDMA service in **UDP packets** (UDP port 4791) for transport over standard Ethernet fabrics, making it routable at Layer 3 — unlike RoCE v1 which was Layer 2 only.

> "AI training clusters demand networking that traditional Ethernet was never designed for: lossless behavior, microsecond latency, and sustained 400 Gbps+ throughput between GPUs."  
> **Source:** [netpilot.io — RoCEv2 vs InfiniBand](https://www.netpilot.io/blog/ai-cluster-networking-rocev2)

---

## RoCE v1 vs v2

| Feature | RoCE v1 | RoCE v2 |
|---------|---------|---------|
| **Layer** | Layer 2 (Ethertype `0x8915`) | Layer 3 (UDP/IP, **port 4791**) |
| **Routability** | Single subnet only | Fully routable across networks |
| **IP header** | No | Yes (UDP encapsulation) |
| **Release** | 2010 | **2014** |
| **Deployment** | Limited / legacy | **Mainstream today** |

> "RoCEv2 dominates because it is **routable, cheap, and leverages Ethernet's massive ecosystem**."  
> **Source:** [fibermall.com](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)

---

## Technical Principles

### RDMA Basics

**Remote Direct Memory Access (RDMA)** allows data to move directly from the memory of one computer to another **without involving the CPU, OS kernel, or multiple data copies**.

> "RDMA eliminates these, enabling **zero-copy**, **kernel-bypass**, and **CPU offload** — perfect for AI workloads where GPUs need to exchange gigabytes of gradients instantly."  
> **Source:** [fibermall.com](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)

### Key Benefits

| Benefit | Typical Improvement |
|---------|-------------------|
| **Latency** | ~2 μs vs 10–50 μs (TCP/IP) |
| **CPU usage** | Near-zero for data path |
| **Throughput** | Near-line-rate at 400 Gbps |

> "For most GPU cluster deployments in 2026, properly configured Ethernet with RoCEv2 delivers **85-95% of InfiniBand's training throughput** according to industry benchmarks — at significantly lower cost and with skills that network engineers already have."  
> **Source:** [FirstPassLab — RoCE vs InfiniBand](https://firstpasslab.com/blog/2026-03-09-roce-vs-infiniband-ai-data-center-networking/)

---

## Lossless Ethernet Requirements

RoCEv2 demands **zero packet loss**, as RDMA has no built-in retransmission for unreliable transports. Traditional Ethernet drops packets under congestion — unacceptable for RDMA.

### Core Technologies

| Technology | Standard | Purpose |
|------------|----------|---------|
| **PFC (Priority Flow Control)** | IEEE 802.1Qbb | Hop-by-hop pause mechanism |
| **ECN (Explicit Congestion Notification)** | RFC 3168 | End-to-end congestion signaling |
| **DCQCN (Data Center Quantized Congestion Notification)** | — | End-to-end congestion control for RoCE |
| **DCB (Data Center Bridging)** | IEEE 802.1Qxx | Set of standards for converged networks |
| **ETS (Enhanced Transmission Selection)** | IEEE 802.1Qaz | Bandwidth allocation between traffic classes |
| **DCBX (Data Center Bridging Exchange)** | IEEE 802.1Qaz | Auto-configuration of DCB features |

> "Data Center Bridging, with its cornerstone features of Priority-based Flow Control (PFC) and Explicit Congestion Notification (ECN), is not merely an optimization or nice-to-have feature; it is a **critically important and fundamental enabler** for building efficient, reliable, and scalable AI computing solutions with large investments in GPUs."  
> **Source:** [Edgecore Networks — Powering AI at Scale](https://www.edge-core.com/blogs/powering-ai-at-scale-the-indispensable-role-of-data-center-bridging-with-pfc-and-ecn-for-large-gpu-deployments/)

### Latency by Transport

| Transport | Typical Latency |
|-----------|----------------|
| **InfiniBand** | < 10 μs |
| **RoCE v2** | 15–30 μs |
| **NVMe/TCP** | 50–100 μs |

> "RoCEv2 is the ecosystem choice — runs on standard Ethernet your team already operates, with **2-5 μs latency** that's acceptable for most AI workloads."  
> **Source:** [netpilot.io](https://www.netpilot.io/blog/ai-cluster-networking-rocev2)

> "InfiniBand: Adds 2-5 μs latency. Purpose-built fabric (ConnectX adapters, Quantum switches). Dominant in HPC/AI clusters. RoCEv2: Adds 5-10 μs latency. Uses standard Ethernet switches. **Requires lossless fabric:** PFC, ECN."  
> **Source:** [Murat Karslioglu — NVMe-oF: Promise and Pain](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

---

## Network Topology for AI Clusters

### Dual-Network Architecture (Meta)

Meta's training clusters use **two completely independent networks**:

| Network | Purpose | Design |
|---------|---------|--------|
| **Frontend (FE)** | Data ingestion, checkpointing, logging | Standard DC hierarchy |
| **Backend (BE)** | Training traffic | Non-blocking fabric, lossless, high-bandwidth, low-latency |

> "Training clusters use two completely independent networks."  
> **Source:** [Meta Engineering — RoCE Networks](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/)

### Two-Stage Clos Topology ("AI Zone")

> "The backend fabric uses a two-stage Clos topology within each AI zone."  
> **Source:** [Meta Engineering](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/)

| Layer | Component | Connectivity |
|-------|-----------|-------------|
| **Leaf** | Rack Training Switch (RTSW) | GPUs within rack via **copper DAC** |
| **Spine** | Cluster Training Switch (CTSW) | Scale-out via **single-mode fiber, 400G transceivers** |

### Multi-Rail Deployment

> "**Multi-rail deployment** connects each server's **8 GPUs** to separate Leaf switches, maximizing POD size and reducing cross-POD congestion."  
> **Source:** [fibermall.com](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)

---

## Congestion Control: DCQCN

**DCQCN** (Data Center Quantized Congestion Notification) is the congestion control algorithm used with RoCEv2.

### Performance Sensitivity

> "RDMA networks are **extremely sensitive to packet loss**. AI workloads rely on synchronized operations like AllReduce, where the slowest packet dictates performance."  
> **Source:** [Oracle Cloud Infrastructure Blog](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)

### Impact of Packet Loss

| Loss Rate | Effect |
|-----------|--------|
| **0.1%** | GPU utilization drops by 10%+ |
| **1%** | Throughput collapses by 90%+ |

> "Without lossless delivery, RDMA becomes a liability instead of an advantage."  
> **Source:** [Oracle Cloud Infrastructure Blog](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)

### Meta's DCQCN Experience at 400G

> "Attempted to tune DCQCN for 400G networks (doubled ECN thresholds vs 200G). **Result: Degraded performance.** Investigation revealed **firmware bugs** in DCQCN implementation, plus reduced visibility."  
> **Source:** [Meta Engineering](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/)

---

## Meta's RoCE Deployment at Scale

Meta operates one of the world's largest AI training networks, built on RoCEv2.

> "We have successfully expanded our RoCE networks, evolving from prototypes to the deployment of numerous clusters, each accommodating **thousands of GPUs**. These RoCE clusters support an extensive range of production distributed GPU training jobs, including ranking, content recommendation, content understanding, natural language processing, and GenAI model training, among other workloads."  
> **Source:** [Meta Engineering Blog](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/)

### Scale

- **24,000 H100 GPUs** in a single RoCE-based cluster (used for Llama 3 training)
- Clusters span multiple racks with **NVIDIA H100 8 GPUs + 8 Mellanox 400G dual-port CX7 NICs** per server

> "Our paper, 'RDMA over Ethernet for Distributed AI Training at Meta Scale,' provides the details on how we design, implement, and operate one of the world's largest AI networks at scale."  
> — SIGCOMM 2024  
> **Source:** [ACM DL](https://dl.acm.org/doi/10.1145/3651890.3672233)

### Routing Evolution

Meta evolved their routing through three stages:

| Stage | Approach | Result |
|-------|----------|--------|
| **Stage 1: ECMP** | Standard 5-tuple hash | Poor performance (low entropy) |
| **Stage 2: Path Pinning** | Pin by destination slice | >30% degradation on partial rack allocation |
| **Stage 3: QP Scaling + E-ECMP** | Hash on destination QP field | **Up to 40% improvement** for AllReduce |

> "Leveraged **Queue Pair (QP) scaling** in the collective library to increase flow count ... switches perform **E-ECMP**, additionally hashing on the **destination QP field** ... Up to **40% performance improvement** for AllReduce vs baseline ECMP."  
> **Source:** [Meta Engineering](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/)

---

## Performance Benchmarks

### IBM Research RoCE Deployment

> "We deployed our solution to a number of GPU clusters spanning multiple racks of servers with Nvidia's H100 8 GPUs and 8 Mellanox 400G dual-port CX7 NICs per server using a 2-stage Clos topology. Using network micro-benchmarks and NCCL benchmarks that mimic the demands of LLM model training, our results show that the network **sustains near-line-rate throughput** under traffic patterns where up to 100% of the traffic traverses the spine layer. In production deployments, running LLM training jobs for months, we observe that our RoCE network **achieves similar performance compared to InfiniBand networks**."  
> **Source:** [IBM Research — Designing RoCE Networks for AI Workloads](https://research.ibm.com/publications/designing-roce-networks-for-ai-workloads)

### Oracle Cloud H100 Benchmarks

| Metric | Value |
|--------|-------|
| **RDMA NIC bandwidth (400Gbps)** | 48.77 GB/s |
| **GPU to GPU Latency over RDMA** | 29 μs |
| **NCCL on single node (8 GPUs)** | 476.37 GB/s |

> **Source:** [Oracle Cloud Infrastructure Blog](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)

### StarWind NVMe-oF Benchmark (April 2024)

| Metric | NVMe-oF RDMA | NVMe-oF TCP | iSCSI |
|--------|-------------|-------------|-------|
| **4K Read IOPS** | 1,558,000 | 1,503,000 | 1,272,000 |
| **4K Read Latency** | 0.082 ms | 0.382 ms | 0.806 ms |

> **Source:** [StarWind — iSCSI vs NVMe-oF](https://www.starwindsoftware.com/blog/iscsi-vs-nvme-of-performance-comparison/)

---

## RoCE v2 vs InfiniBand

| Aspect | RoCE v2 | InfiniBand |
|--------|---------|------------|
| **Latency** | 15-30 μs | < 10 μs |
| **Throughput** | Near-line-rate | Near-line-rate |
| **Network** | Standard Ethernet | Dedicated IB fabric |
| **Cost** | Lower (Ethernet economies) | Higher (specialized HW) |
| **Skills** | Standard CCIE DC | Specialized IB training |
| **Vendor** | Multiple (Mellanox, Broadcom, Cisco) | Primarily NVIDIA/Mellanox |
| **Scalability** | Proven at 10K+ GPUs | Proven at 100K+ GPUs |
| **Deployment time** | Months (PFC tuning) | Faster turnkey |
| **Training throughput** | 85-95% of IB | Baseline |

> "RoCEv2 matches InfiniBand latency and throughput for AI workloads ... Switch to RoCE by upgrading NICs only — no full IB rip-and-replace."  
> **Source:** [fibermall.com](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)

---

## Deployment Complexity & Pain Points

### The "Dirty Secret" of RoCEv2

> "The dirty secret of RoCEv2: Works flawlessly in controlled single-rack environments. **Budget six months for a 500-node multi-tier cluster.**"  
> **Source:** [Murat Karslioglu — NVMe-oF: Promise and Pain](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

### Key Pain Points

| Issue | Description |
|-------|-------------|
| **PFC/ECN tuning** | Requires meticulous per-port configuration |
| **Firmware bugs** | DCQCN firmware issues at 400G (Meta experience) |
| **DCB consistency** | Must be configured on every switch, NIC, and kernel |
| **Congestion spreading** | Head-of-line blocking with lossless fabrics |
| **Cross-zone oversubscription** | Multi-zone clusters require careful scheduling |

> "AI data center roles require expertise in lossless Ethernet (PFC, ECN, DCQCN), VXLAN EVPN fabric design, QoS at scale, and understanding of RDMA concepts."  
> **Source:** [FirstPassLab](https://firstpasslab.com/blog/2026-03-09-roce-vs-infiniband-ai-data-center-networking/)

---

## Vendor Implementations

### NVIDIA / Mellanox

- **ConnectX-7** (400 Gbps dual-port) — primary RoCEv2 NIC for H100 clusters
- **ConnectX-8** (coming) — 800 Gbps support
- **Spectrum-X** switches — optimized for AI workloads
- **Quantum-X** switches — IB/Ethernet converged

> "NVIDIA H100 8 GPUs and 8 Mellanox 400G dual-port CX7 NICs per server."  
> **Source:** [IBM Research](https://research.ibm.com/publications/designing-roce-networks-for-ai-workloads)

### Broadcom

- **Tomahawk 5** ASIC — 165.2 MB buffer, supports PFC/ECN
- Buffer sizing is critical for lossless at scale

> **Buffer ∝ Latency × Throughput × Bidirectionality × Safety Margin**  
> **Source:** [Oracle OCI Blog](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)

### Cisco

- **Nexus 9000** series with RoCEv2 support
- **Hyperfabric AI** — managed AI networking

### H3C

- AD-DC (Application-Driven Data Center) for automated RoCE deployment
- S12500R-2XL, S9855 (400G/800G), S9825 series

> "Deployment from weeks to days; troubleshooting from days to minutes."  
> **Source:** [fibermall.com](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)

---

## Linux Kernel & Tools

### Kernel Support

- **RDMA subsystem** in Linux kernel (since ~2.6)
- **Soft-RoCE** — software RoCE for testing without hardware
- **MLNX_OFED / MOFED** — NVIDIA's RDMA driver stack
- **Linux kernel 6.2+** — PCI P2PDMA support for GPUDirect Storage

### Diagnostic Tools

| Tool | Purpose |
|------|---------|
| `ib_write_bw` / `ib_read_bw` | Bandwidth testing |
| `perftest` | Full RDMA performance suite |
| `ibstat` / `ibstatus` | RDMA port status |
| `mst status` / `flint` | NIC firmware management |
| `dcqcn_tool` | DCQCN parameter tuning |

### Red Hat OpenShift SR-IOV Example

```yaml
apiVersion: sriovnetwork.openshift.io/v1
kind: SriovNetworkNodePolicy
metadata:
  name: mlnx-port-1
  namespace: openshift-sriov-network-operator
spec:
  deviceType: netdevice
  isRdma: true
  nicSelector:
    pfNames:
    - ens3f0np0
    vendor: 15b3
  nodeSelector:
    feature.node.kubernetes.io/network-sriov.capable: 'true'
  numVfs: 1
  priority: 99
  resourceName: port1
```
> **Source:** [Red Hat Developers — RoCE multi-node AI training](https://developers.redhat.com/learning/learn:openshift:roce-multi-node-ai-training-red-hat-openshift/resource/resources:training-set-and-cluster-configuration)

---

## References

- [Meta Engineering — RoCE Networks at Scale](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/)
- [ACM SIGCOMM 2024 Paper — RDMA over Ethernet at Meta Scale](https://dl.acm.org/doi/10.1145/3651890.3672233)
- [fibermall.com — RoCEv2 Ultimate Guide](https://www.fibermall.com/blog/rocev2-ultimate-guide-to-low-latency.htm)
- [netpilot.io — RoCEv2 vs InfiniBand (2026)](https://www.netpilot.io/blog/ai-cluster-networking-rocev2)
- [FirstPassLab — RoCE vs InfiniBand](https://firstpasslab.com/blog/2026-03-09-roce-vs-infiniband-ai-data-center-networking/)
- [IBM Research — Designing RoCE Networks for AI](https://research.ibm.com/publications/designing-roce-networks-for-ai-workloads)
- [Oracle OCI — H100 Benchmarks](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)
- [Edgecore — PFC/ECN for AI](https://www.edge-core.com/blogs/powering-ai-at-scale-the-indispensable-role-of-data-center-bridging-with-pfc-and-ecn-for-large-gpu-deployments/)
- [NVIDIA RoCE Docs](https://docs.nvidia.com/networking/display/MLNXOFEDv497100LTS/RDMA+over+Converged+Ethernet+(RoCE))
- [Red Hat — RoCE on OpenShift](https://developers.redhat.com/learning/learn:openshift:roce-multi-node-ai-training-red-hat-openshift/resource/resources:training-set-and-cluster-configuration)
- [Microsoft Answers — DCB for RoCE](https://learn.microsoft.com/en-us/answers/questions/5823870/configuring-rdma-roce-v2-bandwidth-dcb-for-26-node)
- [Murat Karslioglu — NVMe-oF: Promise and Pain](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

---

*Last updated: 2026-05-02*
