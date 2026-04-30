# NVMe-oF (NVMe over Fabrics) for ML Training Infrastructure

> **Note**: This document covers NVMe over Fabrics (NVMe-oF) as high-performance shared storage for ML training workloads. For local NVMe SSD selection and benchmarking, see the local storage research. NVMe-oF extends NVMe semantics to network fabrics, enabling remote flash with near-local latency.

## Table of Contents

- [Overview](#overview)
- [Protocol Specification: RDMA vs TCP](#protocol-specification-rdma-vs-tcp)
- [NVIDIA GPUDirect Storage (GDS)](#nvidia-gpudirect-storage-gds)
- [Filesystem Semantics vs Object Semantics](#filesystem-semantics-vs-object-semantics)
- [Block Device vs File Volume in Kubernetes](#block-device-vs-file-volume-in-kubernetes)
- [CSI Drivers for NVMe-oF](#csi-drivers-for-nvme-of)
- [Performance Benchmarks for Training Workloads](#performance-benchmarks-for-training-workloads)
- [Comparison: Local NVMe, NFS, Lustre](#comparison-local-nvme-nfs-lustre)
- [References](#references)

---

## Overview

NVMe over Fabrics (NVMe-oF) extends NVMe command semantics to networked storage, enabling remote NVMe devices to appear as locally attached drives.

**Source:** [NVM Express - NVMe over Fabrics Specification](https://nvmexpress.org/specification/nvme-of-specification/)

> "The NVMe® over Fabrics (NVMe-oF™) specification was created to enable non-volatile memory express commands that transfer data between a host and SSD or system over a networked fabric. The first NVMe-oF 1.0 specification was released in June 2016 and extended NVMe technology to additional transports beyond PCIe, such as Ethernet, Fibre Channel, Infiniband and RDMA."

**Source:** [NetApp NVMe vs NVMe-oF Comparison](https://www.netapp.com/learn/cvo-blg-nvme-vs-nvme-of-a-comparison/)

> "NVMe-oF is an augmented protocol extending NVMe performance over Fibre and Ethernet channels. Uses NVMe commands to exchange data between media channels over a fabric network, connecting storage controller arrays while reducing CPU utilization."

**Spec evolution note:** NVMe-oF 1.0 (2016) and 1.1 (2019) specs are now historical. NVMe-oF content migrated to NVMe 2.0 Base specification.

---

## Protocol Specification: RDMA vs TCP

### Transport Types

| Transport | Typical Latency | Hardware Requirements | Implementation Complexity |
|-----------|----------------|----------------------|--------------------------|
| **RDMA (InfiniBand)** | <10 microseconds | Specialized hardware | High |
| **RoCE v2 (RDMA over Converged Ethernet)** | 15-30 microseconds | DCB switches, PFC/ECN | High |
| **NVMe/TCP** | 50-100 microseconds | Standard Ethernet | Low |

**Source:** [NVMe-oF Transport Selection: RDMA, RoCE, and TCP Latency Budgets](https://eureka.patsnap.com/report-nvme-of-transport-selection-rdma-roce-and-tcp-latency-budgets)

### NVMe/RDMA (RoCE and InfiniBand)

**Source:** [Intelligent Visibility - NVMe-oF TCP vs RDMA](https://intelligentvisibility.com/nvme-over-fabrics-ethernet-comparison)

> "NVMe/RoCE: Remote Direct Memory Access (RDMA) bypasses OS network stack for direct memory-to-memory transfers. RoCEv2 operates over UDP/IP (routable). Benefits: Ultra-low latency (sub-10 to tens of microseconds), high throughput with efficient data transfers, significantly reduced CPU utilization."

**Source:** [Murat Karslioglu - NVMe-oF: The Promise, The Pain, and What Actually Works in 2026](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

> "NVMe/TCP adds 30-80 microseconds of latency, depending on network conditions, CPU load, and how many TCP connections you're multiplexing."

> "InfiniBand: Adds 2-5 μs latency. Purpose-built fabric (ConnectX adapters, Quantum switches). Dominant in HPC/AI clusters."

> "RoCEv2: Adds 5-10 μs latency. Uses standard Ethernet switches. **Requires lossless fabric:** PFC (Priority Flow Control), ECN (Explicit Congestion Notification). The dirty secret of RoCEv2: Works flawlessly in controlled single-rack environments. Budget six months for a 500-node multi-tier cluster."

### NVMe/TCP

**Source:** [Intelligent Visibility - NVMe-oF TCP vs RDMA](https://intelligentvisibility.com/nvme-over-fabrics-ethernet-comparison)

> "NVMe/TCP: NVMe commands encapsulated within TCP packets over standard Ethernet. Benefits: Broad compatibility with existing Ethernet hardware, no special network configurations (DCB/RDMA) required, routable across complex topologies and WANs."

> "NVMe/TCP's inherent latency from the TCP stack is generally higher and may not match the ultra-low latency achievable with RDMA-based solutions like RoCE."

**Source:** [Murat Karslioglu - NVMe-oF: The Promise, The Pain, and What Actually Works in 2026](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

> "NVMe/TCP adds 30-80 μs latency overhead. CPU cost: TCP processing consumes host cycles at high IOPS (500K+). TCP transport performance is particularly affected, as both the NVMe target processing and TCP stack run in kernel context, competing for CPU."

### Performance Comparison

**Source:** [StarWind - iSCSI vs NVMe-oF Performance Comparison](https://www.starwindsoftware.com/blog/iscsi-vs-nvme-of-performance-comparison/) (April 2024)

| Metric | NVMe-oF RDMA | NVMe-oF TCP | iSCSI |
|--------|-------------|-------------|-------|
| **4K Read IOPS** | 1,558,000 | 1,503,000 | 1,272,000 |
| **4K Read Latency** | 0.082 ms | 0.382 ms | 0.806 ms |
| **4K Read CPU** | 7% | 19% | 47% |
| **4K Write Latency** | 0.055 ms | 0.260 ms | 0.906 ms |
| **4K Write CPU** | 5% | 12% | 42% |

> "NVMe-oF over RDMA demonstrates approximately 88% lower CPU usage compared to iSCSI and approximately 58% lower CPU usage compared to NVMe-oF over TCP."

### Latency by Ethernet Speed

**Source:** [Murat Karslioglu](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

| Ethernet Speed | NVMe/TCP 4KB (sw) | NVMe/TCP 4KB (offload) | NVMe/RDMA |
|---------------|------------------|------------------------|-----------|
| 25 GbE | ~60-80 μs | ~35-50 μs | ~10-15 μs |
| 100 GbE | ~40-60 μs | ~20-35 μs | ~8-12 μs |
| 200 GbE | ~30-50 μs | ~15-25 μs | ~7-10 μs |
| 400 GbE | ~25-40 μs | ~10-20 μs | ~5-8 μs |

---

## NVIDIA GPUDirect Storage (GDS)

**Source:** [NVIDIA GPUDirect Storage Overview](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

> "GPUDirect Storage creates a direct data path between storage and GPU memory and avoids extra copies through a bounce buffer in the CPU's memory."

> "GDS enables a direct data path for direct memory access (DMA) transfers between GPU memory and storage, which avoids a bounce buffer through This direct path increases system bandwidth and decreases the latency and utilization load on the CPU."

### Key Capabilities

**Source:** [NVIDIA GPUDirect Storage Overview](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

| Benefit | Description |
|---------|-------------|
| **Direct Path** | Enables direct DMA transfers between GPU memory and storage |
| **Reduced CPU Load** | Relieves bandwidth bottlenecks and reduces latency |
| **Performance Multiplier** | GPU has first/last touch of data in fully GPU-migrated pipelines |
| **Generic Storage Support** | Works across local/distributed file systems, block interfaces, NFS, Lustre, NVMe-oF |

### GDS + NVMe-oF

**Source:** [NVIDIA GPUDirect Storage Overview](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

> "Starting from CUDA Toolkit 12.2 (GDS version 1.7.x) even for files opened in non-O_DIRECT mode, the cuFile library takes the GDS driven O_DIRECT path for transfers between GPU memory and storage for page aligned buffers with aligned sizes and offsets."

> "NVIDIA's initial implementations for cuFile focus on distributed filesystems and systems where appropriate drivers have been installed to enable a direct transfer between storage and GPU memory without using a bounce buffer in the CPU."

### GDS Benchmarking

**Source:** [NVIDIA GPUDirect Storage Benchmarking and Configuration Guide](https://docs.nvidia.com/gpudirect-storage/configuration-guide/index.html)

> "GDS enables high throughput and low latency data transfer between storage and GPU memory, which allows you to program the DMA engine of a PCIe device with the correct mappings to move data in and out of a target GPU's memory."

### cuFile API

**Source:** [NVIDIA GPUDirect Storage Overview](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)

> "cuFile API is the primary API for file I/O independent of memory type for GPU applications. Best-performing access to local or distributed file/block storage. Increased performance and lower CPU load vs standard Linux file IO."

**Source:** [Xinnor Blog - xiRAID MLPerf Validation](https://xinnor.io/blog/validating-xiraid-performance-for-ai-workloads-high-throughput-storage-testing-with-mlperf-benchmarks/)

> "One of the primary challenges in designing high-performance systems for AI and ML workloads is creating a storage subsystem that can deliver consistently high throughput without becoming a bottleneck during large-scale training or checkpointing operations."

---

## Filesystem Semantics vs Object Semantics

### Filesystem Semantics (POSIX)

**Source:** [Lustre Wiki - NFS vs Lustre](https://wiki.lustre.org/NFS_vs._Lustre)

> "NFS4 implementations are able to handle Posix advisory locks, but unlike Lustre, they don't support full Posix filesystem semantics. With Lustre, individual write()s are atomic and becomes immediately visible to all clients."

> "NFS4 still follows the traditional NFS close-to-open cache consistency model whereas with Lustre, individual write()s are atomic and become immediately visible to all clients."

### Object Semantics

**Source:** [ApX ML - Object Storage Semantics](https://apxml.com/courses/intro-data-lake-architectures/chapter-1-architectural-foundations/object-storage-semantics)

> "The most significant difference between a file system and object storage is the absence of a true directory hierarchy."

### Key Differences for ML Training

| Aspect | Filesystem (POSIX) | Object Storage |
|--------|-------------------|----------------|
| **Consistency** | Strong (write-after-close visible) | Eventual |
| **Atomicity** | Per-write atomic | Per-object atomic |
| **Latency** | Lower for small I/O | Higher per-object overhead |
| **Scalability** | Limited by namespace design | Designed for massive scale |
| **Use Case** | Checkpoints, code, config | Dataset archives, model artifacts |

### NVMe-oF Position

NVMe-oF exposes block devices that can be formatted with any filesystem (XFS, ext4, etc.), preserving POSIX semantics. This makes it suitable for:
- Training checkpoint saves/loads (atomic writes, POSIX compatibility)
- Code and dependency storage
- Scratch workspaces

Object storage (S3, minio) is better for:
- Model artifact registries
- Dataset versioning
- Cross-region replication

---

## Block Device vs File Volume in Kubernetes

### Volume Mode Comparison

**Source:** [Stack Overflow - Kubernetes Block, File, Object](https://stackoverflow.com/questions/65868661/how-kubernetes-block-file-and-object-storage-types-are-used-inside-containers)

> "Block storage is Read-Write Only (RWO) which means it can be mounted read-write only to single pod. NFS is Read-Write Many (RWX) which means it can be mounted read-write to many pods."

> "The main thing about block storage is that in most cases it's Read-Write Only (RWO) which means it can be mounted read-write only to single pod."

### Block Volumes

- Raw device access, no filesystem overhead
- Maximum performance for database workloads
- RWO (single pod write access)
- Requires filesystem formatting after provision
- NVMe-oF typically exposed as block device

### File Volumes

- NFS, CIFS, or filesystem-backed
- RWX (multi-pod read-write)
- Shared storage use cases
- Includes filesystem metadata operations

### CSI Implications

**Source:** [Rook Ceph - NVMe-oF Block Storage](https://rook.io/docs/rook/latest-release/Storage-Configuration/Block-Storage-RBD/nvme-of/)

> "This PVC is created for CSI driver provisioning. The volume will be accessible via NVMe-oF protocol by both Kubernetes pods within the cluster and external hosts."

---

## CSI Drivers for NVMe-oF

### SPDK CSI Driver

**Source:** [GitHub - spdk/spdk-csi](https://github.com/spdk/spdk-csi)

> "SPDK CSI plugin brings SPDK to Kubernetes storage by enabling Pods to access storage provisioned on an SPDK NVMe-oF target. CSI driver to bring SPDK to Kubernetes storage through NVMe-oF or iSCSI. Supports dynamic volume provisioning and enables Pods to use SPDK storage transparently."

| Component | Version |
|-----------|---------|
| CSI Spec | v1.7.0 |
| Architectures | x86_64, Arm64 |
| Kubernetes | 1.25.0 (minimal: 1.20+) |
| Status | **Beta** |

**Source:** [SPDK Documentation - NVMe over Fabrics Target](https://spdk.io/doc/nvmf.html)

> "The SPDK NVMe over Fabrics target is a user space application that presents block devices over a fabrics such as Ethernet, Infiniband or Fibre Channel. SPDK currently supports RDMA and TCP transports."

### Simplyblock CSI Driver

**Source:** [GitHub - simplyblock/simplyblock-csi](https://github.com/simplyblock/simplyblock-csi)

> "Simplyblock's CSI driver enables enterprise-grade, NVMe/TCP-powered block storage directly inside Kubernetes, offering high performance, scalability, and"

### HPE CSI Driver with NVMe/TCP

**Source:** [HPE Community](https://community.hpe.com/t5/around-the-storage-block/introducing-nvme-tcp-for-hpe-alletra-storage-mp-b10000-with-hpe/ba-p/7262519)

> "In the era of cloud native and twelve factor apps, deployments are getting smaller while the compute layer is growing with more cores and larger memory footprint with specialized hardware such a GPUs attached This in turn means that more connections for storage resources must be made over larger more complex and faster networks."

### Huawei NVMe-oF CSI

**Source:** [Eureka Report - NVMe-oF in Kubernetes](https://eureka.patsnap.com/report-nvme-of-in-kubernetes-csi-integration-node-affinity-and-failure-domains)

> "Implementation of Container Storage Interface (CSI) drivers for NVMe over Fabrics (NVMe-oF) to enable containerized applications to utilize high-performance NVMe storage in Kubernetes environments."

### Rook Ceph NVMe-oF

**Source:** [Rook Ceph - NVMe-oF](https://rook.io/docs/rook/latest-release/Storage-Configuration/Block-Storage-RBD/nvme-of/)

> "This PVC is created for CSI driver provisioning. The volume will be accessible via NVMe-oF protocol by both Kubernetes pods within the cluster and external hosts."

---

## Performance Benchmarks for Training Workloads

### MLPerf Storage Benchmark

**Source:** [MLCommons - MLPerf Storage Benchmark v2.0](https://www.blocksandfiles.com/ai-ml/2025/08/05/arrays-speed-up-for-mlperf-storage-benchmark-v20/101034)

> "The second edition of the MLPerf Storage benchmark shows tested systems serving roughly twice the number of accelerators than in the 2023 v1.0 benchmark round."

> "At the scale of computation being implemented for training large AI models, regular component failures are simply a fact of life. Checkpointing is now a standard practice in these systems to mitigate failures, and we are proud to be providing critical benchmark data on storage systems to allow stakeholders to optimize their training performance."

### Alluxio MLPerf v2.0 Results

**Source:** [Alluxio Blog - MLPerf Storage v2.0](https://www.alluxio.io/blog/alluxio-demonstrates-strong-performance-in-mlperf-storage-v2-0-benchmarks)

> "In the latest MLPerf Storage v2.0 benchmarks, Alluxio demonstrated how distributed caching accelerates I/O for AI training and checkpointing workloads, achieving up to 99.57% GPU utilization across multiple workloads that typically suffer from underutilized GPU resources caused by I/O bottlenecks."

> "The true challenge in AI training infrastructure isn't raw computing power: it's ensuring your expensive GPUs aren't stalled by slow data loading during training epochs or lengthy checkpoint operations during model saves."

### Local NVMe Baseline

**Source:** [Simplyblock - NVMe Latency](https://simplyblock.io/glossary/nvme-latency/)

> "Modern NVMe devices achieve random read latency around 20–70 microseconds at the device level, versus 100–200 μs for SATA SSDs and 5–10 ms for HDDs."

### NVMe-oF vs Local NVMe (StarWind)

**Source:** [StarWind - iSCSI vs NVMe-oF Performance Comparison](https://www.starwindsoftware.com/blog/iscsi-vs-nvme-of-performance-comparison/)

| Pattern | Local IOPS | NVMe-oF RDMA IOPS | Performance % |
|---------|-----------|-------------------|---------------|
| 4K Random Read | 1,552,000 | 1,558,000 | **100.4%** |
| 4K Random Write | 1,160,000 | 1,143,000 | **98.5%** |

> "NVMe-oF over RDMA is the clear winner for high-performance storage applications demanding the absolute best speed and efficiency."

---

## Comparison: Local NVMe, NFS, Lustre

### Local NVMe vs Remote NVMe-oF

**Source:** [StarWind Benchmark](https://www.starwindsoftware.com/blog/iscsi-vs-nvme-of-performance-comparison/)

| Metric | Local NVMe | NVMe-oF RDMA | NVMe-oF TCP |
|--------|-----------|-------------|-------------|
| 4K Read Latency | 0.015 ms | 0.082 ms | 0.382 ms |
| 4K Write Latency | 0.020 ms | 0.055 ms | 0.260 ms |
| CPU Usage (Read) | 1-4% | 7% | 19% |

### NFS vs Lustre

**Source:** [Lustre Wiki - NFS vs Lustre](https://wiki.lustre.org/NFS_vs._Lustre)

| Factor | NFS | Lustre |
|--------|-----|--------|
| **Setup/Maintenance** | Far easier | More complex |
| **Scalability** | Does NOT scale linearly | Scales linearly with hardware |
| **I/O Path** | In-band (metadata + data together) | 3rd-party transfer (data bypasses MDS) |
| **POSIX Semantics** | Incomplete (no O_APPEND, close-to-open consistency) | Full (individual write()s atomic) |
| **Cache Coherency** | Close-to-open model | Immediate visibility |
| **Portability** | Universal (built into everything) | Always add-on |

> "NFS does NOT scale: It's still an in-band protocol. The data is carried in the same message as the request and is, practically, limited in size."

> "Lustre DOES scale: Uses 3rd-party transfer (data moves directly between storage and client). More storage components = less contention = more throughput."

### Performance Summary for ML Training

| Storage Type | Checkpoint Write | Checkpoint Read | GPUDirect Storage | Scalability |
|-------------|------------------|-----------------|-------------------|-------------|
| **Local NVMe** | Fastest (~15 μs) | Fastest (~10 μs) | Via GDS | None (single node) |
| **NVMe-oF RDMA** | ~15-20 μs | ~10-15 μs | Native GDS | Excellent (fabric) |
| **NVMe-oF TCP** | ~40-80 μs | ~30-50 μs | Limited | Good |
| **NFS** | Higher overhead | Higher overhead | No native support | Limited |
| **Lustre** | Good | Good | Via DDN EXAScaler | Excellent |

### When to Use NVMe-oF for ML Training

**Use NVMe-oF when:**
- Multi-node training requires shared checkpoint storage
- GPU cluster elasticity requires persistent storage across node restarts
- Checkpoint size exceeds single-node NVMe capacity
- Fault tolerance requires replicas across storage nodes

**Prefer local NVMe when:**
- Single-node training (no checkpoint sharing needed)
- Maximum latency sensitivity (no network hop)
- Cost sensitivity (no network infrastructure needed)

### Network Requirements for AI Clusters

**Source:** [Murat Karslioglu](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)

> "This is the most compelling NVMe-oF deployment model in 2026: AI clusters where InfiniBand is a given, BlueField handles the storage fabric, and the performance requirements (feeding 8x H100/B200 GPUs with training data) justify the infrastructure investment."

> "Running a disaggregated NVMe-oF fabric requires expertise in NVMe target management, fabric zoning, multipath configuration, timeout tuning, performance monitoring."

---

## References

### Official Specifications

- [NVMe over Fabrics (oF) Specification (Historical)](https://nvmexpress.org/specification/nvme-of-specification/)
- [NVMe over RDMA Transport Specification](https://nvmexpress.org/specification/rdma-transport-specification/)
- [NVMe over TCP Transport Specification](https://nvmexpress.org/specification/tcp-transport-specification/)

### NVIDIA Documentation

- [GPUDirect Storage Overview Guide](https://docs.nvidia.com/gpudirect-storage/overview-guide/index.html)
- [GPUDirect Storage Benchmarking Guide](https://docs.nvidia.com/gpudirect-storage/configuration-guide/index.html)
- [Magnum IO GPUDirect Storage](https://developer.nvidia.com/gpudirect-storage)
- [GDS PDF Documentation](https://docs.nvidia.com/cuda/archive/11.4.2/pdf/GDS.pdf)

### Implementations

- [SPDK NVMe-oF Target](https://spdk.io/doc/nvmf.html)
- [SPDK CSI Driver](https://github.com/spdk/spdk-csi)
- [Simplyblock CSI Driver](https://github.com/simplyblock/simplyblock-csi)
- [Rook Ceph NVMe-oF](https://rook.io/docs/rook/latest-release/Storage-Configuration/Block-Storage-RBD/nvme-of/)

### Performance & Benchmarks

- [StarWind - iSCSI vs NVMe-oF Performance Comparison](https://www.starwindsoftware.com/blog/iscsi-vs-nvme-of-performance-comparison/)
- [MLCommons MLPerf Storage v2.0](https://www.blocksandfiles.com/ai-ml/2025/08/05/arrays-speed-up-for-mlperf-storage-benchmark-v20/101034)
- [Alluxio MLPerf Storage v2.0 Results](https://www.alluxio.io/blog/alluxio-demonstrates-strong-performance-in-mlperf-storage-v2-0-benchmarks)
- [Xinnor - xiRAID MLPerf Validation](https://xinnor.io/blog/validating-xiraid-performance-for-ai-workloads-high-throughput-storage-testing-with-mlperf-benchmarks/)

### Analysis & Technical Reports

- [Murat Karslioglu - NVMe-oF: The Promise, The Pain, and What Actually Works in 2026](https://muratkarslioglu.com/blog/nvme-of-promise-and-pain/)
- [Intelligent Visibility - NVMe-oF TCP vs RDMA](https://intelligentvisibility.com/nvme-over-fabrics-ethernet-comparison)
- [NVMe-oF Transport Selection: RDMA, RoCE, TCP Latency Budgets](https://eureka.patsnap.com/report-nvme-of-transport-selection-rdma-roce-and-tcp-latency-budgets)
- [NetApp - NVMe vs NVMe-oF Comparison](https://www.netapp.com/learn/cvo-blg-nvme-vs-nvme-of-a-comparison/)
- [Lustre Wiki - NFS vs Lustre](https://wiki.lustre.org/NFS_vs._Lustre)

### Blog Posts

- [NVIDIA Blog - Accelerating IO](https://developer.nvidia.com/blog/accelerating-io-in-the-modern-data-center-magnum-io-storage-partnerships/)
- [Xinnor - Lustre on NVMe-oF](https://xinnor.io/blog/running-lustre-on-disaggregated-storage-with-xiraid/)
- [DataCore - Breaking Storage Bottlenecks with NVMe-oF](https://www.datacore.com/blog/breaking-storage-bottlenecks-with-nvme-of/)
