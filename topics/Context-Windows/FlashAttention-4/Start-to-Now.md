# FlashAttention-4: Start to Now (FA3 → FA4 Migration)

> **Paper:** [arXiv:2603.05451](https://arxiv.org/abs/2603.05451) · **GitHub:** [Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention) · **PyPI:** [flash-attn-4](https://pypi.org/project/flash-attn-4/)
> **Releases:** [GitHub Releases](https://github.com/Dao-AILab/flash-attention/releases)

---

## Overview

FA4 represents a **breaking change** from FA2/FA3 — different package name, different import paths, and **hardware incompatibility**. This doc covers the complete evolution, release timeline, and migration from FA3 to FA4.

> *"FA3 does NOT work on Blackwell — uses WGMMA instructions deprecated on SM10.x."* — [GitHub Issue #2003](https://github.com/Dao-AILab/flash-attention/issues/2003)

---

## Complete Version History

| Version | Year | Target GPU | Key Innovations | Package |
|---------|------|-----------|----------------|---------|
| **FA1** | 2022 | Ampere (A100) | IO-aware tiling, on-chip buffering, exact attention | `flash-attn` |
| **FA2** | 2023 | Ampere/Ada (A100, RTX 30/40) | Warp scheduling, sliced-Q, MQA/GQA, head dim up to 256 | `flash-attn` (v2.x) |
| **FA3** | Jul 2024 | Hopper (H100/H800) | Async MMA via TMA, warp-specialization, interleave GEMM/softmax, FP8 | `flash-attn` + env var |
| **FA4** | Mar 2026 | **Blackwell (B200, SM100/SM120)** | 5-stage pipeline, software exp, conditional softmax, TMEM, 2-CTA MMA | **`flash-attn-4`** (separate package) |

---

## FlashAttention-4 Release Timeline

**Initial release: March 5, 2026** (`fa4-v4.0.0.beta0`)

| Beta Release | Date | Key Additions |
|-------------|------|---------------|
| **4.0.0b0** | Mar 5, 2026 | Initial Blackwell (SM100) support — forward + backward, CUDA 13, CUTLASS v4.3.0, Paged attention, SplitKV, GQA for SM100 backward, 45+ contributors |
| **4.0.0b4** | Mar 5, 2026 | Stable tagged release |
| **4.0.0b6** | Mar 25, 2026 | SM120 (Blackwell GeForce / DGX Spark), varlen for SM90, AMD ROCm RDNA 3/4 |
| **4.0.0b7** | Apr 1, 2026 | FA4 CI on B200 runner, LSE accum strides fix, deterministic bwd for SM100 |
| **4.0.0b8** | Apr 8, 2026 | Latest beta, ROCm NaN fix, CI extensions |
| **4.0.0b9** | (latest) | Current pre-release on PyPI |

**Source:** [GitHub Releases — Dao-AILab/flash-attention](https://github.com/Dao-AILab/flash-attention/releases)

---

## Breaking Change: Different Package Names

**FA4 uses a separate Python package** from FA2/FA3:

| Version | Package Name | Import Statement |
|---------|-------------|-----------------|
| **FA2 / FA3** | `flash-attn` (PyPI) | `from flash_attn import flash_attn_func` |
| **FA4** | `flash-attn-4` (PyPI) | `from flash_attn.cute import flash_attn_func, flash_attn_varlen_func` |

### FA4 Installation

```bash
pip install flash-attn-4              # CUDA 12
pip install "flash-attn-4[cu13]"     # CUDA 13 (B200)
pip install -e "flash_attn/cute[dev,cu13]"  # Dev install from source
```

### FA4 Import Structure

```python
# FA4 API
from flash_attn.cute import flash_attn_func, flash_attn_varlen_func

# Forward pass
out = flash_attn_func(q, k, v, causal=True)

# Variable-length sequences (varlen)
from flash_attn.cute import flash_attn_varlen_func
```

---

## Deprecations & Missing Features

### FA4 Backward Pass Limitations (as of beta8, April 2026)

| Feature | Forward Pass | Backward Pass |
|---------|-------------|---------------|
| Fixed-length standard attention | ✅ Available | ✅ Available |
| Variable-length sequences (varlen) | ✅ SM100 | ❌ Missing |
| GQA/MQA | ✅ SM100 | ❌ Missing |
| Paged KV attention | ✅ SM100 | ⚠️ Under development |
| Deterministic mode | ✅ | ✅ |
| Block sparsity / FlexAttention | ✅ | ⚠️ Partial |

**Source:** [GitHub Issue #1989 — FA4 backward pass missing varlen/GQA](https://github.com/Dao-AILab/flash-attention/issues/1989)

### FA3 Cannot Run on Blackwell

> *"FA3 does NOT work on Blackwell — uses WGMMA instructions deprecated on SM10.x."* — [GitHub Issue #2003](https://github.com/Dao-AILab/flash-attention/issues/2003)

- FA3 requires **Hopper (SM90)**
- FA4 requires **Blackwell (SM100/SM120)**
- No cross-compatibility

---

## Migration Guide: FA3 → FA4

### Step 1: Check Hardware

FA4 **only runs on Blackwell**. Verify your GPU:

```bash
nvidia-smi | grep -i blackwell
# or
python -c "import torch; print(torch.cuda.get_device_capability())"
# SM100 = 10.0, SM120 = 12.0 (both Blackwell)
```

> **Do NOT install FA4 on Hopper systems** — FA3 cannot be loaded if FA4 is installed.

### Step 2: Install FA4 Package

```bash
pip install "flash-attn-4[cu13]"  # CUDA 13 for B200
```

### Step 3: Update Import Paths

```python
# ❌ Old (FA2/FA3):
from flash_attn import flash_attn_func

# ✅ New (FA4):
from flash_attn.cute import flash_attn_func
```

### Step 4: Check Feature Support

For **inference-only** — FA4 forward is fully functional.

For **training** — FA4 backward pass is incomplete. Use FA3 on Hopper for training workloads.

Fallback required for:
- Variable-length sequences (varlen)
- GQA/MQA models (backward pass)
- Paged KV attention (backward)

### Step 5: Keep Both Packages Available

Both `flash-attn` (FA2/FA3) and `flash-attn-4` can be installed simultaneously:

```bash
pip install flash-attn          # FA2/FA3 for Hopper
pip install flash-attn-4       # FA4 for Blackwell
```

> **Note:** Coexistence issues have been reported and fixed ([PR #2331](https://github.com/Dao-AILab/flash-attention/pull/2331)).

---

## Feature Comparison: FA2 vs FA3 vs FA4

| Feature | FA2 | FA3 | FA4 |
|---------|-----|-----|-----|
| **Target GPU** | Ampere/Ada | Hopper | Blackwell |
| **Max Head Dim** | 256 | Any | Any |
| **MQA/GQA** | ✅ | ✅ | ✅ (forward) |
| **FP8** | ❌ | ✅ | ❌ |
| **SplitKV** | ✅ | ✅ | ✅ |
| **Paged Attention** | ❌ | ✅ | ✅ |
| **Varlen** | ✅ | ✅ | ✅ (fwd only) |
| **Deterministic** | ❌ | ✅ | ✅ |
| **Block Sparsity** | ❌ | ❌ | ✅ |
| **Triton Backend** | ❌ | ✅ | ❌ (CUDA only) |
| **Compile Time** | Minutes | Minutes | **~2.5 seconds** |
| **Language** | CUDA C++ | CUDA C++ | **CuTe-DSL (Python)** |

---

## Key Resources

| Resource | URL |
|----------|-----|
| **FA4 Paper** | [arXiv:2603.05451](https://arxiv.org/abs/2603.05451) |
| **FA4 HTML Paper** | [arXiv HTML](https://arxiv.org/html/2603.05451v1) |
| **FA4 PyPI Package** | [pypi.org/project/flash-attn-4](https://pypi.org/project/flash-attn-4/) |
| **FA3 Paper** | [arXiv:2407.08608](https://arxiv.org/abs/2407.08608) |
| **FA3 PyTorch Blog** | [pytorch.org/blog/flashattention-3](https://pytorch.org/blog/flashattention-3/) |
| **FA2 Blog (Hazy Research)** | [hazyresearch.stanford.edu/blog/2023-07-17-flash2](https://hazyresearch.stanford.edu/blog/2023-07-17-flash2) |
| **Modal: Reverse-Engineered FA4** | [modal.com/blog/reverse-engineer-flash-attention-4](https://modal.com/blog/reverse-engineer-flash-attention-4) |
| **Together.ai: FA4 Technical Blog** | [together.ai/blog/flashattention-4](https://www.together.ai/blog/flashattention-4) |
| **DigitalOcean: FA4 Tutorial** | [digitalocean.com/community/tutorials/flashattention-4-llm-inference-optimization](https://www.digitalocean.com/community/tutorials/flashattention-4-llm-inference-optimization) |
| **GitHub Releases** | [github.com/Dao-AILab/flash-attention/releases](https://github.com/Dao-AILab/flash-attention/releases) |
| **FA4 Backward Missing Issues** | [Issue #1989](https://github.com/Dao-AILab/flash-attention/issues/1989) |

---

## Summary

| Aspect | Description |
|--------|-------------|
| **FA4 released** | March 5, 2026 |
| **Hardware** | Blackwell only (SM100/SM120) |
| **Package** | `flash-attn-4` (separate from `flash-attn`) |
| **Import change** | `flash_attn.cute` instead of `flash_attn` |
| **FA3 status** | Incompatible with Blackwell |
| **Backward pass** | Incomplete — missing varlen, GQA |
| **Key advantage** | 2.7× faster than Triton, 20–30× faster compile |

---

*Last updated: 2026-04-20*