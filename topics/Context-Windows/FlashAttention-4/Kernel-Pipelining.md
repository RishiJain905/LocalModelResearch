# FlashAttention-4: Kernel Pipelining & Triton Backend

> **Paper:** [arXiv:2603.05451](https://arxiv.org/abs/2603.05451) · **GitHub:** ⭐23,400 [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) · **Blog:** [Modal — Reverse-Engineered FA4](https://modal.com/blog/reverse-engineer-flash-attention-4) · [PyTorch — FlexAttention + FA4](https://pytorch.org/blog/flexattention-flashattention-4-fast-and-flexible/)
> **Analysis:** [Together AI — FA4 Technical Deep-Dive](https://www.together.ai/blog/flashattention-4)

---

## Overview

> *"High-performance attention on Blackwell requires deeply pipelined, warp-specialized kernels — not expressible in Triton."* — PyTorch Blog

FA4's performance gains come from **kernel-level redesign** exploiting Blackwell's architectural features that cannot be expressed in higher-level frameworks. This doc covers the 5-stage pipeline mechanics, why Triton falls behind, and the CuTe-DSL implementation.

---

## FlashAttention Evolution

| Version | Year | Target | Key Innovation |
|---------|------|--------|----------------|
| **FA1** | 2022 | Ampere (A100) | IO-aware tiling + on-chip buffering |
| **FA2** | 2023 | Ampere/Ada | Warp scheduling, sliced-Q, MQA/GQA |
| **FA3** | 2024 | Hopper (H100) | Async MMA via TMA, warp specialization |
| **FA4** | 2026 | **Blackwell (B200)** | 5-stage pipeline, TMEM, software exp, conditional softmax |

---

## The 5-Stage Warp-Specialized Pipeline

FA4 breaks attention into **5 concurrent stages** handled by dedicated warp groups:

```
Global Memory → Shared Memory → MMA → Softmax → TMEM → Global Memory
     [Load]        [MMA]     [Softmax]  [Correction]  [Store]
```

### Stage Details

| Stage | Warp Count | Operation |
|-------|-----------|-----------|
| **1. Load** | 1 warp | TMA async loads Q/K/V tiles from global → shared memory |
| **2. MMA** | 1 warp | Tensor Core matmul: Q×K^T (score), P×V (output) |
| **3. Softmax** | 8 warps | Normalize S→P, track running max/sum statistics |
| **4. Correction** | 4 warps | Rescale output when normalization changes |
| **5. Epilogue** | 1-2 warps | Write output tiles to global memory |

### Ping-Pong Q Tile Schedule

Each CTA processes **two Q tiles alternately**:
- While Tile A runs tensor cores → Tile B computes softmax
- Two separate softmax warpgroups (one per Q tile) with explicit synchronization
- **Avoids MUFU.EX2 contention** between softmax operations

---

## Why Triton Falls Behind on Blackwell

| Metric | FA4 CUDA (CuTeDSL) | Triton |
|--------|-------------------|--------|
| **Peak Throughput (B200 BF16)** | **1605 TFLOPs/s** | ~600 TFLOPs/s |
| **vs FA4** | baseline | **2.7× slower** |

### Why Triton Cannot Express FA4

| Limitation | Impact |
|-----------|--------|
| **No warp specialization** | Cannot assign fixed warp roles per stage |
| **No TMEM access** | Cannot store intermediates in 256KB/SM Tensor Memory |
| **No inline PTX for tcgen05.mma** | Cannot generate 5th-gen tensor core instructions |
| **No 2-CTA MMA coordination** | Cannot pair CTAs for 256×256 MMA tiles |
| **Limited async execution** | Cannot overlap MMA, softmax, memory at required granularity |

> *"High-performance attention on Blackwell requires deeply pipelined, warp-specialized kernels — not expressible in Triton."* — PyTorch Blog

### FA4 Triton Backend Status

FA4 forward is **not available in Triton** — it requires CUDA-level kernel design:
- FA3 has a Triton implementation: `flash_attn/flash_attn_triton.py`
- FA4 is `flash_attn/cute/flash_fwd_sm100.py` (CuTe-DSL)
- **Triton is suitable for prototyping but not production FA4 on Blackwell**

---

## WGMMA vs TCGEN05 (Hopper vs Blackwell)

| Feature | Hopper WGMMA (FA3) | Blackwell TCGEN05 (FA4) |
|---------|-------------------|------------------------|
| **Max MMA Tile** | 64×128×16 | 128×256×16 (single CTA), 256×256×16 (2-CTA) |
| **Accumulation** | Registers | **Tensor Memory (TMEM)** |
| **Async MMA** | Partial | **Fully async, output to TMEM** |
| **Operand B sourcing** | Shared memory | **TMEM** (enables MMA chaining) |
| **Instruction** | `wgmma.mma_async` | `tcgen05.mma` |

### Key Blackwell Advantage: MMA Chaining

Because MMA output can write directly to TMEM, subsequent MMA instructions can **chain** without waiting for full tile completion — FA4 can interleave Q×K^T and P×V GEMMs more efficiently.

---

## Key Fusion Techniques

### 1. Software Emulated Exponential

On B200, SFU count didn't scale with tensor cores:
- B200 SFU: **16 ops/cycle** vs tensor core **8,192 ops/cycle** (512× gap)
- FA4 uses **cubic polynomial** on FMA units instead:

```python
# 2^x ≈ p0 + p1*x + p2*x² + p3*x³
p0, p1, p2, p3 = 1.0, 0.69514614, 0.22756439, 0.07711909
```

- Only 10–25% of entries use emulation; rest uses hardware MUFU.EX2
- BF16 relative error: **3.90×10⁻³**

### 2. Conditional Online Softmax Rescaling

FA3: Rescale **every iteration** when new max observed

FA4: Rescale only when `m_j - m_{j-1} > τ` (τ = log₂(256) = 8.0)

> *"Reduced the number of corrections by a factor of 10."* — Tri Dao

### 3. TMEM Intermediate Storage

Stores S (scores), P (probabilities), O (output) in **Tensor Memory**:
- Relieves register pressure from large tile sizes
- Enables overlap: dQ/dK MMAs from previous iteration overlap with current softmax
- **Backward pass bottleneck shift:** Shared memory traffic 3328 → 2688 cycles/tile

### 4. 2-CTA MMA Mode

Two CTAs cooperatively execute a single MMA spanning both TMEMs:
- Tile: **256×256×16** (M and N split across pair)
- **Halves shared memory operand B traffic** across five GEMMs
- **Halves atomic reductions** for dQ gradient

---

## CuTe-DSL Implementation

FA4 is implemented **entirely in CuTe-DSL** embedded in Python — no CUDA C++:

```python
# FA4 forward source
flash_attn/cute/flash_fwd_sm100.py

# Compiles: Python → PTX → SASS
# Compile time: ~2.5 seconds (vs 55s for C++ template FA3)
```

**Advantages of CuTe-DSL:**
- Python-embedded DSL → faster development and iteration
- 20–30× faster compile times
- JIT-style workflow support
- **Disadvantage:** Requires Blackwell-specific hardware knowledge to use directly

---

## PyTorch FlexAttention Integration

FA4 powers the **Flash backend** for PyTorch FlexAttention:

```python
from functools import partial
from torch.nn.attention.flex_attention import flex_attention

# JIT-compile with FA4 backend
flex_flash = torch.compile(
    partial(flex_attention, kernel_options={"BACKEND": "FLASH"}),
    dynamic=False
)
```

PyTorch auto-generates CuTeDSL score/mask modification functions, JIT-instantiating FA4 for custom attention variants.

---

## Current Status

| Feature | Status |
|---------|--------|
| **Hardware** | Blackwell (SM100/SM120) only |
| **Forward Pass** | ✅ Functional |
| **Backward Pass** | ⚠️ Limited (missing varlen, GQA) |
| **PyTorch SDPA** | ⚠️ Not in stable releases |
| **vLLM** | ❌ Not supported (FA4 throws error on non-Blackwell) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **Core innovation** | 5-stage warp-specialized pipeline exploiting Blackwell's asynchronous MMA |
| **Why Triton fails** | Cannot express warp specialization, TMEM access, or 2-CTA MMA |
| **FA4 speedup** | 2.7× over Triton, 1.3× over cuDNN 9.13 on B200 |
| **Key techniques** | Software exp (bypasses SFU), conditional softmax (10× fewer rescalings), TMEM intermediates |
| **Language** | CuTe-DSL (Python) — 20–30× faster compile than C++ |
| **Backward status** | Incomplete — missing varlen and GQA support |

---

*Last updated: 2026-04-20*