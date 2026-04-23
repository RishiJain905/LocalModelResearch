# Memory-Efficiency-Techniques

## 📌 Overview

Software-level memory optimization techniques for LLM training and fine-tuning — applicable across both NVIDIA and AMD GPUs. These complement hardware-level optimizations in [gfx-hardware-overrides](../gfx-hardware-overrides/).

---

## Subtopics

| File | Topic | Type |
|------|-------|------|
| [xformers-ROCm.md](./xformers-ROCm.md) | xFormers on AMD ROCm | Memory-efficient attention |
| [Gradient-Checkpointing.md](./Gradient-Checkpointing.md) | Gradient Checkpointing | Activation recomputation |

---

## Summary Matrix

| Technique | Memory Savings | Compute Overhead | AMD ROCm | Consumer GPU |
|-----------|---------------|-----------------|----------|-------------|
| **xFormers FMHA** | ~24% VRAM vs naive | Faster | ⚠️ MI200+ only | ❌ RX 7900 XTX not supported |
| **Gradient Checkpointing** | 60–70% activation | +20–33% | ✅ Full support | ✅ All GPUs |

---

## Key Findings

- **xFormers-ROCm**: Only works on CDNA GPUs (MI200/MI300). RX 7900 XTX (RDNA3) NOT supported.
- **Gradient Checkpointing**: Works on all AMD GPUs via PyTorch core. LoRA + checkpointing requires enabling checkpointing BEFORE applying adapters.
- **xFormers memory savings**: ~24% at seq=1024, batch=64. FA2 achieves 2–4× speedup + 10–20× less memory.
- **Gradient checkpointing**: Reduces activation memory 40GB→12GB for Llama 3 8B at seq=2048.

---

## Related Topics

- [gfx-hardware-overrides](../gfx-hardware-overrides/) — Hardware-level AMD GPU optimization (HSA_OVERRIDE_GFX, RDNA3)
