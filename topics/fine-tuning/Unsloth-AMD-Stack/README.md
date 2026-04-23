# Unsloth-AMD-Stack

## 📌 Overview

The Unsloth AMD Stack refers to the software components required to run [Unsloth](https://unsloth.ai) fine-tuning on AMD GPUs via the ROCm/HIP platform. Unsloth enables fine-tuning of LLMs up to 2× faster with ~70% less memory on AMD hardware.

> *"Fine-tune LLMs up to 2x faster with ~70% less memory on AMD hardware, no NVIDIA required."*
> — [Unsloth AMD Docs](https://unsloth.ai/docs/get-started/install/amd)

---

## Topics

| Topic | Description |
|-------|-------------|
| [ROCm-7x](./ROCm-7x.md) | AMD ROCm 7.x: GPU compatibility, PyTorch wheel availability, installation |
| [HIP-SDK](./HIP-SDK.md) | HIP programming: CUDA→HIP translation, hipify tools, PyTorch on AMD |
| [bitsandbytes-amd](./bitsandbytes-amd.md) | 4-bit/8-bit quantization on AMD: ROCm fork, installation, issues |

---

## Stack Overview

```
Unsloth
  ↓
PyTorch (rocm index)
  ↓
ROCm 7.x (6.0+ minimum)
  ↓
HIP SDK / HIP Runtime
  ↓
AMD GPU Driver
  ↓
AMD Hardware (MI300X, RX 7900 XTX, etc.)
```

---

## Quick Install

```bash
# 1. Install ROCm 7.0 or 7.1 (7.2+ no PyTorch wheels yet)
# 2. Install PyTorch
uv pip install "torch>=2.4,<2.11.0" \
  --index-url https://download.pytorch.org/whl/rocm7.1 --upgrade --force-reinstall

# 3. Install Unsloth
uv pip install unsloth[amd]

# 4. Install bitsandbytes pre-release (required)
pip install --force-reinstall --no-cache-dir --no-deps \
  "https://github.com/bitsandbytes-foundation/bitsandbytes/releases/download/continuous-release_main/bitsandbytes-1.33.7.preview-py3-none-manylinux_2_24_x86_64.whl"

# 5. Environment variables
export HSA_OVERRIDE_GFX_VERSION=9.4.2   # For MI300X
export HF_HUB_DISABLE_XET=1            # For all AMD users
```

---

## GPU Compatibility Summary

| GPU | Type | ROCm 7.0 | ROCm 7.2 | bitsandbytes |
|-----|------|----------|----------|-------------|
| **MI300X** (192GB) | Data Center | ✅ | ✅ | ✅ (8-bit) |
| **MI325X** | Data Center | ✅ | ✅ | ✅ (8-bit) |
| **RX 7900 XTX** | Consumer | ⚠️ | ⚠️ | ⚠️ |
| **RX 7800 XT** | Consumer | ✅ | ✅ | ✅ (8-bit) |
| **RX 9070 XT** | Consumer (RDNA4) | ❌ | ✅ | ❌ |
| **RX 9060 XT** | Consumer (RDNA4) | ❌ | ✅ | ❌ |

> *⚠️ = Unofficially works via architecture compatibility*

---

## Known Limitations

| Issue | Severity | Workaround |
|-------|----------|------------|
| 4-bit quantization unstable on AMD | ⚠️ Unsloth disabled | Use 8-bit instead |
| ROCm 7.2+ no PyTorch wheels | 🔴 Must use rocm7.1 index | Wait for wheels |
| Flash Attention 2 unavailable | ℹ️ Xformers fallback | Automatic |
| RX 9070 XT undefined symbols | 🔴 | Wait for ROCm 7.2+ fix |

---

## Key Sources

| Resource | URL |
|----------|-----|
| **Unsloth AMD Install** | [unsloth.ai/docs/get-started/install/amd](https://unsloth.ai/docs/get-started/install/amd) |
| **ROCm Releases** | [github.com/ROCm/ROCm/releases](https://github.com/ROCm/ROCm/releases) |
| **ROCm/bitsandbytes** | [github.com/ROCm/bitsandbytes](https://github.com/ROCm/bitsandbytes) |
| **AMD AI Dev Hub — Unsloth** | [ROCm Unsloth Llama Tutorial](https://rocm.docs.amd.com/projects/ai-developer-hub/en/v3.1/notebooks/fine_tune/unsloth_Llama3_1_8B_GRPO.html) |

---

## Related Topics

- [Acceleration-Frameworks](../Acceleration-Frameworks/) — Fine-tuning frameworks including Unsloth
- [PEFT-Consumer-VRAM](../PEFT-Consumer-VRAM/) — Memory-efficient PEFT methods
