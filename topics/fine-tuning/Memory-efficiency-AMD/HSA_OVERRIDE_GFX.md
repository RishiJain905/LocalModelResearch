# HSA_OVERRIDE_GFX_VERSION

## 📌 Overview

| Field | Value |
|-------|-------|
| **Variable** | `HSA_OVERRIDE_GFX_VERSION` |
| **Purpose** | Override GPU architecture detection in AMD ROCm runtime |
| **Platform** | Linux only (does NOT work on Windows) |
| **Typical Use** | Enable ROCm on consumer RDNA2/RDNA3 GPUs not in official whitelist |

> "Your card is already talking to the driver — `rocminfo` proves it. The runtime just refuses to hand it work."

— [CraftRigs](https://craftrigs.com/troubleshooting/rocm-unsupported-amd-gpu-hsa-override-gfx-version/)

---

## What It Does

ROCm's HSA runtime queries an internal `amdgpu.ids` whitelist at initialization. If the GPU's GFX version is not on the list, the runtime rejects it — causing either **silent CPU fallback** or `"invalid device function"` errors.

`HSA_OVERRIDE_GFX_VERSION` intercepts this check after the driver loads but before runtime initialization, forcing the GPU to be treated as a supported architecture.

### Syntax

```bash
export HSA_OVERRIDE_GFX_VERSION={MAJOR}.{MINOR}.{PATCH}

# Examples:
export HSA_OVERRIDE_GFX_VERSION=10.3.0   # Override to gfx1030 (Navi 21/22)
export HSA_OVERRIDE_GFX_VERSION=10.3.1   # Override to gfx1031 (Navi 23)
export HSA_OVERRIDE_GFX_VERSION=11.0.0   # Override to gfx1100 (RDNA3)
export HSA_OVERRIDE_GFX_VERSION=9.4.2    # Override to gfx942 (MI300X)
```

**Use version strings like `10.3.0`, NOT raw gfx names like `gfx1031`.**

---

## GPU Override Matrix

| GPU | Native GFX | Override Target | Version | Status |
|-----|-----------|---------------|---------|--------|
| RX 6900 XT / 6950 XT | gfx1030 | None (native) | — | ✅ Official |
| RX 6800 XT / 6800 | gfx1030 | None (native) | — | ✅ Official |
| RX 6750 XT / 6700 XT | gfx1031 | gfx1030 | `10.3.0` | ⚠️ Community |
| RX 6650 XT / 6600 XT / 6600 | gfx1032 | gfx1030 | `10.3.1` | ⚠️ Community |
| RX 6500 XT / 6400 | gfx1034 | gfx1030 | `10.3.1` | ⚠️ Community |
| AMD Strix Halo | gfx1151 | gfx1100 | `11.0.0` | ⚠️ Community |
| RX 7900 XTX / 7900 XT | gfx1100 | None (native) | — | ✅ Official |
| AMD PRO W6600 | gfx1032 | gfx1030 | `10.3.0` | ⚠️ Community |
| MI300X | gfx942 | None (native) | — | ✅ Official |

---

## Key Sources

### CraftRigs — "Fixing ROCm for Unsupported AMD GPUs"

| URL | [craftrigs.com](https://craftrigs.com/troubleshooting/rocm-unsupported-amd-gpu-hsa-override-gfx-version/) |
|-----|------|

> "ROCm's kernel driver loads for every AMD GPU and `rocminfo` lists your card correctly, but the HSA runtime queries an internal `amdgpu.ids` whitelist and rejects non-supported gfx versions. The result is a silent fallback to CPU."

> "The community consensus is that this is market segmentation, not a technical limitation — the cards share identical RDNA2 compute units, ISAs, and memory controllers with supported models."

### ROCm GitHub Issue #1797 — MNIST Training on gfx1032

| URL | [github.com/ROCm/ROCm/issues/1797](https://github.com/ROCm/ROCm/issues/1797) |

**Without override:**
```
hipErrorNoBinaryForGpu: Unable to find code object for all current devices!
```

**With override `HSA_OVERRIDE_GFX_VERSION=10.3.0`:**
```
torch.cuda.is_available()
True
```
Training completed through Epoch 14 with 99% accuracy.

### Ollama GitHub Issue #8473 — RX 6600 Silent CPU Fallback

| URL | [github.com/ollama/ollama/issues/8473](https://github.com/ollama/ollama/issues/8473) |

**Without override:**
```
level=WARN source=amd_linux.go:378 msg="amdgpu is not supported (supported types:[gfx1030 gfx1100 gfx1101 gfx1102 gfx900 gfx906 gfx908 gfx90a gfx940 gfx941 gfx942])"
gpu_type=gfx1032
level=INFO source=amd_linux.go:404 msg="no compatible amdgpu devices detected"
```

**With override `HSA_OVERRIDE_GFX_VERSION=10.3.0`:**
```
level=INFO source=amd_linux.go:391 msg="skipping rocm gfx compatibility check"
level=INFO source=types.go:131 msg="inference compute" id=0 library=rocm compute=gfx1032
```

---

## Usage Examples

### Ollama (Docker)

```bash
docker run --rm -it --device=/dev/kfd --device=/dev/dri \
  -e HSA_OVERRIDE_GFX_VERSION=10.3.0 \
  -v $(pwd)/models:/models \
  ollama/ollama:rocm
```

### llama.cpp

```bash
HSA_OVERRIDE_GFX_VERSION=10.3.0 ./llama-cli -m model.gguf -p "Hello"
HSA_OVERRIDE_GFX_VERSION=10.3.0 ./llama-server -m model.gguf --host 0.0.0.0
```

### PyTorch ROCm

```bash
HSA_OVERRIDE_GFX_VERSION=10.3.0 python3
>>> import torch
>>> torch.cuda.is_available()
True
```

---

## Official ROCm Support (ROCM 7.1.0)

| GPU Family | GFX | Official? |
|-----------|-----|-----------|
| MI300X / MI325X / MI300A | gfx942 | ✅ |
| MI250X / MI250 / MI210 | gfx90a | ✅ |
| MI100 | gfx908 | ✅ |
| RX 7900 XTX / 7900 XT / 7800 XT | gfx1100 | ✅ |
| RX 6900 / RX 6800 | gfx1030 | ✅ |
| RX 6700 XT (Navi 22) | gfx1031 | ❌ Needs override |
| RX 6600 (Navi 23) | gfx1032 | ❌ Needs override |
| RX 6500 XT (Navi 24) | gfx1034 | ❌ Needs override |
| AMD Strix Halo | gfx1151 | ❌ Needs override |

---

## Limitations

### Windows NOT Supported

> "On Linux this doesn't work. I'm just curious as to why the hack doesn't work on Windows and is this circumventable somehow?"

— [ROCm/TheRock Issue #1719](https://github.com/ROCm/TheRock/issues/1719)

### Wrong Value Breaks Things

> "In the systemd unit, the environment variable `HSA_OVERRIDE_GFX_VERSION=10.3.0` is causing the loading of a model (llama3) to fail."

— [Arch Linux GitLab Issue #4](https://gitlab.archlinux.org/archlinux/packaging/packages/ollama/-/issues/4)

The user's GPU was `gfx1102`, so `10.3.0` (gfx1030 target) was incorrect.

### Multi-GPU Mixed Architecture Crash

> "Find device IDs via `rocminfo`, then restrict with: `HIP_VISIBLE_DEVICES=<device_id>`. Otherwise the runtime may crash if it sees two different architectures."

— [AMDGPU.jl](https://amdgpu.juliagpu.org/stable/install_tips)

---

## Validation

```bash
# Step 1: Check native GFX (unchanged)
rocminfo | grep gfx

# Step 2: Check with override
HSA_OVERRIDE_GFX_VERSION=10.3.0 rocminfo | grep "Device Type"
# Must return "Device Type: GPU" without HSA errors

# Step 3: First inference with debug logging
HSA_OVERRIDE_GFX_VERSION=10.3.0 OLLAMA_DEBUG=1 ollama run llama3.1:8b
# Look for "loaded ROCm library" and "using GPU"
```

---

## Related Topics

- [RDNA3-Optimization.md](./RDNA3-Optimization.md) — RDNA3 consumer GPU optimization
- [Memory-efficiency-AMD](./README.md) — Parent folder overview

---

## References

- [CraftRigs — Fixing ROCm for Unsupported AMD GPUs](https://craftrigs.com/troubleshooting/rocm-unsupported-amd-gpu-hsa-override-gfx-version/)
- [ROCm GitHub Issue #1797 — gfx1032 MNIST workaround](https://github.com/ROCm/ROCm/issues/1797)
- [Ollama Issue #8473 — RX 6600 silent fallback](https://github.com/ollama/ollama/issues/8473)
- [LocalAI Issue #5822 — gfx1151 workaround](https://github.com/mudler/LocalAI/issues/5822)
- [Arch Linux GitLab Issue #4 — Wrong override breaks Ollama](https://gitlab.archlinux.org/archlinux/packaging/packages/ollama/-/issues/4)
- [ROCm/TheRock Issue #1719 — Windows not supported](https://github.com/ROCm/TheRock/issues/1719)
- [AMD GPU Architecture Specifications](https://rocm.docs.amd.com/en/docs-7.0.1/reference/gpu-arch-specs.html)
