# Dynamic Provisioning (Storage/Compute) for Kubernetes ML Training

## Table of Contents

- [Overview](#overview)
- [StorageClasses and Dynamic Provisioning](#storageclasses-and-dynamic-provisioning)
- [PVC Templates and VolumeClaimTemplates](#pvc-templates-and-volumeclaimtemplates)
- [Local Persistent Volumes](#local-persistent-volumes)
- [Volume Snapshots](#volume-snapshots)
- [Cloud Provider Provisioners](#cloud-provider-provisioners)
  - [AWS EBS](#aws-ebs)
  - [AWS EFS (NFS)](#aws-efs-nfs)
  - [GCP GKE Persistent Disks](#gcp-gke-persistent-disks)
- [emptyDir Ephemeral Volumes](#emptydir-ephemeral-volumes)
- [ML Training Workload Patterns](#ml-training-workload-patterns)
- [Storage for Datasets and Checkpoints](#storage-for-datasets-and-checkpoints)
- [Key Resources](#key-resources)

---

## Overview

> "Dynamic volume provisioning allows storage volumes to be created on-demand. Without dynamic provisioning, cluster administrators have to manually make calls to their cloud or storage provider to provision new storage volumes, and then create PersistentVolume objects to represent them in Kubernetes."
>
> — [Kubernetes Official Docs](https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/)

Dynamic provisioning eliminates pre-allocation of storage by automatically creating PersistentVolume (PV) objects when users create PersistentVolumeClaim (PVC) objects. The implementation is built on the `StorageClass` API object.

---

## StorageClasses and Dynamic Provisioning

A StorageClass defines the provisioner and parameters for a type of storage. Users reference StorageClasses in PVCs to request dynamic provisioning.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Provisioner** | Storage backend plugin (e.g., `ebs.csi.aws.com`, `kubernetes.io/gce-pd`) |
| **Parameters** | Backend-specific settings passed to the provisioner |
| **reclaimPolicy** | `Delete` (default for dynamic) or `Retain` when PVC is deleted |
| **volumeBindingMode** | `Immediate` (default) or `WaitForFirstConsumer` for topology awareness |
| **allowVolumeExpansion** | Enables online PVC resizing (optional) |

### Default StorageClass Behavior

> "If `storageClassName` is **not specified** in the PVC, the **default storage class will be used** for provisioning."
>
> — [Kubernetes Blog](https://kubernetes.io/blog/2017/03/dynamic-provisioning-and-storage-classes-kubernetes/)

### Example: StorageClass Definitions

```yaml
# Standard/gp2 (default in AWS EKS)
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp2
provisioner: aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: Immediate
allowVolumeExpansion: true

# SSD/gp3 with WaitForFirstConsumer
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3-ssd
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

---

## PVC Templates and VolumeClaimTemplates

### PersistentVolumeClaim (PVC)

Standard PVC requests storage from a StorageClass:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: training-data-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: fast-storage
  resources:
    requests:
      storage: 100Gi
```

### VolumeClaimTemplates (for StatefulSets)

> "VolumeClaimTemplates is a list of claims that pods are allowed to. Once you deploy your pods you notice that your pods are being created and `PVC` is requested during the creation."
>
> — [Stack Overflow](https://stackoverflow.com/questions/63076923/dynamic-volume-provisioning-folder-per-pod-in-a-volume)

VolumeClaimTemplates are used with StatefulSets to dynamically provision a unique PVC for each replica pod:

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: training-worker
spec:
  serviceName: "training-service"
  replicas: 4
  selector:
    matchLabels:
      app: training-worker
  template:
    metadata:
      labels:
        app: training-worker
    spec:
      containers:
      - name: trainer
        image: pytorch-training:latest
        volumeMounts:
        - name: dataset-volume
          mountPath: /data
        - name: checkpoint-volume
          mountPath: /checkpoints
  volumeClaimTemplates:
  - metadata:
      name: dataset-volume
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-nvme"
      resources:
        requests:
          storage: 500Gi
  - metadata:
      name: checkpoint-volume
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-nvme"
      resources:
        requests:
          storage: 200Gi
```

> "The name of the volume claim must always be `elasticsearch-data`... By default the operator creates a `PersistentVolumeClaim` with a capacity of 1Gi for each pod in an Elasticsearch cluster to prevent data loss in case of accidental pod deletion."
>
> — [Elastic ECK Docs](https://www.elastic.co/docs/deploy-manage/deploy/cloud-on-k8s/volume-claim-templates)

---

## Local Persistent Volumes

> "Local Persistent Volumes feature promoted to **GA** (General Availability) in Kubernetes 1.14."
>
> — [Kubernetes Blog](https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/)

### Local PV vs HostPath

| Feature | HostPath | Local Persistent Volume |
|---------|----------|------------------------|
| Scheduler awareness | No | Yes |
| Pod rescheduling risk | May move to different node (data loss) | Always scheduled to same node |
| Security | Pods can access any path | PVCs managed by admin |
| Inline volume support | Yes | No (PVC only) |
| Block device formatting | No | Yes |
| fsGroup ownership | No | Yes |

### Key Limitation

> "Local Persistent Volumes do **not support dynamic volume provisioning**."
>
> — [Kubernetes Blog](https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/)

**Solution:** Use the external static provisioner for lifecycle management of local disks.

### Local PV StorageClass

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
```

### Local Static Provisioner (SIGs)

> "The local volume static provisioner manages PersistentVolume lifecycle for pre-allocated disks by detecting and creating PVs for each local disk on the host."
>
> — [kubernetes-sigs/sig-storage-local-static-provisioner](https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner)

**GitHub:** https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner

### OpenEBS Dynamic LocalPV

> "OpenEBS Dynamic Local PV provisioner is an open-source Kubernetes component that automates the dynamic provisioning of local persistent volumes. It converts local storage available on Kubernetes nodes, such as hostPath directories into persistent volumes accessible via PVCs."
>
> — [OpenEBS GitHub](https://github.com/openebs/dynamic-localpv-provisioner)

**GitHub:** https://github.com/openebs/dynamic-localpv-provisioner

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-hostpath
  annotations:
    openebs.io/cas-type: local
provisioner: openebs.io/local
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---

## Volume Snapshots

> "Volume Snapshots provide point-in-time copy functionality for persistent volumes. A snapshot can be used to provision a new volume pre-populated with snapshot data, or to restore an existing volume to a previous state."
>
> — [Kubernetes CSI Docs](https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html)

### CSI Snapshot CRDs

| CRD | Purpose |
|-----|---------|
| `VolumeSnapshot` | Represents a snapshot request |
| `VolumeSnapshotContent` | Represents actual snapshot on storage |
| `VolumeSnapshotClass` | Storage class configuration for snapshots |

### Version Status

| Status | K8s Versions | Feature |
|--------|--------------|---------|
| **Beta** | **1.17+** | Enabled by default |

### Volume Snapshot Example

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: ebs-snapshot-class
driver: ebs.csi.aws.com
deletionPolicy: Retain
parameters:
  tagSpecification_1: 'Environment=production'

---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: checkpoint-snapshot-001
  namespace: training
spec:
  volumeSnapshotClassName: ebs-snapshot-class
  source:
    persistentVolumeClaimName: checkpoint-pvc
```

### Restore PVC from Snapshot

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-checkpoint-pvc
  namespace: training
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3-standard
  resources:
    requests:
      storage: 100Gi
  dataSource:
    name: checkpoint-snapshot-001
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
```

---

## Cloud Provider Provisioners

### AWS EBS

> "The Amazon EBS CSI driver makes Amazon EBS volumes for these types of Kubernetes volumes: generic ephemeral volumes and persistent volumes."
>
> — [AWS EKS Documentation](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html)

**GitHub:** https://github.com/kubernetes-sigs/aws-ebs-csi-driver

**Features:**
- Dynamic Provisioning — Automatically create EBS volumes
- Volume Snapshots — Create and restore snapshots
- Volume Expansion — Online resizing
- Encryption — AES-256 with KMS

**StorageClass (gp3):**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  allowVolumeExpansion: "true"
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete
```

**EBS Volume Types:**

| Type | Use Case |
|------|----------|
| `gp3` | General purpose (SSD) — baseline 3000 IOPS, 125 MB/s |
| `gp2` | Legacy general purpose |
| `io2` | High-performance block storage |

### AWS EFS (NFS)

> "Amazon EFS CSI Driver v1.2 introduces dynamic provisioning, enabling automatic creation of PersistentVolumes (PVs) for EFS file systems without manual pre-provisioning."
>
> — [AWS Blog](https://aws.amazon.com/blogs/containers/introducing-efs-csi-dynamic-provisioning/)

**GitHub:** https://github.com/kubernetes-sigs/aws-efs-csi-driver

**Features:**
- `ReadWriteMany` — Shared access across AZs
- Encryption in transit (enabled by default)
- Dynamic provisioning via EFS Access Points
- Up to 120 PVs per EFS file system

**Key Limitation:**

> **AWS Fargate:** Dynamic Provisioning **not yet supported** on Fargate pods. Controller requires access to IMDS.

### GCP GKE Persistent Disks

**GitHub:** https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver

**Regional PD StorageClass:**

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: regionalpd-storageclass
provisioner: pd.csi.storage.gke.io
parameters:
  type: pd-balanced
  replication-type: regional-pd
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- key: topology.gke.io/zone
  values:
  - us-central1-a
  - us-central1-b
```

---

## emptyDir Ephemeral Volumes

> "An emptyDir volume is a type of ephemeral storage that is created when a pod is assigned to a node. The data stored in an emptyDir volume is initially empty and can be accessed by all containers within the same pod."
>
> — [Zesty.co](https://zesty.co/finops-glossary/emptydir-volume-in-kubernetes/)

**emptyDir Example:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: training-pod
spec:
  containers:
  - name: trainer
    image: pytorch-training:latest
    volumeMounts:
    - mountPath: /tmp
      name: scratch-space
  volumes:
  - name: scratch-space
    emptyDir:
      medium: ""  # Uses node's default disk
      sizeLimit: 10Gi
```

**Use Cases for Training:**

- Inter-container shared scratch space
- Temporary model intermediate outputs
- **Not suitable for:** dataset storage, checkpoints, model weights

**Memory-backed emptyDir:**

```yaml
volumes:
- name: ram-temp
  emptyDir:
    medium: Memory
    sizeLimit: 32Gi
```

> "When configured with `medium: "Memory"`, an `emptyDir` volume uses the node's RAM instead of disk, providing faster access to data."
>
> — [Zesty.co](https://zesty.co/finops-glossary/emptydir-volume-in-kubernetes/)

---

## ML Training Workload Patterns

> "AI/ML workloads segment into **four distinct storage use cases**, each with its own I/O profile: Dataset Ingestion, Checkpoint Storage, Model Serving/Inference Caches, Vector Databases."
>
> — [Simplyblock](https://simplyblock.io/glossary/kubernetes-storage-for-ai-workloads/)

### I/O Patterns Summary

| Use Case | Storage Type | Pattern | Requirements |
|----------|--------------|---------|--------------|
| Training datasets | Object (S3/GCS) or Block | Sequential reads | High throughput streaming |
| Training checkpoints | **Block (NVMe PVC)** | Burst writes | High burst write bandwidth |
| Vector databases | **Block (NVMe PVC)** | Random read/write | High IOPS |
| Inference caches | **Block (NVMe PVC)** | Random reads | Low latency |

### Checkpoint Storage Sizing

> "Checkpoint PVC size ≈ model_size_GB × checkpoints_retained × safety_factor (1.5x)"
>
> — [Simplyblock](https://simplyblock.io/glossary/kubernetes-storage-for-ai-workloads/)

**Example:** 70B parameter model (~140 GB in BF16) with 3 retained checkpoints:
```
140 GB × 3 × 1.5 = 630 GB (with safety margin)
```

### Kubeflow VolumeClaimTemplate Support

> "We want to add volumeClaimTemplate to job spec" — [Kubeflow GitHub](https://github.com/kubeflow/common/issues/19)

---

## Storage for Datasets and Checkpoints

### Best Practice

> "Use fast NVMe block storage rather than object storage or NFS for checkpoint and vector database volumes."
>
> — [Simplyblock](https://simplyblock.io/glossary/kubernetes-storage-for-ai-workloads/)

### Training Job Checkpoint Pattern

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: checkpoint-pvc
spec:
  storageClassName: nvme-fast
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 630Gi  # 140GB model × 3 checkpoints × 1.5 safety
---
apiVersion: v1
kind: Pod
metadata:
  name: training-pod
spec:
  containers:
  - name: trainer
    image: pytorch-training:latest
    volumeMounts:
    - name: checkpoint-storage
      mountPath: /checkpoints
  volumes:
  - name: checkpoint-storage
    persistentVolumeClaim:
      claimName: checkpoint-pvc
```

### QoS Isolation

> "Training jobs run large checkpoint writes that can saturate the storage fabric, causing latency spikes in inference serving pods on the same cluster. Without storage-level QoS isolation, a single large training run can cause inference timeout errors."
>
> — [Simplyblock](https://simplyblock.io/glossary/kubernetes-storage-for-ai-workloads/)

---

## Key Resources

### Official Documentation

| Resource | URL |
|----------|-----|
| Dynamic Volume Provisioning | https://kubernetes.io/docs/concepts/storage/dynamic-provisioning/ |
| Storage Classes | https://kubernetes.io/docs/concepts/storage/storage-classes/ |
| Local Persistent Volumes (GA in 1.14) | https://kubernetes.io/blog/2019/04/04/kubernetes-1.14-local-persistent-volumes-ga/ |
| CSI Snapshot/Restore | https://kubernetes-csi.github.io/docs/snapshot-restore-feature.html |

### GitHub Repos

| Repo | URL |
|------|-----|
| AWS EBS CSI Driver | https://github.com/kubernetes-sigs/aws-ebs-csi-driver |
| AWS EFS CSI Driver | https://github.com/kubernetes-sigs/aws-efs-csi-driver |
| GCP PD CSI Driver | https://github.com/kubernetes-sigs/gcp-compute-persistent-disk-csi-driver |
| Local Static Provisioner | https://github.com/kubernetes-sigs/sig-storage-local-static-provisioner |
| OpenEBS Dynamic LocalPV | https://github.com/openebs/dynamic-localpv-provisioner |
| External NFS Provisioner (legacy) | https://github.com/kubernetes-incubator/external-storage/tree/master/nfs |

### Blog Posts & Technical Reports

| Title | Source |
|-------|--------|
| Dynamic Provisioning and Storage Classes in Kubernetes (2017) | https://kubernetes.io/blog/2017/03/dynamic-provisioning-and-storage-classes-kubernetes/ |
| Introducing Amazon EFS CSI Dynamic Provisioning | https://aws.amazon.com/blogs/containers/introducing-efs-csi-dynamic-provisioning/ |
| Architecting Scalable Checkpoint Storage for Large-Scale ML Training on AWS | https://aws.amazon.com/blogs/storage/architecting-scalable-checkpoint-storage-for-large-scale-ml-training-on-aws/ |
| Kubernetes Storage for AI and ML Workloads | https://simplyblock.io/glossary/kubernetes-storage-for-ai-workloads/ |
| Comparing Cloud Provider Kubernetes Storage Solutions for ML | https://www.softwareseni.com/comparing-cloud-provider-kubernetes-storage-solutions-for-machine-learning/ |
| Building Persistent Storage in Kubernetes with NFS | https://medium.com/@tradingcontentdrive/building-persistent-storage-in-kubernetes-with-nfs-a-complete-step-by-step-guide-180d2219a0a6 |

### Cloud Provider Docs

| Provider | Doc Link |
|----------|----------|
| AWS EBS CSI Driver (EKS) | https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html |
| GKE Regional Persistent Disks | https://docs.cloud.google.com/kubernetes-engine/docs/how-to/persistent-volumes/regional-pd |
| Azure Container Storage | https://learn.microsoft.com/en-us/azure/containers/storage/ |

---

## Summary

| Storage Type | Provisioner | Access Mode | Use for Training |
|--------------|------------|-------------|------------------|
| **EBS (gp3/io2)** | `ebs.csi.aws.com` | RWO | Checkpoints, model weights |
| **EFS (NFS)** | `efs.csi.aws.com` | RWX | Shared dataset access |
| **GCP Regional PD** | `pd.csi.storage.gke.io` | RWO | Regional HA workloads |
| **Local NVMe** | `kubernetes.io/no-provisioner` + static provisioner | RWO | High-performance local storage |
| **OpenEBS LocalPV** | `openebs.io/local` | RWO | Dynamic local storage |
| **emptyDir** | N/A (ephemeral) | N/A | Scratch space, tmp files |

---

*Research compiled for k8s-elastic-training wiki — Dynamic Provisioning (Storage/Compute) topic.*
*Last updated: 2026-04-30*
