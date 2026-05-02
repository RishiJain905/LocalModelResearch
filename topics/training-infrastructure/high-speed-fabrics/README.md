# high-speed-fabrics

## 📌 Overview

High-speed networking fabrics for distributed AI/HPC training — covering next-generation Ethernet standards (Ultra Ethernet), RDMA over Converged Ethernet (RoCE v2), and NVIDIA's GPUDirect technologies for zero-copy GPU communication.

## Subtopics

| File | Description |
|------|-------------|
| [Ultra-Ethernet.md](./Ultra-Ethernet.md) | Ultra Ethernet Consortium (UEC) 1.0 spec, UET transport protocol, packet spraying, congestion control, vs InfiniBand/RoCEv2 |
| [RoCE-v2.md](./RoCE-v2.md) | RDMA over Converged Ethernet v2 — lossless Ethernet (PFC/ECN/DCQCN), Clos topology, Meta's 24K GPU deployment, vs InfiniBand |
| [GPUDirect.md](./GPUDirect.md) | NVIDIA GPUDirect RDMA, GPUDirect Storage (GDS/cuFile), GPUDirect P2P — performance benchmarks, hardware support, NCCL integration |

---

## Related Topics

- [Storage-Streaming](../Storage-Streaming/) — NVMe-oF, Lustre, model streamers
- [Hardware-Accelerators](../../Hardware-Accelerators/) — GPU/NPU hardware
- [Inference-Serving](../../Inference-Serving/) — Model serving infrastructure

---

*Last updated: 2026-05-02*
