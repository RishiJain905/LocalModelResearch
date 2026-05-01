# green-ai-thermals

## 📌 Overview

Green-AI-Thermals covers the intersection of **power consumption, thermal management, energy efficiency, and carbon awareness** for ML training infrastructure — from individual GPU TDP specs to cluster-wide carbon-aware scheduling and renewable energy procurement.

## Subtopics

| File | Description |
|------|-------------|
| [TDP-Metrics.md](./TDP-Metrics.md) | Thermal Design Power (TDP) specs for NVIDIA/AMD GPUs — H100 (700W), B200 (~1,000W), MI300X (750W), power capping (nvidia-smi/rocm-smi), MLPerf energy benchmarks, DGX/HGX power envelopes, and cooling infrastructure guidance |
| [DFS-Undervolting.md](./DFS-Undervolting.md) | Dynamic Frequency Scaling (DFS) and GPU undervolting — NVIDIA GPU Boost 3.0, AMD P-State, DVFS energy savings (8.7–23.1%), kernel-level DVFS for LLMs (14.6% energy reduction), Zeus profiling, and undervolting stability testing |
| [Carbon-Tracking.md](./Carbon-Tracking.md) | Carbon tracking and carbon-aware computing — GPT-3/BERT CO₂ footprint, CodeCarbon, Electricity Maps/WattTime APIs, Compute Gardener/kube-green schedulers, green GPU clouds (Gensyn, Lambda, NexGen), and hyperscaler renewable commitments |

---

## Related

- [Kubernetes Training Infrastructure](../) — Parent topic
- [MLCommons MLPerf](https://mlcommons.org/benchmarks/training/) — Energy and performance benchmarks
- [Green Software Foundation](https://greensoftware.foundation/) — Carbon-aware computing standards
- [CodeCarbon](https://github.com/mlco2/codecarbon) — ML carbon tracking library

*Last updated: 2026-04-30*
