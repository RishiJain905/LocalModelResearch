# Memory-efficiency-AMD

## 📌 Overview

This section covers memory efficiency techniques and optimization strategies for running LLM training and inference on AMD GPUs — including ROCm environment variables, RDNA3 consumer GPU support, and memory management techniques.

---

## Subtopics

| File | Topic | Type |
|------|-------|------|
| [HSA_OVERRIDE_GFX.md](./HSA_OVERRIDE_GFX.md) | HSA_OVERRIDE_GFX_VERSION | ROCm environment variable |
| [RDNA3-Optimization.md](./RDNA3-Optimization.md) | RDNA3 Optimization | Consumer GPU ROCm support |

---

## Summary Matrix

| Aspect | Details |
|--------|---------|
| **HSA_OVERRIDE_GFX** | Forces ROCm runtime to treat GPU as supported architecture |
| **RDNA3 Support** | RX 7900 XTX officially supported since ROCm 5.7 |
| **RX 7800 XT** | gfx1102 — community-only, needs HSA_OVERRIDE_GFX |
| **Windows** | Not supported for HSA_OVERRIDE_GFX |
| **vLLM AMD** | First-class since v0.14.0 (January 2026) |

---

## Related Topics

- [PEFT-Consumer-VRAM](./PEFT-Consumer-VRAM/) — LoRA/QLoRA consumer GPU memory optimization
- [Acceleration-Frameworks](./Acceleration-Frameworks/) — Unsloth, custom kernels
