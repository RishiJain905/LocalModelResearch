# ROCm 7.x

## 📌 Overview

**ROCm** (Radeon Open Compute) is AMD's open-source GPU compute platform for Linux. ROCm 7.x is the latest major version series supporting AMD Instinct data center GPUs and AMD Radeon consumer GPUs for AI/ML workloads including Unsloth fine-tuning.

> *"Fine-tune LLMs up to 2x faster with ~70% less memory on AMD hardware, no NVIDIA required."*
> — [Unsloth AMD Docs](https://unsloth.ai/docs/get-started/install/amd)

---

## ROCm 7.x Versions

| Version | Release | Key Additions |
|---------|---------|---------------|
| **7.0.0** | Oct 2025 | MI355X/MI350X support, KVM Passthrough |
| **7.1.0/7.1.1** | Late 2025 | RHEL 10.0 KVM SR-IOV, Debian 13 extended |
| **7.2.0** | Jan 21, 2026 | RX 9000 (RDNA4) support, Ryzen AI 300/400, Windows PyTorch preview, ROCm Optiq |

> *"ROCm 7.0.0 adds support for AMD Instinct MI355X and MI350X."*
> — [ROCm Releases](https://github.com/ROCm/ROCm/releases)

> *"ROCm 7.2.0 released January 21st, 2026."*
> — [AMD ROCm](https://github.com/ROCm/ROCm/releases)

---

## ⚠️ Critical: PyTorch Wheel Availability

**PyTorch wheels for ROCm 7.2+ do not exist yet.**

> *"If ROCm version is 7.2 or higher, use `rocm7.1` in the command above — **no PyTorch wheels exist yet for 7.2+**."*
> — [Unsloth AMD Install](https://unsloth.ai/docs/get-started/install/amd)

```bash
# Install command for ROCm 7.2+
uv pip install "torch>=2.4,<2.11.0" "torchvision<0.26.0" "torchaudio<2.11.0" \
  --index-url https://download.pytorch.org/whl/rocm7.1 --upgrade --force-reinstall
```

**Available PyTorch wheels (via official ROCm Docker images, ROCm 7.2.1):**

| PyTorch Version | Ubuntu 24.04 | Ubuntu 22.04 |
|-----------------|--------------|--------------|
| 2.9.1 | ✅ | ✅ |
| 2.8.0 | ✅ | ✅ |
| 2.7.1 | ✅ | ✅ |

> *"PyTorch wheels for ROCm 7.2+ do not exist as of research date."*
> — [ROCm Docs](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/3rd-party/pytorch-install.html)

---

## GPU Compatibility

### AMD Instinct (Data Center)

| GPU | Architecture | ROCm 7.0 | ROCm 7.1 | ROCm 7.2 |
|-----|-------------|----------|----------|----------|
| MI355X (288GB) | CDNA4 (gfx950) | ✅ | ✅ | ✅ |
| MI350X | CDNA4 (gfx950) | ✅ | ✅ | ✅ |
| MI325X | CDNA3 (gfx942) | ✅ | ✅ | ✅ |
| MI300X (192GB) | CDNA3 (gfx942) | ✅ | ✅ | ✅ |
| MI300A | CDNA3 (gfx942) | ✅ | ✅ | ✅ |
| MI250X | CDNA2 (gfx90a) | ✅ | ✅ | ✅ |

### AMD Radeon (Consumer)

| GPU | Architecture | ROCm 7.0 | ROCm 7.1 | ROCm 7.2 |
|-----|-------------|----------|----------|----------|
| RX 9070 XT | RDNA4 (gfx1201) | ❌ | ❌ | ✅ |
| RX 9070 | RDNA4 (gfx1201) | ❌ | ❌ | ✅ |
| RX 9060 XT | RDNA4 (gfx1200) | ❌ | ❌ | ✅ |
| RX 7900 XTX | RDNA3 (gfx1100) | ⚠️ | ⚠️ | ⚠️ |
| RX 7800 XT | RDNA3 (gfx1101) | ✅ | ✅ | ✅ |
| RX 7700 XT | RDNA3 (gfx1101) | ✅ | ✅ | ✅ |

> *"RX 7900 XTX: Not explicitly listed in official compatibility matrix, but shares architecture (gfx1100) with PRO W7900. Community reports it works via compatible GPU overrides."*
> — [ROCm System Requirements](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/reference/system-requirements.html)

---

## OS Support

| OS | ROCm 7.0 | ROCm 7.1 | ROCm 7.2 |
|----|----------|----------|----------|
| Ubuntu 24.04 | ✅ | ✅ | ✅ |
| Ubuntu 22.04 | ✅ | ✅ | ✅ |
| RHEL 10.x | ✅ | ✅ | ✅ |
| RHEL 9.x | ✅ | ✅ | ✅ |
| Debian 13 | ✅ | ✅ | ✅ |
| Debian 12 | ✅ | ✅ | ✅ |
| Rocky Linux 9 | ✅ | ✅ | ✅ |

---

## Unsloth-Specific Configuration

### Required Environment Variables

**For MI300X users:**
```bash
export HSA_OVERRIDE_GFX_VERSION=9.4.2
```

> *"HSA_OVERRIDE_GFX_VERSION=9.4.2 tells ROCm to treat your GPU as gfx942 (MI300X). Without this, some kernels may fail to compile or run."*
> — [Unsloth AMD Docs](https://unsloth.ai/docs/get-started/install/amd)

**For all AMD users (HuggingFace download fix):**
```bash
export HF_HUB_DISABLE_XET=1
```

### Attention Backend

> *"On AMD GPUs, Flash Attention 2 is not available. Unsloth automatically falls back to Xformers, which provides equivalent performance on ROCm."*
> — [Unsloth AMD Docs](https://unsloth.ai/docs/get-started/install/amd)

---

## ROCm 7.0 Key Features

> *"AMD Instinct MI355X platform (8x GPU) with FP4 precision delivered up to 1.3× better inference throughput than an NVIDIA B200 8x GPU platform on the DeepSeek R1 model."*
> — [AMD Blog](https://www.amd.com/en/blogs/2025/rocm7-supercharging-ai-and-hpc-infrastructure.html)

- **FP4 precision** support on MI355X
- **Kubernetes & MLOps integration** with container orchestration, multi-tenant support, RBAC
- **AI Tensor Engine for ROCm (AITER)** software integration

---

## ROCm 7.2 Key Features

> *"ROCm 7.2 brings consumer RDNA4 GPUs (RX 9070/9060) into official support."*
> — [AMD ROCm What's New](https://www.amd.com/en/products/software/rocm/whats-new.html)

- **Windows PyTorch support** (preview) for RX 7000/9000 and Ryzen AI
- **Ryzen AI 300/400 series** support including up to 128GB shared memory APUs
- **Radeon RX 9000 series** (RDNA4) official support
- **ROCm Optiq** introduced for optimized inference

---

## Sources

| Type | Citation |
|------|----------|
| **Docs** | [Unsloth AMD Installation](https://unsloth.ai/docs/get-started/install/amd) |
| **Docs** | [ROCm System Requirements](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/reference/system-requirements.html) |
| **Blog** | [AMD ROCm 7 Blog](https://www.amd.com/en/blogs/2025/rocm7-supercharging-ai-and-hpc-infrastructure.html) |
| **Blog** | [AMD ROCm What's New](https://www.amd.com/en/products/software/rocm/whats-new.html) |
| **GitHub** | [ROCm Releases](https://github.com/ROCm/ROCm/releases) |
| **Docs** | [ROCm PyTorch Installation](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/install/3rd-party/pytorch-install.html) |

---

## Related Topics

- [HIP-SDK](./HIP-SDK.md) — HIP programming and CUDA porting tools
- [bitsandbytes-amd](./bitsandbytes-amd.md) — 4-bit/8-bit quantization on AMD
