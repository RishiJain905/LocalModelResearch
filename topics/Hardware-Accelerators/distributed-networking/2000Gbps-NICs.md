# 2000Gbps NICs

## 📌 Overview

As of April 2026, no true 2Tbps (2000Gbps) single NIC exists commercially. The current frontier is **800Gbps (800G)** — the **Broadcom Thor Ultra** (announced October 2025, now sampling) is the industry's first 800G AI Ethernet NIC, fully UEC 1.0 compliant. The previous generation, **NVIDIA ConnectX-7**, maxes out at 400Gbps (NDR InfiniBand / 400GbE). **NVLink 5** (Blackwell) delivers 1.8TB/s per GPU for *scale-up* within-rack GPU interconnect — fundamentally different from network fabrics connecting between racks. The **Ultra Ethernet Consortium (UEC)** published Specification 1.0 in June 2025 to create an open, Ethernet-based alternative to InfiniBand for AI/HPC workloads.

---

## Definitions & Standards

### RDMA (Remote Direct Memory Access)

Enables direct memory access between GPUs across the network without CPU involvement — critical for AI training workloads.

### RoCEv2 (RDMA over Converged Ethernet v2)

RDMA protocol encapsulated in UDP/IP over Ethernet, used by Meta, Google, and Microsoft. The dominant fabric for AI clusters under ~10K GPUs.

### InfiniBand NDR (Network Data Rate)

400Gb/s InfiniBand speed grade — the current maximum for InfiniBand.

### Ultra Ethernet Consortium (UEC) Specification 1.0

> "The Ultra Ethernet 1.0 Specification is the result of an outstanding collaboration of AI, HPC, and networking experts, system and silicon vendors, and network operators."

> "As Chair of the Ultra Ethernet Consortium, I'm proud to announce the release of the UEC 1.0 specification—a major milestone in our mission to redefine Ethernet for the age of AI and high-performance computing."

— [UEC Press Release](https://ultraethernet.org/ultra-ethernet-consortium-uec-launches-specification-1-0-transforming-ethernet-for-ai-and-hpc-at-scale/), June 11, 2025

**Specification:** [UE-Specification-6.11.25.pdf](https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UE-Specification-6.11.25.pdf)

---

## Key Hardware Specifications

### NVIDIA ConnectX-7

> "ConnectX-7 supports InfiniBand and Ethernet protocols and a range of speeds up to 400 gigabits per second (Gb/s). It enables a wide range of advanced, scalable, and secure networking solutions."

> "InfiniBand: NDR 400Gb/s (Default speed). Ethernet: 400GbE."

— [NVIDIA ConnectX-7 Datasheet](https://www.nvidia.com/content/dam/en-zz/Solutions/networking/infiniband/connectx-7-datasheet.pdf) (April 2021)

**Max bandwidth: 400Gb/s** — was the standard AI networking NIC in 2023-2024.

### Broadcom Thor Ultra 800G (NEW — October 2025)

> "Thor Ultra delivers on the vision of Ultra Ethernet Consortium for modernizing RDMA for large AI clusters. Designed from the ground up, Thor Ultra is the industry's first 800G Ethernet NIC and is fully feature compliant with UEC specification."

> "Thor Ultra introduces: Packet-Level multipathing for efficient load balancing. Out-of-Order packet delivery directly to XPU memory for maximizing fabric utilization. Selective retransmission for efficient data transfer. Programmable Receiver-based and Sender-based congestion control algorithms."

— [Broadcom Press Release](https://www.globenewswire.com/news-release/2025/10/14/3166262/19933/en/Broadcom-Introduces-Industry-s-First-800G-AI-Ethernet-NIC.html), October 14, 2025

**Product page:** [Broadcom P1800GO](https://www.broadcom.com/products/ethernet-connectivity/network-adapters/p1800go) — Single-Port 800GbE PCIe 6.0 x16

### NVIDIA NVLink 5 (Blackwell) — Scale-Up vs Scale-Out

> "A single NVIDIA Blackwell GPU supports up to 18 NVLink connections at 100 gigabytes per second each, delivering 1.8 terabytes per second of total bandwidth—14 times the bandwidth of PCIe Gen5."

> "The GB200 NVL72 system connects 72 GPUs in a single NVLink domain with 130 terabytes per second of aggregate bandwidth. NVIDIA's NVLink Switch enables 576 GPUs in a non-blocking compute fabric with over 1 petabyte per second of total bandwidth."

— [Introl Blog — NVLink and scale-up networking](https://introl.com/blog/nvlink-scale-up-networking-gpu-interconnect-infrastructure-2025), 2025

**Key distinction:** NVLink handles *scale-up* (within rack); Ethernet/InfiniBand handles *scale-out* (between racks). These are complementary fabrics — not competitors.

### NVIDIA BlueField-3 DPU

> "BlueField-3 DPU – 2x 200Gb/s FHHL form factor"

> "Ethernet - 1, 2, 4 ports with up to 400 Gb/s connectivity. InfiniBand - Single port of NDR (400Gb/s), or dual ports of NDR200/HDR (200Gb/s)."

— [NVIDIA BlueField-3 Datasheet](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/documents/datasheet-nvidia-bluefield-3-dpu.pdf)

---

## Key Sources

### Meta Engineering — RoCE at Scale (SIGCOMM 2024)

> "RDMA over Ethernet for Distributed AI Training at Meta Scale"

— [Meta Engineering Blog](https://engineering.fb.com/2024/08/05/data-center-engineering/roce-network-distributed-ai-training-at-scale/), August 5, 2024
**Paper:** [ACM SIGCOMM 2024](https://dl.acm.org/doi/10.1145/3651890.3672233)

### Meta RSC (Research SuperCluster)

> "We used RSC to train LLaMA...We used RSC to train the world's first AI-powered translation system for a primarily oral language, enabling Hokkien speakers to hold conversations with English speakers in real time."

— [Meta AI Blog](https://ai.meta.com/blog/supercomputer-meta-research-supercluster-2023/)

Meta operates 24,576 H100 GPUs per cluster, built both RoCEv2 and InfiniBand clusters in parallel.

### NCCLX — Collective Communication for 100K+ GPUs (Meta)

> "This paper presents the NCCLX collective communication framework, developed at Meta and engineered to optimize performance across the full LLM lifecycle, from the synchronous demands of large-scale training to the low-latency requirements of inference."

> "Empirical gains on Llama4: Up to **12% reduction in per-step latency**; **11× faster initialization** at 96K GPU scale."

— [arXiv:2510.20171v4](https://arxiv.org/html/2510.20171v4)

**Code:** [meta-pytorch/torchcomms](https://github.com/meta-pytorch/torchcomms/tree/main/comms/ncclx)

### RoCE vs InfiniBand Debate

> "RoCEv2 (RDMA over Converged Ethernet version 2) has quietly become the dominant GPU interconnect for AI training clusters — and most network engineers haven't noticed yet."

> "For deployments up to ~10K GPUs, properly tuned Ethernet with RoCEv2 delivers 85-95% of InfiniBand's training throughput at a fraction of the cost."

— [DEV Community / First Pass Lab](https://dev.to/firstpasslab/roce-vs-infiniband-why-ethernet-is-winning-the-ai-data-center-networking-war-20g7)

### UEC 1.0 White Paper (Intersect360 Research)

> "The Ultra Ethernet Consortium (UEC) is enhancing Ethernet's capabilities through a collaborative, standards-driven approach, with the goal of standardizing a high-performance Ethernet stack purpose-built for the unique demands of HPC and AI."

— [UEC 1.0 White Paper](https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UEC1.0Whitepaper.pdf)

---

## Repositories & Tools

| Tool | URL | Description |
|------|-----|-------------|
| **NVIDIA NCCL** | [github.com/NVIDIA/nccl](https://github.com/NVIDIA/nccl) | Standard collective communication library for GPUs |
| **Meta NCCLX** | [github.com/meta-pytorch/torchcomms](https://github.com/meta-pytorch/torchcomms/tree/main/comms/ncclx) | Meta's custom collective comms for 100k+ GPUs |
| **FUSCO** | [github.com/infinigence/FUSCO](https://github.com/infinigence/FUSCO) | NCCL abstraction for diverse cluster networks |
| **NVIDIA SHARP** | [network.nvidia.com/files/related-docs/solutions/hpc/paperieee_copyright.pdf](https://network.nvidia.com/files/related-docs/solutions/hpc/paperieee_copyright.pdf) | In-network aggregation for collective ops |

---

## Debates & Controversies

### InfiniBand vs. RoCE/Ethernet

| Aspect | InfiniBand | RoCE/Ethernet |
|--------|-----------|--------------|
| Latency | Lowest at extreme scale (10K+ GPUs) | 85-95% of IB throughput for <10K GPUs |
| Cost | Proprietary, expensive | Standard Ethernet, lower cost |
| Failure Recovery | Static fabric, ~30× longer | Dynamic, BGP/BFD, faster |
| Optical Reliability | Up to 5% optics failure rate/month | Better copper reach, faster recovery |
| Scale | Best optimized at 10K+ GPUs | Better for 256-10K GPU clusters |

### PFC (Priority Flow Control) Risks

Required for lossless RoCE but introduces: PFC deadlock, pause frame storms, RDMA livelock. Microsoft Research addressed all of these with DCQCN congestion control and DSCP-based PFC.

### NVLink vs. Network Fabrics

NVLink 5 (1.8TB/s per GPU) operates *within* racks; Ethernet/InfiniBand at 400-800Gb/s connects *between* racks. The 18× bandwidth gap means these serve fundamentally different purposes.

---

## Summary Table

| NIC / Fabric | Max Speed | Type | Notes |
|---|---|---|---|
| **ConnectX-7** | 400Gb/s | InfiniBand NDR / 400GbE | Previous gen AI standard |
| **Broadcom Thor Ultra** | 800Gb/s | 800G Ethernet (UEC 1.0) | First 800G NIC; announced Oct 2025 |
| **BlueField-3 DPU** | 400Gb/s | Ethernet/InfiniBand | DPU form factor |
| **NVLink 5 (Blackwell)** | 1.8 TB/s per GPU | Scale-up interconnect | Within-rack only; 14× PCIe Gen5 |
| **GB200 NVL72 NVLink domain** | 130 TB/s aggregate | Scale-up | 72 GPUs in single NVLink domain |
| **UEC Specification 1.0** | — | Ethernet standard | Released June 11, 2025 |

### Key Performance Numbers

| Finding | Value | Source |
|---|---|---|
| RoCE throughput vs IB (<10K GPUs) | 85-95% | First Pass Lab |
| NCCLX init speedup (96K GPUs) | 11× faster | arXiv:2510.20171 |
| NCCLX per-step latency reduction (Llama4) | 12% | arXiv:2510.20171 |
| NVIDIA networking revenue (FY2024) | $7.3B | Introl Blog |
| Spectrum-X annualized run rate | >$10B | Introl Blog |

---

## Related Topics

- [llm-d-Cluster-Setups](./llm-d-Cluster-Setups.md) — How these NICs/fabrics connect in Kubernetes-based distributed inference clusters
- [GPU-Architectures](../README.md) — Hardware context (Blackwell, H100, NVLink)
