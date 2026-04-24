# Acceleration Frameworks

> Fine-tuning acceleration frameworks reduce VRAM usage, training time, or both — via custom kernels (Triton/CUDA), quantization-aware training, distributed sharding, or efficient adapter methods.

## Contents

| Framework | Type | Key Benefit | GitHub |
|-----------|------|-------------|--------|
| [Unsloth](./Unsloth.md) | Single-GPU optimized | ~2× speed, ~70% less VRAM via Triton kernels + smart packing | [unslothai/unsloth](https://github.com/unslothai/unsloth) ⭐ 56.5k |
| [Axolotl](./Axolotl.md) | Multi-GPU / YAML-driven | All major training methods, FSDP2/DDP/DeepSpeed backends | [axolotl-ai-cloud/axolotl](https://github.com/axolotl-ai-cloud/axolotl) |
| [QAT](./QAT.md) | Quantization technique | Recovers up to 96% of accuracy lost to PTQ at INT4 | [facebookresearch/LLM-QAT](https://github.com/facebookresearch/LLM-QAT) |
| [FSDP2](./FSDP2.md) | Distributed sharding | Per-parameter DTensor sharding, 7% less memory vs FSDP1 | [pytorch/torchtitan](https://github.com/pytorch/torchtitan) |

---

## Summary Comparison

| Framework | Best For | VRAM Savings | Speedup | Distributed? |
|-----------|----------|-------------|---------|-------------|
| **Unsloth** | Single-GPU fine-tuning, fast iteration | ~70% | ~2× | No (single-GPU) |
| **Axolotl** | Multi-GPU, all methods, YAML-config | Varies | Varies | Yes (FSDP1/2, DeepSpeed) |
| **QAT** | Low-bit fine-tuning accuracy recovery | Depends | — | N/A (technique) |
| **FSDP2** | Native PyTorch multi-GPU, mid-range models | 7% vs FSDP1 | ~1.5% better MFU | Yes (native PyTorch) |

---

## Related Topics

- [Fine-tuning overview](../fine-tuning/) — parent folder
- [Unsloth](./Unsloth.md) — custom Triton kernels, smart packing, GRPO support
- [Axolotl](./Axolotl.md) — YAML-driven multi-method fine-tuning
- [QAT](./QAT.md) — Quantization-Aware Training vs Post-Training Quantization
- [FSDP2](./FSDP2.md) — PyTorch's recommended distributed sharding (FSDP1 deprecated)
