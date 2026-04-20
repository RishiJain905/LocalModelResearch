# FlashAttention-4 (FA4)

> Fast and accurate attention on NVIDIA Blackwell GPUs — up to **1605 TFLOPs/s** (71% BF16 utilization)

**📄 Paper:** [arXiv:2603.05451](https://arxiv.org/abs/2603.05451) · **⭐ GitHub:** [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) (23,400 stars) · **📦 PyPI:** [flash-attn-4](https://pypi.org/project/flash-attn-4/)

---

## Overview

FlashAttention-4 is a ground-up redesign for **NVIDIA Blackwell (B200/GB200)** hardware. Released March 5, 2026, FA4 exploits Blackwell's fully asynchronous tensor cores (tcgen05.mma), Tensor Memory (TMEM), and new kernel fusion techniques to achieve near-matmul speeds for attention.

> *"On the Blackwell GPU, even though the bottlenecks are quite different, the execution speed of the attention mechanism is now almost as fast as matrix multiplication!"* — **Tri Dao**

---

## Contents

| Document | Description |
|----------|-------------|
| **[The-Breakthrough.md](./The-Breakthrough.md)** | FA4's core innovations vs FA3 — 5-stage pipeline, software exp(), conditional softmax, TMEM, architecture comparison |
| **[Kernel-Pipelining.md](./Kernel-Pipelining.md)** | Deep dive into kernel design — warp-specialized stages, why Triton fails, WGMMA vs TCGEN05, fusion techniques |
| **[Start-to-Now.md](./Start-to-Now.md)** | Complete FA1→FA2→FA3→FA4 evolution, release timeline, FA3→FA4 migration guide, API changes, deprecations |

---

## Key Facts

| Property | Value |
|----------|-------|
| **Target GPU** | NVIDIA Blackwell (SM100/SM120) only |
| **Released** | March 5, 2026 (beta0) |
| **Package** | `flash-attn-4` (separate from `flash-attn`) |
| **Language** | CuTe-DSL (Python) — 20–30× faster compile than C++ |
| **Performance** | 1605–1613 TFLOPs/s BF16 (71% utilization on B200) |
| **vs Triton** | 2.7× faster |
| **vs cuDNN 9.13** | 1.3× faster |
| **Compile time** | ~2.5 seconds |
| **Backward status** | Incomplete — missing varlen, GQA support |

---

## Architecture Comparison: FA3 vs FA4

| Feature | FA3 (Hopper H100) | FA4 (Blackwell B200) |
|---------|------------------|---------------------|
| **Tensor Core** | WGMMA (4th gen) | tcgen05.mma (5th gen) |
| **Pipeline** | ~2-stage | **5-stage warp-specialized** |
| **Exp method** | SFU MUFU.EX2 | **Software emulation** + MUFU.EX2 |
| **Accumulation** | Registers | **Tensor Memory (TMEM)** |
| **Rescaling** | Every new max | **Conditional** (~10× fewer) |
| **Language** | C++ (CUDA) | **CuTe-DSL (Python)** |
| **Hardware** | SM90 only | SM100/SM120 only |

---

## FA4 Core Innovations

1. **5-Stage Warp-Specialized Pipeline** — Dedicated warp groups for load/MMA/softmax/correction/store
2. **Software-Emulated Exponential** — Cubic polynomial on FMA units bypasses SFU bottleneck
3. **Conditional Softmax Rescaling** — ~10× fewer rescaling operations
4. **Tensor Memory (TMEM)** — 256KB/SM for intermediate storage, reduces SMEM traffic
5. **2-CTA MMA Mode** — Pairs of CTAs cooperate for 256×256×16 tiles

---

## External Resources

| Resource | Link |
|----------|------|
| **FA4 Paper** | [arXiv:2603.05451](https://arxiv.org/abs/2603.05451) |
| **FA4 HTML Paper** | [arXiv HTML](https://arxiv.org/html/2603.05451v1) |
| **Together AI Blog** | [together.ai/blog/flashattention-4](https://www.together.ai/blog/flashattention-4) |
| **PyTorch Blog** | [pytorch.org/blog/flexattention-flashattention-4-fast-and-flexible](https://pytorch.org/blog/flexattention-flashattention-4-fast-and-flexible/) |
| **Modal Reverse-Engineering** | [modal.com/blog/reverse-engineer-flash-attention-4](https://modal.com/blog/reverse-engineer-flash-attention-4) |
| **DigitalOcean Tutorial** | [digitalocean.com/community/tutorials/flashattention-4-llm-inference-optimization](https://www.digitalocean.com/community/tutorials/flashattention-4-llm-inference-optimization) |
| **GitHub Repo** | [github.com/Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) |
| **FA4 PyPI** | [pypi.org/project/flash-attn-4](https://pypi.org/project/flash-attn-4/) |

---

*Part of the [Context-Windows](../) research track. See also: [Long-Context-Stability](../Long-Context-Stability/), [KV-Cache-Management](../KV-Cache-Management/), [Managed-Agents-Decoupled-Memory](../Managed-Agents-Decoupled-Memory/)*