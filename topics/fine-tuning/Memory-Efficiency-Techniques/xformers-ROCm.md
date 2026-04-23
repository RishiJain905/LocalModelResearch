# xFormers-ROCm

## 📌 Overview

| Field | Value |
|-------|-------|
| **Library** | xFormers |
| **Organization** | Meta / Facebook AI |
| **Repo** | [facebookresearch/xformers](https://github.com/facebookresearch/xformers) (10.4k stars) |
| **ROCm Status** | Experimental (v0.0.35+, ROCm 7.1) |
| **Consumer GPU Support** | ❌ RX 7900 XTX NOT supported |
| **Data Center GPU Support** | ✅ MI200, MI250X, MI300X |

> "xFormers is a modular, hackable, and optimized toolbox for Transformer research."

— [Meta xFormers Team](https://github.com/facebookresearch/xformers)

---

## What Is xFormers?

xFormers is Meta's modular Transformer building block library featuring:

- **Memory-efficient exact attention** — up to 10× faster, exact (not approximate)
- **Sparse attention, Block-sparse attention, Sliding window attention**
- **Fused operators** — softmax, linear layer, layer norm, SwiGLU
- **Core API:** `xformers.ops.memory_efficient_attention(query, key, value, attn_bias=None, p=0.0, scale=None)`

> "xFormers is focused on the following values: Field agnostic, Composable, Extensible, Optimized."

— [xFormers Documentation](https://facebookresearch.github.io/xformers/what_is_xformers.html)

---

## ROCm Support History

| Year | Status |
|------|--------|
| 2022 | Meta official: "We do not have plans to support ROCm at the moment." |
| 2024–2025 | AMD forks xFormers to `github.com/ROCm/xformers` |
| Feb 2026 | v0.0.35 ships experimental ROCm 7.1 wheels |

> "Supported CUDA/ROCm variants: [linux only] (EXPERIMENTAL) rocm 7.1 version"

— [xFormers GitHub README](https://github.com/facebookresearch/xformers)

---

## GPU Support Matrix

| GPU | Architecture | xFormers CK Support |
|-----|-------------|-------------------|
| AMD Instinct MI300X | gfx942 (CDNA3) | ✅ Full |
| AMD Instinct MI250X | gfx90a (CDNA2) | ✅ Full |
| AMD Instinct MI210 | gfx908 (CDNA) | ✅ Full |
| AMD Radeon RX 7900 XTX | gfx1100 (RDNA3) | ❌ NOT supported |
| AMD Radeon RX 6800 XT | gfx1030 (RDNA2) | ⚠️ Partial (no CK FMHA) |

> "Currently, xformers on ROCm only works with MI200/MI300. So unfortunately, 7900 XTX will not be able to run it."

— [AMD ROCm maintainer](https://github.com/ROCm/composable_kernel/issues/1171)

---

## Installation

### Official xFormers (experimental ROCm 7.1)
```bash
pip install xformers  # Requires PyTorch 2.10.0+
```

### AMD ROCm Fork
```bash
git clone https://github.com/ROCm/xformers.git
cd xformers
python setup.py install
```

### Via PyTorch ROCm Wheel Index
```bash
uv pip install xformers --index-url https://download.pytorch.org/whl/rocm6.4
```

> "In the past this did not work at all. But it seems to work now! uv pip install xformers --index-url https://download.pytorch.org/whl/rocm6.4"

— [Eric Hartford](https://erichartford.com/practical-ai-with-amd-instinct-mi300x)

---

## Available Operators on AMD ROCm

| Operator | AMD ROCm Status |
|----------|----------------|
| ckF (Composable Kernel Forward) | ✅ Available |
| ckB (Composable Kernel Backward) | ✅ Available |
| ck_decoderF | ✅ Available |
| ck_splitKF | ✅ Available |
| cutlassF-pt | ❌ NOT available |
| cutlassB-pt | ❌ NOT available |
| fa2F (Flash Attention 2) | ❌ NOT available |
| fa3F (Flash Attention 3) | ❌ NOT available |
| triton_splitKF | ✅ Available |

> "On CUDA, XFormers has cutlass operators which do not exist on AMD GPUs. AMD only supports ckF."

— [A1111 Discussion #16782](https://github.com/AUTOMATIC1111/stable-diffusion-webui/discussions/16782)

---

## CK Backend Limitations

| Limitation | Details |
|-----------|---------|
| **Head dimension** | max(query.shape[-1], value.shape[-1]) <= 256 |
| **Data types** | float32 NOT supported — only bfloat16 and float16 |
| **No CUTLASS** | AMD equivalent is CK, but only CDNA archs |
| **No FA2 CK backend** | Flash Attention 2 via CK not available on AMD |

> "ckF is not supported because: max(query.shape[-1], value.shape[-1]) > 256"
> "ckF is not supported because: dtype=torch.float32 (supported: {torch.bfloat16, torch.float16})"

— [A1111 Issue #16490](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/16490)

---

## Memory & Performance

### Memory Savings
| Kernel | Memory vs Naive |
|--------|----------------|
| FA2 Triton FP8 | ~24% savings |
| Torch SDPA (CK) | ~24% savings |
| Flex Attention | ~24% savings |
| Naive attention | Baseline |

> "At sequence length 1024 and batch size 64, efficient kernels save roughly 24% VRAM versus naive attention."

— [ZD Tech Substack](https://zdtech.substack.com/p/the-state-of-flash-attention-on-rocm)

> "FlashAttention achieves a 2-4x speedup and 10-20x less memory usage compared to the standard approach."

— [ZD Tech Substack](https://zdtech.substack.com/p/the-state-of-flash-attention-on-rocm)

### Performance — Oracle OCI 8× MI300X Fine-Tuning
| Metric | Value |
|--------|-------|
| Model | Llama 3 70B (LoRA) |
| Hardware | 8× AMD Instinct MI300X |
| train_samples_per_second | 2.996 |
| eval_samples_per_second | 5.375 |
| train_runtime | ~8.1 hours |

> "Success underscores AMD's ROCm Software Stack as mature and capable for ML training workloads."

— [Oracle OCI Blog](https://blogs.oracle.com/cloud-infrastructure/optimized-performance-results-mi300x-lora-finetuning-llms)

---

## Common Errors

```
ckF is not supported because: max(query.shape[-1], value.shape[-1]) > 256
ckF is not supported because: dtype=torch.float32 (supported: {torch.bfloat16, torch.float16})
fatal error: ck/tensor/tensor_view.hpp file not found
NotImplementedError: Could not run xformers::efficient_attention_forward_ck with arguments from the CUDA backend
```

---

## Ecosystem Integration

### PyTorch
- `xformers.ops.memory_efficient_attention` — xFormers' core attention API
- `torch.nn.functional.scaled_dot_product_attention` (PyTorch 2.3+) dispatches to Flash Attention on ROCm
- ROCm PyTorch 2.2.0+ supports TunableOp for automatic GEMM kernel selection

### Triton
- xFormers uses OpenAI Triton for some kernels
- `triton_splitKF` is available on AMD
- Triton FA2 backend: `FLASH_ATTENTION_TRITON_AMD_ENABLE=TRUE`

### ROCm Composable Kernel (CK)
- Primary AMD backend for xFormers FMHA
- AMD equivalent of NVIDIA CUTLASS
- Supports MI200 (gfx908), MI300 (gfx90a, gfx942)

---

## Limitations Summary

| Issue | Affected GPUs |
|-------|--------------|
| CK only supports CDNA (MI200/MI300) | RDNA GPUs (RX 7900, RX 6800) |
| No float32 in CK | All AMD |
| Head dimension limit (≤256) | All AMD |
| No CUTLASS on AMD | All AMD |
| No FA2 CK backend on AMD | All AMD |
| No Flash Attention 3 | All AMD (Hopper-specific) |

### Ecosystem Issues
| Issue | Description |
|-------|-------------|
| CUDA/ROCm wheel mismatch | Default PyPI xFormers wheels are built for CUDA |
| Dependency clobbering | Axolotl overwrites ROCm dependencies |
| Compile from source | Often requires HIP flags |

> "Axolotl's setup has a bad habit of clobbering all my carefully manually installed dependencies."

— [Eric Hartford](https://erichartford.com/from-zero-to-fineturning-with-axolotl-on-rocm)

---

## Related Topics

- [Gradient-Checkpointing.md](./Gradient-Checkpointing.md) — Memory optimization via activation recomputation
- [gfx-hardware-overrides](./gfx-hardware-overrides/) — HSA_OVERRIDE_GFX, RDNA3 optimization
- [Memory-Efficiency-Techniques](./README.md) — Parent folder overview

---

## References

- [xFormers Official GitHub](https://github.com/facebookresearch/xformers)
- [ROCm/xformers (AMD Fork)](https://github.com/ROCm/xformers)
- [xFormers Official Documentation](https://facebookresearch.github.io/xformers/what_is_xformers.html)
- [ROCm Documentation — Model Acceleration Libraries](https://rocm.docs.amd.com/en/latest/how-to/rocm-for-ai/inference-optimization/model-acceleration-libraries.html)
- [AMD Composable Kernel Issue #1171 — RDNA3 not supported](https://github.com/ROCm/composable_kernel/issues/1171)
- [xFormers Issue #807 — No ROCm plans](https://github.com/facebookresearch/xformers/issues/807)
- [ZD Tech Substack — State of Flash Attention on ROCm](https://zdtech.substack.com/p/the-state-of-flash-attention-on-rocm)
- [Eric Hartford — Practical AI with AMD MI300X](https://erichartford.com/practical-ai-with-amd-instinct-mi300x)
- [Oracle OCI — MI300X LoRA Fine-Tuning Results](https://blogs.oracle.com/cloud-infrastructure/optimized-performance-results-mi300x-lora-finetuning-llms)
