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
| [hardware-compilers](./hardware-compilers/) | Triton 3.5 Python DSL (fused kernels, FlashAttention, FP8, TMA), Mojo language (MLIR, 6-35x Python, MAX engine), AMD RDNA3 intrinsics (WMMA, wavefront), ROCm HIP, and GPU kernel compiler comparison (CUDA/CUTLASS vs Triton vs Mojo) |
| [fsdp2-vescale](./fsdp2-vescale/) | FSDP2 + veScale: RaggedShard, structure-aware planning, Muon optimizers — 5–66% throughput gains, ~52% FLOPs vs AdamW |
| [Token-Economics](./Token-Economics/) | Cloud vs local TCO, breakeven analysis, and token economics for AI infrastructure: GPU rental vs ownership, utilization thresholds, LCOAI, and billing models |
| [observability-resilience](./observability-resilience/) | Dynamic routing for training jobs (Headless Services, Istio/Envoy), fault-tolerant logging (Fluent Bit, Loki), and resilience patterns for elastic ML training |
|| [Storage-Streaming](./Storage-Streaming/) | Managed Lustre (FSx, GCS, Azure), Model Streamers (SafeTensors, Run:ai, Tensorizer), NVMe-oF (RDMA/TCP, GPUDirect Storage), and storage benchmarks (MLPerf) |
|| [high-speed-fabrics](./high-speed-fabrics/) | Ultra Ethernet Consortium (UEC), RoCE v2 lossless Ethernet for GPU clusters, NVIDIA GPUDirect RDMA/Storage/P2P for zero-copy GPU networking |
| [green-ai-thermals](./green-ai-thermals/) | TDP specs (H100/B200/MI300X), DFS/undervolting (GPU Boost 3.0, DVFS), carbon tracking (CodeCarbon, Electricity Maps, Compute Gardener), and green GPU cloud providers |

---

## Related Topics

- [fine-tuning](../fine-tuning/) — Fine-tuning methods and recipes
- [Hardware-Accelerators](../Hardware-Accelerators/) — GPU/NPU hardware for training
- [Inference-Serving](../Inference-Serving/) — Model serving infrastructure

---

*Last updated: 2026-04-30*
