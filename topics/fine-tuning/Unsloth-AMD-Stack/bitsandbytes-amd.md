# bitsandbytes-amd

## 📌 Overview

**bitsandbytes** on AMD GPUs enables 4-bit and 8-bit quantization for memory-efficient LLM inference and fine-tuning. AMD's ROCm platform has official support via the upstream `bitsandbytes-foundation/bitsandbytes` multi-backend refactor and AMD's official fork at `ROCm/bitsandbytes`.

> *"8-bit CUDA functions for PyTorch. Enables accessible large language models via k-bit quantization for PyTorch."*
> — [ROCm/bitsandbytes GitHub](https://github.com/ROCm/bitsandbytes)

---

## AMD Support Status

### ⚠️ Unsloth 4-bit Quantization Disabled on AMD

> *"Unsloth: AMD currently is not stable with 4bit bitsandbytes. Disabling for now."*
> — [Unsloth GitHub Issue #4133](https://github.com/unslothai/unsloth/issues/4133)

> *"All ROCm systems need a pre-release bitsandbytes build. Versions ≤ 0.49.2 have a 4-bit decode NaN bug on every AMD GPU."*
> — [Unsloth AMD Docs](https://unsloth.ai/docs/get-started/install/amd)

### 8-bit Support

8-bit quantization (`Linear8bitLt`) is more stable than 4-bit on AMD ROCm.

---

## Installation

### Pre-release (Required for AMD)

```bash
pip install --force-reinstall --no-cache-dir --no-deps \
  "https://github.com/bitsandbytes-foundation/bitsandbytes/releases/download/continuous-release_main/bitsandbytes-1.33.7.preview-py3-none-manylinux_2_24_x86_64.whl"
```

> *"All ROCm systems need a pre-release bitsandbytes build (≥0.49.1) due to a 4-bit decode NaN bug on every AMD GPU in older versions."*
> — [Unsloth AMD Docs](https://unsloth.ai/docs/get-started/install/amd)

### Build from ROCm Fork

```bash
git clone --recurse https://github.com/ROCm/bitsandbytes.git
cd bitsandbytes
git checkout rocm_enabled
pip install -r requirements.txt
cmake -DCOMPUTE_BACKEND=hip -DBNB_ROCM_ARCH="gfx1100" -S .
make -j$(nproc) && pip install .
```

### ROCm Architecture Flags

| Architecture | GPU Examples | Flag |
|-------------|--------------|------|
| gfx1030 | RDNA 2 (RX 6000 series) | `gfx1030` |
| gfx1100 | RDNA 3 (RX 7900, RX 7800, RX 7600) | `gfx1100` |
| gfx1200 | RX 9060 XT (RDNA 3.5) | `gfx1200` |
| gfx1201 | RX 9070 XT/9070 (RDNA 4) | `gfx1201` |
| gfx90a | CDNA 2 (MI250X) | `gfx90a` |
| gfx942 | CDNA 3 (MI300X) | `gfx942` |

---

## Supported AMD GPUs

### Official Support (ROCm/bitsandbytes)

| Architecture | GPUs |
|-------------|------|
| **CDNA (gfx90a)** | MI250X |
| **CDNA (gfx942)** | MI300X, MI325X, MI350X, MI355X |
| **RDNA (gfx1100)** | RX 7900 XTX, RX 7800 XT, RX 7700 XT, RX 7600 XT |

> *"Supported AMD GPUs: CDNA: gfx90a, gfx942 (MI250, MI300 series); RDNA: gfx1100 (RX 7900 XTX, RX 7800 XT, RX 7600 XT)."*
> — [ROCm/bitsandbytes](https://github.com/ROCm/bitsandbytes)

---

## Reported Issues

### RX 7600 XT (gfx1100)

Partial/unstable with ROCm 6.0.2, 6.1, 6.2.

### RX 9070 XT (gfx1200)

> *"Undefined symbol errors with ROCm 6.3.3/6.4."*
> — [GitHub Issues](https://github.com/ROCm/bitsandbytes/issues)

### ROCm Version Misidentification

ROCm version detected as CUDA in some version detection scenarios.

### gfx1200 (Newer RDNA 3.5)

Not yet properly supported.

---

## bitsandbytes-rocmlite

For ROCm 6.0+, AMD also provides:

```bash
pip install bitsandbytes-rocmlite
```

> *"To install bitsandbytes for ROCm 6.0 (and later), use: `pip install bitsandbytes-rocmlite`."*
> — [ROCm Docs](https://rocm.docs.amd.com/)

---

## Key Components

| Component | Description |
|-----------|-------------|
| `bitsandbytes.nn.Linear8bitLt` | 8-bit quantization |
| `bitsandbytes.nn.Linear4bit` | 4-bit quantization (unstable on AMD) |
| `bitsandbytes.optim` | 8-bit optimizers |

---

## Multi-Backend Refactor

> *"This space is intended to receive feedback from users that are willing to help us by alpha testing the current implementation of the AMD ROCm backend."*
> — [bitsandbytes Discussion #1339](https://github.com/bitsandbytes-foundation/bitsandbytes/discussions/1339)

The upstream `bitsandbytes-foundation/bitsandbytes` multi-backend refactor branch has added AMD ROCm support and is now integrated into main.

---

## Key Resources

| Resource | URL |
|----------|-----|
| **ROCm/bitsandbytes Fork** | [github.com/ROCm/bitsandbytes](https://github.com/ROCm/bitsandbytes) |
| **bitsandbytes Multi-Backend** | [bitsandbytes-foundation/bitsandbytes](https://github.com/bitsandbytes-foundation/bitsandbytes) |
| **Unsloth AMD Install** | [unsloth.ai/docs/get-started/install/amd](https://unsloth.ai/docs/get-started/install/amd) |

---

## Sources

| Type | Citation |
|------|----------|
| **GitHub** | [ROCm/bitsandbytes](https://github.com/ROCm/bitsandbytes) |
| **GitHub** | [bitsandbytes-foundation/bitsandbytes](https://github.com/bitsandbytes-foundation/bitsandbytes) |
| **Discussion** | [bitsandbytes ROCm Backend](https://github.com/bitsandbytes-foundation/bitsandbytes/discussions/1339) |
| **Issue** | [Unsloth #4133](https://github.com/unslothai/unsloth/issues/4133) |

---

## Related Topics

- [ROCm-7x](./ROCm-7x.md) — ROCm 7.x GPU compatibility
- [HIP-SDK](./HIP-SDK.md) — HIP programming and PyTorch on AMD
