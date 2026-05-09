# Ultra Ethernet Consortium (UEC)

> **Scope:** The Ultra Ethernet Consortium (UEC) specification, Ultra Ethernet Transport (UET) protocol, and ecosystem for AI/HPC networking.  
> **Last updated:** May 2026

## Table of Contents

- [Overview](#overview)
- [History & Consortium](#history--consortium)
- [UEC 1.0 Specification](#uec-10-specification)
- [Ultra Ethernet Transport (UET) Protocol](#ultra-ethernet-transport-uet-protocol)
- [Congestion Control](#congestion-control)
- [UEC vs InfiniBand](#uec-vs-infiniband)
- [UEC vs RoCEv2](#uec-vs-rocev2)
- [Deployment Status & Adoption](#deployment-status--adoption)
- [Key Papers & Reports](#key-papers--reports)
- [Key Repositories & Software](#key-repositories--software)
- [References](#references)

---

## Overview

The **Ultra Ethernet Consortium (UEC)** was launched in **mid-2023** under the **Linux Foundation's Joint Development Foundation** to rebuild Ethernet from the ground up for AI and HPC workloads.

> "Its goal is clear: to rebuild Ethernet from scratch, tailoring it for the demanding needs of modern data centers running large-scale AI and HPC workloads."  
> **Source:** [STORDIS — Ultra Ethernet Consortium Explained](https://stordis.com/ultra-ethernet-consortium/)

UEC released **Specification 1.0** on **June 11, 2025** (updated to **1.0.2** by January 28, 2026). The spec is **560+ pages** and covers the full networking stack — NICs, switches, optics, cables, transport, congestion control, and security — to enable **multi-vendor interoperability** without vendor lock-in.

> "UEC Spec 1.0 ignites a new era of interoperable, high performance, Ethernet Innovation"  
> **Source:** [Ultra Ethernet Consortium — UEC 1.0 Launch](https://ultraethernet.org/ultra-ethernet-consortium-uec-launches-specification-1-0-transforming-ethernet-for-ai-and-hpc-at-scale/)

---

## History & Consortium

### Timeline

| Date | Milestone |
|------|-----------|
| **Q1 2022** | Initial group (AMD, Broadcom, HPE, Intel, Microsoft) forms, initially called *HiPER* |
| **July 2022** | Consortium formally established |
| **July 2023** | Ultra Ethernet Consortium (UEC) officially announced; adds Arista, Cisco, Eviden, Meta, and others |
| **October 2023** | Formal collaboration with Open Compute Project (OCP) announced |
| **June 11, 2025** | **UEC Specification 1.0** released |
| **September 2025** | UEC 1.0.1 released (editorial corrections, RCCC algorithm fixes) |
| **January 28, 2026** | UEC 1.0.2 released (CMS congestion control corrections) |

> "By end of 2024: **100+ member companies, 1,500+ participants**. An open Linux Foundation JDF project."  
> **Source:** [arXiv:2508.08906v1 — Ultra Ethernet's Design Principles](https://arxiv.org/html/2508.08906v1)

### Founding Members

AMD, Intel, Broadcom, Cisco, Arista, Meta, Microsoft, Dell, Samsung, Huawei, and 50+ technology companies.

> "The UEC unites **50+ technology companies** and **hundreds of engineers**."  
> **Source:** [STORDIS](https://stordis.com/ultra-ethernet-consortium/)

### OCP Collaboration

> "While OCP focuses on open hardware design and platform specifications, **UEC is bringing a new Ethernet transport layer to that foundation** — one optimized for the massive data flows and coordination required in AI clusters."  
> **Source:** [STORDIS](https://stordis.com/ultra-ethernet-consortium/)

This alignment ensures open network software stacks like **SONiC** integrate seamlessly with UEC-compliant devices.

---

## UEC 1.0 Specification

### Core Capabilities

- **Modern RDMA for Ethernet and IP** — intelligent, low-latency transport for high-throughput environments
- **Open Standards and Interoperability** — avoids vendor lock-in
- **End-to-End Scalability** — scales to **millions of endpoints**

> "As Chair of the Ultra Ethernet Consortium, I'm proud to announce the release of the UEC 1.0 specification — a major milestone in our mission to redefine Ethernet for the age of AI and high-performance computing. This standard is the result of unprecedented collaboration across the industry, and it delivers the low-latency, high-bandwidth, and intelligent transport needed for the most demanding workloads of today and tomorrow."  
> — **Dr. J Metz**, Chair of the Steering Committee, UEC  
> **Source:** [UEC Launch Announcement](https://ultraethernet.org/ultra-ethernet-consortium-uec-launches-specification-1-0-transforming-ethernet-for-ai-and-hpc-at-scale/)

### Specification Layers

| Layer | Key Enhancement |
|-------|----------------|
| **Physical Layer** | Standard support for **QSFP-DD and OSFP optics** |
| **Data Link Layer** | Native **RDMA** support |
| **Transport Layer** | New **Ultra Ethernet Transport (UET)** protocol for intelligent flow control |
| **Management/Observability** | Open APIs for observability and automation |

> "The specification covers the full networking stack — **NICs, switches, optics, and cables** — to enable seamless **multi-vendor integration**."  
> **Source:** [UEC Launch](https://ultraethernet.org/ultra-ethernet-consortium-uec-launches-specification-1-0-transforming-ethernet-for-ai-and-hpc-at-scale/)

---

## Ultra Ethernet Transport (UET) Protocol

UET is a **clean-slate RDMA transport** designed for demanding AI and HPC workloads.

> "UET embraces Remote Direct Memory Access (RDMA) mechanisms due to their superior performance, providing hardware-centric placement directly from network to host or accelerator memory, known as direct data placement. This bypasses the operating system kernel, ensuring the lowest latency and highest performance."  
> **Source:** [UEC Specification Update](https://ultraethernet.org/ultra-ethernet-specification-update/)

### Key Design Principles

| Principle | Description |
|-----------|-------------|
| **Ephemeral connections** | No handshake required before transmission. Connection state is established *while communicating* and discarded at transaction end. |
| **Packet spraying** | Senders distribute packets across many paths, avoiding ECMP flow collisions. |
| **Lossy (best-effort) by default** | Avoids head-of-line blocking; optional PFC for lossless environments. |
| **Out-of-order delivery** | Native support without receiver reordering overhead. |
| **Hardware offload-ready** | Can be fully hardware-accelerated. |

> "Unlike InfiniBand, the last major standardization effort in high-performance networking over two decades ago, UE leverages the expansive Ethernet ecosystem and the **1,000x gains in computational efficiency per moved bit** to deliver a new era of high-performance networking."  
> **Source:** [arXiv:2508.08906v1](https://arxiv.org/html/2508.08906v1)

### Why Not Just RoCEv2?

> "**RoCEv2** embedded InfiniBand's transport into routable Ethernet but inherited its core constraints: lossless transport, strict in-order packet delivery, and reliance on Priority Flow Control (PFC)."  
> **Source:** [arXiv:2508.08906v1](https://arxiv.org/html/2508.08906v1)

> "PFC requires separate traffic classes with substantial headroom buffers and suffers from **congestion spreading and head-of-line blocking**. In-order delivery limits path choice, leading to suboptimal performance."  
> **Source:** [arXiv:2508.08906v1](https://arxiv.org/html/2508.08906v1)

### UET Profiles

| Profile | Includes |
|---------|----------|
| **AI Base** | Core AI use cases (distributed inference and training) |
| **AI Full** | Adds Deferrable Send, exact-matching tagged sends, extended atomic operations |
| **HPC** | Advanced tagging semantics, additional atomic operations, full rendezvous support |

> "The software for UET is based on **libfabric v2.0 APIs**, extended to support the new UET."  
> **Source:** [UEC Specification Update](https://ultraethernet.org/ultra-ethernet-specification-update/)

### IANA Port Assignment

UET has been assigned **UDP port 4793** by IANA.

> "UET has been assigned **UDP port 4793** by IANA — 'not only a beautiful large prime number, but also (++RoCEv2)++.'"  
> **Source:** [arXiv:2508.08906v1](https://arxiv.org/html/2508.08906v1)

---

## Congestion Control

UET introduces novel congestion management optimized for AI/HPC traffic patterns.

### Mechanisms

| Mechanism | Description |
|-----------|-------------|
| **Packet spraying** | Distribute packets across paths to avoid ECMP collisions |
| **Real-time feedback** | Lightweight congestion info from endpoints, transit nodes, and last hop |
| **Dynamic path shifting** | Move packets away from congested paths immediately |
| **Sender-based control** | Adaptive window scheme using RTT, ECN markings, and packet loss |
| **Receiver-based control** | Sender requests permission; receiver grants credits. Supports optimistic transmission via pre-allocated credits. |
| **Packet trimming** | Optional switch feature truncates (rather than drops) packets during congestion. Provides early explicit congestion indication for fast loss recovery. |

> "UET overcomes ECMP limitations with optimized load-balancing ... Senders distribute packets across many paths, avoiding ECMP flow collisions and loading links more evenly."  
> **Source:** [UEC Specification Update](https://ultraethernet.org/ultra-ethernet-specification-update/)

> "**Instantaneous ramp**: Can ramp from 'zero' to 'wire rate' instantly and back off rapidly using an AI/HPC-adapted additive-increase/multiplicative-decrease scheme."  
> **Source:** [UEC Specification Update](https://ultraethernet.org/ultra-ethernet-specification-update/)

---

## UEC vs InfiniBand

| Aspect | UEC | InfiniBand |
|--------|-----|------------|
| **Ecosystem** | Open Ethernet ecosystem | Proprietary, NVIDIA/Mellanox dominated |
| **Hardware** | Standard Ethernet NICs, switches, optics | Dedicated IB NICs, switches, cables |
| **Vendor lock-in** | Designed to prevent | Deep vendor lock-in |
| **Scalability target** | Millions of endpoints | Tens of thousands |
| **Congestion control** | Programmable, multi-path aware | Fixed DCQCN / fixed transport |
| **Loss model** | Best-effort (lossy) by default | Lossless (PFC-dependent) |
| **Route diversity** | Per-packet spraying | In-order delivery limits choices |
| **Cost** | Ethernet economies of scale | Premium IB hardware pricing |

> "UEC delivers **InfiniBand performance** with **Ethernet's openness and flexibility** — no exotic cables, no licensing hurdles, no single-vendor dependency."  
> **Source:** [STORDIS](https://stordis.com/ultra-ethernet-consortium/)

---

## UEC vs RoCEv2

| Aspect | UEC / UET | RoCEv2 |
|--------|-----------|--------|
| **Transport** | Clean-slate UET | InfiniBand transport over UDP/IP |
| **Connection model** | Ephemeral / connectionless | Connection-oriented (QP-based) |
| **Loss handling** | Best-effort, fast recovery | Requires lossless fabric (PFC) |
| **Path diversity** | Per-packet spraying | ECMP flow hashing |
| **In-order requirement** | No — out-of-order supported | Yes |
| **Congestion control** | Programmable, multi-source feedback | DCQCN or vendor-fixed |
| **Scalability** | Millions of endpoints | Practical limit ~10K–100K QPs |
| ** Deployment complexity** | Lower (standard Ethernet) | High (PFC/ECN/DCB tuning) |

> "**RoCEv2** embedded InfiniBand's transport into routable Ethernet but inherited its core constraints ... UEC changes that."  
> **Source:** [arXiv:2508.08906v1](https://arxiv.org/html/2508.08906v1)

---

## Deployment Status & Adoption

> "UEC 1.0 is built on the standard Ethernet protocol, designed for low-friction deployment across hardware through applications."  
> **Source:** [UEC Launch](https://ultraethernet.org/ultra-ethernet-consortium-uec-launches-specification-1-0-transforming-ethernet-for-ai-and-hpc-at-scale/)

- **Compliance programs** and active implementations already in progress.
- **2026 focus:** Helping industry implement UEC with confidence. Clear documentation and practical examples are a major priority.

> "First, we want to help the industry implement UEC with confidence. Supporting implementations and helping members put UEC into practice will be a major focus in the year ahead."  
> — **Dr. J Metz**, UEC Chair  
> **Source:** [UEC 2025 in Review](https://ultraethernet.org/uec-2025-in-review-preparing-for-what-comes-next-a-letter-from-uecs-chair/)

### Target Beneficiaries

- Cloud infrastructure operators
- Hyperscalers
- DevOps teams
- AI engineering leads

---

## Key Papers & Reports

| Paper / Report | Date | URL |
|----------------|------|-----|
| **Ultra Ethernet's Design Principles and Architectural Innovations** | arXiv 2025 | [arXiv:2508.08906v1](https://arxiv.org/html/2508.08906v1) |
| **UEC 1.0 Whitepaper (Intersect360 Research)** | June 2025 | [UEC1.0Whitepaper.pdf](https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UEC1.0Whitepaper.pdf) |
| **UEC Specification v1.0.2** | January 2026 | [PDF](https://ultraethernet.org/wp-content/uploads/sites/20/2026/01/UE-Specification-1.0.2-1.pdf) |
| **UEC Release Notes v1.0.2** | January 2026 | [PDF](https://ultraethernet.org/wp-content/uploads/sites/20/2026/01/UE-Specification-1.0.2-release-notes.pdf) |
| **A (mostly) Unbiased Review of the Ultra Ethernet Specification v1.0** | Medium 2025 | [Link](https://medium.com/@tom_84912/a-mostly-unbiased-review-of-the-ultra-ethernet-specification-10d816227839) |
| **Ultra Ethernet for Next-Gen AI and HPC (HotI 2025 Slides)** | August 2025 | [PDF](https://hoti.org/assets/slides/2025_08_21_day2_Invited_talk_Ultra_Ethernet.pdf) |

---

## Key Repositories & Software

| Repo / Project | Description |
|----------------|-------------|
| [libfabric](https://github.com/ofiwg/libfabric) | Open Fabrics Interfaces — base for UET software |
| [SONiC](https://github.com/sonic-net/SONiC) | Open network OS expected to integrate UEC |

---

## References

- [Ultra Ethernet Consortium — Official Site](https://ultraethernet.org/)
- [UEC Specification 1.0 Launch Announcement](https://ultraethernet.org/ultra-ethernet-consortium-uec-launches-specification-1-0-transforming-ethernet-for-ai-and-hpc-at-scale/)
- [STORDIS — Ultra Ethernet Consortium Explained](https://stordis.com/ultra-ethernet-consortium/)
- [UEC Specification Update (UET Details)](https://ultraethernet.org/ultra-ethernet-specification-update/)
- [arXiv:2508.08906v1 — Design Principles](https://arxiv.org/html/2508.08906v1)
- [UEC 2025 in Review](https://ultraethernet.org/uec-2025-in-review-preparing-for-what-comes-next-a-letter-from-uecs-chair/)
- [Medium — Unbiased Review of UEC v1.0](https://medium.com/@tom_84912/a-mostly-unbiased-review-of-the-ultra-ethernet-specification-10d816227839)
- [Intersect360 UEC 1.0 Whitepaper PDF](https://ultraethernet.org/wp-content/uploads/sites/20/2025/06/UEC1.0Whitepaper.pdf)
- [UEC Spec v1.0.2 PDF](https://ultraethernet.org/wp-content/uploads/sites/20/2026/01/UE-Specification-1.0.2-1.pdf)

---

*Last updated: 2026-05-02*
