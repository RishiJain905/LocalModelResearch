# DGX Spark

> *"Powered by the NVIDIA Grace Blackwell architecture, DGX Spark enables developers, researchers, and data scientists to prototype, deploy, and fine-tune large AI models on their desktop."* — [NVIDIA DGX Spark Hardware Overview](https://docs.nvidia.com/dgx/dgx-spark/hardware.html)

## Contents

1. [Definitions & Overview](#definitions--overview)
2. [Hardware Specifications](#hardware-specifications)
3. [Performance Metrics](#performance-metrics)
4. [Pricing & Availability](#pricing--availability)
5. [Comparison with Alternatives](#comparison-with-alternatives)
6. [Use Cases](#use-cases)
7. [Key Findings](#key-findings)

---

## Definitions & Overview

### What is DGX Spark?

DGX Spark is NVIDIA's **personal AI desktop supercomputer** based on the Grace Blackwell architecture. It is a compact workstation-class system designed for developers, researchers, and data scientists to prototype, deploy, and fine-tune large AI models locally.

> *"A Grace Blackwell AI supercomputer on your desk."*
> — [NVIDIA Marketplace](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/)

### Pricing

> *$4,699.00* — [NVIDIA Marketplace](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/)

> *PNY preorder listing showed Nvidia DGX Spark at $4,299.99* — [Reddit / LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1ndz1k4/pny_preorder_listing_shows_nvidia_dgx_spark_at/)

> *"Special Bonus: Enjoy a free, DLI hands-on AI course included with your NVIDIA DGX Spark purchase ($90 value)."*
> — [NVIDIA Marketplace](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/)

---

## Hardware Specifications

### GB10 System-on-Chip (SOC)

| Component | Specification |
|-----------|---------------|
| **CPU** | 20-core Arm (10× Cortex-X925 + 10× Cortex-A725) |
| **GPU** | Blackwell-generation GPU with 5th-gen Tensor Cores and FP4 support |
| **AI Performance** | Up to 1,000 AI TOPS, 1 petaFLOP of FP4 performance |
| **Tensor Cores** | 5th Generation with FP4 support |
| **TDP** | 140W (GB10 SOC) |
| **System Power Budget** | 240W total (100W available for other components) |

> *"Powered by the NVIDIA Grace Blackwell architecture... 20-core Arm processor (10 Cortex-X925 + 10 Cortex-A725)"*
> — [NVIDIA DGX Spark Hardware Overview](https://docs.nvidia.com/dgx/dgx-spark/hardware.html)

### Memory & Storage

| Component | Specification |
|-----------|---------------|
| **System Memory** | 128 GB LPDDR5x, coherent unified memory |
| **Memory Bandwidth** | 273 GB/s |
| **Storage** | 4 TB NVMe M.2 (with self-encryption) |
| **Memory Type** | LPDDR5x (not HBM — desktop-class) |

> *"128 GB LPDDR5x, coherent unified system memory. 256-bit. Up to 273 GB/s. 4TB NVME.M2 with self-encryption."*
> — [Amazon](https://www.amazon.com/NVIDIA-DGX-SparkTM-Supercomputer-Blackwell/dp/B0FWJ16CCH)

### Connectivity

| Port/Feature | Specification |
|--------------|---------------|
| **Network** | 1× RJ-45 (10 GbE), ConnectX-7 Smart NIC |
| **Wireless** | Wi-Fi 7, Bluetooth 5.4 |
| **USB** | 4× USB Type-C (one for power delivery) |
| **Video** | 1× HDMI 2.1a, HDMI multichannel audio |
| **Power** | 240W power supply required for optimal performance |

> *"1x RJ-45 (10 GbE), ConnectX-7 Smart NIC, Wi-Fi 7, Bluetooth 5.4. 4x USB Type-C, 1x HDMI 2.1a, HDMI multichannel audio."*
> — [NVIDIA DGX Spark Hardware Overview](https://docs.nvidia.com/dgx/dgx-spark/hardware.html)

---

## Performance Metrics

### GPU Performance (FP16)

| Metric | Value |
|--------|-------|
| **FP16 Peak TFLOPS** | 100.0 |
| **AI TOPS** | Up to 1,000 |

> *"FP16 Peak TFLOPS: 100.0"* — [Flopper.io](https://flopper.io/gpu/nvidia-dgx-spark)

### NVIDIA Product Line Comparison

| Product | Target | GPU | Memory | Price |
|---------|--------|-----|--------|-------|
| **DGX Spark** | Personal/Desktop | GB10 (Blackwell) | 128 GB LPDDR5x | $4,699 |
| **DGX B200** | Enterprise/8-GPU | 8× B200 | 1.44 TB HBM3e | ~$515,000 |
| **DGX H100** | Enterprise/8-GPU | 8× H100 | 640 GB HBM3 | ~$460,000 |
| **RTX 5090** | Consumer | AD102 (Blackwell) | 32 GB GDDR7 | ~$1,999 |
| **GB200 NVL72** | Rack-scale | 72× B200 | 13.4 TB HBM3e | Rack-scale only |

### B200 vs GB10 Comparison

| Feature | B200 (Datacenter) | GB10 (DGX Spark) |
|---------|-------------------|-------------------|
| **Memory Type** | HBM3e | LPDDR5x |
| **Memory Bandwidth** | 8 TB/s | 273 GB/s |
| **FP4 TOPS (sparse)** | 18,000 | ~1,000 |
| **FP16 TFLOPS** | 1,800 | ~100 |
| **Memory Capacity** | 192 GB | 128 GB |
| **TDP** | 1,000W | 140W (SOC) |

> *GB10 delivers "up to 1,000 AI TOPS, or 1 petaFLOP of FP4 performance"* — [Reddit / LocalLLaMA](https://www.reddit.com/r/LocalLLaMA/comments/1ndz1k4/pny_preorder_listing_shows_nvidia_dgx_spark_at/)

---

## Pricing & Availability

| Item | Price | Source |
|------|-------|--------|
| **DGX Spark (NVIDIA official)** | $4,699 | [NVIDIA Marketplace](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/) |
| **DGX Spark (PNY preorder)** | $4,299.99 | [Reddit](https://www.reddit.com/r/LocalLLaMA/comments/1ndz1k4/pny_preorder_listing_shows_nvidia_dgx_spark_at/) |
| **Free DLI Course (bundled)** | $90 value | [NVIDIA Marketplace](https://marketplace.nvidia.com/en-us/enterprise/personal-ai-supercomputers/dgx-spark/) |
| **DGX B200 (8-GPU system)** | ~$515,000 | [Modal](https://modal.com/blog/nvidia-b200-pricing) |

---

## Comparison with Alternatives

| System | GPU | Memory | FP4 | Price | Best For |
|--------|-----|--------|-----|-------|----------|
| **DGX Spark** | GB10 | 128 GB LPDDR5x | ✅ | $4,699 | Personal dev/fine-tune |
| **DGX B200** | 8× B200 | 1.44 TB HBM3e | ✅ | ~$515,000 | Enterprise training |
| **RTX 5090** | AD102 | 32 GB GDDR7 | ✅ | ~$1,999 | Consumer AI |
| **Mac Studio M4 Ultra** | Apple UltraFusion | 192 GB unified | ❌ | $3,999 | Local inference (AppleSilicon) |
| **Lambda/Paperspace** | H100 cloud | Pay-per-use | ✅ | Varies | Cloud training |

> *"The NVIDIA DGX Spark is optimized for high-performance computing tasks with Grace Blackwell architecture delivering high TFLOPS of compute power."*
> — [Flopper.io](https://flopper.io/gpu/nvidia-dgx-spark)

### Key Trade-off vs Cloud

- DGX Spark: $4,699 upfront, 273 GB/s LPDDR5x bandwidth, FP4 support, no recurring costs
- Cloud B200: ~$515,000 upfront equivalent, 8 TB/s HBM3e bandwidth, no local hardware

---

## Use Cases

> *"Enables developers, researchers, and data scientists to prototype, deploy, and fine-tune large AI models on their desktop."*
> — [NVIDIA DGX Spark Hardware Overview](https://docs.nvidia.com/dgx/dgx-spark/hardware.html)

### Recommended For

1. **Local fine-tuning** of open-source LLMs (Llama, Mistral, Gemma)
2. **Prototyping** AI applications before cloud deployment
3. **Running quantized models** (FP4/NVFP4) locally
4. **Multimodal AI workloads** (vision + language models)
5. **AI coursework and learning** (DLI course included)

### Not Suitable For

- Large-scale training (needs DGX B200 or GB200 NVL72)
- Memory-bandwidth-bound inference at scale (HBM3e systems required)
- Production multi-user deployments

---

## Key Findings

- **$4,699** — NVIDIA's most affordable Blackwell system, bringing FP4 to the desktop
- **GB10 SOC**: 20-core Arm (Cortex-X925 + A725) + Blackwell GPU, 140W TDP
- **128 GB LPDDR5x** unified memory (not HBM) at 273 GB/s — significantly lower bandwidth than datacenter HBM3e
- **FP4 support**: First personal AI supercomputer with native NVFP4 Tensor Cores
- **1,000 AI TOPS / 1 PFLOPS FP4** — compact FP4 performance for local inference
- **vs DGX B200**: ~100x price difference, but GB10 lacks HBM3e and has 30x less memory bandwidth
- **vs RTX 5090**: Similar consumer GPU tier but with Arm CPU, more memory (128 vs 32 GB), and Grace Blackwell architecture

---

## Related Topics

- [FP4-Tensor-Cores](./FP4-Tensor-Cores.md) — NVFP4 precision supported on DGX Spark's GB10
- [CG-HBM](./CG-HBM.md) — Note: DGX Spark uses LPDDR5x, not HBM3e (unlike datacenter Blackwell)
