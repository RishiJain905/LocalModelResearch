# Storage-Streaming

## 📌 Overview

Storage and data streaming technologies for distributed ML training on Kubernetes — covering parallel filesystems (Lustre), model loading streamers, NVMe over Fabrics, and the storage pipeline from dataset to GPU memory.

## Subtopics

| File | Description |
|------|-------------|
| [Managed-Lustre.md](./Managed-Lustre.md) | Managed Lustre (AWS FSx, Google Cloud, Azure) — parallel filesystem specs, MLPerf Storage benchmarks, S3 integration, POSIX performance, and Kubernetes CSI driver integration |
| [Model-Streamers.md](./Model-Streamers.md) | Model streaming tools — HuggingFace snapshot_download, SafeTensors, DeepSpeed ZeRO, Run:ai Model Streamer, CoreWeave Tensorizer, torch.load mmap, GPUDirect Storage, and Kubernetes PVC patterns |
| [NVMe-oF.md](./NVMe-oF.md) | NVMe over Fabrics — RDMA vs TCP transport, NVIDIA GPUDirect Storage (GDS), cuFile API, CSI drivers, performance benchmarks vs local NVMe/NFS/Lustre |

---

## Related

- [Kubernetes Training Infrastructure](../) — Parent topic
- [AWS FSx for Lustre](https://docs.aws.amazon.com/fsx/latest/LustreGuide/) — Cloud Lustre
- [MLCommons MLPerf Storage](https://mlcommons.org/benchmarks/storage/) — Storage benchmarks for AI training
- [NVIDIA GPUDirect Storage](https://docs.nvidia.com/gpudirect-storage/) — Direct storage-to-GPU data path
- [SafeTensors](https://github.com/huggingface/safetensors) — Zero-copy model format

*Last updated: 2026-04-30*
