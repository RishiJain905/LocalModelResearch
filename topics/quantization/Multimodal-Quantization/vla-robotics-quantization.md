# Quantizing Vision-Language-Action (VLA) Models for Robotics

> **Key Papers:** BitVLA ([arXiv:2506.07530](https://arxiv.org/abs/2506.07530)) · QuantVLA ([arXiv:2602.20309](https://arxiv.org/abs/2602.20309)) · QVLA ([arXiv:2602.03782](https://arxiv.org/abs/2602.03782)) · VQ-VLA ([arXiv:2507.01016](https://arxiv.org/abs/2507.01016)) · EaqVLA ([arXiv:2505.21567](https://arxiv.org/abs/2505.21567))  
> **Survey:** [arXiv:2507.01925](https://arxiv.org/abs/2507.01925) (Action Tokenization for VLAs)  
> **GitHub:** [ustcwhy/BitVLA](https://github.com/ustcwhy/BitVLA) · [openvla/openvla](https://github.com/openvla/openvla) · [xiaoxiao0406/VQ-VLA](https://github.com/xiaoxiao0406/VQ-VLA)

---

## Overview

> *"VLA quantization is not standard LLM quantization — action tokens require special handling due to temporal dependencies and physical consequences of errors."* — QuantVLA (arXiv:2602.20309)

**VLA (Vision-Language-Action)** models extend VLMs to robotics — they output **discrete action tokens** that control robot movements. Quantizing VLAs is fundamentally different from standard LLM quantization because action errors have **direct physical consequences**.

---

## How Action Tokens Differ from Vision/Language Tokens

| Aspect | Vision Tokens | Language Tokens | Action Tokens |
|--------|-------------|-----------------|---------------|
| **Structure** | 2D spatial grid | 1D sequential | Time-series with temporal dependencies |
| **Quantization Sensitivity** | Low (redundancy) | High (semantic) | **Very high** — errors compound |
| **Error Propagation** | Tolerable | Moderate | **Severe** — physical failures |
| **Precision Requirement** | ~8-bit sufficient | ~8-bit for reasoning | **Higher needed** for physical execution |
| **Temporal Dependencies** | Spatial redundancy | Long-range context | **Strong short-term correlations** |

### Core Challenge: Discretizing Continuous Robot Actions

```
Robot actions: end-effector deltas, joint velocities (continuous)
    ↓ Tokenization (VQ-VAE, BPE, DCT)
Discrete action tokens for autoregressive training
    ↓
Small quantization error → physical robot failure
```

### Key Finding from EaqVLA

| Observation | Detail |
|------------|--------|
| **Language gradients** | Order of magnitude higher than vision modality |
| **Projector quantization** | **MUST NOT** use compensation-based quantization (GPTQ) — breaks entirely |
| **Mixed-precision strategy** | 4-bit vision (ViT), FP16 projector, 8-bit language (LLaMA) |

### Key Finding from QuantVLA

Two systematic drifts identified in VLA quantization:

| Drift | Description |
|-------|-------------|
| **Attention temperature drift** | Variance changes in Q/K alter softmax temperature |
| **Residual energy drift** | Amplitude changes modify residual injection gain in deep DiT stacks |

---

## Key Methods

### 1. BitVLA: 1-bit VLAs

> **arXiv:** [2506.07530](https://arxiv.org/abs/2506.07530) · **GitHub:** [ustcwhy/BitVLA](https://github.com/ustcwhy/BitVLA)

**Key contribution:** 1.58-bit quantization; 3.0B model at **1.4GB** (vs 15+GB full-precision).

> *"BitVLA achieves 96.0 avg success rate on LIBERO at 1.4GB, compared to OpenVLA at 76.5 with 15.1GB."*

| Metric | BitVLA | OpenVLA |
|--------|--------|---------|
| **Memory** | 1.4GB | 15.1GB |
| **Success Rate** | 96.0 | 76.5 |
| **Bits** | 1.58 | 16 |
| **Implementation** | Master weights in BF16; uses BitNet.cpp |

---

### 2. QuantVLA: Scale-Calibrated PTQ (CVPR 2026)

> **arXiv:** [2602.20309](https://arxiv.org/abs/2602.20309) (CVPR 2026)

**Key contribution:** First training-free PTQ for VLAs. W4A8 achieves **~70% memory savings**, exceeds full-precision baselines.

> *"Standard PTQ methods designed for unimodal models fail under VLA's tight multimodal-diffusion coupling."*

| Result | Detail |
|--------|--------|
| **Memory savings** | ~70% with W4A8 |
| **Performance** | Exceeds full-precision baselines |
| **Training required** | None |

---

### 3. QVLA: Not All Channels Are Equal (ICLR 2026)

> **arXiv:** [2602.03782](https://arxiv.org/abs/2602.03782) (ICLR 2026)

**Key contribution:** Per-channel bit allocation for VLA via OpenVLA-OFT.

| Result | Detail |
|--------|--------|
| **Memory** | 29.2% of full-precision |
| **Performance maintained** | 98.9% (vs FP16) |

---

### 4. VQ-VLA: Vector-Quantized Action Tokenizers (ICCV 2025)

> **arXiv:** [2507.01016](https://arxiv.org/abs/2507.01016) · **GitHub:** [xiaoxiao0406/VQ-VLA](https://github.com/xiaoxiao0406/VQ-VLA)

**Key contribution:** Large-scale VQ action tokenizer — **100× more data** than prior methods. **30% higher success** on long-horizon tasks.

| Feature | Detail |
|---------|--------|
| **Tokenization** | 5× compression in VQ-VAE |
| **Inference speedup** | 3× |
| **Data scale** | 100× more than prior |
| **Long-horizon improvement** | 30% higher success |

---

### 5. EaqVLA: Encoding-Aligned Quantization (2025)

> **arXiv:** [2505.21567](https://arxiv.org/abs/2505.21567)

**Key contribution:** Modular mixed-precision — **4-bit vision**, **FP16 projector**, **8-bit language**.

> *"Avoids quantizing projector module entirely — it is the bridge between vision and language and breaks under most quantization methods."*

---

### 6. HBVLA: Pushing 1-Bit Post-Training Quantization

> **arXiv:** [2602.13710](https://arxiv.org/abs/2602.13710)

**Key contribution:** Ultra-low-bit (1-bit) PTQ for VLAs enabling deployment on **hardware-limited robots**.

---

### 7. DyQ-VLA: Temporal-Dynamic-Aware Quantization

> **arXiv:** [2603.07904](https://arxiv.org/abs/2603.07904)

**Key contribution:** Temporal-aware quantization accounting for the **dynamic nature of robot actions**.

---

## Benchmarks

### LIBERO (Language-Conditioned Manipulation)

> **GitHub:** [Lifelong-Robot-Learning/LIBERO](https://github.com/Lifelong-Robot-Learning/LIBERO)

| Tasks | Suites | Focus |
|------|--------|-------|
| 130 tasks | spatial, object, goal, long | Language-conditioned manipulation |

### VLABench

> **GitHub:** [OpenMOSS/VLABench](https://github.com/OpenMOSS/VLABench) · [vlabench.github.io](https://vlabench.github.io/)

| Feature | Detail |
|---------|--------|
| Tasks | 100 task categories |
| Focus | Long-horizon reasoning, world knowledge, common sense |
| Venue | ICCV 2025 |

### Other Benchmarks

| Benchmark | Focus | URL |
|----------|-------|-----|
| **RLBench** | 100 tasks, PyRep-based manipulation | [stepjam/RLBench](https://github.com/stepjam/RLBench) |
| **CALVIN** | Multi-step, compounding error | [mees/calvin](https://github.com/mees/calvin) |
| **ManiSkill** | Physics-based manipulation | [maniskill.ai](https://www.maniskill.ai/research) |
| **OpenX-Embodiment** | 970K episodes, cross-embodiment | HuggingFace |

---

## Results Summary

### Memory vs Performance

| Model | Memory | Success Rate | Notes |
|-------|--------|-------------|-------|
| **OpenVLA (FP16)** | 15.1GB | 76.5 | Baseline |
| **BitVLA (1.58-bit)** | 1.4GB | 96.0 | +19.5% success, 10× smaller |
| **QVLA-OFT** | 29.2% | 98.9% | 98.9% of FP16 |
| **QuantVLA W4A8** | ~30% | >FP16 | 70% memory savings |

---

## Survey: Action Tokenization for VLAs

> **arXiv:** [2507.01925](https://arxiv.org/abs/2507.01925)

**70-page survey** covering action tokenization methods in VLAs — comprehensive reference.

---

## Papers Table

| Paper | Year | Venue | arXiv | Key Contribution |
|-------|------|-------|-------|-----------------|
| **BitVLA** | 2025 | — | [2506.07530](https://arxiv.org/abs/2506.07530) | 1-bit VLA, 1.4GB, 96.0 success |
| **QuantVLA** | 2026 | CVPR | [2602.20309](https://arxiv.org/abs/2602.20309) | Training-free PTQ, 70% memory savings |
| **QVLA** | 2026 | ICLR | [2602.03782](https://arxiv.org/abs/2602.03782) | Per-channel, 29.2% memory, 98.9% perf |
| **VQ-VLA** | 2025 | ICCV | [2507.01016](https://arxiv.org/abs/2507.01016) | 100× data, 30% higher long-horizon |
| **EaqVLA** | 2025 | — | [2505.21567](https://arxiv.org/abs/2505.21567) | 4-bit vis, FP16 proj, 8-bit LLM |
| **HBVLA** | 2026 | — | [2602.13710](https://arxiv.org/abs/2602.13710) | 1-bit PTQ for hardware-limited robots |
| **DyQ-VLA** | 2026 | — | [2603.07904](https://arxiv.org/abs/2603.07904) | Temporal-dynamic-aware quantization |
| **Survey** | 2025 | — | [2507.01925](https://arxiv.org/abs/2507.01925) | 70-page action tokenization survey |

---

## GitHub Repositories

| Repo | Description |
|------|-------------|
| [ustcwhy/BitVLA](https://github.com/ustcwhy/BitVLA) | 1-bit VLA, 1.4GB, BitNet.cpp |
| [openvla/openvla](https://github.com/openvla/openvla) | 7B open VLA, LoRA fine-tuning |
| [xiaoxiao0406/VQ-VLA](https://github.com/xiaoxiao0406/VQ-VLA) | VQ action tokenizer, ICCV 2025 |
| [Lifelong-Robot-Learning/LIBERO](https://github.com/Lifelong-Robot-Learning/LIBERO) | 130 language-conditioned tasks |
| [OpenMOSS/VLABench](https://github.com/OpenMOSS/VLABench) | ICCV 2025 benchmark, 100 categories |
| [physical-intelligence/fast](https://huggingface.co/physical-intelligence/fast) | DCT+BPE action tokenization |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core difference** | Action errors have physical consequences — not like text/vision |
| **Key challenge** | Temporal dependencies + compounding errors |
| **DiT action head** | Main bottleneck — needs careful quantization (keep projections in FP32) |
| **Projector** | Do NOT quantize — breaks under most methods |
| **BitVLA** | 1.58-bit → 1.4GB, 96.0 success (vs 76.5 baseline) |
| **QuantVLA** | W4A8 → 70% memory savings, exceeds FP baselines |
| **VQ-VLA** | 100× more training data, 30% better long-horizon |

---

*Last updated: 2026-04-19*
