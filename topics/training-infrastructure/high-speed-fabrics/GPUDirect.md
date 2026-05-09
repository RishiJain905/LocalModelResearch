# NVIDIA GPUDirect

> **Scope:** NVIDIA GPUDirect family — GPUDirect RDMA, GPUDirect Storage (GDS), GPUDirect P2P — for ML/AI training and inference infrastructure.  
> **Last updated:** May 2026

## Table of Contents

- [Overview](#overview)
- [GPUDirect RDMA](#gpudirect-rdma)
- [GPUDirect Storage (GDS)](#gpudirect-storage-gds)
- [GPUDirect P2P](#gpudirect-p2p)
- [Software Stack](#software-stack)
- [Performance Benchmarks](#performance-benchmarks)
- [Supported Hardware](#supported-hardware)
- [Integration with NCCL](#integration-with-nccl)
- [Compatibility Mode & Fallback](#compatibility-mode--fallback)
- [Key Repositories](#key-repositories)
- [References](#references)

---

## Overview

**NVIDIA GPUDirect** is a family of technologies that enable **direct data paths between GPU memory and external devices** — network adapters, storage controllers, and other GPUs — bypassing the CPU and system memory.

> "GPUDirect® Storage (GDS) enables a direct data path for direct memory access (DMA) transfers between GPU memory and storage, which avoids a bounce buffer through the CPU."  
> **Source:** [NVIDIA GPUDirect Storage Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

The GPUDirect family consists of three main technologies:

| Technology | Use Case | Data Path |
|------------|----------|-----------|
| **GPUDirect RDMA** | Inter-node GPU-to-GPU over network | GPU ↔ NIC ↔ Network ↔ NIC ↔ GPU |
| **GPUDirect Storage (GDS)** | Direct storage-to-GPU data path | Storage ↔ GPU (bypassing CPU/DRAM) |
| **GPUDirect P2P** | Intra-node GPU-to-GPU | GPU ↔ GPU via PCIe/NVLink |

> "GPUDirect RDMA at the lowest level provides for data exchange 'directly' between a GPU and a non-GPU device like a Networking Adapter or possibly a FPGA device **in the same node**."  
> **Source:** [NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/the-relationship-between-gpudirect-rdma-gpudirect-p2p-nvidia-ipc-nccl-and-nvshmem/316874)

---

## GPUDirect RDMA

### What It Does

GPUDirect RDMA enables **direct memory-to-memory transfers** between GPU memory on one node and a network adapter (NIC) — bypassing CPU system memory entirely.

**In the data path:**
- GPU memory ↔ NIC (DMA transfer)
- NIC sends/receives over network
- Remote NIC ↔ Remote GPU memory (DMA)

> "NCCL is a communication library that is intended to support the types of collective communication that you might see with MPI. The basic paradigm is that of shared memory which is addressable and accessible from multiple processes; this shared memory can be on the device."  
> **Source:** [NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/the-relationship-between-gpudirect-rdma-gpudirect-p2p-nvidia-ipc-nccl-and-nvshmem/316874)

### Technical Requirements

| Requirement | Description |
|-------------|-------------|
| **BAR space** | GPU Base Address Register must be accessible from NIC |
| **nvidia_p2p driver** | Kernel module exposing GPU memory to external devices |
| **MOFED / MLNX_OFED** | NVIDIA's RDMA driver stack |
| **CUDA-aware MPI / NCCL** | User-space libraries that leverage GPUDirect RDMA |
| **Supported NICs** | Mellanox ConnectX-4+, NVIDIA BlueField |

### GPUDirect RDMA + RoCEv2

> "In multi-GPU systems with PCIe switches, traditional TCP/IP networking often relies on CPU-mediated paths. RDMA — particularly with GPUDirect — enables direct memory-to-memory transfers between NICs and GPUs in the PCIe P2P downlink domain, bypassing the CPU."  
> **Source:** [Oracle Cloud Infrastructure Blog](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)

### Performance Notes

> "Enabling RDMA (and especially GPUDirect RDMA) does improve throughput and latency, but the gains are often **modest for small-scale setups** (e.g., 2 nodes, 1 GPU each). The largest benefits from RDMA/GDRDMA are seen in **large tensor-parallel (TP) deployments**, where inter-node communication is the main bottleneck."  
> **Source:** [vLLM Discuss](https://discuss.vllm.ai/t/vllm-benchmarking-why-is-gpudirect-rdma-not-outperforming-standard-rdma-in-a-pipeline-parallel-setup/1377)

---

## GPUDirect Storage (GDS)

### What It Does

GDS enables **direct DMA transfers between GPU memory and storage devices** — local NVMe SSDs, distributed filesystems (Lustre, BeeGFS, GPFS, WekaFS), or network-attached storage — bypassing the CPU bounce buffer.

### Key Benefits

> "The direct path increases system bandwidth and decreases the latency and utilization load on the CPU."  
> **Source:** [NVIDIA GDS Configuration Guide](https://docs.nvidia.com/gpudirect-storage/configuration-guide/index.html)

| Benefit | Description |
|---------|-------------|
| **Direct path** | GPU memory ↔ storage without CPU first/last touch |
| **Performance multiplier** | Especially for pipelines fully migrated to GPU |
| **Interoperability** | Works alongside traditional OS file IO |
| **Generality** | Supports local/distributed filesystems, block interfaces |

### cuFile API

The `cuFile` API is the core interface for GDS:

> "cuFile APIs and their implementation is the mechanism by which CUDA adds support for file IO."  
> **Source:** [NVIDIA GDS Overview PDF](https://docs.nvidia.com/gpudirect-storage/pdf/overview-guide.pdf)

#### Explicit vs Implicit Modes

| Type | Description | Overhead |
|------|-------------|----------|
| **Explicit (proactive)** | cuFile API immediately invokes the transfer | **Lower** |
| **Implicit (reactive)** | GPU page miss induces transfer back to CPU/storage | Higher |

> "Reactive activity tends to induce more overhead. As a result of being explicit and proactive, GDS maximizes performance with its explicit cuFile APIs."  
> **Source:** [NVIDIA GDS Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

#### Code Comparison: POSIX vs cuFile

**POSIX (CPU bounce buffer + extra copy):**
```c
int fd = open(...);
void *sysmem_buf, *gpumem_buf;
sysmem_buf = malloc(buf_size);
cudaMalloc(gpumem_buf, buf_size);
pread(fd, sysmem_buf, buf_size);
cudaMemcpy(sysmem_buf, gpumem_buf, buf_size, H2D);
cuStreamSynchronize(0);
doit<<<gpumem_buf, ...>>>;
```

**cuFile (direct to GPU):**
```c
int fd = open(file_name, O_DIRECT,...);
CUFileHandle_t *fh;
CUFileDescr_t desc;
desc.type=CU_FILE_HANDLE_TYPE_OPAQUE_FD;
desc.handle.fd = fd;
cuFileHandleRegister(&fh, &desc);
void *gpumem_buf;
cudaMalloc(gpumem_buf, buf_size);
cuFileRead(&fh, gpumem_buf, buf_size, ...);
doit<<<gpumem_buf, ...>>>;
```
> **Source:** [NVIDIA GDS Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

### O_DIRECT Requirement

> "For direct data transfers between GPU memory and storage, the file must be opened in **O_DIRECT** mode. If the file is not opened in this mode, contents might be buffered in the CPU system memory, which is incompatible with direct transfers."  
> **Source:** [NVIDIA GDS Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

**Exception:** Starting from **CUDA Toolkit 12.2 (GDS 1.7.x)**, cuFile automatically takes the GDS-driven `O_DIRECT` path for **page-aligned buffers with aligned sizes and offsets**, even if the file was opened in non-`O_DIRECT` mode.

### Supported Storage Platforms

- Local NVMe drives
- Lustre, BeeGFS, GPFS, WekaFS
- VAST Data, NetApp ONTAP
- RDMA-enabled NFS

> **Source:** [EOFS Workshop 2024 — Performance Characteristics of GDS](https://hps.vi4io.org/_media/events/2024/mst3.pdf)

---

## GPUDirect P2P

### What It Does

GPUDirect P2P enables **direct peer-to-peer memory access between GPUs within the same node** — bypassing CPU system memory via PCIe or NVLink.

> "P2P_DIRECT provides substantial performance benefits through both simplified memory addressing and a more direct data transfer path."  > **Source:** [arXiv:2507.04786v3 — GPU Communication Protocols](https://arxiv.org/html/2507.04786v3)

### Technical Basis

- GPUs map each other's memory into their own address space
- Direct loads/stores between GPU memories
- Works over **PCIe** or **NVLink**

### Enabling P2P

```c
int canAccessPeer;
cudaDeviceCanAccessPeer(&canAccessPeer, dev0, dev1);
if (canAccessPeer) {
    cudaDeviceEnablePeerAccess(dev1, 0);
}
```

### Limitations

- Only works between GPUs that share a compatible PCIe topology (same root complex or switch)
- NVLink provides higher bandwidth than PCIe P2P
- Must be enabled per-process

---

## Software Stack

### Required Components

| Component | Purpose |
|-----------|---------|
| **nvidia.ko** | NVIDIA GPU driver |
| **nvidia-fs.ko** (GDS) | GPUDirect Storage kernel driver |
| **nvidia_p2p.ko** (RDMA) | Peer memory access driver |
| **MLNX_OFED / MOFED** | Mellanox/NVIDIA RDMA stack |
| **CUDA Toolkit** | cuFile API, GPUDirect APIs |
| **NCCL** | Collective ops leveraging GPUDirect |

### Linux Kernel Support

- **Linux kernel 6.2+** adds PCI P2PDMA support, enabling `ZONE_DEVICE` address pointers to GPU memory to be passed through VFS without page faults

> "Linux has added support for PCI P2PDMA in Linux kernel version 6.2 and above kernels, enabling ZONE_DEVICE address pointers to GPU memory to be passed through the VFS without causing a page fault."  > **Source:** [NVIDIA GDS Overview PDF](https://docs.nvidia.com/gpudirect-storage/pdf/overview-guide.pdf)

---

## Performance Benchmarks

### Oracle Cloud H100 Cluster

| Metric | Value |
|--------|-------|
| **RDMA NIC bandwidth (400 Gbps)** | 48.77 GB/s |
| **GPU to GPU Latency over RDMA** | 29 μs |
| **NCCL on single node (8 GPUs)** | 476.37 GB/s |

> **Source:** [Oracle Cloud Infrastructure Blog](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)

### H200 vs H100 NCCL AllReduce (Multi-node)

| GPU | Estimated AllReduce (Multi-node) |
|-----|----------------------------------|
| **H100** | ~950 GB/s |
| **H200** | **>1.4 TB/s** |

> "An estimated AllReduce performance of over **1.4 TB/s (multi-node) for the H200**, compared to ~950 GB/s for the H100."  > **Source:** [UVATION — H200 and NCCL](https://uvation.com/articles/why-nvidia-h200-and-nccl-are-reshaping-ai-training-efficiency-at-scale)

### GPUDirect Storage: Local NVMe (Grete+ Cluster)

| Metric | CPU POSIX | CUDA | cuFile (GDS) |
|--------|-----------|------|--------------|
| **4k rand read** (IOPS) | 934,276 | 349,722 | 828,364 |
| **4M seq read** (MiB/s) | 5,336 | 5,325 | **5,311** |
| **4k rand write** (IOPS) | 348,002 | 138,581 | 171,072 |

**Hardware:** AMD Epyc Milan, A100 GPUs, Intel P4510 NVMe  
> **Source:** [EOFS Workshop 2024](https://hps.vi4io.org/_media/events/2024/mst3.pdf)

### GPUDirect Storage: Lustre 2-Node (Grete+ Cluster)

| Metric | CPU POSIX | CUDA | cuFile (GDS) |
|--------|-----------|------|--------------|
| **4k rand read** (IOPS) | 2,190,572 | 497,150 | **2,215,756** |
| **4M seq read** (MiB/s) | 45,988 | 26,347 | **70,726** |
| **4k rand write** (IOPS) | 152,127 | 146,593 | 147,537 |

> "For large sequential reads (4M) across two nodes, **GDS significantly outperforms both CPU POSIX and CUDA I/O** (70.726 MiB/s vs. ~46–26 MiB/s)."  > **Source:** [EOFS Workshop 2024](https://hps.vi4io.org/_media/events/2024/mst3.pdf)

### Critical Finding: POSIX Fallback vs True GDS

| Metric | cuFile (POSIX fallback) | cuFile (True GDS) |
|--------|------------------------|-------------------|
| **4k rand read** (IOPS) | 1,933,847 | 2,215,756 |
| **512k rand read** (MiB/s) | 57,396 | 55,650 |
| **GPU utilization** | **>80%** | **<5%** |

> "Throughput is very similar, but **GPU utilization differs drastically** (>80% vs. <5%). This indicates that the value of true GDS is not always raw throughput, but **offloading I/O burden from the GPU**."  > **Source:** [EOFS Workshop 2024](https://hps.vi4io.org/_media/events/2024/mst3.pdf)

### Micron 9400 + GDS Benchmark (DGX A100)

| Condition | Max Gain |
|-----------|----------|
| **GDS vs Legacy IO** (busy system, 4KB) | **6.4x higher performance**, **7.3x better response time** |
| **GDS vs Legacy IO** (1MB) | **1.5x higher performance** |
| **Micron 9400 vs Competitor** (4KB GDS) | **42% higher performance**, **40% better response time** |

> "Enabling GDS (blue) shows up to **6.4x higher performance** and **7.3x better response time** than the legacy IO configuration."  > — [Micron White Paper](https://assets.micron.com/adobe/assets/urn:aaid:aem:02305c93-ce98-4d43-a640-8739f748a808/renditions/original/as/micron-9400-nvidia-gds-vs-comp-white-paper.pdf)

### Storage Vendor GDS Rankings (HPCwire 2023)

**Sequential Read Bandwidth:**

| Rank | Vendor | Read Bandwidth |
|------|--------|----------------|
| 1 | **DDN** | **90 GBps** |
| 2 | VAST Data | — |
| 3 | IBM ESS3200 | — |

**Sequential Write Bandwidth:**

| Rank | Vendor | Write Bandwidth |
|------|--------|----------------|
| 1 | **DDN** | **65 GBps** |
| 2 | IBM | — |

> "DDN is the only vendor in the survey that delivered both fast reads and fast writes."  > **Source:** [HPCwire](https://www.hpcwire.com/2023/08/02/data-io-performance-using-nvidia-gpudirect-storage/)

---

## Supported Hardware

### GPUs

| GPU | GPUDirect RDMA | GPUDirect Storage | GPUDirect P2P |
|-----|---------------|-------------------|---------------|
| **A100** | ✅ | ✅ | ✅ (NVLink/PCIe) |
| **H100** | ✅ | ✅ | ✅ (NVLink 3.0) |
| **H200** | ✅ | ✅ | ✅ (NVLink 3.0) |
| **B100** | ✅ | ✅ | ✅ (NVLink 4.0) |
| **B200** | ✅ | ✅ | ✅ (NVLink 4.0) |
| **DGX Spark / GB10** | ⚠️ Some limitations | — | — |

> "NCCL/RoCEv2 Issues with Duplicates, GPU Direct RDMA, and Fusion — DGX Spark / GB10"  > **Source:** [NVIDIA Developer Forums](https://forums.developer.nvidia.com/t/nccl-rocev2-issues-with-duplicates-gpu-direct-rdma-and-fusion/351460)

### NICs

- **Mellanox ConnectX-4+** — minimum for GPUDirect RDMA
- **Mellanox ConnectX-7** — 400 Gbps, full feature set
- **NVIDIA BlueField-3 DPU** — embedded RDMA

### NVMe Requirements (GDS)

- NVMe drives on the **same PCIe switch or root complex** as the GPU
- DMA-capable storage controllers

---

## Integration with NCCL

NCCL (NVIDIA Collective Communications Library) leverages all three GPUDirect technologies:

| GPUDirect Type | NCCL Usage |
|----------------|-----------|
| **P2P** | Intra-node collectives (AllReduce, Broadcast within a node) |
| **RDMA** | Inter-node collectives over RoCEv2 or InfiniBand |
| **GDS** | Checkpoint loading, dataset streaming |

> "NCCL (NVIDIA Collective Communications Library) serves as the 'glue' that synchronises and orchestrates data movement in multi-GPU and multi-node AI training."  > **Source:** [UVATION — H200 and NCCL](https://uvation.com/articles/why-nvidia-h200-and-nccl-are-reshaping-ai-training-efficiency-at-scale)

### GPU-Initiated Networking (Emerging)

> "The GPUDirect Async Kernel-Initiated backend leverages DOCA GPUNetIO for direct GPU-to-NIC communication, while the Proxy backend provides ..."  > **Source:** [ResearchGate — GPU-Initiated Networking for NCCL](https://www.researchgate.net/publication/397780825_GPU-Initiated_Networking_for_NCCL)

---

## Compatibility Mode & Fallback

When a direct path is unavailable, cuFile seamlessly stages through CPU system memory.

### Triggers for Fallback

- Explicit config via user `cufile.json`
- Missing `nvidia-fs.ko` kernel driver
- Filesystem mount does not support GDS
- `O_DIRECT` cannot be applied
- Unaligned buffers, offsets, or sizes
- IO request too small
- Transfer exceeds GPU BAR1 aperture
- Intermediate staging required (e.g., NVLink hop)

> "Performance in compatibility mode is generally at least the same or better than standard CPU-based POSIX APIs."  > **Source:** [NVIDIA GDS Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

### Silent POSIX Fallback Warning

> ```
> [pid=21356] NOTICE cufio:1538 cuFileHandleRegister GDS not supported or disabled by config, using cuFile posix read/write with compat mode enabled
> ```
> "**Implication:** GDS can silently fall back to POSIX compatibility mode. Users may believe they are running GDS when they are actually executing POSIX I/O through the cuFile API wrapper."  > **Source:** [EOFS Workshop 2024](https://hps.vi4io.org/_media/events/2024/mst3.pdf)

---

## Key Repositories

| Repository | Description |
|------------|-------------|
| [NVIDIA/gds-nvidia-fs](https://github.com/NVIDIA/gds-nvidia-fs) | GPUDirect Storage kernel driver (`nvidia-fs.ko`) — 340 stars, 55 forks |
| [Mellanox/gpu_direct_rdma_access](https://github.com/Mellanox/gpu_direct_rdma_access) | Mellanox DC QP RDMA Read/Write to GPU memory |
| [RWTH-ACS/infiniGPUdirect](https://github.com/RWTH-ACS/infiniGPUdirect) | Infiniband GPUDirect RDMA benchmarking tool |
| [DolphinICS/cuda-rdma-bench](https://github.com/DolphinICS/cuda-rdma-bench) | SISCI + GPUDirect RDMA bandwidth benchmark |
| [gpudirect/nv_peer_memory](https://github.com/gpudirect/nv_peer_memory) | Peer memory client for IB CORE |

---

## References

- [NVIDIA GPUDirect Storage Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)
- [NVIDIA GPUDirect Storage Configuration / Benchmarking Guide](https://docs.nvidia.com/gpudirect-storage/configuration-guide/index.html)
- [NVIDIA GDS Overview PDF](https://docs.nvidia.com/gpudirect-storage/pdf/overview-guide.pdf)
- [NVIDIA GPUDirect RDMA Docs](https://docs.nvidia.com/cuda/gpudirect-rdma/)
- [NVIDIA Developer Forums — GPUDirect Relationships](https://forums.developer.nvidia.com/t/the-relationship-between-gpudirect-rdma-gpudirect-p2p-nvidia-ipc-nccl-and-nvshmem/316874)
- [Oracle OCI — H100 Benchmarks](https://blogs.oracle.com/cloud-infrastructure/zettascale-osu-nccl-benchmark-h100-ai-workloads)
- [UVATION — H200 + NCCL](https://uvation.com/articles/why-nvidia-h200-and-nccl-are-reshaping-ai-training-efficiency-at-scale)
- [EOFS Workshop 2024 — GDS Performance](https://hps.vi4io.org/_media/events/2024/mst3.pdf)
- [Micron 9400 + GDS White Paper](https://assets.micron.com/adobe/assets/urn:aaid:aem:02305c93-ce98-4d43-a640-8739f748a808/renditions/original/as/micron-9400-nvidia-gds-vs-comp-white-paper.pdf)
- [HPCwire — GDS Vendor Comparison](https://www.hpcwire.com/2023/08/02/data-io-performance-using-nvidia-gpudirect-storage/)
- [NVIDIA gds-nvidia-fs GitHub](https://github.com/NVIDIA/gds-nvidia-fs)
- [vLLM Discuss — GPUDirect RDMA Benchmarking](https://discuss.vllm.ai/t/vllm-benchmarking-why-is-gpudirect-rdma-not-outperforming-standard-rdma-in-a-pipeline-parallel-setup/1377)
- [NVIDIA Developer Forums — DGX Spark GPUDirect](https://forums.developer.nvidia.com/t/nccl-rocev2-issues-with-duplicates-gpu-direct-rdma-and-fusion/351460)

---

*Last updated: 2026-05-02*
