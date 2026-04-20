# FlashAttention-4: The Breakthrough

> **Paper:** [arXiv:2603.05451](https://arxiv.org/abs/2603.05451) · **GitHub:** ⭐23,400 [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) · **Blog:** [Together AI — FlashAttention-4](https://www.together.ai/blog/flashattention-4) · [PyTorch — FlexAttention + FA4](https://pytorch.org/blog/flexattention-flashattention-4-fast-and-flexible/)
> **Released:** March 5, 2026

---

## Overview

> *"On the Blackwell GPU, even though the bottlenecks are quite different, the execution speed of the attention mechanism is now almost as fast as matrix multiplication!"* — Tri Dao

FlashAttention-4 (FA4) is a ground-up redesign for **NVIDIA Blackwell (B200/GB200)** hardware. Where FA3 optimized for Hopper's asymmetric bottlenecks, FA4 exploits Blackwell's fully asynchronous tensor cores and Tensor Memory (TMEM) to achieve near-matmul speeds — **71% of theoretical BF16 peak** on B200.

> *"Attention can now reach a speed close to that of matrix multiplication, which means that the computational bottleneck will completely shift to memory and communication."* — 36kr

---

## Core Problem: Asymmetric Hardware Scaling

From H100 → B200, tensor core throughput **doubled** (1 → 2.25 PFLOPs BF16), but SFU count and shared memory bandwidth remained unchanged:

| Unit | Operations/cycle/SM | H100 → B200 Change |
|------|---------------------|-------------------|
| Tensor Cores (BF16) | 8,192 ops | **2.25× increase** |
| Exponential (SFU) | 16 ops | No change |
| Shared Memory | 128 bytes | No change |

**Result:** On B200, forward pass is bottlenecked by exponential compute; backward pass is bottlenecked by shared memory bandwidth.

---

## FA4 Architecture vs FA3 (Hopper)

| Feature | FlashAttention-3 (Hopper H100) | FlashAttention-4 (Blackwell B200) |
|---------|--------------------------------|-----------------------------------|
| **Tensor Core** | WGMMA (4th gen) | tcgen05.mma (5th gen) |
| **Max MMA Tile** | 64×128×16 | 128×256×16 (single CTA), 256×256×16 (2-CTA) |
| **Accumulation** | Registers | **Tensor Memory (TMEM)** — 256KB/SM |
| **Pipeline** | ~2-stage load/compute overlap | **5-stage warp-specialized** |
| **Exp Method** | MUFU.EX2 (SFU hardware) | **Software emulation** + MUFU.EX2 fallback |
| **Rescaling** | Every new max observed | **Conditional** — ~10× fewer corrections |
| **Language** | C++ (CUDA) | **CuTe-DSL (Python)** |
| **Compile Time** | Minutes | **~2.5 seconds** |
| **Hardware** | SM90 (Hopper) only | **SM100/SM120 (Blackwell) only** |

---

## Key Innovations

### 1. Warp-Specialized 5-Stage Pipeline

FA4 breaks computation into concurrent stages handled by **dedicated warp groups**:

| Warp | Role |
|------|------|
| **Load Warp** | TMA async loads Q/K/V from global → shared memory |
| **MMA Warp** | Tensor Core matmul (Q×K and P×V) |
| **Softmax Warp** (×8 warps) | Normalize S→P, track running stats |
| **Correction Warp** (×4 warps) | Rescale output when normalization changes |
| **Epilogue Warp** (×1-2) | Write output to global memory |

> *"In an async GPU program like FA4, a single warp implements a single transition in a similar state machine."* — Modal Blog

### 2. Software-Emulated Exponential

FA4 replaces hardware SFU exp() with a **cubic polynomial approximation** on FMA units:

```
2^x ≈ p₀ + p₁·x + p₂·x² + p₃·x³
  where p₀=1.0, p₁≈0.6951, p₂≈0.2276, p₃≈0.0771
```

- Only **10–25%** of entries use emulation (tuned per-tile); remainder uses hardware MUFU.EX2
- BF16 relative error: **3.90×10⁻³** — matches hardware within 1 ULP on 99% of inputs
- **Bypasses the SFU bottleneck** entirely

### 3. Conditional Softmax Rescaling

> *"This reduced the number of corrections by a factor of 10."* — Tri Dao at Hot Chips

- FA3: Rescaled **every time** a new maximum was observed
- FA4: Only rescale when `m_j - m_{j-1} > τ` where `τ = log₂(256) = 8.0`
- **~10× reduction** in rescaling operations

### 4. Tensor Memory (TMEM) Integration

256KB per SM on B200 — tightly coupled with tensor cores:
- Stores intermediate results (tS, tP, tO tiles)
- Reduces shared memory traffic significantly
- Enables fully asynchronous MMA (output written directly to TMEM)
- **Backward pass:** TMEM + 2-CTA mode reduces shared memory traffic ~30%

### 5. 2-CTA MMA Mode (Backward Pass)

Two CTAs cooperate on a single MMA operation:
- Spans **256×256×16 tile** (vs 128×256 single-CTA)
- Halves shared memory operand B traffic across five GEMMs
- Halves atomic reductions for dQ gradient

---

## Performance Benchmarks (B200, BF16)

| Configuration | Throughput | vs cuDNN 9.13 | vs Triton |
|---------------|-----------|---------------|----------|
| **FA4 CUDA (CuTeDSL)** | **1605–1613 TFLOPs/s** (71% utilization) | **1.3× faster** | **2.7× faster** |
| FA4 Triton | ~600 TFLOPs/s | baseline | baseline |
| FlashAttention-3 on H100 | ~70% of B200 | — | — |

> *"Perhaps surprisingly, the architecture of FA4 is readily understandable by a general software engineering audience."* — Modal Blog

---

## PyTorch FlexAttention Integration

FA4 powers the **Flash backend** for PyTorch FlexAttention:

```python
from functools import partial
from torch.nn.attention.flex_attention import flex_attention

flex_flash = torch.compile(
    partial(flex_attention, kernel_options={"BACKEND": "FLASH"}),
    dynamic=False
)
```

> *"FA4 kernel work (CuTeDSL implementation, scheduling, score/mask mods, block sparsity) lives upstream in Dao-AILab/flash-attention, while compiler + integration (FlexAttention API, Inductor lowering, CuTeDSL codegen) lives upstream in pytorch/pytorch."*

---

## Key Quotes

> *"On the Blackwell GPU, even though the bottlenecks are quite different, the execution speed of the attention mechanism is now almost as fast as matrix multiplication!"* — **Tri Dao**, Princeton/Together AI

> *"The delta in performance between fully optimized versions is perhaps worth the gain in flexibility on Hopper, but it's a different story on newer Blackwell GPUs."* — **PyTorch Blog**

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Target hardware** | NVIDIA Blackwell (B200, SM100/SM120) only |
| **Key breakthrough** | 5-stage warp-specialized pipeline exploiting asynchronous MMA + TMEM |
| **Performance** | 71% BF16 utilization (1605 TFLOPs/s), 2.7× faster than Triton |
| **Software exp()** | Cubic polynomial on FMA units, bypasses SFU bottleneck |
| **Conditional softmax** | ~10× fewer rescaling operations |
| **Language** | CuTe-DSL (Python) — 20–30× faster compile (2.5s vs 55s) |
| **Backward status** | Limited — missing varlen and GQA support (as of beta8) |

---

*Last updated: 2026-04-20*