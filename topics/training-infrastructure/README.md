# training-infrastructure

## 📌 Overview

Infrastructure, tooling, and systems for training large language models and other large-scale ML models — including distributed training frameworks, checkpointing, fault tolerance, hardware orchestration, and MLOps pipelines.

## Subtopics

| Folder / File | Description |
|---------------|-------------|
| [Dynamic-Provisioning.md](./Dynamic-Provisioning.md) | Dynamic PVC provisioning, StorageClasses, local PV, volume snapshots, NFS/EBS/GCS, emptyDir, and storage patterns for ML training datasets and model checkpoints |
| [karpenter.md](./karpenter.md) | Karpenter autoscaler — GPU node provisioning, Spot instance management, training job preemption, and elastic training on Kubernetes |
| [kubeflow.md](./kubeflow.md) | Kubeflow ML platform — TFJob/PyTorchJob/MPIJob operators, TrainJob API, KServe, Katib, Pipelines, and elastic training patterns |

| Folder | Description |
|--------|-------------|
| [fsdp2-vescale](./fsdp2-vescale/) | FSDP2 + veScale: RaggedShard, structure-aware planning, Muon optimizers — 5–66% throughput gains, ~52% FLOPs vs AdamW |
| [Token-Economics](./Token-Economics/) | Cloud vs local TCO, breakeven analysis, and token economics for AI infrastructure: GPU rental vs ownership, utilization thresholds, LCOAI, and billing models |

---

## Related Topics

- [fine-tuning](../fine-tuning/) — Fine-tuning methods and recipes
- [Hardware-Accelerators](../Hardware-Accelerators/) — GPU/NPU hardware for training
- [Inference-Serving](../Inference-Serving/) — Model serving infrastructure

---

*Last updated: 2026-04-26*
