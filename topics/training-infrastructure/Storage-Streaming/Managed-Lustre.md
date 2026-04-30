# Managed Lustre for ML Training

> **Last updated:** April 2026  
> **Scope:** Cloud-managed and on-premises Lustre parallel filesystem for ML training workloads

---

## Table of Contents

1. [Overview](#overview)
2. [Cloud Providers](#cloud-providers)
   - [AWS FSx for Lustre](#aws-fsx-for-lustre)
   - [Google Cloud Managed Lustre](#google-cloud-managed-lustre)
   - [Azure Managed Lustre](#azure-managed-lustre)
3. [On-Premises Lustre](#on-premises-lustre)
4. [Lustre Architecture](#lustre-architecture)
5. [ML Training Performance](#ml-training-performance)
6. [Checkpoint Strategies](#checkpoint-strategies)
7. [Comparison: Lustre vs Object Storage](#comparison-lustre-vs-object-storage)
8. [Kubernetes Integration](#kubernetes-integration)
9. [Key Benchmarks & Research](#key-benchmarks--research)
10. [Resources](#resources)

---

## Overview

**Lustre** is an open-source, global single-namespace, POSIX-compliant, distributed parallel file system designed for scalability, high-performance, and high-availability.

> "Lustre is an open-source, distributed parallel file system licensed under the GPL 2.0 license for use with Linux."  
> — [Lustre Development](https://www.lustre.org/development/)

### Key Specifications

| Parameter | Value |
|-----------|-------|
| **Max filesystem size** | Up to 8EB (ZFS backend) |
| **Max throughput** | Terabytes per second |
| **Max clients** | Thousands |
| **Network support** | Ethernet, InfiniBand, Omni-Path |
| ** POSIX compliant** | Yes |

> "Lustre servers for a single file system instance can, in aggregate, present up to hundreds of petabytes of storage to thousands of compute clients, with multiple terabytes per second of combined throughput."  
> — [Lustre Wiki - Introduction](https://wiki.lustre.org/Introduction_to_Lustre)

---

## Cloud Providers

### AWS FSx for Lustre

**Product page:** [Amazon FSx for Lustre](https://aws.amazon.com/fsx/lustre/)

#### Overview

> "Amazon FSx for Lustre is fully managed, shared storage built on the world's most popular high-performance file system."  
> — [AWS](https://aws.amazon.com/fsx/lustre/)

#### Performance

> "FSx for Lustre provides the fastest storage performance for GPU instances in the cloud with up to terabytes per second of throughput, millions of IOPS, sub-millisecond latencies, and 99.99% availability."  
> — [AWS](https://aws.amazon.com/fsx/lustre/)

##### Metadata Performance (Persistent 2)

| Operation Type | Operations per second per provisioned metadata IOPS |
|----------------|---------------------------------------------------|
| File Create, Open, Close | 2 |
| File Delete | 1 |
| Directory Create, Rename | 0.1 |
| Directory Delete | 0.2 |

> Source: [FSx for Lustre Performance](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance.html)

##### Client Throughput Limits

| File System Type | Client Interface | Max Throughput/Client |
|-----------------|------------------|-----------------------|
| Non EFA-enabled | Any | 100 Gbps* |
| EFA-enabled | ENA | 100 Gbps* |
| EFA-enabled | ENA Express | 100 Gbps |
| EFA-enabled | EFA | 700 Gbps |
| EFA-enabled | EFA + GDS | 1,200 Gbps |

> *Individual client-to-storage-server traffic limited to **5 Gbps**

> "For file systems with **>10 GBps throughput**, enable **Elastic Fabric Adapter (EFA)**."  
> — [AWS Documentation](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance.html)

#### S3 Integration

> "FSx for Lustre is deeply integrated with Amazon S3. This integration means that you can seamlessly access the objects stored in your Amazon S3 buckets from applications mounting your Amazon FSx for Lustre file system."  
> — [AWS FSx CSI Driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver/blob/master/examples/kubernetes/dynamic_provisioning_s3/README.md)

#### Performance Tips

> "Because Amazon FSx for Lustre is a network file system, each file operation goes through a round trip between the client and Amazon FSx for Lustre, incurring a small latency overhead."  
> — [AWS Performance Tips](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance-tips.html)

> "Overall throughput increases as average I/O size increases because the per-operation latency overhead is amortized over larger amounts of data."

**Key recommendations:**
- Enable asynchronous writes for lower latencies
- Limit directories to less than 100K files for optimal metadata performance
- Stripe high-throughput files across all OSTs

> "By enabling asynchronous writes to your file system, pending write operations are buffered on the Amazon EC2 instance before they are written to Amazon FSx for Lustre asynchronously."  
> — [AWS Documentation](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance-tips.html)

#### File Striping

| File System Creation Date | Default Layout |
|---------------------------|----------------|
| Before Dec 18, 2020 | Stripe count = **1** |
| Dec 18, 2020 – Aug 24, 2023 | Files <1GiB: 1 stripe; Files ≥1GiB: **5 stripes** |
| After Aug 25, 2023 | **4-component progressive file layout** (PFL) |

> "S3-imported files use `ImportedFileChunkSize` parameter (default **1GiB**)"

---

### Google Cloud Managed Lustre

**Product page:** [Google Cloud Managed Lustre](https://cloud.google.com/products/managed-lustre)

#### Overview

> "Managed Lustre with Google Cloud is a fully managed parallel file system built on DDN Exascaler. The aim is to solve the demanding requirements of data preparation, model training, and inference in AI workloads."  
> — [Tech Field Day](https://techfieldday.com/video/intro-to-managed-lustre-with-google-cloud/)

> "Managed Lustre provides high throughput to keep GPUs and TPUs fully utilized and enables quick writing and reading for checkpoints."  
> — [Google Cloud](https://techfieldday.com/video/intro-to-managed-lustre-with-google-cloud/)

#### Specifications

| Parameter | Value |
|-----------|-------|
| **Scale** | 18 terabytes to petabyte scale |
| **Latency** | Sub-millisecond |
| **Initial throughput** | 1 terabyte per second |
| **Availability SLA** | 99.9% in a single zone |
| **POSIX compliant** | Yes |

> "Google Cloud Managed Lustre makes it easier for customers to bring their workloads to the cloud without re-architecting. It optimizes TCO by maximizing the utilization of expensive GPUs and TPUs."  
> — [Tech Field Day](https://techfieldday.com/video/intro-to-managed-lustre-with-google-cloud/)

#### Performance Tiers

| Tier | Throughput | Best For |
|------|------------|----------|
| **1000 MB/s/TiB** | Highest | Foundation model training, large-scale physics simulations |
| **250 MB/s/TiB** | Balanced | General HPC workloads, AI inference serving, analytics pipelines |
| **125 MB/s/TiB** | Lower | Large-capacity use cases, on-premises workload migration |

> Source: [GKE Managed Lustre CSI Driver Best Practices](https://cloud.google.com/blog/products/containers-kubernetes/gke-managed-lustre-csi-driver-for-aiml-and-hpc-workloads)

#### Architecture

> "By boosting Managed Lustre performance to **10 TB/s**, we've cleared the path for massive AI/ML workloads, accelerating both training and inference pipelines."  
> — [Google Cloud](https://cloud.google.com/products/managed-lustre)

#### Training Optimization

> "Compared to using Cloud Storage alone, using Managed Lustre significantly reduces training time and improves the responsiveness of your models."  
> — [Google Cloud Architecture](https://docs.cloud.google.com/architecture/optimize-ai-ml-workloads-managed-lustre)

**Training Data Flow:**
1. Training data imported to Managed Lustre from Cloud Storage
2. GKE pods read training data directly from Managed Lustre
3. Model checkpoints written back to Managed Lustre

> Source: [Google Cloud Architecture Center](https://docs.cloud.google.com/architecture/optimize-ai-ml-workloads-managed-lustre)

---

### Azure Managed Lustre

**Documentation:** [Azure Managed Lustre File System](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/)

#### Overview

> "Azure Managed Lustre is a **managed file system** that offers scalable, powerful, cost-effective storage for **high-performance computing (HPC)** workloads."  
> — [Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/amlfs-overview)

> "Lustre is an open-source parallel file system that can scale to massive storage sizes while providing high-performance throughput. It's used by the world's fastest supercomputers and in data-centric workflows across many industries."  
> — [Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/amlfs-overview)

#### Data Security

| Security Feature | Details |
|-----------------|---------|
| **Encryption at Rest** | All data stored in Azure is encrypted using Azure managed keys by default |
| **VM Host Encryption** | Protected by VM host encryption on managed disks holding your data |
| **Customer-Managed Keys** | Optional additional security layer for high-security needs |
| **Data Residency** | Azure Managed Lustre doesn't store customer data outside the deployment region |

#### Blob Storage Integration

> "Azure Managed Lustre seamlessly integrates with Azure Blob Storage, enabling **Lustre Hierarchical Storage Management (HSM)** capabilities."

| Feature | Description |
|---------|-------------|
| **Selective Import** | Specify files to import from blob containers without importing entire datasets |
| **Flexible Job Storage** | Create different file systems for different jobs |
| **Export Capability** | After HPC jobs complete, export changed data to Blob Storage and delete the Lustre system |

> Source: [Azure Managed Lustre Overview](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/amlfs-overview)

#### Kubernetes Integration

> "If you want to use an Azure Managed Lustre storage system with your Kubernetes containers, you can use the Azure Lustre container support interface (CSI) driver for Kubernetes, which is compatible with Azure Kubernetes Service (AKS)."  
> — [Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/amlfs-overview)

---

## On-Premises Lustre

**Official site:** [Lustre.org](https://www.lustre.org/)

**Documentation:** [Lustre Wiki](https://wiki.lustre.org/Main_Page)

**Manual:** [Lustre Software Release 2.x](https://doc.lustre.org/lustre_manual.xhtml)

### Architecture Components

> "The building blocks of a Lustre file system are the Metadata Servers and Object Storage Servers, which provide namespace operations and bulk IO services respectively."  
> — [Lustre Wiki](https://wiki.lustre.org/Introduction_to_Lustre)

#### Metadata Server (MDS)

- Manages all namespace operations (directories, filenames)
- Stores directory hierarchy and file information on **Metadata Targets (MDT)**
- Controls: file access permissions, opening/closing files, deletions, renames, object allocation

#### Object Storage Server (OSS)

- Manages bulk storage for file contents
- File data stored on **Object Storage Targets (OST)**
- Single file system can scale to **hundreds of OSS nodes** and **thousands of OSTs**

#### Management Server (MGS)

- Global registry for configuration information
- Stores configuration for **all Lustre filesystems** in a cluster

### Scalability Specifications

| Parameter | LDISKFS Backend | ZFS Backend |
|-----------|-----------------|-------------|
| **Max stripe count** | 2000 | 2000 |
| **Max file size** | 31.25PB | 512PB |
| **Max filesystem size** | 512PB | 8EB |
| **Max files in single directory** | 10M (48-byte names) / 5M (128-byte names) | 248 |

> Source: [Lustre Wiki - Introduction to Lustre](https://wiki.lustre.org/Introduction_to_Lustre)

### Client Mounting

> "Start a Lustre client using the mount command, the basic syntax of which is:  
> `mount -t lustre [-o OPTIONS ] MGSNID[:MGSNID]:/FSNAME /lustre/FSNAME`"  
> — [Lustre Wiki](https://wiki.lustre.org/Mounting_a_Lustre_File_System_on_Client_Nodes)

---

## ML Training Performance

### MLPerf Storage Benchmark

> "MLPerf Storage assesses how fast storage systems can process inputs and produce results using a **trained model** (no actual training required)."  
> — [AI/ML Benchmarking and Lustre - LUG 2024](https://wiki.lustre.org/images/5/52/LUG2024-AI_ML_Benchmarking_and_Lustre-Samar.pdf)

#### MLPerf Storage Workloads

| Dataset | Sample Size | Framework | Model | Problem Area |
|---------|-------------|-----------|-------|--------------|
| KiTS 19 | 140MB/sample | PyTorch | 3D U-Net | Image Segmentation |
| Wikipedia | 2.5KB/sample | PyTorch | BERT-large | Language Processing |
| CosmoFlow | 2MB/sample | PyTorch | CosmoFlow | Cosmology parameter prediction |

> Source: [AI/ML Benchmarking and Lustre - HPE](https://wiki.lustre.org/images/5/52/LUG2024-AI_ML_Benchmarking_and_Lustre-Samar.pdf)

#### Key Metric: Accelerator Utilization (AU)

> "**Accelerator Utilization (AU)** represents the percentage of total test time during which accelerators are active, with a minimum requirement of **90%** to pass the benchmark."

> "AU = (Total Compute Time) / (Total Test Time) × 100"

#### MLPerf Storage v2.0 Checkpointing

> "Checkpointing is now a standard practice in these systems to mitigate failures, and we are proud to be providing critical benchmark data on storage systems to allow stakeholders to optimize their training performance."  
> — Curtis Anderson, MLPerf Storage Working Group Co-Chair

> "This initial round of checkpoint benchmark results shows us that current storage systems offer a wide range of performance specifications, and not all systems are well-matched to every checkpointing scenario."  
> — Curtis Anderson

> Source: [MLPerf Storage v2.0 Results](https://mlcommons.org/2025/08/mlperf-storage-v2-0-results/)

### Failure at Scale

> "If the mean time to failure for an accelerator is 50,000 hours, then a 100,000-accelerator cluster running for extended periods at full utilization will likely experience a failure every half-hour. A cluster with one million accelerators would expect to see a failure every three minutes."  
> — [MLCommons](https://mlcommons.org/2025/08/mlperf-storage-v2-0-results/)

### Throughput Benchmarks

> "With SSDs, Lustre's aggregate throughput can reach **dozens of GB/s** across multiple OSTs."  
> — [Marco Santucci](https://marcosantucci.eu/understanding-lustre-performance-throughput-in-high-demand-scenarios/)

> "For AI workflows, this increased throughput translates into faster data loading and processing, crucial for training models that require continuous data access."

### Use Case Recommendations

| Use Case | Recommended Solution |
|----------|---------------------|
| AI/ML training, checkpointing (low latency <1ms, high throughput, small files) | **Managed Lustre** |
| Data archiving, large files (>50MB), higher latency tolerance | Cloud Storage FUSE with Anywhere Cache |

> Source: [Google Cloud Blog](https://cloud.google.com/blog/products/containers-kubernetes/gke-managed-lustre-csi-driver-for-aiml-and-hpc-workloads)

---

## Checkpoint Strategies

### Checkpoint Sizing

> "Single Model Replica (MR) Checkpoint Size = (P × Wp) + (P × Os)"  
> Where: Model weights (BF16/FP16) = 2 bytes, Optimizer state = 8 bytes, **Total per parameter = 10 bytes**

**Example: 100B Parameter Model on 4,000 Accelerators**

| Parameter | Value |
|-----------|-------|
| Model Parameters | 100 billion |
| World Size (Accelerators) | 4,000 |
| Model Parallel Size | 32 GPUs per MR |
| Number of MRs | 125 (4,000 ÷ 32) |
| **Checkpoint per MR** | **1 TB** |
| **Total Read Size** | **125 TB** |
| **Total Write Size** | **1 TB** (single replica) |

> Source: [AWS Storage Blog](https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/)

### Reading Strategies

| Approach | Best For | Limitations |
|----------|----------|-------------|
| **Concurrent Loading** | Smaller deployments, checkpoints <100 GB, 5+ min loading acceptable | Network contention at scale |
| **Hierarchical Distribution** | Large clusters with terabyte checkpoints | Requires EFA network |

> "Loading 125 TB in 2 minutes requires **8.3 Tbps** throughput from storage—creating massive demand spikes regardless of storage solution."

### Hierarchical Distribution Solution

> "Storage throughput requirement: 125 TB ÷ 120 seconds = 8.33 GB/s = 66.7 Gbps"

> "This achieves a **125x reduction** in storage throughput requirements compared to concurrent loading."

**Architecture:**
```
Leader Nodes (n/8 instances) → Load from Storage → NCCL Broadcast → Worker Nodes (N/8 - n/8 instances)
```

> Source: [AWS Storage Blog](https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/)

### Writing Strategies

#### Synchronous Checkpointing Impact

> "For a 4,000-accelerator cluster with 3-minute checkpoint pause:
> - **200 GPU-hours** cluster-wide idle time per checkpoint
> - **9,600 GPU-hours** lost daily (at 30-minute checkpoint frequency, 48 checkpoints/day)"

#### Multi-Level Checkpointing

| Tier | Frequency | Storage | Use Case |
|------|------------|---------|----------|
| Fast | 5 minutes | Local NVMe | Quick recovery |
| Mid | 30 minutes | FSx for Lustre | Balance speed/durability |
| Durable | 1+ hours | Amazon S3 | Long-term persistence |

> Source: [AWS Storage Blog](https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/)

---

## Comparison: Lustre vs Object Storage

### Performance Comparison

| Aspect | Lustre (FSx/Managed) | Object Storage (S3/GCS) |
|--------|---------------------|-------------------------|
| **Latency** | Sub-millisecond | 10-100+ ms |
| **Throughput** | TB/s aggregate | 100s GB/s |
| **IOPS** | Millions | 100s K |
| **POSIX compliance** | Native | Via FUSE/adapter |
| **Cost per GB** | Higher | Lower |
| **Access pattern** | File-based | Object-based |
| **Use case** | Training/Checkpointing | Archival/Model storage |

> "Lustre manages high-throughput data ingestion during training, while MinIO handles check-pointing, logging, and long-term storage."  
> — [IEEE Paper](https://ieeexplore.ieee.org/iel8/6287639/11323511/11393949.pdf)

### When to Use Lustre vs Object Storage

> "An inadequate storage solution can significantly bottleneck GPU utilization, causing idle GPUs and hindering data processing from initial data preparation to model serving."

| Workload | Recommended Storage |
|----------|-------------------|
| Active training data | Managed Lustre |
| Checkpoint writes | Managed Lustre |
| Model artifact storage | Object Storage |
| Archival data | Object Storage |

> Source: [Tech Field Day - Google Cloud Storage Overview](https://techfieldday.com/video/overview-of-cloud-storage-storage-for-ai-lustre-gcsfuse-and-anywhere-cache-with-google-cloud/)

### Mountpoint for S3 Alternative

> "When Amazon introduced FSx for Lustre back in 2018, it offered a compelling idea: bring the capabilities of a traditional high-performance file system—based on the open-source Lustre project—into the cloud, while bridging directly to S3 object storage."  
> — [Storj Blog](https://www.storj.io/blog/has-fsx-lost-its-luster)

> "To help address the cost side of FSx for Lustre, Amazon introduced Mountpoint for Amazon S3 more recently in 2023—a tool that allows direct access to S3 buckets without requiring data staging. Mountpoint is free to use (you only pay for S3 storage) and simplifies access to cloud data."

> "For teams in media production, AI/ML, and HPC, FSx for Lustre promised the best of both worlds: higher throughput for massive workloads, paired with the scalability and durability of S3."

---

## Kubernetes Integration

### AWS FSx for Lustre CSI Driver

**Repository:** [kubernetes-sigs/aws-fsx-csi-driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver)

> "The FSx for Lustre CSI driver is a Kubernetes component that enables dynamic provisioning of FSx volumes."  
> — [Pods and Pixels](https://podsandpixels.com/p/setting-up-amazon-fsx-for-lustre)

#### Features

- Dynamic provisioning of FSx volumes
- S3 data repository integration
- Automatic import/export capabilities

#### Deployment

> "Step 1: Create an IAM role → Step 2: Install the Amazon FSx CSI driver → Step 3: Deploy a storage class, persistent volume claim, and sample app."  
> — [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/fsx-csi-create.html)

### Google Cloud Managed Lustre CSI Driver

> "Many on-premises HPC environments already use parallel file systems, and Managed Lustre makes it easier to bring those workloads to the cloud."  
> — [Google Cloud Blog](https://cloud.google.com/blog/products/containers-kubernetes/gke-managed-lustre-csi-driver-for-aiml-and-hpc-workloads)

#### Best Practices

| Practice | Description |
|----------|-------------|
| **WaitForFirstConsumer** | Set `volumeBindingMode: WaitForFirstConsumer` for zone co-location |
| **Right-size performance** | Choose tiers based on actual I/O profiles |
| **VPC networking** | Enable service networking, create IP range, configure firewall |
| **Dynamic provisioning** | Default to dynamic for workload-coupled storage |
| **Parallel Jobs** | Use Kubernetes Jobs for parallel data preprocessing |

> Source: [GKE Managed Lustre CSI Driver Best Practices](https://cloud.google.com/blog/products/containers-kubernetes/gke-managed-lustre-csi-driver-for-aiml-and-hpc-workloads)

### Azure Lustre CSI Driver

> "Use Azure Managed Lustre with **Azure Kubernetes Service (AKS)** via the **Azure Lustre Container Storage Interface (CSI) driver**."

### CSI Driver Components

| Component | Type | Default Replicas |
|-----------|------|-----------------|
| CSI Controller Plugin | Deployment | 2 |
| CSI Node Plugin | DaemonSet | Configurable |

> Source: [Microsoft Learn](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/amlfs-overview)

---

## Key Benchmarks & Research

### MLPerf Storage v2.0 Results (August 2025)

| Metric | Value |
|--------|-------|
| Performance Results | >200 |
| Submitting Organizations | 26 |
| Countries Represented | 7 |
| Accelerators Supported | ~2x vs v1.0 |

> "Everything is scaling up: models, parameters, training datasets, clusters, and accelerators."  
> — Oana Balmau, MLPerf Storage Working Group Co-Chair

> "Necessity continues to be the mother of invention: faced with the need to deliver storage solutions that are both high-performance and at unprecedented scale, the technical community has stepped up once again and is innovating at a furious pace."

> Source: [MLCommons](https://mlcommons.org/2025/08/mlperf-storage-v2-0-results/)

### AI/ML Benchmarking and Lustre (LUG 2024)

**Authors:** Sakib Samar, HPE HPC Storage Team

> "The MLPerf Storage v0.5 employs simulated NVIDIA V100 GPUs, while MLPerf Storage v1.0 uses NVIDIA A100 and NVIDIA H100 GPUs."

| Version | Emulated GPUs |
|---------|---------------|
| MLPerf Storage v0.5 | NVIDIA V100 |
| MLPerf Storage v1.0 | NVIDIA A100, NVIDIA H100 |

> Source: [PDF - LUG2024-AI_ML_Benchmarking_and_Lustre-Samar.pdf](https://wiki.lustre.org/images/5/52/LUG2024-AI_ML_Benchmarking_and_Lustre-Samar.pdf)

### IEEE Paper: LustreFS vs S3-Compatible Object Storage

> "In the converged setup, Lustre manages high-throughput data ingestion during training, while MinIO handles check-pointing, logging, and long-term storage."

> Source: [IEEE](https://ieeexplore.ieee.org/iel8/6287639/11323511/11393949.pdf)

---

## Resources

### Official Documentation

| Provider | Documentation |
|----------|--------------|
| AWS FSx for Lustre | [Performance Guide](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance.html), [Performance Tips](https://docs.aws.amazon.com/fsx/latest/LustreGuide/performance-tips.html) |
| Google Cloud Managed Lustre | [Overview](https://cloud.google.com/managed-lustre/docs/overview), [GKE CSI Driver](https://cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/lustre-csi-driver-new-volume) |
| Azure Managed Lustre | [Documentation](https://learn.microsoft.com/en-us/azure/azure-managed-lustre/) |
| Lustre (Open Source) | [Lustre.org](https://www.lustre.org/), [Wiki](https://wiki.lustre.org/Main_Page), [Manual](https://doc.lustre.org/lustre_manual.xhtml) |

### GitHub Repositories

| Repository | Description |
|------------|-------------|
| [kubernetes-sigs/aws-fsx-csi-driver](https://github.com/kubernetes-sigs/aws-fsx-csi-driver) | AWS FSx CSI driver for Kubernetes |
| [DDN EXAScaler](https://www.ddn.com/products/lustre-file-system-exascaler/) | Commercial Lustre implementation (Google Cloud backend) |

### Key Blog Posts

- [Architecting scalable checkpoint storage for large-scale ML training on AWS](https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/)
- [GKE Managed Lustre CSI Driver Best Practices](https://cloud.google.com/blog/products/containers-kubernetes/gke-managed-lustre-csi-driver-for-aiml-and-hpc-workloads)
- [Optimize AI and ML workloads with Google Cloud Managed Lustre](https://docs.cloud.google.com/architecture/optimize-ai-ml-workloads-managed-lustre)
- [Speed up training on SageMaker using FSx for Lustre](https://aws.amazon.com/blogs/machine-learning/speed-up-training-on-amazon-sagemaker-using-amazon-efs-or-amazon-fsx-for-lustre-file-systems/)
- [Has FSx lost its luster?](https://www.storj.io/blog/has-fsx-lost-its-luster)

### Benchmark Resources

- [MLPerf Storage Benchmark](https://mlcommons.org/benchmarks/storage/)
- [MLPerf Storage v2.0 Results](https://mlcommons.org/2025/08/mlperf-storage-v2-0-results/)
- [AI/ML Benchmarking and Lustre (LUG 2024)](https://wiki.lustre.org/images/5/52/LUG2024-AI_ML_Benchmarking_and_Lustre-Samar.pdf)

---

## Summary

Managed Lustre provides high-performance, POSIX-compliant parallel file system storage essential for ML training workloads requiring:

- **Sub-millisecond latency** for GPU/TPU utilization
- **TB/s aggregate throughput** for dataset streaming
- **Checkpoint optimization** with hierarchical strategies
- **Kubernetes integration** via CSI drivers
- **S3/GCS integration** for cloud-native workflows

Cloud providers offer fully managed Lustre solutions:
- **AWS FSx for Lustre**: Up to 1.2 Tbps client throughput with EFA
- **Google Cloud Managed Lustre**: Up to 10 TB/s with DDN EXAScaler
- **Azure Managed Lustre**: Integrated Blob Storage HSM with AKS CSI support

For Kubernetes deployments, CSI drivers from all three providers enable dynamic volume provisioning with storage class integration.